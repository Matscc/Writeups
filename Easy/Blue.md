# Máquina: Blue(Try Hack Me)

## Enumeração

Iniciei a enumeração realizando um scan de portas  com o Nmap para identificar os serviços disponíveis na máquina alvo:
```bash
nmap -sS <targetIp>
```

<img width="569" height="276" alt="nmap blue" src="https://github.com/user-attachments/assets/e7f7c214-e8ec-4e67-8bcc-6da9f66e1514" />  

O nmap revelou que havia um serviço SMB na porta 445. O serviço permitia acesso anônimo mas não havi nada explorável, então rodei um script no nmap para verificar a versão do SMB e se havia alguma vulnerabilidade:

```bash
nmap --script vuln -p 445 <targetIp>
```
<img width="856" height="362" alt="nmap smbv1 blue" src="https://github.com/user-attachments/assets/5356bc43-e5eb-4328-8df7-06c4b2cf0f8a" />  

O script nos revelou que a versão do SMB era a SMBv1 que possui a famosa vulnerabilidade ms17-010(Eternal blue), que nos permite executar um RCE conseguindo controle a máquina alvo.

## Exploração
Utilizei o Metasploit para procurar exploits para vulnerabilidade ms17-010:

<img width="1280" height="646" alt="exploit ms17-010 blue" src="https://github.com/user-attachments/assets/3976147c-f43c-4131-9b9b-23600b2afd21" />  

Irei utilizar o exploit "0". Agora preciso "settar" as informações necessárias para o exploit funcionar:

<img width="1280" height="631" alt="requerimentos blue" src="https://github.com/user-attachments/assets/0aa7a86f-3426-420e-818a-cfd4ddae8300" />

Foi necessário somente informar o ip da máquina alvo e da minha máquina para poder rodar o exploit. Também informei um payload que a tarefa do try hack me exigiu ao invés do default e rodei o exploit.  

<img width="1280" height="557" alt="payload blue" src="https://github.com/user-attachments/assets/bc9a2400-1ea7-4079-9a83-b6cdfbf0892e" />

Já com o acesso ao shell da máquina alvo, decidi buscar um shell melhor utilizando o meterpreter que já tem ferramentas úteis para nós:

<img width="1050" height="494" alt="shell to meterpeter blue" src="https://github.com/user-attachments/assets/34800dc5-1a1d-40b5-a3d0-f0da56411938" />

Com uma nova sessão meterpreter, fui verificar as sessões ativas para averiguar se eu tinha acesso System na sessão ativa do meterpreter.

<img width="1280" height="160" alt="sessions blue" src="https://github.com/user-attachments/assets/841dab5c-3af1-4af5-9952-17df8b7c5d3a" />

## Pós-exploração

Com essa confirmação, eu já tenho acesso total a máquina alvo, então só preciso encontrar as flags do CTF.  
A primeira flag se encontra na raíz do sistema:  

<img width="1280" height="317" alt="flag 1 blue" src="https://github.com/user-attachments/assets/67ce7e6c-a40d-4b54-9f73-227ced763899" />  

A segunda flag se encontra onde as senhas são armazenadas no sistema: 

<img width="750" height="291" alt="flag 2 blue" src="https://github.com/user-attachments/assets/76e18e56-8139-497d-a739-ac721a2233cd" />

A terceira e última flag está no usuário "Jon" no diretório "Documents":

<img width="574" height="164" alt="flag 3 blue" src="https://github.com/user-attachments/assets/b940940e-aa53-4c60-a864-e4ea3d210835" />  

Finalizando assim, mais uma máquina com sucesso.

## Conclusão

A máquina Blue demonstrou de forma prática o impacto crítico de serviços desatualizados e mal configurados em ambientes Windows. Através de uma enumeração inicial com Nmap, foi possível identificar a presença do SMB vulnerável, o que indicou imediatamente a possibilidade de exploração da falha MS17-010 (EternalBlue).

A exploração dessa vulnerabilidade permitiu a execução remota de código sem autenticação, resultando em acesso direto ao sistema com privilégios máximos (NT AUTHORITY\SYSTEM). Isso evidencia o quão devastadoras podem ser falhas em serviços de rede expostos, principalmente quando combinadas com protocolos legados, como o SMBv1.














