# Action1 Forensics Cheat Sheet
[![Made by Chicken0248](https://img.shields.io/badge/Made%20by-Chicken0248-blue)](https://chicken0248.fyi/)
[![RMM](https://img.shields.io/badge/RMM-Forensics)](#)
[![Action1](https://img.shields.io/badge/Action1-forensics)](#)

## 📋 Overview
This cheat sheet summarizes key forensic artifacts related to Action1 usage on Windows systems, focusing on installation, remote desktop (VNC), Run Script, Deploy Software with its P2P distribution, and what survives an agent uninstall.

This cheat sheet was made alongside [Action1 RMM Forensics Artifacts for Windows](https://chicken0248.fyi/research/action1-forensics-windows/index.html) blog so give it a read to understand whole context

Action1 installs into `C:\Windows\Action1\` and runs as **SYSTEM**. Its logging is unusually verbose, which makes it one of the friendlier RMM tools to investigate — **provided you preserve the folder before anyone runs the uninstall.**

---

## 🔍 Key Forensic Artifacts

| Goal | What to Look For | Where to Look |
|----|----|----|
| Identify Action1 installation | Install folder | `C:\Windows\Action1\` — installing into `C:\Windows\` is unusual on its own and a strong signal |
| | Service installation | Look for service `A1Agent` (display name **"Action1 Agent"**) in `System.evtx` (Event ID **7045**), auto-start, SYSTEM, image path `C:\Windows\Action1\action1_agent.exe service` |
| | Services Registry | `HKLM\SYSTEM\CurrentControlSet\Services\A1Agent` |
| | Agent config Registry | `HKLM\SOFTWARE\WOW6432Node\Action1` — certificate, public key, **customer/org ID**, MSI package ID, install path, agent GUID (`agent.guid`), hardware ID (`system.id`), disk serial |
| | MSI install events | `Application` log, **MsiInstaller** Event IDs **1040** (transaction begin, full MSI path + first PID), **1042** (transaction end), **11707** (product install completed), **1033** (install success, product "Action1 Agent", manufacturer "Action1 Corporation") |
| | Installer file | `action1_agent(<ORG>).msi` — **the filename embeds the deploying org name**. Signed by Sectigo, signer `Action1 Corporation`, product code `{E6D8B066-2E74-458F-9111-24857C1C627A}` |
| | Binaries | `action1_agent.exe` (always-running agent, executes every command), `action1_remote.exe` (TightVNC-based remote server, only runs during remote control), plus `7z.dll` and `sas.dll` |
| Identify who deployed it | Marker files | `previousUser.txt` (installing admin in `WORKSTATION\user` format), `first_install` (first install timestamp in **UTC**), `what_is_this.txt` (org ID, plus Action1's own note for legal contact) |
| | Customer ID and agent ID | Both appear in `C:\Windows\Action1\logs\*.log` — the **Customer ID** matches `what_is_this.txt`, the **service/agent ID** identifies the deploying org. Preserve these for takedown or legal action |
| Identify network activity | Install-time pull | `104.18.38.233` — Cloudflare (AS13335). Do not blanket-block this range |
| | Persistent C2 channel | `server.<region>.action1.com` (AWS, region follows the operator's tenant) over **443** plus high port **22543** outbound |
| | Remote session relay | `relay.<region>.action1.com` (443) — only present during a remote desktop session |
| | Deploy payload CDN | `eu-cdn.action1.com` (BunnyCDN web-seed), 443 |
| | LAN P2P distribution | Inbound **22551** (TCP/UDP peer transfer) and **6771/udp** (BitTorrent Local Peer Discovery, multicast `239.192.152.143`) tied to `action1_agent.exe` |
| Identify remote desktop (VNC) | Display enumeration | `action1_agent.exe monitorcount` — enumerates displays via WMI `Win32_DesktopMonitor`, re-runs roughly every **60 seconds** during a session |
| | Session process chain | `action1_remote.exe -controlservice -slave` → `queryconnection -peer 127.0.0.1 -accept -timeout 15` (consent prompt, **auto-accepts on timeout**) → `desktopserver -logdir "C:\Windows\Action1\logs" -loglevel 0 -shmemname Global\<id>` (screen goes live) |
| | Session start | `REMOTE_SESSION_CONNECT` in `C:\Windows\Action1\logs\*.log` |
| | Session end | `REMOTE_SESSION_CLOSE` in the log, and `action1_remote.exe -controlservice -shutdown` in the process view. Log teardown begins with `Relay closed connection, closing the session.` |
| | Heartbeat log | Search `HEARTBEAT_ACK` — the heartbeat log is also the largest file by size |
| | Timeline accuracy | `REMOTE_SESSION_CONNECT` is when the **transport** came up; `desktopserver` spawning (~25s later) is when the operator could actually **see the screen**. The gap is the consent auto-accept plus spawn latency — pair log with process for a defensible timeline |
| | Consent prompt is not proof of consent | Default **15 seconds**, configurable **1–180 seconds**, and it **auto-accepts on timeout**. An operator can set 1 second so the prompt barely blinks. Prompt text and logo are customizable too |
| | Visual tell | Endpoint wallpaper is swapped to a plain dark background by default during a session |
| Identify Run Script execution | Automation process | `action1_agent.exe schedule:<automation>_<unixts> runaction:N` — the **automation name and UNIX timestamp are readable straight off the command line** |
| | Command Prompt case | Temp `.cmd` script dropped in `C:\Windows\Action1\scripts\`, executed via `cmd.exe` as a child of `action1_agent.exe`, then **deleted immediately after execution**. Output file also deleted once uploaded |
| | PowerShell case | **No `.cmd` script** — `action1_agent.exe` runs commands through `powershell.exe` directly. External binaries (`whoami`, `hostname`) appear as children of `powershell.exe` |
| | What actually persists | Dedicated per-run log in `C:\Windows\Action1\logs\` (start time, schedule ID, temp script, the command the operator ran, and the result) and `C:\Windows\Action1\batch_data\<automation>_<unixts>.data` |
| | Script Library abuse | Built-in templates include **"Enable Local Admin"** and **"Turn Off Windows Firewall"** (no parameters), plus "Local Password Reset" and "Download from the Internet". Operators can add their own — this is the documented path for dropping a second RMM |
| Identify Deploy Software | Deployment process | `action1_agent.exe schedule:` with the **installer running as a child process** |
| | Temp automation files | `C:\Windows\Action1\running_batches\` — deleted as soon as the automation finishes |
| | Payload remnants | `C:\Windows\Action1\package_downloads\` — the installer plus `.json` (filename + **MD5**), `.result` (`OK` on success), `.metadata` (bencode torrent), `.resume` (libtorrent state) |
| | Per-run logs | **Three** separate logs per deploy: check-existing-software → check-requirements → install |
| | Firewall fingerprint | Three inbound Allow rules for `action1_agent.exe`: `Action1 Agent (TCP-In)` and `Action1 Agent (UDP-In)` on **22551**, `Action1 Agent LPD (UDP-In)` on **6771** |
| | Who wrote the firewall rules | Added by the agent through the Windows Firewall COM API (`INetFwPolicy2`) and written by **MpsSvc (`svchost.exe`)**, not by the MSI and not by `netsh`. Added on **first deploy**, not at install |
| | Catching the rule creation | Use the **Windows Firewall operational log (Event 2004,** "a rule was added"**)** or outbound 6771 multicast to `239.192.152.143`. A stock SwiftOnSecurity Sysmon config will **not** log it |
| Identify agent removal | Uninstall command | PowerShell spawns `msiexec /X "{E6D8B066-2E74-458F-9111-24857C1C627A}" /quiet /qn /norestart /lv* "C:\Windows\Action1\logs\agent_install_<UNIXTIMESTAMP>.log" ALLOWRESTRICTEDVERSIONS=1`, which calls `action1_agent.exe uninstall-msi` for cleanup |
| | Uninstall log naming trap | A file named `agent_install_<ts>.log` is the **uninstall** log — `/lv*` set that path. Do not let the name mislead the timeline |
| | Removal events | Same `Application`/MsiInstaller event chain as the install, but the details describe a **package removal** |
| | What is deleted | `A1Agent` service, WER registry values, the `HKLM\SOFTWARE\WOW6432Node\Action1` key, and everything under `C:\Windows\Action1\` **except `logs` and `CrashDumps`** — including the whole `package_downloads` payload |
| | What survives | `C:\Windows\Action1\logs\` and `CrashDumps\`. The logs still carry the **Customer ID, agent ID, the names of software deployed via Action1, and the removal timeline** |
| | Tombstone artifact | The three `Action1 Agent*` inbound firewall rules **outlive the agent** — the MSI never registered them, so Windows only prunes them once `action1_agent.exe` is actually deleted. A firewall rule pointing at a missing `action1_agent.exe` is a clean tell of prior Action1 |

---

## ⚠️ Investigator Notes

- **Preserve `C:\Windows\Action1\` before anyone touches the uninstall.** The uninstall wipes `package_downloads`, so the deployed payload and its MD5 are gone. The log remembers the payload's *name* but not the file.
- **Grab both IDs early.** Customer ID and agent GUID are what Action1's team needs for takedown or legal action. They survive in the logs even after the registry key is removed.
- **A consent prompt was probably shown, and it means nothing.** It auto-accepts, and the timeout is operator-controlled down to 1 second.
- **Run Script and Deploy Software are the abuse paths, not remote desktop.** The remote session is plain VNC with no file transfer exposed. Expect operators to use script/deploy to install a *second* RMM, as documented in the Huntress daisy-chaining research.
- **Deploying vulnerable software is not much of an escalation path here** — the agent is already SYSTEM.

---

## 📚 Resources
- https://chicken0248.fyi/research/action1-forensics-windows/index.html
- https://www.action1.com/documentation/
- https://www.huntress.com/blog/daisy-chaining-rogue-rmm-tools
- https://github.com/tsale/Sigma_rules/blob/master/Threat%20Hunting%20Queries/Action1_RMM.yml
- https://lolrmm.io/tools/action1
