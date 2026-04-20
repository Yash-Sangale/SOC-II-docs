# RISC-V Microcontroller Design

!!! info "[Skip to TL;DR](#tldr)"

---

## Context

This section defines the **target architecture** used for vulnerability evaluation. The analysis is performed on a **64-bit single-core RISC-V microcontroller** designed with minimal system complexity and no operating system support.

The architectural choices directly determine the **presence or absence of attack preconditions**.

---

## Design Specification

| Parameter        | Specification                                |
| ---------------- | -------------------------------------------- |
| ISA              | RV64I                                        |
| Core Type        | Single-core                                  |
| Execution Model  | In-order issue with OoO execution capability |
| Pipeline         | Multi-stage with Reorder Buffer (ROB)        |
| Privilege Model  | Unprivileged (Machine mode only)             |
| Memory Model     | Physical addressing (no virtual memory)      |
| Cache            | 2-way set-associative                        |
| Branch Predictor | Present (e.g., 2-bit saturating / BTB)       |
| Exception Model  | Machine-mode exceptions only                 |

---

## Execution Pipeline

The processor implements:

* **In-order instruction issue**
* **Out-of-order execution**
* **Reorder Buffer (ROB)** for commit and rollback

### Execution Model

```mermaid
flowchart LR
A[Fetch] --> B[Decode]
B --> C[Issue (In-order)]
C --> D[Execute (OoO)]
D --> E[ROB]
E --> F[Commit / Flush]
```

* Instructions are issued in program order
* Execution may occur out-of-order
* Results are committed only after validation

??? note
    The presence of a ROB enables speculative execution and rollback of architectural state.

---

## Speculative Execution Capability

Due to:

* OoO execution
* Branch prediction

The processor supports:

* Speculative instruction execution
* Transient data propagation

However:

* Speculative effects are confined to **microarchitectural state**
* No architectural commitment occurs until validation

---

## Privilege Model

The processor operates in a **single privilege domain**:

* Machine mode only
* No user mode (U-mode)
* No supervisor mode (S-mode)

### Implications

* No hardware-enforced memory isolation
* No distinction between:
    * User space
    * Kernel space

??? warning
    All code executes with identical privileges and unrestricted memory access.

---

## Memory System

### Addressing Model

* **Physical addressing only**
* No virtual memory translation

### Missing Components

* No page tables
* No MMU
* No `satp` register
* No page table walker

### Implications

* No address translation latency
* No permission checks on memory access
* No page fault exceptions

??? note
    Memory protection, if required, must be implemented at the software level.

---

## Cache Architecture

* 2-way set-associative cache
* Shared across all executing code

### Properties

* Data-dependent cache fills
* Measurable latency differences (hit vs miss)

### Implications

* Enables **timing side-channel observation**
* No hardware partitioning or isolation

??? warning
The cache remains a shared observable resource and can act as a leakage channel.

---

## Branch Prediction

* Hardware branch predictor is present
* Likely implementation:
    * 2-bit saturating counters
    * Branch Target Buffer (BTB)

### Capabilities

* Predicts conditional branches
* Enables speculative execution

### Implications

* Predictor state can be trained
* Enables **Spectre-style misprediction scenarios**

---

## Exception Handling

* Machine-mode exceptions only
* No user-level exception handling

### Missing Exceptions

* No page fault exceptions
* No privilege violation exceptions

### Implications

* Faults are limited to:

  * Illegal instructions
  * Access faults (non-permission related)

??? note
    Absence of page faults removes the fundamental trigger required for Meltdown.

---

## Architectural Constraints

The design intentionally omits:

* Virtual memory
* Privilege hierarchy
* Process isolation

This results in:

* A **flat execution environment**
* A **single trust domain**

---

## Security-Relevant Properties

| Feature              | Status  | Security Impact           |
| -------------------- | ------- | ------------------------- |
| OoO Execution        | Present | Enables speculation       |
| Branch Predictor     | Present | Enables Spectre           |
| Cache                | Present | Enables side-channel      |
| Privilege Separation | Absent  | Eliminates Meltdown       |
| Virtual Memory       | Absent  | No protected regions      |
| Page Faults          | Absent  | No transient fault window |

---

## Design Rationale

The architecture reflects a **microcontroller-oriented design philosophy**:

* Minimal hardware complexity
* Deterministic behavior
* No OS-level abstraction

This leads to:

* Reduced attack surface for privilege-based exploits
* Retention of microarchitectural side-effects

??? tip
    The design trades isolation for simplicity, shifting security responsibility from hardware to software.

---

## Architectural Consequences

From a security perspective:

* **No hardware isolation boundary exists**
* **All memory is globally accessible**
* **Speculative execution remains active**
* **Cache remains observable**

This creates:

* No Meltdown attack surface
* Reduced Spectre scope
* Persistent side-channel exposure

---

## TL;DR

* RV64I-based single-core microcontroller
* In-order issue + OoO execution (ROB-based)
* No virtual memory, no MMU, no page tables
* Single privilege mode (no isolation)
* Branch predictor and cache are present

Implications:

* Meltdown → not applicable
* Spectre → partially applicable
* Cache timing side-channel → present

!!! info
    The absence of privilege and memory protection mechanisms eliminates entire vulnerability classes but does not remove microarchitectural leakage channels.

---
