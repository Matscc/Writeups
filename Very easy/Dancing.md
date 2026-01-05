# Máquina: Dancing (Hack The Box)

## Enumeração
Inicialmente realizei uma varredura para identificar os serviços disponíveis na máquina alvo.  
````bash
nmap -sC -sV <target ip>
````
<img width="783" height="391" alt="Dancing NMAP" src="https://github.com/user-attachments/assets/19dea730-b774-456f-bff9-39b53cb3b4cb" />



Foi possível identificar a porta 445 aberta. Esta porta é comumente utilizada para o serviço SMB, um serviço microsoft que permite o compartilhamento de arquivos.  
Utilizando o smbclient, tentei fazer a conexão com a máquina alvo anonimamente para verificar se havia compartilhamentos acessivéis sem autenticação.  
````bash
smbclient -L <Target ip> -N
````
-L = lista os compartilhamentos disponíveis para este host.   
-N = Faz uma conexão ao serviço sem password.

<img width="448" height="121" alt="SMB enum" src="https://github.com/user-attachments/assets/575bfd65-75bb-4084-bf6f-6c9926678a6a" />


## Exploração
O serviço nos apresentou 4 diretórios de compartilhamento de arquivos, verifiquei se algum compartilhamento permitia entrada sem autenticação e somente o diretorio "Workshares" permitiu a entrada.
````bash
smbclient\\\\target ip\\WorkShares
````
<img width="667" height="133" alt="diretorios" src="https://github.com/user-attachments/assets/3265a5dc-8c87-449f-b27b-e94c2b4d9f1e" />

Já no diretório, havia duas pastas de arquivos, na "Amy. J" não havia nada de relevante para nós, mas dentro de "James.P" havia o arquivo "flag.txt", então, baixei o arquivo. Assim, a máquina foi finalizada com sucesso.

<img width="831" height="167" alt="James" src="https://github.com/user-attachments/assets/afdc1f43-3b3b-4447-ada1-fd954b842cf2" />



## Conclusão
A máquina Dancing explora uma configuração insegura do serviço SMB, que permite acesso anônimo aos compartilhamentos do sistema. Por meio dessa falha, foi possível enumerar os recursos disponíveis, acessar arquivos sensíveis e obter a flag sem a necessidade de autenticação válida.

