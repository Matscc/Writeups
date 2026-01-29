# Máquina : LazyAdmin(Try Hack Me)

## Enumeração 
Primeiramente, fiz uma varredura no ip alvo para identificar os serviços ativos:
```bash
nmap -sC -sV <TargetIp>
```
 <img width="834" height="281" alt="nmap lazy" src="https://github.com/user-attachments/assets/ae8a94e7-10da-4316-b0d4-78e411c584b7" />


O nmap revelou dois serviços disponíveis:  
22 - SSH  
80 - HTTP (Web server Apache)  

Com essa informação, comecei fazer a enumeração da aplicação web. Na página inicial não havia nada que podia ser explorado, então utilizei o gobuster para enumerar os diretórios ocultos:  
```bash
gobuster dir -u <targetip> -w <CaminhoWordlist>
```
<img width="684" height="166" alt="gobuster 1Lazy" src="https://github.com/user-attachments/assets/4ec6c10a-1c59-4039-a5df-16bb3e41b10b" />


O gobuster nos mostrou a existência do diretório "content". Nesse diretório, havia apenas a informação que o site estava em manuntenção e conseguimos a informação que ele foi feito utilizando o "sweetrice".  
Com nada a ser explorado, listei novamente os diretórios ocultos com o gobuster.  

<img width="774" height="262" alt="gobuster 2Lazy" src="https://github.com/user-attachments/assets/1bb42554-d05c-41f5-8d47-1bf0f59c6328" />


Com a enumeração descobrimos o diretório "as", que é a página de login do sweetrice e encontramos o direório "inc".

## Exploração
No diretório "inc", encontramos o backup do banco de dados do sistema, isso poderia ser a forma de descobrir as credenciais de login. 

 <img width="632" height="797" alt="mysql Lazy" src="https://github.com/user-attachments/assets/34ac17da-7f74-4da5-a9e1-88dd6ca67859" />


Explorando o backup, encontrei uma senha encriptada no formato md5 e o usuário "manager":  

 <img width="1920" height="344" alt="password hash Lazy" src="https://github.com/user-attachments/assets/554a7302-98c9-42e5-9bd5-6ee296ad47fd" />


Desencriptei a senha(utilizando o Md5decrypt), e realizei o login com as credenciais obtidas.  

  <img width="907" height="809" alt="login Lazy" src="https://github.com/user-attachments/assets/77ca71be-76e0-4fc0-bf08-4c96dbfb8604" />


Logado como "manager" no sweetrice, consegui fazer um upload de um reverse shell em php na sessão "Data import".  

  <img width="1372" height="772" alt="data reverse lazy" src="https://github.com/user-attachments/assets/5fdb73a2-50f5-4b78-aaf4-af74ac9c204a" />



Abri um listener com o netcat na minha máquina na porta "8000" e executei o reverse shell na aplicação web, conseguindo acesso ao shell da máquina alvo. Achei a primeira flag do ctf "user.txt" no diretório de usuário "itguy".  

## Escalação de privilégios
Executei "sudo -l" para encontrar algum binário que o usuário poderia rodar como sudo:  

  <img width="958" height="111" alt="sudo -l lazy" src="https://github.com/user-attachments/assets/7814d820-25e4-421c-bb85-a3c73210aee1" />


Descobri que eu poderia executar comandos dentro do arquivo backup.pl (cujo o dono era o root) utilizando o binário perl, mas eu não poderia modificar esse arquivo.  
Dando um cat no "backup.pl", decobri que ele executava um arquivo chamando "copy.sh" e eu tenho permissão de modificar este arquivo.  

<img width="295" height="77" alt="backup lazy" src="https://github.com/user-attachments/assets/48dc3764-1437-4017-9666-7c358fd7b116" />


Visualisando o comando que havia no "copy.sh", descobri que ele realizava uma conexão remota com outra máquina assim que executado. 

<img width="658" height="57" alt="copy sh lazy" src="https://github.com/user-attachments/assets/609f198e-cb3d-4bff-a7e4-0254440f2453" />


Então, abri um listener na minha máquina na porta "4444" e modifiquei o ip que estava no arquivo para o ip da minha máquina.

<img width="900" height="92" alt="echo lazy" src="https://github.com/user-attachments/assets/982c66c4-7160-4130-bc9d-bb654cb9ea5a" />


Executei o  "backup.pl" como sudo e consegui um shell como root.  
<img width="706" height="316" alt="root lazy" src="https://github.com/user-attachments/assets/da183878-2760-4144-a894-9032a559a159" />


Já como root, entrei no diretório "/root", encontrando a segunda e última flag do ctf "root.txt", finalizando a máquina com sucesso.
## Conclusão 

