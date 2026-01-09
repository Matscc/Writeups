# Máquina: Bounty Hacker(Try Hack Me)

## Enumeração
Inicialmente realizei uma varredura para identificar os serviços disponíveis na máquina alvo.  
````bash
nmap -sC -sV <target ip>
````
sC- Roda scripts padrão nos serviços disponíveis.  
sV- Identifica a versão dos serviços encontrados.

<img width="755" height="526" alt="varredura" src="https://github.com/user-attachments/assets/9a099310-cedf-4f5e-b57e-b44185371761" />

O nmap identificou 3 portas abertas:    
21- Rodando o serviço FTP. (nmap já identificou login "anonymous" possível).  
22- Rodando o serviço SSH.    
80- Rodando o serviço HTTP(Há uma aplicação web em execução).    

Primeiramente entrei na aplicação web da máquina, onde só havia uma imagem, mas aparentemente não havia nada que podia ser explorado, nenhum diretório com informações expostas e nada importante no código HTML. Como eu não tenho as credenciais da conexão SSH, tudo o que restou para explorar foi o serviço ftp que permitia conexão com login "anonymous".  

## Exploração
Loguei no serviço FTP e encontrei dois arquivos ".txt", os arquivos "locks.txt" e "tasks.txt". Baixei os arquivos na minha máquina e visualizei.  
No aquivo "locks.txt" havia uma wordlist que possivelmente será utilizada para realizar um brute force no serviço SSH:  

<img width="189" height="432" alt="locks" src="https://github.com/user-attachments/assets/38dcb755-e828-4bdf-acfa-e86f3869d7ac" />  

No arquivo "tasks.txt" havia uma lista de afazeres escrita por alguém chamada "lin",(possivelmente o user do serviço ssh).  

<img width="391" height="86" alt="tasks" src="https://github.com/user-attachments/assets/7232b62f-16ef-4c86-9a54-65c70dbbfe1a" />  

Já com as possíveis credenciais em mãos, fui realizar o bruteforce no serviço ssh da máquina alvo utilizando a ferramenta hydra.  
````bash
hydra -l lin -P locks.txt <target ip> ssh
````
-l = Utilizado para identificar o user único possível.  
-P = Utilizado para identificar o caminho da wordlist.  

O hydra consiguiu indentificar a senha que permite o login com o user "lin', o que permitiu minha autenticação no serviço ssh.
<img width="732" height="119" alt="hydra" src="https://github.com/user-attachments/assets/793b3a3b-0ba3-4249-899a-0b3ec3689ca3" />  

-consegui a primeira flag "user.txt'.

## Escalação de privilégios

O usuário "lin", era um suário sudo, então fui checar suas permissões como sudo, para analisar como eu conseguiria me tornar root no sistema.  
````bash
sudo -l
````

Descobri que o usuário tem a permissão de executar o binário "tar" como root.  

<img width="965" height="111" alt="perm" src="https://github.com/user-attachments/assets/692405b0-9b71-4444-a752-a60b69e61cb2" />

-O tar é um programa de de compactação de extração de arquivos, mas quando tem a permissão de rodar como root ele pode ser abusado para executar comandos. O tar tem a função  "checpoint-action" que permite executar comandos, o que podemos eplorar para alcançar o root.  
Acessando o GTFObins encontrei um script tar que utiliza a função checkpoint para nos disponibilizar um shell como root:  
````bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
````
Executando este script na máquina alvo, consegui acesso root, explorei a máquina e encontrei a segunda e última flag "root.txt" no diretório "root", finalizando essa máquina com sucesso.  

## Conclusão
A máquina Bounty Hacker apresenta um fluxo clássico de exploração, combinando enumeração inicial de serviços, reutilização de credenciais e escalonamento de privilégios por meio de uma má configuração de permissões sudo. A enumeração revelou serviços acessíveis que permitiram a obtenção de credenciais válidas, possibilitando o acesso ao sistema como um usuário comum.

Após o acesso inicial, a verificação das permissões sudo revelou que o usuário possuía autorização para executar o binário tar como root sem necessidade de senha. Essa configuração insegura permitiu o abuso da funcionalidade checkpoint-action do tar, resultando na execução de comandos com privilégios elevados e, consequentemente, na obtenção de um shell como root.

Essa máquina reforça a importância da enumeração local e da análise cuidadosa de permissões sudo, demonstrando como binários aparentemente inofensivos podem ser explorados para escalonamento de privilégios quando mal configurados. Além disso, destaca a relevância de conhecer ferramentas como o GTFOBins para identificar rapidamente vetores de exploração em ambientes Linux.
