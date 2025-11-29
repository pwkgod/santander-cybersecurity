<h1>
  <img src="imagens/santander-bootcamp-logo.png" width="50" style="vertical-align: middle;">
  Santander - Cibersegurança 2025
</h1>
Repositório destinado ao desafio do curso Santander - Cibersegurança 2025.

Este repositório documenta toda a construção de um laboratório e execução dos ataques simulados, utilizando Kali Linux, Metasploitable 2, DVWA, Medusa, Burp Suite e Nmap. O foco do projeto é entender na prática como funcionam ataques de brute-force.

🎯 **Objetivo do Desafio**

O desafio consiste em montar um ambiente controlado e reproduzir ataques comuns solicitados no desafio do curso:  
● Brute-force em **FTP**  
● Brute-force em formulário web (**DVWA**)  
● Password spraying em **SMB**  

🧩 **Cenário do Laboratório**

Todo o ambiente foi montado localmente usando VirtualBox, com rede Host-only para isolar as VMs da internet:  
● VM 1 (Atacante): Kali Linux  
● VM 2 (Alvo): Metasploitable 2  
Rede usada: Host-only Network

📌 **Download do Metasploitable 2**

O Metasploitable 2 foi baixado diretamente do SourceForge, o repositório oficial do projeto. A ISO da máquina contém vários serviços vulneráveis, ideais para laboratórios de pentest.

<img src="imagens/1-download-metasploitable.png" width="500">

📌 **Criação da VM no VirtualBox**

Após o download, foi feita a criação da VM:  
● Importação da ISO do Metasploitable  
● Configuração da rede como Host-only Adapter para ambas VMs se comunicarem  
● Inicialização da máquina Metasploitable para permitir que os serviços subissem

<img src="imagens/2-start-metasploitable.png" width="500">

📌 **Login padrão**

Para acessar o sistema usamos as credenciais:
**msfadmin** / **msfadmin**

<img src="imagens/3-config-metasploitable.png" width="500">

🌐 **Verificação de Comunicação entre VMs**

Antes de realizarmos os ataques, validamos se as máquinas realmente se comunicam. No Kali, verifiquei o IP das VMs com:

`ifconfig`  

E testei a comunicação entre elas com:  

`ping 192.168.1.8`

<img src="imagens/4-ping-metasploitable.png" width="450">

🛰️ **Enumeração com Nmap**

Foi feito a enumeração das portas da máquina Metasploitable com o scanner Nmap:  

`nmap -sV -v -p- 192.168.1.7`  

**Flags utilizadas:**  

**-sV** → descobre serviços e versões  
**-v** → modo verbose (scan detalhado)  
**-p-** → scan em todas as portas  

<img src="imagens/5-nmap-metasploitable.png" width="500">

**Resultado:** Descobrimos que a porta 21 (FTP) está aberta e vulnerável, além de smb, http e outros serviços inseguros que podem ser explorados.

🔐 **Ataque 1: Brute-force em FTP com Medusa**

Para este ataque, realizamos a criação manual de duas wordlists básicas com os comandos:  

`echo "admin\nmsfadmin\nroot\nteste\nftp" > usuarios_comuns.txt`  
`echo "12345\nmsfadmin\nteste\n123456789\nroot\nadmin\nftp" > senhas_comuns.txt`  

<img src="imagens/6-criando-wordlist.png" width="550">

**Ataque com Medusa:**

`medusa -h 192.168.1.7 -U usuarios_comuns.txt -P senhas_comuns.txt -M ftp -t 5`  

**Flags utilizadas:**

**-h** → nosso alvo  
**-U** → arquivo com lista de usuários  
**-P** → arquivo com lista de senhas  
**-M ftp** → módulo do protocolo FTP  
**-t 5** → cinco threads simultâneas para o ataque ser mais rápido  

<img src="imagens/7-brute-force-ftp.png" width="550">

Credencial encontrada:  

**msfadmin** : **msfadmin**  

Este foi um brute-force simples com a ferramenta Medusa, mas funciona porque o FTP explorado não possui proteção contra tentativas excessivas, além do serviço estar exposto.


🕸️ **Ataque 2: Brute-force em Formulário Web DVWA com Burp Suite**  

Para ataques em formulários de login, não adianta usar ferramentas como Medusa, pois precisamos manipular requisições HTTP. Para isso, o Burp Suite é perfeito, pois nos vai permitir por meio do proxy, a modificação das requisições entre nossa máquina Kali e o site do DVWA.
   
<img src="imagens/8-brute-force-dvwa.png" width="500">

Acessei o DVWA e tentei logar para interceptar a requisição GET de login, onde a requisição foi capturada no Burp:

<img src="imagens/9-burp-suite-request.png" width="500">

Com a requisição capturada, realizamos o envio para o Intruder, que vai nos possibilitar modifica-la:  

→ Botão direito → Send to Intruder

<img src="imagens/10-burp-suite-intruder.png" width="450">

No Intruder, configuramos o modo **Cluster Bomb Attack**, esse modo de ataque nos permite testar combinações diferentes de usuários e senhas (wordlist dupla):   

**posição 1** → username  
**posição 2** → password  
**payloads** → wordlists do SecLists

<img src="imagens/11-burp-suite-intruder-2.png" width="450">

Ao configurarmos o ataque, executamos e observamos na tela de requisições enviadas que todas as respostas retornam o status **200 OK**, mas uma tem tamanho diferente, indicando que houve uma mudança na página, redirecionamento interno ou um login bem-sucedido.

<img src="imagens/12-burp-suite-intruder-3.png" width="550">

Copiamos as credenciais da requisição diferente e realizamos login, confirmando que a credencial descoberta foi:   

**admin** : **password**

📁 **Ataque 3: Password Spraying em SMB com Medusa**

Diferente do brute-force, no password spraying usamos uma senha para vários usuários. Isso é útil quando tem chance de sermos bloqueados por várias tentativas de login ou o sistema usa senhas fracas ou com padrão.  

Utilizamos o seguinte comando para realizarmos este ataque:  

`medusa -h 192.168.1.7 -U /home/kali/Wordlists/Sec-Lists-master/Usernames/top-usernames-shortlist.txt -p "user" -M smbnt -f`

**Flags utilizadas:**

**-U** → lista de usuários  
**-p** → senha única (password spraying)  
**-M** smbnt → módulo para protocolo SMB  
**-f** → encerra o ataque ao encontrar uma credencial válida  

<img src="imagens/13-password-spraying-smb.png" width="550">

Credencial encontrada:  

**user** : **user**  

Neste ataque, o SMB da máquina é vulnerável de propósito, mas esse ataque é muito comum em empresas, especialmente quando usuários tem senhas fracas ou a empresa segue padrões de senha para serviços e contas.
