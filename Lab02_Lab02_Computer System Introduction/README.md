# Lab 1B: PC System Fundamentals, Disk Sectors and Partition Analysis

**Course:** CIP-B101 Basic Computer Skills for Digital Forensics
**Institution:** International Cybersecurity and Digital Forensics Academy (ICDFA)
**Author:** Beatrice Nduta

  | Filed | Details |
  |---|---|  
  | Student Name | Beatrice Nduta  
  | Student ID | C11/26/PCDFIR/17016 |
  | Course Name | Digital Forensics & Incident Response |
  | Instructor Name | Basiru Aliyu |
  | Date of submission | 03/11/2026

----------------------------------------------------------------------------------------------------


A read-only practical lab that builds an initial system profile of a computer, the way a junior digital forensic trainee would before an investigation. It covers system identification, processes, disk sector arithmetic, partition table inspection, the boot process and file system identification.

> **Safety note:** Every command in this lab is read-only. No formatting, partitioning, `dd` writes, `mkfs`, or read-write mounts were performed, so the original state of the evidence was preserved.

---

## Table of contents

- [Environment](#environment)
- [Part A: System identification and evidence layers](#part-a-system-identification-and-evidence-layers)
- [Part B: Program, process and thread](#part-b-program-process-and-thread)
- [Part C: Disk sector and size calculation](#part-c-disk-sector-and-size-calculation)
- [Part D: Safe partition table inspection](#part-d-safe-partition-table-inspection)
- [Part E: MBR, GPT and boot process](#part-e-mbr-gpt-and-boot-process)
- [Part F: File system identification](#part-f-file-system-identification)
- [Part G: Final practical assignment](#part-g-final-practical-assignment)
- [Conclusion](#conclusion)
- [Repository structure](#repository-structure)

---

## Environment

| Item | Detail |
|---|---|
| Host machine | Lenovo ThinkPad E15 Gen 4 |
| Operating system | Windows 11 Pro (build 26200, x64) |
| Processor | 12th Gen Intel Core i7-1255U (10 cores, 12 logical processors) |
| Memory | 8 GB RAM |
| Storage | WD PC SN740 512 GB NVMe SSD (476.94 GB reported, GPT) |
| Boot mode | UEFI (Secure Boot: Off) |
| Linux VM | Kali Linux, 80.1 GB virtual disk |

---

## Part A: System identification and evidence layers

### A1. System specification

`msinfo32` shows the OS, system type, processor, memory, **BIOS Mode: UEFI** and Secure Boot State.

![Fig 1: Windows System Information (msinfo32) showing BIOS Mode: UEFI](images/fig01_msinfo32.png)

*Fig 1: Windows System Information (msinfo32) showing BIOS Mode: UEFI*

The first page of the `systeminfo` command output:

![Fig 2: Output of the systeminfo command](images/fig02_systeminfo.png)

*Fig 2: Output of the systeminfo command*

### A2. Storage information

`Get-Disk`, `Get-Partition` and `Get-Volume` show the disk model, size, partition style, partitions and file systems.

![Fig 3: Storage model, size, partitions and file systems](images/fig03_storage_output.png)

*Fig 3: Storage model, size, partitions and file systems (`Get-Disk`, `Get-Partition`, `Get-Volume`)*

### A3. Evidence layers

| Layer / source | Example evidence | Where it may be found | Why it matters |
|---|---|---|---|
| User layer | Opened files, searched terms, typed usernames | Recent files, browser history, shell history | Links user action to system activity |
| Application layer | Office document metadata, browser cache, chat logs | Application folders and databases | Shows what software created or used evidence |
| OS layer | Event logs, installed programs, user profiles | Windows logs, registry, Linux `/var/log` | Shows system events and account activity |
| Disk layer | Deleted files, file fragments, partitions | File system and unallocated space | Supports recovery and timeline analysis |
| Memory/process layer | Running malware, PIDs, network connections | RAM capture, process list | Shows live system behaviour |

The same action often leaves traces in more than one layer, so corroborating across layers makes findings stronger.

---

## Part B: Program, process and thread

### B1. Observing running processes

Task Manager showing running apps and background processes:

![Fig 4: Task Manager showing running processes](images/fig04_task_manager.png)

*Fig 4: Task Manager showing running processes*

Three processes selected for analysis:

![Fig 5: Selected processes with PIDs](images/fig05_selected_processes.png)

*Fig 5: Selected processes with PIDs*

| Process | PID | Likely purpose |
|---|---|---|
| `audiodg.exe` | 2292 | Windows Audio Device Graph Isolation: runs audio processing in isolation from the audio service |
| `backgroundTaskHost.exe` | 24908 | Background Task Host: runs background tasks for Windows apps |
| `brave.exe` | 27264 | Brave web browser process (tab, extension or helper) |

The same data from PowerShell:

![Fig 6: Running processes listed with PIDs](images/fig06_get_process.png)

*Fig 6: Running processes listed with PIDs (`Get-Process`)*

### B2. Program, process and thread

- A **program** is a passive set of instructions stored on disk, waiting to be launched.
- A **process** is a program that is currently running, with its own memory space and a unique PID.
- A **thread** is the unit of execution inside a process and is scheduled by the CPU. Threads in one process share its memory.

**Why process information matters:** it shows what the system is doing at a given moment. It can reveal suspicious process names, unusual file paths, odd parent-child relationships (for example Word starting PowerShell), and network connections tied to a PID. It is volatile, so it should be collected before shutdown.

---

## Part C: Disk sector and size calculation

**Key formula**

```text
Size (bytes) = number of sectors x bytes per sector
Size (GB)    = bytes / (1024 x 1024 x 1024)
```

### C1. Slide calculation

Sectors per track = 400, heads = 12, cylinders = 17,000, bytes per sector = 512.

| Item | Formula / value | Answer |
|---|---|---|
| Total sectors | 400 x 12 x 17,000 | 81,600,000 sectors |
| Total bytes | 81,600,000 x 512 | 41,779,200,000 bytes |
| Kilobytes | 41,779,200,000 / 1024 | 40,800,000 KB |
| Megabytes | 40,800,000 / 1024 | 39,843.75 MB |
| Gigabytes | 39,843.75 / 1024 | 38.91 GB |

### C2. Partition calculations

`Get-Disk` confirms a logical sector size of **512 bytes** (physical 4096, a 512e drive). Windows reports sizes in bytes, so sectors = bytes / 512, then multiplied back to verify.

**Partition 1 (EFI System)**

| Item | Formula / value | Answer |
|---|---|---|
| Total sectors | 104,857,600 / 512 | 204,800 sectors |
| Total bytes | 204,800 x 512 | 104,857,600 bytes |
| Kilobytes | 104,857,600 / 1024 | 102,400 KB |
| Megabytes | 102,400 / 1024 | 100 MB |
| Gigabytes | 100 / 1024 | 0.098 GB |

**Partition 2 (Microsoft Reserved)**

| Item | Formula / value | Answer |
|---|---|---|
| Total sectors | 16,777,216 / 512 | 32,768 sectors |
| Total bytes | 32,768 x 512 | 16,777,216 bytes |
| Kilobytes | 16,777,216 / 1024 | 16,384 KB |
| Megabytes | 16,384 / 1024 | 16 MB |
| Gigabytes | 16 / 1024 | 0.016 GB |

Partition table evidence for partitions 1 and 2:

![Fig 7: Get-Partition output for partitions 1 and 2](images/fig07_get_partition_p1_p2.png)

*Fig 7: `Get-Partition | Select-Object DiskNumber, PartitionNumber, Type, Offset, Size`*

### All four partitions

| # | Type | File system | Start sector | Sectors | Size (bytes) | Size (GB) |
|---|---|---|---|---|---|---|
| 1 | System (EFI) | FAT32* | 2,048 | 204,800 | 104,857,600 | 0.098 |
| 2 | Reserved (MSR) | None | 206,848 | 32,768 | 16,777,216 | 0.016 |
| 3 | Basic (C:) | NTFS | 239,616 | 683,726,848 | 350,068,146,176 | 326.03 |
| 4 | Recovery | NTFS | 683,966,464 | 2,654,208 | 1,358,954,496 | 1.266 |

\*FAT32 is the standard file system for an EFI System Partition. It was not shown in the `Get-Volume` output.

### Why sizes differ between tools

- **GB vs GiB:** 350.07 decimal GB equals 326.03 binary GB for partition 3
- **Rounding** in tool output
- **Reserved areas:** EFI, MSR and Recovery partitions
- **File system overhead:** MFT, journal, backup boot sector, cluster rounding. Each NTFS volume is exactly 4,096 bytes smaller than its partition.
- **Alignment:** the first partition starts at 1 MiB (sector 2,048)

---

## Part D: Safe partition table inspection

### D1. Windows: Disk Management, PowerShell and DiskPart

Disk Management shows the disk layout: EFI (100 MB), C: (326.03 GB NTFS), Recovery (1.27 GB) and 149.53 GB unallocated.

![Fig 9: Disk Management showing the disk layout](images/fig09_disk_management.png)

*Fig 9: Disk Management showing the disk layout*

DiskPart was used for listing only (`list disk`, `select disk 0`, `list partition`):

![Fig 10: Read-only partition listing in DiskPart](images/fig10_diskpart_listing.png)

*Fig 10: Read-only partition listing in PowerShell and DiskPart*

### D2. Linux (Kali VM): read-only inspection

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT
sudo fdisk -l
sudo parted -l
blkid
df -h
```

![Fig 11: Read-only inspection of the Kali Linux VM disk](images/fig11_kali_read_only.png)

*Fig 11: Read-only inspection of the Kali Linux VM disk. One 80.1 GB ext4 partition (`/dev/sda1`) on an MBR (msdos) disk with 512-byte sectors.*

---

## Part E: MBR, GPT and boot process

### E1. Boot process

1. **Power-on:** the CPU resets and runs firmware code
2. **POST:** hardware self-test
3. **UEFI/BIOS firmware** starts
4. **CMOS/NVRAM settings** (boot order, boot mode) are read
5. **Boot records are located:** GPT and EFI System Partition (or the MBR on legacy BIOS)
6. **Boot loader/manager runs:** Windows Boot Manager or GRUB
7. **Kernel is loaded** into memory
8. **Operating system takes control:** services, drivers and the sign-in screen

| Stage | What happens | Forensic relevance |
|---|---|---|
| Power-on and POST | Hardware is initialised and checked | Hardware failure or tampering may affect evidence access |
| BIOS/UEFI firmware | Boot device order and firmware settings are applied | Boot mode can explain partition style and boot artefacts |
| MBR/GPT or EFI boot | System locates bootable partition or EFI boot entry | Boot records may contain evidence of boot configuration |
| Boot loader/manager | OS selection and pre-kernel loading occurs | May show multi-boot or hidden OS behaviour |
| Kernel and OS control | Operating system fully starts | Logs and user activity begin to accumulate |

**Boot mode: UEFI.** `msinfo32` reports BIOS Mode: UEFI (Fig 1), and the disk uses a GPT partition table with a 100 MB EFI System Partition (Fig 3). The system therefore boots using UEFI rather than Legacy BIOS/MBR. Secure Boot State is shown as Off.

### E2. MBR vs GPT

| Feature | MBR | GPT |
|---|---|---|
| Age / design | Older partitioning system from the 1980s | Modern system used with UEFI |
| Size limitation | Commonly limited to about 2 TB | Supports very large disks (about 9.4 ZB) |
| Partition information | Partition table in the first sector with boot code | Primary and backup GPT headers and entries |
| Forensic issue | Critical data in sector 0; damage affects partition interpretation | Backup structures help recovery and validation |

---

## Part F: File system identification

Windows (`Get-Volume`): both the C: volume and the Recovery volume use **NTFS** (Fig 3). Linux (Kali): the root partition `/dev/sda1` uses **ext4**.

![Fig 12: File system identification on the Kali Linux VM](images/fig12_kali_file_systems.png)

*Fig 12: File system identification on the Kali Linux VM (`lsblk -f`, `blkid`, `df -Th`)*

**Why a file system matters in forensics:** it controls how data is organised, named, timestamped, allocated and deleted. Knowing it lets an examiner interpret raw data, recover deleted files, inspect slack and unallocated space, build timelines from metadata and journals (NTFS MFT and USN journal, ext4 inodes), choose suitable tools, and avoid actions that alter evidence.

---

## Part G: Final practical assignment

Verification of partitions 1 and 2 using partition table evidence:

![Fig 8: Partition table evidence for partitions 1 and 2](images/fig08_partition_table_evidence.png)

*Fig 8: Partition table evidence (`Get-Partition`) for partitions 1 and 2*

| Task | Partition 1 | Partition 2 |
|---|---|---|
| Offset (bytes) | 1,048,576 | 105,906,176 |
| Start sector (offset / 512) | 2,048 | 206,848 |
| Size (bytes) | 104,857,600 | 16,777,216 |
| Number of sectors (size / 512) | 204,800 | 32,768 |
| End sector | 206,847 | 239,615 |
| Size in GB (bytes / 1024^3) | 0.098 | 0.016 |
| OS-reported size | 100 MB | 16 MB |

Partition 2 starts at sector 206,848, immediately after partition 1 ends at sector 206,847, so there is no gap between them.

---

## Conclusion

The system is a UEFI/GPT Windows 11 machine with four partitions whose sizes were verified by calculation and cross-checked against Windows output. Working read-only preserved the original state of the evidence. The lab showed why sector-level arithmetic, boot knowledge and file system awareness are core skills for digital forensic work.

---

## Repository structure

```text
.
├── README.md
├── images/
│   ├── fig01_msinfo32.png
│   ├── fig02_systeminfo.png
│   ├── fig03_storage_output.png
│   ├── fig04_task_manager.png
│   ├── fig05_selected_processes.png
│   ├── fig06_get_process.png
│   ├── fig07_get_partition_p1_p2.png
│   ├── fig08_partition_table_evidence.png
│   ├── fig09_disk_management.png
│   ├── fig10_diskpart_listing.png
│   ├── fig11_kali_read_only.png
│   └── fig12_kali_file_systems.png
└── report/
    ├── CIP-B101_Lab_1B_Beatrice_Nduta_Final_Merged.pdf
    └── Lab_1B_Two_Page_Report_Beatrice_Nduta.pdf
```

---

## Academic note

This repository documents my own lab work for coursework. If you are on the same course, use it for reference only and complete the practical on your own machine.
