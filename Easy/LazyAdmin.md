# Máquina : LazyAdmin(Try Hack Me)

## Enumeração 
Primeiramente, fiz uma varredura no ip alvo para identificar os serviços ativos:
```bash
nmap -sC -sV <TargetIp>
```
(IMAGE NMAP)  

O nmap revelou dois serviços disponíveis:  
22 - SSH  
80 - HTTP (Web server Apache)  

Com essa informação, comecei fazer a enumeração da aplicação web. Na página inicial não havia nada que podia ser explorado, então utilizei o gobuster para enumerar os diretórios ocultos:  
```bash
gobuster dir -u <targetip> -w <CaminhoWordlist>
```
IMAGE(Gobuster1)  

O gobuster nos mostrou a existência do diretório "content". Nesse diretório, havia apenas a informação que o site estava em manuntenção e conseguimos a informação que ele foi feito utilizando o "sweetrice".  
Com nada a ser explorado, listei novamente os diretórios ocultos com o gobuster.  

IMAGE(Gobuster2)

Com a enumeração descobrimos o diretório "as", que é a página de login do sweetrice e encontramos o direório "inc".

## Exploração
No diretório "inc", encontramos o backup do banco de dados do sistema, isso poderia ser a forma de descobrir as credenciais de login. 

IMAGE(mysql)  

Explorando o backup, encontrei uma senha encriptada no formato md5 e o usuário "manager":  

IMAGE(banco)  

Desencriptei a senha(utilizando o Md5decrypt), e realizei o login com as credenciais obtidas.  

IMAGE(login)
