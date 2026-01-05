# Máquina: Fawn (Hack the box)

## Enumeração
Já com o endereço ip da máquina alvo disponível era possível fazer a enumeração dos serviços em execução.  
Utilizei o nmap para o scan das portas mais comuns com os comandos: -sV -sC (-sV identifica a versão dos serviços encontrados, -sC roda scripts padrão nos serviços disponíveis).  
```bash
nmap -sC -sV <TARGET IP>
```

Ao finalizar o scan, foi identificada a porta 21 aberta com um serviço ftp em execução. Com o script padrão, o nmap identificou o login "anonymous" permitido no serviço.  
<img width="676" height="310" alt="FAWN" src="https://github.com/user-attachments/assets/3bfafd6f-d6d3-4d14-b405-5a03729189f3" />




## Exploração
Foi realizada a tentativa de login no serviço ftp utilizando usuário e senha como "anonymous" o que nos permitiu a entrada ao serviço.  
Já conectado, rodei o comando "ls" para buscar os arquivos diponíveis e identifiquei o arquivo "flag.txt", rodei o comando "get" para baixar o arquivo na minha máquina e visualizar, concluindo essa máquina com sucesso.
<img width="640" height="320" alt="FAWN1" src="https://github.com/user-attachments/assets/410421fd-9dd2-4f94-88a6-2b48a0702298" />


## Conclusão
A máquina Fawn explora uma configuração insegura de FTP com acesso anônimo habilitado, permitindo que qualquer usuário conecte-se ao serviço e faça download de arquivos sensíveis sem autenticação adequada.




