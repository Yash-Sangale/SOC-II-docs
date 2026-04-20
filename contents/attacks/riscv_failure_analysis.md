# RISC-V Failure Analysis

!!! info "[Skip to TL;DR](#tldr)"

---

## Context

This section constructs **hypothetical Meltdown and Spectre attack attempts** on the analyzed RISC-V microcontroller and traces their execution through the pipeline.

The objective is to identify the **exact failure point** of each attack based on architectural constraints .

---

## Meltdown: Failure Analysis

### Attack Setup

Assume:

* Attacker attempts to read memory at address `0x80001000`
* Memory is *logically* considered sensitive (software convention)
* Standard Meltdown sequence is executed

---

## Execution Trace

| Step | Action                   | Outcome                                  |
| ---- | ------------------------ | ---------------------------------------- |
| 1    | Flush probe array        | Achievable (via eviction loop if needed) |
| 2    | Load from target address | **Succeeds architecturally**             |
| 3    | Speculative encode       | Redundant                                |
| 4    | Timing measurement       | Possible                                 |
| 5    | Secret recovery          | Directly available                       |

---

## Critical Observation

```mermaid
flowchart LR
A[Load Instruction] --> B[Physical Address Access]
B --> C[Data Returned]
C --> D[No Fault Generated]
D --> E[No Speculative Window]
```

* No page table walk
* No privilege check
* No exception generated

??? warning
    The load does not fault. Therefore, no transient execution window is created.

---

## Root Cause of Failure

Meltdown requires:

* Faulting access + delayed exception

In this design:

* Memory access is always valid
* No protection boundary exists

### Result

* Data is returned **architecturally**, not transiently
* No need for speculative bypass

---

## Key Finding

> The attack succeeds as a **normal memory read**, not as a Meltdown attack.

??? note
    This is not a vulnerability—it is a consequence of the flat memory model.

---

## Formal Interpretation

Let:

* (M) = Meltdown attack
* (P) = protected memory domain

Requirement:

* (\exists P) such that access causes fault

In this system:

* (P) does not exist

Therefore:

* (M) is undefined

---

## Spectre V1: Failure / Conditional Success

### Attack Setup

Assume:

* Multiple software tasks (A and B) share execution
* Shared function:

```c
void process_input(uint32_t x) {
    if (x < buffer_size) {
        val = buffer[x];
        temp = probe[val * 512];
    }
}
```

---

## Execution Trace

| Step | Action                 | Outcome  |
| ---- | ---------------------- | -------- |
| 1    | Train branch predictor | Succeeds |
| 2    | Mispredict branch      | Succeeds |
| 3    | Speculative access     | Succeeds |
| 4    | Cache encode           | Succeeds |
| 5    | Timing measurement     | Succeeds |
| 6    | Secret recovery        | Succeeds |

---

## Mechanism

```mermaid
flowchart LR
A[Train Predictor] --> B[Mispredict]
B --> C[Speculative Access]
C --> D[Cache Update]
D --> E[Rollback]
E --> F[Timing Leak]
```

---

## Key Observation

* No hardware memory protection
* Tasks share same physical address space

??? warning
Speculative access to another task’s data is not blocked by hardware.

---

## Result

| Condition           | Outcome              |
| ------------------- | -------------------- |
| Single trust domain | No meaningful attack |
| Multiple components | Attack possible      |

---

## Interpretation

* Attack is **not prevented by hardware**
* Feasibility depends on **software-level isolation**

??? note
    Spectre V1 is valid only if a logical boundary exists between software components.

---

## Spectre V2: Failure Analysis

### Requirements

* Separate execution contexts
* Shared predictor state
* Indirect branch targets

### Observation

* Single privilege level
* No context isolation
* No victim domain

---

## Execution Breakdown

| Step | Action                    | Outcome             |
| ---- | ------------------------- | ------------------- |
| 1    | Poison BTB                | Possible            |
| 2    | Redirect victim execution | **No valid target** |
| 3    | Execute gadget            | Not applicable      |
| 4    | Leak data                 | Not applicable      |

---

## Root Cause of Failure

```mermaid
flowchart LR
A[Single Execution Context] --> B[No Victim Isolation]
B --> C[No Redirection Target]
C --> D[Attack Fails]
```

* No distinct victim context exists
* All execution occurs in same domain

---

## Result

* Spectre V2 is **not applicable**

---

## Cache Side-Channel Behavior

### Observation

* Cache is shared
* Timing differences measurable

### Implication

* Side-channel remains functional

??? note
    Cache timing leakage is independent of Meltdown/Spectre applicability.

---

## Direct Comparison

| Property              | RISC-V MCU     | Privileged CPU |
| --------------------- | -------------- | -------------- |
| Privilege boundary    | Absent         | Present        |
| Page faults           | Absent         | Present        |
| Direct memory read    | Allowed        | Restricted     |
| Speculative execution | Present        | Present        |
| Cache side-channel    | Present        | Present        |
| Meltdown              | Not applicable | Exploitable    |
| Spectre V1            | Conditional    | Exploitable    |
| Spectre V2            | Not applicable | Exploitable    |

---

## Structural Reasoning

The failure of attacks is due to:

| Missing Feature      | Effect                    |
| -------------------- | ------------------------- |
| Privilege boundary   | No Meltdown target        |
| Page fault mechanism | No transient fault window |
| Context isolation    | No Spectre V2 target      |

---

## Key Insight

The system does not *mitigate* attacks—it **eliminates their preconditions**.

??? tip
    This represents security through architectural simplification rather than defensive mechanisms.

---

## TL;DR

* Meltdown:
    * Fails because no fault occurs
    * Memory access succeeds directly

* Spectre V1:
    * Possible only with shared software components
    * Not prevented by hardware

* Spectre V2:
    * Fails due to lack of context separation

* Cache side-channel:
    * Present and observable

!!! info ""
    The microcontroller is immune to Meltdown and partially exposed to Spectre solely due to its architectural design choices.

---
