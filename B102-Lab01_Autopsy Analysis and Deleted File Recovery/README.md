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

## 2. Authority and scope

This individual practical examined one authorised training image, `Ch01InChap01.dd`, supplied through the ICDFA course slides. The scenario asked me to identify relevant office documents, investigate deleted items, run justified keyword searches and produce a repeatable report. The work was offline, used the Autopsy GUI and local hash utilities only (no Sleuth Kit command-line tools), and kept observations separate from assumptions.

**Evidence handling declaration:** I worked only on the authorised image, set the original read-only, saved all exports and reports outside the image, and confirmed by re-hashing after analysis that the image was not altered.

## 3. Image provenance and integrity

| Field | Value |
|---|---|
| Source | Dropbox link cited in the ICDFA course slides; downloaded with `wget` in Kali on 22 Nov 2026, 12:49:07 EAT (10:49 WAT) |
| File and type | `Ch01InChap01.dd`: DOS/MBR boot sector, FAT (12 bit) disk image; 512-byte sectors |
| Size | 1,474,560 bytes; matched in Kali (`ls`), PowerShell (`Get-Item`) and Autopsy |
| Protection | `chmod a-w` applied (permissions `-r--r--r--`); examined copy kept at `C:\Users\bnduta\Desktop\Ch01InChap01.dd` |
| SHA-256 (acquisition) | `3ce8053e4f3d9c8ab98b3aadb2480685efb8e4980d34297b83bd5a09b1a7b122` |
| MD5 (acquisition) | `a117773bcf1fc88ec0ab8e0a349fbbcb` |
| Re-verification | SHA-256 recomputed after all analysis; identical to the acquisition value. Autopsy showed image hashes as "Not calculated", so independent hashes were the reference. |

```bash
sha256sum Ch01InChap01.dd | tee ../reports/evidence_sha256.txt
md5sum Ch01InChap01.dd | tee ../reports/evidence_md5.txt
chmod a-w Ch01InChap01.dd
```

<img width="676" height="287" alt="Fig 3 sha256sum_mD5sum" src="https://github.com/user-attachments/assets/6c37ad44-43ac-4f13-8c85-b53a534a4143" />


*Figure 1: Hashes of the image taken in Kali, then write permission removed.*

## 4. Autopsy method

I created the case `CIP-B102-Lab1-Beatrice_Nduta` (22 Nov 2026, 18:26 EAT), added the raw image as a data source and set the display time zone to Africa/Lagos.

1. The first ingest was cancelled because the Cyber Triage Malware Scanner had no licence, so I disabled that module and re-ingested with file-type identification and keyword indexing.
2. A first keyword search then failed ("No files were indexed"), so I re-ingested with the Keyword Search module and Solr indexing enabled.
3. After re-ingest, Office files rose from 3 to 5 and Deleted Files (All) from 4 to 6, while Deleted Files (File System) stayed at 4; the extra items are carved from unallocated space.
4. I then tagged items, exported files, ran keyword searches and generated an HTML report.

## 5. Findings

| File (deleted) | Size (B) | Modified (WAT) | Status | Observation |
|---|---|---|---|---|
| `Billing Letter.doc` | 24,064 | 2005-12-09 06:50:28 | Name and metadata unallocated | Word document; author Amelia Phillips, company Starships CMS. Letter from George Montgomery about domain registration (USD 500 fee). **Exported.** SHA-256 `7EEFEB4187BB551F366DA33BCAB5B7224F905BB98F37797C28F183F72AEC2E99` |
| `Regrets.doc` | 23,552 | 2005-12-09 06:50:52 | Name and metadata unallocated | Word letter dated 02 Nov 2005 to Randall Watson saying all five requested domain-name versions were already purchased. **Exported.** SHA-256 `109048540A0A1A452F9346522A325219B471EAE41B85A6298890E94AE750B965` |
| `confirmation.txt` | 227 | 2005-12-09 06:52:58 | Deleted | Message to "Laura" confirming a domain registration and giving an FTP address and login details (sensitive; not reproduced here). Tagged. |
| `letter1.txt` | 121 | 2005-12-09 06:51:50 | Deleted | Small plain-text file listed with the deleted entries. Tagged. |

**Exports:** two exports succeeded (Extract File(s) to the case Export folder); sizes matched Autopsy (24,064 and 23,552 bytes). Four deleted files were tagged "Recovered file". `Income.xls` is an allocated (not deleted) Office file.

<img width="1352" height="765" alt="Fig 11 file system contents" src="https://github.com/user-attachments/assets/a4a614af-0e59-4b35-b65f-fce6b687a678" />


*Figure 2: Autopsy file views: 4 deleted file-system entries listed.*

<img width="1918" height="970" alt="Fig 19 File extract and hashing" src="https://github.com/user-attachments/assets/a1f3e3c3-a0dd-40a5-b8b3-8da6f69d5002" />

*Figure 3: Exported Office files and their SHA-256 hashes (PowerShell).*

<img width="1675" height="970" alt="Fig 25 Autopsy report" src="https://github.com/user-attachments/assets/554280a7-1145-4604-8b4b-6dc49c1af2e1" />


*Figure 4: Autopsy HTML report: 44 keyword hits, tagged files.*

## 6. Keyword analysis

| Term | Justification | Mode | Result |
|---|---|---|---|
| George | Term used in the slides; signer of the letters | Exact | 9 hits, including `Billing Letter.doc`, `confirmation.txt` and a carved fragment |
| Montgomery | Surname of the signer | Exact | Included in the 44 total hits |
| Watson | Recipient, Randall Watson | Exact | Included in the 44 total hits |
| domain | Subject of the letters (domain names) | Substring | 7 hits; fragment in `unalloc_4_121344_1474560` (letter of 13 Oct 2005 to Laura Roper) |
| nowhere.com | Sender email domain; could link other files | Substring | Included in the 44 total hits |

Hits were tagged as **Keyword Hit - Finding** (supports a finding) or **Keyword Hit - Lead** (needs follow-up). The carved fragment has no filename, path or timestamps, so it supports the content of the letter but not its origin or timing.

<img width="1352" height="682" alt="Fig 22 Keyword search George" src="https://github.com/user-attachments/assets/cd50003f-92bb-436d-8539-0f32f941cf3d" />


*Figure 5: Keyword search for "George": 9 results across files and unallocated space.*

<img width="1340" height="760" alt="Fig 23 Keyword Domain" src="https://github.com/user-attachments/assets/c87a3f70-c0cc-4912-9f88-fc006775aa80" />


*Figure 6: Keyword "domain": carved letter fragment from unallocated space.*

## 7. Conclusion and limitations

The image holds four deleted file-system entries (two Word letters and two text files) that together appear to document a domain-name registration between George Montgomery, Laura Roper and Randall Watson. The deleted documents were recovered and hashed, relevant keyword hits were tagged, and the original image was unchanged, so the examination is repeatable. These are observations from training data, not proof of any real offence.

**Limitations and corrective actions**

1. The first ingest failed on an unlicensed module, corrected by disabling it and re-ingesting.
2. The first keyword search failed until Solr indexing was enabled on re-ingest.
3. Autopsy did not calculate image hashes, so independent hashes were used.
4. The file system records no deletion time, so when files were deleted cannot be stated.
5. Timestamps are interpreted in Africa/Lagos while the Kali clock used EAT.

## 8. Supporting evidence

| Item | Value |
|---|---|
| Autopsy HTML report | `C:\Users\bnduta\Desktop\CIP-B102-Lab1-Beatrice_Nduta\Reports\...HTML Report 11-24-2026-19-43-27\report.html` |
| Generated | 24 Nov 2026, 19:43 EAT (17:43 WAT) |
| `report.html` SHA-256 | `E7D38D6ECBA31A3419607462850FFBBE3127322F96E353B58F62E41BFCFD6A68` |
| Exported files | Case Export folder (`Billing Letter.doc`, `Regrets.doc`) |

## Repository structure

```text
CIP-B102-Lab1-Beatrice_Nduta/
├── README.md        # this report
└── screenshots/     # figures used in this report
```

> The original image `Ch01InChap01.dd` is **not** included. Do not commit it unless your instructor explicitly requests it.

## Ethics and disclaimer

- This work was carried out on an authorised training image in an offline lab environment.
- The original evidence was never modified or uploaded to third-party services.
- This repository is for educational purposes only.
