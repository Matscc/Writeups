# Machine: Fawn (Hack The Box)

## Enumeration

With the target machine's IP address available, I started enumerating the running services.

I used Nmap to scan the most common ports with the following flags: `-sV` (identifies service versions) and `-sC` (runs default scripts against discovered services).

```bash
nmap -sC -sV <TARGET IP>
```

After the scan completed, port 21 was found open with an FTP service running. The default Nmap scripts also identified that **anonymous login** was enabled on the service.

<img width="676" height="310" alt="FAWN" src="https://github.com/user-attachments/assets/3bfafd6f-d6d3-4d14-b405-5a03729189f3" />

## Exploitation

I attempted to log into the FTP service using `anonymous` as both the username and password — which granted access to the service.

Once connected, I ran `ls` to list the available files and found `flag.txt`. I used the `get` command to download it to my local machine and read its contents, successfully completing the machine.

<img width="640" height="320" alt="FAWN1" src="https://github.com/user-attachments/assets/410421fd-9dd2-4f94-88a6-2b48a0702298" />

## Conclusion

The Fawn machine exploits an insecure FTP configuration with anonymous access enabled, allowing any user to connect to the service and download sensitive files without proper authentication.
