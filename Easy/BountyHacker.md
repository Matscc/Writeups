# Machine: Bounty Hacker (TryHackMe)

## Enumeration

I started by running a port scan to identify the available services on the target machine.

```bash
nmap -sC -sV <target ip>
```

- `-sC` — runs default scripts against discovered services.
- `-sV` — identifies service versions.

<img width="755" height="526" alt="scan" src="https://github.com/user-attachments/assets/9a099310-cedf-4f5e-b57e-b44185371761" />

Nmap identified 3 open ports:
- **21** — FTP service (anonymous login detected by Nmap scripts)
- **22** — SSH service
- **80** — HTTP service (web application running)

I first visited the web application, which only displayed an image. There were no exploitable directories, no sensitive information in the HTML source, and nothing further to enumerate. Since I had no SSH credentials, the only remaining attack surface was the FTP service, which allowed anonymous login.

## Exploitation

I logged into the FTP service anonymously and found two `.txt` files: `locks.txt` and `tasks.txt`. I downloaded both files and examined their contents.

The `locks.txt` file contained a wordlist — likely intended for a brute force attack against the SSH service.

<img width="189" height="432" alt="locks" src="https://github.com/user-attachments/assets/38dcb755-e828-4bdf-acfa-e86f3869d7ac" />

The `tasks.txt` file contained a to-do list written by someone named **lin** — a potential SSH username.

<img width="391" height="86" alt="tasks" src="https://github.com/user-attachments/assets/7232b62f-16ef-4c86-9a54-65c70dbbfe1a" />

With a potential username and wordlist in hand, I launched a brute force attack against the SSH service using Hydra.

```bash
hydra -l lin -P locks.txt <target ip> ssh
```

- `-l` — specifies a single username.
- `-P` — specifies the path to the wordlist.

Hydra successfully found the password for user **lin**, granting SSH access.

<img width="732" height="119" alt="hydra" src="https://github.com/user-attachments/assets/793b3a3b-0ba3-4249-899a-0b3ec3689ca3" />

I logged in via SSH and retrieved the first flag — `user.txt`.

## Privilege Escalation

User **lin** had sudo privileges. I checked what commands could be run as root.

```bash
sudo -l
```

The output revealed that **lin** was allowed to run the `tar` binary as root without a password.

<img width="965" height="111" alt="permissions" src="https://github.com/user-attachments/assets/692405b0-9b71-4444-a752-a60b69e61cb2" />

> **Note:** `tar` is a file compression and extraction utility. When allowed to run as root, it can be abused via its `--checkpoint-action` flag to execute arbitrary commands with elevated privileges.

I referenced GTFOBins and used the following command to spawn a root shell:

```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

With root access, I navigated to the `/root` directory and retrieved the second and final flag — `root.txt` — successfully completing the machine.

## Conclusion

The Bounty Hacker machine presents a classic exploitation flow combining service enumeration, credential discovery, and privilege escalation through a misconfigured sudo permission. The enumeration revealed an anonymously accessible FTP service containing files that led to valid SSH credentials.

After gaining initial access, checking sudo permissions revealed that the user could run `tar` as root without a password. This misconfiguration allowed abuse of the `--checkpoint-action` functionality, resulting in command execution with elevated privileges and a root shell.

This machine reinforces the importance of local enumeration and careful analysis of sudo permissions, demonstrating how seemingly harmless binaries can be leveraged for privilege escalation when misconfigured. It also highlights the value of tools like GTFOBins for quickly identifying exploitation vectors in Linux environments.
