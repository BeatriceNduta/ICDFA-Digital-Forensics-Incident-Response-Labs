# CIP-B101 Lab 1A: Number Systems, ASCII, Timestamps and Data Representation

# Student Information  

  | Filed | Details |
  |---|---|  
  | Student Name | Beatrice Nduta  
  | Student ID | C11/26/PCDFIR/17016 |
  | Course Name | Digital Forensics & Incident Response |
  | Instructor Name |  Aminu Idris, AMCPN |
  | Date of submission | 03/11/2026

----------------------------------------------------------------------------------------------------

Overview

This lab builds the skills a digital forensics trainee needs to read raw evidence correctly: number
bases, text encodings, timestamps, hashes and byte order. Every value was converted manually, then
verified in the terminal using shell arithmetic, bc, printf, xxd, date and Python.

## Tools Used

| Tool | Purpose |
| --- | --- |
| `bash` | Shell arithmetic and file creation |
| `python3` | Conversion and timestamp validation |
| `bc` | Decimal to binary conversion |
| `xxd` | Hex dump and hex-to-text conversion |
| `date` | Unix timestamp conversion |
| `md5sum` / `sha256sum` | Hash verification |


Task A: Number Systems

Hexadecimal is used in forensics because one byte is exactly two hex digits, which makes memory,
sector and file-header values compact. Binary shows the underlying bits, and each hex digit maps to
4 bits.

  -------------------------------------------------------------------------------------------------------------------
  Value               Base                Result              Manual method       Verification
  ------------------- ------------------- ------------------- ------------------- -----------------------------------
  101010              Binary              42                  32 + 8 + 2          echo $((2#101010))

  AB2                 Hex                 2738                (10×256) +          echo $((16#AB2))
                                                              (11×16) + 2         

  126                 Decimal             1111110             Repeated division   echo "obase=2;126" | bc
                                                              by 2                

  2738                Decimal             AB2                 2738÷16 = 171 r2,   printf "%X" 2738
                                                              171÷16 = 10 r11     

  101010110010        Binary              AB2                 Groups 1010 | 1011  printf '%X\n' $((2#101010110010))
                                                              | 0010              
  -------------------------------------------------------------------------------------------------------------------

All manual answers matched the Bash and Python outputs.

Task B: ASCII, UTF-8 and Hex Evidence

    echo -n "Digital Forensics Begins With Bytes" > beatrice_lab1a.txt
    xxd beatrice_lab1a.txt          # hex + ASCII view
    xxd -p beatrice_lab1a.txt       # plain hex
    printf "<plain hex>" | xxd -r -p   # hex back to text

-   xxd output began 4469 6769 7461 6c20, which is “Digital”.
-   Python confirmed the mapping: Hello → 48656c6c6f, ord('H') = 72, chr(72) = 'H'.
-   Why 0A appears: echo without -n appends a newline byte (0x0A). It is invisible as text but it is
    real data, so -n is used when exact content matters.
-   Not all hex is text: bytes may represent images, compressed or encrypted data, or binary
    structures such as timestamps and offsets. The file type and context must be identified before
    decoding.

Task C: Timestamps (all results in UTC)

  -------------------------------------------------------------------------------------------------------------------
  Value                    Format                   Command                                  Readable UTC result
  ------------------------ ------------------------ ---------------------------------------- ------------------------
  1643643491               Unix seconds             date -u -d @1643643491                   Mon Jan 31 03:38:11 PM
                                                                                             UTC 2022

  1675008832               Unix seconds             date -u -d @1675008832                   Sun Jan 29 04:13:52 PM
                                                                                             UTC 2023

  725846400                Mac Absolute Time        date -u -d @$((725846400 + 978307200))   Tue Jan 2 12:00:00 AM
                                                                                             UTC 2024

  Current time             Unix seconds             now=$(date +%s); date -u -d @$now        Tue Nov 3 09:52:57 PM
                                                                                             UTC 2026
  -------------------------------------------------------------------------------------------------------------------

Unix time counts seconds from 1970-01-01 and Mac Absolute Time from 2001-01-01. The offset between
them is 978307200 seconds. Using the wrong epoch, or mixing local time with UTC, can place an event
at the wrong time, so every result is labelled UTC.

Task D: Hashing and the Newline Problem

echo "Hello" writes 48 65 6c 6c 6f 0a (6 bytes), while echo -n "Hello" writes 48 65 6c 6c 6f (5
bytes). Hashes are byte-sensitive, so one extra byte produces a completely different MD5 and SHA-256
(the avalanche effect).

  File                        MD5                                SHA-256 (start…end)
  --------------------------- ---------------------------------- ---------------------
  Hello, with newline         09f7e02f1290be211da707a266f153b3   66a045b4…c1bb35f18
  Hello, without newline      8b1a9953c4611296a827abf8c47804d7   185f8db3…6381969
  Beatrice, with newline      489e78107340a536d88dff942d2ddb18   1d4f0740…582d90c6
  Beatrice, without newline   ab3aec6d954571c7551a186ea1cd98ff   1075bbd8…f5ade9fe

To hash exactly the visible characters, use echo -n (or printf without \n):

    echo -n "beatrice" | sha256sum

A mismatch caused by a stray newline could wrongly suggest evidence was altered.

Task E: Endianness

The system byte order is little-endian (sys.byteorder). Little-endian stores the least significant
byte first; big-endian stores the most significant byte first.

  Raw bytes   Little-endian   Big-endian
  ----------- --------------- ------------
  01000000    1               16777216
  FF000000    255             4278190080
  00000100    65536           256
  78563412    305419896       2018915346

In forensics, file sizes, sector counts, offsets and timestamps in structures such as NTFS, FAT or
memory dumps are stored in a defined byte order. Reading them in the wrong order gives wildly wrong
values.

Task F: Mini Forensic Case

  Item       Value                    Decoded meaning                 Use
  ---------- ------------------------ ------------------------------- ------------
  EVID-001   48656c6c6f204943444641   “Hello ICDFA”                   Content
  EVID-002   01001000 01101001        “Hi” (0x48, 0x69)               Content
  EVID-003   1675008832               Sun Jan 29 2023, 16:13:52 UTC   Timeline
  EVID-004   725846400 (Mac)          2024-01-02 00:00:00 UTC         Timeline
  EVID-005   78563412                 LE 305419896, BE 2018915346     Byte order

EVID-003 and EVID-004 are the most useful for timeline analysis, while EVID-001 and EVID-002 are for
content interpretation. EVID-005 differs because the same bytes are read in opposite order, so the
examiner must know the source format.

Reflection

Number systems and timestamps matter because digital evidence is stored as bytes, and every
interpretation depends on the correct base, encoding, epoch, time zone and byte order. A small
mistake, such as an extra newline, a wrong epoch or reversed bytes, can change a hash, shift a
timeline or misstate a value, weakening the reliability of a finding. An examiner must understand
and explain the output, not only run the command, and should validate results with more than one
tool.


References

-   Bash Shell Arithmetic
-   Bash printf command
-   Unicode
-   Coordinated Universal Time
-   MD5 of echo vs Python (Stack Overflow)
-   Migration techniques for different endianness (IAR)
