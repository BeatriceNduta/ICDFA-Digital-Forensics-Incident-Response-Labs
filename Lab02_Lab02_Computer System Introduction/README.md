LAB 1B: PC SYSTEM FUNDAMENTALS, DISK SECTORS AND PARTITION ANALYSIS
-------------------------------------------------------------------

  | Filed | Details |
  |---|---|  
  | Student Name | Beatrice Nduta  
  | Student ID | C11/26/PCDFIR/17016 |
  | Course Name | Digital Forensics & Incident Response |
  | Instructor Name | Basiru Aliyu |
  | Date of submission | 03/11/2026

----------------------------------------------------------------------------------------------------
Course: CIP-B101 Basic Computer Skills for Digital Forensics
Institution: International Cybersecurity and Digital Forensics Academy (ICDFA)
Author: Beatrice Nduta

A read-only practical lab that builds an initial system profile of a computer, the way a junior digital forensic trainee would before an investigation. It covers system identification, processes, disk sector arithmetic, partition table inspection, the PC boot process and file system identification.

Safety note: Every command in this lab is read-only. No formatting, partitioning, dd writes, mkfs, or read-write mounts were performed. This preserves the original state of the evidence.

---

CONTENTS
--------

  File  |  Description
  CIP-B101_Lab_1B_Beatrice_Nduta_Final_Merged.pdf  |  Full lab document with screenshots, calculations and the 2-page summary report
  CIP-B101_Lab_1B_Beatrice_Nduta_Final_Merged.docx  |  Editable Word version of the same document
  README.txt  |  This file

---

LAB OBJECTIVES
--------------

- Identify system hardware, operating system, storage and memory details
- Differentiate between a program, a process and a thread
- Calculate disk and partition sizes from sector count and sector size
- Inspect partition layouts using safe, read-only commands
- Explain the boot sequence from power-on to operating system control
- Compare MBR and GPT, and identify file systems
- Explain why file systems matter in digital forensics

---

ENVIRONMENT
-----------

  Item  |  Detail
  Host machine  |  Lenovo ThinkPad E15 Gen 4
  Operating system  |  Windows 11 Pro (build 26200, x64)
  Processor  |  12th Gen Intel Core i7-1255U (10 cores, 12 logical processors)
  Memory  |  8 GB RAM
  Storage  |  WD PC SN740 512 GB NVMe SSD (476.94 GB reported)
  Boot mode  |  UEFI (Secure Boot: Off)
  Linux VM  |  Kali Linux, 80.1 GB virtual disk

---

KEY FINDINGS
------------

PARTITION TABLE (WINDOWS HOST, GPT, 512-BYTE LOGICAL SECTORS)
-------------------------------------------------------------

  #  |  Type  |  File system  |  Start sector  |  Sectors  |  Size (bytes)  |  Size (GB)
  1  |  System (EFI)  |  FAT32*  |  2,048  |  204,800  |  104,857,600  |  0.098
  2  |  Reserved (MSR)  |  None  |  206,848  |  32,768  |  16,777,216  |  0.016
  3  |  Basic (C:)  |  NTFS  |  239,616  |  683,726,848  |  350,068,146,176  |  326.03
  4  |  Recovery  |  NTFS  |  683,966,464  |  2,654,208  |  1,358,954,496  |  1.266

*FAT32 is the standard file system for an EFI System Partition. It was not shown in the Get-Volume output.

OTHER RESULTS
-------------

- Boot mode: UEFI. msinfo32 reports BIOS Mode: UEFI, and the disk is GPT with an EFI System Partition.
- Sector size: logical 512 bytes, physical 4096 bytes (a 512e drive).
- Partition vs volume size: each NTFS volume is exactly 4,096 bytes smaller than its partition (file system overhead).
- Kali VM: one ext4 partition (/dev/sda1, 80.1 GB) on an MBR (msdos) disk.
- Processes observed: audiodg.exe (PID 2292), backgroundTaskHost.exe (PID 24908), brave.exe (PID 27264).

---

METHOD
------

SIZE CALCULATION
----------------

Size (bytes)    = number of sectors × bytes per sector
Number of sectors = size in bytes ÷ bytes per sector
Size (GB)       = bytes ÷ (1024 × 1024 × 1024)

Worked example for partition 1:

Start sector = 1,048,576 ÷ 512      = 2,048
Sectors      = 104,857,600 ÷ 512    = 204,800
Check        = 204,800 × 512        = 104,857,600 bytes
             ÷ 1024 = 102,400 KB  ÷ 1024 = 100 MB  ÷ 1024 = 0.098 GB

Windows reports sizes in bytes, so the sector count is derived by dividing by 512 and then verified by multiplying back.

WINDOWS COMMANDS (READ-ONLY)
----------------------------

msinfo32                                   # System Information (Win+R)
systeminfo
Get-Disk
Get-Partition
Get-Volume
Get-Process | Select-Object -First 20 Name,Id,CPU,WorkingSet
tasklist
diskmgmt.msc                               # Disk Management

DiskPart was used for listing only:

diskpart
list disk
select disk 0
list partition
exit

LINUX COMMANDS (READ-ONLY)
--------------------------

lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT
sudo fdisk -l        # always with -l, which only lists
sudo parted -l
blkid
df -h

---

BOOT PROCESS SUMMARY
--------------------

1. Power-on: the CPU resets and runs firmware code
2. POST: hardware self-test
3. UEFI/BIOS firmware starts
4. CMOS/NVRAM settings are read (boot order, boot mode, Secure Boot)
5. Boot records are located: GPT and EFI System Partition (or the MBR on legacy BIOS)
6. Boot manager runs: Windows Boot Manager or GRUB
7. Kernel is loaded into memory
8. Operating system takes control: services, drivers and sign-in

---

WHY GB VALUES DIFFER BETWEEN TOOLS
----------------------------------

- GB vs GiB: decimal (1 GB = 1,000,000,000 bytes) versus binary (1 GiB = 1024³ bytes). Partition 3 is 350.07 decimal GB or 326.03 binary GB.
- Rounding in tool output
- Reserved areas: EFI, MSR and Recovery partitions
- File system overhead: MFT, journal, backup boot sector, cluster rounding
- Alignment gap: the first partition starts at 1 MiB (sector 2,048)

---

PRIVACY
-------

Screenshots in the lab document have had the computer name, username and device serial details redacted. Partition sizes and file system evidence were left visible as required for grading.

---

REFERENCES
----------

- Course slide deck: *Computer System Introduction* (CIP-B101)
- Microsoft documentation for Get-Disk, Get-Partition, Get-Volume and DiskPart
- Linux man pages for lsblk, fdisk, parted, blkid and df

---

ACADEMIC NOTE
-------------

This repository documents my own lab work for coursework. If you are a student on the same course, use it for reference only and complete your own practical on your own machine.
