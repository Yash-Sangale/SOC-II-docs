# Spectre

!!! info "[Skip to TL;DR](#tldr)"

---

## Definition

Spectre is a class of **transient execution attacks** that exploit **speculative execution triggered by branch misprediction**.

Unlike Meltdown, Spectre does not require a privilege violation. Instead, it causes a victim to **speculatively execute unintended instructions**, which leak sensitive data through microarchitectural side-effects.

---

## Root Cause

Modern processors use **branch prediction** to improve performance:

* Predict the outcome of conditional branches
* Speculatively execute instructions before the branch resolves

If the prediction is incorrect:

* Speculative results are discarded architecturally
* **Microarchitectural side-effects remain**

This allows an attacker to:

* Influence the prediction mechanism
* Control speculative execution paths
* Extract data via cache timing

??? note
    Spectre exploits *correctly implemented* performance optimizations rather than violating access permissions.

---

## Spectre Variants

### Variant 1 — Bounds Check Bypass (CVE-2017-5753)

* Exploits conditional branches (e.g., bounds checks)
* Trains predictor to assume branch is always taken
* Causes speculative execution of out-of-bounds access

---

### Variant 2 — Branch Target Injection (CVE-2017-5715)

* Exploits indirect branch prediction (BTB poisoning)
* Redirects speculative execution to attacker-controlled gadgets
* Enables cross-context data leakage

??? note
    Variant 1 operates within the same context. Variant 2 targets cross-context or cross-privilege execution.

---

## Execution Flow (Variant 1)

```mermaid
flowchart LR
A[Train Branch Predictor] --> B[Send Malicious Input]
B --> C[Branch Misprediction]
C --> D[Speculative Execution]
D --> E[Out-of-Bounds Access]
E --> F[Cache Encoding]
F --> G[Rollback]
G --> H[Timing Measurement]
```

---

## Step-by-Step Mechanism (Variant 1)

### 1. Predictor Training

The attacker repeatedly executes a branch with valid inputs:

```c
if (x < size) { ... }
```

* Branch predictor learns: **branch is taken**

---

### 2. Misprediction Trigger

The attacker supplies an **out-of-bounds index**:

* Predictor still assumes branch is taken
* Speculative execution enters the branch body

---

### 3. Transient Execution

Speculative path performs:

```c
value = array[x];                  // out-of-bounds read
temp = probe[value * 512];         // encode into cache
```

* Secret value is used to access probe array
* Cache state is modified

---

### 4. Rollback

* Branch resolves as incorrect
* Speculative instructions are discarded
* Architectural state is restored

??? note
    Cache modifications persist despite rollback.

---

### 5. Data Exfiltration

The attacker measures access time to probe array:

* Fast access → cached → reveals secret
* Slow access → not cached

---

## Variant 2 (High-Level)

Variant 2 differs in mechanism:

* Targets **indirect branches**
* Attacker poisons **Branch Target Buffer (BTB)**
* Victim speculatively jumps to attacker-chosen code

This enables:

* Execution of **gadgets**
* Leakage of sensitive data across contexts

??? warning
    Variant 2 requires shared predictor state across execution contexts.

---

## Required Architectural Conditions

Spectre requires:

| Requirement             | Role                             |
| ----------------------- | -------------------------------- |
| Branch predictor        | Enables misprediction            |
| Speculative execution   | Executes unintended instructions |
| Shared cache            | Provides timing side-channel     |
| Predictable victim code | Provides exploitable gadget      |

For Variant 2:

| Additional Requirement | Role                            |
| ---------------------- | ------------------------------- |
| Shared predictor state | Enables cross-context poisoning |
| Indirect branches      | Required for BTB manipulation   |

---

## Key Property

The defining characteristic of Spectre is:

> **Speculative execution follows attacker-influenced paths within otherwise valid code.**

No access violation is required.

---

## Differences from Meltdown

| Property                     | Spectre               | Meltdown         |
| ---------------------------- | --------------------- | ---------------- |
| Requires privilege violation | No                    | Yes              |
| Trigger mechanism            | Branch misprediction  | Faulting load    |
| Target                       | Victim code paths     | Protected memory |
| Scope                        | Same or cross-context | Cross-privilege  |

---

## Limitations

Spectre requires:

* A **predictable victim code pattern**
* A **shared microarchitectural resource** (cache, predictor)

It does not:

* Directly bypass hardware memory protection
* Guarantee leakage without a usable side-channel

---

## TL;DR

* Exploits **branch misprediction + speculative execution**
* Causes execution of unintended code paths
* Leaks data via **cache timing side-channel**
* Variant 1:
    * Bounds check bypass
    * Same-context attack

* Variant 2:
    * Branch target injection
    * Cross-context attack

* Requires:
    * Branch predictor
    * Speculative execution
    * Shared cache

!!! info ""
    Spectre is a control-flow manipulation attack that leverages speculative execution to expose data, rather than violating memory access permissions.

---
