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
- `SystemC` `is capable of` modeling hardware and software together at multiple level of abstraction (Algorithm / Functional level, Transaction-Level Modeling, Register Transfer Level)
> Modeling is the process of creating a simplified version of a real system.
Simulation is the process of executing the model over time to see how it behaves.
- [Reference](https://learnsystemc.com/)

```bash
systemc/
├── src/
│   ├── sysc/
│   │   ├── kernel/         # Simulation kernel (sc_module, sc_event, processes, sc_time…)
│   │   ├── communication/  # Ports, signals, FIFOs, mutexes, exports
│   │   ├── datatypes/      # Bit/integer/fixed-point data types
│   │   ├── tracing/        # VCD and WIF trace file writers
│   │   └── utils/          # Reporting, sc_vector, sc_string_view
│   ├── tlm_core/
│   │   ├── tlm_1/          # TLM-1 legacy interfaces and channels
│   │   └── tlm_2/          # TLM-2.0 (generic payload, sockets, phases, DMI)
│   ├── tlm_utils/          # TLM utilities (simple_target_socket, simple_initiator_socket, peq…)
│   ├── systemc.h           # Top-level SystemC include
│   └── tlm.h               # Top-level TLM include
├── examples/
│   ├── sysc/               # SystemC examples (pipe, RISC CPU, FFT, simple_bus…)
│   └── tlm/                # TLM-2.0 examples (LT, AT 1/2/4-phase, DMI, mixed endian…)
├── tests/
│   ├── systemc/            # Regression tests for SystemC core
│   └── tlm/                # Regression tests for TLM
├── docs/
│   ├── sysc/               # SystemC documentation
│   └── tlm/                # TLM documentation
├── cmake/                  # CMake helper scripts and config templates
├── docker/                 # Docker build environments (Ubuntu, AlmaLinux)
├── CMakeLists.txt
├── INSTALL.md
└── README.md
```

**SystemC** runs as a **discrete-event** simulator. The high-level flow is:

1. **Elaboration** – Modules are instantiated, ports are bound to channels.
2. **Initialization** – Initial process activations and signal updates.
3. **Simulation** – The kernel repeatedly executes `delta cycles` (zero-time updates) and advances the simulation clock.
4. **Termination** – When `sc_stop()` is called or the event queue is empty.


**TLM-2.0** replaces pin-level signal connections with high-level function calls carrying a `tlm_generic_payload` transaction object. An **initiator** (e.g., a CPU model) calls transport functions on a **target** (e.g., a memory) through typed **sockets** and a **bus/router**. Two coding styles exist:

- **Loosely-Timed (LT)** – uses blocking `b_transport()`. Simpler and faster; time advances by annotating a `sc_time` delay.
- **Approximately-Timed (AT)** – uses non-blocking `nb_transport_fw()` / `nb_transport_bw()`. Models pipeline stages and bus protocols with explicit phases.


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

```cpp
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
```

## 4. Operation Stages
- SystemC application has three phases/stages of operation:
  - `Elaboration`: to create internal data structures with in the kernel, execution of statements prior to `sc_start()`
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


## 5. Core Language
### 5.1. Class headers
- `<systemc>` vs `systemc.h`

### 5.2. Module `<sc_module>`
-  **Modules** are the principle structural building blocks of SystemC.
   - It encapsulates `structure (sub-modules)`, `communication (ports/signals)`, and `behavior (processes)`.
   - It inherits the `sc_module` class.
   - It used to represent a component in read systems.
- e.g.
```cpp
SC_MODULE(Counter) {
    sc_in<bool>       clk;
    sc_in<bool>       reset;
    sc_out<sc_uint<8>> count;

    sc_uint<8> cnt;

    void do_count() {
        if (reset.read()) cnt = 0;
        else              cnt++;
        count.write(cnt);
    }

    SC_CTOR(Counter) {
        SC_METHOD(do_count);
        sensitive << clk.pos();
    }
};
```

- Use `SC_CTOR` or `SC_HAS_PROCESS` for create the constructor
- `sc_module_name` class for name param, access via `name()`

### 5.3. Process
- A **process** is  a member function of a **module**, that registered with the simulation kernel.
- Three types of process: `SC_METHOD`, `SC_THREAD`, `SC_CTHREAD`
  - Method process:  cannot be suspended, should not contain infinite 
loops,  execute in zero time.
  - `SC_THREAD(func)`: has its own thread of execution, normally have infinite loops, can be suspended/ `wait()`, 
  - `SC_CTHREAD(func)`: that similar to the `SC_THREAD` but only have a static sensitivity of a clock edge event ?'

    | Type | Macro | Description |
    |------|-------|-------------|
    | `SC_METHOD` | `SC_METHOD(func)` | Re-executes on every trigger; **no** `wait()` allowed. |
    | `SC_THREAD` | `SC_THREAD(func)` | Runs once; can call `wait()` to suspend and resume. Keeps state. |
    | `SC_CTHREAD` | `SC_CTHREAD(func, clk.pos())` | Like `SC_THREAD` but clocked; uses `wait()` to advance one clock. |

- We can register the simulation with the simulation kernel in:
  - `constructor`
  - `before_end_of_elaboration` or `end_of_elaboration` callbacks of a module

#### 5.3.1. Method Process `<SC_METHOD>`
- `Initially triggered` when the kernel calls the function associated with the process
- Does not allow `wait()`
- `static sensitivity` to defines when the process is triggered again (`notifications` take effect in the next delta cycle)
- `dynamic sensitivity` by calling `next_trigger()` to be triggered again.
-  run in the same execution context as the simulation kernel

#### 5.3.2. Thread Processes `<SC_THREAD>`
- function is invoked once by the kernel and typically contains an infinite loop to prevent it from **terminating**
- `wait()` to suspend execution
- resumes execution from the point immediately after `wait()`
- may have `static sensitivity`
- `dynamic sensitivity` is created by calling `wait()`
- requires its own execution stack (thread) (higher overhead than SC_METHOD)

#### 5.3.3. Clock Thread Processes `<SC_CTHREAD>`
- similar to `SC_THREAD`, but specialized for clocked (cycle-based) modeling
- must be a static process (cannot be spawned)
- must be statically sensitive to exactly `one clock edge`
- executes once per clock cycle after each, and allows only `wait()` or `wait(int)` to advance clock
- supports synchronous and asynchronous reset signals


### 5.4. Sensitivity
- The **sensitivity of a process instance** is the set of `events and time-outs` that can potentially cause `the process to be resumed or triggered`.
- A process instance is sensitive to an event if the event has been added to the `static/dynamic sensitivity` of the process instance.
- The time-out occurs when a given interval time has elapsed.
- There are two types of sensitivities:
  - `Static`: class `sc_module` should have a data member typed `sc_sensitive` named **sensitive**, used to create static sensitivity. (`sensitive << event << event;` & `wait()`)
  - `Dynamic`: under control of the process itself. (`wait(event | event)` & `next_trigger()`)
> static sensitive is created at the time the process instance is created, so i means applies to the most recently declared process

### 5.5. Event `<sc_event>`
- An **event** is an object, represented by class `sc_event`,that determines whether and when a process execution should be triggered or resumed.
- **Event Occurrence**: Event object keeps a list of processes that are sensitive to it. The owner of the event report the change to the event -> event object inform the scheduler of which processes to trigger. 
- **Event Notification**: Events can be notifies in three ways - `immediate`, `delta-cycle delayed`, `and timed` (*An earlier notification will always override one scheduled to occur later*)
- **Canceling Event Notification**: A pending delayed event notification may be canceled using `cancel()`.

```cpp
sc_event my_event;

// Notify types:
my_event.notify();                          // Immediate notification (current delta)
my_event.notify(SC_ZERO_TIME);             // Delta-cycle notification
my_event.notify(10, SC_NS);               // Timed notification after 10 ns

// Cancel a pending notification:
my_event.cancel();

// Wait for an event inside SC_THREAD:
wait(my_event);

// Wait for multiple events:
wait(event_a | event_b);
wait(event_a & event_b);
```

----

### 5.6. Time 
- SystemC uses an integer-valued absolute time model.  Time is represented by an `unsigned integer of at least 64-bits`.
- There is two type of `time measurements`:
  - `wall-clock time`: is the time `from the start` of execution `to completion`.
  - `simulated time`: is the time being modeled by the simulation, which maybe less/greater than the simulation's wall-clock time.

#### 5.6.1. `<sc_time>`
- **sc_time** is used to represent **simulation time** and **time intervals**, including delays and time-outs.
- `SC_SEC, SC_MS, _US, _NS, _PS, _FC`
- `sc_time(1, SC_FS)`: to create a `sc_time` `(sc_time( double v, sc_time_unit tu ))`
- `sc_time_stamp()` to get the current simulated time.
- `SC_ZERO_TIME`: a time value of zero, to create a delta notification or delta time-out 

#### 5.6.2. Time Resolution
- Default is **1ps**
- `sc_set_time_resolution()` to set time resolution once

#### 5.6.3. Default Time Unit 
- Default is **1ns**
- `sc_set_default_time_unit()` to set time unit once

----

### 5.7. Data Types

- 4-valued Logic type
- 4-valued Logic vectors
- Bits and Bit Vectors
- Arbitrary Precision Intergers
- Fixed-point types
- C++ types

### 4.1. Valued Logic Type
- SystemC introduces the `sc_logic` type to represent digital signals with four possible values:
  - '0' → Logic 0
  - '1' → Logic 1
  - 'Z' → High impedance (tri-state)
  - 'X' → Unknown / undefined

```cpp
// for modeling real hardware behavior where signals are not always strictly 0 or 1.
sc_logic a = SC_LOGIC_0;
sc_logic b = SC_LOGIC_1;
sc_logic c = SC_LOGIC_Z;
sc_logic d = SC_LOGIC_X;
```

### 4.2. 4-Valued Logic Vectors
- To represent multiple logic bits, SystemC provides `sc_lv<N>` (logic vector). Each bit can be '0', '1', 'Z', or 'X'
```cpp
sc_lv<4> data = "10XZ"; // Used for buses and signals where unknown or high-impedance states must be modeled.
```

### 4.3. Bits and Bit Vectors
- For 2-valued logic (only 0 and 1), SystemC provides:
  - `sc_bit` → single bit
  - `sc_bv<N>` → bit vector
```cpp
// These are faster than 4-valued types because they only support binary values.
sc_bit b = 1;
sc_bv<8> byte = "10101010";
```

### 4.4. Arbitrary Precision Integers
- SystemC supports integers with customizable bit widths:
  - `sc_int<N>` → signed integer
  - `sc_uint<N>` → unsigned integer
```cpp
// Useful for modeling registers and datapaths with exact bit sizes.
sc_int<8> a = -10;
sc_uint<8> b = 255;
```

### 4.5. Fixed-Point Types
- For precise fractional arithmetic, SystemC provides fixed-point types:
  - `sc_fixed<W, I>` → signed fixed-point
  - `sc_ufixed<W, I>` → unsigned fixed-point
  - `W` = total number of bits, `I` = number of integer bits
  
```cpp
// used in DSP and embedded systems.
sc_fixed<8,4> a = 3.25;
sc_ufixed<8,4> b = 2.5;
```

### 4.6. C++ Types
- They are not bit-accurate

## 5. Core Language & Primitive Channels
- Core language:
  - Modules
  - Ports
  - Processes
  - Interface
  - Channels
  - Events
  - ...
- Primitive Channels
  - Signal
  - Mutex
  - Semaphore
  - FIFO
  - ...



  
### 5.5. Concurrency
- SystemC is not true concurrent execution, the processes are simulated as running concurrently, only one is executed at a particular time.
- The processes are running on the same `simulated time` because the simulated time `remain unchanged` until they finished.

- **Notification types:**

    | Type | When processes are triggered |
    |------|------------------------------|
    | Immediate (`notify()`) | In the **same** delta cycle (current evaluation phase) |
    | Delta (`notify(SC_ZERO_TIME)`) | At the **next** delta cycle (next update phase) |
    | Timed (`notify(t, SC_NS)`) | After simulated time `t` has elapsed |

- **`sc_event` and Synchronization**
```cpp
SC_MODULE(Test) {
  int data; // Shared variable
  sc_event e;
  SC_CTOR(Test) {
    SC_THREAD(producer);
    SC_THREAD(consumer);
  }
  void producer() {
    wait(1, SC_NS);
    for (data = 0; data < 10; data++) {
      e.notify();    // Schedule event immediately
      wait(1, SC_NS);
    }
  }
  void consumer() {
    for (;;) {
      wait(e);    // Resume when event occurs
      cout << "Received " << data << endl;
    }
  }
};
```

### 5.7. Delta Cycle
- A delta cycle is a very small step of time within the simulation. SystemC keeps running delta cycles until no more events are pending.
- Multi delta cycles maybe occur at a particular simulated time.
- When a signal assignment occurs, other process do not see the update until the next delta cycle.
- The delta cycle is used when:
  - `notify(SC_ZERO_TIME)`: a.k.a delta notification, event to be notified `in the evaluate phase of the next delta cycle`
  - `request_update()`, `update()`: to be called `in the update phase of the current delta cycle`

### 5.9. Initialization
- It is a part of the execution stage, which happens after `sc_start()`
- `dont_initialize()` prevents a process from running at time 0 (initialization phase).
- TBD https://learnsystemc.com/basic/initialization

### 5.10. Process Method - SC_METHOD
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

### 5.11. Event Queue - sc_event_queue
- A queue that can contain any number of pending notifications.
- TBD https://learnsystemc.com/basic/event_queue
https://learnsystemc.com/basic/event_queue_combined

### 5.12. Mutex
- SystemC Mutex is a predefined channel intended to model the behavior of a mutual exclusion lock used to control access to a resource shared by concurrent processes.
- Member functions:
  - `int lock()`:  shall suspend until the mutex is unlocked
  - `int trylock()`: -1 if is already locked.
  - `int unlock()`: return the value –1. The mutex shall remain unlocked.

### 5.13. Semaphore
https://learnsystemc.com/basic/channel_semaphore

### 5.14. FIFO - sc_fifo
- FIFO is used to model the behavior of a fifo.
- The number of slots is fixed when the object is constructed.
- It implements the `sc_fifo_in_if<T>` or `sc_fifo_out_if<T>` interfaces.
https://learnsystemc.com/basic/channel_fifo

### 5.15. Signal <sc_signal>
- A **signal** is a primitive channel that holds a value and notifies connected processes when the value changes.

    ```cpp
    sc_signal<double> in1;       // signal declaration
    sc_signal<bool>   clk;
    
    // Writing
    clk.write(1);
    
    // Reading inside a process
    double val = in1.read();
    ```
    
- It used to model the behavior of a single piece of wire carrying a digital electronic signal, which can only be written by one process at each delta cycle.
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

- `sc_in<T>`,`sc_out<T>`: is a specialized port class for use with signals.

### 5.16. Buffer - <sc_buffer>
- `sc_buffer` is a predefined primitive channel derived from `sc_signal`
- The difference from `signal` is that a value-changed event is notified `whenever the buffer is written` >< when the value of the buffer is changed.

### 5.17. Communication
![image](/images/sysc_communicate.png)

#### 5.17.1. Port
- A **Interface** (Interface Method Call) is an abstract class derived from `sc_interface`, which contains a set of pure virtual functions that shall be defined in one more channels derived from that interface.
  - A channel implements an interface
  - An interface is an abstract base class of the channel
  - A module calls interface methods via a port


  - e.g.
  - ![image](/images/sysc_interface.png)
    ```cpp
    // Declaration of Interfaces
    #pragma once
    #include <systemc>
    using namespace sc_core;
    
    class write_if : public sc_interface {
     public:
      virtual void write(char) = 0;
      virtual void reset() = 0;
    };
    
    class read_if : public sc_interface {
     public:
      virtual void read(char&) = 0;
      virtual int num_available() const = 0;
    };
    ```

----

- A **Channel** is a non-abstract class derived from one or more interfaces. A channel may be `a primitive channel` or `a hierarchical channel`. If not, it is strongly recommended that a channel be derived from the class `sc_object`.
    - `sc_prim_channel` is the base class for all primitive channels.
    - channel may provide public member functions that can be called using the interface method call paradigm.
    - a primitive channel shall implement one or more interfaces.
  
    - **separate communication from functionality**:
      - **Channels are containers for communication protocols and synchronization events**
      - **Channels implement one/more Interface(s) or Ports**
  - e.g.
    ```cpp
    // declaration of FIFO channels
    #pragma once
    #include <systemc>
    #include "Interface.h"
    
    using namespace sc_core;
    
    class fifo : public sc_channel, public write_if, public read_if {
     public:
      fifo(sc_module_name name);
      void write(char c) override;
      void read(char& c) override;
      void reset() override;
      int num_available() const override;
    
     private:
      enum e { max_elements = 10 };
      char data[max_elements];
      int num_elements;
      int first;  // index of the oldest element (read position)
      sc_event write_event;
      sc_event read_event;
    
      bool fifo_empty();
      bool fifo_full();
    };
    ```

----

- **Ports** provide the interface between a **module** and **its external channels**. They are templated on a channel interface.


| Class | Direction |
|-------|-----------|
| `sc_in<T>` | Input port |
| `sc_out<T>` | Output port |
| `sc_inout<T>` | Bidirectional port |
| `sc_port<IF>` | Generic port bound to any interface `IF` |

It is either a class derived from the class `sc_port` or an object of the class `sc_port`(*basically, it is a pointer to channel*). It requires **services**, **interface defines services**, **channel implements services**.
    - provides the means by which a module can be written such that it is independent of the context in which it is instantiated.
    - forwards interface method calls to the channel to which the port is bound.
    - defines a set of services (as identified by the type of the port) that are required by the module containing the port.
    -  **allow a Modules to connect to Channels through an Interface**
    -   **are bound to the interfaces of channels**

- When to use port:
  1. If a module is to call a member function belonging to a channel that is outside the module itself, that call should be made using an interface method **call through a port of the module**.
  2. However, a call to a member function belonging to a channel instantiated within the current module may be made directly. This is known as **portless** channel access.
  3. If a module is to call a member function belonging to a channel instance within a child module, that call should be made through an **export of the child module.**
   
#### 5.17.2. Export
- An **Export** is `sc_export` class.
  - allows a module to provide an interface to its parent module
  - forwards interface method to the channel to which the export is bound
  - defines a set of services that are provided by the module containing the export
- When to use export:
  1. Providing an interface through an export is an alternative to a module simply implementing the interface.
  2. The use of an explicit export allows a single module instance to provide multiple interfaces in a structured manner.
  3. If a module is to call a member function belonging to a channel instance within a child module, that call should be made through an export of the child module.

#### 5.17.3. Port to Port
- So far we covered the cases of:
    - **connecting two processes of same module via channel:**
      `process1() --> channel --> process2() `
    - **connecting two processes of different modules via port and channel:**
      `module1::process1() --> module1::port1 --> channel --> module2::port2 --> module2::process2()`
    - **connecting two processes of different modules via export:**
      `module1::process1() --> module1::channel --> module1::export1 --> module2::port2 --> module2::process2()`
  
- Now we have these cases like `module::port1 --> module::submodule::port2`

#### 5.17.4. Specialized Ports
https://learnsystemc.com/basic/specialized_port

#### 5.17.5. Port Array
https://learnsystemc.com/basic/port_array

### 5.18. Primitive Channel
https://learnsystemc.com/basic/primitive_channel

### 5.19. Hierarchical Channel
https://learnsystemc.com/basic/hierarchical_channel

### 5.20. Clock
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
### 5.21. Customized Data Type
https://learnsystemc.com/basic/customized_datatype


## 6. Trace File & Error/Message Report
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

## 7. TLM 2.0 - Transaction Level Modeling
![image](/images/sysc_tml_overview.png)

### 7.1 Introduce

TLM is a library built on top of the SystemC library. It consists of a set of `core interfaces, objects, base protocols, and utilities` that enable the `TLM concept`.

- There are two specific **coding styles**:
  - **Loosely-timed (LT):** Focuses on functionality, not precise timing. Uses one delay for the whole transaction. *(blocking transport interface)*
  - **Approximately-timed (AT):** Models timing behavior more accurately than LT and is more realistic, but slower. *(non-blocking transport interface)*

![image](/images/sysc_tml_coding_styles_uc.png)

----
### 7.2 Initiator / Target
![image](/images/sysc_tml_transaction.png)

- An **Initiator** is a module that can create new transaction objects and pass them on by calling a method using the `core interface`. 
- A **Target** is a module that acts as the `final destination` for a transaction.

- A **transaction object** is a data structure passed between initiators and targets using function calls.
- An **interconnect** component is a module that has both a target socket and an initiator socket.
----
### 7.3 Path
- Path is ?
- **Forward path:** A transaction object is created by an initiator and passed to other modules.
- **Return path:** The transaction object is returned automatically.
- **Backward path:** Other modules send the transaction object back by calling a certain method.
----
### 7.4 Socket
![image](/images/sysc_tml_sockets.png)
**Sockets** are used to pass transactions between `initiators` and `targets` in TLM. It combines `a port` with `an export`.
- **Initiator socket**: has a `port` for the `forward path` and an `export` for the `backward path`.
- **Target socket**: has a `export` for the `forward path` and an `port` for the `backward path`.
- In TLM, sockets are connected using `bind()` or `operator()`.
    - An initiator socket connects to a target socket.
    - An interconnect may connect upstream and downstream sockets.
    - Binding establishes the communication path between components.

---
### 7.5 Interface (TLM-2.0 Core Interfaces )
An **interface** defines how transactions are communicated between components (initiator and target). There are three types of interface:
- **Transport interface:** used to transport transactions between initiators, targets and interconnect components.
- **Direct memory interface (DMI):** Provides `read/write` access to an area of `memory owned by a target` using a `direct pointer`.
- **Debug transport interface:** Provides debug `read/write` access to an area of `memory owned by a target`.

----
### 7.6 Transport Interface
- **Blocking transport interface**
  - Each transaction has 2 timing points: **START** and **END**.
  - Uses **forward path and return path only**.
  - The initiator completes a transaction with a target using a single call.

- **Non-blocking transport interface**
  - Each transaction has multiple timing points.
  - Uses **all paths**.
  - A transaction is finished through multiple calls or a single call.

----
### 7.7 TLM 2.0 Library Overview

The standard namespace is **`tlm`**.

- Initiator/target sockets: `tlm_initiator_socket`, `tlm_target_socket`
- Transaction object: `tlm_generic_payload`
- Blocking transport: `tlm_blocking_transport_if`
- Non-blocking transport:
  - `tlm_fw_nonblocking_transport_if`
  - `tlm_bw_nonblocking_transport_if`
- `tlm_phase`: Shows phase information
- `tlm_sync_enum`: Synchronization result between initiator and target
- Forward/backward transport: `tlm_fw_transport_if`, `tlm_bw_transport_if`
- Socket usage: `tlm_initiator_socket<>`, `tlm_target_socket<>`, `bind()`, `operator()`

- **Generic payload:** `<tlm_generic_payload>` is an object that includes `all information of transaction` transferred among modules.

### 7.8 Examples
### 7.8.1 Connection
- **Module and socket:**
```cpp
#include <systemc>
#include <tlm>

using namespace sc_core;

// Initiator Module with socket
class InitiatorModule
    : public sc_module,
      public tlm::tlm_bw_transport_if<> {  // 1. inherit the backward interface
 public:
  tlm::tlm_initiator_socket<> ini_socket;  // 2. declare a socket

  SC_CTOR(InitiatorModule) {
    ini_socket(*this); // 3. connect module to socket
  }  
};

// Target Module with socket
class TargetModule : public sc_module, public tlm::tlm_fw_transport_if<> {
  tlm::tlm_target_socket<> target_socket;

  SC_CTOR(TargetModule) { target_socket(*this); }
};

```
- **Socket and another socket**
```cpp
#include <systemc>
#include <tlm>

using namespace sc_core;
using namespace tlm;

class TargetModule : public sc_module,
                     virtual public tlm::tlm_fw_transport_if<> {
 public:
  tlm::tlm_target_socket<> target_socket;

  SC_CTOR(TargetModule) { target_socket(*this); }

  // Methods for forward path <tlm_fw_transport_if>
  tlm_sync_enum nb_transport_fw(tlm_generic_payload& trans, tlm_phase& phase,
                                sc_time& delay) override {
    // @TODO:
    return TLM_COMPLETED;
  }

  bool get_direct_mem_ptr(tlm_generic_payload&, tlm_dmi& dmi_data) override {
    // @TODO:
    return true;
  }

  unsigned int transport_dbg(tlm_generic_payload& trans) override {
    // @TODO:
    return 0;
  }

  void b_transport(tlm::tlm_generic_payload& trans,
                   sc_core::sc_time& t) override {
    // @TODO:
  }
};

class InitiatorModule
    : public sc_module,
      public tlm::tlm_bw_transport_if<> {  

 public:
  tlm::tlm_initiator_socket<> initiator_socket;

  // target module connect to
  TargetModule* target_module;

  SC_CTOR(InitiatorModule) {

    initiator_socket.bind(*this);

    target_module = new TargetModule("target_m");
    // initiator socket connect to garget socket
    initiator_socket.bind(target_module->target_socket);
  }
};

```

### 7.9 Communication Through Generic Payload
- **Preparation**: Initiator and target communicate together by calling functions. All function must be defined in Initiator module and target module before communication
```cpp
#include <systemc>
#include <tlm>

using namespace sc_core;
using namespace tlm;

class TargetModule : public sc_module,
                     virtual public tlm::tlm_fw_transport_if<> {
 public:
  tlm::tlm_target_socket<> target_socket;

  SC_CTOR(TargetModule) { target_socket(*this); }

  // Methods for forward path <tlm_fw_transport_if>

  // used in non-blocking transaction
  tlm_sync_enum nb_transport_fw(tlm_generic_payload& trans, tlm_phase& phase,
                                sc_time& delay) override {
    // @TODO:
    return TLM_COMPLETED;
  }

  // used in Direct memory interface
  bool get_direct_mem_ptr(tlm_generic_payload&, tlm_dmi& dmi_data) override {
    // @TODO:
    return true;
  }

  // used in Debug memory interface
  unsigned int transport_dbg(tlm_generic_payload& trans) override {
    // @TODO:
    return 0;
  }

  // used in blocking transaction
  void b_transport(tlm::tlm_generic_payload& trans,
                   sc_core::sc_time& t) override {
    // @TODO:
  }
};

// Combined interface required by socket: inherit the backward interface
class InitiatorModule : public sc_module, public tlm::tlm_bw_transport_if<> {

 public:
  // initiator socket, protocol type defaults to base protocol
  tlm::tlm_initiator_socket<> initiator_socket;

  // target module connect to
  TargetModule* target_module;

  SC_CTOR(InitiatorModule) {

    // initiator socket bound to module itself
    // initiator_socket(*this);
    initiator_socket.bind(*this);

    target_module = new TargetModule("target_m");
    // initiator socket connect to garget socket
    initiator_socket.bind(target_module->target_socket);
  }

  // main function for non-blocking transaction
  tlm_sync_enum nb_transport_bw(tlm::tlm_generic_payload& trans,
                                tlm::tlm_phase& phase,
                                sc_core::sc_time& t) override {
    // @TODO
  }

  // used in direct memory interface
  void invalidate_direct_mem_ptr(sc_dt::uint64 start_range,
                                 sc_dt::uint64 end_range) override {
    // @TODO
  }
};
```
> Blocking interface: Initiator module > Target module
    (Initiator socket  >  Target socket)
    Step 1: Create transaction object (tlm::tlm_generic_payload object)
    Step 2: Set values or information for transaction object by using methods 
    of tlm::tlm_generic_payload
    Step 3: From Initiator socket call b_transport function

# SystemC Reference Implementation

## Description

This repository contains the **official reference implementation** of the SystemC class library, as standardized by the [IEEE Std. 1666-2023][ieee] and developed under the [Accellera Systems Initiative][accellera]. SystemC is a C++ class library that enables hardware/software co-design, system-level modeling, architectural exploration, functional verification, and high-level synthesis. It spans the gap between hardware description languages (HDLs) and traditional software programming.

This implementation includes:

- **SystemC core library** – modules, processes, ports, signals, events, clocks, FIFOs, mutexes, and more.
- **TLM-2.0 (Transaction-Level Modeling)** – a higher-level modeling abstraction for on-chip communication (bus/interconnect models).
- **Datatypes** – fixed-point (`sc_fixed`), arbitrary-precision integers (`sc_int`, `sc_bigint`), and logic vectors (`sc_lv`, `sc_bv`).
- **Tracing** – VCD and WIF waveform output.

---

## Features

- Full C++ implementation of the IEEE 1666-2023 SystemC standard.
- TLM-2.0 support: Loosely-Timed (LT) and Approximately-Timed (AT) modeling styles.
- Direct Memory Interface (DMI) for TLM-2.0 fast-path access.
- sc_module hierarchy with ports, signals, and events.
- Multiple process types: `SC_METHOD`, `SC_THREAD`, `SC_CTHREAD`.
- Waveform tracing to VCD and WIF formats.
- Multi-platform support: Linux, macOS, Windows (MinGW / MSYS / Visual Studio).
- CMake and GNU Autotools build systems.
- Extensive built-in examples (SystemC and TLM-2.0) and regression tests.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | C++17 |
| Standard | IEEE Std. 1666-2023 |
| Build System | CMake ≥ 3.5, GNU Autotools (legacy) |
| Compilers | GCC ≥ 9.3, Clang ≥ 13.0, MSVC 2019+ |
| Platforms | Linux (x86_64, aarch64), macOS, Windows |
| Context switching | QuickThreads, POSIX Threads, Windows Fibers, C++ std::thread |
| Tracing | VCD (Value Change Dump), WIF (Waveform Intermediate Format) |

---

## Project Structure

```
systemc/
├── src/
│   ├── sysc/
│   │   ├── kernel/         # Simulation kernel (sc_module, sc_event, processes, sc_time…)
│   │   ├── communication/  # Ports, signals, FIFOs, mutexes, exports
│   │   ├── datatypes/      # Bit/integer/fixed-point data types
│   │   ├── tracing/        # VCD and WIF trace file writers
│   │   └── utils/          # Reporting, sc_vector, sc_string_view
│   ├── tlm_core/
│   │   ├── tlm_1/          # TLM-1 legacy interfaces and channels
│   │   └── tlm_2/          # TLM-2.0 (generic payload, sockets, phases, DMI)
│   ├── tlm_utils/          # TLM utilities (simple_target_socket, simple_initiator_socket, peq…)
│   ├── systemc.h           # Top-level SystemC include
│   └── tlm.h               # Top-level TLM include
├── examples/
│   ├── sysc/               # SystemC examples (pipe, RISC CPU, FFT, simple_bus…)
│   └── tlm/                # TLM-2.0 examples (LT, AT 1/2/4-phase, DMI, mixed endian…)
├── tests/
│   ├── systemc/            # Regression tests for SystemC core
│   └── tlm/                # Regression tests for TLM
├── docs/
│   ├── sysc/               # SystemC documentation
│   └── tlm/                # TLM documentation
├── cmake/                  # CMake helper scripts and config templates
├── docker/                 # Docker build environments (Ubuntu, AlmaLinux)
├── CMakeLists.txt
├── INSTALL.md
└── README.md
```

---

## How It Works

### SystemC Simulation Engine

SystemC runs as a discrete-event simulator. The high-level flow is:

1. **Elaboration** – Modules are instantiated, ports are bound to channels.
2. **Initialization** – Initial process activations and signal updates.
3. **Simulation** – The kernel repeatedly executes `delta cycles` (zero-time updates) and advances the simulation clock.
4. **Termination** – When `sc_stop()` is called or the event queue is empty.

### TLM-2.0 Communication

TLM-2.0 replaces pin-level signal connections with high-level function calls carrying a `tlm_generic_payload` transaction object. An **initiator** (e.g., a CPU model) calls transport functions on a **target** (e.g., a memory) through typed **sockets** and a **bus/router**. Two coding styles exist:

- **Loosely-Timed (LT)** – uses blocking `b_transport()`. Simpler and faster; time advances by annotating a `sc_time` delay.
- **Approximately-Timed (AT)** – uses non-blocking `nb_transport_fw()` / `nb_transport_bw()`. Models pipeline stages and bus protocols with explicit phases.

---

## SystemC Concepts

### Module (`sc_module`)

A module is the fundamental building block of a SystemC design. It encapsulates structure (sub-modules), communication (ports/signals), and behavior (processes).

```cpp
SC_MODULE(Counter) {
    sc_in<bool>       clk;
    sc_in<bool>       reset;
    sc_out<sc_uint<8>> count;

    sc_uint<8> cnt;

    void do_count() {
        if (reset.read()) cnt = 0;
        else              cnt++;
        count.write(cnt);
    }

    SC_CTOR(Counter) {
        SC_METHOD(do_count);
        sensitive << clk.pos();
    }
};
```

### Processes

A process is a member function of a module registered with the simulator. Three types exist:

| Type | Macro | Description |
|------|-------|-------------|
| `SC_METHOD` | `SC_METHOD(func)` | Re-executes on every trigger; **no** `wait()` allowed. |
| `SC_THREAD` | `SC_THREAD(func)` | Runs once; can call `wait()` to suspend and resume. Keeps state. |
| `SC_CTHREAD` | `SC_CTHREAD(func, clk.pos())` | Like `SC_THREAD` but clocked; uses `wait()` to advance one clock. |

**Difference**: `SC_METHOD` is stateless (re-enters from top each trigger), while `SC_THREAD`/`SC_CTHREAD` are stateful coroutines that suspend mid-execution.

### Ports

Ports provide the interface between a module and its external channels. They are templated on a channel interface.

| Class | Direction |
|-------|-----------|
| `sc_in<T>` | Input port |
| `sc_out<T>` | Output port |
| `sc_inout<T>` | Bidirectional port |
| `sc_port<IF>` | Generic port bound to any interface `IF` |

**Binding ports:**

```cpp
// Named binding (recommended)
stage1.in1(signal_in1);
stage1.clk(clk_signal);

// Positional binding
numgen N("numgen");
N(out1, out2, clk);       // ports bound left-to-right as declared
```

### Signal (`sc_signal<T>`)

A signal is a primitive channel that holds a value and notifies connected processes when the value changes. Reads happen via `.read()` and writes via `.write()`.

```cpp
sc_signal<double> in1;       // signal declaration
sc_signal<bool>   clk;

// Writing
clk.write(1);

// Reading inside a process
double val = in1.read();
```

### Event (`sc_event`)

An event is a primitive synchronization object. Processes can wait on events and other code can notify them.

```cpp
sc_event my_event;

// Notify types:
my_event.notify();                          // Immediate notification (current delta)
my_event.notify(SC_ZERO_TIME);             // Delta-cycle notification
my_event.notify(10, SC_NS);               // Timed notification after 10 ns

// Cancel a pending notification:
my_event.cancel();

// Wait for an event inside SC_THREAD:
wait(my_event);

// Wait for multiple events:
wait(event_a | event_b);
wait(event_a & event_b);
```

**Notification types:**

| Type | When processes are triggered |
|------|------------------------------|
| Immediate (`notify()`) | In the **same** delta cycle (current evaluation phase) |
| Delta (`notify(SC_ZERO_TIME)`) | At the **next** delta cycle (next update phase) |
| Timed (`notify(t, SC_NS)`) | After simulated time `t` has elapsed |

---

## TLM-2.0 Concepts

### What is TLM-2.0?

Transaction-Level Modeling 2.0 (TLM-2.0) is a modeling methodology defined in IEEE 1666-2023. Instead of communicating through individual signals at every clock cycle, modules exchange complete **transactions** (read/write requests) using C++ function calls. This raises the abstraction level, dramatically increasing simulation speed and enabling early software development before RTL is available.

### Why use TLM?

- **Simulation speed**: 10x–100x faster than signal-level or RTL simulation.
- **Software development**: A processor model can run firmware against a bus and memory model long before hardware is taped out.
- **IP reuse**: Standardized sockets and payloads allow mixing IP blocks from different vendors.
- **Virtual prototyping**: Enables full system-on-chip (SoC) simulation.

### Basic Objects

| Object | Description |
|--------|-------------|
| **Initiator** | A module that initiates transactions (e.g., a CPU or DMA). Owns an `tlm_initiator_socket`. |
| **Target** | A module that responds to transactions (e.g., memory, peripheral). Owns a `tlm_target_socket`. |
| **Socket** | Typed connector that binds initiators to targets and carries the transport interface. `tlm_initiator_socket<>` and `tlm_target_socket<>`. |
| **Generic Payload (`tlm_generic_payload`)** | The standard transaction object containing address, command (READ/WRITE), data pointer, byte enables, response status, and optional extensions. |
| **Phase (`tlm_phase`)** | Marks the state of a non-blocking (AT) transaction: `BEGIN_REQ`, `END_REQ`, `BEGIN_RESP`, `END_RESP`. |
| **Path / Interface** | `tlm_fw_transport_if` (forward: initiator→target) and `tlm_bw_transport_if` (backward: target→initiator). |

### `tlm_generic_payload` Members

| Member | Accessor | Description |
|--------|----------|-------------|
| Command | `set_command(TLM_READ_COMMAND)` | Read or write |
| Address | `set_address(addr)` | Target address (64-bit) |
| Data pointer | `set_data_ptr(ptr)` | Pointer to read/write buffer |
| Data length | `set_data_length(len)` | Number of bytes |
| Byte enables | `set_byte_enable_ptr(be)` | Per-byte write mask |
| Response status | `set_response_status(TLM_OK_RESPONSE)` | Result of the transaction |
| DMI allowed | `set_dmi_allowed(true)` | Hint that DMI path is available |
| Extensions | `set_extension(ext)` | User-defined metadata |

### LT vs AT Coding Styles

```
Loosely-Timed (LT):
  initiator --b_transport(payload, delay)--> target
  (one blocking call, time annotated via delay)

Approximately-Timed (AT):
  initiator --nb_transport_fw(payload, BEGIN_REQ, t)--> target
  target    --nb_transport_bw(payload, END_REQ, t)  --> initiator
  target    --nb_transport_bw(payload, BEGIN_RESP, t)--> initiator
  initiator --nb_transport_fw(payload, END_RESP, t) --> target
```

### Route in LT Mode

1. Initiator calls `socket->b_transport(trans, delay)` (blocking).
2. Bus/router decodes the address and forwards to the appropriate target.
3. Target executes the read/write, sets `response_status = TLM_OK_RESPONSE`.
4. Call returns; initiator advances local time by `delay`.

### Route in AT Mode

1. Initiator calls `socket->nb_transport_fw(trans, phase=BEGIN_REQ, t)`.
2. Bus/router or target returns `TLM_ACCEPTED` (pipeline continues).
3. Target sends `nb_transport_bw(trans, BEGIN_RESP, t)` when data is ready.
4. Initiator consumes the response and calls `nb_transport_fw(trans, END_RESP, t)`.

---

## Mapping Design to SystemC Source Code

### Interface → Ports and Sockets

| Design Concept | SystemC Construct |
|---------------|-------------------|
| Signal/Wire | `sc_signal<T>` + `sc_in<T>` / `sc_out<T>` |
| Bus interface | `sc_port<interface_class>` |
| TLM initiator interface | `tlm::tlm_initiator_socket<>` |
| TLM target interface | `tlm::tlm_target_socket<>` |
| Hierarchical TLM port | `tlm_bw_transport_if` + `tlm_fw_transport_if` |

### Register → Member Variable

```cpp
// A 32-bit control register at offset 0x00
sc_uint<32> ctrl_reg;      // inside SC_MODULE or target class
```

### Function → Process or Method

| Design / Hardware Function | SystemC Mapping |
|---------------------------|-----------------|
| Reset sequence | `SC_THREAD` triggered on `reset` port |
| Clock-edge computation | `SC_METHOD` sensitive to `clk.pos()` |
| Read register | `b_transport` or `nb_transport_fw` handler in target |
| Interrupt | `sc_event` + `sc_out<bool>` interrupt port |

### Parts of a SystemC Model

| File / Class | Responsibility | C++ or SystemC? |
|-------------|----------------|-----------------|
| `*_module.h/.cpp` | Structural module (ports, sub-modules, process registration) | **SystemC** |
| `*_process.cpp` | Behavioral process bodies (`SC_THREAD`, `SC_METHOD`) | SystemC + C++ |
| `*_reg.h` | Register model (bit fields, access functions) | **C++ only** |
| `*_algorithm.cpp` | Pure computation (DSP, encoding, etc.) | **C++ only** |
| `sc_main.cpp` | Top-level instantiation and simulation control | SystemC (`sc_start`) |

> **Why separate SystemC and C++?**  
> SystemC-specific code (ports, signals, processes) depends on the simulator. Keeping pure algorithmic code in plain C++ allows it to be unit-tested, shared with firmware, or synthesized independently—without needing the SystemC runtime.

---

## Creating Bus Master/Slave IP

### Master (Initiator) IP

```cpp
SC_MODULE(MyMaster) {
    tlm::tlm_initiator_socket<> master_socket;

    void do_transfer() {
        tlm::tlm_generic_payload trans;
        sc_core::sc_time delay = sc_core::SC_ZERO_TIME;
        unsigned char data[4];

        trans.set_command(tlm::TLM_WRITE_COMMAND);
        trans.set_address(0x1000);
        trans.set_data_ptr(data);
        trans.set_data_length(4);
        trans.set_response_status(tlm::TLM_INCOMPLETE_RESPONSE);

        master_socket->b_transport(trans, delay);  // LT: blocking call
    }

    SC_CTOR(MyMaster) {
        SC_THREAD(do_transfer);
    }
};
```

### Slave (Target) IP

```cpp
SC_MODULE(MyTarget)
  : public tlm::tlm_fw_transport_if<>
{
    tlm::tlm_target_socket<> slave_socket;
    unsigned char mem[4096];

    void b_transport(tlm::tlm_generic_payload& trans,
                     sc_core::sc_time& delay) override {
        sc_dt::uint64 addr = trans.get_address();
        unsigned char* ptr  = trans.get_data_ptr();
        unsigned int   len  = trans.get_data_length();

        if (trans.get_command() == tlm::TLM_WRITE_COMMAND)
            memcpy(mem + addr, ptr, len);
        else
            memcpy(ptr, mem + addr, len);

        trans.set_response_status(tlm::TLM_OK_RESPONSE);
        delay += sc_core::sc_time(10, sc_core::SC_NS);
    }

    SC_CTOR(MyTarget) {
        slave_socket.bind(*this);
    }
    // Other required interface methods omitted for brevity
};
```

---

## Example: SystemC Producer–Consumer with Event

The following example demonstrates `sc_module`, `SC_THREAD`, `sc_signal`, `sc_event`, and port binding in a single self-contained program.

```cpp
#include <systemc.h>

SC_MODULE(Producer) {
    sc_out<int>  data_out;
    sc_out<bool> valid_out;
    sc_event     request_event;   // external code can notify this

    void produce() {
        int val = 0;
        while (true) {
            wait(10, SC_NS);           // advance time
            data_out.write(val++);
            valid_out.write(true);
            wait(SC_ZERO_TIME);        // delta cycle for the write to propagate
            valid_out.write(false);
        }
    }

    SC_CTOR(Producer) {
        SC_THREAD(produce);
    }
};

SC_MODULE(Consumer) {
    sc_in<int>  data_in;
    sc_in<bool> valid_in;

    void consume() {
        while (true) {
            wait(valid_in.posedge_event());   // wait for valid rising edge
            int v = data_in.read();
            std::cout << "@" << sc_time_stamp()
                      << " Received: " << v << "\n";
        }
    }

    SC_CTOR(Consumer) {
        SC_THREAD(consume);
    }
};

int sc_main(int, char*[]) {
    sc_signal<int>  data;
    sc_signal<bool> valid;

    Producer P("producer");
    Consumer C("consumer");

    // Named port binding
    P.data_out(data);
    P.valid_out(valid);
    C.data_in(data);
    C.valid_in(valid);

    sc_start(100, SC_NS);
    return 0;
}
```

---

## Example Exercise: 4-bit Up-Counter

The following implements a 4-bit up-counter with synchronous reset — a typical introductory SystemC design exercise.

```cpp
#include <systemc.h>

// ── Module Definition ───────────────────────────────────────────────────────
SC_MODULE(UpCounter4) {
    sc_in<bool>        clk;
    sc_in<bool>        rst;    // synchronous active-high reset
    sc_out<sc_uint<4>> q;      // 4-bit count output

    sc_uint<4> count;

    void do_count() {
        if (rst.read())
            count = 0;
        else
            count = count + 1;
        q.write(count);
    }

    SC_CTOR(UpCounter4) : count(0) {
        SC_METHOD(do_count);
        sensitive << clk.pos();   // triggered on rising clock edge
    }
};

// ── Testbench ───────────────────────────────────────────────────────────────
SC_MODULE(Testbench) {
    sc_out<bool> clk;
    sc_out<bool> rst;
    sc_in<sc_uint<4>> q;

    void run() {
        rst.write(1);
        clk.write(0);
        wait(20, SC_NS);

        rst.write(0);
        for (int i = 0; i < 20; i++) {
            clk.write(1); wait(5, SC_NS);
            clk.write(0); wait(5, SC_NS);
            std::cout << "@" << sc_time_stamp()
                      << "  count = " << q.read() << "\n";
        }
        sc_stop();
    }

    SC_CTOR(Testbench) {
        SC_THREAD(run);
    }
};

int sc_main(int, char*[]) {
    sc_signal<bool>        clk_s, rst_s;
    sc_signal<sc_uint<4>>  q_s;

    UpCounter4 dut("dut");
    dut.clk(clk_s);
    dut.rst(rst_s);
    dut.q(q_s);

    Testbench tb("tb");
    tb.clk(clk_s);
    tb.rst(rst_s);
    tb.q(q_s);

    sc_start();
    return 0;
}
```

---

## Installation

### Prerequisites

- CMake ≥ 3.5
- GCC ≥ 9.3 **or** Clang ≥ 13.0 **or** MSVC 2019+
- C++17 support enabled

### Build with CMake (Linux / macOS)

```bash
# Clone and enter the directory
cd systemc/

# Create and enter a build directory
mkdir build && cd build

# Configure (Release build, C++17)
cmake .. -DCMAKE_BUILD_TYPE=Release \
         -DCMAKE_CXX_STANDARD=17 \
         -DENABLE_EXAMPLES=ON

# Build the library and examples
make -j$(nproc)

# (Optional) Install
sudo make install
```

### Build with CMake (Windows / Visual Studio)

```powershell
mkdir build; cd build
cmake .. -G "Visual Studio 16 2019" -A x64 -DCMAKE_CXX_STANDARD=17
cmake --build . --config Release
```

### Run the Examples

After building, examples are placed in `build/examples/`:

```bash
# Run a built SystemC example directly
./examples/sysc/pipe/pipe

# Run via CMake check target
make check-examples
```

---

## Usage

### Minimal SystemC Program Template

```cpp
#include <systemc.h>

SC_MODULE(MyModule) {
    sc_in<bool>  clk;
    // ... ports ...

    void my_process() {
        // behavior
    }

    SC_CTOR(MyModule) {
        SC_METHOD(my_process);
        sensitive << clk.pos();
    }
};

int sc_main(int argc, char* argv[]) {
    sc_signal<bool> clk;
    MyModule m("m");
    m.clk(clk);

    sc_start(100, SC_NS);
    return 0;
}
```

### Compile a Custom Program Against the Installed Library

```bash
g++ -std=c++17 -I/opt/systemc/include \
    my_design.cpp \
    -L/opt/systemc/lib -lsystemc -Wl,-rpath,/opt/systemc/lib \
    -o my_design
```

---

## Notes / Limitations

- **No analog/mixed-signal** – This repository covers the digital SystemC and TLM-2.0 libraries. Analog extensions (SystemC AMS) are maintained separately.
- **IEEE standard is authoritative** – In any discrepancy between this implementation and the IEEE 1666-2023 standard, the standard takes precedence.
- **Regression tests are optional** – Set `ENABLE_REGRESSION=ON` in CMake to build and run the full test suite.
- **Context switching backend** – The library auto-selects between QuickThreads, POSIX threads, or C++ `std::thread` depending on platform and availability; `SC_THREAD`/`SC_CTHREAD` performance may vary.
- **Legacy GNU Autotools** – Still supported but CMake is recommended. See `docs/INSTALL_USING_AUTOTOOLS.md`.

---

## References

- [IEEE Std. 1666-2023 – SystemC Language Reference Manual][ieee]
- [Accellera Systems Initiative][accellera]
- [SystemC Community][community]
- [TLM-2.0 Community Forum][tlmforum]

[ieee]: https://ieeexplore.ieee.org/document/10246125
[accellera]: https://accellera.org
[community]: https://systemc.org
[tlmforum]: https://forums.accellera.org/forum/14-systemc-tlm-transaction-level-modeling/
