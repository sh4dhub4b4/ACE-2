# 🐞 Comprehensive Bug Fix & Incident Report
**Document ID:** ECI-BFR-2026-001  
**Incident Name:** The "Phantom Cache" EOFError Crash  
**Target Architecture:** Polyglot Microservices Platform (ECI) - K8s Sandboxing Engine  
**Status:** RESOLVED (Hotfixed via Dynamic Build Pipeline)  
**Severity:** CRITICAL (Total execution failure for `stdin` dependent payloads)  

---

## 📊 1. Count-Driven Incident Metrics
* **Total Downtime / Debugging Hours:** ~14 Hours
* **Failed Execution Attempts (Exit Code 1):** 47+
* **Redundant Docker Builds Executed:** 22
* **Zombie Sandbox Pods Terminated:** 100+ (`eci-sandboxes` namespace)
* **Actual Lines of Code Changed to Fix the Bug:** 2 Lines in `api-gateway` (Relative import fix), 3 Lines in `Makefile` (Dynamic tagging), 2 Lines in `k8s_manager.py` (Env injection).
* **Number of `EOFError` Exceptions Logged:** 50+
* **Number of Times Cache Deceived the Developer:** Too many to count.

---

## 🔍 2. Executive Summary
The ECI (Educational Cloud Infrastructure) platform experienced a critical failure where Python code requiring standard input (`stdin`) crashed immediately with an `EOFError: EOF when reading a line` upon execution inside the secure C++ sandbox. While the API Gateway and Orchestrator were successfully routing the `stdin_data` payload, the K8s execution pod was consistently dropping the data. 

The investigation revealed a two-fold issue: a missing file descriptor redirection (`freopen`) in the C++ engine (`PythonStrategy.hpp`), heavily compounded by a vicious Docker/Kubernetes caching anomaly. The caching issue created a "mirage" where local code updates were never actually deployed to the isolated K8s pods, trapping the developer in an infinite loop of false-negative tests.

---

## 🧩 3. Root Cause Analysis (Technical Breakdown)

### A. The Core Execution Flaw
In the original `PythonStrategy.hpp`, the child process (spawned via `fork()`) was executing Python using `execlp()` without mapping the container's standard input to the dynamically generated `/tmp/in.txt` file. Consequently, when the user's Python code called `input()`, it attempted to read from the non-existent interactive terminal rather than the injected payload, immediately triggering an `EOFError` and an Exit Code 1.

### B. The Gateway Import Crash
Simultaneously, a minor syntax error in `src/api-gateway/src/main.py` (`from .presentation.execution_ws` instead of `from presentation.execution_ws`) caused the API Gateway to crash loop, leading to `websockets.exceptions.InvalidMessage` and immediate connection closures during the E2E gauntlet test.

---

## 🛑 4. The Kubernetes & Docker "Latest Tag" Cache Hassle (The Real Nightmare)
*(Post-Mortem of the DevOps Friction)*

While the C++ I/O bug was a standard algorithmic oversight, the true nightmare of this incident—accounting for 95% of the debugging time—was the psychological warfare waged by Docker Desktop's local registry and Kubernetes' `containerd` image caching policies. This specific operational friction transformed a 5-minute code fix into an hours-long architectural wild goose chase.

The hassle originated from the usage of the static `:latest` tag for the microservices (`eci-cpp-engine:latest`). When the `PythonStrategy.hpp` file was updated locally with the correct `freopen` logic, the build command (`docker build --no-cache ...`) was executed perfectly. The developer watched the Docker daemon painstakingly rebuild the Ubuntu base, install dependencies, compile the C++ binary via `g++`, and export the new image. The terminal boldly declared `FINISHED`. 

However, when `kubectl rollout restart` was triggered, Kubernetes committed the ultimate betrayal. Governed by the `image_pull_policy="IfNotPresent"` directive in the `k8s_manager.py` file, the Kubelet inspected the local node's container cache. It saw a request for `eci-cpp-engine:latest`. It looked in its registry, saw an image *already* named `eci-cpp-engine:latest` (the broken version from hours ago), and unilaterally decided to skip the pull process. It instantly spawned the new sandbox using the old, broken binary. 

This created a maddening disconnect: the code in the IDE was perfect, the Docker build was fresh, yet the isolated `/app/src/PythonStrategy.hpp` inside the live K8s pod stubbornly remained outdated. The developer was forced to dive into the K8s trenches—manually port-forwarding, executing `kubectl exec -it` into ephemeral sandboxes, and running raw `cat` commands against the C++ headers just to prove sanity. To bypass this aggressive optimization, the developer had to brutally nuke the local Docker builder cache (`docker builder prune --all --force`), manually delete dozens of stuck pods, and fundamentally redesign the deployment architecture to use mathematically unique, timestamp-based dynamic tags (`v1779061833`). This ensured that Kubernetes was forcefully evicted from its lazy caching habits, compelling it to strictly honor the developer's latest compiled artifacts. The sheer operational overhead of outsmarting the orchestrator's "helpful" caching layers proved to be the most exhausting hurdle of the entire infrastructure lifecycle.

---

## 🛠️ 5. Implementation of the Fix

To permanently eradicate this class of deployment synchronization bugs, a robust **Dynamic Versioning Pipeline** was implemented:

1. **Auto-Bumping Makefile Pipeline:** Replaced static build commands with a dynamic `TAG := $(shell date +%s)` injection. Every single execution of `make all` now guarantees a mathematically unique image tag (e.g., `v177906...`), completely bypassing all K8s cache traps.
2. **Dynamic Orchestrator Injection:** Updated the `Makefile` to dynamically patch the Orchestrator's environment variables via `kubectl set env deployment/eci-orchestrator CPP_ENGINE_TAG=v$(TAG)`.
3. **Environment-Aware Provisioning:** Modified `k8s_manager.py` to read `os.getenv("CPP_ENGINE_TAG")` instead of relying on a hardcoded string, ensuring every new student sandbox strictly spawns from the exact image compiled seconds prior.
4. **I/O Redirection Secured:** Successfully propagated the `freopen` logic into the C++ sandbox, ensuring Python flawlessly reads `stdin_data` from the orchestrator payload.

**System Status:** 100% Operational. E2E execution tests (including advanced SHA-256 cryptographic payloads) are now yielding `Exit Code: 0`.
