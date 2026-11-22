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

<img width="1477" height="627" alt="Fig 1 - making folder   dir" src="https://github.com/user-attachments/assets/ec24bf16-f47f-4690-a3c7-3f2986559917" />


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

<img width="957" height="150" alt="Fig 2 - current dir display" src="https://github.com/user-attachments/assets/1a9255f8-37b0-4cd4-9967-260f9892e373" />


**1.2 Windows root directory**

<img width="677" height="442" alt="Fig 3 - disk c dir display" src="https://github.com/user-attachments/assets/8e0a4a56-00ef-4a79-a609-f8d502223724" />


**1.3 Users and Public folders**

<img width="915" height="913" alt="Fig 4 user display and dir " src="https://github.com/user-attachments/assets/6499a3b0-5a1e-48b3-a409-373121998d3b" />


**1.4 Moving back with a relative path**

<img width="603" height="201" alt="Fig 5 - paths" src="https://github.com/user-attachments/assets/53da4a82-e794-4db9-980d-2e60e3e8c091" />


**1.5 Folder structure with tree**

<img width="717" height="332" alt="Fig 6 - tree to display structure" src="https://github.com/user-attachments/assets/204a676a-c0a1-4f66-9da9-8dd8269bc5cf" />


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

<img width="1561" height="298" alt="Fig 7 - text file generation" src="https://github.com/user-attachments/assets/6fba85b7-0970-473e-a6fb-a9b9b6296eae" />


**Creating evidence_note.txt with > and >>**

<img width="1152" height="155" alt="Fig 8 - file evidence note" src="https://github.com/user-attachments/assets/9dce293c-75e7-4287-95fa-e0da3c7c7324" />


**Listing the workspace folder**

<img width="847" height="342" alt="Fig 9 - file dir" src="https://github.com/user-attachments/assets/add9228a-de28-41e4-a35c-58408d8bf65b" />


| Operator | Behaviour | How I used it |
| --- | --- | --- |
| `>` | Overwrites: sends output to a file and replaces existing content | First line, to start `evidence_note.txt` cleanly |
| `>>` | Appends: adds output to the end and keeps existing content | Second and third lines, so earlier lines were not lost |

## Task 3: Copy, Rename and Delete Files Safely

**3.1 Copy a file to another folder** (`copy evidence_note.txt ..\workspace2\evidence_note_copy.txt`)

<img width="1402" height="381" alt="Fig 10 - File copying" src="https://github.com/user-attachments/assets/aeb96d5f-b7c9-44f2-9a16-688e8a71e333" />


**3.2 Back up a whole folder** (`xcopy workspace workspace3\workspace_backup /E /I /H`)

<img width="1248" height="302" alt="Fig 11 - recursive file copy " src="https://github.com/user-attachments/assets/9e040819-0111-4f47-bb69-8d34129ea5c4" />


**3.3 Rename the copied file** (`ren evidence_note_copy.txt renamed_evidence_note.txt`)

<img width="1362" height="412" alt="Fig 12 - file renaming" src="https://github.com/user-attachments/assets/4b336686-402b-48a1-a15a-93ffc8e31275" />


**3.4 Create a temporary test file**

<img width="1147" height="540" alt="Fig 13 - temp file creation" src="https://github.com/user-attachments/assets/38e93d94-947a-47aa-8d23-f79f7d354cb2" />


**3.4 After `del delete_me.txt`**

<img width="803" height="480" alt="Fig 14 - file deletion" src="https://github.com/user-attachments/assets/02370eda-5358-4c13-86f6-6fb9f2e8e21f" />


**3.5 Create an empty folder**

<img width="920" height="502" alt="Fig 15 - Empty folder deletion" src="https://github.com/user-attachments/assets/5d49b6d5-b8a8-4718-a3d6-5db4ac3158c9" />


**3.5 After `rmdir empty_test_folder`**

<img width="868" height="477" alt="Fig 16 - Folder deletion" src="https://github.com/user-attachments/assets/487ab031-0905-46f5-9e54-dbc036b67a5f" />


Each step was verified with `dir` or `tree` before and after. Only the temporary test file and the empty test folder were removed, and the evidence folders were left intact.

## Task 4: How Windows Finds Applications (PATH)

```bat
echo %PATH%
where notepad
notepad
vol C:
```

**PATH variable and the location of Notepad**

<img width="1897" height="625" alt="Fig 17 - Notepad path " src="https://github.com/user-attachments/assets/727228e1-a421-4db3-a607-ea845bc35b5f" />


**Drive information with vol C:**

<img width="657" height="78" alt="Fig 18 - cVOL " src="https://github.com/user-attachments/assets/540c6e48-eeba-46c8-8c78-9325ab214fe3" />


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

<img width="975" height="335" alt="Fig 19 - sys info" src="https://github.com/user-attachments/assets/e0d897ec-7dc7-45f5-910d-da13516692bf" />


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

<img width="1057" height="617" alt="fig 20 - Network info and connectivity test" src="https://github.com/user-attachments/assets/b5c3652e-6dae-4492-8770-b74cac26b084" />


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

<img width="1013" height="657" alt="Fig 21 - suspecious login" src="https://github.com/user-attachments/assets/d3ca51c9-d782-437a-aed6-561bdfeb8f55" />


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

<img width="801" height="403" alt="Fig 22 - dir info and ownership" src="https://github.com/user-attachments/assets/a4cec332-1415-445b-bb21-764e6d573d46" />


- `icacls` shows that the `BUILTIN\Administrators` group has Full control (`F`) of `mini_case_log.txt`.
- The `(I)` flag means the permission is inherited from the parent `evidence` folder rather than set on the file itself.
- `icacls` lists permissions; the owner name is shown by `dir /q`.

## Task 9: Process Listing

**Process listing and browser process search**

<img width="1346" height="672" alt="Fig 23 - running services" src="https://github.com/user-attachments/assets/c5e52ca5-6d00-4270-a4cc-904156f0ebc8" />


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

<img width="700" height="177" alt="Fig 24 - exe batch" src="https://github.com/user-attachments/assets/b50ab66a-32df-4e12-bba1-4a05b9915599" />


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

<img width="700" height="177" alt="Fig 24 - exe batch" src="https://github.com/user-attachments/assets/1eb96d41-5250-4317-bb20-40ec066bebf3" />



**sys_info.bat execution**

<img width="1856" height="967" alt="Fig 25 - bat execution" src="https://github.com/user-attachments/assets/6e878dfb-1610-46c7-bf49-ecaad7d781dc" />



**Generated sys_info_report.txt**

<img width="911" height="682" alt="Fig 26 - sys info using script" src="https://github.com/user-attachments/assets/4ecda1ae-5c2d-4c5f-bc4b-3050e64ec0e0" />



## Task 11: Making sys_info.bat Callable from Any Folder

```bat
cd \Users\Public\CIP-B101_Lab1C_Beatrice_Nduta
copy scripts\sys_info.bat workspace4\sys_info.bat
dir workspace4
```

**Copying sys_info.bat into workspace4**

<img width="1050" height="390" alt="image" src="https://github.com/user-attachments/assets/e31dc0ff-1ee0-42b6-817d-73bb4599190f" />



```bat
set PATH=%PATH%;C:\Users\Public\CIP-B101_Lab1C_Beatrice_Nduta\workspace4
cd \
sys_info.bat
```

**Running sys_info.bat from C:\ after the PATH update**

<img width="491" height="82" alt="Fig 28 - callable file" src="https://github.com/user-attachments/assets/687c9b90-9b35-4b71-af02-d75e244d4826" />


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
