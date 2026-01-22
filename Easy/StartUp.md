# Máquina: StartUp (Try Hack Me)

## Enumeração 
Primeiramente, realizei um scan na máquina alvo utilizando Nmap:
```bash
nmap -sC -sV <targetIp>
```

<img width="827" height="533" alt="nmap star" src="https://github.com/user-attachments/assets/a6dd2ccb-c36d-4ed8-a104-2ef694b9d5ef" />  

O nmap nos revelou 3 portas abertas:    
22 - SSH  
21 - FTP  
80 - HTTP    

O nmap também nos mostrou que o serviço ftp permite autenticação anônima, com os arquivos "important.jpg", "notice.txt" e o diretório "ftp" que me permitia adicionar outros arquivos. Entrei no serviço e baixei os arquivos na minha máquina, mas não  havia nada relevante neles.  

Em seguida, fui explorar a aplicação web. Na página inicial, só me foi apresentada uma mensagem informando que o site estava em manuntenção.

<img width="1272" height="694" alt="web star" src="https://github.com/user-attachments/assets/73d94146-1557-4ff2-82f6-84e4170bcc37" />  

Utilizei o gobuster para descobrir diretórios ocultos na aplicação:
```bash
gobuster dir -u <targetIP> -w <wordlist> -x php
```
<img width="674" height="159" alt="gobuster star" src="https://github.com/user-attachments/assets/196e4cc5-2166-466a-97b6-6f7b4788ed70" />  

O gobuster revelou o diretório "files" disponível, então, o acessando descobri que o mesmo serviço ftp da máquina também está disponível na aplicação web:

<img width="733" height="373" alt="files star" src="https://github.com/user-attachments/assets/ace28f6c-6eec-4fd9-880a-1394b7563a07" />

## Exploração

Então, com a permissão de adicionar arquivos ao diretório "ftp", eu poderia adicionar um reverse shell em php e executar no serviço web.  
Adicionei o reverse shell no diretório:

<img width="418" height="143" alt="reverse star" src="https://github.com/user-attachments/assets/ae30903e-0c38-4b9f-877b-dfcd0e1d816a" />  

Abri um listener na porta "8000" com o netcat:
```bash
nc -lvnp 8000
```
Executei o reverse shell no service web e consegui um shell na máquina.  
Já com acesso a máquina, fui explorar os diretórios de usuários e encontrei o usuário "lennie", mas não tinha acesso a ele, então ao explorar mais achei o diretório "incidents" e nele havia um arquivo "pcap".  
Analisando este pcap no wireshark, encontrei a connexão do usuário lennie ao sistema e sua senha estava em evidência:


<img width="865" height="703" alt="wire star" src="https://github.com/user-attachments/assets/974a4cfb-f534-4325-b693-abeb7c4946bc" />

Agora com a senha e o usuário, eu posso fazer a autenticação pelo serviço ssh:
```bash
ssh lennie@<targetIp>
```
No sistema como "lennie" eu posso acessar o diretório, nele havia a primeira flag do ctf "user.txt" e também havia um diretório chamado "scripts".

## Escalação de privilégios
No diretório "scripts" havia o arquivo "planner.sh" e "startup_list.txt".  
Dando um cat no "startup_list.txt" não havia nada presente, mas dando um cat no planner.sh havia um script que repassava uma lista exatamente para o "startup_list.txt" e logo em seguida executava um arquivo chamado "print.sh":

<img width="549" height="160" alt="planner star" src="https://github.com/user-attachments/assets/4e5784b7-d9d1-43f2-ac85-11cb87bad569" />  

Verifiquei que esse arquivos pertenciam ao root e fiquei obervando o "startup_list.txt" para ver se ele recebia atualizações. Descobri que o arquivo recebia atualizações a cada um minuto, então com certeza havia uma automatização que executava o arquivo "planner.sh" a cada um minuto, (havia um cronjob root que executava o "planner.sh").  


Como eu não havia permissão de modificar o planner.sh diretamente, eu poderia modificar o "print.sh".

<img width="631" height="184" alt="root star" src="https://github.com/user-attachments/assets/1d525d9f-c43f-4ccf-92ac-8dfa80d783c8" />  

Adicionei um reverse shell para minha máquina no arquivo, que faria que quando o "planner.sh" fosse executado, me daria um shell root.  
Abri um listener na porta 789:
```bash
nc -lvnp 789
```
Após um minuto, consegui acesso root na máquina alvo, acessei o diretório "/root" e encontrei a segunda e última flag do CTF, finalizando esta máquina com sucesso.

## Conclusão
A máquina StartUp apresenta uma cadeia de vulnerabilidades clássicas, porém extremamente relevantes no contexto de segurança real. A exploração demonstra como falhas simples de configuração, quando combinadas, podem resultar em comprometimento total do sistema.

O acesso inicial evidencia a ausência de controles adequados sobre serviços expostos e permissões de arquivos, permitindo que um atacante obtenha execução de código com privilégios limitados. A partir desse ponto, a escalada de privilégios ocorre devido a um cronjob executado como root que faz uso de scripts armazenados em diretórios graváveis por usuários não privilegiados. Essa falha configura um cenário crítico de execução arbitrária de comandos com privilégios elevados, um dos vetores mais perigosos em ambientes Linux.

A possibilidade de modificar o script chamado automaticamente pelo cronjob demonstra a falta de segregação de permissões e de validação da integridade dos arquivos executados por tarefas agendadas. Em um ambiente real, esse tipo de erro permitiria não apenas a obtenção de um shell root, mas também a instalação de backdoors, alteração de usuários do sistema e comprometimento persistente do servidor.











