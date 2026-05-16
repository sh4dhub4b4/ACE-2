# 🏗️ ২. Future Language Addition (The Architect's Blueprint)

যেহেতু তুমি মাইক্রোসার্ভিস আর্কিটেকচারে **Strategy Pattern** ব্যবহার করেছো, ভবিষ্যতে নতুন ল্যাঙ্গুয়েজ (যেমন: `C++` বা `Golang`) অ্যাড করা এখন ডাল-ভাত।

ভবিষ্যতে নতুন ল্যাঙ্গুয়েজ যোগ করতে হলে ঠিক এই **৪টি কাজ** করতে হবে:

---

## ১. Create the Strategy Class

`IExecutionStrategy` কে ইনহেরিট করে নতুন একটি হেডার ফাইল বানাবে।

উদাহরণ:

* `CppStrategy.hpp`
* `GolangStrategy.hpp`

---

## ২. Implement Compilation & Execution

### 🔹 Interpreted Language (Python, Node.js)

সরাসরি `execlp()` দিয়ে রান করবে।

### 🔹 Compiled Language (C++, Go)

`execute()` ফাংশনের ভেতরে:

1. প্রথমে কম্পাইল কমান্ড রান করবে
   উদাহরণ:

```cpp
g++ code.cpp -o run
```

2. তারপর কম্পাইল হওয়া বাইনারিটা `execlp()` দিয়ে রান করবে।

---

## ৩. Update the Orchestrator

`SandboxOrchestrator.hpp` এর `set_language()` মেথডে নতুন ল্যাঙ্গুয়েজের কন্ডিশন অ্যাড করবে।

```cpp
else if (language == "cpp") {
    strategy = std::make_unique<CppStrategy>();
}
```

---

## ৪. Update Dockerfile

ওই ল্যাঙ্গুয়েজ রান করার জন্য প্রয়োজনীয় ডিপেন্ডেন্সিগুলো `eci-cpp-engine` এর Dockerfile-এ `apt-get install` দিয়ে ইনস্টল করবে।

উদাহরণ:

* `gcc`
* `g++`
* `golang`

---

# ✅ Final Result

গেটওয়ে বা ক্লাস্টারের অন্য কোথাও **একটা লাইনও চেঞ্জ করতে হবে না!**

এটাই Strategy Pattern + Microservice Architecture এর সবচেয়ে বড় শক্তি —
**Open for extension, closed for modification.**
