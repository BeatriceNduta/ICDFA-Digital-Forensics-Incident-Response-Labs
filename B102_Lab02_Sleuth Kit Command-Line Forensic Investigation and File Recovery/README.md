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



<img width="1573" height="717" alt="Fig 1 working Dir" src="https://github.com/user-attachments/assets/b97286b5-9896-4cad-bba6-a8782b2e493c" />



---



## 2. Evidence Register and Filesystem Geometry



Acquisition checks (`stat`, `file`, `md5sum`, `sha256sum`) matched the ICDFA reference baseline. Structural inspection via `img_stat`, `mmls`, and `fsstat` confirmed an unpartitioned raw FAT12 volume (2,880 sectors x 512 bytes; `Sectors before file system: 0`), permitting direct TSK execution without a sector offset (`-o`).



| Register / Geometry Field | Verified Observation |

| :--- | :--- |

| **Evidence Path & Size** | `evidence/Ch01InChap01.dd` — **1,474,560 bytes** (2,880 sectors; range `0–2879`) |

| **MD5 Hash** | `a117773bcf1fc88ec0ab8e0a349fbbcb` |

| **SHA-256 Hash** | `3ce8053e4f3d9c8ab98b3aadb2480685efb8e4980d34297b83bd5a09b1a7b122` |

| **FAT12 Volume Layout** | OEM: `,'k0nIHC` \| Boot: `0` \| FAT0: `1–9` \| FAT1: `10–18` \| Root Dir: `19–32` \| Clusters: `33–2879` |



<img width="1356" height="413" alt="Fig 2 Evidence file Download   Hashing" src="https://github.com/user-attachments/assets/00adbd59-5545-4c37-8c76-0512a876e86e" />

<img width="1363" height="708" alt="Fig 3 Image   Filesys examination" src="https://github.com/user-attachments/assets/fbc2370f-eddc-41cd-85cb-3999c05546d6" />



---



## 3. Directory Enumeration and Deleted File Recovery (`letter1.txt`)



Enumeration via `fls -r` and `fls -r -d` identified six root entries (plus four TSK virtual entries), comprising two *allocated* live files (`5: Client Info.mdb`, `13: Income.xls`) and **four deleted (unallocated) entries** marked with `*`: `8: Billing Letter.doc`, `11: confirmation.txt`, `15: letter1.txt`, and `17: Regrets.doc`. Deleted FAT entries have their leading character overwritten with deletion byte `0xE5` (shown as `_` in `istat`) and their FAT chains cleared, freeing clusters for reuse while leaving sector payloads intact until overwritten.



* **Metadata (`istat Ch01InChap01.dd 15`):** `_ETTER1.TXT` (`Not Allocated`), size **121 bytes**, single sector **312** (Written `2005-12-09 06:51:50 EAT`; Created `06:59:09 EAT`).

* **Recovery & Slack Analysis:** Metadata recovery via `icat -r Ch01InChap01.dd 15 > ../recovered/letter1_icat.txt` yielded valid CRLF ASCII text (SHA-256: `341065f2c3c2168e246aa9a346439dfa4e6e4c55de4b1b8ecc78e4b83206545f`) containing a note from "George" to "Earl" requesting a meeting on August 18th. Raw inspection via `blkcat Ch01InChap01.dd 312 | xxd` demonstrated that while `icat` truncates output to the exact 121-byte file size, `blkcat` reads all 512 bytes of sector 312, exposing **391 bytes of trailing `0x00` slack padding** from offset `0x79` to `0x1ff`.



<img width="1188" height="692" alt="Fig 4 fls deleted file listing " src="https://github.com/user-attachments/assets/5b46f42a-2273-420d-9bb7-c6b672ce1041" />

<img width="1007" height="450" alt="Fig 5 letter1 deleted file meta" src="https://github.com/user-attachments/assets/82b3a456-4282-42cd-97bf-8d41887a4780" /> 

<img width="1365" height="703" alt="Fig 6 evidence file display using icat   blkcat" src="https://github.com/user-attachments/assets/9dcf8fbb-f169-46f1-86d9-7ee3f5d53b99" />



---



## 4. Multi-Method Recovery and Validation of `INCOME.XLS`



`istat Ch01InChap01.dd 13` confirmed `INCOME.XLS` as `Allocated`, size **13,824 bytes**, occupying **27 contiguous sectors** (`285–311`). Three independent extraction methods were executed and compared using `sha256sum`, `cmp -s`, and `file`:



| Method & Command | Recovered Output Path | Size | SHA-256 Digest & Verification |

| :--- | :--- | :--- | :--- |

| **Metadata (`icat`)** | `../recovered/INCOME_icat.xls` | 13,824 B | `8d7cd7204d3dae161a8fa879e184865b3bc4a57a4e688abd522a9ff03f62252d` |

| **Sector (`blkcat` + `head`)** | `../recovered/INCOME_blkcat.xls` | 13,824 B | `8d7cd7204d3dae161a8fa879e184865b3bc4a57a4e688abd522a9ff03f62252d` (`MATCH`) |

| **Automated (`tsk_recover -e`)** | `../recovered/tsk_output/Income.xls` | 13,824 B | Identical 13,824-byte root file (6 files recovered total) |



Because `27 x 512 = 13,824` bytes, zero sector slack existed at sector 311, and `cmp -s` confirmed a bit-for-bit match. `file` verified an intact Microsoft Excel OLE2 compound document authored by "Amelia Phillips" (Created `Nov 23 2002`; Last Saved `Dec 9 2005 14:52:18`).



<img width="1337" height="697" alt="Fig 7 file multiple sectors" src="https://github.com/user-attachments/assets/cf5174d7-5cf4-41a9-9516-c0353ced7cb2" />

<img width="1363" height="321" alt="Fig 8 data unit recovery method " src="https://github.com/user-attachments/assets/72403c21-2700-4b31-aab5-f57d106c6ab4" />

<img width="1361" height="696" alt="Fig 9 tsk automated recovery method" src="https://github.com/user-attachments/assets/9eef1c98-d8bb-44c0-931f-edd92981d436" /> 

<img width="1362" height="728" alt="Fig 10 file Integrity and content comparison" src="https://github.com/user-attachments/assets/641833e2-dbed-4f21-bfd9-398348b0df3a" />



---



## 5. Independent Utility (`ffind`), Limitations, and Conclusion



Executing `ffind Ch01InChap01.dd 13` resolved metadata address `13` to `/Income.xls` without modifying the image, confirming root-directory placement and mixed-case long-filename preservation alongside `istat` entry `13`. While `ffind` maps metadata addresses to paths (and requires `-o` on partitioned disks), it does not validate underlying sector integrity. Overall, all laboratory objectives were achieved with full evidence preservation, and every recovered file was substantiated by repeatable command-line extraction and cryptographic verification.



<img width="1365" height="706" alt="Fig 11 ffind output" src="https://github.com/user-attachments/assets/bce81e77-56c6-4a8c-af29-c49c9a5b48c6" />
