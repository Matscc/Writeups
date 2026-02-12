# Máquina: Simple CTF(Try Hack Me)

## Enumeração
Iniciei com a enumeração simples de serviços ativos na máquina alvo utilizando o nmap:
```bash
nmap -sS <targetIp>
```

<img width="547" height="138" alt="Nmap simple" src="https://github.com/user-attachments/assets/42f0c9a6-ed8d-4388-a73e-9b1e9d6b2670" />  

O nmap revelou 3 serviços ativos:  
21 - ftp  
80 - http  
2222- (Com um melhor scan do nmap, é revelado que o serviço real é o ssh rodando em uma porta não convencional)  

Comecei acessando o serviço ftp que permitia acesso anônimo e tudo que encontrei foi uma mensagem para um dev chamado Mitch:  

<img width="1265" height="109" alt="Ftp simple" src="https://github.com/user-attachments/assets/decb52bd-04bd-404b-b18a-31474ba51590" />  

Após isso, acessei o serviço http, que na página inicial não parecia ter nada relevante, então, utilizei o gobuster para encontrar diretórios ocultos na aplicação.  

<img width="672" height="211" alt="gobuster simple" src="https://github.com/user-attachments/assets/12fa4d4b-fd77-4f5b-9e6b-8c56ccf0a8ed" />  

O gobuster nos revelou o diretório "simple", que quando acessado foi revelado que a aplicação web era gerênciado pelo sistema "CMS made simple" na versão 2.2.8"

<img width="1280" height="720" alt="CMS simple" src="https://github.com/user-attachments/assets/108efa37-4b24-48d3-80cd-b0e8a00be062" /> 

## Exploração

Sabendo disso, utilizei o metasploit para verificar se havia algum exploit presente nessa versâo do CMS, e encontrei um exploit que explora uma vulnerabilidade de sql injetion nas versões inferiores a 2.2.10, permitindo a extração de informações no serviço web.

<img width="1280" height="284" alt="exploit simple" src="https://github.com/user-attachments/assets/f8674be7-e369-49bf-89a9-8c4241354357" />  

Executando o exploit:  

  
````bash
python2 46635.py -u <url> -c <crackOpcionalDeSenha> -w <CaminhoWordlist>
````

  

  
<img width="460" height="91" alt="py simple" src="https://github.com/user-attachments/assets/2fb7329d-c496-47f1-91da-4fdd53d697e3" />  
Ao finalizar, descobrimos a senha "secret" e o usuário "mitch". Com essas informações, já pocemos nos conectar no serviço ssh na porta 2222.  
```bash
ssh mitch@<targetip> -p 2222
```
Realizando a conexão com exito, encontramos a primeira flag "user.txt".

## Escalação de privilégios

Agora com acesso a máquina alvo, preciso do acesso root ao sistema para conseguir a segunda flag. Rodo o comando "sudo -l" para verificar se há algum arquivo que possamos utilizar como sudo para conseguir esse acesso.

<img width="474" height="82" alt="sudo - l simple" src="https://github.com/user-attachments/assets/d52e12c3-153b-40be-a155-7c0bad76247c" />  

Para a minha surpresa, nos era permitido executar o vim como root sendo sudo, o que já nos garante um root direto. Entrei no GTFObins e encontrei um comando que nos daria um shell root explorando essa má configuração do vim.  
```bash
vim -c ":!/bin/sh"
```
<img width="489" height="247" alt="root simple" src="https://github.com/user-attachments/assets/67eb976c-fd54-4297-bc58-3e9a90d6216a" />  

Após a execução, conseguimos o shell root e encontramos a segunda e última flag do CTF, finalizando a máquina com sucesso.

## Conclusão
A máquina Simple CMS demonstrou, de forma prática, como uma aplicação web mal configurada e desatualizada pode comprometer completamente a segurança de um servidor. A exploração iniciou-se com uma enumeração básica de serviços, onde foi identificada uma aplicação baseada no CMS Made Simple, versão vulnerável à CVE-2019-9053.

Essa vulnerabilidade permitiu a extração de hashes de credenciais diretamente da aplicação web, sem a necessidade de autenticação prévia, evidenciando uma falha crítica de segurança no tratamento de entradas SQL. A partir da obtenção do hash, foi possível realizar o processo de quebra de senha, garantindo acesso legítimo ao sistema via SSH.

Com acesso ao sistema, a etapa seguinte consistiu na enumeração local em busca de vetores para escalonamento de privilégios. A análise revelou permissões indevidas associadas ao binário vim, permitindo sua execução com privilégios elevados. Explorando essa configuração incorreta, foi possível obter acesso root completo, comprometendo totalmente o sistema.







