# CIP-B101 Lab 2A: Linux Command Line for Digital Forensics Basics

![Platform](https://img.shields.io/badge/platform-Kali%20Linux-557C94)
![Shell](https://img.shields.io/badge/shell-sh%20%2F%20zsh-4EAA25)
![Course](https://img.shields.io/badge/course-CIP--B101-1F3A5F)

Practical lab work for **CIP-B101 Basic Computer Skills for Digital Forensics**, International Cybersecurity and Digital Forensics Academy (ICDFA).

**Author:** Beatrice Nduta
**Environment:** Kali Linux VM (kernel 6.19.14+kali-amd64), user `kali`

---

## Overview

This lab builds a structured Linux evidence workspace and practises the command-line skills that underpin digital forensics work: navigation, file handling, metadata inspection, content search, safe network checks, scripting and archiving.

## Workspace Structure

```text
CIP-B101_Lab2A_Beatrice_Nduta/
├── notes/         # case notes
├── evidence/      # evidence files, logs, backups
├── scripts/       # lab2a_sys_summary.sh
├── outputs/       # system_summary.txt
├── screenshots/   # captioned evidence screenshots
└── archive/
```

## Tasks Completed

| # | Task | Key commands |
|---|------|--------------|
| 1 | Prepare evidence workspace | `mkdir -p`, `pwd`, `ls -la` |
| 2 | Inspect filesystem and OS info | `ls /`, `cat /etc/os-release`, `uname -a`, `whoami`, `id` |
| 3 | Inodes and disk information | `ls -li`, `stat`, `df -h`, `df -i` |
| 4 | Absolute and relative paths | `cd`, `readlink -f` |
| 5 | Create, copy, rename, remove files | `cp`, `cp -r`, `mv`, `rm` |
| 6 | Search evidence with grep | `grep`, `grep -n`, `grep -Ei` |
| 7 | File search and directory tree | `tree`, `find` |
| 8 | Safe network checks (localhost only) | `ip addr`, `ip route`, `ping`, `ss -lnt`, `nmap` |
| 9 | Forensic summary script | `nano`, `chmod +x` |
| 10 | Script callable via PATH | `export PATH=...` |
| 11 | Compress the workspace | `zip -r`, `tar -czvf` |

## Usage

Replace the folder name with your own if reproducing this lab.

```bash
# 1. Create the workspace
cd ~
mkdir -p CIP-B101_Lab2A_Beatrice_Nduta/{notes,evidence,scripts,outputs,screenshots,archive}

# 2. Make the script executable and run it
cd ~/CIP-B101_Lab2A_Beatrice_Nduta/scripts
chmod +x lab2a_sys_summary.sh
./lab2a_sys_summary.sh

# 3. Run it from any folder by adding scripts/ to PATH (current session only)
export PATH=$HOME/CIP-B101_Lab2A_Beatrice_Nduta/scripts:$PATH
cd /tmp
lab2a_sys_summary.sh

# 4. Review the output
head -n 15 ~/CIP-B101_Lab2A_Beatrice_Nduta/outputs/system_summary.txt
```

The script records the date, user, hostname, kernel, current folder, disk space, inode usage and a tree of the workspace into `outputs/system_summary.txt`.

## Key Findings

- **Failed login:** `grep -n "failed" evidence/activity_log.txt` shows an admin login failure from `192.168.56.15`, a different IP from the successful login source (`192.168.56.10`), followed by an admin password change.
- **PATH:** commands such as `ls` run from any folder because their executables live in directories listed in `$PATH` (for example `/usr/bin`).
- **Inodes:** an inode stores a file's metadata; the filename is only a pointer to it.

## Archiving

```bash
cd ~
zip -r CIP-B101_Lab2A_Beatrice_Nduta.zip CIP-B101_Lab2A_Beatrice_Nduta
tar -czvf CIP-B101_Lab2A_Beatrice_Nduta.tar.gz CIP-B101_Lab2A_Beatrice_Nduta
ls -lh CIP-B101_Lab2A_Beatrice_Nduta.zip CIP-B101_Lab2A_Beatrice_Nduta.tar.gz
```

## Safety Note

All commands were run only on my own Kali VM. Network testing was limited to `localhost` (`127.0.0.1`). Do not scan or probe systems you do not own or have written permission to test.

## Files

- `CIP-B101_Lab2A_Report_Beatrice_Nduta.pdf`: 2-page lab report
- `CIP-B101_Lab2A_Beatrice_Nduta.zip` / `.tar.gz`: compressed evidence workspace

## License

Coursework submitted for educational purposes.
