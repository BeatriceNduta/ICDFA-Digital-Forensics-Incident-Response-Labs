# Windows Command Line

# CIP-B101 Lab 1C: Windows Command Line for Digital Forensics Basics

| Field | Details |
|---|---|
| Student Name | Beatrice Nduta |
| Student ID | C11/26/PCDFIR/17016 |
| Course Name | Digital Forensics & Incident Response |
| Course Code / Title | CIP-B101 Basic Computer Skills for Digital Forensics |
| Instructor Name | Basiru Aliyu |
| Date Completed | 11-12/11/2026 |

----------------------------------------------------------------------------------------------------

## Overview

This lab builds the Windows command-line habits a trainee digital forensic analyst needs for basic triage: navigating paths, creating and managing evidence folders, collecting system and network information, searching file contents, reviewing permissions, listing processes, and automating collection with batch files. Every command was run on my own Windows machine, inside a lab-only workspace, and failed commands are documented rather than hidden.

**Lab workspace and subfolders created in Command Prompt**

<img width="800" alt="Fig 1 Lab workspace created" src="screenshots/fig01_workspace_created.png" />

## Lab Environment

| Item | Details |
| --- | --- |
| Operating system | Microsoft Windows 11 Pro (build 26200, x64-based PC) |
| Processor | Intel Core i7-1255U |
| Memory | About 7.8 GB RAM |
| Storage | 512 GB solid-state drive |
| Workspace | `C:\Users\Public\CIP-B101_Lab1C_Beatrice_Nduta` |

> **Privacy note:** IP addresses and gateway details are partly redacted in the screenshots.

## Tools Used

| Tool | Purpose |
| --- | --- |
| `cd`, `dir`, `tree` | Navigation and folder structure |
| `echo`, `type`, `copy`, `xcopy`, `ren`, `del`, `rmdir` | File and folder management |
| `echo %PATH%`, `where`, `vol` | PATH and drive information |
| `systeminfo`, PowerShell `Get-CimInstance` | Host, CPU and disk information |
| `ipconfig`, `getmac`, `ping` | Network information and connectivity |
| `findstr` | Searching text evidence |
| `dir /q`, `icacls` | Ownership and permissions |
| `tasklist` | Process listing |
| Notepad, batch files (`.bat`) | Repeatable collection scripts |

## Workspace Structure

```
CIP-B101_Lab1C_Beatrice_Nduta/
├── evidence/        # mini_case_log.txt
├── screenshots/
├── outputs/         # systeminfo, cpu, disk, ipconfig, MAC, process and sys_info reports
├── scripts/         # hello_world.bat, sys_info.bat
├── workspace/       # my_first_file.txt, evidence_note.txt
├── workspace2/      # copied and renamed note
├── workspace3/      # xcopy backup of workspace
└── workspace4/      # copy of sys_info.bat used for the PATH task
```

## Task 1: Windows Paths and Navigation

```bat
cd
echo Current directory is: %cd%
cd \
dir
cd \Users
cd Public
cd ..
cd Public\CIP-B101_Lab1C_Beatrice_Nduta
tree
```

**1.1 Current directory**

<img width="800" alt="Fig 2 Current directory" src="screenshots/fig02_current_directory.png" />

**1.2 Windows root directory**

<img width="600" alt="Fig 3 Windows root" src="screenshots/fig03_windows_root.png" />

**1.3 Users and Public folders**

<img width="700" alt="Fig 4 Users and Public" src="screenshots/fig04_users_public.png" />

**1.4 Moving back with a relative path**

<img width="600" alt="Fig 5 Relative path" src="screenshots/fig05_relative_path.png" />

**1.5 Folder structure with tree**

<img width="700" alt="Fig 6 Tree structure" src="screenshots/fig06_tree_structure.png" />

| Term | Meaning | Example from this lab |
| --- | --- | --- |
| Absolute path | Full address starting at the drive root; works from any location | `cd \Users\Public\CIP-B101_Lab1C_Beatrice_Nduta` |
| Relative path | Location described from the current folder; shorter but depends on the starting folder | `cd ..` moves one level up; `cd Public` moves into a folder below |

An analyst must know exactly where evidence sits. Relative paths speed up navigation, while absolute paths remove any doubt about location.

## Task 2: Basic File and Folder Creation

```bat
echo hello world > my_first_file.txt
type my_first_file.txt
echo Student Name: Beatrice Nduta > evidence_note.txt
echo Course: CIP-B101 Lab 1C >> evidence_note.txt
echo Purpose: Windows command line forensic basics >> evidence_note.txt
type evidence_note.txt
dir
```

**Creating my_first_file.txt**

<img width="800" alt="Fig 7 my_first_file" src="screenshots/fig07_my_first_file.png" />

**Creating evidence_note.txt with > and >>**

<img width="800" alt="Fig 8 evidence_note" src="screenshots/fig08_evidence_note.png" />

**Listing the workspace folder**

<img width="700" alt="Fig 9 dir workspace" src="screenshots/fig09_dir_workspace.png" />

| Operator | Behaviour | How I used it |
| --- | --- | --- |
| `>` | Overwrites: sends output to a file and replaces existing content | First line, to start `evidence_note.txt` cleanly |
| `>>` | Appends: adds output to the end and keeps existing content | Second and third lines, so earlier lines were not lost |

## Task 3: Copy, Rename and Delete Files Safely

**3.1 Copy a file to another folder** (`copy evidence_note.txt ..\workspace2\evidence_note_copy.txt`)

<img width="800" alt="Fig 10 copy file" src="screenshots/fig10_copy_file.png" />

**3.2 Back up a whole folder** (`xcopy workspace workspace3\workspace_backup /E /I /H`)

<img width="800" alt="Fig 11 xcopy backup" src="screenshots/fig11_xcopy_backup.png" />

**3.3 Rename the copied file** (`ren evidence_note_copy.txt renamed_evidence_note.txt`)

<img width="800" alt="Fig 12 rename file" src="screenshots/fig12_rename_file.png" />

**3.4 Create a temporary test file**

<img width="700" alt="Fig 13 delete test before" src="screenshots/fig13_delete_test_before.png" />

**3.4 After `del delete_me.txt`**

<img width="700" alt="Fig 14 delete test after" src="screenshots/fig14_delete_test_after.png" />

**3.5 Create an empty folder**

<img width="700" alt="Fig 15 mkdir empty folder" src="screenshots/fig15_mkdir_empty_folder.png" />

**3.5 After `rmdir empty_test_folder`**

<img width="700" alt="Fig 16 rmdir empty folder" src="screenshots/fig16_rmdir_empty_folder.png" />

Each step was verified with `dir` or `tree` before and after. Only the temporary test file and the empty test folder were removed, and the evidence folders were left intact.

## Task 4: How Windows Finds Applications (PATH)

```bat
echo %PATH%
where notepad
notepad
vol C:
```

**PATH variable and the location of Notepad**

<img width="900" alt="Fig 17 PATH and where notepad" src="screenshots/fig17_path_where_notepad.png" />

**Drive information with vol C:**

<img width="600" alt="Fig 18 vol C" src="screenshots/fig18_vol_c.png" />

**Why PATH matters in forensics:** Windows searches the PATH folders, in order, when a command name is typed. If a malicious program is placed in a PATH folder, or a PATH entry is altered, it can run in place of a trusted tool. Reviewing PATH is therefore part of malware triage and system review.

## Task 5: System Information

```bat
systeminfo > systeminfo_full.txt
systeminfo | find "OS Name"
systeminfo | find "OS Version"
systeminfo | find "System Type"
systeminfo | find "Total Physical Memory"
```

**OS, system type, memory, CPU and disk information**

<img width="800" alt="Fig 19 System information" src="screenshots/fig19_system_info.png" />

| Item | Finding |
| --- | --- |
| Operating system | Microsoft Windows 11 Pro, build 26200 |
| System type | x64-based PC |
| Processor | Intel Core i7-1255U |
| Memory | About 7.8 GB |
| Disk | 512 GB solid-state drive |

> **Failed command:** `wmic` is not recognised on this version of Windows because Microsoft has removed it, so `cpu_info.txt` and `disk_info.txt` were first created empty (0 bytes). I regenerated them with the supported PowerShell command `Get-CimInstance` (`Win32_Processor` and `Win32_DiskDrive`).

## Task 6: Network Information

```bat
ipconfig /all > ipconfig_all.txt
ipconfig | findstr IPv4
ipconfig | findstr Gateway
getmac /v > mac_addresses.txt
ping 127.0.0.1
ping 8.8.8.8
ping google.com
```

**IPv4 address, gateway, MAC address and ping results**

<img width="900" alt="Fig 20 Network information" src="screenshots/fig20_network_info.png" />

- `ipconfig /all` was saved to `ipconfig_all.txt`, and `ipconfig | findstr` returned the IPv4 addresses and default gateway.
- The loopback ping to `127.0.0.1` succeeded with 0% loss, confirming the TCP/IP stack is working.
- **Failed command:** `wmic nic get macaddress,description` failed because `wmic` is unavailable. I used `getmac /v` instead.
- **If an internet ping fails:** likely causes are no internet connection, a DNS problem (a name such as `google.com` does not resolve), or an incorrect IP or gateway setting. A ping to `8.8.8.8` succeeding while `google.com` fails would point to DNS.

## Task 7: Searching for Evidence Text

```bat
echo User: Beatrice Nduta > mini_case_log.txt
echo Email: trainee@example.com >> mini_case_log.txt
echo IP: 192.168.56.10 >> mini_case_log.txt
echo Keyword: suspicious_login >> mini_case_log.txt
type mini_case_log.txt
findstr /i "email" mini_case_log.txt
findstr /i "suspicious" mini_case_log.txt
findstr /n /i "ip" mini_case_log.txt
```

**Creating mini_case_log.txt and searching it with findstr**

<img width="900" alt="Fig 21 findstr search" src="screenshots/fig21_findstr_search.png" />

| Switch | Function |
| --- | --- |
| `/i` | Case-insensitive search, so `email` matches `Email` |
| `/n` | Prints the line number before each match, helping an analyst point to the exact location of a hit |

## Task 8: File Owner and Permissions

```bat
dir /q
icacls mini_case_log.txt
```

**Ownership and permission output**

<img width="800" alt="Fig 22 icacls permissions" src="screenshots/fig22_icacls_permissions.png" />

- `icacls` shows that the `BUILTIN\Administrators` group has Full control (`F`) of `mini_case_log.txt`.
- The `(I)` flag means the permission is inherited from the parent `evidence` folder rather than set on the file itself.
- `icacls` lists permissions; the owner name is shown by `dir /q`.

## Task 9: Process Listing

**Process listing and browser process search**

<img width="900" alt="Fig 23 Process list" src="screenshots/fig23_process_list.png" />

> **Failed command:** `wmic process list brief` returned "'wmic' is not recognized", so no `process_list.txt` was produced by that command. I replaced it with the built-in `tasklist` command, for example:
>
> ```bat
> tasklist > process_list.txt
> tasklist /fi "imagename eq chrome.exe"
> tasklist /fi "imagename eq msedge.exe"
> ```

A process list is a snapshot of what was running. Analysts compare it against what is expected to spot unusual or suspicious programs.

## Task 10: Batch Files

### 10.1 hello_world.bat

```bat
@ECHO OFF
ECHO Hello World!
PAUSE
```

**Running hello_world.bat**

<img width="600" alt="Fig 24 hello_world.bat" src="screenshots/fig24_hello_world_bat.png" />

My first attempt typed the lines directly into Command Prompt instead of saving them in a file. I corrected this by creating the file with Notepad and running `hello_world.bat`.

### 10.2 sys_info.bat

The script collects the Windows version, system type, memory, CPU, disk, IPv4 and MAC address information and writes it to `outputs\sys_info_report.txt` under three headings: **WINDOWS INFO**, **HARDWARE INFO** and **NETWORK INFO**.

```bat
@ECHO OFF
:: CIP-B101 Lab 1C - Windows system information collection
TITLE CIP-B101 System Info Collection
ECHO Please wait... Checking system information.
ECHO ========================== > ..\outputs\sys_info_report.txt
ECHO WINDOWS INFO >> ..\outputs\sys_info_report.txt
...
ECHO Collection complete. See outputs\sys_info_report.txt
PAUSE
```

In the original script the three `wmic` lines failed, so I replaced them with PowerShell commands (`Get-CimInstance` and `Get-NetAdapter`). The script then ran without errors and produced all three report sections.

**Launching sys_info.bat**

<img width="600" alt="Fig 25 sys_info.bat run 1" src="screenshots/fig25_sys_info_bat_run_1.png" />

**sys_info.bat execution**

<img width="900" alt="Fig 26 sys_info.bat run 2" src="screenshots/fig26_sys_info_bat_run_2.png" />

**Generated sys_info_report.txt**

<img width="800" alt="Fig 27 sys_info_report" src="screenshots/fig27_sys_info_report.png" />

## Task 11: Making sys_info.bat Callable from Any Folder

```bat
cd \Users\Public\CIP-B101_Lab1C_Beatrice_Nduta
copy scripts\sys_info.bat workspace4\sys_info.bat
dir workspace4
```

**Copying sys_info.bat into workspace4**

<img width="800" alt="Fig 28 copy to workspace4" src="screenshots/fig28_copy_to_workspace4.png" />

```bat
set PATH=%PATH%;C:\Users\Public\CIP-B101_Lab1C_Beatrice_Nduta\workspace4
cd \
sys_info.bat
```

**Running sys_info.bat from C:\ after the PATH update**

<img width="491" alt="Fig 29 PATH run from root" src="screenshots/fig29_path_run_from_root.png" />

- **PATH method used: temporary (Option A).** `set PATH=...` applies to the current Command Prompt window only and disappears when it is closed. I did not use `setx`, so no permanent change was made.
- **Proof:** from `C:\`, Windows located `sys_info.bat` through PATH, and the script ran from outside its own folder.
- **Issue found and fixed:** the original script used the relative path `..\outputs\`, which fails when the script is run from `C:\` because `..` no longer points to the lab folder. I changed every occurrence to the absolute path `C:\Users\Public\CIP-B101_Lab1C_Beatrice_Nduta\outputs\`.

## Task 12: Mini Forensic Interpretation Questions

**1. What is the difference between an absolute path and a relative path?**
An absolute path gives the full location starting from the drive root, so it works from any folder. A relative path is interpreted from the current folder, so it can break if the command is run from somewhere else.
- Absolute example: `C:\Users\Public\CIP-B101_Lab1C_Beatrice_Nduta\outputs\sys_info_report.txt`
- Relative example: `..\outputs\sys_info_report.txt`, which failed when I ran the script from `C:\`

**2. Why create a separate lab/evidence folder before running commands?**
It keeps all output files in one known place, so nothing is scattered, overwritten or mixed with other files. It also protects other people's data from accidental deletion and makes the evidence easy to find, review and submit.

**3. Which command showed the IP address and which showed the MAC address?**
The IP address was shown by `ipconfig` (`ipconfig /all` and `ipconfig | findstr IPv4`). The MAC address was shown by `getmac /v`, and also appears as "Physical Address" in `ipconfig /all`. The original `wmic nic get macaddress` command failed on this version of Windows.

**4. Why is a batch file useful in forensic triage?**
It runs the same commands in the same order every time, making collection fast, consistent and repeatable. It reduces typing errors on a live system and writes results to a report file automatically, so another analyst can run it and get comparable results.

**5. Why document failed commands or pings instead of hiding them?**
A failed command is part of the record. Documenting it shows exactly what was attempted and what happened, which keeps the work honest and reproducible. It also explains gaps in the evidence (for example, `wmic` failing explains the empty output files), and a failure can itself be a finding, such as a blocked network or a disabled service.

**6. What risk occurs if a suspicious executable is placed in a PATH directory?**
Windows searches PATH folders when a command name is typed, so a malicious program placed there could run instead of, or before, the legitimate one (PATH hijacking). It would run with the permissions of whoever typed the command, which can lead to malware execution or persistence. PATH should therefore contain only trusted folders, and permanent PATH changes need approval.

## Challenges and Resolution

| Issue | Resolution |
| --- | --- |
| `wmic` is not recognised on this version of Windows (CPU, disk, MAC and process commands) | Used `Get-CimInstance`, `Get-NetAdapter`, `getmac /v` and `tasklist` instead, and documented the failures |
| `cpu_info.txt` and `disk_info.txt` were created empty (0 bytes) | Regenerated them with PowerShell |
| `hello_world.bat` lines were typed directly into Command Prompt | Created the file with Notepad and ran it |
| `sys_info.bat` failed from `C:\` because of the relative path `..\outputs\` | Changed every occurrence to an absolute path |

## Reflection

The Windows command line lets an analyst collect system, network, file and process information quickly and repeatably, and a batch file turns those steps into a consistent routine. This lab also showed that commands change between Windows versions (`wmic` no longer exists), that relative paths can silently break a script, and that PATH is both a convenience and a security risk. Documenting failures, verifying each step, and working only inside a lab folder on my own machine are the habits that make a command-line record trustworthy as evidence.

## References

- [Windows 10 system requirements (Microsoft Support)](https://support.microsoft.com/en-us/windows/windows-10-system-requirements-6d4e9a79-66bf-7950-467c-795cf0386715)
