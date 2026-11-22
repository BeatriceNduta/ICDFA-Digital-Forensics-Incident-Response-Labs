# Lab 2B Report: Advanced Linux Commands, Standard Streams and Controlled Evidence Transfer

**International Cybersecurity and Digital Forensics Academy (ICDFA) | CIP-B101 Basic Computer Skills for Digital Forensics**

| Field | Details |
|---|---|
| Student | Beatrice Nduta |
| Student ID | C11/26/PCDFIR/17016 |
| Course | Digital Forensics & Incident Response |
| Instructor | Basiru Aliyu |
| Date | 14 November 2026 |

---

## 1. Lab Objective

This lab practised safe command-line evidence handling between two authorised machines. The aims were to explain and use standard input, output and error (file descriptors 0, 1 and 2), redirect command output into evidence logs, transfer files in both directions with `netcat`/`ncat`, and prove integrity by comparing cryptographic hashes taken before and after each transfer.

## 2. Lab Environment

| Item | Details |
|---|---|
| Machine 1 (Linux) | Kali Linux VM in VMware, hostname `kali`, `eth0` IP `192.168.146.128`, gateway `192.168.146.2`, netcat at `/usr/bin/nc` |
| Machine 2 (Windows) | Windows computer with `ncat` (from Nmap) and `certutil`; IPv4 `192.168.x.x` (partly redacted in screenshots) |
| Tools used | `bash`, `echo`, `cat`, `tee`, `ls`, `ip`, `ping`, `sha256sum`, `md5sum`, `dd`, `nc`/`ncat`, `certutil`, `wget` |
| Ports used | TCP 9090 (Part D), 9091 (Part E), 9092 (Part F), on an authorised lab network only |

## 3. Procedure

| Part | What I did (key commands) | Outcome |
|---|---|---|
| A: Streams | Created the evidence folder structure and `evidence_note.txt`, then redirected output with `>` (stdout), `2>` (stderr) and `2>&1` (both), e.g. `ls -la evidence > logs/evidence_listing_stdout.txt` | stdout, stderr and combined output saved as log files |
| B: Baseline | Recorded `hostname`, `ip address` and `ip route` on Linux with `tee`; `ipconfig` and `where ncat` on Windows; tested connectivity with `ping` | Both machines documented; 4/4 ping replies, 0% loss |
| C: Hashing | Hashed the source file before transfer with `sha256sum` and `md5sum`, saved to the hashes folder | Source hashes recorded |
| D: Linux to Windows | Windows listener: `ncat -l -p 9090 > received_evidence_note.txt`. Linux sender: `nc WINDOWS_IP 9090 < evidence_note.txt`. Verified with `certutil` | File received; hashes identical |
| E: Windows to Linux | Linux listener: `nc -l -p 9091 > BeatriceNdutaEvidence_received.txt`. Windows sender: `ncat LINUX_IP 9091 < BeatriceNdutaEvidence.txt`. Verified with `sha256sum` | File received; hash identical |
| F: dd image | Created a 5 MiB test image with `dd if=/dev/zero bs=1M count=5`, wrote an ICDFA marker at offset 1024, hashed it and sent it on port 9092 | Image received; hash identical |
| G: Block size | Ran `dd_obs_test.sh` against test files only | Fastest rates (about 1.2 GB/s) at 64 KiB to 256 KiB |

## 4. Evidence Integrity

Hashes were taken on the source machine before transfer and on the receiving machine afterwards. In every case the values were identical, so each file arrived unchanged.

| File | Route | SHA-256 (source vs destination) | Result |
|---|---|---|---|
| `evidence_note.txt` (73 bytes) | Linux to Windows, 9090 | Src `146a5490aa5cda5847ec2dfe518728dcdad6e1403c656570bcd391f85e821706`<br>Dst `146a5490aa5cda5847ec2dfe518728dcdad6e1403c656570bcd391f85e821706` | Match |
| `BeatriceNdutaEvidence.txt` | Windows to Linux, 9091 | Src `176c99f7ccb4c0d4da5a8838e6f23e45e8abf6a11f4b3ca67e94e450bdddb0e5`<br>Dst `176c99f7ccb4c0d4da5a8838e6f23e45e8abf6a11f4b3ca67e94e450bdddb0e5` | Match |
| `test_evidence_image.dd` (5.0 MiB) | Linux to Windows, 9092 | Src `9c370f38b2da04216eb1f8fc92b465747c448e01df83fe904f1c401686588679`<br>Dst `9c370f38b2da04216eb1f8fc92b465747c448e01df83fe904f1c401686588679` | Match |

MD5 for `evidence_note.txt` also matched on both machines: `e202487e8bc830238b35f058fea510a4`.

## 5. Screenshots

**Fig 1. Linux network details: `eth0` 192.168.146.128, ncat found at `/usr/bin/nc`**

<img width="700" alt="Fig 1 Linux network details" src="screenshots/fig01_linux_network_details.png" />

**Fig 2. Source SHA-256 and MD5 hashes recorded before transfer**

<img width="700" alt="Fig 2 Source hashes" src="screenshots/fig02_source_hashes.png" />

**Fig 3. Windows verification: SHA-256 and MD5 match the Linux source**

<img width="700" alt="Fig 3 Windows verification" src="screenshots/fig03_windows_verification.png" />

**Fig 4. Windows hash of the received dd image matches the Linux source hash**

<img width="800" alt="Fig 4 Windows image hash" src="screenshots/fig04_windows_image_hash.png" />

**Fig 5. Linux received the Windows file; SHA-256 recorded and matched**

<img width="722" alt="Fig 5 Linux received Windows file" src="screenshots/fig05_linux_received_windows_file.png" />

**Fig 6. dd test image created (5.0 MiB) and hashed on Linux**

<img width="777" alt="Fig 6 dd test image" src="screenshots/fig06_dd_test_image.png" />

## 6. Findings

- **Redirection:** `>` sent stdout to a file, `2>` sent error messages to a file, and `2>&1` merged both into one log, giving a permanent, accurate command record.
- **Transfers:** three transfers (two text files, one 5 MiB image) succeeded in both directions with no data change.
- **Issue noted:** my first ping log failed with "No such file or directory" because the command was run from the home folder instead of the lab folder; running it from the lab folder fixes this.
- **Performance:** `dd` speed varied with block size and peaked at roughly 1.2 GB/s (64 KiB to 256 KiB); results may be inflated because the kernel cache was not cleared.

## 7. Reflection Answers

1. **stdout vs stderr:** stdout (fd 1) carries normal output and stderr (fd 2) carries error messages. Keeping them separate lets errors be redirected or logged on their own.
2. **Why redirection helps:** it saves exact, reviewable output as a permanent record that can be hashed and repeated.
3. **Hash before and after:** a matching hash proves the copy is identical; a mismatch shows alteration or corruption and the transfer must be repeated.
4. **Unauthorised remote access:** it is illegal and unethical, risks changing or damaging a system, breaches privacy and can make evidence inadmissible.
5. **Bind vs reverse shell:** a bind shell listens on the target and the attacker connects in; a reverse shell makes the target connect out to the attacker's listener.
6. **Firewalls:** many firewalls block unsolicited inbound connections but allow outbound ones, so a reverse connection may pass where a bind connection is blocked.
7. **Test image:** a small instructor-approved image teaches the same integrity and transfer skills without risking a real disk or acquiring data without authorisation.

## 8. Conclusion

The lab showed that redirection and `tee` turn terminal activity into evidence, that netcat can move files between authorised machines, and that hashing before and after transfer is what proves integrity. In digital forensics, acquisition must be authorised, documented and repeatable, and every copy must be verified before it is relied upon.
