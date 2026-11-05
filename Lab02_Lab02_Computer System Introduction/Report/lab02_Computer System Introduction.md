# Lab 1B Report: PC System Fundamentals, Disk Sectors and Partition Analysis

**CIP-B101 Basic Computer Skills for Digital Forensics** | International Cybersecurity and Digital Forensics Academy (ICDFA) | 

| Filed | Details |
  |---|---|  
  | Student Name | Beatrice Nduta  
  | Student ID | C11/26/PCDFIR/17016 |
  | Course Name | Digital Forensics & Incident Response |
  | Instructor Name | Basiru Aliyu |
  | Date of submission | 03/11/2026

----------------------------------------------------------------------------------------------------

---

## 1. Introduction and tools

This lab builds an initial, read-only profile of a computer that could later be examined in a forensic investigation. I identified the system hardware and software, observed running processes, calculated partition sizes from sector data, explained the boot process and identified file systems. No command that writes to disk was used.

**Tools:** `msinfo32`, `systeminfo`, Task Manager, PowerShell (`Get-Disk`, `Get-Partition`, `Get-Volume`), Disk Management, DiskPart (list only), and a Kali Linux VM (`lsblk`, `fdisk -l`, `parted -l`, `blkid`, `df -h`).

---

## 2. System profile and evidence layers

The host is a Lenovo ThinkPad E15 Gen 4 running Windows 11 Pro (build 26200, x64) with a 12th Gen Intel Core i7-1255U processor (10 cores, 12 logical processors) and 8 GB RAM. Storage is a WD PC SN740 512 GB NVMe SSD (476.94 GB reported, GPT, healthy).

Evidence can sit in several layers at once:

- **User activity:** recent files, browser history
- **Applications:** caches, document metadata
- **Operating system:** event logs, registry
- **Disk:** deleted files, unallocated space
- **Memory:** running processes, network connections

The same action often leaves traces in more than one layer, so corroborating across layers makes findings stronger.

---

## 3. Program, process and thread

A **program** is a passive set of instructions stored on disk. A **process** is a program that is running, with its own memory space and a unique PID. A **thread** is the unit of execution inside a process and is scheduled by the CPU; threads in one process share its memory.

Process data matters in malware analysis and live triage because it shows what the system is doing now: suspicious names, unusual file paths, odd parent-child relationships (for example Word starting PowerShell), and network connections tied to a PID. It is volatile, so it must be collected before shutdown.

| Process | PID | Likely purpose |
|---|---|---|
| `audiodg.exe` | 2292 | Windows Audio Device Graph Isolation: runs audio processing in isolation from the audio service |
| `backgroundTaskHost.exe` | 24908 | Background Task Host: runs background tasks for Windows apps |
| `brave.exe` | 27264 | Brave web browser process (tab, extension or helper) |

---

## 4. Disk sectors and partition size calculation

**Formula:** size (bytes) = number of sectors × bytes per sector; GB = bytes ÷ (1024 × 1024 × 1024). `Get-Disk` confirms a logical sector size of 512 bytes (physical 4096, a 512e drive). Windows reports offsets and sizes in bytes, so sectors = bytes ÷ 512. The slide example (400 × 12 × 17,000 × 512) gives 41,779,200,000 bytes = 38.91 GB.

| # | Type | File system | Start sector | Sectors | Size (bytes) | Size (GB) |
|---|---|---|---|---|---|---|
| 1 | System (EFI) | FAT32* | 2,048 | 204,800 | 104,857,600 | 0.098 |
| 2 | Reserved (MSR) | None | 206,848 | 32,768 | 16,777,216 | 0.016 |
| 3 | Basic (C:) | NTFS | 239,616 | 683,726,848 | 350,068,146,176 | 326.03 |
| 4 | Recovery | NTFS | 683,966,464 | 2,654,208 | 1,358,954,496 | 1.266 |

*\*FAT32 is the standard file system for an EFI System Partition; it was not shown in my `Get-Volume` output.*

![Get-Disk, Get-Partition and Get-Volume output](images/fig3_storage_output.png)

*Fig 3: Get-Disk, Get-Partition and Get-Volume output (disk 0, GPT, four partitions, NTFS volumes)*

**Worked example, partition 1:**

```text
Start sector = 1,048,576 ÷ 512      = 2,048
Sectors      = 104,857,600 ÷ 512    = 204,800
Check        = 204,800 × 512        = 104,857,600 bytes
             ÷ 1024 = 102,400 KB
             ÷ 1024 = 100 MB
             ÷ 1024 = 0.098 GB
```

**Worked example, partition 2:**

```text
Sectors = 16,777,216 ÷ 512 = 32,768
Size    = 16,777,216 ÷ 1024 = 16,384 KB
        ÷ 1024 = 16 MB
        ÷ 1024 = 0.016 GB
```

**Why sizes differ between tools:** GB versus GiB (350.07 decimal GB equals 326.03 binary GB for partition 3), rounding, reserved areas (EFI, MSR, recovery), file system overhead such as the MFT, journal and backup boot sector, and alignment at the start of the disk. Each volume in `Get-Volume` is exactly 4,096 bytes smaller than its partition. The partitions are contiguous, and about 149.5 GB of the disk is unallocated.

---

## 5. Boot process and boot mode

1. **Power-on:** the CPU resets and runs firmware code.
2. **POST** tests CPU, RAM and devices.
3. **UEFI firmware** starts.
4. **CMOS/NVRAM settings** (boot order, boot mode, Secure Boot) are read.
5. The **GPT and EFI System Partition** are located (on legacy BIOS the MBR in sector 0 is read instead).
6. The **boot manager** (Windows Boot Manager, or GRUB on Linux) loads the chosen OS.
7. The **kernel** is loaded into memory and initialises drivers.
8. The **operating system takes control** and starts services and the sign-in screen.

**Boot mode: UEFI.** `msinfo32` shows BIOS Mode: UEFI (Secure Boot State: Off), and the disk is GPT with a 100 MB EFI System Partition and a 16 MB MSR. MBR is an older scheme with its partition table and boot code in sector 0 and a limit of about 2 TB; GPT supports far larger disks and keeps primary and backup headers, which helps recovery and validation.

---

## 6. File systems and forensic importance

`Get-Volume` shows NTFS on both the C: and Recovery volumes. The Kali VM uses one ext4 partition (`/dev/sda1`, 80.1 GB) on an MBR (msdos) disk, with 512-byte sectors.

A file system matters in forensics because it controls how data is organised, named, timestamped, allocated and deleted. Knowing it lets an examiner interpret raw data, recover deleted files, inspect slack and unallocated space, build timelines from metadata and journals (NTFS MFT and USN journal, ext4 inodes), choose suitable tools, and avoid actions that alter evidence.

---

## 7. Conclusion

The system is a UEFI/GPT Windows 11 machine with four partitions whose sizes I verified by calculation and cross-checked against Windows output. Working read-only preserved the original state of the evidence, and the lab showed why sector-level arithmetic, boot knowledge and file system awareness are core skills for digital forensic work. Screenshots supporting each finding are in the full lab document (Figs 1 to 12).
