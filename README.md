# 🛡️ Análise de Segurança: Ataques de Força Bruta e Password Spraying com Medusa e Kali Linux

Este projeto é uma entrega prática e documental para o programa **Santander - Cibersegurança do Zero à Prática**, realizado na [DIO (Digital Innovation One)](https://dio.me).

O foco deste trabalho é a análise detalhada de técnicas de auditoria de senhas e autenticação (Força Bruta, Ataque de Dicionário e Password Spraying) utilizando a ferramenta **Medusa** no **Kali Linux**, acompanhado de um plano aprofundado de **mitigação e defesa cibernética (Blue Team)**.

> ℹ️ **Nota sobre o Ambiente:** Devido a restrições de virtualização de hardware, este projeto foi desenvolvido como um **Estudo de Caso e Relatório Técnico Aprofundado**, com análise de sintaxe de comandos, comportamento de protocolos de rede e estratégias de prevenção, conforme a diretriz de flexibilidade proposta no enunciado do desafio.

---

## 📑 Sumário

- [1. Fundamentação Teórica](#1-fundamentação-teórica)
  - [Tipos de Ataques de Autenticação](#tipos-de-ataques-de-autenticação)
  - [A Ferramenta Medusa](#a-ferramenta-medusa)
- [2. Cenários de Simulação e Análise Técnica](#2-cenários-de-simulação-e-análise-técnica)
  - [Cenário 1: Autenticação FTP](#cenário-1-autenticação-ftp)
  - [Cenário 2: Formulários Web (DVWA)](#cenário-2-formulários-web-dvwa)
  - [Cenário 3: Protocolo SMB e Password Spraying](#cenário-3-protocolo-smb-e-password-spraying)
- [3. Matriz de Mitigação e Defesa (Blue Team)](#3-matriz-de-mitigação-e-defesa-blue-team)
- [4. Conclusão e Aprendizados](#4-conclusão-e-aprendizados)
- [5. Referências e Recursos](#5-referências-e-recursos)

---

## 1. Fundamentação Teórica

### Tipos de Ataques de Autenticação

1. **Força Bruta Exaustiva (*Pure Brute Force*):**
   - Testa todas as combinações possíveis de caracteres de forma sequencial (ex.: `a`, `b`, ..., `aa`, `ab`).
   - Apresenta complexidade computacional exponencial; pode levar anos ou séculos dependendo do tamanho da senha.

2. **Ataque de Dicionário (*Dictionary Attack*):**
   - Utiliza listas de palavras pré-compiladas (*wordlists*, como a `rockyou.txt`) contendo senhas vazadas ou comumente utilizadas por humanos.
   - Muito mais eficiente e rápido que a força bruta pura.

3. **Pulverização de Senhas (*Password Spraying*):**
   - Testa **uma única senha comum** (ex.: `Empresa@2024`) contra uma **ampla lista de usuários diferentes**.
   - **Furtividade:** Não dispara políticas tradicionais de bloqueio de conta por tentativas consecutivas no mesmo usuário, sendo um dos métodos mais perigosos em redes corporativas (Active Directory).

### A Ferramenta Medusa

O **Medusa** é uma ferramenta de auditoria de força bruta modular, rápida e concorrente via rede.

* **Arquitetura Modular:** Trabalha por meio de módulos específicos para cada protocolo (`.mod`), como `ftp`, `http`, `smbnt`, `ssh`, etc.
* **Processamento Paralelo:** Suporta concorrência baseada em *threads*, permitindo testar múltiplos hosts, usuários e senhas simultaneamente.
* **Uso Ético:** Ferramenta voltada para profissionais de segurança avaliarem a robustez das credenciais e a eficácia dos mecanismos de defesa de uma organização.

---

## 2. Cenários de Simulação e Análise Técnica

### Cenário 1: Autenticação FTP

O protocolo **FTP (File Transfer Protocol)** opera por padrão na porta `21/TCP` e transmite credenciais em texto claro.

#### Comando no Medusa:
```bash
medusa -h 192.168.56.101 -u msfadmin -P /usr/share/wordlists/rockyou.txt -M ftp -t 5 -f
```

* `-h 192.168.56.101`: Endereço IP do host alvo.
* `-u msfadmin`: Usuário específico a ser testado.
* `-P rockyou.txt`: Caminho da *wordlist* de senhas.
* `-M ftp`: Módulo de autenticação para o serviço FTP.
* `-t 5`: Número de *threads* simultâneas (evita sobrecarga no serviço).
* `-f`: Encerra o processo assim que a primeira senha válida for encontrada (*stop on found*).

**Respostas do Protocolo:**
- Código `530 Login incorrect`: Tentativa falha. O Medusa segue para a próxima senha.
- Código `230 User logged in, proceed`: Credencial válida identificada com sucesso.

---

### Cenário 2: Formulários Web (DVWA)

Aplicações Web utilizam os protocolos **HTTP/HTTPS** (portas `80`/`443`). Diferente de serviços de rede diretos, as credenciais trafegam por métodos como `POST` ou `GET`, e o servidor quase sempre responde com código `200 OK`.

#### Comando no Medusa:
```bash
medusa -h 192.168.56.101 -u admin -P wordlist_senhas.txt -M web-form -m "URL:/dvwa/login.php:FORM:username=&password=&Login=Login:DENY:Login failed" -f
```

* `-M web-form`: Módulo para envio de requisições a formulários web.
* `-m "..."`: Parâmetros específicos do formulário:
  * `URL`: Caminho do endpoint de login.
  * `FORM`: Nomes dos campos do formulário HTML (`username`, `password`, botão `Login`).
  * `DENY`: Mensagem retornada pelo sistema em caso de falha (`Login failed`). Caso a resposta não contenha esse texto, o Medusa assume que o login foi bem-sucedido.

---

### Cenário 3: Protocolo SMB e Password Spraying

O protocolo **SMB (Server Message Block)** opera nas portas `445/TCP` e `139/TCP`, sendo essencial para compartilhamento de recursos e autenticação em redes Windows.

#### 1. Enumeração Prévia de Usuários:
```bash
enum4linux -U 192.168.56.101 > usuarios_identificados.txt
```

#### 2. Execução do Password Spraying com Medusa:
```bash
medusa -h 192.168.56.101 -U usuarios_identificados.txt -p "Empresa@2024" -M smbnt -t 2
```

* `-U usuarios_identificados.txt`: Lista contendo múltiplos nomes de usuários.
* `-p "Empresa@2024"`: **Flag `-p` minúscula**: testa uma única senha estática em toda a lista.
* `-M smbnt`: Módulo para autenticação NTLM/SMB.
* `-t 2`: Taxa de concorrência baixa para evitar tráfego anômalo e bloqueios.

---

## 3. Matriz de Mitigação e Defesa (Blue Team)

| Vetor de Ataque | Serviço Afetado | Controles e Medidas de Mitigação |
| :--- | :--- | :--- |
| **Dicionário / Força Bruta** | FTP / SSH | • Implementação de **Fail2ban** (bloqueio por IP após falhas consecutivas).<br>• Migração do FTP para **SFTP**.<br>• Política de senhas com comprimento mínimo (12+ caracteres). |

| **Força Bruta Web** | Formulários HTTP | • **MFA / 2FA** obrigatório.<br>• Inserção de **CAPTCHA** adaptativo após 3 tentativas.<br>• *Rate limiting* e proteção via **WAF (Web Application Firewall)**. |

| **Password Spraying** | SMB / Active Directory | • Monitoramento de eventos de logon no SIEM (Event ID `4625` em lote).<br>• Filtro de senhas corporativas proibidas (estações do ano, nome da empresa).<br>• Desativação de SMBv1 e isolamento de rede da porta 445. |

---

## 4. Conclusão e Aprendizados

A realização deste estudo permitiu compreender na prática como funcionam os mecanismos de autenticação de diferentes serviços de rede e aplicações web. 

Ficou evidente que a segurança de um ambiente não deve depender exclusivamente da complexidade das senhas dos usuários, mas sim de uma arquitetura de **Defesa em Profundidade**, integrando:
1. Autenticação Multifator (MFA).
2. Monitoramento e correlação de logs de segurança em tempo real (SIEM).
3. Mecanismos automatizados de bloqueio (*Rate Limiting* / *Fail2ban*).

---

## 5. Referências e Recursos

- [Kali Linux Documentation](https://www.kali.org/)
- [Medusa Documentation](http://www.foofus.net/jmk/medusa/medusa.html)
- [DVWA (Damn Vulnerable Web Application)](http://www.dvwa.co.uk/)
- [NIST Special Publication 800-63B - Digital Identity Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [DIO - Santander Cibersegurança do Zero à Prática](https://dio.me)
