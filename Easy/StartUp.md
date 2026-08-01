# Machine: StartUp (TryHackMe)

## Enumeration 
First, I performed a port scan on the target machine using Nmap:
```bash
nmap -sC -sV <targetIp>
```

<img width="827" height="533" alt="nmap star" src="https://github.com/user-attachments/assets/a6dd2ccb-c36d-4ed8-a104-2ef694b9d5ef" />  

Nmap revealed 3 open ports:    
22 - SSH  
21 - FTP  
80 - HTTP    

Nmap also showed that the FTP service allowed anonymous authentication, containing the files "important.jpg", "notice.txt", and an "ftp" directory where I had write permissions. I connected to the service and downloaded the files to my machine, but there was nothing relevant in them.  

Next, I explored the web application. On the homepage, I was presented with a simple message stating that the site was under maintenance.

<img width="1272" height="694" alt="web star" src="https://github.com/user-attachments/assets/73d94146-1557-4ff2-82f6-84e4170bcc37" />  

I used Gobuster to discover hidden directories in the application:
```bash
gobuster dir -u <targetIP> -w <wordlist> -x php
```
<img width="674" height="159" alt="gobuster star" src="https://github.com/user-attachments/assets/196e4cc5-2166-466a-97b6-6f7b4788ed70" />  

Gobuster revealed the directory "files". Upon accessing it, I realized that the same FTP directory was available through the web application:

<img width="733" height="373" alt="files star" src="https://github.com/user-attachments/assets/ace28f6c-6eec-4fd9-880a-1394b7563a07" />

## Exploitation

With permission to upload files to the "ftp" directory, I could upload a PHP reverse shell and execute it through the web service.  
I uploaded the reverse shell to the directory:

<img width="418" height="143" alt="reverse star" src="https://github.com/user-attachments/assets/ae30903e-0c38-4b9f-877b-dfcd0e1d816a" />  

I set up a netcat listener on port "8000":
```bash
nc -lvnp 8000
```
I triggered the reverse shell on the web service and obtained access to the machine.  
Inside the target, I explored the user directories and found the user "lennie", but I did not have permission to access their folder. Exploring further, I found the "incidents" directory containing a "pcap" file.  
Analyzing this pcap file in Wireshark, I found a capture of the user lennie authenticating to the system, with the password exposed in cleartext:

<img width="865" height="703" alt="wire star" src="https://github.com/user-attachments/assets/974a4cfb-f534-4325-b693-abeb7c4946bc" />

With the username and password, I authenticated via SSH:
```bash
ssh lennie@<targetIp>
```
As "lennie", I accessed the home directory, which contained the first CTF flag "user.txt" as well as a directory named "scripts".

## Privilege Escalation
Inside the "scripts" directory were two files: "planner.sh" and "startup_list.txt".  
Checking "startup_list.txt" revealed nothing, but inspecting "planner.sh" showed a script that appended a list to "startup_list.txt" and then executed another file called "print.sh":

<img width="549" height="160" alt="planner star" src="https://github.com/user-attachments/assets/4e5784b7-d9d1-43f2-ac85-11cb87bad569" />  

I verified that these files belonged to root. Observing "startup_list.txt", I noticed it updated every minute, confirming an automated process running "planner.sh" every minute (a root cronjob executing "planner.sh").  

Although I didn't have write permissions to modify "planner.sh" directly, I could modify "print.sh".

<img width="631" height="184" alt="root star" src="https://github.com/user-attachments/assets/1d525d9f-c43f-4ccf-92ac-8dfa80d783c8" />  

I added a reverse shell command targeting my machine inside "print.sh". When "planner.sh" was executed by root, it would trigger my reverse shell.  
I started a listener on port 789:
```bash
nc -lvnp 789
```
After one minute, I gained root access on the target machine, accessed the `/root` directory, and found the second and final flag of the CTF, completing the machine successfully.

## Conclusion
The StartUp machine presents a chain of classic yet highly relevant security vulnerabilities in real-world scenarios. The exploitation demonstrates how simple configuration oversights, when combined, can lead to complete system compromise.

Initial access highlights the lack of adequate controls on exposed services and file permissions, allowing an attacker to achieve code execution with limited privileges. From there, privilege escalation occurs due to a root cronjob that executes scripts located in directories writable by unprivileged users. This misconfiguration leads to arbitrary command execution with elevated privileges, one of the most dangerous attack vectors in Linux environments.

The ability to modify a script called automatically by cron demonstrates a lack of permission segregation and file integrity validation for scheduled tasks. In a production environment, such a flaw would allow not only obtaining a root shell but also installing backdoors, modifying system users, and establishing persistent unauthorized access to the server.
