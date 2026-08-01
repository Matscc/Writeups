# Machine: Dancing (Hack The Box)

## Enumeration

I started by running a port scan to identify the available services on the target machine.

```bash
nmap -sC -sV <target ip>
```

<img width="783" height="391" alt="Dancing NMAP" src="https://github.com/user-attachments/assets/19dea730-b774-456f-bff9-39b53cb3b4cb" />

Port 445 was found open. This port is commonly used by the SMB service, a Microsoft protocol that enables file sharing across a network.

Using smbclient, I attempted an anonymous connection to the target machine to check whether any shares were accessible without authentication.

```bash
smbclient -L <target ip> -N
```

- `-L` — lists the available shares on the host.
- `-N` — connects to the service without a password.

<img width="448" height="121" alt="SMB enum" src="https://github.com/user-attachments/assets/575bfd65-75bb-4084-bf6f-6c9926678a6a" />

## Exploitation

The service returned 4 shared directories. I checked each one to find out which allowed unauthenticated access — only the **WorkShares** directory permitted entry without credentials.

```bash
smbclient \\\\<target ip>\\WorkShares
```

<img width="667" height="133" alt="directories" src="https://github.com/user-attachments/assets/3265a5dc-8c87-449f-b27b-e94c2b4d9f1e" />

Inside the share, there were two folders. The **Amy.J** directory contained nothing of interest, but inside **James.P** there was a file named `flag.txt`. I downloaded the file and successfully completed the machine.

<img width="831" height="167" alt="James" src="https://github.com/user-attachments/assets/afdc1f43-3b3b-4447-ada1-fd954b842cf2" />

## Conclusion

The Dancing machine exploits an insecure SMB configuration that allows anonymous access to shared resources. By leveraging this misconfiguration, it was possible to enumerate available shares, access sensitive files, and retrieve the flag without valid credentials.
