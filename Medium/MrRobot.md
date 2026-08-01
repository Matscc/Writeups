# Machine: Mr. Robot (TryHackMe)

## Enumeration

I started the enumeration process by running an Nmap scan to identify the services available on the target machine:

```bash
nmap -sV <target_ip>
```

<img width="562" height="141" alt="nmap robot" src="https://github.com/user-attachments/assets/eb834e34-45c3-45c1-a9d2-77012f8eeaf4" />

The scan revealed three open ports on the target machine:

- **22** - Running SSH
- **80** - Running HTTP
- **443** - Running HTTPS

After identifying the open ports with Nmap, I moved on to enumerate the web application hosted on port 80.

The application initially displayed a menu that didn't provide any useful information, so I decided to enumerate the available directories using Gobuster.

```bash
gobuster dir -u <targetIp> -w <localWordlist>
```

<img width="415" height="798" alt="gobuster robot" src="https://github.com/user-attachments/assets/d7873891-66cb-47f9-afd2-a722a629f135" />

Gobuster revealed several directories. The most interesting ones were **robots.txt** and the WordPress login page **wp-login**.

## Exploitation

Inside the **robots.txt** file, I found the first CTF flag as well as a wordlist that could be used to enumerate WordPress users.

*The WordPress version disclosed whether the username existed by returning a different error message when only the password was incorrect, making user enumeration possible.*

The wordlist contained many duplicate entries, so I cleaned it before using it:

```bash
sort wordlist.txt | uniq > cleanWordlist.txt
```

With the cleaned wordlist, I launched a brute-force attack against the WordPress login page using Hydra:

```bash
hydra -L <cleanWordlist.txt> -p "test" <targetIp> http-post-form "/login.php:log=^USER^&pwd=^PASS^:Invalid Username"
```

- The login form fields were named **log** and **pwd**, so I specified them in the Hydra command.
- **Invalid Username** is the error message displayed when the username does not exist, allowing Hydra to distinguish valid usernames.

<img width="763" height="136" alt="elliot robot" src="https://github.com/user-attachments/assets/83509584-1d67-4110-bf15-61a8afe71a86" />

Hydra successfully identified the user **Elliot**, so the next step was to find the password.

```bash
hydra -l "Elliot" -P <cleanWordlist.txt> <targetIp> http-post-form "/login.php:log=^USER^&pwd=^PASS^:The password you entered for the username"
```

<img width="825" height="146" alt="senha robot" src="https://github.com/user-attachments/assets/017eff36-b799-4293-94b2-e060bc467091" />

After discovering the credentials, I logged into the WordPress admin panel.

From there, I searched for a way to obtain a reverse shell by uploading a PHP file. Under **Appearance**, while editing the **Archives** template, I found that it was possible to upload a PHP reverse shell.

<img width="1681" height="772" alt="reverse mr robot" src="https://github.com/user-attachments/assets/ceaa6fec-90ac-4f2a-b9f6-dfd2385c763e" />

I started a Netcat listener on port **8000** on my machine and then accessed the uploaded reverse shell at:

`wp-content/themes/twentyfifteen/archive.php`

This successfully established a reverse shell to the target machine.

<img width="351" height="124" alt="shell robot" src="https://github.com/user-attachments/assets/9a033c09-55a8-4f11-aa46-b729b5091417" />

Once inside the system, I searched for the second flag. I found the **robot** user's home directory, but I didn't have permission to read the flag.

Inside the same directory, I found an MD5 password hash. After cracking the hash (this can be done using publicly available online tools), I switched to the **robot** user and successfully retrieved the second CTF flag.

## Privilege Escalation

The next objective was to obtain root privileges.

To do this, I searched for binaries with the SUID bit set:

```bash
find / -perm /4000 2> /dev/null
```

The scan revealed that **Nmap** had the SUID bit enabled, making it exploitable.

<img width="429" height="274" alt="nmapvuln robot" src="https://github.com/user-attachments/assets/10d66d2e-e4de-4187-9f95-827c3705d4d0" />

I visited **GTFOBins** and searched for Nmap privilege escalation techniques. I found that Nmap's interactive mode could be used to spawn a shell. Since Nmap was running with the SUID bit, the spawned shell would execute with root privileges.

<img width="1110" height="600" alt="gtfobins robot" src="https://github.com/user-attachments/assets/ec801913-a228-4d03-a0c2-4990b3061a94" />

```bash
nmap --interactive
!sh
```

This granted me root access to the system.

I navigated to the **/root** directory and found the third and final flag, successfully completing the machine.

## Conclusion

The Mr. Robot machine demonstrates how multiple security weaknesses can be chained together to achieve full system compromise.

The initial vulnerability lies within the web application, which exposes sensitive information and allows user enumeration using tools such as Hydra. Weak credentials, combined with the lack of protection against brute-force attacks and username enumeration, make initial access relatively straightforward.

Another critical issue is the insecure WordPress configuration, where administrative users can be identified and exploited due to poor security practices, including weak authentication, unlimited login attempts, and unnecessary information disclosure.

After gaining access, the operating system presents additional security misconfigurations, particularly regarding file permissions. The presence of SUID binaries configured insecurely allows a low-privileged user to execute commands with elevated privileges, ultimately leading to full root compromise.

These issues highlight the absence of fundamental security principles such as the Principle of Least Privilege, proper permission management, and system hardening. In a real-world environment, vulnerabilities like these could lead to complete server compromise, exposure of sensitive information, and abuse of the infrastructure for further attacks.

Overall, the Mr. Robot machine demonstrates how individually simple vulnerabilities can be chained together into a complete attack path, emphasizing the importance of secure application development, proper system configuration, and defense-in-depth security practices.


