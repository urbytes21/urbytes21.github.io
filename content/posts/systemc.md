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