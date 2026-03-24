---
author: "Phong Nguyen"
title: "System C"
date: "2026-03-23"
description: "System C Notes"
tags: ["c, embedded,systemc"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
## 1. Introduce
- `SystemC` is a C++ class library, that provides a mechanism for managing complex systems involving large numbers of components.
- `SystemC` `is capable of` modeling hardware and software together at multiple level of abstraction.
> Modeling is the process of creating a simplified version of a real system.
Simulation is the process of executing the model over time to see how it behaves.
- [Reference](https://learnsystemc.com/)

## 2. Environment Setup
```bash
$ cat Dockerfile
FROM ubuntu:24.04

# Avoid interactive prompts
ENV DEBIAN_FRONTEND=noninteractive

# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    wget \
    git \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /opt

# Download SystemC (you can change version if needed)
RUN wget https://www.accellera.org/images/downloads/standards/systemc/systemc-2.3.3.tar.gz \
    && tar -xzf systemc-2.3.3.tar.gz \
    && cd systemc-2.3.3 \
    && mkdir build && cd build \
    && ../configure \
    && make -j$(nproc) \
    && make install

# Set environment variables
ENV SYSTEMC_HOME=/usr/local/systemc-2.3.3
ENV LD_LIBRARY_PATH=$SYSTEMC_HOME/lib-linux64:$LD_LIBRARY_PATH

# Default shell
WORKDIR /workspace
CMD ["/bin/bash"]
```

## 3. Hello World
- Header files: `<systemc.h>` or `<systemc>`
- Entry point: `int sc_main(int argc, char* argv[])` (systemC library has the `main()` function already defined)
- SystemC module: is a class that inherits the `sc_module` base class.

```bash
$ ls
Makefile  helloworld.cpp
$ cat Makefile
TARGET = out

IDIR = .
SDIR = .
ODIR = .

SRC = $(wildcard $(SDIR)/*.cpp)
OBJ = $(SRC:$(SDIR)/%.c=$(ODIR)/%.o)

CXX = g++
CXXFLAGS = -I$(IDIR)
CXXFLAGS += -g -O0
CXXFLAGS += -Iinclude
CFLAGS += -Wall
SCPATH = /usr/local/systemc-2.3.3
LIBS = -lm

$(TARGET): $(OBJ)
        $(CXX) $(CXXFLAGS) $(LDFLAGS) -I$(SCPATH)/include -L. -L$(SCPATH)/lib-linux64 -Wl,-rpath $(SCPATH)/lib-linux64 $^ $(LIBS) -o $@ -lsystemc

$(ODIR)/%.o: $(SDIR)/%.c
        $(CXX) $(CXXFLAGS) $(CFLAGS) -c $< -o $@

clean:
        $(RM) $(TARGET)

$ cat helloworld.cpp
#include <systemc.h>  // systemc library header

// Define a SystemC module
struct MyModule : sc_module{
        void helloWorld(){
                std::cout << "Hello SystemC World!\n\n";
        }

        // System C module constructor
        SC_CTOR(MyModule){
                // register a member function to the kernel
                SC_METHOD(helloWorld);
        }
};

int sc_main(int argc, char* argv[]){
        // instantiate a SystemC module
        MyModule hello_world_module("hw_module");

        // let systemC simulation kernel
        sc_start();

        return 0;
}

$ make
$ ./out

        SystemC 2.3.3-Accellera --- Oct  4 2020 22:59:38
        Copyright (c) 1996-2018 by all Contributors,
        ALL RIGHTS RESERVED
Hello SystemC World!
```

## 4. Language Basics
### 4.1. Module - SC_MODULE / sc_module
- A **systemC module** is the `smallest unit` that contains `state, behavior, and structure`, and support hierarchical connectivity.
  - It inherits the `sc_module` class.
  - It used to represent a component in read systems.

- To define a module:
  - Use `SC_CTOR` or `SC_HAS_PROCESS` for create the constructor
  - Shall have at least one constructor, if a module has no simulation process, don't use this.
  - Shall have one param of class `sc_module_name`, (then access via `name()`)
  - Use the explicit constructor to provide user-defined arguments to the constructor.
```cpp
// 1. using defined macro
SC_MODULE(module_A){
    SC_CTOR(module_A){
        std::cout << name() << " constructed\n";
    }
};

// 2. struct
struct module_B : public sc_module{
    SC_CTOR(module_B){
        std::cout << name() << " constructed\n";
    }
};

// 3. class
class module_C : public sc_module{
    public:
    SC_CTOR(module_C){
        std::cout << name() << " constructed\n";
    }
}; 

class module_d : public sc_module{
        private:
         int x_,y_;
        public:
         SC_CTOR(module_d);    // become useless, confuse for readers,reviewers => SC_HAS_PROCESS

         void printout(){
                std::cout << "Print out: " << name() << " x=" << x_ << " y=" << y_;
                std::cout << std::endl;
         }

         // explicit the constructor
         module_d(sc_module_name name, int x, int y): sc_module(name),x_{x}, y_{y}{
                SC_METHOD(printout);
         }
};

class module_e : public sc_module{
        private:
         int x_,y_;
        public:

         void printout(){
                std::cout << "Print out: " << name() << " x=" << x_ << " y=" << y_;
                std::cout << std::endl;
         }

         // use SC_HAS_PROCESS
         SC_HAS_PROCESS(module_e);

        module_e(sc_module_name name, int x, int y): sc_module(name),x_{x}, y_{y}{
                SC_METHOD(printout);
         }
};

int sc_main(int argc, char* argv[]){
    module_A modA("my_module_a");
    module_B modB("my_module_b");
    module_C modC("my_module_c");
    module_d m_d("my_module_d",2,3);
    module_e m_e("my_module_e",99,100);
    sc_start();
}

$ ./out

        SystemC 2.3.3-Accellera --- Oct  4 2020 22:59:38
        Copyright (c) 1996-2018 by all Contributors,
        ALL RIGHTS RESERVED
my_module_a constructed
my_module_b constructed
my_module_c constructed
Print out: my_module_d x=2 y=3
Print out: my_module_e x=99 y=100
```

### 4.2. Simulation Process
- A simulation process is a member function of the `sc_module` that has no in/return, and `is registered with the simulation kernel`.
- There are three ways to register a simulation process, following:
  - `SC_METHOD(func)`: none
  - `SC_THREAD(func)`: has its own thread of execution, may consume simulated time, can be suspended/ `wait()`
  - `SC_CTHREAD(func)`: that similar to the `SC_THREAD` but only have a static sensitivity of a clock edge event ?
- We can register the simulation with the simulation kernel in:
  - `constructor`
  - `before_end_of_elaboration` or `end_of_elaboration` callbacks of a module

```cpp
#include <systemc.h>
SC_MODULE(ProcessModule) {
  sc_clock clk;

  // sc method
  void method() {
    std::cout << "\tSC_METHOD triggered at: " << sc_time_stamp() << '\n';

    next_trigger(sc_time(1, SC_SEC));  // trigger after 1 sec
  }

  // sc_thread
  void thread() {
    while (true) {
      std::cout << "\t\tSC_THREAD method triggered at: " << sc_time_stamp()
                << '\n';
      wait(2, SC_SEC);  // wait 2 sec
    }
  }

  // sc_cthread
  void cthread() {
    while (true) {
      std::cout << "\t\t\tSC_CTHREAD triggered at: " << sc_time_stamp() << '\n';
      wait();  // wait for next clk event
    }
  }

  SC_CTOR(ProcessModule) : clk("clk", 5, SC_SEC) {
    SC_METHOD(method);
    SC_THREAD(thread);
    SC_CTHREAD(cthread, clk);  // input clk T = 5
  }
};

int sc_main(int, char*[]) {
  // Create module
  ProcessModule m_module("my_process_module");

  std::cout << "execution phase begin @ " << sc_time_stamp() << std::endl;
  sc_start(10, SC_SEC);  // run simulation for 10 secs
  std::cout << "execution phase ends @ " << sc_time_stamp() << std::endl;

  return 0;
}
```
```bash
$ ./app
        SystemC 2.3.3-Accellera --- Mar 23 2026 15:03:48
        Copyright (c) 1996-2018 by all Contributors,
        ALL RIGHTS RESERVED
execution phase begin @ 0 s
        SC_METHOD triggered at: 0 s
                SC_THREAD method triggered at: 0 s
                        SC_CTHREAD triggered at: 0 s
        SC_METHOD triggered at: 1 s
        SC_METHOD triggered at: 2 s
                SC_THREAD method triggered at: 2 s
        SC_METHOD triggered at: 3 s
        SC_METHOD triggered at: 4 s
                SC_THREAD method triggered at: 4 s
        SC_METHOD triggered at: 5 s
                        SC_CTHREAD triggered at: 5 s
        SC_METHOD triggered at: 6 s
                SC_THREAD method triggered at: 6 s
        SC_METHOD triggered at: 7 s
        SC_METHOD triggered at: 8 s
                SC_THREAD method triggered at: 8 s
        SC_METHOD triggered at: 9 s
execution phase ends @ 10 s
```

### 4.3. Simulation Stages
- SystemC application has three phases/stages of operation:
  - `Elaboration`: execution of statements prior to `sc_start()`, to create internal data structures.
  - `Execution`: includes two main stages **initialization & simulation**
  - `Cleanup or post-processing`: destroy objects, releases memory, and close open files,...

> Execution: further break-down to two stages:
    a) Initialization
      simulation kernel identifies all simulation processes and place them in either a runnable or waiting process set.
      All simulation processes are in runnable set except those requesting "no initialization".
    b) Simulation
      is commonly described as a state machine that schedules processes to run, and advances simulation time. It has two internal phases:
        1) evaluate: run all runnable processes one at a time. Each process runs till reaches wait() or return. Stops if no runnable processes left.
        2) advance-time: once the set of runnable processes is emptied, simulation enters advance-time phase where it does:
          a) move simulated time to the closest time with a scheduled event
          b) move processes waiting for that particular time into the runnable set
          c) return to evaluation phase
        The progression from evaluate to advance-time continues until one of the three things occurs. Then it moves to the cleanup phase.
          a) all processes have yielded
          b) a process has executed sc_stop()
          c) maximum time is reached
          Evaluation phase
          Update phase
          Delta notification phase

- There are four callback functions are called by the kernel during these stages:
  - `virtual void before_end_of_elaboration()`: is called `after the construction` of the module hierarchy.
  - `virtual void end_of_elaboration()`: is called at the end of elaboration (completion of any instantiation or port binding, ...)
  - `virtual void start_of_simulation()`: is called on the `first call` to `sc_start`
  - `virtual void end_of_simulation()`: is called when the scheduler halts because of `sc_stop` or at the very end of simulation, only once.

- e.g.
```cpp
#include <systemc.h>

class StageModule : public sc_module {
 public:
  SC_HAS_PROCESS(StageModule);

  StageModule(sc_module_name name);
  ~StageModule();

  void before_end_of_elaboration() override;
  void end_of_elaboration() override;
  void start_of_simulation() override;
  void end_of_simulation() override;

 private:
  void thread();
};

StageModule::StageModule(sc_module_name name) {
  std::cout << sc_time_stamp() << " Module: " << name
            << " Elaboration: constructor\n";
  SC_THREAD(thread);
}

StageModule::~StageModule() {
  std::cout << sc_time_stamp() << " Module: " << name()
            << " Cleanup: destructor\n";
}

void StageModule::thread() {
  std::cout << sc_time_stamp() << " Module: " << name() << " Execution: init\n";
  int index = 0;
  while (true) {
    wait(1, SC_SEC);
    std::cout << sc_time_stamp() << " Module: " << name()
              << " Execution: sim\n";
    if (index == 10) {
      sc_stop();
    }
    index++;
  }
}

void StageModule::before_end_of_elaboration() {
  std::cout << sc_time_stamp() << " Module: " << name()
            << " before_end_of_elaboration\n";
}

void StageModule::end_of_elaboration() {
  std::cout << sc_time_stamp() << " Module: " << name()
            << " end_of_elaboration\n";
}

void StageModule::start_of_simulation() {
  std::cout << sc_time_stamp() << " Module: " << name()
            << " start_of_simulation\n";
}

void StageModule::end_of_simulation() {
  std::cout << sc_time_stamp() << " Module: " << name()
            << " end_of_simulation\n";
}

int sc_main(int, char*[]) {
  // Create module
  StageModule m_module("my_stage_module");

  std::cout << "execution phase begin @ " << sc_time_stamp() << std::endl;
  sc_start(30, SC_SEC);  // run simulation for 30 secs
  std::cout << "execution phase ends @ " << sc_time_stamp() << std::endl;

  return 0;
}
```
```bash
$ ./app # Run the application to observe the output

        SystemC 2.3.3-Accellera --- Mar 23 2026 15:03:48
        Copyright (c) 1996-2018 by all Contributors,
        ALL RIGHTS RESERVED
0 s Module: my_stage_module Elaboration: constructor
execution phase begin @ 0 s
0 s Module: my_stage_module before_end_of_elaboration
0 s Module: my_stage_module end_of_elaboration
0 s Module: my_stage_module start_of_simulation
0 s Module: my_stage_module Execution: init
1 s Module: my_stage_module Execution: sim
2 s Module: my_stage_module Execution: sim
3 s Module: my_stage_module Execution: sim
4 s Module: my_stage_module Execution: sim
5 s Module: my_stage_module Execution: sim
6 s Module: my_stage_module Execution: sim
7 s Module: my_stage_module Execution: sim
8 s Module: my_stage_module Execution: sim
9 s Module: my_stage_module Execution: sim
10 s Module: my_stage_module Execution: sim
11 s Module: my_stage_module Execution: sim

Info: /OSCI/SystemC: Simulation stopped by user.
11 s Module: my_stage_module end_of_simulation
execution phase ends @ 11 s
11 s Module: my_stage_module Cleanup: destructor
```

### 4.4. Time Notation
- There is two type of `time measurements`:
  - `wall-clock time`: is the time `from the start` of execution `to completion`.
  - `simulated time`: is the time being modeled by the simulation, which maybe less/greater than the simulation's wall-clock time.
- `sc_time` is the data type used by simulation kernel to track simulated time. `SC_SEC, SC_MS, _US, _NS, _PS, _FC`
- `sc_time(1, SC_FS)`: to create a `sc_time` `(sc_time( double v, sc_time_unit tu ))`
- `sc_time_stamp()` to get the current simulated time.
- `SC_ZERO_TIME` is a macro representing a time value of zero.
  
### 4.5. Concurrency
- SystemC is not true concurrent execution, the processes are simulated as running concurrently, only one is executed at a particular time.
- The processes are running on the same `simulated time` because the simulated time `remain unchanged` until they finished.

### 4.6. Event
- An event is an object of `sc_event` class, that used for process synchronization.
- A process instance maybe triggered or resumed on the occurrence event.
- It has the following methods:
  - `void notify()`: to create an immediate notification.
  - `void notify(sc_time)/(double,sc_time_unit)`: to create a `timed notification`
  - `cancel()`: delete any pending notifications for this event
  
- e.g.
```cpp
#include <systemc>

using namespace sc_core;

class EventModule : public sc_module {
 public:
  SC_CTOR(EventModule) {
    SC_THREAD(thread_trigger);
    SC_THREAD(thread_catcher);
  }

 private:
  sc_event e_;

  void thread_trigger() {
    while (true) {
      std::cout << "Event notified at " << sc_time_stamp() << '\n';
      e_.notify(1, SC_SEC);
      if (sc_time_stamp() == sc_time(4, SC_SEC)) {
        std::cout << "\t\tEvent canceled at " << sc_time_stamp() << '\n';
        e_.cancel();
      }
      wait(2, SC_SEC);
    }
  }

  void thread_catcher() {
    while (true) {
      wait(e_);
      std::cout << "\tEvent caught at " << sc_time_stamp() << '\n';
    }
  }
};

int sc_main(int, char*[]) {

  EventModule m_module("m");

  sc_start(10, SC_SEC);

  return 0;
}


//         SystemC 2.3.3-Accellera --- Mar 23 2026 15:03:48
//         Copyright (c) 1996-2018 by all Contributors,
//         ALL RIGHTS RESERVED
// Event notified at 0 s
//         Event caught at 1 s
// Event notified at 2 s
//         Event caught at 3 s
// Event notified at 4 s
//                 Event canceled at 4 s
// Event notified at 6 s
//         Event caught at 7 s
// Event notified at 8 s
//         Event caught at 9 s
```

- The are several forms of `wait()` are supported, following:

1.` wait()`: wait on events in sensitivity list (SystemC 1.0).
1. `wait(e1)`: wait on event e1.
2. `wait(e1 | e2 | e3)`: wait on events e1, e2, or e3.
3. `wait(e1 & e2 & e3)`: wait on events e1, e2, and e3.
4. `wait(200, SC_NS)`: wait for 200 ns.
5. `wait(200, SC_NS, e1)`: wait on event e1, timeout after 200 ns.
6. `wait(200, SC_NS, e1 | e2 | e3)`: wait on events e1, e2, or e3, timeout after 200 ns.
7. `wait(200, SC_NS, e1 & e2 & e3)`: wait on events e1, e2, and e3, timeout after 200 ns.
8. `wait(sc_time(200, SC_NS))`: wait for 200 ns.
9.  `wait(sc_time(200, SC_NS), e1)`: wait on event e1, timeout after 200 ns.
10. `wait(sc_time(200, SC_NS), e1 | e2 | e3)`: wait on events e1, e2, or e3, timeout after 200 ns.
12. `wait(sc_time(200, SC_NS), e1 & e2 & e3 )`: wait on events e1, e2, and e3, timeout after 200 ns.
13. `wait(200)`: wait for 200 clock cycles, SC_CTHREAD only (SystemC 1.0)
14. `wait(0, SC_NS)`: wait one delta cycle.
15. `wait(SC_ZERO_TIME)`: wait one delta cycle.

### 4.7. Delta Cycle
- A delta cycle is a very small step of time within the simulation. SystemC keeps running delta cycles until no more events are pending.
- Multi delta cycles maybe occur at a particular simulated time.
- When a signal assignment occurs, other process do not see the update until the next delta cycle.
- The delta cycle is used when:
  - `notify(SC_ZERO_TIME)`: a.k.a delta notification, event to be notified `in the evaluate phase of the next delta cycle`
  - `request_update()`, `update()`: to be called `in the update phase of the current delta cycle`

### 4.8. Sensitivity
- The **sensitivity of a process instance** is the set of `events and time-outs` that can potentially cause `the process to be resumed or triggered`.
- A process instance is sensitive to an event if the event has been added to the `static/dynamic sensitivity` of the process instance.
- The time-out occurs when a given interval time has elapsed.
- There are two types of sensitivities:
  - `Static`: is fixed during elaboration. (`sensitive << event << event;` & `wait()`)
  - `Dynamic`: under control of the process itself. (`wait(event | event)` & `next_trigger()`)
- It applies to the most recently declared process
### 4.9. Initialization
- It is a part of the execution stage, which happens after `sc_start()`
- `dont_initialize()` prevents a process from running at time 0 (initialization phase).
- TBD https://learnsystemc.com/basic/initialization

### 4.10. Process Method - SC_METHOD
- Method may have static/dynamic sensitivity (`next_trigger`)
- e.g.
```cpp
#include <systemc>
using namespace sc_core;

SC_MODULE(PROCESS) {
  SC_CTOR(PROCESS) { // constructor
    SC_METHOD(method); // register a method process
  }

  void method() {
    // notice there's no while loop here
    int idx = 0; // re-declare every time method is triggered
    std::cout << "method" << idx++ << " @ " << sc_time_stamp() << std::endl;
    next_trigger(1, SC_SEC);
  }
};

int sc_main(int, char*[]) {
  PROCESS process("process");
  sc_start(4, SC_SEC);
  return 0;
}
```

### 4.11. Event Queue - sc_event_queue
- A queue that can contain any number of pending notifications.
- TBD https://learnsystemc.com/basic/event_queue
https://learnsystemc.com/basic/event_queue_combined

### 4.12. Mutex
- SystemC Mutex is a predefined channel intended to model the behavior of a mutual exclusion lock used to control access to a resource shared by concurrent processes.
- Member functions:
  - `int lock()`:  shall suspend until the mutex is unlocked
  - `int trylock()`: -1 if is already locked.
  - `int unlock()`: return the value –1. The mutex shall remain unlocked.

```cpp
#include <systemc>
using namespace sc_core;

SC_MODULE(MutexModule) {
  sc_mutex mutex_;  // create a mutex

  SC_CTOR(MutexModule) {
    SC_THREAD(thread_1);
    SC_THREAD(thread_2);
  }

  void thread_1() {
    while (true) {
      // try to lock or wait
      if (mutex_.trylock() == -1) {
        std::cout << sc_time_stamp()
                  << ": thread_1 suspend until the mutex is unlocked\n";
        mutex_.lock();
        std::cout << sc_time_stamp()
                  << ": thread_1 obtained resource by lock()\n";
      } else {
        std::cout << sc_time_stamp()
                  << ": thread_1 obtained resource by trylock()\n";
      }

      wait(1, SC_SEC);

      // unlock
      mutex_.unlock();
      std::cout << sc_time_stamp()
                << ": thread_1 released resource by unlock()\n";

      wait(SC_ZERO_TIME);
    }
  }

  void thread_2() {
    while (true) {
      // try to lock or wait
      if (mutex_.trylock() == -1) {
        std::cout << sc_time_stamp()
                  << ": thread_2  suspend until the mutex is unlocked\n";
        mutex_.lock();
        std::cout << sc_time_stamp()
                  << ": thread_2 obtained resource by lock()\n";
      } else {
        std::cout << sc_time_stamp()
                  << ": thread_2 obtained resource by trylock()\n";
      }

      wait(1, SC_SEC);

      // unlock
      mutex_.unlock();
      std::cout << sc_time_stamp()
                << ": thread_2 released resource by unlock()\n";

      wait(SC_ZERO_TIME);
    }
  }
};

int sc_main(int, char*[]) {
  MutexModule module("mutex");
  sc_start(4, SC_SEC);
  return 0;
}
```

### 4.13. Semaphore
https://learnsystemc.com/basic/channel_semaphore

### 4.14. FIFO - sc_fifo
- FIFO is used to model the behavior of a fifo.
- The number of slots is fixed when the object is constructed.
- It implements the `sc_fifo_in_if<T>` or `sc_fifo_out_if<T>` interfaces.
https://learnsystemc.com/basic/channel_fifo

### 4.15. Signal <sc_signal>
- **Signal** is a predefined primitive channel intended to model the behavior of a single piece of wire carrying a digital electronic signal, which can only be written by one process at each delta cycle.
- It implements the `sc_signal_inout_if<T>` interface
- Features:
  - is an object of the class `sc_signal`
  - has only one slot for rw
  - triggers and update request only if the new value is different from the current value
  - read won't remove the value

- Constructors:
  - `sc_signal()`
  - `sc_signal(const char* name)`
- Member functions:
  - `read()`, `operator ()` return a reference to the current value
  - `write(<v>)`, `operator =`: modifies the value of the signal
  - `sc_event& default_event()`, `sc_event& value_changed_event()`: return a reference to the value-changed event.
  - `bool event()`: return true if the value of the signal changed in the update phase of the immediately preceding delta cycle and at the current simulation time.

- **Many writers**: https://learnsystemc.com/basic/signal_many_writer

- **Resolved Signal <sc_signal_resolved, sc_signal_rv>**
  - A resolved signal may be written by multi process.
  - The difference between `sc_signal_resolved` and `sc_signal_rv` is the argument to the base class template.
    - class `sc_signal_resolved`: public sc_signal<sc_dt::sc_logic,SC_MANY_WRITERS>
    - template <int W> class `sc_signal_rv`: public sc_signal<sc_dt::sc_lv<W>,SC_MANY_WRITERS>

- **sc_signal<bool>** provides additional member functions appropriate for two-valued signals.
  - `posedge_event()` : returns **reference to an event** that is notified whenever the value of the channel **changes** and the new value of the channel is **true or '1'**.
  - `negedge_event()` : returns **reference to an event** that is notified whenever the value of the channel **changes** and the new value of the channel is **false or '0'**.
  - `posedge()`: returns **true** if and only if the **value of the channel changed** in the update phase of the immediately preceding delta cycle and at the current simulation time, and the **new value of the channel is true or '1'**.
  - `negedge()`: returns **true** if and only if the **value of the channel changed** in the update phase of the immediately preceding delta cycle and at the current simulation time, and the **new value of the channel is false or '0'**.

### 4.16. Buffer - <sc_buffer>
- `sc_buffer` is a predefined primitive channel derived from `sc_signal`
- The difference from `signal` is that a value-changed event is notified `whenever the buffer is written` >< when the value of the buffer is changed.

### 4.17. Communication
#### 4.17.1. Port
- A **Interface** is an abstract class derived from `sc_interface` but not `sc_object`, which contains a set of pure virtual functions that shall be defined in one more channels derived from that interface.
- A **Channel** is a non-abstract class derived from one or more interfaces. A channel may be a primitive channel or a hierarchical channel. If not, it is strongly recommended that a channel be derived from the class `sc_object`.
    - `sc_prim_channel` is the base class for all primitive channels.
    - channel may provide public member functions that can be called using the interface method call paradigm.
    - a primitive channel shall implement one or more interfaces.
- A **Port** an interface of a module used to communicate with the outside world. It is either a class derived from the class `sc_port` or an object of the class `sc_port`(*basically, it is a pointer to channel*). It requires **services**, **interface defines services**, **channel implements services**.
    - provides the means by which a module can be written such that it is independent of the context in which it is instantiated.
    - forwards interface method calls to the channel to which the port is bound.
    - defines a set of services (as identified by the type of the port) that are required by the module containing the port.

- When to use port:
  1. If a module is to call a member function belonging to a channel that is outside the module itself, that call should be made using an interface method **call through a port of the module**.
  2. However, a call to a member function belonging to a channel instantiated within the current module may be made directly. This is known as **portless** channel access.
  3. If a module is to call a member function belonging to a channel instance within a child module, that call should be made through an **export of the child module.**
   
- e.g.
```cpp
#include <systemc>

using namespace sc_core;

class ModuleA : public sc_module {
 public:
  SC_CTOR(ModuleA) {
    // write thread (runs continuously and drives values to the channel)
    SC_THREAD(outsideWrite);
  }

  // binds (connect) port to channel (signal)
  // this connects the module's output port to an external sc_signal
  void bind(sc_signal<int>& s) { port_(s); }

 private:
  // a port used to write to an outside channel
  // sc_port is an interface handle, not storage
  sc_port<sc_signal_out_if<int>> port_;

  void outsideWrite() {
    int val = 1;
    while (true) {
      // write to an outside channel
      // calls the write() method of the bound channel via the interface
      // port_ behaves like a pointer to the channel interface
      port_->write(val++);
      wait(1, SC_SEC);
    }
  }
};

class ModuleB : public sc_module {
 public:
  SC_CTOR(ModuleB) {
    // read thread (reacts to value changes on the bound channel)
    SC_THREAD(outsideRead);

    // static sensitivity: applies to THIS SC_THREAD only
    // triggers when the value in the connected channel changes
    sensitive << port_;

    // prevent execution at time 0, only run when an event occurs
    dont_initialize();
  }

  // binds (connect) port to channel (signal)
  // connects input port to external sc_signal
  void bind(sc_signal<int>& s) { port_(s); }

 private:
  // a port used to read from an outside channel
  // uses input interface of sc_signal
  sc_port<sc_signal_in_if<int>> port_;

  void outsideRead() {
    while (true) {
      // use port to read from the channel
      // read() fetches the current value stored in the signal
      std::cout << sc_time_stamp()
                << ": reads from outside channel, val=" << port_->read()
                << std::endl;

      // wait for next value-change event (as defined by sensitivity)
      wait();
    }
  }
};

int sc_main(int, char*[]) {

  ModuleA ma("MA");
  ModuleB mb("MB");
  
  // declares a signal (channel) outside modules to connect them
  // this signal stores data and propagates events between modules
  sc_signal<int> signal;

  // bind ports to the same channel -> enables communication
  ma.bind(signal);
  mb.bind(signal);

  sc_start(10, SC_SEC);
  return 0;
}


//         SystemC 2.3.3-Accellera --- Mar 23 2026 15:03:48
//         Copyright (c) 1996-2018 by all Contributors,
//         ALL RIGHTS RESERVED
// 0 s: reads from outside channel, val=1
// 1 s: reads from outside channel, val=2
```

#### 4.17.2. Export
- An **Export** is `sc_export` class.
  - allows a module to provide an interface to its parent module
  - forwards interface method to the channel to which the export is bound
  - defines a set of services that are provided by the module containing the export
- When to use export:
  1. Providing an interface through an export is an alternative to a module simply implementing the interface.
  2. The use of an explicit export allows a single module instance to provide multiple interfaces in a structured manner.
  3. If a module is to call a member function belonging to a channel instance within a child module, that call should be made through an export of the child module.

- e.g.
```cpp
class ModuleA : public sc_module {
 public:
  // an export for other module to connect
  sc_export<sc_signal<int>> port_;

  SC_CTOR(ModuleA) {
    port_(signal_);  // bind an export to an internal channel
    SC_THREAD(outsideWrite);
  }

 private:
  sc_signal<int> signal_;

  void outsideWrite() {
    int val = 1;
    while (true) {
      signal_.write(val++);  // write to an channel
      wait(1, SC_SEC);
    }
  }
};

class ModuleB : public sc_module {
 public:
  // a port used to read from an export of another module
  sc_port<sc_signal_in_if<int>> port_;

  SC_CTOR(ModuleB) {
    SC_THREAD(outsideRead);
    sensitive << port_;
    dont_initialize();
  }

 private:
  void outsideRead() {
    while (true) {
      std::cout << sc_time_stamp()
                << ": reads from outside channel, val=" << port_->read()
                << std::endl;
      wait();
    }
  }
};

int sc_main(int, char*[]) {
  Export::ModuleA ma("MA");
  Export::ModuleB mb("MB");

  // connect ModuleA's port to ModuleB's export (interface forwarding)
  mb.port_(ma.port_);

  sc_start(10, SC_SEC);
  return 0;
}

//         SystemC 2.3.3-Accellera --- Mar 23 2026 15:03:48
//         Copyright (c) 1996-2018 by all Contributors,
//         ALL RIGHTS RESERVED
// 0 s: reads from outside channel, val=1
// 1 s: reads from outside channel, val=2
// 2 s: reads from outside channel, val=3
// 3 s: reads from outside channel, val=4
// 4 s: reads from outside channel, val=5
// 5 s: reads from outside channel, val=6
// 6 s: reads from outside channel, val=7
// 7 s: reads from outside channel, val=8
// 8 s: reads from outside channel, val=9
// 9 s: reads from outside channel, val=10
```

#### 4.17.3. Port to Port
- So far we covered the cases of:
    - **connecting two processes of same module via channel:**
      `process1() --> channel --> process2() `
    - **connecting two processes of different modules via port and channel:**
      `module1::process1() --> module1::port1 --> channel --> module2::port2 --> module2::process2()`
    - **connecting two processes of different modules via export:**
      `module1::process1() --> module1::channel --> module1::export1 --> module2::port2 --> module2::process2()`
  
- Now we have these cases like `module::port1 --> module::submodule::port2`

#### 4.17.4. Specialized Ports
https://learnsystemc.com/basic/specialized_port

#### 4.17.5. Port Array
https://learnsystemc.com/basic/port_array

### 4.18. Primitive Channel
https://learnsystemc.com/basic/primitive_channel

### 4.19. Hierarchical Channel
https://learnsystemc.com/basic/hierarchical_channel

### 4.20. Clock
- **Clock** `sc_clock` is a predefined primitive channel to model the behavior of **a digital clock signal**
- `sc_signal_in_if<bool>` to access the value and events of the clock

>Constructor:
sc_clock(
  constchar*name_, // unique module name
  double period_v_, // the time interval between two consecutive transitions from false to true, also equal to the time interval between two consecutive transitions from true to false. Greater than zero, default is 1 nanosecond.
  sc_time_unit period_tu_, // time unit, used for period
  double duty_cycle_, // the proportion of the period during which the clock has the value true. Between 0.0 and 1.0, exclusive. Default is 0.5.
  double start_time_v_, // the absolute time of the first transition of the value of the clock (false to true or true to false). Default is zero.
  sc_time_unit start_time_tu_,
  bool posedge_first_ = true ); // if true, the clock is initialized to false, and changes to true at the start time. Vice versa. Default is true.
- e.g.
```cpp
#include <systemc>

using namespace sc_core;

class ClockModule : public sc_module {
 public:
  sc_port<sc_signal_in_if<bool>> clk_;

  SC_CTOR(ClockModule) {
    SC_THREAD(thread);
    sensitive << clk_;
    dont_initialize();
  }

  void thread() {
    while (true) {
      // print current clock value
      std::cout << sc_time_stamp() << ", value = " << clk_->read() << std::endl;
      // wait for next clock value change
      wait();
    }
  }
};

int sc_main(int, char*[]){
  sc_clock clk("clk", 10, SC_SEC, 0.2, 10, SC_SEC, false);  // 10: 10s period
      // 0.2: 2s true, 8s false
      // 10: start at 10s
      // false: start at false

  ClockModule m_clock("mclock");
  // bind port
  m_clock.clk_(clk);

  sc_start(31, SC_SEC);
  return 0;
}
```
### 4.21. Customized Data Type
https://learnsystemc.com/basic/customized_datatype


## 5. Trace File & Error/Message Report
- A **trace file** records a time-ordered sequence of value changes during simulation.
  -  uses VCD (Value change dump) file format.
  -  can only be created and opened by `sc_create_vcd_trace_file`.
  -  may be opened during elaboration or at any time during simulation.
  -  contains values that can only be traced by `sc_trace`.
  -  shall be opened before values can be traced to that file, and values shall not be traced to a given trace file if one or more delta cycles have elapsed since opening the file.
  -  shall be closed by `sc_close_vcd_trace_file`. A trace file shall not be closed before the final delta cycle of simulation.
- e.g.
```cpp
// Learn with Examples, 2020, MIT license
#include <systemc>
using namespace sc_core;

SC_MODULE(MODULE) { // a module write to a channel
  sc_port<sc_signal<int>> p; // a port
  SC_CTOR(MODULE) {
    SC_THREAD(writer); // a writer process
  }
  void writer() {
    int v = 1;
    while (true) {
      p->write(v++); // write to channel via port
      wait(1, SC_SEC); // write every 1 s
    }
  }
};
int sc_main(int, char*[]) {
  MODULE module("module"); // instantiate module
  sc_signal<int> s; // declares signal channel
  module.p(s); // bind port to channel

  sc_trace_file* file = sc_create_vcd_trace_file("trace"); // open trace file
  sc_trace(file, s, "signal"); // trace "s" under the name of "signal"
  sc_start(5, SC_SEC); // run simulation for 5 s
  sc_close_vcd_trace_file(file); // close trace file
  return 0;
}
```

https://learnsystemc.com/basic/report