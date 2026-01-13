# Máquina: Kenobi(Try Hack me)
## Enumeração
Iniciei a enumeração realizando uma varredura de portas e serviços com o Nmap:
```bash
nmap -sV <targetIp>
````
<img width="769" height="269" alt="Nmap ke" src="https://github.com/user-attachments/assets/e9231e00-3038-478c-82bb-fef1243dcb99" />


O scan revelou 7 portas abertas, destacando:    
21 - FTP  
22 - SSH  
80 - HTTP  
111 - RPCBind  
2049 - NFS  

A presença do serviço RPCBind e NFS indicava compartilhamentos NFS acessíveis remotamente, o que motivou uma enumeração mais aprofundada desse serviço.  
Para verificar os compartilhamentos NFS disponíveis, utilizei:  
```bash
showmount -e <targetIP>
```
O comando revelou que o diretório /var estava sendo compartilhado e acessível para qualquer IP.

<img width="266" height="44" alt="var ke" src="https://github.com/user-attachments/assets/5ef4af7b-c895-4217-ba07-9249bd688df2" />


Em seguida, montei o compartilhamento NFS na minha máquina local:  
```bash
mkdir /mnt/var
mount -t nfs <targetIp>:/var /mnt/var
```
Ao explorar o conteúdo do diretório montado, encontrei um arquivo id_rsa, uma chave privada SSH sem senha, o que indicava um possível vetor de acesso remoto ao sistema.

<img width="677" height="154" alt="rsa" src="https://github.com/user-attachments/assets/ba60fe88-dbee-4e72-83b3-95387a7d4b9d" />


Com o serviço SMB ativo, realizei a listagem dos compartilhamentos disponíveis:  
```bash
smbclient -L \\\\<targetIp>\\
```
Foi identificado o compartilhamento anonymous, acessível sem autenticação. Acessei-o com:  
```bash
smbclient \\\\<targetIp>\\anonymous
```
Dentro do compartilhamento, encontrei o arquivo "log.txt". Ao analisá-lo, foi possível identificar informações de configuração de serviços e, mais importante, um possível usuário do sistema: "kenobi".

<img width="571" height="182" alt="log" src="https://github.com/user-attachments/assets/44e59434-754b-488c-8af4-88a25c19ab1d" />


## Exploração

Com as informações de usuário e chave RSA em mãos, realizei a conexão SSH:
```bash
ssh -i id_rsa kenobi@<targetIp>
```
Após o acesso, localizei a primeira flag do CTF, "user.txt".

## Escalação de privilégios
Para escalar privilégios, iniciei a busca por binários com o bit SUID, que podem permitir execução como root:
```bash
find / -perm /4000 2> /dev/null
```
Foi identificado o binário /usr/bin/menu, que ao ser executado apresentava opções como:  
-Exibir versão do kernel    
-Mostrar status do sistema  
-Executar comandos de rede  

Ao analisar o binário com:  
```bash
strings /usr/bin/menu
```
Percebi que ele executava comandos com o curl sem utilizar o caminho absoluto, o que o tornava vulnerável a PATH Hijacking.

<img width="678" height="409" alt="strings" src="https://github.com/user-attachments/assets/c3731dfd-d72c-4a49-aece-210c6dd968a1" />


Na pasta /tmp, criei um binário falso chamado curl:
```bash
echo /bin/sh > curl
chmod +x curl
```
Em seguida, alterei a variável de ambiente PATH para priorizar o diretório /tmp:
```bash
export PATH=/tmp:$PATH
```
Com isso, ao executar novamente o binário SUID,(/usr/bin/menu), e ao selecionar a opção "status", o programa executou o meu falso curl, que na verdade iniciava um shell /bin/sh. Como o binário menu era SUID e pertencia ao root, o shell foi aberto com privilégios de root.  

Já com acesso root, naveguei até o diretório /root e obtive a segunda e última flag "root.txt", finalizando a máquina com sucesso.

## Conclusão
A máquina Kenobi reforça a importância de uma enumeração cuidadosa e da análise conjunta dos serviços expostos. Ao longo do desafio, foi possível observar como falhas aparentemente simples, quando combinadas, podem levar à completa comprometimento do sistema. O acesso inicial foi obtido por meio de um compartilhamento NFS mal configurado, que expôs uma chave privada SSH sem senha, e posteriormente confirmado com a descoberta do usuário correto através de um compartilhamento SMB acessível anonimamente.

Após o acesso ao sistema, a etapa de escalação de privilégios evidenciou um erro comum, porém crítico, no desenvolvimento de binários privilegiados. A presença de um binário SUID que executava comandos sem utilizar caminhos absolutos permitiu a exploração da variável de ambiente PATH, resultando na execução de um shell com privilégios de root. Esse tipo de falha demonstra como pequenas decisões inseguras na implementação de softwares podem gerar impactos severos na segurança de um sistema.
