# Machine: Facts (Hack The Box)

## Enumeration

I started by running an Nmap scan on the target to discover open ports and running services.

<img width="675" height="95" alt="nmap" src="https://github.com/user-attachments/assets/24019bdc-ab2e-4581-8b7f-3313ce89402e" />

A web service was found running, with what appeared to be a simulated **AWS environment** being served on port **54321**.

Accessing the web application, I used Gobuster to enumerate existing directories and discovered an **admin login page**.

<img width="751" height="254" alt="gobuster" src="https://github.com/user-attachments/assets/2cde337b-6fcb-4cbb-889b-f3cf3109808a" />

The login page offered the option to create a new account. Since I had no credentials, I registered with `joao:joao123`. After logging in, the dashboard revealed the site was running **Camaleon CMS version 2.9.0**.

<img width="1440" height="615" alt="camaleon version" src="https://github.com/user-attachments/assets/939d45df-7cb5-49f8-9a15-8a17af5626e9" />

This version of Camaleon CMS is vulnerable to **CVE-2025-2304**, which allows privilege escalation within the CMS through a **mass assignment** vulnerability. The flaw exists because the `update_ajax` function allows a regular user to update their profile data without filtering which fields can be modified, due to the use of `permit!`.

```ruby
@user.update(params[:password].permit!)
```

## Exploitation

I opened **Burp Suite** to intercept the request triggered when changing my user's password. I modified the request by adding the parameter `password[role]=admin`, escalating my account to admin within the CMS.

<img width="550" height="465" alt="admin cms" src="https://github.com/user-attachments/assets/b65fd426-c80f-4f00-92f7-2b331887f2da" />

> `password%5Brole%5D=admin` (URL encoded)

With admin access, I was able to view sensitive information exposed in the CMS system files — including **AWS S3 access keys** for the simulated service running on port 54321.

<img width="424" height="650" alt="s3 keys" src="https://github.com/user-attachments/assets/22300744-80c5-4056-a6fc-cb3a2b223a90" />

Investigating the S3 service, I found a **private SSH key**.

<img width="888" height="150" alt="aws" src="https://github.com/user-attachments/assets/0cf47be9-a4f8-4548-aa89-8930d1d72138" />

The private key was password-protected. I used **ssh2john** to convert it into a crackable hash format, then used **John the Ripper** to crack the passphrase and successfully connected via SSH.

<img width="1440" height="370" alt="john" src="https://github.com/user-attachments/assets/1ba9311a-987b-40f9-b750-a3884244f6cf" />

<img width="584" height="507" alt="ssh" src="https://github.com/user-attachments/assets/170e1b6d-a5fd-483c-9aac-31d6b7f75a48" />

The user flag was found in **William**'s home directory.

<img width="386" height="130" alt="userflag" src="https://github.com/user-attachments/assets/97c8e31b-ac55-4aab-84f4-76cea9bf8250" />

## Privilege Escalation

After retrieving the user flag, I ran `sudo -l` to check which binaries the current user could execute as root. The `facter` binary was listed.

<img width="610" height="97" alt="sudo -l" src="https://github.com/user-attachments/assets/0858fe4d-9d81-4a9b-9042-36efee0b1dc1" />

Researching `facter`, I found it is an automation tool used to list system configurations. It can be pointed at a custom directory containing a malicious Ruby file, which it will execute as root.

I created a file named `exploit.rb` containing a script that spawns a bash shell.

```bash
echo 'exec "/bin/bash"' > exploit.rb
```

<img width="577" height="109" alt="root and rootflag" src="https://github.com/user-attachments/assets/6c7f5dc7-843b-4892-bbb1-fdc6c0eec38d" />

Running `facter` with sudo pointed at my exploit file gave me a **root shell**. I retrieved the root flag, successfully completing the machine.
