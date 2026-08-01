# Machine: Meow (Hack The Box)

## Enumeration

With the target machine's IP address available, I started enumerating the running services.

I used Nmap to identify open ports and services with the `-sV` flag (service version detection).

```bash
nmap -sV <target ip>
```

- Port 23/TCP was found open
- Telnet service running

> **Note:** Telnet is a network protocol that allows remote access to another computer. It is considered insecure because it transmits data without encryption.

The enumeration revealed a remote access service running with an insecure configuration.

<img width="678" height="179" alt="Nmap scan meow" src="https://github.com/user-attachments/assets/51eae1f2-94ca-4599-839b-da7739297b7f" />

---

## Exploitation

With the Telnet port exposed, I attempted a remote connection to the target.

```bash
telnet <target ip>
```

<img width="436" height="195" alt="telnet login" src="https://github.com/user-attachments/assets/74f3662d-89a7-4c4d-8057-a8257694cac8" />

- The Telnet service was misconfigured
- Authentication was possible using a generic login with no valid credentials
- This misconfiguration allowed direct access to the system

Once inside, I listed the files and found `flag.txt`, successfully completing the machine.

<img width="378" height="84" alt="flag" src="https://github.com/user-attachments/assets/2352f783-77b6-478b-8704-104b654b09ed" />

---

## Conclusion

The Meow machine presents a basic misconfiguration by exposing the Telnet service with default credentials, allowing unauthorized access to the system. The lack of proper authentication enables an attacker to gain direct shell access and read sensitive files.
