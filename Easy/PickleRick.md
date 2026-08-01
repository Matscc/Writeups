# Machine: Pickle Rick (TryHackMe)

## Enumeration

I started by running a port scan to identify the available services on the target machine.

```bash
nmap -sV <targetIp>
```

<img width="785" height="182" alt="nmap rick" src="https://github.com/user-attachments/assets/805f5fbd-67a9-4be2-bb24-6431f16f38f8" />

The scan revealed two open ports:
- **22** — SSH service
- **80** — HTTP service (web application hosted on Apache)

I accessed the web application through the browser. Inspecting the HTML source of the home page, I found a comment disclosing a system username — useful for later stages.

<img width="872" height="357" alt="html rick" src="https://github.com/user-attachments/assets/e956cc3f-6846-406d-bdf6-fc41fa396314" />

I also checked the `robots.txt` file, which contained a seemingly random string — noted for potential future use.

<img width="279" height="187" alt="wuba rick" src="https://github.com/user-attachments/assets/a20f6163-cc53-4774-9050-3d378c462000" />

I used Gobuster to enumerate hidden directories in the application.

```bash
gobuster dir -u http://<target_ip> -w <wordlist> -x php
```

<img width="699" height="324" alt="gobuster rick" src="https://github.com/user-attachments/assets/927575e8-895b-43bb-b37a-ffffedb4d64b" />

The scan discovered the `login.php` directory.

## Exploitation

I tested the credentials gathered during enumeration on the `login.php` page and successfully logged in.

After logging in, I was redirected to a portal that contained a dangerous feature — a **web shell** allowing direct command execution on the system through the web application.

The first CTF flag was found through this panel.

<img width="1384" height="604" alt="panel rick" src="https://github.com/user-attachments/assets/cccc7b61-e8b9-4bca-948c-35f59481e744" />

To obtain a less restricted shell, I used a Python reverse shell payload from the **PayloadsAllTheThings** repository.

I set up a listener on my machine:

```bash
nc -lvnp 8000
```

I executed the reverse shell payload through the web shell and gained remote access. Exploring the filesystem, I found the second flag in user **rick**'s home directory.

## Privilege Escalation

With access to the system as a regular user, I checked for privilege escalation paths.

```bash
sudo -l
```

The output revealed that the current user had **unrestricted sudo permissions** — able to run any command as root without a password.

<img width="758" height="96" alt="nopasswd rick" src="https://github.com/user-attachments/assets/6367929a-16bf-48ed-8693-4aaecf7595d2" />

Privilege escalation was straightforward.

```bash
sudo su
```

With root access, I navigated to `/root` and retrieved the third and final flag, successfully completing the machine.

## Conclusion

Pickle Rick exploits common web application flaws — including sensitive information disclosure in HTML comments, improper use of `robots.txt`, and an unrestricted web shell. Additionally, the misconfigured sudo permissions allowing unrestricted root access make privilege escalation trivial once initial access is obtained.
