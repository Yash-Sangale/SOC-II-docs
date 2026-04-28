# Spectre Attack (Privileged Processors)

!!! info "[Skip to TL;DR](#tldr)"

---

## Context

This section describes the **operational model of Spectre attacks** on processors implementing:

* Speculative execution
* Branch prediction
* Shared microarchitectural state

Unlike Meltdown, Spectre manipulates **control flow speculation** rather than exploiting a faulting access.

---

## Attack Objective

* Force a victim to execute **attacker-influenced speculative paths**
* Access sensitive data during speculation
* Leak data via **cache timing side-channel**[^1][^2] 

---

## Attack Preconditions

Spectre requires:

| Requirement             | Role                        |
| ----------------------- | --------------------------- |
| Branch predictor        | Enables misprediction       |
| Speculative execution   | Executes unintended path    |
| Shared cache            | Encodes leaked data         |
| Predictable victim code | Provides exploitable gadget |

For Variant 2:

| Additional Requirement | Role                            |
| ---------------------- | ------------------------------- |
| Shared predictor state | Enables cross-context poisoning |
| Indirect branches      | Required for BTB attack         |

---

## Variant 1: Bounds Check Bypass

### Victim Code Pattern

```c  {#f6l4y1}
if (x < array1_size) {
    uint8_t val = array1[x];
    temp = probe_array[val * 512];
}
```

---

## Execution Flow (Variant 1)

```mermaid {#q4s8kx}
flowchart LR
A[Train Predictor] --> B[Send Malicious Input]
B --> C[Branch Misprediction]
C --> D[Speculative Execution]
D --> E[Out-of-Bounds Access]
E --> F[Cache Encoding]
F --> G[Rollback]
G --> H[Timing Measurement]
```

---

## Step-by-Step Execution

### 1. Branch Predictor Training

* Attacker repeatedly invokes victim with valid inputs
* Branch predictor learns:

    * Condition is **true (taken)**

---

### 2. Misprediction Trigger

* Attacker supplies out-of-bounds index `x`
* Predictor still assumes branch is taken

Result:

* Speculative execution enters guarded block

---

### 3. Speculative Out-of-Bounds Access

```c {#ymrjpn}
val = array1[x];   // x is attacker-controlled
```

* Access may reference **sensitive memory**
* Occurs only transiently

---

### 4. Cache Encoding

```c {#l8x3d0}
temp = probe_array[val * 512];
```

* Secret-dependent memory access
* Corresponding cache line is loaded

---

### 5. Rollback

* Branch resolves as false
* Speculative instructions are discarded
* Architectural state is restored

??? note
    Cache state persists despite rollback.

---

### 6. Data Exfiltration

* Attacker measures probe array access latency
* Fast access → identifies `val`

---

## Variant 2: Branch Target Injection

### Mechanism

* Targets **indirect branches**
* Attacker poisons **Branch Target Buffer (BTB)**

### Execution Flow

```mermaid {#7qz5rp}
flowchart LR
A[Poison BTB] --> B[Victim Indirect Branch]
B --> C[Speculative Jump to Gadget]
C --> D[Secret Access]
D --> E[Cache Encoding]
E --> F[Rollback]
F --> G[Timing Measurement]
```

---

### Step-by-Step

1. Attacker trains BTB with malicious target
2. Victim executes indirect branch
3. Predictor redirects execution to attacker-controlled gadget
4. Gadget accesses sensitive data
5. Cache encodes data
6. Attacker extracts via timing

??? warning
    Variant 2 enables cross-context attacks (e.g., user → kernel).

---

## Timing Side-Channel Integration

Both variants rely on:

* Cache state modification
* Timing-based observation

Measurement example (RISC-V): Timing measurement using RISC-V cycle counter:[^3]

```c {#4a7m9o}
uint64_t t0, t1;
asm volatile ("rdcycle %0" : "=r"(t0));
volatile uint8_t val = probe_array[i * 512];
asm volatile ("rdcycle %0" : "=r"(t1));

latency = t1 - t0;
```

---

## Key Mechanism

Spectre exploits:

> **Mispredicted control flow leading to transient execution of unintended instructions**

Combined with:

> **Persistent microarchitectural side-effects**

---

## Why It Works

* Branch prediction assumes **past behavior predicts future**
* Speculative execution executes ahead of validation
* No isolation of predictor state across contexts

This allows:

* Attacker-controlled influence over execution path
* Leakage of data without violating access permissions

---

## Differences from Meltdown

| Property                     | Spectre              | Meltdown          |
| ---------------------------- | -------------------- | ----------------- |
| Trigger                      | Branch misprediction | Faulting load     |
| Requires privilege violation | No                   | Yes               |
| Target                       | Control flow         | Memory protection |
| Scope                        | Same/cross context   | Cross-privilege   |

---

## Limitations

Spectre requires:

* Predictable victim code
* Shared microarchitectural resources
* Sufficient speculation depth

It does not:

* Directly bypass hardware permissions
* Work without a side-channel

---

## Relation to RISC-V MCU

In the analyzed microcontroller:

* Branch predictor → Present
* OoO execution → Present
* Cache → Present

However:

* No privilege separation
* No process isolation

Implications:

* Spectre V2 → Not applicable[^3]
* Spectre V1 → Only relevant in multi-component software systems

??? note
    The attack depends on existence of a meaningful isolation boundary.

---

## TL;DR

* Exploits **branch misprediction + speculative execution**

* Forces execution of unintended code paths

* Leaks data via **cache timing**

* Variant 1:
    * Bounds check bypass
    * Same-context attack

* Variant 2:
    * Branch target injection
    * Cross-context attack

* Requires:
    * Branch predictor
    * Speculation
    * Shared cache

!!! info ""
    Spectre is a control-flow manipulation attack that leverages speculation to expose data without violating architectural access rules.

---
[^1]: Lipp et al., *Meltdown*, USENIX Security 2018. [→ References](../references.md#ref-1)
[^2]: Kocher et al., *Spectre Attacks*, IEEE S&P 2019. [→ References](../references.md#ref-2)
[^3]: RISC-V International, *Privileged Architecture Manual v20211203*. [→ References](../references.md#ref-3)
[^5]: Intel Corporation, *Analysis of Speculative Execution Side Channels*, 2018. [→ References](../references.md#ref-5)
[^8]: Patterson & Hennessy, *Computer Organization and Design: RISC-V Edition*, 2017. [→ References](../references.md#ref-8)