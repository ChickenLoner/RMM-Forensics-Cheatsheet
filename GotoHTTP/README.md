# GotoHTTP Forensics Cheat Sheet
[![Made by Chicken0248](https://img.shields.io/badge/Made%20by-Chicken0248-blue)](https://chicken0248.fyi/)
[![RMM](https://img.shields.io/badge/RMM-Forensics)](#)
[![GotoHTTP](https://img.shields.io/badge/GotoHTTP-forensics)](#)

## 📋 Overview
This cheat sheet summarizes key forensic artifacts related to GotoHTTP usage on Windows systems, focusing on the configuration file, SYSTEM service persistence, SuperTerminal, child process anomalies and file transfer.

This cheat sheet was made alongside [Investigating GotoHTTP on Windows: Behavior, Forensic Artifacts & Detection](https://chicken0248.fyi/research/gotohttp-forensics-windows/index.html) blog so give it a read to understand whole context

GotoHTTP is a standalone binary controlled entirely from a browser through an HTTP relay. It is unattended by design, with no prompt or consent on the victim side, and it deliberately writes very little to disk. **Its artifact footprint is minimal, so detection depends on telemetry you collected before the incident.**

---

## 🔍 Key Forensic Artifacts

| Goal | What to Look For | Where to Look |
|----|----|----|
| Identify GotoHTTP execution | Binary metadata | Original filename `GotoHTTP.exe`, Product Name and File Description both `GotoHTTP`, self-contained standalone executable with no installer and no dependencies |
| | Digital signature | Signed by `Hefei Pingbo Network Technology Co. Ltd.` |
| | Binary execution | Look for `GotoHTTP.exe` in `Sysmon` (Event ID **1**) or `Security.evtx` (Event ID **4688**) if the binary is not renamed |
| | Download source | Binary is served from `https://gotohttp.com/goto/download.12x` — Windows Defender flags and blocks it on download, so a Defender exclusion or detection-then-allow may precede execution |
| Identify configuration and IDs | `gotohttp.ini` creation | Created in the **same directory as the binary**, not a fixed path. Contains `sf`, `host` (relay server subdomain, e.g. `hk` for Hong Kong), `name` (9-digit **Computer ID**) and `tmp` (4-digit **Access Code**) |
| | Computer ID and Access Code | `name` and `tmp` values in `gotohttp.ini` — these are the credentials the operator typed into the GotoHTTP website to take control |
| Identify persistence | Service installation | Look for service `TTXN GotoHTTP Agent` in `System.evtx` (Event ID **7045**), auto-start, running as **SYSTEM** |
| | Services Registry | Look for the `TTXN GotoHTTP Agent` service key under `HKLM\SYSTEM\CurrentControlSet\Services\` |
| | Privilege escalation on launch | When run as Administrator the binary escalates to **SYSTEM** and relaunches itself with the `server` argument — look for the `server` argument in `Sysmon` (Event ID **1**) or `Security.evtx` (Event ID **4688**) |
| | Persistence removal | Closing the GotoHTTP window removes auto-start on next boot, **but the service entry remains** — a service with no running process is still evidence |
| Identify remote session activity | DNS queries | Look for `*.gotohttp.com` in DNS logs, firewall logs or network monitoring. Present whenever the agent is running or being controlled — this is the most reliable network indicator |
| Identify operator actions | Child process anomalies | System utilities spawned as **direct children of the GotoHTTP binary** via the browser shortcut panel: `regedit.exe`, `taskmgr.exe`, `cmd.exe`, `explorer.exe`, `control.exe`, `devmgmt.msc` — `Sysmon` (Event ID **1**) or `Security.evtx` (Event ID **4688**) with command-line auditing |
| | SuperTerminal usage | Look for `cmd.exe` spawning `chcp.com 65001` as a child of the GotoHTTP binary. SuperTerminal renders on the controller side only, so **no window appears on the victim screen** — this process pair is the only reliable tell |
| Identify file transfer activity | No native logging | GotoHTTP generates **no log entries and no Windows events** for file transfer — do not expect an application log |
| | USN Journal / FIM | Uploaded files are written with the correct extension but a **random unique filename**, then renamed to the intended name once transfer completes. Correlate this write-then-rename pair in the **USN Journal** or a **File Integrity Monitoring** solution |
| Identify chat usage | Nothing on disk | Chat and file-transfer notices are visible in the client message box **only while the process is running**. No chat history is persisted — once the process is terminated that evidence is gone permanently |

---

## ⚠️ Investigator Notes

- **Capture memory before killing the process.** Chat history and the in-session file transfer list exist only in memory. Terminating GotoHTTP destroys them permanently.
- **`gotohttp.ini` travels with the binary.** There is no fixed install path to sweep for. Hunt on the filename, not a directory.
- **No consent, no notification.** Absence of a user-interaction artifact does not indicate an authorised session — GotoHTTP is unattended by design and always operates that way.
- **Defender blocks it on download.** Unlike AnyDesk or TeamViewer, GotoHTTP is flagged immediately. A Defender detection or exclusion event shortly before first execution is a useful pivot.

---

## 📚 Resources
- https://chicken0248.fyi/research/gotohttp-forensics-windows/index.html
- https://gotohttp.com/
- https://lolrmm.io/tools/gotohttp
