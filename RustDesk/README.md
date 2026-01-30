# RustDesk Forensics Cheat Sheet
[![Made by Chicken0248](https://img.shields.io/badge/Made%20by-Chicken0248-blue)](https://chickenloner.github.io/)
[![RMM](https://img.shields.io/badge/RMM-Forensics)](#)
[![RustDesk](https://img.shields.io/badge/RustDesk-forensics)](https://rustdesk.com/)

## 📋 Overview
This cheat sheet summarizes key forensic artifacts related to RustDesk usage on Windows systems, focusing on installation, connections, file transfer, and how RustDesk stores password and its ID in configuration file.

This cheat sheet was made alongside [Deep dive into RustDesk RMM Investigation & Forensics on Windows](https://medium.com/@chaoskist/deep-dive-into-rustdesk-rmm-investigation-forensics-on-windows-6d8ba816a11e) blog so give it a read to understand whole context

---

## 🔍 Key Forensic Artifacts

| Goal | What to Look For | Where to Look |
|----|----|----|
| Identify usage of RustDesk portable version | RustDesk default program folder | `C:\Users\<username>\AppData\Local\rustdesk`|  
| | RustDesk configuration and log folder | `C:\Users\<username>\AppData\Roaming\rustdesk` | 
| Identify RustDesk installation | Service installation (`RustDesk Service`) | Look for `RustDesk Service` service install in `System.evtx` (Event ID **7045**) |
| | Service installation CLI | Look for `--install` or `--silent-install` argument in `Sysmon` (Event ID **1**) or `Security.evtx` (Event ID **4688**) |
| | Default program folder | `C:\Program Files\RustDesk\` |
| | Services Registry | `HKLM\SYSTEM\CurrentControlSet\Services\RustDesk` registry key |
| | Startup folder | `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup\RustDesk Tray.ink` |
| | Temporary Batch script for installation | `C:\Users\<username>\AppData\Local\Temp\RustDesk_install.bat` | 
| | VBS script execution | `C:\Users\<username>\AppData\Local\Temp\RustDesk_mk_shortcut.vbs` |
| | | `C:\Users\<username>\AppData\Local\Temp\RustDesk_uninstall_shortcut.vbs` |
| | | `C:\Users\<username>\AppData\Local\Temp\RustDesk_tray_shortcut.vbs` |
| Identify connection timestamps | Sending connection request | Look for `Connection opened from` in `C:\Users\<username>\AppData\Roaming\RustDesk\log\rustdesk_rCURRENT.log` |
| | Session Establishment | Look for `new wakelock` in `C:\Users\<username>\AppData\Roaming\RustDesk\log\rustdesk_rCURRENT.log` |
| | Session end time (UTC) | Look for `drop wakelock` in `C:\Users\<username>\AppData\Roaming\RustDesk\log\rustdesk_rCURRENT.log`|
| Identify remote IP address | RustDesk Access Log File | Look for `Connection opened from` in `C:\Users\<username>\AppData\Roaming\RustDesk\log\rustdesk_rCURRENT.log`  |
| Identify file transfer activity (without file manager) | remote -> end device event in RustDesk log file | Look for `client_file_contents_request` and `server_file_contents_response` in `C:\Users\<username>\AppData\Roaming\RustDesk\log\cm\rustdesk_rCURRENT.log`|
| | end -> remote device event in RustDesk log file | Look for `server_file_content_request` and `client_file_content_response` in `C:\Users\<username>\AppData\Roaming\RustDesk\log\cm\rustdesk_rCURRENT.log` | 
| Identify file transfer activity (with file manager) | remote -> end device event in RustDesk log file |  Look for `new write File:` in `C:\Users\<username>\AppData\Roaming\RustDesk\log\cm\rustdesk_rCURRENT.log`|
| | end -> remote device event in RustDesk log file | Look for `new read File:` in `C:\Users\<username>\AppData\Roaming\RustDesk\log\rustdesk_rCURRENT.log` | 
| Identify persistence via unattended access | Password argument in command line | Look for `--password` as an argument in `Sysmon` (Event ID **1**) and `Security.evtx` (Event ID **4688**) |
| | Password setup log in Access Log file | Look for `permanent-password updated` in `C:\Windows\ServiceProfiles\LocalService\AppData\Roaming\RustDesk\log\server\RustDesk_rCurrent.log` |
| | Argument folder creation| Look for `password` folder in `C:\Users\<username>\AppData\Roaming\RustDesk\log` | 
| | Unattended access events in RustDesk log file | Look for `Connection opened from` and `new wakelock` in `C:\Windows\Service Profiles\AppData\Roaming\RustDesk\log\rustdesk_rCURRENT.log` |

* Note: If there are multiple usages of RustDesk (run multiply times), the old `rustdesk_rCURRENT.log` will be renamed to `rustdesk_rYYYY-MM-DD_HH-MM-SS.log` and the timestamp in the filename will be the launching timestamp of the latest RustDesk instance.

---

## 📚 Resources
- https://github.com/rustdesk/rustdesk
- https://www.youtube.com/watch?v=FIEcTNjFZNA
- https://news.drweb.com/show/?i=14755
- https://blackpointcyber.com/blog/chromeloader-telegram-rustdesk-tailscale-soc-incidents-blackpoint-apg/
- https://asec.ahnlab.com/en/84729/
- https://github.com/rustdesk/rustdesk/wiki/FAQ
- https://rustdesk.com/docs/en/self-host/client-configuration/?utm_source=chatgpt.com
