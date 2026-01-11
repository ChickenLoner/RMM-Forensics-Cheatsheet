# AnyDesk Forensics Cheat Sheet

Made by [@Chicken0248](https://chickenloner.github.io/)

## Overview
This cheat sheet summarizes key forensic artifacts related to AnyDesk usage on Windows systems, focusing on installation, connections, authentication methods, file transfer and chat log.

This cheat sheet was made alongside [Deep dive into AnyDesk Investigation & Forensics on Windows](https://medium.com/@chaoskist/deep-dive-into-anydesk-investigation-forensics-on-windows-24dc531bcc78) blog so give it a read to understand whole context

---

## Key Forensic Artifacts

| Goal | What to Look For | Where to Look |
|----|----|----|
| Identify AnyDesk installation | Service installation (`AnyDesk Service`) | Look for `AnyDesk` service install in `System.evtx` (Event ID **7045**) |
| | Service installation CLI | Look for `--install`, `--start-with-win` and `--silent` in `Sysmon` (Event ID **1**) or `Security.evtx` (Event ID **4688**) |
| | Program files created | Look for `C:\Program Files (x86)\AnyDesk\` (default location, can be changed) but `AnyDesk.exe` will be there |
| | Services Registry | Look for `HKLM\SYSTEM\CurrentControlSet\Services\AnyDesk` registry key |
| Determine execution without installation (portable) | AnyDesk binary execution | Look for `AnyDesk.exe` in `Sysmon` (Event ID **1**) and `Security.evtx` (Event ID **4688**) if binary is not renamed |
| | File creation | Look for `gcaapi.dll` which should be located in the same folder as `AnyDesk.exe` or other suspicious binary which signed by `AnyDesk Software GmbH`
| | AnyDesk folder creation | `C:\Users\<username>\AppData\Roaming\AnyDesk` |
| Identify AnyDesk ID (local system) | Local AnyDesk ID in configuration file | `ad.anynet.id` value in `service.conf` which are located in `C:\ProgramData\AnyDesk\` and `C:\Users\<username>\AppData\Roaming\AnyDesk\` |
| | Local AnyDesk ID in AnyDesk log file | Look for `main - * Client ID` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` |
| Identify remote AnyDesk ID | Remote AnyDesk ID in Connection log file | `connection_trace.txt` in `C:\ProgramData\AnyDesk\` or `C:\Users\<username>\AppData\Roaming\AnyDesk\` |
| | Remote AnyDesk ID in AnyDesk log file | Look for `anynet.any_socket - Accept request from` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` | 
| | | Look for `anynet.any_socket - Accepting from` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` | 
| | | Look for `anynet.any_socket - Client-ID:` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` | 
| Identify authentication method | Authentication type (`User`, `Passwd`, `Token`) | Look for `connection_trace.txt` file in `C:\ProgramData\AnyDesk\` or `C:\Users\<username>\AppData\Roaming\AnyDesk\`|
| Identify connection timestamps | Session start time (UTC) | Look for `connection_trace.txt` file (format `YYYY-MM-DD HH:MM` in UTC) |
| | | Look for `anynet.connection_mgr - Making a new connection to client` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` |
| | | Look for `anynet.any_socket - Connect request accepted` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace`|
| | | Look for `anynet.managed_client_conn - Connection established` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` |
| | Session end time (UTC) | Look for `anynet.any_socket - Received quit.` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` |
| | | Look for `app.session - Session closed` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` |
| Identify remote IP address | Incoming request and session establishment in AnyDesk Log File | Look for `anynet.punch_connector - -> Spawning:` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` |
| | | Look for `Incoming connection.` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` |
| Identify file transfer activity | File transfer events | Look for `file_transfer_trace.txt` file in `C:\ProgramData\AnyDesk\` and `C:\Users\<username>\AppData\Roaming\AnyDesk\` |
| | Upload to local (appear as download) event in AnyDesk log file | Look for `app.local_file_transfer - Download` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` |
| | Download from local (appear as upload) events in AnyDesk log file | Look for `app.prepare_task - Preparing files in` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` | 
| | | Look for `app.local_file_transfer - Preparation of` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` | 
| | Delete file from local via file manager events in AnyDesk log file | Look for `app.deleter` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` | 
| Identify chat usage | Chat log file creation | Chat log is named after remote AnyDesk ID in `C:\Users\<username>\AppData\Roaming\AnyDesk\chat` folder |
| Identify persistence via unattended access | Setup password for unattended access in command line | Look for `--set-password` as an argument in `Sysmon` (Event ID **1**) and `Security.evtx` (Event ID **4688**) |
| | | Look for `--set-password` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` | 
| | Unattended access events in AnyDesk log file | Look for `Authenticated with correct passphrase of profile '_unattended_access'` in `C:\Users\<username>\AppData\Roaming\AnyDesk\ad.trace` or `C:\ProgramData\AnyDesk\ad_svc.trace` |

---

## Resources
- https://www.inversecos.com/2021/02/forensic-analysis-of-anydesk-logs.html
- https://medium.com/@nandakumarp/anydesk-forensics-63b43200f8ae
- https://medium.com/@tylerbrozek/anydesk-forensics-anydesk-log-analysis-b77ea37b90f1
- https://support.anydesk.com/docs/command-line-interface-for-windows
- https://github.com/AiGptCode/ANYDESK-BACKDOOR/blob/main/Anydesk-backdoor.ps1
- https://www.nccgroup.com/research-blog/the-dark-side-how-threat-actors-leverage-anydesk-for-malicious-activities/
- https://www.cybertriage.com/blog/dfir-next-steps-suspicious-anydesk-use/
- https://www.thedfirspot.com/post/anydesk-investigating-threat-actors-favorite-tool
- https://jsac.jpcert.or.jp/archive/2023/pdf/JSAC2023_1_1_yamashige-nakatani-tanaka_en.pdf
