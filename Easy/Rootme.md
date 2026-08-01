# Machine: RootMe (TryHackMe)

## Enumeration

I started by running a port scan to identify the available services on the target machine.

```bash
nmap -sV <target ip>
```

- `-sV` — identifies service versions.

<img width="624" height="218" alt="nmap r" src="https://github.com/user-attachments/assets/7a4fb4d8-6d03-463c-b700-54074f0515bf" />

Nmap identified two open ports:
- **22** — SSH service
- **80** — HTTP service (web application hosted on Apache)

I accessed the web application through the browser. The home page had nothing obviously exploitable, so I used Gobuster to search for hidden directories.

```bash
gobuster dir -u http://<targetip> -w <wordlistPath>
```

<img width="625" height="291" alt="gobuster" src="https://github.com/user-attachments/assets/71ca7682-3699-46a1-b67e-f3056d2b4c3d" />

Gobuster revealed an interesting directory — `/panel/` — which contained a file upload functionality, a potential RCE vector for deploying a web shell.

<img width="1280" height="718" alt="uploads" src="https://github.com/user-attachments/assets/a51855f4-4034-46c3-be94-2b75fb23c48c" />

## Exploitation

I downloaded a PHP reverse shell from GitHub and set up a Netcat listener on port 8000.

```bash
nc -lvnp 8000
```

When I attempted to upload the PHP file, the application blocked it. Suspecting the filter only blacklisted the `.php` extension, I renamed the file to `.phtml` — a less common PHP extension often overlooked in blacklists. The upload was accepted and the shell executed successfully.

<img width="623" height="107" alt="web" src="https://github.com/user-attachments/assets/d29d716a-000f-48d0-9126-7576f982e3b1" />

With shell access established, I searched for the first CTF flag (`user.txt`).

```bash
find / -type f -name "user.txt" 2>/dev/null
```

The flag was located at `/var/www/user.txt`.

## Privilege Escalation

With `user.txt` retrieved, I needed to escalate to root to access `root.txt`. I searched for binaries with the **SUID bit** set — which causes the file to execute with the owner's UID rather than the executing user's UID.

```bash
find / -perm /4000 2>/dev/null
```

<img width="433" height="291" alt="pyro" src="https://github.com/user-attachments/assets/975b8d90-c92c-4526-9085-d0227a012dbf" />

I found that **Python** had the SUID bit set and was owned by root — an ideal escalation vector. I referenced GTFOBins and used the following command to spawn a root shell.

```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

I confirmed root access with `whoami`, navigated to `/root`, and retrieved the second and final flag (`root.txt`), successfully completing the machine.

## Conclusion

RootMe presented a classic and realistic web exploitation scenario followed by Linux privilege escalation. The attack began with a misconfigured file upload feature that could be bypassed by using an alternative PHP extension (`.phtml`) — not covered by the blacklist. This allowed remote code execution and initial web shell access.

Privilege escalation was achieved by identifying a binary with improper SUID permissions, demonstrating how incorrect configuration of special file permissions can lead to full system compromise. This machine reinforces key offensive security concepts including file upload filter bypass, web interpreter behavior, Linux permission management, and enumeration-driven privilege escalation.
