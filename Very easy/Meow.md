# Máquina: Meow (Hack The Box)

## 🔍 Enumeração
Com o endereço da máquina disponível, foi realizada a enumeração dos serviços em execução.

- Utilização do Nmap para identificação de portas e serviços (comando varredura simples: nmap -sV target ip)
- Identificação da porta 23/TCP aberta
- Serviço Telnet em execução
** Telnet é um protocolo de rede que permite o acesso remoto a outro computador, inseguro por transmitir dados sem criptografia.
  
A enumeração indicou a presença do serviço de acesso remoto configurado de forma insegura.

---

## 🧪 Exploração
Com a porta Telnet exposta, foi realizada uma tentativa de conexão remota.(comando: telnet target ip)

- O serviço Telnet estava mal configurado
- Foi possível autenticar sem credenciais adequadas, somente utilizando o login "root".
- A falha permitiu acesso direto ao sistema

Após o acesso, com o listamento de arquivos foi possível encontrar o arquivo "flag.txt" , finalizando assim o CTF.

---

## 🛠 Ferramentas Utilizadas
- Nmap
- Telnet

---

## 📚 Aprendizados
- Riscos do uso de Telnet sem configuração adequada
- Importância de restringir acessos remotos
- Necessidade de desabilitar serviços inseguros em produção
