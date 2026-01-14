# Máquina: Pickle Rick(Try Hack Me)
## Enumeração
Inicialmente realizei uma varredura para identificar os serviços disponíveis na máquina alvo.
```bash
nmap -sV <targetIp>
```

<img width="785" height="182" alt="nmap rick" src="https://github.com/user-attachments/assets/805f5fbd-67a9-4be2-bb24-6431f16f38f8" />  

O scan revelou duas portas abertas:   
22 - Rodando o serviço SSH.  
80 - Rodando o serviço HTTP, (Havia uma aplicação web hospedada em Apache).  

Com essas informações, iniciei a análise da aplicação web acessando o IP da máquina pelo navegador.  
Ao inspecionar o código HTML da página inicial, encontrei um comentários informando um user do sistema, o que poderia ser útil em etapas posteriores.  

<img width="872" height="357" alt="html rick" src="https://github.com/user-attachments/assets/e956cc3f-6846-406d-bdf6-fc41fa396314" />  


Em seguida, verifiquei o arquivo robots.txt, onde encontrei uma palavra aparentemente sem sentido naquele momento, mas que foi anotada para possível uso futuro.  

<img width="279" height="187" alt="wuba rick" src="https://github.com/user-attachments/assets/a20f6163-cc53-4774-9050-3d378c462000" />  

Para continuar a enumeração web, utilizei o Gobuster para identificar diretórios ocultos na aplicação:  
```bash
gobuster dir -u http://<target_ip> -w <wordlist> -x php
```
<img width="699" height="324" alt="gobuster rick" src="https://github.com/user-attachments/assets/927575e8-895b-43bb-b37a-ffffedb4d64b" />  

Durante a enumeração, foi encontrado o diretório login.php.  

## Exploração
Ao acessar o diretório login.php, testei as credenciais obtidas anteriormente a partir da enumeração do código HTML, conseguindo realizar o login com sucesso.  
Após o login, fui redirecionado para o diretório portal, que disponibilizava uma funcionalidade perigosa:
um web shell, permitindo a execução direta de comandos no sistema através da aplicação web.

Nesse painel, foi possível localizar a primeira flag do CTF.  

<img width="1384" height="604" alt="painel rick" src="https://github.com/user-attachments/assets/cccc7b61-e8b9-4bca-948c-35f59481e744" />   

Com a possibilidade de executar comandos, optei por obter um acesso mcom menos retrições ao sistema. Para isso, utilizei um reverse shell em Python, encontrado no repositório "PayloadsAllTheThings" (100security).


Na minha máquina, iniciei um listener:
```bash
nc -lvnp 8000
````
E, no web shell, executei o payload de reverse shell, conseguindo acesso a um shell remoto com menos restrições.  
Explorando o sistema, localizei a segunda flag, presente no diretório do usuário "rick".

## Escalação de Privilégios
Com acesso ao sistema como usuário comum, o próximo passo foi verificar possíveis caminhos para escalar privilégios.  
Utilizei o comando:
```bash
sudo -l
```
O resultado revelou que o usuário possuía permissões totais para executar comandos como root, sem necessidade de senha.  
<img width="758" height="96" alt="nopasswd rick" src="https://github.com/user-attachments/assets/6367929a-16bf-48ed-8693-4aaecf7595d2" />  

Diante disso, a escalação de privilégios foi direta, utilizando:
```bash
sudo su
```
Com isso, obtive acesso como root.  
Ao acessar o diretório "/root", encontrei a terceira e última flag, finalizando a máquina com sucesso.

## Conclusão
A máquina Pickle Rick explora falhas comuns em aplicações web, como exposição de informações em comentários HTML, uso inadequado de robots.txt e, principalmente, a presença de um web shell sem restrições adequadas.

Além disso, a configuração incorreta de permissões sudo, permitindo acesso irrestrito ao root, torna a escalada de privilégios trivial após a obtenção de acesso inicial.





