# Máquina: StartUp (Try Hack Me)

## Enumeração 
Primeiramente, realizei um scan na máquina alvo utilizando Nmap:
```bash
nmap -sC -sV <targetIp>
```

<img width="827" height="533" alt="nmap star" src="https://github.com/user-attachments/assets/a6dd2ccb-c36d-4ed8-a104-2ef694b9d5ef" />  

O nmap nos revelou 3 portas abertas:  
22 - SSH
21 - FTP
80 - HTTP  

O nmap também nos mostrou que o serviço ftp permite autenticação anônima, com os arquivos "important.jpg", "notice.txt" e o diretório "ftp" que me permitia adicionar outros arquivos. Entrei no serviço e baixei os arquivos na minha máquina, mas não  havia nada relevante neles.
