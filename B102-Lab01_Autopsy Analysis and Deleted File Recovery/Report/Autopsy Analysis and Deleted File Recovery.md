# CIP-B102 Lab 01 Report: Introduction to Digital Forensics and Autopsy Image Examination

**Student:** Beatrice Nduta
**Course:** CIP-B102 Foundations of Computer and Digital Forensics
**Institution:** International Cybersecurity and Digital Forensics Academy (ICDFA)
**Case:** CIP-B102-Lab1-Beatrice_Nduta
**Report date:** 24 November 2026

> Educational lab work on an authorised training image only. The original evidence image is not included in this repository.

---

## 1. Examiner and case details

| Field | Details |
|---|---|
| Examiner | Beatrice Nduta |
| Case number | InChap_0001 (single-user case) |
| Software | Autopsy 4.23.1, Sleuth Kit 4.15.0 |
| Examination dates | 22 to 24 November 2026 |
| Environments | Kali Linux VM (download, hashing); Windows host (Autopsy, PowerShell) |
| Time zone | Africa/Lagos (WAT, GMT+1) set in Autopsy; the Kali clock showed EAT (+0300) |
---

## Table of contents

1. [Objective](#1-objective)
2. [Environment and tools](#2-environment-and-tools)
3. [Evidence provenance and integrity](#3-evidence-provenance-and-integrity)
4. [Method](#4-method)
5. [Findings](#5-findings)
6. [Keyword analysis](#6-keyword-analysis)
7. [Autopsy report](#7-autopsy-report)
8. [Conclusion](#8-conclusion)
9. [Limitations and corrective actions](#9-limitations-and-corrective-actions)
10. [Repository structure](#10-repository-structure)
11. [Reproducing this lab](#11-reproducing-this-lab)
12. [Ethics and disclaimer](#12-ethics-and-disclaimer)

---

## 1. Objective

Apply the digital investigation workflow (identification, collection, examination, analysis and reporting) to a single training disk image, `Ch01InChap01.dd`, using the Autopsy GUI. The lab covers:

- Creating a chain-of-custody record and verifying evidence with locally computed hashes
- Creating an Autopsy case and adding a raw forensic image as a data source
- Finding, tagging and recovering deleted files without altering the original image
- Running justified keyword searches and documenting the results
- Generating an Autopsy report and preparing a professional write-up

## 2. Environment and tools

| Item | Details |
|---|---|
| Forensic software | Autopsy 4.23.1 (RELEASE), Sleuth Kit 4.15.0, Java 21.0.10 |
| Analysis host | Windows (Autopsy GUI, PowerShell hashing) |
| Linux VM | Kali Linux (download, file identification, hashing) |
| Utilities | `wget`, `file`, `ls`, `sha256sum`, `md5sum`, `chmod`, PowerShell `Get-FileHash` |
| Constraint | Sleuth Kit command-line tools (fls, icat, istat, blkcat) were **not** used; they belong to a separate lab |

<img width="674" height="785" alt="Fig 6 Autopsy version" src="https://github.com/user-attachments/assets/e0a24604-761f-4111-88b2-6dbbcf65196b" />

*Figure 6: Autopsy version information.*

<img width="298" height="49" alt="Fig 7 Time zone" src="https://github.com/user-attachments/assets/b98bcb3c-9b5b-43a7-bb17-0fbf5261d6ec" />

*Figure 7: Examiner workstation time in WAT.*

## 3. Evidence provenance and integrity

| Field | Value |
|---|---|
| Evidence file | `Ch01InChap01.dd` |
| Source | Dropbox link cited in the ICDFA course slides |
| Downloaded | 22 Nov 2026, 12:49:07 EAT (10:49 WAT), via `wget` in Kali |
| File type | DOS/MBR boot sector, FAT (12 bit) disk image, 512-byte sectors |
| Size | 1,474,560 bytes (matched in Kali, PowerShell and Autopsy) |
| Protection | `chmod a-w` applied (permissions `-r--r--r--`) |
| SHA-256 | `3ce8053e4f3d9c8ab98b3aadb2480685efb8e4980d34297b83bd5a09b1a7b122` |
| MD5 | `a117773bcf1fc88ec0ab8e0a349fbbcb` |
| Re-verification | SHA-256 recomputed on 24 Nov 2026 (18:03 WAT) after all analysis; identical to the acquisition hash |

Autopsy displayed the image MD5, SHA1 and SHA-256 as "Not calculated", so the independently computed hashes are the reference for this evidence item.

### Chain of custody (summary)

| Field | Record |
|---|---|
| Case identifier | CIP-B102-Lab1-Beatrice_Nduta |
| Examiner | Beatrice Nduta |
| Source / authority | ICDFA course slides (academic course material) |
| Kali location | `~/CIP-B102-Lab1/evidence/Ch01InChap01.dd` |
| Examined copy | `C:\Users\bnduta\Desktop\Ch01InChap01.dd` |
| Environments | Kali Linux VM (download, hashing); Windows host with Autopsy 4.23.1 |

## 4. Method

### 4.1 Workspace and acquisition (Kali)

```bash
mkdir -p ~/CIP-B102-Lab1/{evidence,outputs,reports,screenshots}
cd ~/CIP-B102-Lab1/evidence
wget -O Ch01InChap01.dd "https://www.dropbox.com/s/nw23q14vzsykyup/Ch01InChap01.dd?dl=1"
file Ch01InChap01.dd
ls -lh Ch01InChap01.dd
```

`mkdir -p` with brace expansion created the case folder and its four sub-folders. `file` identified a DOS/MBR boot sector with a 12-bit FAT file system, so the download is a genuine disk image and not an HTML error page.

<img width="593" height="401" alt="Fig 1 dir making" src="https://github.com/user-attachments/assets/fdcfb606-4d6f-4fb0-81d9-ace198923bca" />

*Figure 1: Local investigation folders created in Kali.*

<img width="1681" height="785" alt="Fig 2 Wget_Disk_Image" src="https://github.com/user-attachments/assets/1e4798b7-ac71-4d52-a51d-a443346fb0b0" />

*Figure 2: Download and identification of the training image.*

### 4.2 Hashing and protection

```bash
sha256sum Ch01InChap01.dd | tee ../reports/evidence_sha256.txt
md5sum Ch01InChap01.dd | tee ../reports/evidence_md5.txt
chmod a-w Ch01InChap01.dd
```

<img width="676" height="287" alt="Fig 3 sha256sum_mD5sum" src="https://github.com/user-attachments/assets/0e449a07-e2b3-492d-9b40-8c072176b081" />

*Figure 3: Hashing the image and removing write permission.*

<img width="702" height="322" alt="Fig 4 file details" src="https://github.com/user-attachments/assets/baed8894-73e8-4ec9-bef9-a3b28d14602e" />

*Figure 4: File details and saved hash records.*

### 4.3 Autopsy case

1. Created case `CIP-B102-Lab1-Beatrice_Nduta` (22 Nov 2026, 18:26 EAT) with the case directory outside the evidence folder.
2. Added `Ch01InChap01.dd` as a data source and set the display time zone to **Africa/Lagos**.
3. First ingest was cancelled: the Cyber Triage Malware Scanner had no licence. The module was disabled and the image was ingested again.
4. The first keyword search failed ("No files were indexed"). The image was re-ingested with the Keyword Search module and Solr indexing enabled.
5. Tagged files, exported two Office documents with Extract File(s), ran keyword searches and generated an HTML report.

<img width="475" height="572" alt="Fig 5 New case details " src="https://github.com/user-attachments/assets/48335cf5-aac8-4975-b536-2ba7cd8ac2bb" />

*Figure 5: Properties of the case folder on the Windows Desktop.*

<img width="1091" height="649" alt="Fig 8 Autopsy case details" src="https://github.com/user-attachments/assets/bf92076a-e7b6-4bc2-a409-c90e111c0ea7" />

*Figure 8: Autopsy case details (contact details redacted).*

<img width="362" height="115" alt="Fig 9 WAT timezone change" src="https://github.com/user-attachments/assets/e40f1f6d-b214-4d4b-b77c-1338a55d3dbf" />

*Figure 9: Display time zone set to Africa/Lagos (WAT).*

<img width="1902" height="1063" alt="Fig 10 error" src="https://github.com/user-attachments/assets/f659480e-9617-40cd-b654-71aaf021cf98" />

*Figure 10: Ingest module startup failure (Cyber Triage licence).*

### 4.4 Verifying the imported evidence

The data source was listed with Raw Single type, 1,474,560 bytes and 512-byte sectors. Autopsy showed MD5, SHA1 and SHA-256 as "Not calculated", and the file size matched the independent PowerShell check.

<img width="1352" height="765" alt="Fig 11 file system contents" src="https://github.com/user-attachments/assets/8fc282dc-65ae-4bd8-9b86-74578a01b2e9" />

*Figure 11: Autopsy file views of the image.*

<img width="821" height="607" alt="Fig 12 file metadata" src="https://github.com/user-attachments/assets/4d543c15-82c1-47de-97b4-7ca24eb53083" />

*Figure 12: File metadata of Regrets.doc.*

<img width="890" height="578" alt="Fig 13 case study metadata" src="https://github.com/user-attachments/assets/b8f528bb-adde-4f27-9de7-b82f232a4ae9" />

*Figure 13: Data source metadata for the image.*

<img width="1003" height="177" alt="Fig 14 File size verification" src="https://github.com/user-attachments/assets/5fb12bc0-87d3-4d41-aca0-a3467c65415a" />

*Figure 14: Independent size check in PowerShell.*

### 4.5 Changes in counts after re-ingest

| Item | Before | After |
|---|---|---|
| Office files | 3 | 5 |
| Deleted Files, All | 4 | 6 |
| Deleted Files, File System | 4 | 4 |
| Keyword hits | none | 23 (44 after all five searches) |

The extra Office and deleted items after re-ingest come from carved data in unallocated space; file-system deleted entries stayed at 4.

## 5. Findings

Four deleted file-system entries were identified (Dir and Meta flags Unallocated). Timestamps are shown in WAT.

| File | Size (bytes) | Modified | Observation |
|---|---|---|---|
| `Billing Letter.doc` | 24,064 | 2005-12-09 06:50:28 | Word document; author Amelia Phillips, company Starships CMS. Letter from George Montgomery about domain registration (USD 500 fee). **Exported.** |
| `Regrets.doc` | 23,552 | 2005-12-09 06:50:52 | Word letter dated 02 Nov 2005 to Randall Watson stating that all five requested domain names were already purchased. **Exported.** |
| `confirmation.txt` | 227 | 2005-12-09 06:52:58 | Message to "Laura" confirming a domain registration and giving FTP access details (sensitive; redacted in the screenshot). Tagged. |
| `letter1.txt` | 121 | 2005-12-09 06:51:50 | Small deleted plain-text file. Tagged. |

`Income.xls` is an allocated (not deleted) Office file.

<img width="1350" height="762" alt="Fig 15 Office docs deleted files view" src="https://github.com/user-attachments/assets/ab95f832-5f0f-4edd-9f34-030c5acd49e3" />

*Figure 15: Office documents with extracted text.*

<img width="1361" height="765" alt="Fig 16 plain text file metadata" src="https://github.com/user-attachments/assets/2f088ed5-c7c3-423d-8811-1936801c8175" />

*Figure 16: Deleted plain text files (credentials redacted).*

<img width="1916" height="1068" alt="Fig 17 Billing letter file details" src="https://github.com/user-attachments/assets/62041207-50a4-432b-8633-ba3f7f8ffb1e" />
*Figure 17: Billing Letter.doc details and extracted metadata.*

<img width="1343" height="365" alt="Fig 18 file tagging" src="https://github.com/user-attachments/assets/6a8ac333-b9f2-4189-904a-6397acd8a008" />

*Figure 18: Deletion flags for the Office files.*

### 5.1 Tagging and export

Four deleted files were tagged **Recovered file**. Two deleted Office documents (`Billing Letter.doc` and `Regrets.doc`) were exported with Autopsy's Extract File(s) function to the case Export folder. Exported sizes (24,064 and 23,552 bytes) matched Autopsy.

| Exported file | SHA-256 |
|---|---|
| `Billing Letter.doc` | `7EEFEB4187BB551F366DA33BCAB5B7224F905BB98F37797C28F183F72AEC2E99` |
| `Regrets.doc` | `109048540A0A1A452F9346522A325219B471EAE41B85A6298890E94AE750B965` |

<img width="1918" height="970" alt="Fig 19 File extract and hashing" src="https://github.com/user-attachments/assets/09d7526a-80de-4cdc-a384-cada1abc7975" />

*Figure 19: Exported files and their SHA-256 hashes.*

<img width="1346" height="676" alt="Fig 20 tags" src="https://github.com/user-attachments/assets/b68bdf83-51e1-4971-98cf-3d0947f4be9a" />

*Figure 20: Deleted files tagged "Recovered file".*

## 6. Keyword analysis

| Term | Justification | Mode | Result |
|---|---|---|---|
| George | Term used in the slides; signer of the letters | Exact | 9 hits (including `Billing Letter.doc`, `confirmation.txt` and a carved fragment) |
| Montgomery | Surname of the signer | Exact | Included in the 44 total hits |
| Watson | Recipient, Randall Watson | Exact | Included in the 44 total hits |
| domain | Subject of the letters (domain names) | Substring | 7 hits; fragment in `unalloc_4_121344_1474560` |
| nowhere.com | Sender email domain; could link other files | Substring | Included in the 44 total hits |

Hits were tagged as **Keyword Hit - Finding** (supports a finding) or **Keyword Hit - Lead** (needs follow-up). The carved fragment (a letter dated 13 October 2005 to Laura Roper) has no filename, path or timestamps, so it supports the content of the letter but not its origin or timing.

<img width="1343" height="727" alt="Fig 21 error" src="https://github.com/user-attachments/assets/39593bac-4c86-4e28-894e-e7f630dabbf5" />

*Figure 21: Keyword search error before re-ingest.*

<img width="1352" height="682" alt="Fig 22 Keyword search George" src="https://github.com/user-attachments/assets/21d2278b-240c-45bf-9b88-7450447a0c53" />

*Figure 22: Keyword search for "George".*

<img width="1340" height="760" alt="Fig 23 Keyword Domain" src="https://github.com/user-attachments/assets/5c9fa8a3-7418-4c61-aaa7-2c853c0f4c85" />

*Figure 23: Keyword search for "domain".*

## 7. Autopsy report

| Item | Value |
|---|---|
| Generated | 24 Nov 2026, 19:43:27 (system clock, EAT) |
| Contents | 44 keyword hits, 6 metadata items, 10 tagged files |
| `report.html` SHA-256 | `E7D38D6ECBA31A3419607462850FFBBE3127322F96E353B58F62E41BFCFD6A68` |

<img width="1356" height="752" alt="Fig 24 Report" src="https://github.com/user-attachments/assets/f49de9e7-a57c-4b80-b614-785e6e2376e7" />

*Figure 24: Autopsy HTML report generated.*

<img width="1675" height="970" alt="Fig 25 Autopsy report" src="https://github.com/user-attachments/assets/a9e40b36-80e2-4200-bb35-42325f664232" />

*Figure 25: Autopsy Forensic Report summary.*

<img width="1897" height="381" alt="Fig 26 Full report path" src="https://github.com/user-attachments/assets/1d0612f6-fa00-4a01-9ea6-2fdc18b0bbe2" />

*Figure 26: Full path and SHA-256 hash of the report file.*

## 8. Conclusion

The image contains four deleted file-system entries (two Word letters and two text files) that appear to document a domain-name registration involving George Montgomery, Laura Roper and Randall Watson. The deleted documents were recovered and hashed, relevant keyword hits were tagged, and the original image was verified as unchanged by re-hashing, so the examination is repeatable. These are observations from training data, not proof of any real offence.

## 9. Limitations and corrective actions

1. The first ingest failed on an unlicensed module; corrected by disabling it and re-ingesting.
2. The first keyword search failed until Solr indexing was enabled on re-ingest.
3. Autopsy did not calculate image hashes, so independent hashes were used.
4. The file system records no deletion time, so when files were deleted cannot be stated.
5. Timestamps are interpreted in Africa/Lagos while the Kali clock used EAT.

## 10. Repository structure

```text
CIP-B102-Lab1-Beatrice_Nduta/
├── README.md                 # this file
└── screenshots/              # Figures 1 to 26 (contact details and credentials redacted)
    ├── fig01_folder_structure.png
    ├── fig02_download_and_file_identification.png
    └── ...
```

> The original image `Ch01InChap01.dd` is **not** included. Do not commit it unless your instructor explicitly requests it.

## 11. Reproducing this lab

1. Obtain the authorised image from the ICDFA course materials and verify its SHA-256 against the value in section 3.
2. Set the image read-only and keep it outside the case and output folders.
3. Create an Autopsy case, add the image and set the display time zone to Africa/Lagos.
4. Enable file type identification and keyword search with Solr indexing; disable any module that lacks a licence.
5. Locate deleted files, tag and export them, then compute SHA-256 hashes of the exports.
6. Run the keyword searches in section 6, generate the HTML report and re-hash the original image.

## 12. Ethics and disclaimer

- This work was carried out on an authorised training image in an offline lab environment.
- The original evidence was never modified or uploaded to third-party services.
- Personal contact details (Figure 8) and login details from a training file (Figure 16) are redacted in the uploaded screenshots.
- This repository is for educational purposes only.
