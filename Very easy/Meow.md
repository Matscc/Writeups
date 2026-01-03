# Máquina: Meow (Hack The Box)

## 🔍 Enumeração
Com o endereço da máquina disponível, foi realizada a enumeração dos serviços em execução.

- Utilização do Nmap para identificação de portas e serviços (comando varredura simples: nmap -sV target ip)
- Identificação da porta 23/TCP aberta
- Serviço Telnet em execução
** Telnet é um protocolo de rede que permite o acesso remoto a outro computador, inseguro por transmitir dados sem criptografia.
  
A enumeração indicou a presença do serviço de acesso remoto configurado de forma insegura.
<img width="678" height="179" alt="Nmap scan meow" src="https://github.com/user-attachments/assets/51eae1f2-94ca-4599-839b-da7739297b7f" />

---

## 🧪 Exploração
Com a porta Telnet exposta, foi realizada uma tentativa de conexão remota.(comando: telnet target ip)

<img width="436" height="195" alt="telnet login" src="https://github.com/user-attachments/assets/74f3662d-89a7-4c4d-8057-a8257694cac8" />



- O serviço Telnet estava mal configurado
- Foi possível autenticar sem credenciais adequadas, somente utilizando um login genérico.
- A falha permitiu acesso direto ao sistema

Após o acesso, com o listamento de arquivos foi possível encontrar o arquivo "flag.txt" , finalizando assim o CTF.

<img width="378" height="84" alt="flag" src="https://github.com/user-attachments/assets/2352f783-77b6-478b-8704-104b654b09ed" />


---

## 🛠 Ferramentas Utilizadas
- Nmap
- Telnet

---

## 📚 Aprendizados
- Riscos do uso de Telnet sem configuração adequada
- Importância de restringir acessos remotos
- Necessidade de desabilitar serviços inseguros em produção
