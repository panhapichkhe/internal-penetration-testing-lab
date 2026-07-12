# Internal Penetration Testing Lab

![Platform](https://img.shields.io/badge/Platform-VirtualBox-blue)
![Attacker](https://img.shields.io/badge/Attacker-Kali%20Linux-red)
![Target](https://img.shields.io/badge/Target-Metasploitable%202-orange)
![Assessment](https://img.shields.io/badge/Assessment-Internal%20Pentest-green)

## Overview

This project documents a controlled internal penetration test performed against a deliberately vulnerable Linux virtual machine.

The objective was to simulate a basic penetration-testing workflow, including:

- Lab environment setup
- Network and service enumeration
- Vulnerability identification
- Manual validation
- Impact analysis
- Risk classification
- Remediation recommendations

The assessment was conducted entirely within an isolated VirtualBox host-only network.

---

## Lab Environment

| Component | Details |
|---|---|
| Attacker Machine | Kali Linux |
| Target Machine | Metasploitable 2 |
| Virtualization | Oracle VirtualBox |
| Network Type | Host-only Adapter |
| Target IP | `192.168.56.3` |
| Assessment Type | Internal Network Penetration Test |

---

## Scope

The assessment was limited to the following authorized target:

```text
192.168.56.3
