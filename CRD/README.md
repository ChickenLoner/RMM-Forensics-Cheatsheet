# Chrome Remote Desktop Forensics Cheat Sheet
[![Made by Chicken0248](https://img.shields.io/badge/Made%20by-Chicken0248-blue)](https://chickenloner.github.io/)
[![RMM](https://img.shields.io/badge/RMM-Forensics)](#)
[![AnyDesk](https://img.shields.io/badge/CRD-forensics)](#)

## 📋 Overview
This cheat sheet summarizes key forensic artifacts related to Chrome Remote Desktop (CRD) usage on Windows systems, focusing on installation and its connections.

This cheat sheet was made alongside [Chrome Remote Desktop RMM Investigation (Windows)](https://medium.com/@chaoskist/chrome-remote-desktop-rmm-investigation-c61f8545da26) blog so give it a read to understand whole context

---

## 🔍 Key Forensic Artifacts

| Goal | What to Look For | Where to Look |
|----|----|----|
| Identify CRD installation | Service installation (`Chrome Remote Desktop Service`) | Look for `Chrome Remote Desktop Service` service install in `System.evtx` (Event ID **7045**) |
| | Service installation CLI | Look for `remoting_host.exe` in `Sysmon` (Event ID **1**) or `Security.evtx` (Event ID **4688**) |
| | Program files folder | Look for `C:\Program Files (x86)\Google\Chrome Remote Desktop` (default location) |
| | hosts file folder creation | `C:\ProgramData\Google\Chrome Remote Desktop\host.json` (if its an unattended access, it should have gmail of the threat actor there) |
| Identify any connection of CRD | Session start time | Look for Event ID **1** and **4** in `Application.evtx` (chromoting) |
| | Session stop time | Look for Event ID **2** in `Application.evtx` (chromoting) |
| Identifying file that was uploaded with CRD | File activity event | Look for any file that was created as `filename.ext.part` which later renamed to `filename.ext` once completed |
| Identify persistence via unattended access | Service changing start type from manual to start |  Look for Event ID **7040** of `Chrome Remote Desktop Service` in `Security.evtx`  |
| Identify is user ever generated access code for CRD | host start up event from chromoting | Look for Event ID **5** in `Application.evtx` (chromoting) and you should be able to see gmail address of the user who generated the access code here |

---
