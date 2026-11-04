#Author  
**Beatrice Nduta**  
Digital Forensics and Incident Response  
International Cybersecurity and Digital Forensics Academy
 ---

 # CIP-B101 Lab 1A: Number Systems, ASCII, Timestamps and Data Representation 

  | Filed | Details |
  |---|---|  
  | Student Name | Beatrice Nduta  
  | Student ID | C11/26/PCDFIR/17016 |
  | Course Name | Digital Forensics & Incident Response |
  | Instructor Name |  Aminu Idris, AMCPN |
  | Date of submission | 03/11/2026

----------------------------------------------------------------------------------------------------

## Overview

This lab builds the skills a digital forensics trainee needs to read raw evidence correctly: number bases, text encodings, timestamps, hashes and byte order. Every value was converted manually, then verified in the terminal using shell arithmetic, `bc`, `printf`, `xxd`, `date` and Python.

**Working folder created in Kali Linux (VMware), showing student ID in the path**

<img width="581" height="240" alt="001 Making a Directory" src="https://github.com/user-attachments/assets/e86ad23b-391a-4d44-8244-7473bcaa6316" />

## Tools Used

| Tool | Purpose |
| --- | --- |
| `bash` | Shell arithmetic and file creation |
| `python3` | Conversion and timestamp validation |
| `bc` | Decimal to binary conversion |
| `xxd` | Hex dump and hex-to-text conversion |
| `date` | Unix timestamp conversion |
| `md5sum` / `sha256sum` | Hash verification |

## Task A: Number Systems

Hexadecimal is used in forensics because one byte is exactly two hex digits, which makes memory, sector and file-header values compact. Binary shows the underlying bits, and each hex digit maps to 4 bits.

| Value | Base | Result | Manual method | Verification |
| --- | --- | --- | --- | --- |
| `101010` | Binary | **42** | 32 + 8 + 2 | `echo $((2#101010))` |
| `AB2` | Hex | **2738** | (10×256) + (11×16) + 2 | `echo $((16#AB2))` |
| `126` | Decimal | **1111110** | Repeated division by 2 | `echo "obase=2;126" \| bc` |
| `2738` | Decimal | **AB2** | 2738÷16 = 171 r2, 171÷16 = 10 r11 | `printf "%X" 2738` |
| `101010110010` | Binary | **AB2** | Groups 1010 \| 1011 \| 0010 | `printf '%X\n' $((2#101010110010))` |

All manual answers matched the Bash and Python outputs.

### Manual working (handwritten)

**Conversions 1-3: binary to decimal, hex to decimal, decimal to binary**

<img width="756" height="1796" alt="Manual conversions evidence" src="https://github.com/user-attachments/assets/740c7017-1711-4670-835d-160626138e28" />

**Conversions 4-5: decimal to hex, binary to hex**

<img width="2885" height="3257" alt="Coversion Evidence" src="https://github.com/user-attachments/assets/9bccbffb-af9d-4db5-886b-0bb091cbe5cc" />

### Terminal verification

**Binary to decimal with shell arithmetic**

<img width="572" height="72" alt="002 Binary to decimal conversion " src="https://github.com/user-attachments/assets/e38a2078-ba7e-4ee4-a84f-56636db8b76a" />


**Hex to decimal with shell arithmetic**

<img width="575" height="71" alt="003 Hexadecimal to decimal conversion" src="https://github.com/user-attachments/assets/dffcedc5-3a20-46bc-b8bb-d95040d52299" />

**Decimal to binary with bc**

<img width="583" height="65" alt="004 Decimal to binary " src="https://github.com/user-attachments/assets/a981484b-9f71-46f7-b3ae-a913b76a5c80" />

**Decimal to hex with printf**

<img width="575" height="95" alt="005 Decimal to Hexadecial using Printf" src="https://github.com/user-attachments/assets/07c1d0ad-e0a9-4d61-bf68-ff222febee9d" />

**Binary to hex with printf**

<img width="568" height="72" alt="007 Binary conversion to hex" src="https://github.com/user-attachments/assets/5a8bf9a6-9972-4a83-99e4-7c57eb437884" />

**Python validation of all conversions**

<img width="582" height="205" alt="006 py validation" src="https://github.com/user-attachments/assets/3204ff44-9df8-4626-ae43-efdf55d7c225" />


## Task B: ASCII, UTF-8 and Hex Evidence

```bash
echo -n "Digital Forensics Begins With Bytes" > beatrice_lab1a.txt
xxd beatrice_lab1a.txt          # hex + ASCII view
xxd -p beatrice_lab1a.txt       # plain hex
printf "<plain hex>" | xxd -r -p   # hex back to text
```

- `xxd` output began `4469 6769 7461 6c20`, which is "Digital ".
- Python confirmed the mapping: `Hello` → `48656c6c6f`, `ord('H') = 72`, `chr(72) = 'H'`.


**Text evidence viewed with cat, xxd and xxd -p**

<img width="595" height="242" alt="008 converting text and hex rep" src="https://github.com/user-attachments/assets/ac9439e9-cb24-41f6-86ea-29062bc2e423" />

**Hex converted back to ASCII**

<img width="583" height="237" alt="009 Text_hex evidence string" src="https://github.com/user-attachments/assets/423a3975-1997-41b0-a5ac-f0e6d35f872d" />

**Creating beatrice_lab1a.txt**

<img width="567" height="46" alt="010 Creating a file for decoding " src="https://github.com/user-attachments/assets/dd87e4ec-7683-4035-9ac2-d44157f71be4" />

**xxd view of the file**

<img width="559" height="81" alt="011 using xxd to display file content" src="https://github.com/user-attachments/assets/9efb9fba-3165-4596-98d6-266123e72a78" />


**Plain hex with xxd -p**

<img width="575" height="76" alt="012 extracting the plain text" src="https://github.com/user-attachments/assets/8aef9e2c-d596-4a64-b51f-f300faaa1d14" />

**Plain hex converted back to readable text**

<img width="601" height="120" alt="013 converts plain hex to text" src="https://github.com/user-attachments/assets/20da7f68-cd60-4bbc-af68-c0562d717c34" />


- **Why `0A` appears:** `echo` without `-n` appends a newline byte (`0x0A`). It is invisible as text but it is real data, so `-n` is used when exact content matters.
- **Not all hex is text:** bytes may represent images, compressed or encrypted data, or binary structures such as timestamps and offsets. The file type and context must be identified before decoding.

## Task C: Timestamps (all results in UTC)

| Value | Format | Command | Readable UTC result |
| --- | --- | --- | --- |
| `1643643491` | Unix seconds | `date -u -d @1643643491` | Mon Jan 31 03:38:11 PM UTC 2022 |
| `1675008832` | Unix seconds | `date -u -d @1675008832` | Sun Jan 29 04:13:52 PM UTC 2023 |
| `725846400` | Mac Absolute Time | `date -u -d @$((725846400 + 978307200))` | Tue Jan 2 12:00:00 AM UTC 2024 |
| Current time | Unix seconds | `now=$(date +%s); date -u -d @$now` | Tue Nov 3 09:52:57 PM UTC 2026 |

Unix time counts seconds from 1970-01-01 and Mac Absolute Time from 2001-01-01. The offset between them is **978307200 seconds**. Using the wrong epoch, or mixing local time with UTC, can place an event at the wrong time, so every result is labelled UTC.


**Unix epoch conversion with Python**
**Local time compared with UTC**

<img width="617" height="302" alt="014 Unix epoch time conversion" src="https://github.com/user-attachments/assets/5a649b01-db1d-4f73-916b-fa22e6d7aa8e" />

**Unix timestamp 1643643491**

<img width="615" height="71" alt="015 unix conversion" src="https://github.com/user-attachments/assets/6a7a411e-1e89-4a48-88bb-5f351e438970" />

**Unix timestamp 1675008832**

<img width="621" height="63" alt="016 unix conversion" src="https://github.com/user-attachments/assets/995688c5-e3f4-46ae-b432-62a25feee3ed" />


**Mac Absolute Time 725846400 with date**

<img width="598" height="65" alt="017 mac time" src="https://github.com/user-attachments/assets/6e23e3c5-680b-4444-84a9-5d8d28f3b845" />

**Current Unix time converted to UTC**

<img width="592" height="91" alt="018 current time unix sec" src="https://github.com/user-attachments/assets/3b7248cb-d212-490e-9475-46b882778a55" />


## Task D: Hashing and the Newline Problem

`echo "Hello"` writes `48 65 6c 6c 6f 0a` (6 bytes), while `echo -n "Hello"` writes `48 65 6c 6c 6f` (5 bytes). Hashes are byte-sensitive, so one extra byte produces a completely different MD5 and SHA-256 (the avalanche effect).

| File | MD5 | SHA-256 (start…end) |
| --- | --- | --- |
| Hello, with newline | `09f7e02f1290be211da707a266f153b3` | `66a045b4…c1bb35f18` |
| Hello, without newline | `8b1a9953c4611296a827abf8c47804d7` | `185f8db3…6381969` |
| Beatrice, with newline | `489e78107340a536d88dff942d2ddb18` | `1d4f0740…582d90c6` |
| Beatrice, without newline | `ab3aec6d954571c7551a186ea1cd98ff` | `1075bbd8…f5ade9fe` |

To hash exactly the visible characters, use `echo -n` (or `printf` without `\n`):

```bash
echo -n "beatrice" | sha256sum
```

A mismatch caused by a stray newline could wrongly suggest evidence was altered.


**xxd and MD5/SHA-256 for Hello with and without newline**

<img width="648" height="462" alt="019 -n explained" src="https://github.com/user-attachments/assets/2e3c3535-0f15-4795-be19-8811e8a1d41a" />


**Name files created with echo and echo -n, compared with xxd**

<img width="593" height="273" alt="020 File creation for hashing comparison" src="https://github.com/user-attachments/assets/e478de8a-1113-4400-8dc7-a6ddc9ab8db1" />

**MD5 and SHA-256 of the name files**

<img width="591" height="241" alt="021hashing without the newline addition" src="https://github.com/user-attachments/assets/c1c8bc30-bbdc-4dfa-9a06-68429b0a2a54" />

## Task E: Endianness

The system byte order is **little-endian** (`sys.byteorder`). Little-endian stores the least significant byte first; big-endian stores the most significant byte first.

| Raw bytes | Little-endian | Big-endian |
| --- | --- | --- |
| `01000000` | 1 | 16777216 |
| `FF000000` | 255 | 4278190080 |
| `00000100` | 65536 | 256 |
| `78563412` | 305419896 | 2018915346 |

<img width="577" height="247" alt="023 task six 001 Endianness" src="https://github.com/user-attachments/assets/c75e97c6-0655-4b80-b30e-34da15aeba34" />
<img width="617" height="193" alt="024 task six 002" src="https://github.com/user-attachments/assets/07506d5c-4fb6-4ab7-a8d5-998c1f1ce15a" />
<img width="675" height="193" alt="025 task six 003" src="https://github.com/user-attachments/assets/521f7e47-defe-47f1-9ebd-b4e57cda086d" />
<img width="686" height="197" alt="026 task six 004" src="https://github.com/user-attachments/assets/fc2dacfb-8978-4434-9a26-06f407149b8c" />


In forensics, file sizes, sector counts, offsets and timestamps in structures such as NTFS, FAT or memory dumps are stored in a defined byte order. Reading them in the wrong order gives wildly wrong values.


**System byte order and little/big-endian interpretation of 78563412**

<img width="727" height="313" alt="022 understanding Endians" src="https://github.com/user-attachments/assets/0f21303b-32ff-4760-8bda-21604f596108" />


## Task F: Mini Forensic Case

| Item | Value | Decoded meaning | Use |
| --- | --- | --- | --- |
| EVID-001 | `48656c6c6f204943444641` | "Hello ICDFA" | Content |
| EVID-002 | `01001000 01101001` | "Hi" (0x48, 0x69) | Content |
| EVID-003 | `1675008832` | Sun Jan 29 2023, 16:13:52 UTC | Timeline |
| EVID-004 | `725846400` (Mac) | 2024-01-02 00:00:00 UTC | Timeline |
| EVID-005 | `78563412` | LE 305419896, BE 2018915346 | Byte order |

#Hex conversion to ASCII

<img width="572" height="81" alt="image" src="https://github.com/user-attachments/assets/1bd0513c-11f9-4af6-82cc-84a17690d3de" />

#Local - UTC conversion 

<img width="587" height="288" alt="Local and UTC time comparison" src="https://github.com/user-attachments/assets/058c6805-f343-46d4-a8f2-daaab6cf45bf" />



EVID-003 and EVID-004 are the most useful for timeline analysis, while EVID-001 and EVID-002 are for content interpretation. EVID-005 differs because the same bytes are read in opposite order, so the examiner must know the source format.

## Reflection

Number systems and timestamps matter because digital evidence is stored as bytes, and every interpretation depends on the correct base, encoding, epoch, time zone and byte order. A small mistake, such as an extra newline, a wrong epoch or reversed bytes, can change a hash, shift a timeline or misstate a value, weakening the reliability of a finding. An examiner must understand and explain the output, not only run the command, and should validate results with more than one tool.

## References

- [Bash Shell Arithmetic](https://www.gnu.org/software/bash/manual/html_node/Shell-Arithmetic.html)
- [Bash printf command](https://linuxize.com/post/bash-printf-command/)
- [Unicode](https://en.wikipedia.org/wiki/Unicode)
- [Coordinated Universal Time](https://en.wikipedia.org/wiki/Coordinated_Universal_Time)
- [MD5 of echo vs Python (Stack Overflow)](https://stackoverflow.com/questions/5693360/why-is-an-md5-hash-created-by-python-different-from-one-created-using-echo-and-m)
- [Migration techniques for different endianness (IAR)](https://www.iar.com/knowledge/learn/programming/migration-techniques-for-different-endianness/)
