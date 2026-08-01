# Machine: LazyAdmin (TryHackMe)

## Enumeration

I started by scanning the target IP to identify active services.

```bash
nmap -sC -sV <TargetIp>
```

<img width="834" height="281" alt="nmap lazy" src="https://github.com/user-attachments/assets/ae8a94e7-10da-4316-b0d4-78e411c584b7" />

Nmap revealed two available services:
- **22** — SSH
- **80** — HTTP (Apache web server)

I proceeded to enumerate the web application. The home page had nothing exploitable, so I used Gobuster to discover hidden directories.

```bash
gobuster dir -u <targetip> -w <wordlistPath>
```

<img width="684" height="166" alt="gobuster 1 lazy" src="https://github.com/user-attachments/assets/4ec6c10a-1c59-4039-a5df-16bb3e41b10b" />

Gobuster revealed a `/content` directory. It only showed a maintenance page, but confirmed the site was built with **SweetRice**. I ran Gobuster again to enumerate further subdirectories.

<img width="774" height="262" alt="gobuster 2 lazy" src="https://github.com/user-attachments/assets/1bb42554-d05c-41f5-8d47-1bf0f59c6328" />

Two additional directories were found:
- `/as` — the SweetRice admin login page
- `/inc` — a directory containing application files

## Exploitation

Inside the `/inc` directory, I found a **database backup** — a potential source of login credentials.

<img width="632" height="797" alt="mysql lazy" src="https://github.com/user-attachments/assets/34ac17da-7f74-4da5-a9e1-88dd6ca67859" />

Analyzing the backup, I found an **MD5-hashed password** and the username **manager**.

<img width="1920" height="344" alt="password hash lazy" src="https://github.com/user-attachments/assets/554a7302-98c9-42e5-9bd5-6ee296ad47fd" />

I cracked the MD5 hash using an online tool (md5decrypt) and logged in with the obtained credentials.

<img width="907" height="809" alt="login lazy" src="https://github.com/user-attachments/assets/77ca71be-76e0-4fc0-bf08-4c96dbfb8604" />

Logged in as **manager** in SweetRice, I uploaded a **PHP reverse shell** via the **Data Import** section.

<img width="1372" height="772" alt="data reverse lazy" src="https://github.com/user-attachments/assets/5fdb73a2-50f5-4b78-aaf4-af74ac9c204a" />

I set up a Netcat listener on port 8000 and triggered the reverse shell through the web application, gaining shell access to the target. I found the first flag (`user.txt`) in user **itguy**'s home directory.

## Privilege Escalation

I ran `sudo -l` to check which commands the current user could execute with elevated privileges.

<img width="958" height="111" alt="sudo -l lazy" src="https://github.com/user-attachments/assets/7814d820-25e4-421c-bb85-a3c73210aee1" />

The output showed that the user could run `backup.pl` (owned by root) using the `perl` binary as sudo — but could not modify the file directly.

Reading `backup.pl` with `cat`, I discovered it called another script named `copy.sh` — which I did have write permissions on.

<img width="295" height="77" alt="backup lazy" src="https://github.com/user-attachments/assets/48dc3764-1437-4017-9666-7c358fd7b116" />

Inspecting `copy.sh`, it contained a command to initiate a remote connection to another machine.

<img width="658" height="57" alt="copy.sh lazy" src="https://github.com/user-attachments/assets/609f198e-cb3d-4bff-a7e4-0254440f2453" />

I set up a Netcat listener on port 4444 and modified `copy.sh` to point to my machine's IP.

<img width="900" height="92" alt="echo lazy" src="https://github.com/user-attachments/assets/982c66c4-7160-4130-bc9d-bb654cb9ea5a" />

I executed `backup.pl` with sudo and received a root shell.

<img width="706" height="316" alt="root lazy" src="https://github.com/user-attachments/assets/da183878-2760-4144-a894-9032a559a159" />

With root access, I navigated to `/root` and retrieved the second and final flag (`root.txt`), successfully completing the machine.

## Conclusion

LazyAdmin demonstrates how a chain of small security flaws can result in full system compromise. The attack began with proper enumeration of exposed services, identifying a vulnerable web application that allowed initial access through weak credentials and misconfigured functionality.

After gaining initial access, sensitive files containing credentials were discovered — highlighting serious failures in secrets management and privilege separation. The privilege escalation phase revealed a misconfigured sudo permission allowing specific commands to run with elevated privileges without proper restriction. This type of flaw turns limited access into full machine control, completely compromising confidentiality, integrity, and availability.
