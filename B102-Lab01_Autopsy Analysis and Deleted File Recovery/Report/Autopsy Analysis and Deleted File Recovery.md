# CIP-B102 Lab 01: Introduction to Digital Forensics and Autopsy Image Examination

**Student:** Beatrice Nduta
**Course:** CIP-B102 Foundations of Computer and Digital Forensics
**Institution:** International Cybersecurity and Digital Forensics Academy (ICDFA)
**Case:** CIP-B102-Lab1-Beatrice_Nduta (case number InChap_0001)
**Dates:** 22 to 24 November 2026
**Time zone used for interpretation:** Africa/Lagos (WAT, GMT+1)

> Educational lab work on an authorised training image only. The original evidence image is not included in this repository.

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

![Figure 6: Autopsy version information](screenshots/fig06_autopsy_version.png)

*Figure 6: Autopsy version information.*

![Figure 7: Examiner workstation time in WAT](screenshots/fig07_workstation_time_wat.png)

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

![Figure 1: Local investigation folders created in Kali](screenshots/fig01_folder_structure.png)

*Figure 1: Local investigation folders created in Kali.*

![Figure 2: Download and identification of the training image](screenshots/fig02_download_and_file_identification.png)

*Figure 2: Download and identification of the training image.*

### 4.2 Hashing and protection

```bash
sha256sum Ch01InChap01.dd | tee ../reports/evidence_sha256.txt
md5sum Ch01InChap01.dd | tee ../reports/evidence_md5.txt
chmod a-w Ch01InChap01.dd
```

![Figure 3: Hashing the image and removing write permission](screenshots/fig03_hashing_and_write_protection.png)

*Figure 3: Hashing the image and removing write permission.*

![Figure 4: File details and saved hash records](screenshots/fig04_file_details_and_hash_records.png)

*Figure 4: File details and saved hash records.*

### 4.3 Autopsy case

1. Created case `CIP-B102-Lab1-Beatrice_Nduta` (22 Nov 2026, 18:26 EAT) with the case directory outside the evidence folder.
2. Added `Ch01InChap01.dd` as a data source and set the display time zone to **Africa/Lagos**.
3. First ingest was cancelled: the Cyber Triage Malware Scanner had no licence. The module was disabled and the image was ingested again.
4. The first keyword search failed ("No files were indexed"). The image was re-ingested with the Keyword Search module and Solr indexing enabled.
5. Tagged files, exported two Office documents with Extract File(s), ran keyword searches and generated an HTML report.

![Figure 5: Properties of the case folder on the Windows Desktop](screenshots/fig05_case_folder_properties.png)

*Figure 5: Properties of the case folder on the Windows Desktop.*

![Figure 8: Autopsy case details (contact details redacted)](screenshots/fig08_autopsy_case_details_redacted.png)

*Figure 8: Autopsy case details (contact details redacted).*

![Figure 9: Display time zone set to Africa/Lagos (WAT)](screenshots/fig09_timezone_africa_lagos.png)

*Figure 9: Display time zone set to Africa/Lagos (WAT).*

![Figure 10: Ingest module startup failure (Cyber Triage licence)](screenshots/fig10_ingest_module_error.png)

*Figure 10: Ingest module startup failure (Cyber Triage licence).*

### 4.4 Verifying the imported evidence

The data source was listed with Raw Single type, 1,474,560 bytes and 512-byte sectors. Autopsy showed MD5, SHA1 and SHA-256 as "Not calculated", and the file size matched the independent PowerShell check.

![Figure 11: Autopsy file views of the image](screenshots/fig11_file_views_deleted_files.png)

*Figure 11: Autopsy file views of the image.*

![Figure 12: File metadata of Regrets.doc](screenshots/fig12_regrets_doc_metadata.png)

*Figure 12: File metadata of Regrets.doc.*

![Figure 13: Data source metadata for the image](screenshots/fig13_data_source_metadata.png)

*Figure 13: Data source metadata for the image.*

![Figure 14: Independent size check in PowerShell](screenshots/fig14_powershell_size_check.png)

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

![Figure 15: Office documents with extracted text](screenshots/fig15_office_documents_text.png)

*Figure 15: Office documents with extracted text.*

![Figure 16: Deleted plain text files (credentials redacted)](screenshots/fig16_deleted_text_files_redacted.png)

*Figure 16: Deleted plain text files (credentials redacted).*

![Figure 17: Billing Letter.doc details and extracted metadata](screenshots/fig17_billing_letter_metadata.png)

*Figure 17: Billing Letter.doc details and extracted metadata.*

![Figure 18: Deletion flags for the Office files](screenshots/fig18_deletion_flags.png)

*Figure 18: Deletion flags for the Office files.*

### 5.1 Tagging and export

Four deleted files were tagged **Recovered file**. Two deleted Office documents (`Billing Letter.doc` and `Regrets.doc`) were exported with Autopsy's Extract File(s) function to the case Export folder. Exported sizes (24,064 and 23,552 bytes) matched Autopsy.

| Exported file | SHA-256 |
|---|---|
| `Billing Letter.doc` | `7EEFEB4187BB551F366DA33BCAB5B7224F905BB98F37797C28F183F72AEC2E99` |
| `Regrets.doc` | `109048540A0A1A452F9346522A325219B471EAE41B85A6298890E94AE750B965` |

![Figure 19: Exported files and their SHA-256 hashes](screenshots/fig19_exported_files_hashes.png)

*Figure 19: Exported files and their SHA-256 hashes.*

![Figure 20: Deleted files tagged "Recovered file"](screenshots/fig20_recovered_file_tags.png)

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

![Figure 21: Keyword search error before re-ingest](screenshots/fig21_keyword_search_error.png)

*Figure 21: Keyword search error before re-ingest.*

![Figure 22: Keyword search for "George"](screenshots/fig22_keyword_george.png)

*Figure 22: Keyword search for "George".*

![Figure 23: Keyword search for "domain"](screenshots/fig23_keyword_domain.png)

*Figure 23: Keyword search for "domain".*

## 7. Autopsy report

| Item | Value |
|---|---|
| Generated | 24 Nov 2026, 19:43:27 (system clock, EAT) |
| Contents | 44 keyword hits, 6 metadata items, 10 tagged files |
| `report.html` SHA-256 | `E7D38D6ECBA31A3419607462850FFBBE3127322F96E353B58F62E41BFCFD6A68` |

![Figure 24: Autopsy HTML report generated](screenshots/fig24_report_generated.png)

*Figure 24: Autopsy HTML report generated.*

![Figure 25: Autopsy Forensic Report summary](screenshots/fig25_autopsy_html_report.png)

*Figure 25: Autopsy Forensic Report summary.*

![Figure 26: Full path and SHA-256 hash of the report file](screenshots/fig26_report_path_and_hash.png)

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
