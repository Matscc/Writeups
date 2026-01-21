# Máquina: Mr.Robot(Try Hack Me)

## Enumeração
Iniciei a enumeração realizando um scan de portas com o Nmap para identificar os serviços disponíveis na máquina alvo:
```bash
nmap -sV <target_ip>
```
<img width="562" height="141" alt="nmap robot" src="https://github.com/user-attachments/assets/eb834e34-45c3-45c1-a9d2-77012f8eeaf4" />

A varredura nos revelou que há 3 portas abertas na máquina:  
22 - rodando SSH.  
80 - rodando HTTP.  
443 - HTTPS. 

Já com as portas enumeradas com o Nmap, fui enumerar a aplicação web disponível na porta 80.  
Na aplicação web, inicialmente foi me apresentado um menu, com opções que não me ofereceram nada de relevante, então decidi verificar os diretórios disponíveis na aplicação, utilizando o gobuster.  
```bash
gobuster dir -u <targetIp> -w <localWordlist>
```

<img width="415" height="798" alt="gobuster robot" src="https://github.com/user-attachments/assets/d7873891-66cb-47f9-afd2-a722a629f135" />  

O gobuster nos revelou alguns diretórios disponíveis, o que mais nos chama atenção é a existência do "robots.txt" e do painel de login do wordpress "wp-login".  

## Exploração
No diretório "Robots.txt", havia a primeira flag do CTF e também havia a uma wordlist, que eu poderia utilizar para enumerar usuários disponíveis no wordpress.  

*A versão do WordPress apresentada havia um erro de segurança, que nos informa quando somente a senha está incorreta, confirmando a existência do usuário.  
A wordlist que encontramos havia muitas palavras duplicadas, então precisamos limpar ela:  
```bash
sort wordlist.txt | uniq > wordlistLimpa.txt
````

Já com nossa wordlist unificada, podemos fazer o brute force no Wordpress, utilizando o Hydra:
```bash
hydra -L <wordlistLimpa.txt> -p "test" <targetIp> http-post-form "/login.php:log=^USER^&pwd=^PASS^:Invalid Username"
```
-O input do login e senha estavam definidos como log e pwd, então precisei informar para o hydra.  
-Invalid Username : mensagem de erro quando o usuário não existe, informamos para o hydra para ele saber quando o usuário não existe.  


<img width="763" height="136" alt="elliot robot" src="https://github.com/user-attachments/assets/83509584-1d67-4110-bf15-61a8afe71a86" />  

O hydra encontrou o usuário "Elliot", então só precisamos da senha:

```bash
hydra -l "Elliot" -P <wordlistLimpa.txt" <targetIp> http-post-form "/login.php:log=^USER^&pwd=^PASS^:The password you entered for the username"
```

<img width="825" height="146" alt="senha robot" src="https://github.com/user-attachments/assets/017eff36-b799-4293-94b2-e060bc467091" />  


Com o usuário e senha descobertos, conseguimos entrar no painel de admin do wordpress. A partir do acesso, explorei algo para conseguir uma reverse shell com algum upload em alguma sessão.  
Na sessão de appaerences, na aba "archives" percebi que era possível o upload de um script reverse shell em php:  

<img width="1681" height="772" alt="reverse mr robot" src="https://github.com/user-attachments/assets/ceaa6fec-90ac-4f2a-b9f6-dfd2385c763e" />  


Abri a porta "8000" na minha máquina com o netcat e acessei o caminho para rodar o reverse shell upado:wp-content/themes/twentyfifteen/archive.php. Assim, consegui uma shell na máquina alvo.  


<img width="351" height="124" alt="shell robot" src="https://github.com/user-attachments/assets/9a033c09-55a8-4f11-aa46-b729b5091417" />  


  

      
Já no sistema, explorei para encontrar a segunda flag, e encontrei o diretório do usuário "robot" mas eu não tinha permissão para ler o arquivo. No diretório robot, também havia uma senha em formato md5, então precisei descodificar a senha,(isso pode ser feito em páginas presentes na internet). já com a senha decodificada, alternei para o usuário "robot" e finalmente consegui ler a segunda flag do CTF.  

## Escalação de privilégios
Agora, estava em busca de algum binário que me permitia executar como root, para realizar essa busca, procurei binários com o bit SUID, para checar se havia algo explorável:
```bash
find / -perm /4000 2> /dev/null
```
Com a busca, foi revelado que o nmap tinha o bit SUID ativado, o que me permitia explorar.  

<img width="429" height="274" alt="nmapvuln robot" src="https://github.com/user-attachments/assets/10d66d2e-e4de-4187-9f95-827c3705d4d0" />


Acessei o GTFObins e procurei por alguma funcionalidade ou script nmap que eu poderia utilizar para obter um shell root, e descobri que o modo interativo do nmap me permite gerar um shell, como o nmap tem permissão SUID, ele rodará esse shell como root.  

<img width="1110" height="600" alt="gtfobins robot" src="https://github.com/user-attachments/assets/ec801913-a228-4d03-a0c2-4990b3061a94" />

```bash
nmap --interactive
!sh
```
Assim, consegui acesso root ao sistema, acessei o diretório "/root" e encontrei a terceita e última flag do CTF, finalizando essa máquina com sucesso.

## Conclusão

A máquina Mr. Robot apresenta uma cadeia de falhas de segurança que, quando combinadas, permitem o comprometimento completo do sistema. O primeiro problema crítico está na aplicação web, que expõe informações sensíveis e permite a enumeração de usuários por meio de ferramentas especializadas, como o Hydra. A utilização de credenciais fracas e a ausência de mecanismos eficazes de proteção contra enumeração e força bruta tornam o acesso inicial relativamente simples.

Outro ponto grave é a má configuração do ambiente WordPress, onde usuários administrativos podem ser descobertos e explorados sem restrições adequadas, evidenciando a falta de boas práticas como autenticação forte, limitação de tentativas de login e ocultação de informações sensíveis da aplicação.

Após o acesso ao sistema, a máquina demonstra falhas críticas de configuração no sistema operacional, especialmente relacionadas a permissões inadequadas. A presença de binários com o bit SUID configurado de forma insegura permite que usuários comuns executem comandos com privilégios elevados, o que resulta diretamente na escalação de privilégios até o nível root.

Esses problemas evidenciam uma ausência de princípios básicos de segurança, como o princípio do menor privilégio, a correta gestão de permissões e o hardening do sistema. Em um ambiente real, falhas desse tipo poderiam levar à perda total de controle do servidor, comprometimento de dados sensíveis e uso da infraestrutura para ataques a terceiros.

Portanto, a máquina Mr. Robot reforça como vulnerabilidades simples — quando não mitigadas — podem ser encadeadas para gerar um impacto crítico, destacando a importância de boas práticas tanto no desenvolvimento de aplicações web quanto na configuração segura do sistema operacional.





