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

Transaction-Level Modeling 2.0 (TLM-2.0) is a modeling methodology defined in IEEE 1666-2023.
Instead of communicating through individual signals at every clock cycle, modules exchange complete **transactions** (read/write requests) using C++ function calls.
This raises the abstraction level, dramatically increasing simulation speed and enabling early software development before RTL is available.

## 1. Introduce
- **TLM-2.0** replaces pin-level signal connections with high-level function calls carrying a `tlm_generic_payload` transaction object. An **initiator** (e.g., a CPU model) calls transport functions on a **target** (e.g., a memory) through typed **sockets** and a **bus/router**.

- **Why use TLM?**
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
- Include:
  - **Untimed**: without `wait()`
  - **Loosely-Timed (LT)**:fast, low timing detail, uses blocking `b_transport()`, time advances by annotating a `sc_time` delay.
  - **Approximately-Timed (AT)**: higher timing accuracy without cycle-level detail ,uses non-blocking `nb_transport_f/bw()`.  Models pipeline stages and bus protocols with explicit phases.


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

---

- **Timing Annotation:** represents timing behavior by passing a delay value along with a transaction.
  - e.g.
```cpp
// Initiator ---------------------------
sc_time delay = SC_ZERO_TIME;

socket->b_transport(trans, delay);

// apply timing
wait(delay);

// Target ------------------------------
void b_transport(tlm::tlm_generic_payload& trans, sc_time& delay) {
    // processing takes 10 ns
    delay += sc_time(10, SC_NS);
}
```

- Temporal Decoupling: is an optimization technique where a process delays synchronization with the SystemC kernel to reduce simulation overhead.
  - each process maintains a local time
  - execution proceeds without calling wait() immediately
  - synchronization occurs only when necessary
```cpp
#include <tlm_utils/tlm_quantumkeeper.h>

tlm_utils::tlm_quantumkeeper qk;

void cpu_thread() {
    qk.reset();

    while (true) {
        sc_time delay = SC_ZERO_TIME;

        socket->b_transport(trans, delay);

        // accumulate local time instead of waiting
        qk.inc(delay);

        // synchronize only when needed
        if (qk.need_sync()) {
            wait(qk.get_local_time());
            qk.reset();
        }
    }
}
```

### 3.1. Loosely-Timed
timing
annotation and temporal decoupling
### 3.2. Approximately-timed

---

## 4.  Initiators, targets, sockets, and transaction bridges

| Object                                      | Description                                                                                                                                     |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| **Initiator**                               | A module that initiates transactions (e.g., a CPU or DMA). Owns an `tlm_initiator_socket`.                                                      |
| **Target**                                  | A module that responds to transactions (e.g., memory, peripheral). Owns a `tlm_target_socket`.                                                  |
| **Socket**                                  | Typed connector that binds initiators to targets and carries the transport interface. `tlm_initiator_socket<>` and `tlm_target_socket<>`.       |
| **Generic Payload (`tlm_generic_payload`)** | The standard transaction object containing address, command, data pointer, byte enables, response status, and optional extensions. |
| **Phase (`tlm_phase`)**                     | Marks the state of a non-blocking (AT) transaction: `BEGIN_REQ`, `END_REQ`, `BEGIN_RESP`, `END_RESP`.                                           |
| **Path / Interface**                        | `tlm_fw_transport_if` (forward: initiator --> target) and `tlm_bw_transport_if` (backward: target --> initiator).                                       |

- **`tlm_generic_payload` members**

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

----

### 4.1. Initiator / Target
![image](/images/sysc_tml_transaction.png)

- A **transaction object** is a data structure passed between initiators and targets using function calls.

- An **Initiator** is a module that can create new transaction objects and pass them on by calling a method using the `core interface`. 

- A **Target** is a module that acts as the `final destination` for a transaction.

- An **Interconnect** component is a module that has both a target socket and an initiator socket.
*The roles of initiator, interconnect, and target can change dynamically.*
----
### 4.2. Path
- **Forward path:** A transaction object is created by an initiator and passed to other modules.
- **Return path:** The transaction object is returned automatically.
- **Backward path:** Other modules send the transaction object back by calling a certain method.

----

### 4.3. Socket

![image](/images/sysc_tml_sockets.png)

- **Sockets** are used to pass transactions between `initiators` and `targets` in TLM. It combines `a port` with `an export`.
  - **`Initiator socket`** has a `port` for the `forward path` and an `export` for the `backward path`.
  - **`Target socket`**: has a `export` for the `forward path` and an `port` for the `backward path`.
- In TLM, sockets are connected using `bind()` or `operator()`.

---

## 5. Interface (TLM-2.0 Core Interfaces )
- An **interface** defines how transactions are communicated between components (initiator and target). There are three types of interface:
  - **Transport interface:** are the primary interfaces, used to transport transactions between initiators, targets and interconnect components.
  - **Direct memory interface (DMI):**  providing direct access to an area of memory owned by a target, has F/BW interface
  - **Debug transport interface:** providing debug access to an area of memory owned by a target, only has FW interface.

----

## 6. Blocking Transport Interface
- Uses only the **forward path** (no separate backward path).
- A complete transaction is executed within a single function call (`b_transport`).
- The initiator is blocked until the target finishes processing.

<br>

- **Roles**:
  - The `b_transport` method may call `wait()`.
  - It must NOT be called from an `SC_METHOD` process.
  - `Timing Model`:
    - Each transaction has only two timing points:
      - **START**: when `b_transport` is called
      - **END**: when `b_transport` returns
    - The `delay` parameter is used for **timing annotation**. It represents **relative time**, not absolute simulation time.

  - `Transaction Object`:
    - The transaction is passed **by reference**.
    - It can be **reused** across multiple calls.
    - It should **not contain timing information**.All timing must be modeled using the `delay` argument.

### 6.1. Blocking Transport

![image](/images/tlm_blocking_trans.png)

- The blocking transport (b_transport) may return immediately or after some simulation time.
- While one call is waiting, another thread in the initiator can call b_transport concurrently.

### 6.2. Temporal Decoupling

![image](/images/tlm_temporal_decoupling.png)

- A temporally decoupled initiator can run ahead of simulation time using a local time offset.
- It passes this offset as the delay argument to `b_transport`.
- Both initiator and target may increase this delay to model time.
- The effective completion time = simulation time + delay.
- However, simulation time only advances when the initiator calls `wait()`
- If `b_transport` itself calls `wait()`, the local time must reset to 0.

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
- A transaction is break downed into multiple `phrases transition`.
- Each phrases transition is associated with a `timing point`
- Each `call/return` from the `non-blocking transport method` may `correspond to` a `phrase transition`
- By restricting the number of timing points to two, it is possible to use the nb transport interface with the LT code style (not recommend)
-  It uses **all paths**, and interfaces `tlm_fw_nonblocking_transport_if` vs `tlm_bw_nonblocking_transport_if`
> **Cơ chế truyền tham số của non-blocking transport interface tương tự blocking transport interface ở chỗ:
truyền tham chiếu không const (non-const reference) tới transaction object truyền timing annotation
Điểm khác biệt là: non-blocking transport method còn truyền thêm một phase để biểu thị trạng thái của transaction
và trả về một giá trị enum để cho biết việc return từ function có đồng thời là một lần chuyển phase hay không**

### 7.1. Paths `nb_transport_fw` and `nb_transport_bw` 

-  `nb_transport` methods shall not call `wait`
-  several successive calls to `nb_transport_fw` from the same process could each initiate separate transactions without having to wait for the first transaction to complete
-  the `final timing point of a transaction` may be marked by a call to or a `return` from S`nb_transport` on either the forward path or the backward path.

### 7.1. Transaction Argument
- 1 transaction = 1 object (while it is active)
- That object moves back and forth between modules
- Everyone touches the same object, not copies
=> don't overwrite data too early or reuse object before transaction finishes

### 7.2. Phrase Argument
- phase = control signal for "who can touch the transaction and when", including:
    - **BEGIN_REQ**: initiator controls
    - **END_REQ**: target takes over
    - **BEGIN_RESP**: target controls
    - **END_RESP**: initiator finishes


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


---
