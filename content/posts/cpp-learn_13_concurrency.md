---
author: "Phong Nguyen"
title: "C++ - Chapter 13: Concurrency"
date: "2026-06-14"
description: "C++ Notes"
tags: ["cpp"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
# Concurrency
**Concurrency** refers to the ability of a program to make progress on multiple tasks during the same period of time.
- On a single-core CPU, concurrency is typically achieved through `context switching`
- On a multi-core CPU, concurrent tasks may execute in `parallel`
- It's used to improve the `program performance` and `response time`

Concurrency in C++ can be implemented using several approach:
- STD Thread <thread> (C++11)
- Async/Future <future> (C++11) 
- Coroutines <coroutine> (C++20)

---
## 1. Thread-Based Concurrency
### 1.1. Threads <thread>
**A thread** is the basic unit of execution within a process (multitasking.)
- Each thread has its own call stack, while all threads in the same process share the heap and other process resources.
- Use threads when running long-lived or complex tasks that need independent execution.

There are many errors and risks associated with concurrency, including:
  - `Deadlocks` refers to the situation where two or more threads are blocked, waiting for each other indefinitely
  - `Race condition` refers to the situation where two or more threads access shared data concurrently, leading to the `undefined behavior`.
  - `Starvation` refers to the situation where a thread is unable to gain regular access to the shared resources.
=> We can avoid these problems by `proper synchronization` between the threads.
<br>

### 1.2. Thread Management
A **thread** is an OS-level thread managed by the kernel. Each thread has it own **call stack**, but all thread` share the **heap**.
- `thread object` is a C++ instance associated with an active thread of execution in hardware level.
- `std::thread(callable)` requests the kernel OS to create a new thread.
- `std::this_thread` refers to the current thread

**Lifecycle control methods:**
- `join()` blocks the current thread until this* thread (a.ka. object thread) finishes its execution. If the exception
is throw before `join`, `std::terminate` might be called, and will kill the entire program process, not just the thread.
- `detach()` separates the `thread of the execution` from the its `thread object`, letting it continue running independently in the background.
- `yield()` voluntarily pauses the current thread to give other threads a chance to run.
- `return` use to to kill a thread's by returning from its callable
> Always ensure a thread is either `join()` or `detach()` before the std::thread object is destroyed. otherwise std::terminate() is called.
<br>

### 1.3. Sharing Data
- `Global/Static Variable`: can be accessed by all threads.
- `Pass By Reference`: we need to explicitly wrap the args in `std::ref` to pass by reference and is the only way to properly get data out of a thread
- `thread_local` creates a separate instance of a static variable per thread, each thread gets its own copy with independent lifetime.
<br>

## 2. Synchronization
Synchronization ensures safe access to shared resources across threads. It can be done by using the following components:
- **Mutex/Lock** `<mutex>` they are used to protect the shared resources, ensure that only one thread can access `the critical sections` at a time.
- **Semaphore** `<semaphore>` (C++20) control access by limiting concurrent thread count 
- **Futures & Promises** `<future>/<promise>` pass results/exceptions between threads asynchronously
- **Condition Variable** `<condition_variable>` block threads until a condition is signaled
<br>

### 2.1. Mutexes and Locks / <mutex>
**A mutex** (mutual exclusion object) is owned by the thread that acquires it. Only the owning thread may access the protected critical section.

#### Raw Mutex Usage
```cpp
#include <mutex>

// Create your mutex here
std::mutex my_mutex;

thread_function()
{
  my_mutex.lock(); // Acquire lock
  // Do some non-thread safe stuff...
  my_mutex.unlock(); // Release lock
}
```

**Mutex Types:**
- `std::mutex` Basic, non-recursive mutex
- `std::timed_mutex` Adds try_lock_for() / try_lock_until()
- `std::recursive_mutex` Same thread can lock multiple times without deadlocking
- `std::recursive_timed_mutex` Recursive + timed
- `std::shared_mutex` (C++17)Supports shared (read) and exclusive (write) ownership
- `std::shared_timed_mutex` Shared + timed
<br>

#### Lock Guard Types (RAII Wrappers)
**Lock Guard Type** is a wrapper mutex that provides a convenient RAII-style mechanism. (lock on construction and automatically unlock on destruction, even if an exception is thrown.)
- `std::lock_guard<mutex>`: The simples lock. Locks immediately, unlocks on scope exit. No manual control.
- `std::scoped_lock<mutex1,mutex2,...>`: lock for multiple mutexes simultaneously. Avoids deadlock via a deadlock-avoidance algorithm.
- `std::unique_lock<mutex>`: lock for a single mutex. Supports deferred locking, manual `lock()/unlock()`, and is required for use with `std::condition_variable`
- `std::shared_lock<shared_mutex>`: lock for shared (read) access. Multiple threads can hold a shared lock concurrently; exclusive (write) locks still block.
<br>

### 2.2. Condition Variable / <condition_variable>
**A `std::condition_variable`** is a synchronization primitive that works alongside a `std::mutex` to block one or more threads until another thread both **modifies** a `shared variable` (the condition) and **notifies** the `std::condition_variable`.

**Methods:**
- `wait(lock, predicate)` releases the lock and blocks the thread until the predicate returns true. Re-acquires the lock before returning.
- `notify_one()` wakes one waiting thread
- `notify_all()` wakes all waiting threads.

```cpp
#include <condition_variable>
#include <iostream>
#include <mutex>
#include <string>
#include <thread>
 
std::mutex m;
std::condition_variable cv;
std::string data;
bool ready = false;
bool processed = false;
 
void worker_thread() {
    // 1. wait until main() sends data
    std::unique_lock lk(m);
    cv.wait(lk, []{ return ready; });
 
    // after the wait, we own the lock
    std::cout << "Worker thread is processing data\n";
    data += " after processing";
 
    // send data back to main()
    processed = true;
    std::cout << "Worker thread signals data processing completed\n";
 
    // manual unlocking is done before notifying, to avoid waking up
    // the waiting thread only to block again (see notify_one for details)
    lk.unlock();
    cv.notify_one();
}
 
void main() {
    std::thread worker(worker_thread);
 
    data = "Example data";
    // send data to the worker thread
    {
        std::lock_guard lk(m);
        ready = true;
        std::cout << "main() signals data ready for processing\n";
    }
    cv.notify_one();
 
    // wait for the worker
    {
        std::unique_lock lk(m);
        cv.wait(lk, []{ return processed; });
    }
    std::cout << "Back in main(), data = " << data << '\n';
 
    worker.join();
}

// Output:
// main() signals data ready for processing
// Worker thread is processing data
// Worker thread signals data processing completed
// Back in main(), data = Example data after processing
```
<br>

### 2.3. Atomic Operation /  <atomic>
**An atomic type** is a type that implements atomic operations. It's used to guarantee no race conditions will occur. (e.g. `std::atomic<bool>` - `std::atomic_bool`)
```cpp
#include <atomic>
#include <iostream>
#include <thread>
#include <vector>
 
std::atomic_int acnt;
int cnt;
 
void f() {
    for (auto n{10000}; n; --n) {
        ++acnt;
        ++cnt;
        // Note: for this example, relaxed memory order is sufficient,
        // e.g. acnt.fetch_add(1, std::memory_order_relaxed);
    }
}
 
void main() {
    {
        std::vector<std::jthread> pool;
        for (int n = 0; n < 10; ++n)
            pool.emplace_back(f);
    }
 
    std::cout << "The atomic counter is " << acnt << '\n'
              << "The non-atomic counter is " << cnt << '\n';
}
```
<br>

---
## 3. Task
A task is a unit of asynchronous work scheduled for execution. It is a higher-level abstraction than a thread.
Tasks are generally preferred over raw threads because:
- The runtime can reuse threads across tasks via a thread pool, avoiding the overhead of creating and destroying OS threads.
- Scheduling is managed automatically by the runtime, rather than manually by the programmer.
- Less boilerplate, no need to manage join(), detach(), or shared state explicitly.

### 3.1. Promises & Futures <future>
```cpp
/// PROMISE / FUTURE

    [Producer Thread]                  [Consumer Thread]
    std::promise<T>                    std::future<T>
          |                                   |
          | get_future() // main bind         |
          +---------------------------------->+
   set_value()     |                          | 
   set_exception() v                          v get()/wait()
                          Shared State
                        +---------------+
                        |   Value /     |
                        |  Exception    |
                        +---------------+
```
#### std::future<T>
A class template that holds a value to be assigned at some point in the future, providing a way to retrieve it once ready.
- blocks the calling thread if `.get()` is called before the value is assigned.
- returned by async operations: `std::async`, `std::promise`, and `std::packaged_task`
- move-only - cannot be copied; only one thread can retrieve the value.

#### std::shared_future<T>
Works the same way as `std::future`, but copyable , so multiple threads are allowed to wait for the same shared state.

#### std::promise<T>
The write end (write side) of a future/promise pair. Stores a value or exception that will be retrieved asynchronously by its associated `std::future`.
  - It is commonly used to pass results between threads.
  - It creates a `std::future` using `get_future()`.
  - The `std::promise` and the `std::future` share a common shared state.
  - The `std::promise` sets the value of the shared state using `set_value()`.
  - The associated `std::future` retrieves the value using `get()`.

**Methods:**
- `get_future()` creates and returns the associated std::future
- `set_value(v)` writes the value into the shared state, unblocking the future
- `set_exception(e)` stores an exception instead of a value

#### Example
```cpp
#include <future>
#include <thread>

void producer(std::promise<int> p) {
    p.set_value(42);              // write result into shared state
}

void main() {
    std::promise<int> p;
    std::future<int> f = p.get_future();  // bind future to promise

    std::thread t(producer, std::move(p));  // promise is move-only

    int result = f.get();         // blocks until set_value() is called
    // result == 42

    t.join();
}
```
<br>

### 3.2. Async & Future <future>
```cpp
/// ASYNC / FUTURE

[Worker Thread]                    [Main Thread]
Callable                           std::future<T>
    |                                     ^
    | return value                        |
    | throw exception                     |
    +-------------------------------------+
                     |
                     v
                Shared State
              +---------------+
              | Value / Ex    |
              +---------------+
```
**Async** is a function template that spawns asynchronous work, returns a `std::future` to retrieve the result once it's ready, without manually managing threads or promises.
```cpp
auto future = std::async(some_function, arg1, arg2);
// ... do other work ...
auto result = future.get();  // blocks until result is ready
```
- A call to `std::async` returns a `std::future` object.
- The future can later be used to retrieve the result of the asynchronous computation.
- `std::async` accepts an optional launch policy as its first argument, controlling when and how the function runs:
  - `std::launch::async`: Runs in a new thread, immediately
  - `std::launch::deferred`: Lazy, runs on the calling thread only when `.get()` or `.wait()` is called
  - Default: defer to system 
    ```cpp
    // Pass in function pointer
    auto future = std::async(std::launch::async, some_function, arg_1, arg_2);

    // Pass in function reference
    auto future = std::async(std::launch::async, &some_function, arg_1, arg_2);

    // Pass in function object
    struct SomeFunctionObject {
        void operator() (int arg_1){}
    };
    auto future = std::async(std::launch::async, SomeFunctionObject(), arg_1);

    // Lambda function
    auto future = std::async(std::launch::async, [](){});
    ```

#### Example
```cpp
#include <future>
int producer() {
    return 42;
}

void main() {
    std::future<int> f = std::async(std::launch::async, producer);
    int result = f.get(); // blocks until producer() returns
    // result == 42
}
```
<br>

---
## 4. Coroutine 
TBD
<!-- C++ 20 Task that can pause and resume without blocking a thread -->

---