---
author: "Phong Nguyen"
title: "System C TLM 2.0"
date: "2026-03-29"
description: "System C, TLM Notes"
tags: ["c, embedded,systemc,tlm"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
TLM 2.0 - Transaction Level Modeling

## 1.1 Introduce

![image](/images/sysc_tml_overview.png)

TLM is a library built on top of the SystemC library. It consists of a set of `core interfaces, objects, base protocols, and utilities` that enable the `TLM concept`.

- There are two specific **coding styles**:
  - **Loosely-timed (LT):** Focuses on functionality, not precise timing. Uses one delay for the whole transaction. *(blocking transport interface)*
  - **Approximately-timed (AT):** Models timing behavior more accurately than LT and is more realistic, but slower. *(non-blocking transport interface)*

![image](/images/sysc_tml_coding_styles_uc.png)

----
## 1.2 Initiator / Target
![image](/images/sysc_tml_transaction.png)

- An **Initiator** is a module that can create new transaction objects and pass them on by calling a method using the `core interface`. 
- A **Target** is a module that acts as the `final destination` for a transaction.

- A **transaction object** is a data structure passed between initiators and targets using function calls.
- An **interconnect** component is a module that has both a target socket and an initiator socket.
----
## 1.3 Path
- Path is ?
- **Forward path:** A transaction object is created by an initiator and passed to other modules.
- **Return path:** The transaction object is returned automatically.
- **Backward path:** Other modules send the transaction object back by calling a certain method.
----
## 1.4 Socket
![image](/images/sysc_tml_sockets.png)
**Sockets** are used to pass transactions between `initiators` and `targets` in TLM. It combines `a port` with `an export`.
- **Initiator socket**: has a `port` for the `forward path` and an `export` for the `backward path`.
- **Target socket**: has a `export` for the `forward path` and an `port` for the `backward path`.
- In TLM, sockets are connected using `bind()` or `operator()`.
    - An initiator socket connects to a target socket.
    - An interconnect may connect upstream and downstream sockets.
    - Binding establishes the communication path between components.

---
## 1.5 Interface (TLM-2.0 Core Interfaces )
An **interface** defines how transactions are communicated between components (initiator and target). There are three types of interface:
- **Transport interface:** used to transport transactions between initiators, targets and interconnect components.
- **Direct memory interface (DMI):** Provides `read/write` access to an area of `memory owned by a target` using a `direct pointer`.
- **Debug transport interface:** Provides debug `read/write` access to an area of `memory owned by a target`.

----
## 1.6 Transport Interface
- **Blocking transport interface**
  - Each transaction has 2 timing points: **START** and **END**.
  - Uses **forward path and return path only**.
  - The initiator completes a transaction with a target using a single call.

- **Non-blocking transport interface**
  - Each transaction has multiple timing points.
  - Uses **all paths**.
  - A transaction is finished through multiple calls or a single call.

----
## 1.7 TLM 2.0 Library Overview

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

## 1.8 Examples
## 1.8.1 Connection
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

## 1.9 Communication Through Generic Payload
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

| Layer             | Technology                                                   |
| ----------------- | ------------------------------------------------------------ |
| Language          | C++17                                                        |
| Standard          | IEEE Std. 1666-2023                                          |
| Build System      | CMake ≥ 3.5, GNU Autotools (legacy)                          |
| Compilers         | GCC ≥ 9.3, Clang ≥ 13.0, MSVC 2019+                          |
| Platforms         | Linux (x86_64, aarch64), macOS, Windows                      |
| Context switching | QuickThreads, POSIX Threads, Windows Fibers, C++ std::thread |
| Tracing           | VCD (Value Change Dump), WIF (Waveform Intermediate Format)  |

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

| Type         | Macro                         | Description                                                       |
| ------------ | ----------------------------- | ----------------------------------------------------------------- |
| `SC_METHOD`  | `SC_METHOD(func)`             | Re-executes on every trigger; **no** `wait()` allowed.            |
| `SC_THREAD`  | `SC_THREAD(func)`             | Runs once; can call `wait()` to suspend and resume. Keeps state.  |
| `SC_CTHREAD` | `SC_CTHREAD(func, clk.pos())` | Like `SC_THREAD` but clocked; uses `wait()` to advance one clock. |

**Difference**: `SC_METHOD` is stateless (re-enters from top each trigger), while `SC_THREAD`/`SC_CTHREAD` are stateful coroutines that suspend mid-execution.

### Ports

Ports provide the interface between a module and its external channels. They are templated on a channel interface.

| Class         | Direction                                |
| ------------- | ---------------------------------------- |
| `sc_in<T>`    | Input port                               |
| `sc_out<T>`   | Output port                              |
| `sc_inout<T>` | Bidirectional port                       |
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

| Type                           | When processes are triggered                           |
| ------------------------------ | ------------------------------------------------------ |
| Immediate (`notify()`)         | In the **same** delta cycle (current evaluation phase) |
| Delta (`notify(SC_ZERO_TIME)`) | At the **next** delta cycle (next update phase)        |
| Timed (`notify(t, SC_NS)`)     | After simulated time `t` has elapsed                   |

---

## TLM-2.0 Concepts

**TLM-2.0** replaces pin-level signal connections with high-level function calls carrying a `tlm_generic_payload` transaction object. An **initiator** (e.g., a CPU model) calls transport functions on a **target** (e.g., a memory) through typed **sockets** and a **bus/router**. Two coding styles exist:

- **Loosely-Timed (LT)** – uses blocking `b_transport()`. Simpler and faster; time advances by annotating a `sc_time` delay.
- **Approximately-Timed (AT)** – uses non-blocking `nb_transport_fw()` / `nb_transport_bw()`. Models pipeline stages and bus protocols with explicit phases.

### What is TLM-2.0?

Transaction-Level Modeling 2.0 (TLM-2.0) is a modeling methodology defined in IEEE 1666-2023. Instead of communicating through individual signals at every clock cycle, modules exchange complete **transactions** (read/write requests) using C++ function calls. This raises the abstraction level, dramatically increasing simulation speed and enabling early software development before RTL is available.

### Why use TLM?

- **Simulation speed**: 10x–100x faster than signal-level or RTL simulation.
- **Software development**: A processor model can run firmware against a bus and memory model long before hardware is taped out.
- **IP reuse**: Standardized sockets and payloads allow mixing IP blocks from different vendors.
- **Virtual prototyping**: Enables full system-on-chip (SoC) simulation.

### Basic Objects

| Object                                      | Description                                                                                                                                     |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Initiator**                               | A module that initiates transactions (e.g., a CPU or DMA). Owns an `tlm_initiator_socket`.                                                      |
| **Target**                                  | A module that responds to transactions (e.g., memory, peripheral). Owns a `tlm_target_socket`.                                                  |
| **Socket**                                  | Typed connector that binds initiators to targets and carries the transport interface. `tlm_initiator_socket<>` and `tlm_target_socket<>`.       |
| **Generic Payload (`tlm_generic_payload`)** | The standard transaction object containing address, command (READ/WRITE), data pointer, byte enables, response status, and optional extensions. |
| **Phase (`tlm_phase`)**                     | Marks the state of a non-blocking (AT) transaction: `BEGIN_REQ`, `END_REQ`, `BEGIN_RESP`, `END_RESP`.                                           |
| **Path / Interface**                        | `tlm_fw_transport_if` (forward: initiator→target) and `tlm_bw_transport_if` (backward: target→initiator).                                       |

### `tlm_generic_payload` Members

| Member          | Accessor                               | Description                     |
| --------------- | -------------------------------------- | ------------------------------- |
| Command         | `set_command(TLM_READ_COMMAND)`        | Read or write                   |
| Address         | `set_address(addr)`                    | Target address (64-bit)         |
| Data pointer    | `set_data_ptr(ptr)`                    | Pointer to read/write buffer    |
| Data length     | `set_data_length(len)`                 | Number of bytes                 |
| Byte enables    | `set_byte_enable_ptr(be)`              | Per-byte write mask             |
| Response status | `set_response_status(TLM_OK_RESPONSE)` | Result of the transaction       |
| DMI allowed     | `set_dmi_allowed(true)`                | Hint that DMI path is available |
| Extensions      | `set_extension(ext)`                   | User-defined metadata           |

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

| Design Concept          | SystemC Construct                             |
| ----------------------- | --------------------------------------------- |
| Signal/Wire             | `sc_signal<T>` + `sc_in<T>` / `sc_out<T>`     |
| Bus interface           | `sc_port<interface_class>`                    |
| TLM initiator interface | `tlm::tlm_initiator_socket<>`                 |
| TLM target interface    | `tlm::tlm_target_socket<>`                    |
| Hierarchical TLM port   | `tlm_bw_transport_if` + `tlm_fw_transport_if` |

### Register → Member Variable

```cpp
// A 32-bit control register at offset 0x00
sc_uint<32> ctrl_reg;      // inside SC_MODULE or target class
```

### Function → Process or Method

| Design / Hardware Function | SystemC Mapping                                      |
| -------------------------- | ---------------------------------------------------- |
| Reset sequence             | `SC_THREAD` triggered on `reset` port                |
| Clock-edge computation     | `SC_METHOD` sensitive to `clk.pos()`                 |
| Read register              | `b_transport` or `nb_transport_fw` handler in target |
| Interrupt                  | `sc_event` + `sc_out<bool>` interrupt port           |

### Parts of a SystemC Model

| File / Class      | Responsibility                                               | C++ or SystemC?      |
| ----------------- | ------------------------------------------------------------ | -------------------- |
| `*_module.h/.cpp` | Structural module (ports, sub-modules, process registration) | **SystemC**          |
| `*_process.cpp`   | Behavioral process bodies (`SC_THREAD`, `SC_METHOD`)         | SystemC + C++        |
| `*_reg.h`         | Register model (bit fields, access functions)                | **C++ only**         |
| `*_algorithm.cpp` | Pure computation (DSP, encoding, etc.)                       | **C++ only**         |
| `sc_main.cpp`     | Top-level instantiation and simulation control               | SystemC (`sc_start`) |

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
