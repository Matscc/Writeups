# Machine: Ignite (TryHackMe)

## Enumeration

I used Nmap to scan for available services running on the target machine.

```bash
nmap -sC -sV <targetip>
```

<img width="537" height="172" alt="nmap ignite" src="https://github.com/user-attachments/assets/42fb23b1-012f-4e64-bc39-a335cd15cbef" />

Nmap revealed an active web application. I proceeded to enumerate it.

<img width="1920" height="852" alt="web ignite" src="https://github.com/user-attachments/assets/c6928b57-9cd7-48cf-beea-467a103c2cc6" />

The home page immediately disclosed that the application was running **Fuel CMS version 1.4**. I searched for known vulnerabilities in this version and found a publicly available **RCE exploit — CVE-2018-16763**.

## Exploitation

I found an exploit for this vulnerability and modified it to target the correct URL.

<img width="1504" height="574" alt="exploit ignite" src="https://github.com/user-attachments/assets/5de913ec-f755-430b-8b7b-d06babf8b5f0" />

With command execution available on the web application, I hosted a Python HTTP server on my machine to serve a reverse shell file.

<img width="386" height="116" alt="reverse ignite" src="https://github.com/user-attachments/assets/32033cf2-6fa0-4adb-8d6f-ebe66d87ff7d" />

I downloaded the file on the target machine, executed it, and received a shell. The first flag was found under the **www-data** web user's directory.

<img width="319" height="48" alt="user ignite" src="https://github.com/user-attachments/assets/5bdafa3c-015c-4d05-a0d1-02dc923f73f5" />

To get a more stable shell, I spawned an interactive bash session using Python.

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

## Privilege Escalation

I needed to escalate to root. I recalled that the Fuel CMS home page had mentioned the path to the application's database configuration — which could contain sensitive credentials.

<img width="1413" height="854" alt="database ignite" src="https://github.com/user-attachments/assets/94eec7be-467f-40d2-a02e-db1298e515db" />

Accessing the database configuration file, I found the **root user's password** stored in plaintext.

<img width="640" height="501" alt="password ignite" src="https://github.com/user-attachments/assets/e560bc95-92e9-4c6a-b6d6-f74edfa17c1b" />

Using the discovered password, I successfully logged in as root.

<img width="518" height="88" alt="root ignite" src="https://github.com/user-attachments/assets/504c3225-541c-4de4-9be7-6be5c8c0d46c" />

With root access, I retrieved the second and final flag from the `/root` directory, successfully completing the machine.

## Conclusion

The Ignite machine was valuable for reinforcing key concepts in web enumeration, CMS exploitation, and privilege escalation. Identifying sensitive files and insecure configurations allowed initial access, demonstrating how simple flaws can fully compromise an application.

After gaining access, local enumeration revealed misconfigurations that enabled privilege escalation to root — highlighting the importance of properly defined permissions and secure protection of critical configuration files. This machine significantly contributed to developing logical reasoning and practical methodology in penetration testing.
