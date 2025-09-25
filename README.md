# 🛡️ Lab de Segurança: Web Application Firewall (WAF) ModSecurity + OWASP CRS com Damn Vulnerable Web Application (DVWA)

## 📚  FORMAÇÃO CYBERSEC - *Kensei Cybersec / Vai na Web*
 

Este projeto tem como objetivo testar a eficácia do **WAF ModSecurity** com as regras do **OWASP Core Rule Set (CRS)** na proteção de uma aplicação web vulnerável (**DVWA**), 
simulando ataques comuns como **SQL Injection (SQLi)** e **Cross-Site Scripting (XSS)**. O ambiente foi construído com **Docker** para garantir isolamento e reprodutibilidade.

## 📋 Objetivos

- Configurar um ambiente isolado com containers **Docker**
- Testar ataques de **SQLi** e **XSS** contra a aplicação **DVWA**
- Avaliar a detecção e bloqueio de ataques pelo **WAF ModSecurity**
- Monitorar logs em tempo real com **Dozzle**
- Aplicar o framework de resposta a incidentes **NIST IR**

### 🛠️ Serviços e Endpoints

- 🐧**Kali Linux**: IP `192.168.35.10`
- 🛡️**WAF ModSecurity**: 🌐 (HTTP) `8080` e 🔒 (HTTPS) `8443`
- 🎯 **DVWA**: Nível de segurança `Low`⚠️
- 🔒 **Dozzle**: Interface web em `http://localhost:9999` 

## ⚙️ Configurações:

### 📦 Pré-requisitos:
- **Docker** e Docker Compose
- Navegador web
- Terminal (macOS Terminal, Windows PowerShell, Linux Terminal)
- Git

### 🚀 Execução:

- Verificar **Docker**
```docker --version```
```docker-compose --version```
- Navegar até o diretório do Lab, verificar os arquivos necessários
  e iniciar todos os containers```cd labs  docker compose up -d --build ```
- verificar status dos containers ```docker ps```
- Testar Conectividade ```curl -s http://localhost:8080 | head -5```

### 🔧 Configuração do DVWA:
1. Acessar e Configurar **DVWA**:
   - URL```http://localhost:8080```
   - Login Usuario: *admin* / senha: *password*
2. Clique em "*Setup*" → "*Create / Reset Database*"
3. Clique em "*DVWA Securit*" → Selecione "*Low*" → "*Submit*"
4. Mantenha o navegador aberto para sessão ativa

### 🔍 Reconhecimento com **Nmap**:
1. Acessar container Kali: ```docker exec -it kali_lab35 /bin/bash```
2. Executar Scan Nmap:```nmap -sS -sV waf_modsec```
3. Resultado esperado:``` 8080/tcp open  http nginx --- 8443/tcp open  ssl/http nginx```
4. Sair do container: ```exit```

### 📊 Monitoramento com **Dozzle**:
1. Acessar URL: ```http://localhost:9999```
2. Login (Usuário: *admin* / Senha: *admin*)
3. Itens para monitorar:
   - Logs em tempo real do container *waf_modsec*
   - Rule IDs: 942100 (SQLi), 941100 (XSS)
   - Status HTTP: 302 (detecção) vs 403 (bloqueio)


### 🎛️ Modos de operação do **WAF**:

- Modo **Detecção**: ```MODSEC_RULE_ENGINE=DetectionOnly```
- Modo **Bloqueio**: ```MODSEC_RULE_ENGINE=On```
- Altere a variável no arquivo *docker-compose.yml* e reinicie o container:
```docker compose up -d --force-recreate waf_modsec```



## 🕵️‍♀️ Testes Realizados:

### ✅ Teste 1 - **Modo Detecção**:
- Editar arquivo *docker-compose.yml* **Modo Detecção**: ```MODSEC_RULE_ENGINE=DetectionOnly```
 
- Reiniciar **WAF**:```docker compose up -d --force-recreate waf_modsec```

- SQL Injection (**SQLi**)
   ```docker exec kali_lab35 curl -s "http://waf_modsec:8080/vulnerabilities/sqli/?id=1'+OR+'1'='1'--+-&Submit=Submit" \-H "Host: dvwa" \-H "Cookie: PHPSESSID=test; security=low" \-w "Status: %{http_code}\n"```

- Cross-Site Scripting (**XSS**)
```docker exec kali_lab35 curl -s "http://waf_modsec:8080/vulnerabilities/xss_r/?name=%3Cscript%3Ealert%28%22XSS%22%29%3C/script%3E" \-H "Host: dvwa" \-H "Cookie: security=low" \-w "Status: %{http_code}\n"```

- Resultado esperado: **Status 302**
  

### ✅ Teste 2 - **Modo Bloqueio**: 
- Editar arquivo *docker-compose.yml* **Modo Bloqueio**: ```MODSEC_RULE_ENGINE=On```
 
- Reiniciar **WAF**: ```docker compose up -d --force-recreate waf_modsec```
  
- SQL Injection (**SQLi**)
   ```docker exec kali_lab35 curl -s "http://waf_modsec:8080/vulnerabilities/sqli/?id=1'+OR+'1'='1'--+-&Submit=Submit" \-H "Host: dvwa" \-H "Cookie: PHPSESSID=test; security=low" \-w "Status: %{http_code}\n"```

- Cross-Site Scripting (**XSS**)
```docker exec kali_lab35 curl -s "http://waf_modsec:8080/vulnerabilities/xss_r/?name=%3Cscript%3Ealert%28%22XSS%22%29%3C/script%3E" \-H "Host: dvwa" \-H "Cookie: security=low" \-w "Status: %{http_code}\n"```

-  Resultado esperado: **Status 403**
  

## 📊 Resultados:

|   Modo   |        Comportamento                | Código HTTP |
|:--------:|:-----------------------------------:|:-----------:|
| Detecção | ATAQUE DETECTADO, MAS NÃO BLOQUEADO |     302     |
| Bloqueio |          ATAQUE BLOQUEADO           |     403 + página "403 Forbidden"     |


## 🛡️ Regras **OWASP CRS** acionadas:

   - **SQLi**: Rule ID 942100 
   - **XSS**: Rule ID 941100
     

## 🚨Resposta a Incidentes (**NIST IR**):

| Fase        | Ação                                                    | Evidência                         |
|-------------|---------------------------------------------------------|-----------------------------------|
| Detecção    | WAF identifica padrão de ataque e registra em log       | Logs com Rule ID e IP do atacante |
| Contenção   | Ativação do modo blocking                               | SecRuleEngine On                  |
| Erradicação | Bloqueio efetivo das requisições maliciosas             | HTTP 403                          |
| Recuperação | Aplicação mantém-se operacional para usuários legítimos | Logs de acesso normal             |


## ✅ Conclusões:

- O **WAF ModSecurity** com **OWASP CRS** mostrou-se eficaz contra **SQLi** e **XSS**.
 
- O modo **DetectionOnly** é útil para análise e aprendizado.

- O modo **Blocking** é essencial para ambientes de produção.

- O monitoramento em tempo real com **Dozzle** facilitou a identificação de ataques.

 - A estrutura **NIST IR** forneceu um guia claro para resposta a incidentes.


## 📋Recomendações:
1. Mantenha o **WAF** em modo **Blocking** em produção.

2. Atualize regularmente as regras do **OWASP CRS**.

3. Monitore logs continuamente.

4. Automatize alertas de segurança.

5. Treine a equipe em resposta rápida a incidentes.


## 🧩 Arquitetura do Ambiente:

Diagrama foi desenvolvido no [draw.io](https://app.diagrams.net) 

<img width="557" height="600" alt="Image" src="https://github.com/user-attachments/assets/680060ca-f832-4814-844b-2347e8b9710b" />   


## 📄 Licença e Ética 

Este projeto foi desenvolvido *exclusivamente para fins educacionais* em  ambientes controlados. 
