# Advanced Linux Commands

# CIP-B101 Lab 2B: Advanced Linux Commands, Standard Streams and Controlled Evidence Transfer

| Field | Details |
|---|---|
| Student Name | Beatrice Nduta |
| Student ID | C11/26/PCDFIR/17016 |
| Course Name | Digital Forensics & Incident Response |
| Course Code / Title | CIP-B101 Basic Computer Skills for Digital Forensics |
| Instructor Name | Basiru Aliyu |
| Date of submission | 14/11/2026 |

----------------------------------------------------------------------------------------------------

## Overview

This lab practises safe command-line evidence handling between two authorised machines. It covers standard input, output and error (file descriptors 0, 1 and 2), redirecting command output into evidence logs, transferring files in both directions with `netcat`/`ncat`, and proving integrity by comparing cryptographic hashes taken before and after each transfer.

> **Safety note:** `nc`/`ncat` was used only as a controlled file-transfer tool between my own lab machines. No shell-execution options (`-e`, `--exec`) were used, and bind-shell and reverse-shell concepts were treated as theory only.

**Evidence folder structure created in Kali Linux (VMware)**

<img width="800" alt="Fig 1 Folder structure" src="screenshots/fig01_folder_structure.png" />

## Lab Environment

| Item | Details |
| --- | --- |
| Machine 1 (Linux) | Kali Linux VM in VMware, hostname `kali`, `eth0` IP `192.168.146.128`, gateway `192.168.146.2`, netcat at `/usr/bin/nc` |
| Machine 2 (Windows) | Windows computer with `ncat` (from Nmap) and `certutil`; IPv4 `192.168.x.x` (partly redacted in screenshots) |
| Ports used | TCP 9090 (Part D), 9091 (Part E), 9092 (Part F), on an authorised lab network only |

## Tools Used

| Tool | Purpose |
| --- | --- |
| `bash`, `echo`, `cat`, `ls` | File creation and viewing |
| `>`, `2>`, `2>&1`, `tee` | Redirecting output and errors into logs |
| `ip`, `ping` | Network documentation and connectivity tests |
| `sha256sum`, `md5sum` | Hashing on Linux |
| `certutil` | Hashing on Windows |
| `nc` / `ncat` | Controlled file transfer between machines |
| `dd` | Creating a small test evidence image and block-size testing |
| `wget`, `chmod` | Downloading and preparing the `dd_obs_test.sh` script |

## Evidence Folder Structure

```
CIP-B101_Lab2B_Beatrice_Nduta/
├── screenshots/
├── logs/            # stdout/stderr logs, network details, ping and dd test results
├── evidence/        # evidence_note.txt, test_evidence_image.dd
├── received/        # file received from Windows
├── hashes/          # source and received hash files
└── report_notes/
```

Keeping source evidence, logs, hashes and received files in separate folders makes the work organised and easy to audit.

## Part A: Standard Input, Output and Error

| Stream | File descriptor | Meaning |
| --- | --- | --- |
| stdin | 0 | Default input source (keyboard or redirected file) |
| stdout | 1 | Normal command output |
| stderr | 2 | Error messages |

### A1: Create a test evidence file

```bash
cd ~/CIP-B101_Lab2B_Beatrice_Nduta/evidence
echo "This is an authorized forensic evidence transfer test for Beatrice Nduta" > evidence_note.txt
cat evidence_note.txt
ls -l evidence_note.txt
```

**evidence_note.txt created, displayed and listed**

<img width="800" alt="Fig 2 evidence_note" src="screenshots/fig02_evidence_note.png" />

The file is 73 bytes with permissions `-rw-rw-r--`: the owner and group can read and write, and other users can only read.

### A2: Redirect standard output

```bash
ls -la evidence > logs/evidence_listing_stdout.txt
cat logs/evidence_listing_stdout.txt
```

**Standard output redirected into logs/evidence_listing_stdout.txt**

<img width="500" alt="Fig 3 stdout redirect" src="screenshots/fig03_stdout_redirect.png" />

The `>` symbol redirects standard output into a file, creating a permanent, accurate record of the command's output that can be reviewed, submitted and hashed later to show it has not been altered.

### A3: Redirect standard error

```bash
cat evidence/file_that_does_not_exist.txt 2> logs/error_stderr.txt
cat logs/error_stderr.txt
```

**Standard error redirected into logs/error_stderr.txt**

<img width="600" alt="Fig 4 stderr redirect" src="screenshots/fig04_stderr_redirect.png" />

The `2>` operator sends file descriptor 2 to a file, so the "No such file or directory" message is kept as separate, reviewable evidence apart from normal output.

### A4: Redirect stdout and stderr together

```bash
(ls evidence && cat evidence/file_that_does_not_exist.txt) > logs/combined_output.txt 2>&1
cat logs/combined_output.txt
```

**Standard output and standard error combined into one log file**

<img width="800" alt="Fig 5 combined output" src="screenshots/fig05_combined_output.png" />

In `> logs/combined_output.txt 2>&1`, the `2>&1` part tells the shell to send stderr to the same place stdout is currently going. The file listing and the error message were both captured, in the order they occurred.

## Part B: Baseline System and Network Documentation

### B1: Linux network details

```bash
hostname | tee logs/linux_hostname.txt
ip address | tee logs/linux_ip_address.txt
ip route | tee logs/linux_ip_route.txt
which nc || which ncat || echo "netcat or ncat not found" | tee logs/netcat_check.txt
```

**Linux network details recorded with tee**

<img width="800" alt="Fig 6 Linux network details" src="screenshots/fig06_linux_network_details.png" />

`tee` displays the output and saves a copy in the logs folder at the same time. The Linux host is `kali`, its `eth0` address is `192.168.146.128/24` with default gateway `192.168.146.2`, and netcat is installed at `/usr/bin/nc`.

### B2: Windows network details

```bat
hostname
ipconfig
where ncat
```

**Windows network details from ipconfig (address partly redacted for privacy)**

<img width="800" alt="Fig 7 Windows ipconfig" src="screenshots/fig07_windows_ipconfig.png" />

**Confirmation that ncat is installed on Windows**

<img width="400" alt="Fig 8 ncat installed" src="screenshots/fig08_ncat_installed_windows.png" />

`where ncat` shows `ncat.exe` at `C:\Program Files (x86)\Nmap\ncat.exe`, so Windows can send and receive files with `ncat`.

### B3: Connectivity test

```bash
ping -c 4 WINDOWS_IP | tee logs/ping_windows.txt
```

**Ping from Linux to the Windows machine**

<img width="700" alt="Fig 9 Ping Linux to Windows" src="screenshots/fig09_ping_linux_to_windows.png" />

**Ping from Windows to the Linux machine**

<img width="600" alt="Fig 10 Ping Windows to Linux" src="screenshots/fig10_ping_windows_to_linux.png" />

| Direction | Result | TTL |
| --- | --- | --- |
| Linux to Windows | 4 sent, 4 received, 0% loss | 128 (typical of Windows) |
| Windows to Linux | 4 sent, 4 received, 0% loss | 64 (typical of Linux) |

Connectivity works in both directions.

> **Issue noted:** the `tee` message "logs/ping_windows.txt: No such file or directory" appeared because the command was run from the home folder instead of the lab folder, so the ping output was displayed but not saved to the log file. Running it from the lab folder fixes this.

## Part C: Hash the Evidence Before Transfer

```bash
cd ~/CIP-B101_Lab2B_Beatrice_Nduta/evidence
sha256sum evidence_note.txt | tee ../hashes/source_evidence_note_sha256.txt
md5sum evidence_note.txt | tee ../hashes/source_evidence_note_md5.txt
cat ../hashes/source_evidence_note_sha256.txt
```

**SHA-256 and MD5 hashes of the source file, recorded before transfer**

<img width="800" alt="Fig 11 Source hashes" src="screenshots/fig11_source_hashes.png" />

| Hash | Value |
| --- | --- |
| SHA-256 | `146a5490aa5cda5847ec2dfe518728dcdad6e1403c656570bcd391f85e821706` |
| MD5 | `e202487e8bc830238b35f058fea510a4` |

A hash is a fixed-length fingerprint of a file, so any change to the file would produce a different value.

## Part D: Linux to Windows Transfer (port 9090)

**D1. Start the listener on Windows**

```bat
mkdir C:\CIP-B101_Lab2B_Received
cd C:\CIP-B101_Lab2B_Received
ncat -l -p 9090 > received_evidence_note.txt
```

<img width="700" alt="Fig 12 Windows listener" src="screenshots/fig12_windows_listener_9090.png" />

*Figure 12: Listener started on Windows, port 9090. The "ls is not recognized" message is expected because `ls` is a Linux command; `dir` is the Windows equivalent.*

**D2. Send the file from Linux**

```bash
cd ~/CIP-B101_Lab2B_Beatrice_Nduta/evidence
ncat WINDOWS_IP 9090 < evidence_note.txt
```

<img width="531" alt="Fig 13 Linux sends file" src="screenshots/fig13_linux_sends_file.png" />

*Figure 13: Linux sending `evidence_note.txt` to the Windows listener. The command finished without error, showing the connection and send completed.*

**D3. Verify the received file on Windows**

```bat
certutil -hashfile received_evidence_note.txt SHA256
certutil -hashfile received_evidence_note.txt MD5
type received_evidence_note.txt
```

<img width="800" alt="Fig 14 Windows hash verification" src="screenshots/fig14_windows_hash_verification.png" />

*Figure 14: Windows verification of the received file using `certutil`. Both hashes are identical to the Linux source hashes in Figure 11, so the file arrived complete and unaltered.*

## Part E: Windows to Linux Transfer (port 9091)

**E1. Create and hash the evidence file on Windows**

```bat
cd C:\CIP-B101_Lab2B_Received
echo hello Beatrice Nduta! > BeatriceNdutaEvidence.txt
type BeatriceNdutaEvidence.txt
certutil -hashfile BeatriceNdutaEvidence.txt SHA256
```

<img width="657" alt="Fig 15 Windows evidence file" src="screenshots/fig15_windows_evidence_file.png" />

*Figure 15: Evidence file created on Windows and hashed before sending. Source SHA-256: `176c99f7ccb4c0d4da5a8838e6f23e45e8abf6a11f4b3ca67e94e450bdddb0e5`.*

**E2. Start the listener on Linux**

```bash
cd ~/CIP-B101_Lab2B_Beatrice_Nduta/received
ncat -l -p 9091 > BeatriceNdutaEvidence_received.txt
```

<img width="655" alt="Fig 16 Linux listener" src="screenshots/fig16_linux_listener_9091.png" />

*Figure 16: Listener started on Linux, port 9091. The terminal waits until the sender connects.*

**E3. Send from Windows**

```bat
ncat 192.168.146.128 9091 < BeatriceNdutaEvidence.txt
```

<img width="688" alt="Fig 17 Windows sends file" src="screenshots/fig17_windows_sends_file.png" />

*Figure 17: Windows sending the evidence file to the Linux listener. The prompt returned with no error, indicating the send completed.*

**E4. Verify on Linux**

```bash
cat BeatriceNdutaEvidence_received.txt
sha256sum BeatriceNdutaEvidence_received.txt | tee ../hashes/received_windows_file_sha256.txt
```

<img width="722" alt="Fig 18 Linux hash verification" src="screenshots/fig18_linux_hash_verification.png" />

*Figure 18: Linux verification of the file received from Windows. The SHA-256 equals the Windows source hash in Figure 15, so the transfer preserved the file exactly.*

## Part F: Test Image Transfer with dd (port 9092)

A small instructor-approved test image was used instead of imaging a real disk. This teaches the same acquisition, transfer and hash-verification skills without risking damage to a real system or acquiring someone's data without authorisation.

**F1. Create and hash the test image on Linux**

```bash
cd ~/CIP-B101_Lab2B_Beatrice_Nduta/evidence
dd if=/dev/zero of=test_evidence_image.dd bs=1M count=5 status=progress
echo "ICDFA test evidence marker - Beatrice Nduta" | dd of=test_evidence_image.dd bs=1 seek=1024 conv=notrunc
ls -lh test_evidence_image.dd
sha256sum test_evidence_image.dd | tee ../hashes/source_test_image_sha256.txt
```

<img width="777" alt="Fig 19 dd test image" src="screenshots/fig19_dd_test_image.png" />

*Figure 19: Test evidence image created and hashed on Linux. The first `dd` created a 5 MiB (5,242,880 byte) file of zeros. The second wrote a 44-byte marker at offset 1024 without truncating the file (`seek=1024`, `conv=notrunc`).*

**F2. Transfer to Windows**

```bat
:: Windows listener
cd C:\CIP-B101_Lab2B_Received
ncat -l -p 9092 > test_evidence_image_received.dd
```

```bash
# Linux sender
cd ~/CIP-B101_Lab2B_Beatrice_Nduta/evidence
nc WINDOWS_IP 9092 < test_evidence_image.dd
```

**F3. Verify on Windows**

```bat
certutil -hashfile test_evidence_image_received.dd SHA256
```

<img width="900" alt="Fig 20 Windows image hash" src="screenshots/fig20_windows_image_hash.png" />

*Figure 20: Windows hash of the received test image. It is identical to the Linux source hash in Figure 19, so the image transfer was forensically reliable. The listener command appears more than once because it was re-entered.*

## Evidence Integrity Summary

Hashes were taken on the source machine before transfer and on the receiving machine afterwards. In every case the values were identical.

| File | Route | SHA-256 (source and destination) | Result |
| --- | --- | --- | --- |
| `evidence_note.txt` (73 bytes) | Linux to Windows, port 9090 | `146a5490aa5cda5847ec2dfe518728dcdad6e1403c656570bcd391f85e821706` | Match |
| `BeatriceNdutaEvidence.txt` | Windows to Linux, port 9091 | `176c99f7ccb4c0d4da5a8838e6f23e45e8abf6a11f4b3ca67e94e450bdddb0e5` | Match |
| `test_evidence_image.dd` (5.0 MiB) | Linux to Windows, port 9092 | `9c370f38b2da04216eb1f8fc92b465747c448e01df83fe904f1c401686588679` | Match |

The MD5 for `evidence_note.txt` also matched on both machines: `e202487e8bc830238b35f058fea510a4`.

## Part G: dd Block Size Test (optional)

```bash
cd ~/CIP-B101_Lab2B_Beatrice_Nduta/logs
wget https://raw.githubusercontent.com/tdg5/blog/master/_includes/scripts/dd_obs_test.sh
chmod +x dd_obs_test.sh
./dd_obs_test.sh | tee dd_block_size_test_result.txt
```

<img width="900" alt="Fig 21 dd block size test" src="screenshots/fig21_dd_block_size_test.png" />

*Figure 21: dd block size test results.*

The script tested block sizes from 512 bytes to 64 MiB. The rate rose from 271 MB/s at 512 bytes to a peak of about 1.2 GB/s at 64 KiB to 256 KiB, then fell for very large blocks. Larger blocks mean fewer read and write calls, but beyond a point memory and cache effects reduce speed. The script warned that the kernel cache was not cleared, so the figures may be inflated.

## Findings

- **Redirection:** `>` sent stdout to a file, `2>` sent error messages to a file, and `2>&1` merged both into one log, giving a permanent and accurate command record.
- **Transfers:** three transfers (two text files and one 5 MiB image) succeeded in both directions with no data change.
- **Integrity:** every source and destination hash matched.
- **Performance:** `dd` speed varied with block size and peaked at roughly 1.2 GB/s (64 KiB to 256 KiB).

## Part H: Reflection Answers

**1. What is the difference between stdout and stderr?**
Standard output (fd 1) carries a command's normal results, while standard error (fd 2) carries its error messages. Keeping them separate lets each be redirected or logged on its own.

**2. Why is redirection useful in forensic documentation?**
It saves exact command output into files that can be kept, reviewed, hashed and submitted, giving an accurate and repeatable record with no retyping errors.

**3. Why hash a file before and after transfer?**
The hash before transfer is the fingerprint of the original evidence and the hash after is the fingerprint of the copy. A match proves the copy is identical. A mismatch shows alteration or corruption, and the transfer must be repeated.

**4. Why is unauthorised remote access unsafe and unethical?**
It is illegal, breaches privacy and professional ethics, risks changing or damaging the system, and can make any evidence collected inadmissible.

**5. Bind shell vs reverse shell?**
In a bind shell, the target opens a listening port and the attacker connects in. In a reverse shell, the target connects out to a listener controlled by the attacker. Both were treated as theory only.

**6. Why might a firewall block one but allow the other?**
Many firewalls block unsolicited inbound connections but allow outbound ones, so a reverse connection may pass where a bind connection is blocked.

**7. Why use a small test image instead of a real disk?**
It teaches the same integrity and transfer skills without risking a real disk or acquiring data without authorisation, and it is quick, safe and easy to repeat.

## Challenges and Resolution

| Issue | Resolution |
| --- | --- |
| `tee` reported "No such file or directory" for `logs/ping_windows.txt` because the ping was run from the home folder | Run the command from the lab folder so the relative `logs/` path exists |
| `ls` is not recognised on Windows | Used `dir`, the Windows equivalent |
| Listener command entered more than once in the Windows terminal | Noted in the evidence; the transfer and hash verification were unaffected |

## Reflection

This lab showed that redirection and `tee` turn terminal activity into evidence, that `netcat` can move files between authorised machines, and that hashing before and after transfer is what proves integrity. In digital forensics, acquisition must be authorised, documented and repeatable, and every copy must be verified before it is relied upon.

## References

- [Nmap / Ncat download](https://nmap.org/download)
- [Standard streams](https://en.wikipedia.org/wiki/Standard_streams)
- [Understanding the shell redirect idiom](https://www.brianstorti.com/understanding-shell-script-idiom-redirect/)
- [Netcat command reference](https://www.computerhope.com/unix/nc.htm)
- [dd_obs_test.sh block-size testing script](https://raw.githubusercontent.com/tdg5/blog/master/_includes/scripts/dd_obs_test.sh)
