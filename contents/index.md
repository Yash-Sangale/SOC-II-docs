# Spectre & Meltdown on RISC-V

### Architecture Vulnerability Analysis for 64-bit Microcontroller Design

---

## About This Documentation

This documentation presents a **technical analysis of transient execution vulnerabilities**, specifically **Meltdown** and **Spectre**, in the context of a **64-bit single-core RISC-V microcontroller design**.

The goal of this work is to:

* Explain how **modern CPU microarchitectural features** introduce security vulnerabilities
* Analyze the **architectural requirements** for Meltdown and Spectre attacks
* Evaluate whether these vulnerabilities apply to a **minimal RISC-V microcontroller**
* Demonstrate **why and where these attacks fail** in a simplified architecture
* Explore **residual risks (e.g., cache timing side-channels)** and mitigation strategies

This documentation is based on a structured research study and has been adapted into an interactive format using **MkDocs** for better readability and navigation.

---

## Scope of the Document

This documentation focuses on:

* Transient execution attacks (**Meltdown & Spectre**)
* Microarchitectural concepts:
    * Out-of-order execution
    * Speculative execution
    * Branch prediction
    * Cache-based side channels
* RISC-V architecture (RV64I) in a **microcontroller context**
* Security implications of:
    * Absence of virtual memory
    * Lack of privilege separation
    * Shared cache behavior

---

## How to Use This Documentation

Depending on your goal, you can navigate this documentation in different ways:

###  For First-Time Readers

Start with:

1. **Abstract** → Overview of the work
2. **Background** → Understand Meltdown & Spectre fundamentals
3. **Architecture** → Learn about the RISC-V design

### For Technical Deep Dive

Jump to:

* **Attack Mechanisms** → Step-by-step execution of attacks
* **RISC-V Failure Analysis** → Why attacks fail on this design
* **Countermeasures** → Hardware & software mitigation strategies

### For Quick Understanding

Read:

* **Vulnerability Analysis Summary**
* **Conclusions**

---

## Document Structure

The documentation is organized into the following sections:

### 1. Overview

Overview of the research objectives, methodology, and key findings.

### 2. Background

* Meltdown (CVE-2017-5754)
* Spectre (CVE-2017-5753, CVE-2017-5715)
* Cache timing side-channel (Flush+Reload)

### 3. Architectural Preconditions

Defines the **necessary hardware conditions** required for these attacks.

### 4. RISC-V Microcontroller Design

Describes the **target system architecture**, including:

* Pipeline structure
* Cache configuration
* Privilege model

### 5. Vulnerability Analysis

Explains:

* Why **Meltdown is not applicable**
* Why **Spectre is limited or conditional**

### 6. Attack Mechanisms (Reference Systems)

Step-by-step explanation of how:

* Meltdown works on privileged processors
* Spectre exploits branch prediction

### 7. RISC-V Attack Failure Analysis

A detailed breakdown of **why attacks fail on the microcontroller**, including:

* Absence of privilege boundary
* No page fault mechanism
* Flat memory model

### 8. Countermeasures

* Software-level mitigations
* Hardware-level protections
* Industry-standard defenses

### 9. Conclusions

Key takeaways regarding:

* Architectural security vs complexity
* Trade-offs in microcontroller design

### 10. References

Academic papers, specifications, and industry resources used in this work.

---

## Quick Navigation

| Section                                                          | Description                                     |
| ---------------------------------------------------------------- | ----------------------------------------------- |
| [Abstract](overview.md)                                          | Overview of the research                        |
| [Meltdown](background/meltdown.md)                               | Transient execution attack via privilege bypass |
| [Spectre](background/spectre.md)                                 | Speculative execution via branch misprediction  |
| [Preconditions](architecture/preconditions.md)                   | Required architectural features                 |
| [RISC-V Design](architecture/riscv_design.md)                    | Target microcontroller architecture             |
| [Vulnerability Analysis](architecture/vulnerability_analysis.md) | Applicability of attacks                        |
| [Meltdown Attack](attacks/meltdown_attack.md)                    | Attack flow on real CPUs                        |
| [Spectre Attack](attacks/spectre_attack.md)                      | Branch predictor exploitation                   |
| [Failure Analysis](attacks/riscv_failure_analysis.md)            | Why attacks fail                                |
| [Countermeasures](countermeasures.md)                            | Defense strategies                              |
| [Conclusion](conclusion.md)                                      | Final insights                                  |
| [References](references.md)                                      | Sources                                         |

---

## Key Takeaway

> **Security vulnerabilities like Meltdown and Spectre are not just implementation flaws — they are consequences of architectural design choices.**

This documentation highlights how a **minimalist RISC-V microcontroller architecture** can inherently eliminate entire classes of vulnerabilities by **removing the conditions required for their existence**.

---

##  Intended Audience

This documentation is intended for:

* Embedded systems engineers
* Computer architecture students
* Hardware security researchers
* RISC-V developers
* Anyone interested in **microarchitectural security**

---

##  Notes

* This documentation assumes basic familiarity with:

  * Computer architecture concepts
  * C programming
  * CPU pipeline fundamentals
* All examples are simplified for clarity but retain technical accuracy.

---
