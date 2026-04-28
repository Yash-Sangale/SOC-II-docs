# References

---

## Core Research Papers

<span id="ref-1">**[1]**</span> Lipp, M., Schwarz, M., Gruss, D., Prescher, T., Haas, W., Horn, J., Mangard, S.
**Meltdown: Reading Kernel Memory from User Space**.
USENIX Security Symposium, 2018.
[:material-file-document: Paper (arXiv)](https://arxiv.org/abs/1801.01207){ .md-button .md-button--primary }
[:material-web: meltdownattack.com](https://meltdownattack.com/){ .md-button }

<span id="ref-2">**[2]**</span> Kocher, P., Horn, J., Fogh, A., Genkin, D., Gruss, D., Haas, W., Yarom, Y.
**Spectre Attacks: Exploiting Speculative Execution**.
IEEE Symposium on Security and Privacy, 2019.
[:material-file-document: Paper (arXiv)](https://arxiv.org/abs/1801.01203){ .md-button .md-button--primary }
[:material-web: spectreattack.com](https://spectreattack.com/){ .md-button }

---

## Architecture Specifications

<span id="ref-3">**[3]**</span> RISC-V International.
**The RISC-V Instruction Set Manual, Volume II: Privileged Architecture**.
Version 20211203, 2021.
[:material-file-document: Specification (PDF)](https://github.com/riscv/riscv-isa-manual/releases/download/Priv-v1.12/riscv-privileged-20211203.pdf){ .md-button .md-button--primary }

<span id="ref-4">**[4]**</span> RISC-V International.
**RISC-V Cache Management Operation ISA Extensions (Zicbom)**.
Version 1.0, 2022.
[:material-file-document: Specification](https://github.com/riscv/riscv-CMOs){ .md-button .md-button--primary }

---

## Industry Whitepapers

<span id="ref-5">**[5]**</span> Intel Corporation.
**Analysis of Speculative Execution Side Channels**.
White Paper, Revision 4.0, 2018.
[:material-file-document: Whitepaper (PDF)](https://newsroom.intel.com/wp-content/uploads/sites/11/2018/01/Intel-Analysis-of-Speculative-Execution-Side-Channels.pdf){ .md-button .md-button--primary }

<span id="ref-7">**[7]**</span> ARM Limited.
**Cache Speculation Side-channels**.
Whitepaper, Version 2.4, 2018.
[:material-file-document: Whitepaper](https://developer.arm.com/support/security-update){ .md-button .md-button--primary }

---

## Side-Channel Research

<span id="ref-6">**[6]**</span> Gruss, D., Maurice, C., Fogh, A., Mangard, S.
**Flush+Flush: A Fast and Stealthy Cache Attack**.
Detection of Intrusions and Malware, and Vulnerability Assessment (DIMVA), 2016.
[:material-file-document: Paper (PDF)](https://gruss.cc/files/flushflush.pdf){ .md-button .md-button--primary }

---

## Textbooks

<span id="ref-8">**[8]**</span> Patterson, D. A., Hennessy, J. L.
**Computer Organization and Design: RISC-V Edition**.
Morgan Kaufmann, 2017.

---

## CVE Identifiers

| CVE | Vulnerability | Variant |
| --- | --- | --- |
| CVE-2017-5754 | Meltdown | Rogue Data Cache Load |
| CVE-2017-5753 | Spectre | Bounds Check Bypass (V1) |
| CVE-2017-5715 | Spectre | Branch Target Injection (V2) |

---

!!! note ""
    References [1]–[8] are cited inline throughout this documentation using footnote notation `[^N]`. Each footnote links back to this page.
