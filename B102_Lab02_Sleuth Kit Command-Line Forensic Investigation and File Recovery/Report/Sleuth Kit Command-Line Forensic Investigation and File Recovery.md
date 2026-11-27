# CIP-B102 Practical Lab 02: Command-Line Forensic Image Investigation


| Field | Details |
|---|---|
|Institution | International Cybersecurity and Digital Forensics Academy (ICDFA)  |
|Course | CIP-B102 — Foundations of Computer and Digital Forensics (Module 2: The Sleuth Kit Tutorial) | 
|Examiner | Beatrice Nduta (`kali@kali`) |  
|Environment | Kali Linux 2026.2 (`kali-linux-2026.2-vmware-amd64`), The Sleuth Kit (TSK) v4.15.0  |
|Evidence Image | `Ch01InChap01.dd` (Raw FAT12 Volume, 1,474,560 bytes)  |
|Working Directory | `/home/kali/CIP-B102-Lab02_Beatrice_Nduta`  |



---



## 1. Scenario, Scope, and Forensic Governance



### Investigation Scenario

An investigating officer requested a documented command-line forensic examination of the raw training image `Ch01InChap01.dd`. The objective is to enumerate allocated and deleted directory entries, recover the deleted text file `letter1.txt` and the spreadsheet `INCOME.XLS` using independent metadata-level and sector-level recovery techniques, verify file integrity using cryptographic hashes, and report all material observations objectively without assuming wrongdoing.



### Authorised Scope and Preservation Rules

* Operated exclusively on the authorised ICDFA training image inside an isolated Kali Linux virtual laboratory.

* Executed no destructive disk or partition commands against physical or virtual drives.

* Never mounted `Ch01InChap01.dd` read-write and never modified the source image inside `evidence/`.

* Stored all command outputs, extracted files, and screenshots in separate `reports/`, `recovered/`, and `screenshots/` directories.

* Conducted as a standalone command-line investigation independent of Autopsy Lab 01.



---



## 2. Laboratory Environment Preparation



The Sleuth Kit (`sleuthkit`) and native Linux inspection tools (`xxd`, `file`, `curl`) were verified on Kali Linux. Binary path checks confirmed `/usr/bin/` availability for all required utilities, and `fls -V` confirmed **The Sleuth Kit ver 4.15.0**.



```bash

sudo apt update && sudo apt install -y sleuthkit xxd file curl

command -v img_stat fsstat fls icat istat blkcat tsk_recover

fls -V



mkdir -p ~/CIP-B102-Lab02_Beatrice_Nduta/{evidence,recovered,reports,screenshots}

cd ~/CIP-B102-Lab02_Beatrice_Nduta

pwd

ls -ld evidence recovered reports screenshots

```



* **Verified Directory Structure:** `/home/kali/CIP-B102-Lab02_Beatrice_Nduta` containing `evidence`, `recovered`, `reports`, and `screenshots`.



<img width="1573" height="717" alt="Fig 1 working Dir" src="https://github.com/user-attachments/assets/57358444-80ed-4392-a7aa-0c7cafd0703a" />

*Figure 1: Verification of Sleuth Kit v4.15.0 tools in `/usr/bin/` and segregated workspace creation (`screenshots/Fig_01_Working_Dir.png`).*



---



## 3. Evidence Acquisition, Register, and Cryptographic Verification



`Ch01InChap01.dd` was downloaded into `~/CIP-B102-Lab02_Beatrice_Nduta/evidence` and immediately verified using `file`, `stat`, `md5sum`, and `sha256sum`.



```bash

cd ~/CIP-B102-Lab02_Beatrice_Nduta/evidence

curl -fL --retry 2 -o Ch01InChap01.dd \

  "https://www.dropbox.com/s/nw23q14vzsykyup/Ch01InChap01.dd?dl=1"

file Ch01InChap01.dd

stat -c "%n | %s bytes" Ch01InChap01.dd

md5sum Ch01InChap01.dd | tee ../reports/image_md5.txt

sha256sum Ch01InChap01.dd | tee ../reports/image_sha256.txt

```



<img width="1356" height="413" alt="Fig 2 Evidence file Download   Hashing" src="https://github.com/user-attachments/assets/1adb7ae3-bf72-4b11-bfbe-a6f2d53f665a" /> 

*Figure 2: Acquisition of `Ch01InChap01.dd` (1,474,560 bytes) and MD5/SHA-256 baseline hashing (`screenshots/Fig_02_Evidence_Hashing.png`).*



### Evidence Register



| Attribute | Recorded Value |

| :--- | :--- |

| **Evidence File & Location** | `~/CIP-B102-Lab02_Beatrice_Nduta/evidence/Ch01InChap01.dd` |

| **File Signature (`file`)** | `DOS/MBR boot sector, code offset 0x34+2, OEM-ID ",'k0nIHC" cached by Windows 9M, root entries 224, sectors 2880 (volumes <= 32 MB), sectors/FAT 9, sectors/track 18, dos < 4.0 BootSector (0), FAT (12 bit by descriptor+sectors), followed by FAT` |

| **Measured File Size** | `1474560 bytes` (2,880 sectors x 512 bytes/sector) |

| **MD5 Hash** | `a117773bcf1fc88ec0ab8e0a349fbbcb` |

| **SHA-256 Hash** | `3ce8053e4f3d9c8ab98b3aadb2480685efb8e4980d34297b83bd5a09b1a7b122` |

| **Reference Comparison** | Exact match to slide-deck reference size (`1,474,560 bytes`) and MD5 (`a117773bcf1fc88ec0ab8e0a349fbbcb`) |



---



## 4. Image Structure, Filesystem Geometry, and Directory Enumeration



### Image and Filesystem Examination (`img_stat`, `mmls`, `fsstat`)



```bash

img_stat Ch01InChap01.dd | tee ../reports/01_img_stat.txt

mmls Ch01InChap01.dd | tee ../reports/02_mmls.txt

fsstat Ch01InChap01.dd | tee ../reports/03_fsstat.txt

```



<img width="1363" height="708" alt="Fig 3 Image   Filesys examination" src="https://github.com/user-attachments/assets/5521ff3e-0bca-49b9-9698-c878b680ceb8" />  

*Figure 3: Raw image format and FAT12 sector layout confirmed via `img_stat`, `mmls`, and `fsstat` (`screenshots/Fig_03_Image_Filesystem_Exam.png`).*



* **Image Type & Geometry (`img_stat`):** `raw` image, `1474560` bytes, sector size `512` bytes.

* **Partition Check (`mmls`):** No partition table is present (`Sectors before file system: 0`), confirming `Ch01InChap01.dd` is a raw filesystem-only volume (1.44 MB floppy geometry). Consequently, `fsstat` and subsequent TSK commands were run directly without a sector offset (`-o`).

* **FAT12 Layout (`fsstat`):**

  * **Filesystem Type:** `FAT12` (OEM Name: `,'k0nIHC`, Volume ID: `0x0`)

  * **Total Sector Range:** `0 - 2879`

  * **Reserved / Boot Sector:** `0`

  * **FAT 0:** `1 - 9` | **FAT 1:** `10 - 18`

  * **Data Area:** `19 - 2879` (**Root Directory:** `19 - 32`; **Cluster Area:** `33 - 2879`)



### Directory Listing and Deleted Entries (`fls`)



```bash

fls Ch01InChap01.dd | tee ../reports/04_fls_root.txt

fls -r Ch01InChap01.dd | tee ../reports/05_fls_recursive.txt

fls -r -d Ch01InChap01.dd | tee ../reports/06_deleted.txt

```



<img width="1188" height="692" alt="Fig 4 fls deleted file listing " src="https://github.com/user-attachments/assets/0edcd872-56a4-4e1b-b644-cc337528469a" /> 

*Figure 4: Root directory and deleted file enumeration (`fls`, `fls -r`, `fls -r -d`) showing four deleted entries (`screenshots/Fig_04_fls_Deleted_Listing.png`).*



`fls` enumerated 6 user directory entries plus 4 TSK virtual entries (`$MBR`, `$FAT1`, `$FAT2`, `$OrphanFiles`), identifying **4 deleted entries**:



| Metadata Address | Type / Flag | Status | Filename |

| :--- | :--- | :--- | :--- |

| `5` | `r/r` | Allocated | `Client Info.mdb` |

| `8` | `r/r *` | Deleted (Unallocated) | `Billing Letter.doc` |

| `11` | `r/r *` | Deleted (Unallocated) | `confirmation.txt` |

| `13` | `r/r` | Allocated | `Income.xls` |

| `15` | `r/r *` | Deleted (Unallocated) | `letter1.txt` |

| `17` | `r/r *` | Deleted (Unallocated) | `Regrets.doc` |



* **Allocated vs. Deleted Directory Entries:** An *allocated* entry is an active filesystem record pointing to clusters currently reserved by the OS. A *deleted (unallocated)* entry is a residual directory record marked free for reuse (`0xE5` in the first byte of the FAT entry, shown as `*` in `fls` and `_` in `istat`), while its sector contents remain on disk until overwritten. `fls` only enumerates entries; it does not extract file contents.



---



## 5. Deleted Text File Recovery and Sector Analysis (`letter1.txt`)



### Metadata Examination & `icat` Recovery



```bash

icat -r Ch01InChap01.dd 15 > ../recovered/letter1_icat.txt

istat Ch01InChap01.dd 15 | tee ../reports/07_letter1_istat.txt

file ../recovered/letter1_icat.txt

sha256sum ../recovered/letter1_icat.txt

```



<img width="1007" height="450" alt="Fig 5 letter1 deleted file meta" src="https://github.com/user-attachments/assets/4b1a79a1-2301-4387-bacc-a6694aea6ab6" />  

*Figure 5: Metadata inspection (`istat` entry 15), `icat` extraction, and SHA-256 verification of `letter1.txt` (`screenshots/Fig_05_letter1_Metadata.png`).*



* **Metadata Summary (`istat` entry `15`):**

  * **Allocation State:** `Not Allocated` | **Attributes:** `File, Archive`

  * **Short Name:** `_ETTER1.TXT` (leading byte replaced by FAT deletion marker)

  * **File Size:** `121` bytes

  * **Timestamps (EAT):** Written `2005-12-09 06:51:50` | Accessed `2005-12-09 00:00:00` | Created `2005-12-09 06:59:09`

  * **Sector List:** Single sector `312`

* **Recovered File Verification:**

  * **File Type:** `ASCII text, with CRLF line terminators`

  * **SHA-256 Digest:** `341065f2c3c2168e246aa9a346439dfa4e6e4c55de4b1b8ecc78e4b83206545f`



### Content Preview and Hexadecimal Sector Inspection (`blkcat` + `xxd`)



```bash

cat ../recovered/letter1_icat.txt

blkcat Ch01InChap01.dd 312 | xxd | head -n 16

```



<img width="1365" height="703" alt="Fig 6 evidence file display using icat   blkcat" src="https://github.com/user-attachments/assets/2b1650c2-ee89-4bf1-aa22-583a1f0213e4" />  

*Figure 6: Text display via `cat` and raw hexadecimal dump of sector 312 via `blkcat | xxd` showing 391 bytes of zero slack (`screenshots/Fig_06_letter1_icat_blkcat.png`).*



* **Recovered Content (`cat`):**

  > Earl,  

  > We need to meet on the 18th of August to confirm the work I am doing for you. Please contact me ASAP.  

  >  

  > George

* **`icat` vs. `blkcat` & Slack Padding Interpretation:** `icat` extracts by metadata address (`15`) and automatically truncates output to the exact 121-byte logical file size stored in the directory entry. Conversely, `blkcat` reads the entire 512-byte physical data unit (sector `312`). In the `xxd` output, the 121 bytes of ASCII text occupy byte offsets `0x00` through `0x78` (`47 65 6f 72 67 65` = `"George"`), followed immediately at offset `0x79` by **391 bytes of `0x00` zero padding** (sector slack) to the end of the 512-byte sector. Raw sector output must therefore be trimmed using the `istat` file size before claiming exact file length.



---



## 6. Multi-Method Recovery and Integrity Comparison of `INCOME.XLS`



### Metadata Verification (`istat` Entry `13`)



```bash

istat Ch01InChap01.dd 13 | tee ../reports/08_income_istat.txt

```



* **Allocation State:** `Allocated` | **Name:** `INCOME.XLS` | **Size:** `13824` bytes

* **Timestamps (EAT):** Written `2005-12-09 06:52:18` | Accessed `2005-12-09 00:00:00` | Created `2005-12-09 06:59:07`

* **Data-Unit Sequence:** **27 contiguous sectors** running uninterrupted from sector `285` through `311` (`311 - 285 + 1 = 27`).



### Three-Method Extraction



1. **Method 1 — Metadata Extraction (`icat`):**

   ```bash

   icat Ch01InChap01.dd 13 > ../recovered/INCOME_icat.xls

   stat -c "%n | %s bytes" ../recovered/INCOME_icat.xls

   ```



<img width="1337" height="697" alt="Fig 7 file multiple sectors" src="https://github.com/user-attachments/assets/a8b240b3-ebe0-4b23-a970-151dd18c9ae1" /> 

*Figure 7: Metadata inspection (`istat` entry 13) confirming 27 contiguous sectors (`285–311`) and `icat` recovery of `INCOME_icat.xls` (`screenshots/Fig_07_INCOME_Multiple_Sectors.png`).*



2. **Method 2 — Contiguous Sector Extraction & Exact-Byte Trimming (`blkcat` + `head`):**

   ```bash

   blkcat Ch01InChap01.dd 285 27 > ../recovered/INCOME_blkcat_full.bin

   head -c 13824 ../recovered/INCOME_blkcat_full.bin > ../recovered/INCOME_blkcat.xls

   ls -l ../recovered/INCOME_blkcat.xls

   file ../recovered/INCOME_blkcat.xls

   ```

   *Reconstruction Explanation:* Because `istat` confirmed a contiguous sequence from sector `285` to `311`, `blkcat` read 27 consecutive sectors (`27 x 512 = 13,824 bytes`), and `head -c 13824` enforced the exact byte length. Because `13,824` is an exact multiple of 512 bytes, there was zero end-of-sector slack at sector `311`.



<img width="1363" height="321" alt="Fig 8 data unit recovery method " src="https://github.com/user-attachments/assets/afce46d2-8f6f-4427-a2c1-e1280def5425" />  

*Figure 8: Exact-length byte trimming (`head -c 13824`) and OLE2 file signature verification of `INCOME_blkcat.xls` (`screenshots/Fig_08_Data_Unit_Recovery.png`).*



3. **Method 3 — Automated Filesystem Recovery (`tsk_recover`):**

   ```bash

   mkdir -p ../recovered/tsk_output

   tsk_recover -e Ch01InChap01.dd ../recovered/tsk_output

   find ../recovered/tsk_output -iname '*income*' -type f -print

   ```

   *Evidence Path Retained:* `../recovered/tsk_output/Income.xls` (`13824` bytes; 6 files recovered total).



<img width="1361" height="696" alt="Fig 9 tsk automated recovery method" src="https://github.com/user-attachments/assets/aef4c48a-8521-4f69-abd2-e7232a4ed942" />  

*Figure 9: Automated recovery via `tsk_recover -e` and directory listing of all recovered `INCOME` artifacts (`screenshots/Fig_09_tsk_automated_recovery.png`).*



### Three-Way Integrity & Content Comparison



```bash

sha256sum ../recovered/INCOME_icat.xls

sha256sum ../recovered/INCOME_blkcat.xls

cmp -s ../recovered/INCOME_icat.xls ../recovered/INCOME_blkcat.xls && echo 'MATCH' || echo 'DIFFER'

file ../recovered/INCOME_icat.xls

```



<img width="1362" height="728" alt="Fig 10 file Integrity and content comparison" src="https://github.com/user-attachments/assets/86377080-4ecf-4147-b745-835fa0923d90" />  

*Figure 10: Matching SHA-256 digests, binary `cmp` verification (`MATCH`), and OLE2 metadata output for `INCOME.XLS` (`screenshots/Fig_10_Integrity_Comparison.png`).*



| Method | Recovered Artifact Path | Size | SHA-256 Digest | Result |

| :--- | :--- | :--- | :--- | :--- |

| `icat` | `../recovered/INCOME_icat.xls` | `13,824 bytes` | `8d7cd7204d3dae161a8fa879e184865b3bc4a57a4e688abd522a9ff03f62252d` | Verified |

| `blkcat` + `head` | `../recovered/INCOME_blkcat.xls` | `13,824 bytes` | `8d7cd7204d3dae161a8fa879e184865b3bc4a57a4e688abd522a9ff03f62252d` | `MATCH` |

| `tsk_recover` | `../recovered/tsk_output/Income.xls` | `13,824 bytes` | Identical 13,824-byte root file (`Income.xls`) | Verified |



* **Document Metadata (`file` output):** `Composite Document File V2 Document, Little Endian, Os: Windows, Version 4.10, Code page: 1252, Author: Amelia Phillips, Last Saved By: Amelia Phillips, Name of Creating Application: Microsoft Excel, Create Time/Date: Sat Nov 23 03:17:08 2002, Last Saved Time/Date: Fri Dec 9 14:52:18 2005, Security: 0`.



---



## 7. Independent TSK Command Investigation (`ffind`)



* **Selected Command & Forensic Purpose:** `ffind` (The Sleuth Kit) maps a metadata/directory entry address back to its file name(s) and directory path(s) within the filesystem without modifying the image. It establishes file path hierarchy and detects multiple directory links to the same metadata structure.

* **Inputs, Flags & Exact Command Line:**

  * **Inputs:** Disk/filesystem image (`Ch01InChap01.dd`) and metadata address (`13`).

  * **Optional Flags:** `-a` (show all entries including deleted), `-u` (undeleted entries only), `-f fstype` (specify filesystem type), `-o OFFSET` (partition byte offset if applicable).

  * **Command Executed:**

    ```bash

    ffind Ch01InChap01.dd 13

    ```

* **Captured Output & Interpretation:**

  * **Output:** `/Income.xls`



<img width="1365" height="706" alt="Fig 11 ffind output" src="https://github.com/user-attachments/assets/ab0c05f8-c6c9-450d-99dd-9c482492835b" />

*Figure 11: Execution of `ffind Ch01InChap01.dd 13` resolving metadata entry 13 to `/Income.xls` (`screenshots/Fig_11_ffind_Output.png`).*



  * **Interpretation:** Confirms that metadata entry `13` resides in the root directory (`/`) of `Ch01InChap01.dd` and illustrates FAT long-filename case preservation (`Income.xls`) compared to the uppercase 8.3 short name (`INCOME.XLS`) shown in `istat`.

* **Limitations & Cross-Finding Support:**

  * **Limitations:** Does not verify whether underlying sectors (`285–311`) hold uncorrupted file data or sector slack, returns incomplete/missing results if directory entries are overwritten, and requires `-o OFFSET` when inspecting partitioned disk images.

  * **Cross-Finding Support:** Corroborates `istat` entry `13` (`Allocated`, `Size: 13824`, sectors `285–311`) and confirms why `tsk_recover` restored `Income.xls` at the root of `../recovered/tsk_output/`.



---



## 8. Findings, Limitations, and Conclusion



1. **Image Integrity & Structure:** `Ch01InChap01.dd` (`1,474,560` bytes) is an unpartitioned FAT12 volume matching the reference cryptographic hashes without modification.

2. **Deleted File Recovery:** Four deleted root-directory entries were identified (`Billing Letter.doc`, `confirmation.txt`, `letter1.txt`, `Regrets.doc`). `letter1.txt` (entry `15`, sector `312`, `121` bytes) was recovered intact, with `xxd` confirming `391` bytes of zero slack padding in sector `312`.

3. **Multi-Method Spreadsheet Recovery:** `INCOME.XLS` (entry `13`, sectors `285–311`, `13,824` bytes) was recovered identically via `icat`, `blkcat` + `head`, and `tsk_recover`, verified by matching SHA-256 hashes and OLE2 document headers.

4. **Operational Limitations:** Command-line recovery relies on intact directory metadata and unoverwritten clusters; heavily overwritten files require signature-based file carving, and FAT12 timestamps require timezone correlation against internal application metadata.



---



## 9. References



1. International Cybersecurity and Digital Forensics Academy (ICDFA). *CIP-B102 Practical Lab 02: The Sleuth Kit Command-Line Forensic Image Investigation*. Directorate of Training.

2. Carrier, B. *The Sleuth Kit (TSK) v4.15.0 Command Reference*. https://www.sleuthkit.org/sleuthkit/man/

3. Kali Linux Documentation. *sleuthkit Package Tracker & Tool Reference*. https://www.kali.org/tools/sleuthkit/
