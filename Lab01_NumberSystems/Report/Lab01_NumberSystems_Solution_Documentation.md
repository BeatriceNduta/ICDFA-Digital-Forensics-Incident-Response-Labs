# CIP-B101 Lab 1A: Number Systems, ASCII, Timestamps and Data Representation

*International Cybersecurity and Digital Forensics Academy (ICDFA) | Basic Computer Skills for Digital Forensics*

| Student | Student ID | Platform | Date |
| --- | --- | --- | --- |
| Beatrice Nduta | 17016 | Kali Linux (VMware), Bash and Python 3.13 | 3 November 2026 |

> Report: [`CIP-B101_Lab1A_Beatrice_Nduta_2-Page_Report.pdf`](CIP-B101_Lab1A_Beatrice_Nduta_2-Page_Report.pdf)

## 1. Objective and Method

This lab builds the skills a digital forensics trainee needs to read raw evidence correctly: number bases, text encodings, timestamps, hashes and byte order. Each value was converted manually, then verified in the Kali terminal using shell arithmetic, `bc`, `printf`, `xxd`, `date` and Python. The screenshots in the lab worksheet show my student ID (17016) in the working folder path as evidence that the outputs are my own.

## 2. Task A: Number Systems

Hexadecimal is used in forensics because one byte is exactly two hex digits, making memory, sector and file-header values compact and readable. Binary shows the underlying bits and hex is a shorthand for them (4 bits = 1 hex digit).

| Value | Base | Result | Manual method | Verification |
| --- | --- | --- | --- | --- |
| `101010` | Binary | **42** | 32 + 8 + 2 (bit weights 32, 8, 2 set) | `echo $((2#101010))` |
| `AB2` | Hex | **2738** | (10×256) + (11×16) + 2 = 2560 + 176 + 2 | `echo $((16#AB2))` |
| `126` | Decimal | **1111110** | Repeated division by 2; remainders read upward | `echo "obase=2;126" \| bc` |
| `2738` | Decimal | **AB2** | 2738÷16 = 171 r2; 171÷16 = 10 r11; 10 → A, 11 → B | `printf "%X" 2738` |
| `101010110010` | Binary | **AB2** | Groups 1010 \| 1011 \| 0010 = A \| B \| 2 | `printf '%X\n' $((2#101010110010))` |

All manual answers matched the Bash and Python outputs (`int()`, `bin()`, `hex()`).

## 3. Task B: ASCII, UTF-8 and Hex Evidence

I created `beatrice_lab1a.txt` with `echo -n "Digital Forensics Begins With Bytes"`. The `xxd` view began `4469 6769 7461 6c20` ("Digital "), and `xxd -p` produced the plain hex string, which `xxd -r -p` converted back to the original sentence. In the text evidence exercise, "Hello" was stored as `48 65 6c 6c 6f` (plain hex `48656c6c6f`) and "Betty" as `42 65 74 74 79`, with no `0a` byte because `echo -n` was used. Python confirmed the same mapping: `ord('H') = 72`, `chr(72) = 'H'`.

- **Why `0A` appears:** `echo` without `-n` appends a newline byte (`0x0A`). It is invisible as text but is real data, so `-n` is used when exact content matters.
- **Not all hex is text:** bytes may be images, compressed or encrypted data, or binary structures such as timestamps and offsets. Decoding everything as ASCII can produce false conclusions, so the context and file type must be identified first.

## 4. Task C: Timestamps (all results in UTC)

| Value | Format | Command | Readable UTC result |
| --- | --- | --- | --- |
| `1643643491` | Unix seconds | `date -u -d @1643643491` | Mon Jan 31 03:38:11 PM UTC 2022 |
| `1675008832` | Unix seconds | `date -u -d @1675008832` | Sun Jan 29 04:13:52 PM UTC 2023 |
| `725846400` | Mac Absolute Time | `date -u -d @$((725846400 + 978307200))` | Tue Jan 2 12:00:00 AM UTC 2024 |
| Current time | Unix seconds | `now=$(date +%s); date -u -d @$now` | Tue Nov 3 09:52:57 PM UTC 2026 |

Unix time counts seconds from 1970-01-01 and Mac Absolute Time from 2001-01-01; the offset is **978307200 s** (Python confirmed this). My local time zone is EAT (UTC+3): `date` showed `Wed Nov 4 12:17:34 AM EAT 2026` while `date -u` showed `Tue Nov 3 09:17:53 PM UTC 2026`, a three-hour gap that even changes the calendar day. Reading the Mac value `725846400` as Unix time would give 1 January 1993, about 31 years too early, so every result above is labelled UTC.

## 5. Task D: Hashing and the Newline Problem

`echo "Hello"` writes `48 65 6c 6c 6f 0a` (6 bytes) while `echo -n "Hello"` writes only `48 65 6c 6c 6f` (5 bytes). Hashes are byte-sensitive, so one extra byte gives a completely different MD5 and SHA-256, which is the avalanche effect.

| File | MD5 | SHA-256 (start…end) |
| --- | --- | --- |
| Hello, with newline | `09f7e02f1290be211da707a266f153b3` | `66a045b4…c1bb35f18` |
| Hello, without newline | `8b1a9953c4611296a827abf8c47804d7` | `185f8db3…6381969` |
| Beatrice, with newline (`0a`) | `489e78107340a536d88dff942d2ddb18` | `1d4f0740…582d90c6` |
| Beatrice, without newline | `ab3aec6d954571c7551a186ea1cd98ff` | `1075bbd8…f5ade9fe` |

To hash exactly the visible characters I would use `echo -n` (or `printf` without `\n`), for example:

```bash
echo -n "beatrice" | sha256sum
```

This matters because a hash mismatch in evidence verification would wrongly suggest the data was altered.

## 6. Task E: Endianness

My system byte order is **little-endian** (`sys.byteorder`). Little-endian stores the least significant byte first, while big-endian stores the most significant byte first, so the same four bytes give different values:

| Raw bytes | Little-endian | Big-endian | Note |
| --- | --- | --- | --- |
| `01000000` | 1 | 16777216 | Typical x86 value of 1 |
| `FF000000` | 255 | 4278190080 | Low byte holds 0xFF |
| `00000100` | 65536 | 256 | Byte 3 vs byte 2 set |
| `78563412` | 305419896 | 2018915346 | = 0x12345678 (LE) / 0x78563412 (BE) |

In forensics, fields such as file sizes, sector counts, offsets and timestamps in NTFS, FAT or memory dumps are stored in a defined byte order. Reading them in the wrong order gives wildly wrong sizes or times.

## 7. Task F: Mini Forensic Case

| Item | Value | Decoded meaning | Method / use |
| --- | --- | --- | --- |
| EVID-001 | `48656c6c6f204943444641` | "Hello ICDFA" | `xxd -r -p`; text/content |
| EVID-002 | `01001000 01101001` | "Hi" (0x48, 0x69) | Binary to ASCII; text/content |
| EVID-003 | `1675008832` | Sun Jan 29 2023, 16:13:52 UTC | `date -u -d`; timeline |
| EVID-004 | `725846400` (Mac) | 2024-01-02 00:00:00 UTC | +978307200, then Python; timeline |
| EVID-005 | `78563412` | LE 305419896; BE 2018915346 | `int.from_bytes`; byte order |

EVID-003 and EVID-004 are the most useful for timeline analysis, while EVID-001 and EVID-002 are for content interpretation. EVID-005 differs because the same bytes are read in opposite order, so the examiner must know the source format.

## 8. Reflection

Number systems and timestamps matter because digital evidence is stored as bytes, and every interpretation depends on the correct base, encoding, epoch, time zone and byte order. A small mistake, such as an extra newline, a wrong epoch or reversed bytes, can change a hash, shift a timeline or misstate a value, weakening the reliability of a finding. This lab showed me that an examiner must understand and explain the output, not only run the command, and validate results with more than one tool.
