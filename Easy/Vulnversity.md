# Máquina: Vulnversity(Try Hack Me)
## Enumeração
Iniciei a enumeração realizando um scan de portas com o Nmap para identificar os serviços disponíveis na máquina alvo:
```bash
nmap -sV <target_ip>
```
<img width="795" height="249" alt="nmap vul" src="https://github.com/user-attachments/assets/93e994bc-85ac-4889-9fb1-5194df99c66d" />  

 O scan revelou 6 portas abertas, com destaque para:  
21 – FTP  
22 – SSH  
3333 – HTTP  

Com essas informações, decidi investigar a aplicação web rodando na porta 3333.

## Exploração
Ao acessar a aplicação web, realizei enumeração de diretórios utilizando o Gobuster, com o objetivo de identificar caminhos ocultos:  

```bash
gobuster dir -u http://<target_ip>:3333 -w <wordlist>
```

<img width="804" height="269" alt="gobuster vul" src="https://github.com/user-attachments/assets/47b46af4-fa54-4421-b22e-6d9fe3544479" />  

Durante a enumeração, descobri o diretório /internal, que permitia upload de arquivos:  


<img width="1280" height="606" alt="internal vul" src="https://github.com/user-attachments/assets/4132bbe3-bd79-4f89-8f84-dfccd79f344d" />

Ao testar o upload, percebi que extensões .php eram bloqueadas por uma blacklist. Para contornar essa restrição, utilizei a extensão .phtml, que também é interpretada como PHP pelo servidor, mas não estava bloqueada.  

Preparei um reverse shell em PHP, iniciei um listener na minha máquina e realizei o upload do arquivo malicioso. Em seguida, acessei o arquivo enviado pelo diretório de uploads, o que resultou na execução do reverse shell e me concedeu acesso à máquina alvo.  
Com o acesso obtido, consegui navegar pelo sistema e encontrei a primeira flag "user.txt" no diretório do usuário "Bill".  

## Escalação de privilégios
Após obter acesso como usuário comum, iniciei a enumeração local em busca de possíveis vetores de escalação de privilégios, focando principalmente em binários com o bit SUID ativado:
```bash
find / -perm /4000 2>/dev/null
```
Durante essa enumeração, identifiquei que o binário systemctl possuía permissões que permitiam sua exploração para obter privilégios elevados.


<img width="552" height="316" alt="system vul" src="https://github.com/user-attachments/assets/a5aa11af-ea22-40d6-857b-e7ea30f80c6f" />  

Sabendo disso, criei na minha máquina um arquivo malicioso chamado "root.service", que continha um serviço capaz de executar comandos como root. 



<img width="605" height="183" alt="rot service" src="https://github.com/user-attachments/assets/be0fe01a-bb6c-4359-9e69-f10887997959" />  




Em seguida, abri um servidor Python na minha máquina para disponibilizar o arquivo:
```bash
python3 -m http.server 8000
```
baixei o arquivo na máquina alvo e executei o serviço utilizando o systemctl.
```bash
systemctl link /tmp/root.service
systemctl enable root.service
systemctl start root.service
```
Ao iniciar o serviço, ele executou o payload com privilégios de root, estabelecendo uma conexão reversa com minha máquina.  
Com acesso root obtido, naveguei até o diretório /root e encontrei a segunda e última flag "root.txt". Assim, finalizando a máquina com sucesso.

## Conclusão

A máquina explorada apresentou um fluxo de ataque claro e bem estruturado, combinando falhas comuns em aplicações web com configurações inseguras no sistema operacional. A partir de uma enumeração inicial eficaz, foi possível identificar uma aplicação web vulnerável a upload de arquivos, onde a ausência de uma validação adequada permitiu o bypass da blacklist de extensões e a execução de um reverse shell.

O acesso inicial obtido evidenciou a importância de uma enumeração local cuidadosa, que levou à identificação de um binário crítico com permissões inadequadas. A possibilidade de explorar o systemctl para executar serviços maliciosos demonstrou como configurações incorretas de privilégios podem comprometer totalmente a segurança de um sistema, mesmo após um acesso inicial limitado.










