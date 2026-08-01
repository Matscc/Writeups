# Machine: Kenobi (TryHackMe)

## Enumeration

I started by running a port and service scan with Nmap.

```bash
nmap -sV <targetIp>
```

<img width="769" height="269" alt="Nmap ke" src="https://github.com/user-attachments/assets/e9231e00-3038-478c-82bb-fef1243dcb99" />

The scan revealed 7 open ports, with the following standing out:
- **21** — FTP
- **22** — SSH
- **80** — HTTP
- **111** — RPCBind
- **2049** — NFS

The presence of **RPCBind** and **NFS** suggested remotely accessible network shares, which motivated a deeper enumeration of those services.

To list the available NFS shares, I ran:

```bash
showmount -e <targetIP>
```

The command revealed that the `/var` directory was being shared and accessible from any IP.

<img width="266" height="44" alt="var ke" src="https://github.com/user-attachments/assets/5ef4af7b-c895-4217-ba07-9249bd688df2" />

I mounted the NFS share on my local machine.

```bash
mkdir /mnt/var
mount -t nfs <targetIp>:/var /mnt/var
```

Exploring the mounted directory, I found an **id_rsa** file — a passwordless SSH private key — representing a direct remote access vector.

<img width="677" height="154" alt="rsa" src="https://github.com/user-attachments/assets/ba60fe88-dbee-4e72-83b3-95387a7d4b9d" />

With SMB also active, I listed the available shares.

```bash
smbclient -L \\\\<targetIp>\\
```

An **anonymous** share was identified, accessible without authentication. I accessed it with:

```bash
smbclient \\\\<targetIp>\\anonymous
```

Inside the share, I found a file named `log.txt`. Analyzing it revealed service configuration details and, more importantly, a potential system username: **kenobi**.

<img width="571" height="182" alt="log" src="https://github.com/user-attachments/assets/44e59434-754b-488c-8af4-88a25c19ab1d" />

## Exploitation

With the username and the RSA private key, I connected via SSH.

```bash
ssh -i id_rsa kenobi@<targetIp>
```

After gaining access, I retrieved the first CTF flag (`user.txt`).

## Privilege Escalation

To escalate privileges, I searched for binaries with the **SUID bit** set, which can allow execution as root.

```bash
find / -perm /4000 2> /dev/null
```

The binary `/usr/bin/menu` was identified. When executed, it presented options such as:
- Display kernel version
- Show system status
- Run network commands

I analyzed the binary with:

```bash
strings /usr/bin/menu
```

I noticed it called `curl` **without using its absolute path**, making it vulnerable to **PATH Hijacking**.

<img width="678" height="409" alt="strings" src="https://github.com/user-attachments/assets/c3731dfd-d72c-4a49-aece-210c6dd968a1" />

In the `/tmp` directory, I created a fake `curl` binary.

```bash
echo /bin/sh > curl
chmod +x curl
```

I then modified the PATH environment variable to prioritize `/tmp`.

```bash
export PATH=/tmp:$PATH
```

When I ran `/usr/bin/menu` again and selected the **status** option, the program executed my fake `curl`, which spawned a `/bin/sh` shell. Since the `menu` binary was SUID and owned by root, the shell opened with **root privileges**.

With root access, I navigated to `/root` and retrieved the second and final flag (`root.txt`), successfully completing the machine.

## Conclusion

Kenobi reinforces the importance of careful enumeration and the combined analysis of exposed services. The initial access was obtained through a misconfigured NFS share that exposed a passwordless SSH private key, later confirmed by discovering the correct username through an anonymously accessible SMB share.

After gaining system access, the privilege escalation phase revealed a common but critical flaw in privileged binary development — a SUID binary calling commands without absolute paths allowed exploitation of the PATH environment variable, resulting in a root shell. This demonstrates how small insecure implementation decisions can have severe security impacts.
