# RMM Forensics Cheat Sheet

[![Made by Chicken0248](https://img.shields.io/badge/Made%20by-Chicken0248-blue)](https://chicken0248.fyi/)
[![RMM](https://img.shields.io/badge/RMM-Forensics)](#)

## 📋 Overview

This repository contains forensic artifacts and indicators for Remote Monitoring and Management (RMM) tools commonly observed in threat actor operations. These cheat sheets are compiled from hands-on research in controlled lab environments to support defensive security operations.

**Every artifact here was reproduced in a lab, not collected from vendor documentation.** Each cheat sheet is written alongside a full research writeup that shows the captures the artifacts came from.

## 🗂️ Coverage

| Tool | Cheat Sheet | Research Writeup |
|---|---|---|
| Action1 | [Action1/](Action1/) | [Action1 RMM Forensics Artifacts for Windows](https://chicken0248.fyi/research/action1-forensics-windows/index.html) |
| AnyDesk | [AnyDesk/](AnyDesk/) | [Deep dive into AnyDesk Investigation & Forensics on Windows](https://chicken0248.fyi/research/anydesk-forensics-windows/index.html) |
| Chrome Remote Desktop | [CRD/](CRD/) | [Chrome Remote Desktop RMM Investigation (Windows)](https://chicken0248.fyi/research/chrome-remote-desktop-forensics/index.html) |
| GotoHTTP | [GotoHTTP/](GotoHTTP/) | [Investigating GotoHTTP on Windows: Behavior, Forensic Artifacts & Detection](https://chicken0248.fyi/research/gotohttp-forensics-windows/index.html) |
| RustDesk | [RustDesk/](RustDesk/) | [Deep dive into RustDesk RMM Investigation & Forensics on Windows](https://chicken0248.fyi/research/rustdesk-forensics-windows/index.html) |

RustDesk also ships a [KAPE target](RustDesk/RustDesk.tkape) for collection.

## 🎯 Purpose

Remote Monitoring and Management tools have become increasingly popular in both legitimate IT operations and malicious campaigns. Threat actors frequently leverage RMM tools for:

- Initial access and persistence
- Command and control (C2) communications
- Lateral movement
- Data exfiltration
- Ransomware deployment

This collection aims to help blue teams identify, investigate, and respond to suspicious RMM tool usage in their environments.

## 🔭 Scope, and where to go for breadth

This repository goes deep on a small number of tools: host artifacts, log strings, process chains, and what survives an uninstall.

For breadth across the RMM landscape, use [LOLRMM](https://lolrmm.io/), which catalogues hundreds of tools with network indicators and detection rules. The two are complementary — LOLRMM tells you which artifacts exist for a tool, these cheat sheets tell you what the artifact actually looks like when you have the disk image open.

## 📚 Resources

- [Author's Website](https://chicken0248.fyi/)
- Report issues or suggestions via GitHub Issues

## 📝 License

This project is provided as-is for educational and defensive security purposes.

---

**Note**: This repository is actively maintained and updated as new RMM tool research finally conducted by me.
