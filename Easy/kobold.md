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
