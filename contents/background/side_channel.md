# Cache Timing Side-Channel

!!! info "[Skip to TL;DR](#tldr)"

---

## Definition

A cache timing side-channel is a **microarchitectural leakage mechanism** that exposes information through **timing differences in memory access latency**.

It is the primary **data exfiltration channel** used by both Meltdown and Spectre .

---

## Core Principle

Memory access latency depends on where the data resides:

| Location | Typical Latency |
| -------- | --------------- |
| L1 Cache | ~4–15 cycles    |
| DRAM     | ~150–300 cycles |

This timing difference is:

* Measurable via high-resolution timers
* Deterministic enough to encode information

??? note
    The attacker does not read data directly; instead, it infers data based on access time.

---

## Information Encoding Mechanism

Sensitive data is encoded into cache state using a **probe array**:

```c
temp = probe_array[secret * STRIDE];
```

* `secret` determines which cache line is accessed
* That cache line becomes **resident (cached)**
* Other lines remain uncached

This creates a **one-to-one mapping between secret value and cache state**

---

## Flush+Reload Technique

The most commonly used technique is **Flush+Reload** .

### Execution Flow

```mermaid
flowchart LR
A[Flush Cache Line] --> B[Victim Access]
B --> C[Cache Line Loaded]
C --> D[Measure Access Time]
D --> E[Infer Cached Index]
```

---

### Step-by-Step

#### 1. Flush Phase

* Attacker evicts probe array from cache
* On x86: `clflush`
* On RISC-V: `cbo.inval` (Zicbom) or eviction loop

---

#### 2. Victim / Speculative Access

* Victim or transient execution accesses:

```c
probe_array[secret * STRIDE];
```

* Corresponding cache line is loaded

---

#### 3. Reload Phase

* Attacker measures access time for each index:

```c
uint64_t t0, t1;
asm volatile ("rdcycle %0" : "=r"(t0));
volatile uint8_t val = probe_array[i * STRIDE];
asm volatile ("rdcycle %0" : "=r"(t1));

latency = t1 - t0;
```

* Fast access → cache hit
* Slow access → cache miss

---

#### 4. Secret Recovery

* Index with lowest latency corresponds to `secret`

??? tip
    Only one cache line is significantly faster, enabling reliable identification of the encoded value.

---

## Why It Works

The attack relies on a fundamental asymmetry:

* **Architectural state** is rolled back after faults or misprediction
* **Microarchitectural state (cache)** is not

??? warning
    Cache contents persist even if the instruction that caused the access is never committed.

This property enables:

* Meltdown → leak across privilege boundary
* Spectre → leak across control-flow boundary

---

## Alternative Techniques

While Flush+Reload requires shared memory, other techniques exist:

### Prime+Probe

* Attacker fills cache sets (prime)
* Victim evicts lines
* Attacker measures which sets were evicted

### Flush+Flush

* Measures time taken to flush cache lines
* Faster and stealthier than Flush+Reload

??? note
    These techniques do not require shared memory, making them applicable in broader scenarios.

---

## RISC-V Timing Measurement

On RISC-V, timing is typically measured using the **cycle counter**:

```c
uint64_t t0, t1;
asm volatile ("rdcycle %0" : "=r"(t0));
volatile uint8_t val = probe_array[i * 512];
asm volatile ("rdcycle %0" : "=r"(t1));

uint64_t latency = t1 - t0;
```

Typical threshold:

* Cache hit: < ~15 cycles
* Cache miss: > ~150 cycles

??? warning
    The `rdcycle` CSR is accessible in unprivileged mode unless explicitly restricted, enabling high-resolution timing attacks.

---

## Required Conditions

A cache timing side-channel requires:

| Condition                    | Role                                   |
| ---------------------------- | -------------------------------------- |
| Shared cache                 | Enables observation of victim activity |
| Timing source                | Measures access latency                |
| Deterministic cache behavior | Ensures repeatability                  |
| Data-dependent memory access | Encodes secret into cache              |

---

## Limitations

The channel depends on:

* Noise (interrupts, scheduling)
* Cache associativity and replacement policy
* Timer resolution

It does not:

* Provide direct memory access
* Work without a measurable timing difference

---

## Role in Meltdown and Spectre

| Attack   | Role of Side-Channel                          |
| -------- | --------------------------------------------- |
| Meltdown | Extracts transiently accessed privileged data |
| Spectre  | Extracts speculatively accessed data          |

??? info
    The side-channel is not the vulnerability itself; it is the mechanism that converts transient execution effects into observable data.

---

## Architectural Insight

The vulnerability arises from:

* Performance optimizations (caching, speculation)
* Lack of rollback for microarchitectural state

This creates a persistent channel:

* Invisible at ISA level
* Observable via timing

---

## TL;DR

* Cache timing side-channel leaks data via **access latency differences**
* Uses techniques like **Flush+Reload** and **Prime+Probe**
* Encodes secrets into **cache state via data-dependent access**
* Requires:
    * Shared cache
    * High-resolution timer
* Not rolled back after speculative execution
* Enables data exfiltration in both Meltdown and Spectre

!!! info ""
    The cache acts as a covert communication channel between transient execution and attacker observation.

---
