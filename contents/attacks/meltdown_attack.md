# Meltdown Attack (Privileged Processors)

!!! info "[Skip to TL;DR](#tldr)"

---

## Context

This section describes the **operational model of the Meltdown attack** on processors that implement:

* Virtual memory
* Hardware privilege separation
* Out-of-order execution

The attack exploits a **transient execution window** created between:

* Memory access issuance
* Permission check resolution

---

## Attack Objective

* Read **privileged (kernel) memory** from an unprivileged context
* Bypass architectural access control using **speculative execution**
* Extract data via **cache timing side-channel**[^1][^6]

---

## Attack Preconditions

Meltdown requires:

| Requirement          | Role                         |
| -------------------- | ---------------------------- |
| Virtual memory (MMU) | Defines protected memory     |
| Privilege levels     | Enforces access restrictions |
| Page fault mechanism | Delays fault handling        |
| OoO execution        | Enables speculative access   |
| Shared cache         | Enables data exfiltration    |

??? note
    All conditions must be present simultaneously for the attack to succeed.

---

## Attack Pipeline

```mermaid
flowchart LR
A[Flush Probe Array] --> B[Illegal Kernel Load]
B --> C[Speculative Execution]
C --> D[Cache Encoding]
D --> E[Fault Raised]
E --> F[Pipeline Flush (ROB)]
F --> G[Timing Measurement]
G --> H[Secret Recovery]
```

---

## Step-by-Step Execution

### 1. Cache Preparation (Flush Phase)

* Attacker evicts probe array from all cache levels
* Ensures deterministic timing behavior

Typical methods:

* x86: `clflush`
* RISC-V: `cbo.inval` (Zicbom) or eviction loops
[^4]
---

### 2. Transient Faulting Load

The attacker executes a load from a **privileged virtual address**:

```c
unsigned char secret = *(unsigned char*)kernel_addr;
```

### Microarchitectural Behavior

* MMU initiates:
    * Page table walk
    * Privilege check

* OoO engine:
    * Executes dependent instructions **before fault resolution**

??? warning
    The load may transiently return valid data before the fault is delivered.

---

### 3. Speculative Data Propagation

The transient value is used in a dependent instruction:

```c
temp = probe_array[secret * 512];
```

Effect:

* A specific cache line is loaded
* Secret is encoded into cache state

---

### 4. Fault Handling and Rollback

* CPU detects illegal access
* Raises **page fault exception**
* Reorder Buffer (ROB) is flushed
* Architectural state is restored

??? note
    Microarchitectural state (cache contents) is not reverted.

---

### 5. Side-Channel Measurement (Reload Phase)

The attacker measures access latency:

```c
uint64_t t0, t1;
asm volatile ("rdtsc" : "=A"(t0));
temp = probe_array[i * 512];
asm volatile ("rdtsc" : "=A"(t1));

latency = t1 - t0;
```

Interpretation:

* Low latency → cache hit
* High latency → cache miss

---

### 6. Secret Reconstruction

* Identify index with lowest latency
* Recover corresponding byte value

Repeat:

* For each byte in target memory
* Sequentially reconstruct full memory region

---

## Timing Characteristics

| Access Type  | Latency         |
| ------------ | --------------- |
| L1 Cache Hit | ~4–10 cycles    |
| DRAM Access  | ~200–300 cycles |

This difference provides:

* High signal-to-noise ratio
* Reliable data extraction

---

## Throughput

Empirical results (from literature):

* ~500–2000 bytes/sec[^1]
* ~2–8 seconds for 4 KB memory page

??? note
    Throughput depends on noise, cache hierarchy, and system load.

---

## Key Mechanism

The attack relies on:

> **Transient availability of unauthorized data before permission enforcement**

Combined with:

> **Non-rollback of microarchitectural state**

---

## Why It Works

The CPU prioritizes performance:

* Executes instructions speculatively
* Defers exception handling

This creates:

* A **temporal gap** between:

    * Data access
    * Access validation

During this gap:

* Sensitive data becomes transiently usable

---

## Limitations

Meltdown is effective only when:

* Memory protection exists
* Faults are deferred
* Speculative execution is deep enough

It does not apply when:

* No privilege boundary exists
* No page faults occur
* Execution is strictly in-order

---

## Relation to RISC-V MCU

In contrast to the analyzed microcontroller:

* No MMU → no page table walk
* No privilege separation → no protected memory
* No page faults → no transient fault window
[^3]
??? warning
    The Meltdown sequence cannot be instantiated without a faulting access condition.

---

## TL;DR

* Exploits **speculative execution before permission checks resolve**

* Reads **kernel memory from user space**

* Uses **cache timing side-channel (Flush+Reload)**

* Requires:

    * MMU + page tables
    * Privilege separation
    * Page faults
    * OoO execution

* Fails if:

    * No protection boundary exists
    * No fault is generated

!!! info ""
    Meltdown is fundamentally a race between data access and permission enforcement, observable via cache state.

---
[^1]: Lipp et al., *Meltdown*, USENIX Security 2018. [→ References](../references.md#ref-1)
[^3]: RISC-V International, *Privileged Architecture Manual v20211203*. [→ References](../references.md#ref-3)
[^4]: RISC-V International, *Zicbom Extension v1.0*. [→ References](../references.md#ref-4)
[^5]: Intel Corporation, *Analysis of Speculative Execution Side Channels*, 2018. [→ References](../references.md#ref-5)
[^6]: Gruss et al., *Flush+Flush*, DIMVA 2016. [→ References](../references.md#ref-6)
[^8]: Patterson & Hennessy, *Computer Organization and Design: RISC-V Edition*, 2017. [→ References](../references.md#ref-8)