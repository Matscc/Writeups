# Machine: Simple CTF (TryHackMe)

## Enumeration
I started with a simple enumeration of active services on the target machine using nmap:
```bash
nmap -sS <targetIp>
```

<img width="547" height="138" alt="Nmap simple" src="https://github.com/user-attachments/assets/42f0c9a6-ed8d-4388-a73e-9b1e9d6b2670" />  

Nmap revealed 3 active services:  
21 - ftp  
80 - http  
2222 - (A deeper nmap scan revealed that the actual service is SSH running on a non-conventional port)  

I started by accessing the FTP service, which allowed anonymous login. All I found was a message left for a developer named Mitch:  

<img width="1265" height="109" alt="Ftp simple" src="https://github.com/user-attachments/assets/decb52bd-04bd-404b-b18a-31474ba51590" />  

After that, I accessed the HTTP service, which did not seem to have anything relevant on the main page. Therefore, I used gobuster to find hidden directories on the application.  

<img width="672" height="211" alt="gobuster simple" src="https://github.com/user-attachments/assets/12fa4d4b-fd77-4f5b-9e6b-8c56ccf0a8ed" />  

Gobuster revealed the "simple" directory. Upon accessing it, I discovered that the web application was managed by "CMS Made Simple" version 2.2.8.

<img width="1280" height="720" alt="CMS simple" src="https://github.com/user-attachments/assets/108efa37-4b24-48d3-80cd-b0e8a00be062" /> 

## Exploitation

Knowing this, I used Metasploit to check if there was an exploit available for this version of the CMS. I found an exploit targeting an SQL injection vulnerability present in versions prior to 2.2.10, allowing information extraction from the web service.

<img width="1280" height="284" alt="exploit simple" src="https://github.com/user-attachments/assets/f8674be7-e369-49bf-89a9-8c4241354357" />  

Executing the exploit:  

```bash
python2 46635.py -u <url> -c <optionalPasswordCracking> -w <wordlistPath>
```

<img width="460" height="91" alt="py simple" src="https://github.com/user-attachments/assets/2fb7329d-c496-47f1-91da-4fdd53d697e3" />  

Upon completion, we uncovered the password "secret" and the username "mitch". With these credentials, we could connect to the SSH service on port 2222:  
```bash
ssh mitch@<targetip> -p 2222
```
After connecting successfully, we found the first flag, "user.txt".

## Privilege Escalation

Now with access to the target machine, I needed root access to retrieve the second flag. I ran the `sudo -l` command to check if there were any files we could execute with sudo privileges to gain root access.

<img width="474" height="82" alt="sudo - l simple" src="https://github.com/user-attachments/assets/d52e12c3-153b-40be-a155-7c0bad76247c" />  

To my surprise, we were allowed to execute `vim` as root via `sudo`, which guarantees direct root access. I checked GTFOBins and found a command to spawn a root shell by exploiting this misconfiguration in `vim`:  
```bash
vim -c ":!/bin/sh"
```
<img width="489" height="247" alt="root simple" src="https://github.com/user-attachments/assets/67eb976c-fd54-4297-bc58-3e9a90d6216a" />  

After running it, we obtained a root shell and found the second and final flag of the CTF, successfully completing the machine.

## Conclusion
The Simple CMS machine demonstrated in a practical way how a misconfigured and outdated web application can completely compromise a server's security. The exploitation began with basic service enumeration, identifying an application based on CMS Made Simple vulnerable to CVE-2019-9053.

This vulnerability allowed credential hashes to be extracted directly from the web application without prior authentication, highlighting a critical security flaw in SQL input handling. After obtaining the hash, password cracking was performed, securing legitimate access to the system via SSH.

Once inside the system, the next phase consisted of local enumeration to identify vectors for privilege escalation. The analysis revealed improper permissions associated with the `vim` binary, allowing execution with elevated privileges. By exploiting this misconfiguration, complete root access was achieved, fully compromising the system.
