# Lab 1C Report: Windows Command Line for Digital Forensics Basics

**CIP-B101 Basic Computer Skills for Digital Forensics | International Cybersecurity and Digital Forensics Academy (ICDFA)**

| Field | Details |
|---|---|
| Student | Beatrice Nduta |
| Student ID | C11/26/PCDFIR/17016 |
| Course | Digital Forensics & Incident Response |
| Instructor | Basiru Aliyu |
| Date | 11-12 November 2026 |
| Workspace | `C:\Users\Public\CIP-B101_Lab1C_Beatrice_Nduta` |

---

## 1. Introduction

The Windows command line lets a forensic analyst collect system, network, file and process information quickly, repeatably and with a clear record of what was run. This lab built a lab-only evidence workspace and practised path navigation, file management, system and network information collection, text searching, permission review, process listing and batch-file automation. These skills support later triage, log review and evidence-collection work.

## 2. Lab Environment

Microsoft Windows 11 Pro (build 26200, x64-based PC), Intel Core i7-1255U, about 7.8 GB RAM and a 512 GB solid-state drive. Tools used: Command Prompt, Notepad, PowerShell (as a replacement for `wmic`), `tree`, `systeminfo`, `ipconfig`, `getmac`, `ping`, `findstr`, `icacls`, `tasklist`. All commands were run only on my own machine inside the lab workspace, and IP details are partly redacted in the screenshots.

## 3. Tasks Performed

| Task | Key commands | What was done / result |
|---|---|---|
| 1 Paths | `cd`, `dir`, `tree`, `cd ..` | Moved with absolute and relative paths and mapped the workspace |
| 2 File creation | `echo >`, `echo >>`, `type` | Created `my_first_file.txt` and `evidence_note.txt` |
| 3 File handling | `copy`, `xcopy /E /I /H`, `ren`, `del`, `rmdir` | Copied, backed up, renamed and deleted only test files and folders |
| 4 PATH | `echo %PATH%`, `where notepad`, `vol C:` | Viewed PATH and located Notepad |
| 5 System info | `systeminfo`, PowerShell `Get-CimInstance` | Recorded OS, system type, memory, CPU and disk |
| 6 Network info | `ipconfig /all`, `getmac /v`, `ping` | Saved IP configuration, MAC addresses; loopback ping 0% loss |
| 7 Text search | `findstr /i`, `findstr /n /i` | Searched `mini_case_log.txt` for email, keyword and IP lines |
| 8 Permissions | `dir /q`, `icacls` | `BUILTIN\Administrators` has Full control (inherited) |
| 9 Processes | `tasklist` | Listed processes and searched for Chrome and Edge |
| 10 Batch files | Notepad, `.bat` | Built `hello_world.bat` and `sys_info.bat` |
| 11 PATH for scripts | `set PATH=%PATH%;...` | Ran `sys_info.bat` from `C:\` using a temporary PATH change |

## 4. Selected Evidence Screenshots

**Fig 1. Lab workspace and subfolders created**

<img width="1477" height="627" alt="Fig 1 - making folder   dir" src="https://github.com/user-attachments/assets/3927d547-0dc6-407e-8d74-871167f24328" />


**Fig 21. `findstr` search results from `mini_case_log.txt`**

<img width="1013" height="657" alt="Fig 21 - txt evidence search" src="https://github.com/user-attachments/assets/376e497f-021d-4955-bd71-c6597322af18" />


**Fig 27. Generated `sys_info_report.txt`**

<img width="876" height="281" alt="Fig 27 - New folder creation for script" src="https://github.com/user-attachments/assets/923b2442-5a7c-45bf-add0-cdbdaf7fdf21" />


**Fig 29. `sys_info.bat` run from `C:\` after the PATH update**

<img width="491" height="82" alt="Fig 28 - callable file" src="https://github.com/user-attachments/assets/b9425d23-dedb-447c-afcf-2f7a05765681" />


*All 29 screenshots are in the `screenshots/` folder of the lab repository.*

## 5. Forensic Interpretation

**Absolute vs relative path.** An absolute path starts at the drive root and works from anywhere, for example `C:\Users\Public\CIP-B101_Lab1C_Beatrice_Nduta\outputs\sys_info_report.txt`. A relative path depends on the current folder, for example `..\outputs\sys_info_report.txt`, which failed when I ran the script from `C:\`.

**`>` vs `>>`.** `>` overwrites a file, while `>>` appends to it. I used `>` once to start `evidence_note.txt` cleanly and `>>` for later lines so nothing was lost.

**PATH and malware triage.** Windows searches PATH folders in order when a command is typed. A malicious program placed in a PATH folder, or an altered PATH entry, could run in place of a trusted tool (PATH hijacking) with the user's permissions. PATH should contain only trusted folders, and permanent changes need approval. I used a temporary `set PATH` change, so nothing permanent was altered.

**`findstr` switches.** `/i` makes the search case-insensitive and `/n` prints line numbers, which lets an analyst point to the exact location of a hit.

**Permissions.** `icacls` showed Full control for `BUILTIN\Administrators`, marked `(I)` because it is inherited from the `evidence` folder. The owner name is shown by `dir /q`.

**Why batch files help.** A batch file runs the same commands in the same order every time and writes results to a report automatically, which makes collection fast, consistent and repeatable between analysts.

**Documenting failures.** A failed command is part of the record. It shows what was attempted, explains gaps in the evidence (for example, empty output files), and can itself be a finding.

## 6. Challenges

- **`wmic` not recognised.** Microsoft has removed `wmic`, so the CPU, disk, MAC and process commands failed and `cpu_info.txt` and `disk_info.txt` were created empty (0 bytes). I replaced them with `Get-CimInstance`, `Get-NetAdapter`, `getmac /v` and `tasklist`, and documented the failures.
- **`hello_world.bat` typed into Command Prompt.** My first attempt typed the lines directly instead of saving a file. I created the file in Notepad and ran it.
- **Relative path failure.** `sys_info.bat` used `..\outputs\`, which broke when run from `C:\`. I changed every occurrence to an absolute path.

## 7. Conclusion

I can now build a safe evidence workspace, navigate by absolute and relative paths, manage files without touching evidence, collect system, network and process information, search and review permissions on text evidence, and automate collection with a batch file that runs from any folder. The lab also showed that commands change between Windows versions, that relative paths can break scripts, and that documenting failures keeps a record honest. These habits form the base for later triage and evidence-collection labs.
