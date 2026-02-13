# Máquina: Ignite(Try Hack Me)

## Enumeração
Utilizei o nmap para fazer um scan de serviços diponíveis rodando na máquina alvo:

```bash
nmap -sC -sV <targetip>
```

<img width="537" height="172" alt="nmap ignite" src="https://github.com/user-attachments/assets/42fb23b1-012f-4e64-bc39-a335cd15cbef" />  

O nmap revelou uma aplicação web ativa, então fui enumerar:

<img width="1920" height="852" alt="web ignite" src="https://github.com/user-attachments/assets/c6928b57-9cd7-48cf-beea-467a103c2cc6" />  

A página inicial já nos revelou que a aplicação é gerenciada pelo Fuel CMS na sua versão 1.4, sabendo disso, procurei se há alguma vulnerabilidade nessa versão e encontrei uma vulnerabilidade RCE disponível (CVE-2018-16763).

## Exploração

Encontrei um exploit para explorar essa vulnerabilidade e alterei para indicar a url alvo:
<img width="1504" height="574" alt="exploit ignite" src="https://github.com/user-attachments/assets/5de913ec-f755-430b-8b7b-d06babf8b5f0" />  

Já com acesso e podendo executar comandos na aplicação web, decidi abrir um server python em minha máquina para disponibilizar um arquivo que continha uma reverse shell que me disponibilizaria um shell na máquina alvo após executado.
<img width="386" height="116" alt="reverse ignite" src="https://github.com/user-attachments/assets/32033cf2-6fa0-4adb-8d6f-ebe66d87ff7d" />  

Após baixar esse arquivo na aplicação web, executei e consegui um shell na máquina alvo. Em seguida, encontrei a primeira flag no usuário web "www-data".  

<img width="319" height="48" alt="user ignite" src="https://github.com/user-attachments/assets/5bdafa3c-015c-4d05-a0d1-02dc923f73f5" />  

Para conseguir um shell melhor, decidi spawnar um em python com  o comando:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

## Escalação de privilégios
Agora eu precisava de acesso root ao sistema. Lembrei que na página inicial do fuel CMS era informado onde era o caminho para acessar a database da aplicação, o que poderia conter informações para podermos explorar.

<img width="1413" height="854" alt="database ignite" src="https://github.com/user-attachments/assets/94eec7be-467f-40d2-a02e-db1298e515db" />  

Acessando a database, podemos encontrar a senha do usuário root.

<img width="640" height="501" alt="senha ignite" src="https://github.com/user-attachments/assets/e560bc95-92e9-4c6a-b6d6-f74edfa17c1b" />  

Finalmente com a senha, consegui acesso root ao sistema.  

<img width="518" height="88" alt="root ignite" src="https://github.com/user-attachments/assets/504c3225-541c-4de4-9be7-6be5c8c0d46c" />  

Com acesso root, encontrei a segunda e última flag do ctf no diretório "root", finalizando a máquina com sucesso.

## Conclusão

A máquina Ignite foi fundamental para reforçar conceitos essenciais de enumeração web, exploração de CMS e escalada de privilégios. A identificação de arquivos sensíveis e configurações inseguras permitiu a obtenção de acesso inicial, demonstrando como falhas simples podem comprometer uma aplicação.

Após o acesso, a enumeração local evidenciou más configurações que possibilitaram a escalada de privilégios até root, destacando a importância de permissões bem definidas e da correta proteção de arquivos críticos. Essa máquina contribuiu significativamente para o desenvolvimento do raciocínio lógico e da metodologia prática em testes de invasão.


