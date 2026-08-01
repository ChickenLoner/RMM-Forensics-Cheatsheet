# Chrome Remote Desktop Forensics Cheat Sheet
[![Made by Chicken0248](https://img.shields.io/badge/Made%20by-Chicken0248-blue)](https://chicken0248.fyi/)
[![RMM](https://img.shields.io/badge/RMM-Forensics)](#)
[![CRD](https://img.shields.io/badge/CRD-forensics)](#)

## 📋 Overview
This cheat sheet summarizes key forensic artifacts related to Chrome Remote Desktop (CRD) usage on Windows systems, focusing on installation, the difference between Remote Support and Remote Access, connections, file transfer and unattended access.

This cheat sheet was made alongside [Chrome Remote Desktop RMM Investigation (Windows)](https://chicken0248.fyi/research/chrome-remote-desktop-forensics/index.html) blog so give it a read to understand whole context

**CRD has two modes and they are not equally dangerous.** *Remote Support* needs a 12-digit code the victim generates and clicks Share on, and the remote user cannot reach the taskbar or Event Viewer. *Remote Access* is unattended, PIN-only, gives full control, and adds file transfer. Establishing which mode was used is the first question in a CRD case.

---

## 🔍 Key Forensic Artifacts

| Goal | What to Look For | Where to Look |
|----|----|----|
| Identify CRD installation | Service installation (`Chrome Remote Desktop Service`) | Look for `Chrome Remote Desktop Service` service install in `System.evtx` (Event ID **7045**) |
| | Service installation CLI | Look for `remoting_host.exe` in `Sysmon` (Event ID **1**) or `Security.evtx` (Event ID **4688**) |
| | MSI install events | `Application.evtx`, **MsiInstaller** Event ID **1040** (MSI path + install start), then **1042**, **11707**, **1033** for the completion timestamp |
| | Program files folder | Look for `C:\Program Files (x86)\Google\Chrome Remote Desktop` (default location), all binaries **signed by Google LLC** |
| | Browser extension | `C:\Users\<username>\AppData\Local\Google\Chrome\User Data\Default\Extensions\inomeogfingihgjfjlpeplalcfajhgai\` |
| | CLI installation | CLI setup produces the **same** artifacts as GUI setup (1040 / 7045 / 7040) — do not assume GUI just because the events are there |
| Identify any connection of CRD | Session start time | Look for Event ID **1** and **4** in `Application.evtx` (chromoting) |
| | Session stop time | Look for Event ID **2** in `Application.evtx` (chromoting), carries the same unique session ID as the start |
| Identify the accounts involved | Gmail of the remote user | Event ID **1** and **4** (chromoting) both contain the Gmail address used to connect |
| | Gmail of the user who generated the code | Event ID **5** (chromoting), also gives the access code generation timestamp |
| | Authorised account for unattended access | `C:\ProgramData\Google\Chrome Remote Desktop\host.json` — contains `host_id`, the assigned computer `name`, and the **Gmail authorised to access this host**. Highest-value artifact in a rogue Remote Access case |
| Determine which mode was used | Remote Support vs Remote Access | Both produce Event IDs **1, 2, 4**. **Event ID 5 is present only for Remote Support** (an access code was generated). Event ID 5 absent + `host.json` present = **unattended Remote Access** |
| Identify persistence via unattended access | Service start type change | Look for Event ID **7040** in `System.evtx` — `Chrome Remote Desktop Service` changing from **demand start (manual)** to **auto start**. This is the moment Remote Access was configured |
| | PIN requirement | Remote Access requires a PIN of at least **6 digits**; only the Gmail that set it up can connect |
| | Session survives browser close | Closing Chrome does **not** end a Remote Access session — the service instance keeps running, so absence of a browser process proves nothing |
| Identify remote session processes | Host process | At least one `remoting_host.exe` running as **SYSTEM** and **LOCAL SERVICE**, spawned from the CRD service |
| | Manual acceptance chain | `remoting_native_messaging.exe` and `remote_assistance_host.exe` spawned as children of `cmd.exe`, with `chrome.exe` as grandparent — indicates a user manually accepted a Remote Support session |
| Identify file transfer activity | Mode restriction | File transfer is available in **Remote Access only**, not Remote Support. File transfer artifacts therefore imply unattended access |
| | File activity event | Look for any file created as `filename.ext.part` which is later renamed to `filename.ext` once completed |
| | Cancelled transfer | A **cancelled download leaves the `.part` file behind** on disk — a useful indicator of an interrupted exfil or drop |
| | Process that writes the files | Every transferred file is created by `remoting_desktop.exe` running as **NT AUTHORITY\SYSTEM** |
| | No transfer logging | Uploads and downloads are **not logged in any event log or application log**. Catch uploads via file creation events (by `remoting_desktop.exe` as SYSTEM), downloads via network activity monitoring |

---

## 🧭 Quick Reference

| Activity | Log Location | Event ID / Artifact |
|---|---|---|
| Service installation | System event log | Event ID **7045** |
| MSI installation | Application event log | Event ID **1040** (then 1042, 11707, 1033) |
| Remote Access setup (auto-start) | System event log | Event ID **7040** |
| Access code generated (Remote Support only) | Application event log | Event ID **5** (chromoting) |
| Connection established | Application event log | Event ID **1** + **4** (chromoting) |
| Connection terminated | Application event log | Event ID **2** (chromoting) |
| Authorised Gmail + host info | `C:\ProgramData\Google\Chrome Remote Desktop\host.json` | `host_id`, `name`, Gmail |
| CRD browser extension | `C:\Users\<username>\AppData\Local\Google\Chrome\User Data\Default\Extensions\inomeogfingihgjfjlpeplalcfajhgai\` | Extension directory |

---

## ⚠️ Investigator Notes

- **Event ID 5 is your mode discriminator.** Present means someone generated an access code and clicked Share. Absent, with connection events and `host.json` on disk, means unattended access that needed no victim interaction at all.
- **`host.json` names the attacker.** In a rogue Remote Access case it directly identifies the Google account that set up the persistence.
- **The 12-digit access code expires in 5 minutes**, so Remote Support requires the victim to be present and cooperating at that moment — relevant when assessing social engineering.
- **CLI setup looks identical in the logs.** TrustedSec documented CLI-based CRD deployment for red team use; do not read GUI artifacts as proof of interactive setup.

---

## 📚 Resources
- https://chicken0248.fyi/research/chrome-remote-desktop-forensics/index.html
- https://trustedsec.com/blog/abusing-chrome-remote-desktop-on-red-team-operations-a-practical-guide
- https://lolrmm.io/tools/chrome_remote_desktop
