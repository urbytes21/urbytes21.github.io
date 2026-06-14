---
author: "Phong Nguyen"
title: "System C TLM 2.0"
date: "2026-03-29"
description: "System C, TLM Notes"
tags: ["systemc,tlm"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 1    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
# SystemC - TLM
**Transaction-Level Modeling 2.0 (TLM-2.0)** is a modeling methodology defined in IEEE 1666-2023.
Instead of communicating through individual signals at every clock cycle, modules exchange complete **transactions** (read/write requests) using C++ function calls.
This raises the `abstraction level`, dramatically increasing simulation speed and enabling early software development before RTL is available.

---
## 1. Introduce
**TLM-2.0** replaces pin-level signal connections with high-level function calls carrying a `tlm_generic_payload` transaction object. An **initiator** (e.g., a CPU model) calls transport functions on a **target** (e.g., a memory) through typed **sockets** and a **bus/router**.

**Why use TLM?**
- **Simulation speed**: 10x–100x faster than signal-level or RTL simulation.
- **Software development**: A processor model can run firmware against a bus and memory model long before hardware is taped out.
- **IP reuse**: Standardized sockets and payloads allow mixing IP blocks from different vendors.
- **Virtual prototyping**: Enables full system-on-chip (SoC) simulation.

----
## 2. Transaction-Level Modeling, Use Cases, and Abstraction

![image](/images/sysc_tml_overview.png)

- TLM models systems at a high abstraction level, focusing on communication (transactions) instead of signal-level details.

- TLM-2.0 separates:
  - **Interfaces (APIs)**: Standard communication (e.g., b_transport, nb_transport)
  - **Coding styles**: Flexible, use-case dependent

- A TLM model requires multiple communicating processes; single-threaded models are not TLM.

----
## 3. Coding Styles
A coding style is a set of programming idioms that work well together, not a specific abstraction level or API. Including:
- **Untimed**: No `wait()` calls. Pure functional behavior, no time passes. Used for early algorithmic exploration.
- **Loosely-Timed (LT)**:  Uses blocking `b_transport()` with an `sc_time` delay annotation. Fast simulation, low timing detail. Best for software development.
- **Approximately-Timed (AT)**:  Uses non-blocking `nb_transport_fw/bw()` with explicit protocol phases (`BEGIN_REQ`, `END_REQ`, `BEGIN_RESP`, `END_RESP`). Models pipeline stages and bus protocols. Best for hardware architecture analysis.
  ```bash
  # Loosely-Timed (LT):
    initiator --b_transport(payload, delay)--> target
    (one blocking call, time annotated via delay)
  # Route
  1. Initiator calls `socket->b_transport(trans, delay)` (blocking).
  2. Bus/router decodes the address and forwards to the appropriate target.
  3. Target executes the read/write, sets `response_status = TLM_OK_RESPONSE`.
  4. Call returns; initiator advances local time by `delay`.


  # Approximately-Timed (AT):
    initiator --nb_transport_fw(payload, BEGIN_REQ, t)--> target
    target    --nb_transport_bw(payload, END_REQ, t)  --> initiator
    target    --nb_transport_bw(payload, BEGIN_RESP, t)--> initiator
    initiator --nb_transport_fw(payload, END_RESP, t) --> target
  # Route
  1. Initiator calls `socket->nb_transport_fw(trans, phase=BEGIN_REQ, t)`.
  2. Bus/router or target returns `TLM_ACCEPTED` (pipeline continues).
  3. Target sends `nb_transport_bw(trans, BEGIN_RESP, t)` when data is ready.
  4. Initiator consumes the response and calls `nb_transport_fw(trans, END_RESP, t)`.
  ```
<br>

**Timing Annotation:** represents timing behavior by passing a delay value along with a transaction.
```cpp
// Initiator ---------------------------
sc_time delay = SC_ZERO_TIME;
socket->b_transport(trans, delay); // send transaction
wait(delay);  // apply accumulated delay afterward

// Target ------------------------------
void b_transport(tlm::tlm_generic_payload& trans, sc_time& delay) {
    delay += sc_time(10, SC_NS);  // annotate 10 ns processing time
}
```
<br>

**Temporal Decoupling** is an optimization technique where a process delays synchronization with the SystemC kernel to reduce simulation overhead.
- Each initiator maintains a `local time` offset (**quantum**)
- Transactions execute without blocking on `wait()` each time
- Synchronization happens only when the `local time` exceeds the `global quantum`
- `need_sync()` checks if the `local time` has exceeded the `global quantum.`
  ```cpp
  tlm_utils::tlm_quantumkeeper qk;
  qk.set_global_quantum(sc_time(100, SC_NS));  // quantum budget = 100 ns

  void cpu_thread() {
      qk.reset();                              // local time = 0

      while (true) {
          sc_time delay = SC_ZERO_TIME;

          // Transaction 1
          socket->b_transport(trans, delay);   // target adds 20 ns -> delay = 20 ns
          // accumulate local time instead of waiting
          qk.inc(delay);                       // local time = 20 ns
          if (qk.need_sync()) { /* */ }        // 20 ns < 100 ns -> keep going
        
          // Transaction 2
          socket->b_transport(trans, delay);   // target adds 40 ns -> delay = 40 ns
          qk.inc(delay);                       // local time = 60 ns
          if (qk.need_sync()) { /* */ }        // 60 ns < 100 ns -> keep going
        
          // Transaction 3
          socket->b_transport(trans, delay);   // target adds 50 ns -> delay = 50 ns
          qk.inc(delay);                       // local time = 110 ns
          if (qk.need_sync()) {                // 110 ns > 100 ns -> SYNC
              wait(qk.get_local_time());       // advance kernel to 110 ns
              qk.reset();                      // local time = 0
          }
      }
  }
  // Without temporal decoupling, every b_transport() call would require a kernel context switch.
  //  With it, hundreds of transactions can execute before a single sync.

  void cpu_thread() {
    while (true) {
        sc_time delay = SC_ZERO_TIME;

        socket->b_transport(trans, delay);   // target adds 20 ns -> delay = 20 ns
        wait(delay);                         // SYNC with kernel immediately

        socket->b_transport(trans, delay);   // target adds 40 ns -> delay = 40 ns
        wait(delay);                         // SYNC with kernel immediately

        socket->b_transport(trans, delay);   // target adds 50 ns -> delay = 50 ns
        wait(delay);                         // SYNC with kernel immediately
    }
  }
  ```
<br>

### 3.1. Loosely-Timed
**Loosely-Timed style** uses `timing annotation` and temporal decoupling together.
- Target annotates delay instead of calling `wait()`
- Initiator accumulates local time via quantum keeper
- Syncs with kernel only when quantum is exceeded
<br>

### 3.2. Approximately-timed
**Approximately-timed style** uses explicit protocol phases and non-blocking transport — no wait() allowed, multiple transactions can be in-flight simultaneously.

---
## 4. Initiators, Targets, Sockets, and Transaction Bridges

| Object                                      | Description                                                                                                                                     |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Initiator**                               | A module that initiates transactions (e.g., a CPU or DMA). Owns an `tlm_initiator_socket`.                                                      |
| **Target**                                  | A module that responds to transactions (e.g., memory, peripheral). Owns a `tlm_target_socket`.                                                  |
| **Socket**                                  | Typed connector that binds initiators to targets and carries the transport interface. `tlm_initiator_socket<>` and `tlm_target_socket<>`.       |
| **Generic Payload (`tlm_generic_payload`)** | The standard transaction object containing address, command, data pointer, byte enables, response status, and optional extensions. |
| **Phase (`tlm_phase`)**                     | Marks the state of a non-blocking (AT) transaction: `BEGIN_REQ`, `END_REQ`, `BEGIN_RESP`, `END_RESP`.                                           |
| **Path / Interface**                        | `tlm_fw_transport_if` (forward: initiator --> target) and `tlm_bw_transport_if` (backward: target --> initiator).                                       |
<br>

**`tlm_generic_payload` members**

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
<br>

### 4.1. Initiator / Target
![image](/images/sysc_tml_transaction.png)

**A transaction object** is a data structure passed between initiators and targets using function calls.

**AnInitiator** is a module that can create new transaction objects and pass them on by calling a method using the `core interface`. 

**A Target** is a module that acts as the `final destination` for a transaction.

**An Interconnect** component is a module that has both a target socket and an initiator socket.
*The roles of initiator, interconnect, and target can change dynamically.*
<br>

### 4.2. Path
**Forward path:** A transaction object is created by an initiator and passed to other modules.
**Return path:** The transaction object is returned automatically.
**Backward path:** Other modules send the transaction object back by calling a certain method.
<br>

### 4.3. Socket

![image](/images/sysc_tml_sockets.png)

**Sockets** are used to pass transactions between `initiators` and `targets` in TLM. It combines `a port` with `an export`.
- **`Initiator socket`** has a `port` for the `forward path` and an `export` for the `backward path`.
- **`Target socket`**: has a `export` for the `forward path` and an `port` for the `backward path`.
- In TLM, sockets are connected using `bind()` or `operator()`.

---
## 5. Interface (TLM-2.0 Core Interfaces )
**An interface** defines how transactions are communicated between components (initiator and target). There are three types of interface:
- **Transport interface:** are the primary interfaces, used to transport transactions between initiators, targets and interconnect components.
- **Direct memory interface (DMI):** providing direct access to an area of memory owned by a target, has F/BW interface
- **Debug transport interface:** providing debug access to an area of memory owned by a target, only has FW interface.

### 5.1. Direct Memory Interface (DMI)
**DMI** bypasses the transport interface, initiator accesses target memory directly without sending transactions. Has both forward and backward paths.
```cpp
// Initiator: request DMI access
tlm::tlm_dmi dmi_data;
if (socket->get_direct_mem_ptr(trans, dmi_data)) {
    // read/write directly to target memory, no b_transport() needed
    unsigned char* ptr = dmi_data.get_dmi_ptr();
    memcpy(ptr, data, length);
}

// Target grant DMI access
bool get_direct_mem_ptr(tlm::tlm_generic_payload& trans, tlm::tlm_dmi& dmi_data) {
    dmi_data.set_dmi_ptr(memory);                  // expose memory pointer
    dmi_data.set_start_address(0x0);
    dmi_data.set_end_address(0xFFFF);
    return true;                                    // grant access
}

// Target, invalidate DMI if memory changes (backward path)
socket->invalidate_direct_mem_ptr(0x0, 0xFFFF);
```

### 5.2.  Debug Transport Interface
**DTI** provides debug access to target memory without affecting simulation time, no `wait()`, no delay. Forward path only.
```cpp
// Initiator — request debug access
unsigned int bytes_read = socket->transport_dbg(trans);

// Target — handle debug request
unsigned int transport_dbg(tlm::tlm_generic_payload& trans) {
    unsigned char* ptr  = trans.get_data_ptr();
    sc_dt::uint64 addr  = trans.get_address();
    unsigned int  len   = trans.get_data_length();
    memcpy(ptr, &memory[addr], len);     // copy memory, no timing side effects
    return len;
}
```

----
## 6. Blocking Transport Interface
**Blocking Transport Interface** is a communication mechanism where the initiator sends a transaction and waits until the target completes it before continuing (like a function call)
- Uses only the **forward path** (no separate backward path).
- A complete transaction is executed within a single function call (`b_transport`).
- The initiator is blocked until the target finishes processing.
<br>

**Roles**:
- The `b_transport` method may call `wait()`.
- It must NOT be called from an `SC_METHOD` process.
- `Timing Model`:
  - Each transaction has only two timing points:
    - **START**: when `b_transport` is called
    - **END**: when `b_transport` returns
  - The `delay` parameter is used for **timing annotation**. It represents **relative time**, not absolute simulation time.
<br>

**Transaction Object**:
- The transaction is passed **by reference**.
- It can be **reused** across multiple calls.
- It should **not contain timing information**.All timing must be modeled using the `delay` argument.
<br>

### 6.1. Blocking Transport

![image](/images/tlm_blocking_trans.png)

- The blocking transport (`b_transport`) may return immediately or after some simulation time.
- While one call is waiting, another thread in the initiator can call `b_transport` concurrently.
<br>

### 6.2. Temporal Decoupling

![image](/images/tlm_temporal_decoupling.png)

- A temporally decoupled initiator can run ahead of simulation time using a local time offset.
- It passes this offset as the delay argument to `b_transport`.
- Both initiator and target may increase this delay to model time.
- The effective completion time = simulation time + delay.
- However, simulation time only advances when the initiator calls `wait()`
- If `b_transport` itself calls `wait()`, the local time must reset to 0.
<br>

### 6.3. Time quantum

![image](/images/tlm_temporal_decoupling.png)

- A temporally decoupled initiator may run ahead of simulation time, but only up to a fixed **time quantum**.
- The initiator accumulates local time using the delay variable.
- When the local time offset exceeds the quantum, the initiator must synchronize by calling `wait(delay)`.
- This synchronization allows other initiators to execute and catch up in simulation time. 
- After synchronization, the local time offset is reset to zero.
- Within a quantum, transactions execute sequentially without advancing simulation time.
- The SystemC scheduler does not track local time; it only advances on `wait()` calls.

----
## 7. Non-blocking Transport Interface
**Non-blocking Transport Interface** is a communication mechanism where the `initiator` sends a `transaction` and continues immediately without waiting — the `target` responds later via a backward path. (like sending an email)
- A transaction is break downed into multiple `phrases transition`.
- Each phrases transition is associated with a `timing point`
- Each `call/return` from the `non-blocking transport method` may `correspond to` a `phrase transition`
- By restricting the number of timing points to two, it is possible to use the nb transport interface with the LT code style (not recommend)
- It uses **all paths**, and interfaces `tlm_fw_nonblocking_transport_if` vs `tlm_bw_nonblocking_transport_if`
<br>

### 7.1. Paths `nb_transport_fw` and `nb_transport_bw` 
**A path** is the direction in which a transaction travels between the `initiator` and `target`.
- `nb_transport` methods shall not call `wait`
- Several successive calls to `nb_transport_fw` from the same process could each initiate separate transactions without having to wait for the first transaction to complete
- The `final timing point of a transaction` may be marked by a call to or a `return` from S`nb_transport` on either the forward path or the backward path.
<br>

### 7.1. Transaction Argument
A transaction argument is the single object passed between initiator and target that carries all transaction data :`address`, `data`, `command`, `response status`.
- 1 transaction = 1 object (while it is active)
- That object moves back and forth between modules
- Everyone touches the same object, not copies
=> Don't overwrite data too early or reuse object before transaction finishes
<br>

### 7.2. Phrase Argument
**A phase argument** tracks the current state of a transaction as it moves between `initiator` and `target`.
`BEGIN_REQ  -->  END_REQ  -->  BEGIN_RESP  -->  END_RESP`
- phase = control signal for "who can touch the transaction and when", including:
  - **BEGIN_REQ**: initiator controls
  - **END_REQ**: target takes over
  - **BEGIN_RESP**: target controls
  - **END_RESP**: initiator finishes
<br>

### 7.3. <tlm_sync_enum> Return Value
- **TLM_ACCEPTED**:
  -  The callee must not modify the transaction object, phase, or time argument.
  -  Indicates that the return path is not used.
  -  The caller typically needs to wait/yield for a future response.
- **TLM_UPDATED**: 
  - Indicates that the return path is used.
  - The protocol state has advanced (phase transition occurred).
  - The caller must inspect updated arguments and react accordingly.
- **TLM_COMPLETED**:The callee has completed the transaction (at this socket).
  - The transaction object and time may be modified.
  - The phase is undefined and should be ignored.
  - No further nb_transport calls are allowed for this transaction on this socket.
  - Completion does not guarantee success, need to check response status.

---
## 8. Examples
### 8.1 Connection
**Module and socket:**
```cpp
#include <systemc>
#include <tlm>

using namespace sc_core;

// Initiator Module with socket
class InitiatorModule : public sc_module,
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
<br>

**Socket and another socket**
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

class InitiatorModule: public sc_module,
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
<br>

## 8.2. Communication Through Generic Payload
`Initiator` and `target` communicate by calling functions. All functions must be defined in both modules before communication begins.
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


---
