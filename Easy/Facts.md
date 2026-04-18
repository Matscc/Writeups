# Máquina: Facts(Hack The Box)

## Enumeração
No primeiro momento, fiz uma varredura no target utilizando o nmap para checar portas abertas e serviços em execução:  
  
<img width="675" height="95" alt="nmap" src="https://github.com/user-attachments/assets/24019bdc-ab2e-4581-8b7f-3313ce89402e" />
  
Encontramos a existência de um serviço web, que há uma provável simulação de aws sendo servido na porta 54321.  
Acessando a aplicação web, decidi enumerar o ditetórios existentes com o gobuster, descobrindo uma página de login "admin".
  
<img width="751" height="254" alt="gobuster" src="https://github.com/user-attachments/assets/2cde337b-6fcb-4cbb-889b-f3cf3109808a" />
  
Acessando a página de login, ele nos da a opção de criar uma nova conta, como não temos nenhuma credencial, criei um login com as credenciais: joao:joao123.  
Logando, o painel nos revelou que o serviço web é gerenciado com o camaleon CMS versão 2.9.0.
  
<img width="1440" height="615" alt="versão camaleon" src="https://github.com/user-attachments/assets/939d45df-7cb5-49f8-9a15-8a17af5626e9" />  
  
Essa versão do cameleon CMS vulnerável a CVE-2025-2304, que nos permite escalar privilégio no CMS através de um mass assignment. Essa vulnerabilidade se da porque a função "Updated_ajax" contém um erro crítico de segurança, permitindo que um user comum atualize seu dados de perfil sem filtrar os campos que podem ser alterados, utilizando o método "permit!".  
  
```bash
@user.update(params[:password].permit!)
```

## Exploração

Sabendo disso, eu abri o Burpsuit para interceptar as requisições com o objetivo de quando eu trocar a senha do meu user, alterar a requisição adicionando o campo password[role]=admin, fazendo eu conseguir acesso admin no CMS.
  
<img width="550" height="465" alt="admin cms" src="https://github.com/user-attachments/assets/b65fd426-c80f-4f00-92f7-2b331887f2da" />  
(password%5Brole%5D=admin) url encode.  
  
Após conseguir elevar o privilégio do meu user, consegui ter acesso a informação exposta no systemfile do CMS, conseguindo extrair as chaves para conexão no aws S3 que esta sendo simulado na porta 54321.
  
<img width="424" height="650" alt="s3 keys" src="https://github.com/user-attachments/assets/22300744-80c5-4056-a6fc-cb3a2b223a90" />
  
Investigando o serviço, encontrei uma private key que me ajudaria a logar no serviço ssh diponível na máquina.
  
<img width="888" height="150" alt="aws" src="https://github.com/user-attachments/assets/0cf47be9-a4f8-4548-aa89-8930d1d72138" />  
  
Essa private key exigia uma senha para conexão,então utilizei o ssh2john para transformar em hash legível para o john crackear, crackeando a senha e conseguindo acesso a máquina alvo via ssh.  
  
<img width="1440" height="370" alt="john" src="https://github.com/user-attachments/assets/1ba9311a-987b-40f9-b750-a3884244f6cf" />  

  
  
<img width="584" height="507" alt="ssh" src="https://github.com/user-attachments/assets/170e1b6d-a5fd-483c-9aac-31d6b7f75a48" />  
  
Encontrei a User flag no diretório do usuário Willian:  
<img width="386" height="130" alt="userflag" src="https://github.com/user-attachments/assets/7393c499-e90c-4fe6-8de1-2e569832e717" />  
  
## Escalação de privilégio

Após descobrir a user flag, dei um "sudo -l" para encontrar os binários que o usuário podia executar como root e encontrei o bin facter. 
  
<img width="610" height="97" alt="sudo -l" src="https://github.com/user-attachments/assets/0858fe4d-9d81-4a9b-9042-36efee0b1dc1" />  
  
Pesquisando mais sobre binário, descobri que ele serve de automação para listar configurações do sistema, podemos costomizar um diretório com um arquivo malicioso em ruby e direcionar para o binário executar como root. Então, criei o arquivo exploit.rb com um script que da spawn em um bash que quando for executado pelo facter, me dará um bash root.  
  
```bash
echo 'exec "/bin/bash"' > exploit.rb
```
  
<img width="577" height="109" alt="root e rootflag" src="https://github.com/user-attachments/assets/d046a29b-67f0-4d3b-8ec0-7b9b9651a578" />
  
Conseguindo um bash root, extraí a root flag, finalizando a máquina.








