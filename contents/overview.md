# Overview

!!! info "[Skip to TL;DR](#tldr)"
---

## Context

Transient execution attacks such as *Meltdown* and *Spectre* exploit the gap between:

* **Architectural correctness** (what the CPU commits)
* **Microarchitectural behavior** (what happens during execution)

Modern processors expose this gap due to:

* Out-of-order execution
* Speculative execution
* Branch prediction
* Shared cache hierarchies

These features introduce observable side-effects (e.g., cache state changes) that persist even when instructions are not architecturally committed.

---

## Target Architecture

The analysis is performed on a **64-bit RISC-V microcontroller** with the following properties:

| Component          | Description                               |
| ------------------ | ----------------------------------------- |
| ISA                | RV64I                                     |
| Execution Model    | In-order issue, OoO execution (ROB-based) |
| Privilege Levels   | Single mode (no U/S separation)           |
| Memory Model       | Physical addressing only                  |
| Cache              | 2-way set-associative                     |
| Branch Predictor   | Present                                   |
| Exception Handling | Machine-mode only                         |

---

## Threat Model

The attacker:

* Executes arbitrary code on the system
* Controls inputs to target code paths
* Measures execution timing (e.g., cycle counter)

The objective is to extract data via:

* **Privilege boundary violation** (Meltdown)
* **Speculative execution misuse** (Spectre)

---

## Attack Preconditions

The feasibility of transient execution attacks depends on the following:

| Precondition                | Role                               |
| --------------------------- | ---------------------------------- |
| Speculative / OoO execution | Enables transient execution window |
| Branch predictor            | Required for Spectre               |
| Shared cache                | Enables timing side-channel        |
| Privilege boundary          | Required for Meltdown              |
| Isolated execution context  | Required for Spectre V2            |

??? note
    The absence of any required precondition invalidates the corresponding attack class.

---

## Meltdown: Applicability

Meltdown requires:

* A **hardware-enforced privilege boundary**
* A **faulting memory access** delayed by speculative execution

In the target system:

* No virtual memory
* No page tables
* No user/kernel separation

As a result:

* No access generates a privilege fault
* No protected memory region exists

??? warning
    Any memory location can be accessed directly. This eliminates the need for a Meltdown-style bypass.

---

## Spectre: Applicability

Spectre relies on:

* Mispredicted branches
* Speculative execution of unintended paths
* Cache-based side-channel leakage

In the target system:

* Branch predictor → Present
* OoO execution → Present
* Cache → Present

However:

* No hardware isolation between software components

??? note
    Spectre Variant 1 remains possible in scenarios with shared software execution (e.g., cooperative multitasking).

??? note
    Spectre Variant 2 is not applicable due to absence of a separate privileged context.

---

## Transient Execution Leakage Model

```mermaid
flowchart LR
A["Speculative Execution"] --> B["Transient Data Access"]
B --> C["Cache State Update"]
C --> D["Rollback (Architectural State)"]
D --> E["Timing Measurement"]
E --> F["Data Recovery"]
```

??? tip
    The cache state is not rolled back after speculative execution, making it a persistent side-channel.

---

## Architectural Implications

The analyzed microcontroller exhibits:

* **Flat memory model** → No access control
* **Uniform privilege level** → No isolation boundary
* **Observable cache behavior** → Timing side-channel present

This leads to:

* Elimination of **Meltdown attack surface**
* Reduction of **Spectre attack scope**
* Retention of **cache-based leakage mechanisms**

---

## TL;DR

* Meltdown is **not applicable**
    * No privilege boundary
    * No page fault mechanism
* Spectre V2 is **not applicable**
    * No separate execution context
* Spectre V1 is **conditionally possible**
    * Requires shared software execution
* Cache timing side-channel **exists**
    * Due to shared cache

??? info ""
    Security properties are determined by architectural composition. Removing privilege separation eliminates entire vulnerability classes rather than mitigating them.

---
