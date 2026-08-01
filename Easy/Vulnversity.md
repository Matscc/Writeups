# Machine: Vulnversity (TryHackMe)

## Enumeration
I started enumeration by running a port scan with Nmap to identify active services on the target machine:
```bash
nmap -sV <target_ip>
```
<img width="795" height="249" alt="nmap vul" src="https://github.com/user-attachments/assets/93e994bc-85ac-4889-9fb1-5194df99c66d" />  

The scan revealed 6 open ports, notably:  
21 – FTP  
22 – SSH  
3333 – HTTP  

With this information, I decided to investigate the web application running on port 3333.

## Exploitation
Upon accessing the web application, I performed directory enumeration using Gobuster to discover hidden paths:  

```bash
gobuster dir -u http://<target_ip>:3333 -w <wordlist>
```

<img width="804" height="269" alt="gobuster vul" src="https://github.com/user-attachments/assets/47b46af4-fa54-4421-b22e-6d9fe3544479" />  

During enumeration, I discovered the `/internal` directory, which allowed file uploads:  

<img width="1280" height="606" alt="internal vul" src="https://github.com/user-attachments/assets/4132bbe3-bd79-4f89-8f84-dfccd79f344d" />

Testing the upload functionality revealed that `.php` extensions were blocked by a blacklist. To bypass this restriction, I used the `.phtml` extension, which is also executed as PHP by the server but was not blacklisted.  

I prepared a PHP reverse shell, started a listener on my machine, and uploaded the payload. Next, I accessed the uploaded file through the uploads directory, triggering the execution of the reverse shell and granting me shell access to the target host.  
Once inside, I navigated the system and located the first flag, "user.txt", in the home directory of the user "Bill".  

## Privilege Escalation
After obtaining initial access as an unprivileged user, I conducted local enumeration to search for potential privilege escalation vectors, focusing on binaries with the SUID bit set:
```bash
find / -perm /4000 2>/dev/null
```
During this check, I identified that the `systemctl` binary had SUID permissions enabled, allowing it to be leveraged for privilege escalation.

<img width="552" height="316" alt="system vul" src="https://github.com/user-attachments/assets/a5aa11af-ea22-40d6-857b-e7ea30f80c6f" />  

Knowing this, I created a custom service file named `root.service` on my local machine designed to execute commands as root.

<img width="605" height="183" alt="rot service" src="https://github.com/user-attachments/assets/be0fe01a-bb6c-4359-9e69-f10887997959" />  

Next, I hosted a Python HTTP server on my machine to serve the file:
```bash
python3 -m http.server 8000
```
I downloaded the service file onto the target machine and enabled it using `systemctl`:
```bash
systemctl link /tmp/root.service
systemctl enable root.service
systemctl start root.service
```
Upon starting the service, it executed the payload with root privileges, establishing a reverse shell connection back to my machine.  
With root access achieved, I navigated to the `/root` directory and extracted the second and final flag, "root.txt", successfully finishing the machine.

## Conclusion

The target machine presented a clear and well-structured attack path, combining common web application vulnerabilities with insecure operating system configurations. Effective initial enumeration allowed identifying a web application susceptible to file upload bypass, where inadequate validation permitted running a PHP reverse shell.

The initial foothold highlighted the importance of thorough local enumeration, leading to the discovery of a critical SUID binary. Exploiting `systemctl` to execute custom malicious services demonstrated how privilege misconfigurations can lead to complete system takeover, even from a restricted initial shell.
