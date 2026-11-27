# CIP-B102 Practical Lab 02: Command-Line Forensic Investigation (Concise Report)



| Field | Details |
|---|---|
|Institution | International Cybersecurity and Digital Forensics Academy (ICDFA)  |
|Course | CIP-B102 — Foundations of Computer and Digital Forensics (Module 2: The Sleuth Kit Tutorial) | 
|Examiner | Beatrice Nduta (`kali@kali`) |  
|Environment | Kali Linux 2026.2 (`kali-linux-2026.2-vmware-amd64`), The Sleuth Kit (TSK) v4.15.0  |
|Evidence Image | `Ch01InChap01.dd` (Raw FAT12 Volume, 1,474,560 bytes)  |
|Working Directory | `/home/kali/CIP-B102-Lab02_Beatrice_Nduta`  |



---



## 1. Scenario, Governance, and Environment Setup



An investigating officer requested a command-line examination of raw image `Ch01InChap01.dd` to enumerate directory records, recover deleted text file `letter1.txt` and spreadsheet `INCOME.XLS` via metadata and sector methods, and verify integrity through cryptographic digests. All operations ran in an isolated Kali Linux environment (`/home/kali/CIP-B102-Lab02_Beatrice_Nduta`) with verified TSK v4.15.0 utilities. The source image remained read-only and unmodified inside `evidence/`, with logs and extracted artefacts written exclusively to `reports/`, `recovered/`, and `screenshots/`.



![Fig. 1: Workspace and TSK verification](screenshots/Fig_01_Working_Dir.png)



---



## 2. Evidence Register and Filesystem Geometry



Acquisition checks (`stat`, `file`, `md5sum`, `sha256sum`) matched the ICDFA reference baseline. Structural inspection via `img_stat`, `mmls`, and `fsstat` confirmed an unpartitioned raw FAT12 volume (2,880 sectors x 512 bytes; `Sectors before file system: 0`), permitting direct TSK execution without a sector offset (`-o`).



| Register / Geometry Field | Verified Observation |

| :--- | :--- |

| **Evidence Path & Size** | `evidence/Ch01InChap01.dd` — **1,474,560 bytes** (2,880 sectors; range `0–2879`) |

| **MD5 Hash** | `a117773bcf1fc88ec0ab8e0a349fbbcb` |

| **SHA-256 Hash** | `3ce8053e4f3d9c8ab98b3aadb2480685efb8e4980d34297b83bd5a09b1a7b122` |

| **FAT12 Volume Layout** | OEM: `,'k0nIHC` \| Boot: `0` \| FAT0: `1–9` \| FAT1: `10–18` \| Root Dir: `19–32` \| Clusters: `33–2879` |



![Fig. 2: Evidence hashing](screenshots/Fig_02_Evidence_Hashing.png)  

![Fig. 3: Image and filesystem examination](screenshots/Fig_03_Image_Filesystem_Exam.png)



---



## 3. Directory Enumeration and Deleted File Recovery (`letter1.txt`)



Enumeration via `fls -r` and `fls -r -d` identified six root entries (plus four TSK virtual entries), comprising two *allocated* live files (`5: Client Info.mdb`, `13: Income.xls`) and **four deleted (unallocated) entries** marked with `*`: `8: Billing Letter.doc`, `11: confirmation.txt`, `15: letter1.txt`, and `17: Regrets.doc`. Deleted FAT entries have their leading character overwritten with deletion byte `0xE5` (shown as `_` in `istat`) and their FAT chains cleared, freeing clusters for reuse while leaving sector payloads intact until overwritten.



* **Metadata (`istat Ch01InChap01.dd 15`):** `_ETTER1.TXT` (`Not Allocated`), size **121 bytes**, single sector **312** (Written `2005-12-09 06:51:50 EAT`; Created `06:59:09 EAT`).

* **Recovery & Slack Analysis:** Metadata recovery via `icat -r Ch01InChap01.dd 15 > ../recovered/letter1_icat.txt` yielded valid CRLF ASCII text (SHA-256: `341065f2c3c2168e246aa9a346439dfa4e6e4c55de4b1b8ecc78e4b83206545f`) containing a note from "George" to "Earl" requesting a meeting on August 18th. Raw inspection via `blkcat Ch01InChap01.dd 312 | xxd` demonstrated that while `icat` truncates output to the exact 121-byte file size, `blkcat` reads all 512 bytes of sector 312, exposing **391 bytes of trailing `0x00` slack padding** from offset `0x79` to `0x1ff`.



![Fig. 4: Deleted directory listing](screenshots/Fig_04_fls_Deleted_Listing.png)  

![Fig. 5: Metadata of letter1.txt](screenshots/Fig_05_letter1_Metadata.png)  

![Fig. 6: Hexadecimal sector slack inspection](screenshots/Fig_06_letter1_icat_blkcat.png)



---



## 4. Multi-Method Recovery and Validation of `INCOME.XLS`



`istat Ch01InChap01.dd 13` confirmed `INCOME.XLS` as `Allocated`, size **13,824 bytes**, occupying **27 contiguous sectors** (`285–311`). Three independent extraction methods were executed and compared using `sha256sum`, `cmp -s`, and `file`:



| Method & Command | Recovered Output Path | Size | SHA-256 Digest & Verification |

| :--- | :--- | :--- | :--- |

| **Metadata (`icat`)** | `../recovered/INCOME_icat.xls` | 13,824 B | `8d7cd7204d3dae161a8fa879e184865b3bc4a57a4e688abd522a9ff03f62252d` |

| **Sector (`blkcat` + `head`)** | `../recovered/INCOME_blkcat.xls` | 13,824 B | `8d7cd7204d3dae161a8fa879e184865b3bc4a57a4e688abd522a9ff03f62252d` (`MATCH`) |

| **Automated (`tsk_recover -e`)** | `../recovered/tsk_output/Income.xls` | 13,824 B | Identical 13,824-byte root file (6 files recovered total) |



Because `27 x 512 = 13,824` bytes, zero sector slack existed at sector 311, and `cmp -s` confirmed a bit-for-bit match. `file` verified an intact Microsoft Excel OLE2 compound document authored by "Amelia Phillips" (Created `Nov 23 2002`; Last Saved `Dec 9 2005 14:52:18`).



![Fig. 7: istat and icat extraction of INCOME.XLS](screenshots/Fig_07_INCOME_Multiple_Sectors.png)  

![Fig. 8: Sector-level extraction and byte trimming](screenshots/Fig_08_Data_Unit_Recovery.png)  

![Fig. 9: tsk_recover automated output](screenshots/Fig_09_tsk_automated_recovery.png)  

![Fig. 10: Three-way hash and cmp verification](screenshots/Fig_10_Integrity_Comparison.png)



---



## 5. Independent Utility (`ffind`), Limitations, and Conclusion



Executing `ffind Ch01InChap01.dd 13` resolved metadata address `13` to `/Income.xls` without modifying the image, confirming root-directory placement and mixed-case long-filename preservation alongside `istat` entry `13`. While `ffind` maps metadata addresses to paths (and requires `-o` on partitioned disks), it does not validate underlying sector integrity. Overall, all laboratory objectives were achieved with full evidence preservation, and every recovered file was substantiated by repeatable command-line extraction and cryptographic verification.



![Fig. 11: ffind path resolution](screenshots/Fig_11_ffind_Output.png)
