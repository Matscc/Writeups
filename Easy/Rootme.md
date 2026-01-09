# Máquina: Rootme(Try Hack Me)

## Enumeração
Inicialmente realizei uma varredura para identificar os serviços disponíveis na máquina alvo.
```bash
nmap -sV <target ip>
```
sV- Identifica a versão dos serviços encontrados.

<img width="624" height="218" alt="nmap r" src="https://github.com/user-attachments/assets/7a4fb4d8-6d03-463c-b700-54074f0515bf" />  

O nmap identificou duas portas abertas:   
22- Rodando o serviço SSH.    
80- Rodando o serviço HTTP(aplicação web hospedada em servidor apache).  

Só era possível investigar o que havia na aplicação web da máquina alvo, então acessei a página online. Nela aparentemente não havia nada que eu podia explorar, decidi procurar diretórios escondidos na aplicação utilizando o gobuster.  
```bash
gobuster dir -u <http://targetip> -w <CaminhoDaWordlist>
```
<img width="625" height="291" alt="gobuster" src="https://github.com/user-attachments/assets/71ca7682-3699-46a1-b67e-f3056d2b4c3d" />

O gobuster revelou um diretório interessante para nós, que podia ser explorado, o "/panel/". Lá era possível fazer uploads de arquivos na aplicação, poderia ser uma possível vulnerabilidade RCE que nos daria acesso a um web shell.  

<img width="1280" height="718" alt="uploads" src="https://github.com/user-attachments/assets/a51855f4-4034-46c3-be94-2b75fb23c48c" />


## Exploração
Baixei um reverse shell em php no github e abri um listener na porta 8000 na minha máquina, utilizando o netcat:  
```bash
nc -lvnp 8000
````
Quando fui tentar fazer o upload do arquivo, a aplicação não permitia arquivos em php, mas poderia ser que o filtro da página só estava barrando o nome de uma única extensão php (.php), então, renomeei o arquivo com outra extensão php menos utilizada que pode ser esquecidas na blacklists (.phtml).Fiz o upload do arquivo, ele foi aceito e executado, assim consegui acesso ao web shell. 

<img width="623" height="107" alt="web" src="https://github.com/user-attachments/assets/d29d716a-000f-48d0-9126-7576f982e3b1" />

Já tendo uma shell do sistema, fui em busca da primeira flag do ctf "user.txt", então rodei o comando find para encontrar a flag: 
```bash
find / -type f -name "user.txt" 2< /dev/null
````
O que nos revelou que a flag estava em /var/www/user.txt, dando um cat neste caminho consegui a primeira flag do ctf.

## Escalação de privilégios
Já com o "user.txt" obtido, precisava da segunda flag "root.txt" que possivelmente estava no diretório root, para isso era necessário conseguirmos acesso ao shell como root.  
Procurei por binários que tinham o bit SUID(permissão que faz com que o arquivo seja executado com o UID do dono dele, não do usuário que xecutou), o que nos permitira explorar alguma vulnerabilidade:
````bash
find / -perm /4000 2> /dev/null
````
Analisando os arquivos com essa permissão, encontrei o "python", nos deixando em uma boa situação para conseguir root direto se o dono do arquivo "python" for root. 

<img width="433" height="291" alt="pyro" src="https://github.com/user-attachments/assets/975b8d90-c92c-4526-9085-d0227a012dbf" />  

Visitando o GTFObins, consegui um script em python que nos dava um shell root no sistema:
````bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
````
Depois de rodar o script, rodei o whoami para checar qual usuário eu consegui me tornar, confirmando que consegui o root do sistema.
já sendo root, entrei no diretório root e lá estava a segunda e última flag "root.txt", assim, finalizando essa máquina com sucesso.

## Conclusão

A máquina RootMe apresentou um cenário clássico e muito realista de exploração web seguida de escalada de privilégios em um sistema Linux. A exploração iniciou-se a partir de uma funcionalidade de upload de arquivos mal configurada, onde foi possível contornar a validação de extensões por meio do uso de uma extensão alternativa interpretada pelo PHP (.phtml). Esse bypass permitiu a execução remota de comandos, resultando em um web shell inicial.

Durante essa etapa, foram analisadas permissões, usuários e binários com privilégios elevados, destacando a importância da enumeração manual e do entendimento do funcionamento do sistema operacional.

A escalada de privilégios foi alcançada por meio da identificação de um binário com permissões indevidas, demonstrando como configurações incorretas de SUID podem levar à execução de comandos com privilégios elevados. Ao compreender o comportamento dessas permissões, foi possível obter acesso ao usuário root e concluir a máquina com sucesso.

Essa máquina reforça conceitos fundamentais de segurança ofensiva, como bypass de validações no upload de arquivos, entendimento do interpretador da aplicação web, gestão de permissões em sistemas Linux e importância da enumeração para privesc.


