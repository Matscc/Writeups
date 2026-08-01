# Machine: Blue (TryHackMe)

## Enumeration

I started the enumeration by running a port scan with Nmap to identify the available services on the target machine.

```bash
nmap -sS <targetIp>
```

<img width="569" height="276" alt="nmap blue" src="https://github.com/user-attachments/assets/e7f7c214-e8ec-4e67-8bcc-6da9f66e1514" />

Nmap revealed an SMB service running on port 445. The service allowed anonymous access, but there was nothing directly exploitable. I then ran a Nmap vulnerability script to check the SMB version and identify known vulnerabilities.

```bash
nmap --script vuln -p 445 <targetIp>
```

<img width="856" height="362" alt="nmap smbv1 blue" src="https://github.com/user-attachments/assets/5356bc43-e5eb-4328-8df7-06c4b2cf0f8a" />

The script revealed that the SMB version in use was **SMBv1**, which is affected by the well-known vulnerability **MS17-010 (EternalBlue)** — allowing Remote Code Execution (RCE) and full control over the target machine.

## Exploitation

I used Metasploit to search for exploits targeting the MS17-010 vulnerability.

<img width="1280" height="646" alt="exploit ms17-010 blue" src="https://github.com/user-attachments/assets/3976147c-f43c-4131-9b9b-23600b2afd21" />

I selected exploit **0** and configured the required options.

<img width="1280" height="631" alt="requirements blue" src="https://github.com/user-attachments/assets/0aa7a86f-3426-420e-818a-cfd4ddae8300" />

Only the target IP and local machine IP were required. I also set a specific payload as requested by the TryHackMe task instead of using the default one, then ran the exploit.

<img width="1280" height="557" alt="payload blue" src="https://github.com/user-attachments/assets/bc9a2400-1ea7-4079-9a83-b6cdfbf0892e" />

With shell access established, I upgraded to a Meterpreter session for better post-exploitation capabilities.

<img width="1050" height="494" alt="shell to meterpreter blue" src="https://github.com/user-attachments/assets/34800dc5-1a1d-40b5-a3d0-f0da56411938" />

I then verified the active sessions to confirm I had **NT AUTHORITY\SYSTEM** privileges.

<img width="1280" height="160" alt="sessions blue" src="https://github.com/user-attachments/assets/841dab5c-3af1-4af5-9952-17df8b7c5d3a" />

## Post-Exploitation

With full system access confirmed, I proceeded to find the CTF flags.

The first flag was located at the root of the filesystem.

<img width="1280" height="317" alt="flag 1 blue" src="https://github.com/user-attachments/assets/67ce7e6c-a40d-4b54-9f73-227ced763899" />

The second flag was stored where passwords are kept on the system.

<img width="750" height="291" alt="flag 2 blue" src="https://github.com/user-attachments/assets/76e18e56-8139-497d-a739-ac721a2233cd" />

The third and final flag was found in user **Jon**'s Documents directory.

<img width="574" height="164" alt="flag 3 blue" src="https://github.com/user-attachments/assets/b940940e-aa53-4c60-a864-e4ea3d210835" />

## Conclusion

The Blue machine demonstrated the critical impact of outdated and misconfigured services in Windows environments. Through initial enumeration with Nmap, it was possible to identify a vulnerable SMB service, which immediately indicated the possibility of exploiting the MS17-010 (EternalBlue) vulnerability.

Exploiting this flaw allowed unauthenticated Remote Code Execution, resulting in direct system access with maximum privileges (NT AUTHORITY\SYSTEM). This highlights how devastating network service vulnerabilities can be, especially when combined with legacy protocols such as SMBv1.
