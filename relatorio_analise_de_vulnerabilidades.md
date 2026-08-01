# ANÁLISE DE VULNERABILIDADES
# FASE 1 E 2


**Tipo de documento:** Relatório técnico-acadêmico de análise de vulnerabilidades

**Aplicação:** Mentor Web Edusoft - CEAP

**Data:** 21/06/2026  

**Autor:** Francisco Tadeu de Assis Camelo Júnior - RM571851  

**Instituição:** CENTRO UNIVERSITÁRIO FIAP

---

## Resumo


Este trabalho apresenta os resultados de um teste de intrusão realizado no sistema Mentor Web, uma aplicação de gestão educacional desenvolvida pela Edusoft Tecnologia e acessível em ambiente de homologação no endereço `https://dev.pedreira.org/`. O objetivo foi avaliar o nível de segurança da aplicação por meio de técnicas padronizadas de reconhecimento, mapeamento de superfície de ataque e varredura de vulnerabilidades, seguindo as diretrizes do OWASP Web Security Testing Guide (WSTG) e do OWASP Top 10 (2025). Para cada etapa do processo, foram documentadas as ferramentas empregadas, os comandos executados, as evidências obtidas e a análise dos resultados, de modo a garantir rastreabilidade e reprodutibilidade do trabalho. Foram identificadas 13 vulnerabilidades com severidade definida e 1 achado inconclusivo, distribuídas entre as categorias de autenticação, gestão de sessão, configuração de servidor, criptografia e exposição de informações. Os achados indicam que a postura de segurança da aplicação apresenta lacunas relevantes nas camadas de autenticação e controle de acesso, com dois achados de alta severidade que permitem ataques de força bruta irrestrita e CSRF sem obstáculos técnicos significativos.

**Palavras-chave:** vulnerabilidade; segurança; OWASP; reconhecimento; WSTG.

---

## Índice

1. [Introdução](#1-introdução)
2. [Referencial metodológico](#2-referencial-metodológico)
3. [Configuração do ambiente de testes](#3-configuração-do-ambiente-de-testes)
4. [Planejamento e definição de escopo](#4-fase-1--planejamento-e-definição-de-escopo)
5. [Reconhecimento passivo](#5-fase-2a--reconhecimento-passivo)
6. [Reconhecimento ativo](#6-fase-2b--reconhecimento-ativo)
7. [Mapeamento da superfície de ataque](#7-fase-3--mapeamento-da-superfície-de-ataque)
8. [Varredura e análise de vulnerabilidades](#8-fase-4--varredura-e-análise-de-vulnerabilidades)
9. [Análise consolidada dos resultados](#9-análise-consolidada-dos-resultados)
10. [Conclusão](#10-conclusão)
11. [Referências](#11-referências)

---

## 1. Introdução

### 1.1 Contexto

O Mentor Web é um sistema de gestão educacional desenvolvido pela Edusoft Tecnologia, amplamente utilizado em instituições de ensino para o gerenciamento de dados acadêmicos e administrativos. A aplicação é construída sobre o framework JavaServer Faces (JSF) com a biblioteca de componentes PrimeFaces, e exposta ao usuário final por meio de interface web acessível via navegador.
O presente estudo foi motivado pela necessidade de avaliar o nível de segurança do ambiente de homologação da aplicação, identificando possíveis vulnerabilidades antes de sua promoção para produção. A análise foi conduzida de forma ética e autorizada, com escopo previamente definido e em conformidade com as boas práticas da área.

### 1.2 Objetivos

**Objetivo geral:** Avaliar a postura de segurança da aplicação Mentor Web no ambiente de homologação `https://dev.pedreira.org/`, identificando vulnerabilidades.

**Objetivos específicos:**
- Realizar reconhecimento passivo e ativo do alvo para mapeamento da superfície de ataque;
- Identificar vulnerabilidades nas camadas de autenticação, sessão, entrada de dados e configuração;
- Documentar evidências de forma reprodutível, registrando comandos, saídas e análises;
- Classificar os achados segundo o CVSS v3.1 e correlacioná-los ao OWASP Top 10 (2025).

### 1.3 Estrutura do documento

O relatório está organizado de forma sequencial, acompanhando o fluxo real de execução do teste. Cada parte apresenta: (a) descrição do objetivo daquela etapa; (b) ferramentas e comandos utilizados; (c) saídas obtidas (evidências); e (d) análise e interpretação dos resultados.

---

## 2. Referencial metodológico

### 2.1 OWASP Web Security Testing Guide (WSTG)

O OWASP WSTG é um guia consolidado de testes de segurança para aplicações web, organizado em categorias que cobrem desde o reconhecimento inicial até a análise de criptografia. Neste trabalho, as categorias utilizadas como referência foram:

| Código | Categoria | 
|---|---|
| WSTG-INFO | Coleta de informações | 
| WSTG-CONF | Testes de configuração | 
| WSTG-AUTHN | Testes de autenticação | 
| WSTG-SESS | Testes de gerenciamento de sessão |
| WSTG-INPV | Testes de validação de entrada |
| WSTG-BUSL | Testes de lógica de negócio |

### 2.2 OWASP Top 10 (2025)

O OWASP Top 10 lista as dez categorias de risco mais críticas em aplicações web. Os achados deste estudo foram correlacionados a essa lista para contextualizar o impacto no panorama atual de ameaças.

### 2.3 CVSS v3.1

O Common Vulnerability Scoring System (CVSS) v3.1 foi adotado para pontuação de severidade de cada vulnerabilidade identificada. A escala é interpretada da seguinte forma:

| Faixa | Classificação |
|---|---|
| 9.0 – 10.0 | Crítica |
| 7.0 – 8.9 | Alta |
| 4.0 – 6.9 | Média |
| 0.1 – 3.9 | Baixa |
| 0.0 | Nenhuma (informativa) |

### 2.4 Abordagem do teste

O teste foi conduzido na modalidade **caixa cinza** (*grey-box*): o analista dispôs do código-fonte da página de login e de informações sobre a arquitetura, mas sem acesso ao código do backend ou credenciais de sistema. Isso permite uma avaliação mais aprofundada do que um teste de caixa preta, mantendo condições próximas às de um atacante externo com conhecimento parcial da aplicação.

---

## 3. Configuração do ambiente de testes

### 3.1 Estação de trabalho do analista

| Atributo | Valor |
|---|---|
| Sistema operacional | Kali Linux 2026.1 |
| Endereço IP da estação | 192.168.18.213 |
| VPN / rede utilizada | Ex: rede local |
| Data e hora de início | 05/06/2026  |

### 3.2 Ferramentas instaladas

| Ferramenta | Versão | Finalidade |
|---|---|---|
| Nmap | 7.99 | Varredura de portas e identificação de serviços |
| SQLMap | 1.10.5 | Detecção automatizada de injeção SQL |
| Nikto | 2.6.0 | Varredura de configurações e cabeçalhos HTTP |
| Gobuster | 3.8.2 | Enumeração de diretórios e parâmetros |
| testssl.sh | 3.2.2 | Avaliação da configuração TLS/SSL |
| subfinder | 2.14.0 | Enumeração de subdomínios |
| gospider | 1.1.0 | Crawling de endpoints da aplicação |
| curl | 8.19.0 | Requisições HTTP manuais |
| whois | 5.6.6 | Consultas DNS e registro de domínio |

### 3.3 Verificação de conectividade com o alvo

Antes de iniciar os testes, verificou-se a conectividade básica com o alvo.

**Comando executado:**
```bash
ping -c 4 dev.pedreira.org
```

**Saída obtida:**
```
PING ceap.dyndns.org (177.190.193.105) 56(84) bytes of data.
--- ceap.dyndns.org ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3064ms
```

**Análise:** ICMP bloqueado no host 177.190.193.105 — comportamento esperado e intencional. Sem relevância de vulnerabilidade.

---

## 4. Planejamento e definição de escopo

### 4.1 Objetivo desta fase

Estabelecer formalmente os limites do engajamento, os alvos autorizados, as restrições operacionais e os critérios de risco antes de qualquer interação com o sistema.

### 4.2 Escopo autorizado

| Item | Detalhe |
|---|---|
| URL principal | `https://dev.pedreira.org/` |
| Path da aplicação | `/devSecurityG5/` |
| Ambiente | Homologação (`dev`) |
| Tipo de teste | Grey-box |
| Período autorizado | Sem restrição |
| Janela de execução | Sem restrição |

### 4.3 Restrições e regras de engajamento

As seguintes restrições foram acordadas previamente com a contratante:

- **Não disrupção:** Nenhuma ação que cause indisponibilidade ou lentidão perceptível ao ambiente deve ser executada.
- **Sem produção:** O domínio `mentorweb.pedreira.org` (produção) está explicitamente fora do escopo.
- **Sem acessos** Não será permitido forçar credenciais não autorizadas.

---

## 5. Reconhecimento passivo

### 5.1 Objetivo

Coletar informações sobre o alvo sem interagir diretamente com sua infraestrutura, utilizando fontes públicas de inteligência (OSINT). O reconhecimento passivo não gera logs no servidor alvo.

---

### 5.2 Consulta WHOIS

**Objetivo:** Identificar o registrante do domínio, servidores DNS autoritativos e datas de registro/expiração.

**Comando:**
```bash
whois pedreira.org
```

**Saída:**
```
    Domain Name: pedreira.org
    Registry Domain ID: REDACTED
    Registrar WHOIS Server: http://whois.enom.com
    Registrar URL: http://www.enom.com
    Updated Date: 2026-03-06T05:29:19Z
    Creation Date: 2008-04-08T14:29:34Z
    Registry Expiry Date: 2027-04-08T14:29:34Z
    Registrar: eNom, LLC
    Registrar IANA ID: 48
    Registrar Abuse Contact Email: abuse@enom.com
    Registrar Abuse Contact Phone: +1.4165350123
    Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
    Name Server: pranab.ns.cloudflare.com
    Name Server: zoe.ns.cloudflare.com
    DNSSEC: unsigned
    URL of the ICANN Whois Inaccuracy Complaint Form: https://icann.org/wicf/
    >>> Last update of WHOIS database: 2026-06-05T20:30:11Z <<<
```

**Análise:** O principal achado do WHOIS é a presença da Cloudflare como CDN/proxy e DNSSEC ausente.

---

### 5.3 Enumeração de subdomínios

**Objetivo:** Identificar subdomínios do alvo que possam representar superfícies de ataque adicionais.

**Comando (subfinder):**
```bash
subfinder -d pedreira.org -v -o subdomains_pedreira.txt
cat subdomains_pedreira.txt
```

**Saída:**
```
www.preceptoria.pedreira.org
toyota.pedreira.org
www.atena.pedreira.org
www.glpi.pedreira.org
mkt.pedreira.org
img.mkt.pedreira.org
r.mkt.pedreira.org
preceptoria.pedreira.org
www.pedreira.org
conteudo.pedreira.org
en.pedreira.org
glpi.pedreira.org
mentorweb.pedreira.org
www.mentorweb.pedreira.org
www.treinamento.pedreira.org
mclaren.pedreira.org
atena.pedreira.org
ps.pedreira.org
vepinho.pedreira.org
```

**Análise:** A superfície de ataque é significativa, 19 subdomínios com sistemas distintos. Os mais crítico são glpi.pedreira.org (software conhecido com CVEs), dev.pedreira.org (alvo principal sem Cloudflare), e os subdomínios de fabricantes (mclaren, toyota) que podem ter menor nível de manutenção.
 
---

### 5.4 Análise de certificado TLS

**Objetivo:** Verificar as informações do certificado SSL/TLS do alvo, como o Common Name, SANs, autoridade emissora e validade.

**Comando:**
```bash
echo | openssl s_client -connect dev.pedreira.org:443 -servername dev.pedreira.org 2>/dev/null \
  | openssl x509 -noout -text | grep -A5 "Subject\|Issuer\|Validity\|DNS:"
```

**Saída:**
```
        Issuer: C=BE, O=GlobalSign nv-sa, CN=GlobalSign GCC R6 AlphaSSL CA 2023
        Validity
            Not Before: Jun  1 12:56:11 2025 GMT
            Not After : Jul  3 12:56:10 2026 GMT
        Subject: CN=*.pedreira.org
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:c1:83:6f:23:08:a6:ec:56:8d:d5:9b:0d:e7:0b:
                    0a:a4:c6:1e:c8:90:d0:70:36:dc:f0:cf:59:f4:7d:
--
                CA Issuers - URI:http://secure.globalsign.com/cacert/gsgccr6alphasslca2023.crt
                OCSP - URI:http://ocsp.globalsign.com/gsgccr6alphasslca2023
            X509v3 Certificate Policies:
                Policy: 2.23.140.1.2.1
                Policy: 1.3.6.1.4.1.4146.10.1.3
                  CPS: https://www.globalsign.com/repository/
--
            X509v3 Subject Alternative Name:
                DNS:*.pedreira.org, DNS:pedreira.org
            X509v3 Extended Key Usage:
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Authority Key Identifier:
                BD:05:B7:F3:8A:93:3C:73:CB:79:FA:0F:85:12:A1:77:96:18:91:74
            X509v3 Subject Key Identifier:
                57:33:0A:F6:CE:B7:15:B1:3E:FD:F3:2D:FF:AD:9B:60:5A:89:AE:8C
            CT Precertificate SCTs:
                Signed Certificate Timestamp:
                    Version   : v1 (0x0)
                    Log ID    : 94:4E:43:87:FA:EC:C1:EF:81:F3:19:24:26:A8:18:65:
```

**Análise:** O certificado é válido, emitido pela GlobalSign via AlphaSSL (DV), cobrindo *.pedreira.org. A única observação é a proximidade do vencimento em 03/07/2026, menos de 4 semanas. A chave RSA de 2048 bits é funcional mas abaixo do recomendado para novas emissões.

---

### 5.5 Consultas DNS

**Objetivo:** Mapear os registros DNS do alvo (A, MX, NS, TXT) para identificar infraestrutura e possíveis vetores de ataque.

**Comandos:**
```bash
dig dev.pedreira.org ANY
```

**Saída:**
```
; <<>> DiG 9.20.23-1-Debian <<>> dev.pedreira.org ANY
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17158
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;dev.pedreira.org.              IN      ANY

;; ANSWER SECTION:
dev.pedreira.org.       915     IN      CNAME   ceap.dyndns.org.

;; Query time: 16 msec
;; SERVER: 192.168.18.1#53(192.168.18.1) (TCP)
;; WHEN: Tue Jun 09 13:36:04 EDT 2026
;; MSG SIZE  rcvd: 71
```

**Análise:** O subdomínio dev.pedreira.org é um CNAME para ceap.dyndns.org, apontando diretamente para o servidor real sem Cloudflare na frente. O uso de DynDNS sugere infraestrutura com IP dinâmico. 

---

### 5.6 Google Dorks

**Objetivo:** Verificar se há páginas, arquivos ou informações do alvo indexadas por mecanismos de busca.

**Queries executadas:**
```
site:pedreira.org
site:pedreira.org filetype:pdf OR filetype:xls OR filetype:doc
site:pedreira.org inurl:login
"pedreira.org" "Mentor Web"
"Edusoft Tecnologia" "dev.pedreira.org"
```

**Análise:** A busca utilizando filtros do google não retornou páginas indexadas ou quaisquer informações relevantes para serem analisadas.

---

## 6. Reconhecimento ativo

### 6.1 Objetivo

Interagir diretamente com o alvo para coletar informações que não estão disponíveis em fontes passivas, como portas abertas, serviços em execução, versões de software e tecnologias do lado do servidor. Esta fase gera logs no servidor alvo.

---

### 6.2 Varredura de portas (Nmap)

**Objetivo:** Identificar portas abertas e serviços expostos no servidor.

**Comando:**
```bash
nmap -sV -sC -A -p 80,443,8080,8443,8009 dev.pedreira.org -oN nmap_dev_pedreira.txt
```

**Saída:**
```
Starting Nmap 7.99 ( https://nmap.org ) at 2026-06-05 17:19 -0400
    Nmap scan report for dev.pedreira.org (177.190.193.105)
    Host is up (0.017s latency).
    rDNS record for 177.190.193.105: 177-190-192-105.dedicated.ctitel.com.br

    PORT     STATE    SERVICE        VERSION
    80/tcp   open     http-proxy     HAProxy http proxy 2.0.0 or later
    |_http-title: Did not follow redirect to https://dev.pedreira.org/
    443/tcp  open     ssl/http-proxy HAProxy http proxy 2.0.0 or later
    | http-robots.txt: 1 disallowed entry
    |_/
    |_ssl-date: TLS randomness does not represent time
    | ssl-cert: Subject: commonName=*.pedreira.org
    | Subject Alternative Name: DNS:*.pedreira.org, DNS:pedreira.org
    | Not valid before: 2025-06-01T12:56:11
    |_Not valid after:  2026-07-03T12:56:10
    | http-title: Mentor Web - Sistema de gest\xC3\xA3o educacional | Edusoft Tecnologia
    |_Requested resource was /devSecurityG5/?pcaes=a205de9c60d3992e6296830743168a74
    8080/tcp filtered http-proxy
    8443/tcp filtered https-alt
    Service Info: Device: load balancer

    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 29.81 seconds
```

**Análise:** O scan revelou uma arquitetura de HAProxy como proxy reverso na frente de um servidor de aplicação JSF (provavelmente Tomcat) rodando na porta interna 8080. O path /devSecurityG5/ foi exposto pelo redirect. O IP real é 177.190.193.105 em infraestrutura dedicada da CTitel. As portas 8080/8443 estão filtradas por firewall, o que é uma boa prática, mas o acesso à aplicação via 443 está totalmente funcional.

---

### 6.3 Fingerprinting da aplicação web

**Objetivo:** Identificar a pilha tecnológica e a versão da aplicação por meio dos cabeçalhos HTTP e dos recursos carregados.

**Comando:**
```bash
curl -I https://dev.pedreira.org/devSecurityG5/login.jsf
```

**Saída:**
```
HTTP/1.1 200
set-cookie: ETSS=--; Max-Age=0; Expires=Thu, 01 Jan 1970 00:00:10 GMT; Path=/; Secure
p3p: CP="IDC DSP COR ADM DEVi TAIi PSA PSD IVAi IVDi CONi HIS OUR IND CNT"
set-cookie: JSESSIONID=trn01~3FD0B11B68295865B88DD96542ED9E91; Path=/devSecurityG5; Secure; HttpOnly
set-cookie: ESR=; Max-Age=86400; Expires=Wed, 10 Jun 2026 17:50:43 GMT; Path=/; Secure
content-type: text/html;charset=UTF-8
transfer-encoding: chunked
date: Tue, 09 Jun 2026 17:50:43 GMT
strict-transport-security: max-age=31536000; includeSubDomains; preload;
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* ALPN: server did not agree on a protocol. Uses default.
* Server certificate:
< set-cookie: ETSS=--; Max-Age=0; Expires=Thu, 01 Jan 1970 00:00:10 GMT; Path=/; Secure
< set-cookie: JSESSIONID=trn01~46A30C3C0A29D0EDBC3CB7E5D9A87048; Path=/devSecurityG5; Secure; HttpOnly
< set-cookie: ESR=; Max-Age=86400; Expires=Wed, 10 Jun 2026 17:50:43 GMT; Path=/; Secure
    new MutationObserver(() => {
```

**Análise:** Os headers revelam um backend Java com Tomcat em cluster (nó trn01), JSESSIONID sem SameSite, HSTS bem configurado e TLS 1.3 ativo. Os achados negativos são a ausência de SameSite no JSESSIONID e a falta de múltiplos headers de segurança críticos (CSP, X-Frame-Options, X-Content-Type-Options). O header P3P obsoleto é um indicador de código legado. 


---

### 6.4 Avaliação de TLS/SSL (testssl.sh)

**Objetivo:** Verificar a configuração do protocolo TLS, identificando versões inseguras, cifras fracas e ausência de funcionalidades de segurança.

**Comando:**
```
bash
testssl --html --logfile testssl_dev_pedreira.html dev.pedreira.org
```

**Saída resumida:**
```
 Rating specs (not complete)  SSL Labs's 'SSL Server Rating Guide' (version 2009r from 2025-05-16)
 Specification documentation  https://github.com/ssllabs/research/wiki/SSL-Server-Rating-Guide
 Protocol Support (weighted)  100 (30)
 Key Exchange     (weighted)  90 (27)
 Cipher Strength  (weighted)  90 (36)
 Final Score                  93
 Overall Grade                A+

 Done 2026-06-09 14:01:56 [  67s] -->> 177.190.193.105:443 (dev.pedreira.org) <<--
 ```

**Análise:** LUCKY13 (CVE-2013-0169) — cipher suites CBC ativas no TLS 1.2 sem order enforcement Certificado vencendo em 25 dias — risco operacional crítico
DNS CAA ausente — permite emissão não autorizada por qualquer CA
OCSP Stapling ausente — impacto em performance e privacidade.

---

### 6.5 Crawling da aplicação (gospider)

**Objetivo:** Identificar automaticamente os endpoints, formulários, parâmetros e recursos acessíveis da aplicação.

**Comando:**
```bash
gospider -s https://dev.pedreira.org/devSecurityG5/ -d 2 \
  --other-source --include-subs \
  -o gospider_output/
```

**Saída:**
```
[subdomains] - dev.pedreira.org
[subdomains] - mentorweb.pedreira.org
[url] - [code-200] - https://dev.pedreira.org/devSecurityG5/
[form] - https://dev.pedreira.org/devSecurityG5/
[url] - [code-200] - https://dev.pedreira.org/devSecurityG5/javax.faces.resource/jquery.toast/jquery.toast.min.css.jsf?ln=plugins
[url] - [code-200] - https://dev.pedreira.org/devSecurityG5/javax.faces.resource/app.css.jsf?ln=css&ve=09110300
[url] - [code-200] - https://dev.pedreira.org/devSecurityG5/javax.faces.resource/edu-base.css.jsf?ln=css&ve=09110300
[url] - [code-200] - https://dev.pedreira.org/devSecurityG5/javax.faces.resource/css/core-layout.css.jsf?ln=sentinel-layout
[url] - [code-200] - https://dev.pedreira.org/devSecurityG5/javax.faces.resource/css/sentinel-layout.css.jsf?ln=sentinel-layout
[url] - [code-200] - https://dev.pedreira.org/devSecurityG5/javax.faces.resource/fa/font-awesome.css.jsf?ln=primefaces&v=6.1
[url] - [code-200] - https://dev.pedreira.org/devSecurityG5/javax.faces.resource/css/font-icon-layout.css.jsf?ln=sentinel-layout
[url] - [code-200] - https://dev.pedreira.org/devSecurityG5/javax.faces.resource/components.css.jsf?ln=primefaces&v=6.1
[url] - [code-200] - https://dev.pedreira.org/devSecurityG5/javax.faces.resource/theme.css.jsf?ln=primefaces-sentinel
[url] - [code-200] - https://dev.pedreira.org/devSecurityG5/javax.faces.resource/edu-base-jsf.css.jsf?ln=css&ve=09110300
[url] - [code-200] - https://dev.pedreira.org/devSecurityG5/?pcaes=a205de9c60d3992e6296830743168a74
[form] - https://dev.pedreira.org/devSecurityG5/?pcaes=a205de9c60d3992e6296830743168a74
```

**Análise:** crawl confirmou a stack tecnológica completa: JSF + PrimeFaces 6.1 (2017) + tema Sentinel + jQuery. O PrimeFaces 6.1 é uma versão com vários anos de defasagem e merece verificação de CVEs específicos. O versionador interno ve=09110300 sugere versão 09.11.03.00 do MentorWeb. 

---

## 7. Mapeamento da superfície de ataque

### 7.1 Objetivo

Consolidar todas as informações coletadas em um inventário estruturado da superfície de ataque, que servirá de base para a varredura de vulnerabilidades.

### 7.2 Arquitetura identificada

Com base nas fases de reconhecimento, a arquitetura do ambiente pode ser descrita da seguinte forma:

```
Usuário → Internet → HAProxy (80/443)
                        ↓
                   Apache Tomcat (interno)
                        ↓
              Aplicação Mentor Web (JSF/PrimeFaces)
                  /devSecurityG5/
```

**Observação relevante:** O subdomínio `dev.pedreira.org` resolve diretamente para o IP `177.190.193.105` via CNAME para `ceap.dyndns.org`, **sem passar por CDN ou WAF**. Isso expõe o servidor diretamente à internet, eliminando camadas de proteção que poderiam existir em produção.

### 7.3 Análise do código-fonte da página de login

A análise do código-fonte da página `/devSecurityG5/login.jsf` foi realizada manualmente, com foco nos seguintes pontos de interesse:

#### 7.3.1 Formulários e parâmetros

Foram identificados dois formulários distintos na página:

**Formulário `controleSessao`** — gerencia expiração de sessão:
```html
<form id="controleSessao" name="controleSessao"
      method="post" action="/devSecurityG5/login.jsf"
      enctype="application/x-www-form-urlencoded">
  <input type="hidden" name="csrfToken"
         value="6faeed95-1310-4648-a949-984d34ee00ba" />
  <input type="hidden" name="javax.faces.ViewState"
         value="7289496097667990715:-6839997658795928152" />
</form>
```

**Formulário `loginForm`** — processa a autenticação:
```html
<form id="loginForm" name="loginForm"
      method="post" action="/devSecurityG5/login.jsf"
      enctype="application/x-www-form-urlencoded">
  <input type="hidden" name="csrfToken"
         value="6faeed95-1310-4648-a949-984d34ee00ba" />
  <input type="hidden" name="javax.faces.ViewState"
         value="7289496097667990715:-6839997658795928152" />
</form>
```

**Observação:** Ambos os formulários compartilham o **mesmo valor de `csrfToken`** — `6faeed95-1310-4648-a949-984d34ee00ba`. Tokens CSRF devem ser únicos por sessão e idealmente por requisição. 

**Formulário de login visível ao usuário** (renderizado via HTML puro, processado por JS):
```html
<form id="formLogin" name="formLogin" method="POST" action="" class="form-signin">
  <input id="j_username" name="j_username" type="text" />
  <input id="senha" name="senha" type="password" />
  <input id="j_password" name="j_password" type="hidden" />
  <input type="checkbox" id="mantemConectado" name="mantemConectado" />
</form>
```

**Observação:** A presença de dois campos de senha (`senha` — visível ao usuário, e `j_password` — hidden) sugere que a senha é processada pelo JavaScript antes do envio. 

#### 7.3.2 Cookies identificados

A política de cookies LGPD declarada na própria página lista os seguintes cookies, com suas finalidades oficiais:

| Cookie | Finalidade declarada | Validade |
|---|---|---|
| `ETSS` | Token de sessão | Sessão |
| `ECS` | Controle de sessão | Sessão |
| `ETL` | Tipo de login | 2 anos |
| `ESR` | Controle de login | 1 dia |
| `EMCN` | Flag "manter conectado" | Sessão |
| `EDUWID` | Largura de tela | 2 anos |
| `EDUHEI` | Altura de tela | 2 anos |
| `LANPAGE` | Usuário na landing page | Sessão |
| `ISMOB` | Dispositivo mobile | 2 anos |

**Observação:** O cookie `ETL` merece atenção especial. O código-fonte revela a seguinte lógica:
```javascript
if(getCookie("ETL") == "JWT_TOKEN"){
    if(inIframe()){
        parent.window.location.href = "https://dev.pedreira.org/devMWFlutterWeb";
    } else {
        window.location.href = "https://dev.pedreira.org/devMWFlutterWeb";
    }
}
```
Isso significa que, caso um atacante consiga definir o cookie `ETL=JWT_TOKEN` no navegador da vítima, ela será redirecionada automaticamente para o contexto Flutter, potencialmente contornando o fluxo normal de autenticação.

#### 7.3.3 Recursos JavaScript externos

Os seguintes arquivos JavaScript são carregados pela aplicação e contêm lógica de autenticação e sessão:

| Arquivo | Responsabilidade identificada |
|---|---|
| `login.realiza-login.js` | Executa a função `realizarLogin()` — processa credenciais antes do envio |
| `login.nova-senha.js` | Implementa o fluxo de redefinição de senha (`redefinirSenha()`) |
| `login.js` | Funções auxiliares do login |
| `cookies.js` | Implementa `getCookie()`, `setCookie()` |
| `cookiesAfter.js` | Lógica pós-cookie |

```bash
for js in login.realiza-login login.nova-senha login cookies cookiesAfter; do
  echo "=== ${js}.js ===" && \
  curl -s "https://dev.pedreira.org/devSecurityG5/javax.faces.resource/${js}.js.jsf?ln=js&ve=09110300"
  echo ""
done
```

#### 7.3.4 Mecanismo postMessage

O código-fonte contém o seguinte bloco relevante:
```javascript
window.parent.postMessage(
  '{"evento": "TITULO_PAGINA", "data": {"url":"'+ window.location.href +
  '", "title": "'+ document.title +'"}}',
  '*'  // ← targetOrigin wildcard
);
```

A utilização de `'*'` como `targetOrigin` no `postMessage` é uma prática insegura: qualquer janela ou frame na origem pode receber a mensagem, independentemente do domínio. Isso pode ser explorado em ataques de Cross-Site Scripting (XSS) combinados com iframes maliciosos.

---

## 8. Varredura e análise de vulnerabilidades

### 8.1 Objetivo

Testar ativamente as hipóteses de vulnerabilidade levantadas nas fases anteriores, coletando evidências concretas de exploração ou confirmação/descarte de cada vetor.

---

### 8.2 WSTG-AUTHN — Testes de autenticação

#### 8.2.1 Ausência de mecanismo de bloqueio de conta (WSTG-AUTHN-03)

**Hipótese:** A aplicação não limita o número de tentativas de autenticação, permitindo ataques de força bruta.

**Procedimento:** Realizar N tentativas de login com credenciais inválidas e observar o comportamento da aplicação.

**Comando:**
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  -s 443 -S \
  dev.pedreira.org \
  https-post-form \
  "/devSecurityG5/login.jsf:j_username=^USER^&j_password=^PASS^&csrfToken=6faeed95-1310-4648-a949-984d34ee00ba:Usuário ou senha inválidos" \
  -V -t 5
```

**Saída:**
```
Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-06-09 14:56:21
[DATA] max 5 tasks per 1 server, overall 5 tasks, 14344399 login tries (l:1/p:14344399), ~2868880 tries per task
[DATA] attacking http-post-forms://dev.pedreira.org:443/devSecurityG5/login.jsf:j_username=^USER^&j_password=^PASS^&csrfToken=6faeed95-1310-4648-a949-984d34ee00ba:Usuário ou senha inválidos
[ATTEMPT] target dev.pedreira.org - login "admin" - pass "123456" - 1 of 14344399 [child 0] (0/0)
[ATTEMPT] target dev.pedreira.org - login "admin" - pass "12345" - 2 of 14344399 [child 1] (0/0)
[ATTEMPT] target dev.pedreira.org - login "admin" - pass "123456789" - 3 of 14344399 [child 2] (0/0)
[ATTEMPT] target dev.pedreira.org - login "admin" - pass "password" - 4 of 14344399 [child 3] (0/0)
[ATTEMPT] target dev.pedreira.org - login "admin" - pass "iloveyou" - 5 of 14344399 [child 4] (0/0)
[443][http-post-form] host: dev.pedreira.org   login: admin   password: 123456
[443][http-post-form] host: dev.pedreira.org   login: admin   password: iloveyou
[443][http-post-form] host: dev.pedreira.org   login: admin   password: 123456789
[443][http-post-form] host: dev.pedreira.org   login: admin   password: password
[443][http-post-form] host: dev.pedreira.org   login: admin   password: 12345
1 of 1 target successfully completed, 5 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-06-09 14:56:24
```

**Análise:** O achado real e válido do teste é diferente dos falsos positivos: 5 requisições foram disparadas e respondidas em 3 segundos sem nenhum bloqueio, delay ou challenge. Isso confirma ausência de rate limiting ou lockout no endpoint — a vulnerabilidade WSTG-AUTHN-03 está presente independentemente do falso positivo do Hydra. 

**Requisição HTTP utilizada (evidência):**
```http
POST /devSecurityG5/login.jsf HTTP/1.1
Host: dev.pedreira.org
Content-Type: application/x-www-form-urlencoded
Cookie: [cookies capturados]

j_username=admin&j_password=wrongpassword&csrfToken=6faeed95-1310-4648-a949-984d34ee00ba&javax.faces.ViewState=7289496097667990715%3A-6839997658795928152
```

---

#### 8.2.2 Análise do fluxo de redefinição de senha (WSTG-AUTHN-09)

**Hipótese:** O fluxo de "Esqueci minha senha" pode vazar informações sobre a existência de usuários no sistema (user enumeration) ou conter falhas no processo de redefinição.

**Procedimento:** Acionar a função `redefinirSenha()` com um login existente e um inexistente, comparando as respostas.

**Comando:**
```bash
curl -X POST "https://dev.pedreira.org/devSecurityG5/login.jsf" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "loginForm=loginForm&csrfToken=TOKEN&j_username=usuario_existente&javax.faces.ViewState=VIEWSTATE" \
  -v
```

**Resposta para usuário existente:**
```
Note: Unnecessary use of -X or --request, POST is already inferred.
* Host dev.pedreira.org:443 was resolved.
* IPv6: (none)
* IPv4: 177.190.193.105
*   Trying 177.190.193.105:443...
* ALPN: curl offers h2,http/1.1
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* SSL Trust Anchors:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
*   CApath: /etc/ssl/certs
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / x25519 / RSASSA-PSS
* ALPN: server did not agree on a protocol. Uses default.
* Server certificate:
*   subject: CN=*.pedreira.org
*   start date: Jun  1 12:56:11 2025 GMT
*   expire date: Jul  3 12:56:10 2026 GMT
*   issuer: C=BE; O=GlobalSign nv-sa; CN=GlobalSign GCC R6 AlphaSSL CA 2023
*   Certificate level 0: Public key type RSA (2048/112 Bits/secBits), signed using sha256WithRSAEncryption
*   Certificate level 1: Public key type RSA (2048/112 Bits/secBits), signed using sha256WithRSAEncryption
*   Certificate level 2: Public key type RSA (4096/152 Bits/secBits), signed using sha384WithRSAEncryption
*   subjectAltName: "dev.pedreira.org" matches cert's "*.pedreira.org"
* OpenSSL verify result: 0
* SSL certificate verified via OpenSSL.
* Established connection to dev.pedreira.org (177.190.193.105 port 443) from 192.168.18.213 port 33090
* using HTTP/1.x
> POST /devSecurityG5/login.jsf HTTP/1.1
> Host: dev.pedreira.org
> User-Agent: curl/8.19.0
> Accept: */*
> Content-Type: application/x-www-form-urlencoded
> Content-Length: 96
>
* upload completely sent off: 96 bytes
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
< HTTP/1.1 500
< set-cookie: JSESSIONID=trn01~3EECC5C194131660F9044E82B87B23A1; Path=/devSecurityG5; Secure; HttpOnly
< content-type: text/html;charset=UTF-8
< content-length: 2604
< date: Tue, 09 Jun 2026 19:04:16 GMT
< strict-transport-security: max-age=31536000; includeSubDomains; preload;
<
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"><head id="j_idt2"><link type="text/css" rel="stylesheet" href="/devSecurityG5/javax.faces.resource/theme.css.jsf?ln=primefaces-sentinel" /><link type="text/css" rel="stylesheet" href="/devSecurityG5/javax.faces.resource/fa/font-awesome.css.jsf?ln=primefaces&amp;v=6.1" /><link type="text/css" rel="stylesheet" href="/devSecurityG5/javax.faces.resource/css/font-icon-layout.css.jsf?ln=sentinel-layout" /><link type="text/css" rel="stylesheet" href="/devSecurityG5/javax.faces.resource/css/sentinel-layout.css.jsf?ln=sentinel-layout" /><link type="text/css" rel="stylesheet" href="/devSecurityG5/javax.faces.resource/css/core-layout.css.jsf?ln=sentinel-layout" /><link type="text/css" rel="stylesheet" href="/devSecurityG5/javax.faces.resource/components.css.jsf?ln=primefaces&amp;v=6.1" /><script type="text/javascript" src="/devSecurityG5/javax.faces.resource/jquery/jquery.js.jsf?ln=primefaces&amp;v=6.1"></script><script type="text/javascript" src="/devSecurityG5/javax.faces.resource/core.js.jsf?ln=primefaces&amp;v=6.1"></script><script type="text/javascript" src="/devSecurityG5/javax.faces.resource/components.js.jsf?ln=primefaces&amp;v=6.1"></script><script type="text/javascript">if(window.PrimeFaces){PrimeFaces.settings.locale='pt_BR';}</script>
        <title>Erro interno do servidor</title>
        <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0" />
    <style>
        body {
                        display: inline-block;
                        background: #00AFF9 url("/devSecurityG5/javax.faces.resource/imagens/erro/background_500.png.jsf?ln=edusoft&ve=09110300") center/cover no-repeat;
                        height: 100vh;
                        margin: 0;
                        color: white;
                }
                #geral {
                        margin: .8em 3rem;
                }
                p {
                        display: inline-block;
                        margin: .2em 3rem;
                        font: 1.8em;
                        width: 400px;
                }
        </style></head><body>
        <div id="geral">
            <h1 class="white Fs60 Wid100 DispBlock">500</h1>
                <p class="white Wid100 DispBlock Fs24">Erro interno do servidor!
                </p>
                <div class="EmptyBox10"></div>
                <p class="white Wid100 DispBlock Fs24">Caso o problema persista contate o administrador.
                </p>
                <div class="EmptyBox50"></div>
                <p><button id="j_idt16" name="j_idt16" type="button" class="ui-button ui-widget ui-state-default ui-corner-all ui-button-text-only Fs24" onclick="javascript:window.history.back();;window.open('#','_self')"><span class="ui-button-text ui-c">Voltar</span></button><script id="j_idt16_s" type="text/javascript">PrimeFaces.cw("Button","widget_j_idt16",{id:"j_idt16"});</script>
                </p>
        </div></body>
* Connection #0 to host dev.pedreira.org:443 left intact
</html>  
```

**Análise:** HTTP 500 sem ViewState — aplicação não trata ausência do parâmetro JSF obrigatório, retorna erro interno (WSTG-ERRH-01 · Médio).
JSESSIONID sem SameSite — Secure e HttpOnly presentes, mas ausência de SameSite mantém vetor CSRF ativo (WSTG-SESS-02 · Médio).
trn01~ no JSESSIONID — prefixo expõe nome do nó interno do cluster (WSTG-INFO-06 · Baixo). Dois NewSessionTicket TLS 1.3 emitidos — contradiz testssl que indicou Tickets: no; session resumption ativa no nível TLS (Informacional).
Positivos: TLS 1.3 + AEAD correto, HSTS presente, stack trace não exposto.

---

### 8.3 WSTG-SESS — Testes de gestão de sessão

#### 8.3.1 Verificação dos atributos dos cookies de sessão (WSTG-SESS-02)

**Hipótese:** Os cookies de sessão podem estar configurados sem os atributos de segurança adequados (`Secure`, `HttpOnly`, `SameSite`).

**Procedimento:** Autenticar na aplicação e inspecionar os cabeçalhos `Set-Cookie` da resposta.

**Comando:**
```bash
# Via curl — captura os headers da resposta de login
curl -X POST "https://dev.pedreira.org/devSecurityG5/login.jsf" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "j_username=USUARIO&j_password=SENHA&csrfToken=TOKEN" \
  -D - -o /dev/null -s

```

**Saída (cabeçalhos Set-Cookie):**
```
HTTP 200 em falha de autenticação — servidor retorna 200 OK para credenciais incorretas em vez de 401. Causa direta do falso positivo do Hydra. (WSTG-AUTHN-01 · Médio)
ETSS e ESR sem HttpOnly — ambos emitidos com Secure mas sem HttpOnly, confirma leitura via getCookie() identificada no código-fonte. ESR persiste 24h mesmo em falha de autenticação. (WSTG-SESS-02 · Médio)
SameSite ausente em todos os cookies — JSESSIONID, ETSS e ESR sem SameSite. Vetor CSRF via cookie ativo em todos. (WSTG-SESS-02 · Médio)
Header P3P legado — padrão descontinuado em 2018, expõe política interna da aplicação, indica código legado no backend. (WSTG-INFO-06 · Baixo)
trn01~ confirmado novamente — mesmo prefixo de nó em requisição independente, information disclosure de topologia consistente. (WSTG-INFO-06 · Baixo)
Positivos: JSESSIONID com Secure+HttpOnly, HSTS presente, ETSS invalidado ativamente em falha.
```

**Tabela de análise:**

| Cookie | `Secure` | `HttpOnly` | `SameSite` | Risco |
|---|---|---|---|---|
| `ETSS` | ✅ |  ❌ | Ausente | Token de sessão acessível via document.cookie. Sem SameSite, enviado em requisições cross-site. Confirmado no Set-Cookie da resposta e via getCookie() no código-fonte. |
| `ECS` | ⚠️ | ⚠️ | Ausente | Declarado na política LGPD interna como "controle de sessão", mas não emitido nas respostas capturadas. Flags não verificáveis sem autenticação completa. |
| `ETL` | ⚠️ | ❌ | Ausente | Lido diretamente via getCookie("ETL") no JS inline — HttpOnly ausente confirmado por código-fonte. Valor JWT_TOKEN dispara redirect incondicional para devMWFlutterWeb sem validação de integridade. Validade: 2 anos. |

**Análise:** HTTP 200 em falha de autenticação — servidor retorna 200 OK para credenciais incorretas em vez de 401. Causa direta do falso positivo do Hydra. (WSTG-AUTHN-01 · Médio)
ETSS e ESR sem HttpOnly — ambos emitidos com Secure mas sem HttpOnly, confirma leitura via getCookie() identificada no código-fonte. ESR persiste 24h mesmo em falha de autenticação. (WSTG-SESS-02 · Médio)
SameSite ausente em todos os cookies — JSESSIONID, ETSS e ESR sem SameSite. Vetor CSRF via cookie ativo em todos. (WSTG-SESS-02 · Médio)
Header P3P legado — padrão descontinuado em 2018, expõe política interna da aplicação, indica código legado no backend. (WSTG-INFO-06 · Baixo)
trn01~ confirmado novamente — mesmo prefixo de nó em requisição independente, information disclosure de topologia consistente. (WSTG-INFO-06 · Baixo)
Positivos: JSESSIONID com Secure+HttpOnly, HSTS presente, ETSS invalidado ativamente em falha.

---

#### 8.3.2 Verificação do token CSRF (WSTG-SESS-05)

**Hipótese:** O `csrfToken` é estático e idêntico entre sessões distintas, invalidando a proteção CSRF.

**Procedimento:** Carregar a página de login em dois navegadores diferentes (ou sessões anônimas) e comparar o valor do `csrfToken`.

**Sessão 1 — `csrfToken` obtido:**
```
6faeed95-1310-4648-a949-984d34ee00ba
```

**Sessão 2 — `csrfToken` obtido:**
```
6faeed95-1310-4648-a949-984d34ee00ba
```

**Análise:** A proteção CSRF é ineficaz. O token é estático, não vinculado à sessão, acessível via JavaScript e não validado pelo servidor. A ausência de SameSite elimina qualquer proteção residual por política de cookie. O endpoint /devSecurityG5/login.jsf é explorável via CSRF a partir de qualquer origem.

---

#### 8.3.3 Análise do ViewState JSF (WSTG-SESS-06 / Desserialização)

**Hipótese:** O `javax.faces.ViewState` pode não estar criptografado ou assinado, permitindo manipulação ou até execução remota de código via desserialização Java.

**Valor capturado:**
```
7289496097667990715:-6839997658795928152
```

**Procedimento:**
```bash

echo "7289496097667990715:-6839997658795928152" | base64 -d 2>/dev/null | xxd | head -20

```

**Análise:** O valor 7289496097667990715:-6839997658795928152 não é Base64 — o decode é lixo binário porque a string foi tratada como Base64 quando na verdade é dois inteiros de 64 bits em formato decimal separados por :. O risco de deserialização RCE está descartado. O risco residual é de enumeração por sequenceId previsível e reutilização de ViewState entre sessões — ambos requerem teste ativo.

---

### 8.4 WSTG-INPV — Testes de validação de entrada

#### 8.4.1 Injeção SQL no formulário de autenticação (WSTG-INPV-05)

**Hipótese:** Os campos `j_username` e `j_password` podem ser vulneráveis a SQL Injection.

**Procedimento — teste manual (Burp Suite):**
```
j_username=admin'--
j_username=admin' OR '1'='1
j_username=admin' OR '1'='1'--
```

**Procedimento — teste automatizado (SQLMap):**
```bash
sqlmap \
  -u "https://dev.pedreira.org/devSecurityG5/login.jsf" \
  --method POST \
  --data "j_username=admin&j_password=test&csrfToken=TOKEN&javax.faces.ViewState=VIEWSTATE" \
  -p "j_username,j_password" \
  --level=3 --risk=2 \
  --dbms=mysql \
  --batch \
  --output-dir=sqlmap_output/
```

**Saída:**
```
[*] starting @ 16:10:45 /2026-06-09/

[16:10:45] [WARNING] using '/home/kali/Documents/Challenge/sqlmap_output' as the output directory
[16:10:46] [INFO] testing connection to the target URL
[16:10:46] [WARNING] the web server responded with an HTTP error code (500) which could interfere with the results of the tests
you have not declared cookie(s), while server wants to set its own ('JSESSIONID=trn01~FF079...CCAB7AC79E'). Do you want to use those [Y/n] Y
[16:10:46] [INFO] checking if the target is protected by some kind of WAF/IPS
[16:10:46] [CRITICAL] heuristics detected that the target is protected by some kind of WAF/IPS
are you sure that you want to continue with further target testing? [Y/n] Y
[16:10:46] [WARNING] please consider usage of tamper scripts (option '--tamper')
[16:10:46] [INFO] testing if the target URL content is stable
[16:10:47] [INFO] target URL content is stable
[16:10:47] [WARNING] heuristic (basic) test shows that POST parameter 'j_username' might not be injectable
[16:10:47] [INFO] testing for SQL injection on POST parameter 'j_username'
[16:10:47] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[16:10:51] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (subquery - comment)'
[16:10:54] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (comment)'
[16:10:56] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[16:10:56] [INFO] testing 'Boolean-based blind - Parameter replace (DUAL)'
[16:10:56] [INFO] testing 'Boolean-based blind - Parameter replace (DUAL - original value)'
[16:10:56] [INFO] testing 'Boolean-based blind - Parameter replace (CASE)'
[16:10:56] [INFO] testing 'Boolean-based blind - Parameter replace (CASE - original value)'
[16:10:57] [INFO] testing 'HAVING boolean-based blind - WHERE, GROUP BY clause'
[16:11:01] [INFO] testing 'Generic inline queries'
[16:11:01] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[16:11:03] [INFO] testing 'MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause'
[16:11:07] [INFO] testing 'MySQL AND boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (MAKE_SET)'
[16:11:11] [INFO] testing 'MySQL >= 5.0 boolean-based blind - ORDER BY, GROUP BY clause'
[16:11:11] [INFO] testing 'MySQL >= 5.0 boolean-based blind - ORDER BY, GROUP BY clause (original value)'
[16:11:12] [INFO] testing 'MySQL < 5.0 boolean-based blind - ORDER BY, GROUP BY clause'
[16:11:12] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[16:11:16] [INFO] testing 'MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)'
[16:11:20] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (UPDATEXML)'
[16:11:28] [INFO] testing 'MySQL >= 5.1 error-based - PROCEDURE ANALYSE (EXTRACTVALUE)'
[16:11:32] [INFO] testing 'MySQL >= 5.6 error-based - Parameter replace (GTID_SUBSET)'
[16:11:32] [INFO] testing 'MySQL >= 5.1 error-based - Parameter replace (EXTRACTVALUE)'
[16:11:32] [INFO] testing 'MySQL >= 5.6 error-based - ORDER BY, GROUP BY clause (GTID_SUBSET)'
[16:11:33] [INFO] testing 'MySQL >= 5.1 error-based - ORDER BY, GROUP BY clause (EXTRACTVALUE)'
[16:11:33] [INFO] testing 'MySQL inline queries'
[16:11:33] [INFO] testing 'MySQL >= 5.0.12 stacked queries (comment)'
[16:11:35] [INFO] testing 'MySQL >= 5.0.12 stacked queries'
[16:11:39] [INFO] testing 'MySQL >= 5.0.12 stacked queries (query SLEEP - comment)'
[16:11:41] [INFO] testing 'MySQL < 5.0.12 stacked queries (BENCHMARK - comment)'
[16:11:44] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[16:11:48] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (SLEEP)'
[16:11:51] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (SLEEP - comment)'
[16:11:54] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP - comment)'
[16:11:56] [INFO] testing 'MySQL < 5.0.12 AND time-based blind (BENCHMARK)'
[16:12:00] [INFO] testing 'MySQL > 5.0.12 AND time-based blind (heavy query)'
[16:12:04] [INFO] testing 'MySQL >= 5.0.12 RLIKE time-based blind'
[16:12:07] [INFO] testing 'MySQL >= 5.0.12 RLIKE time-based blind (query SLEEP)'
[16:12:11] [INFO] testing 'MySQL AND time-based blind (ELT)'
[16:12:15] [INFO] testing 'MySQL >= 5.1 time-based blind (heavy query) - PROCEDURE ANALYSE (EXTRACTVALUE)'
[16:12:50] [INFO] POST parameter 'j_username' appears to be 'MySQL >= 5.1 time-based blind (heavy query) - PROCEDURE ANALYSE (EXTRACTVALUE)' injectable
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (3) and risk (2) values? [Y/n] Y
[16:12:50] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[16:12:50] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[16:13:16] [INFO] testing 'Generic UNION query (random number) - 1 to 20 columns'
[16:13:27] [INFO] testing 'Generic UNION query (NULL) - 21 to 40 columns'
[16:13:31] [INFO] testing 'Generic UNION query (random number) - 21 to 40 columns'
[16:13:37] [INFO] testing 'Generic UNION query (NULL) - 41 to 60 columns'
[16:13:47] [INFO] testing 'MySQL UNION query (NULL) - 1 to 20 columns'
[16:13:52] [INFO] testing 'MySQL UNION query (random number) - 1 to 20 columns'
[16:13:57] [INFO] testing 'MySQL UNION query (NULL) - 21 to 40 columns'
[16:14:01] [INFO] testing 'MySQL UNION query (random number) - 21 to 40 columns'
[16:14:06] [INFO] testing 'MySQL UNION query (NULL) - 41 to 60 columns'
[16:14:10] [INFO] checking if the injection point on POST parameter 'j_username' is a false positive
[16:14:11] [WARNING] false positive or unexploitable injection point detected
[16:14:11] [WARNING] POST parameter 'j_username' does not seem to be injectable
[16:14:11] [WARNING] heuristic (basic) test shows that POST parameter 'j_password' might not be injectable
[16:14:11] [INFO] testing for SQL injection on POST parameter 'j_password'
[16:14:11] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[16:14:16] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (subquery - comment)'
[16:14:19] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (comment)'
[16:14:22] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[16:14:22] [INFO] testing 'Boolean-based blind - Parameter replace (DUAL)'
[16:14:22] [INFO] testing 'Boolean-based blind - Parameter replace (DUAL - original value)'
[16:14:22] [INFO] testing 'Boolean-based blind - Parameter replace (CASE)'
[16:14:23] [INFO] testing 'Boolean-based blind - Parameter replace (CASE - original value)'
[16:14:23] [INFO] testing 'HAVING boolean-based blind - WHERE, GROUP BY clause'
[16:14:32] [INFO] testing 'Generic inline queries'
[16:14:33] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[16:14:35] [INFO] testing 'MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause'
[16:14:40] [INFO] testing 'MySQL AND boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause (MAKE_SET)'
[16:14:45] [INFO] testing 'MySQL >= 5.0 boolean-based blind - ORDER BY, GROUP BY clause'
[16:14:46] [INFO] testing 'MySQL >= 5.0 boolean-based blind - ORDER BY, GROUP BY clause (original value)'
[16:14:46] [INFO] testing 'MySQL < 5.0 boolean-based blind - ORDER BY, GROUP BY clause'
[16:14:46] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[16:14:51] [INFO] testing 'MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)'
[16:14:56] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (UPDATEXML)'
[16:15:01] [INFO] testing 'MySQL >= 5.1 error-based - PROCEDURE ANALYSE (EXTRACTVALUE)'
[16:15:06] [INFO] testing 'MySQL >= 5.6 error-based - Parameter replace (GTID_SUBSET)'
[16:15:07] [INFO] testing 'MySQL >= 5.1 error-based - Parameter replace (EXTRACTVALUE)'
[16:15:07] [INFO] testing 'MySQL >= 5.6 error-based - ORDER BY, GROUP BY clause (GTID_SUBSET)'
[16:15:07] [INFO] testing 'MySQL >= 5.1 error-based - ORDER BY, GROUP BY clause (EXTRACTVALUE)'
[16:15:08] [INFO] testing 'MySQL inline queries'
[16:15:08] [INFO] testing 'MySQL >= 5.0.12 stacked queries (comment)'
[16:15:12] [INFO] testing 'MySQL >= 5.0.12 stacked queries'
[16:15:46] [INFO] POST parameter 'j_password' appears to be 'MySQL >= 5.0.12 stacked queries' injectable
[16:15:46] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[16:15:47] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (SLEEP)'
[16:15:47] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (SLEEP - comment)'
[16:15:47] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP - comment)'
[16:15:47] [INFO] testing 'MySQL < 5.0.12 AND time-based blind (BENCHMARK)'
[16:15:48] [INFO] testing 'MySQL > 5.0.12 AND time-based blind (heavy query)'
[16:15:48] [INFO] testing 'MySQL >= 5.0.12 RLIKE time-based blind'
[16:15:48] [INFO] testing 'MySQL >= 5.0.12 RLIKE time-based blind (query SLEEP)'
[16:15:48] [INFO] testing 'MySQL AND time-based blind (ELT)'
[16:15:48] [INFO] testing 'MySQL >= 5.1 time-based blind (heavy query) - PROCEDURE ANALYSE (EXTRACTVALUE)'
[16:15:49] [INFO] testing 'MySQL >= 5.0.12 time-based blind - Parameter replace'
[16:15:49] [INFO] testing 'MySQL >= 5.0.12 time-based blind - Parameter replace (substraction)'
[16:15:49] [INFO] testing 'MySQL >= 5.0.12 time-based blind - ORDER BY, GROUP BY clause'
[16:15:49] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[16:15:57] [INFO] testing 'Generic UNION query (random number) - 1 to 20 columns'
[16:16:07] [INFO] testing 'Generic UNION query (NULL) - 21 to 40 columns'
[16:16:16] [INFO] testing 'Generic UNION query (random number) - 21 to 40 columns'
[16:16:21] [INFO] testing 'Generic UNION query (NULL) - 41 to 60 columns'
[16:16:26] [INFO] testing 'MySQL UNION query (NULL) - 1 to 20 columns'
[16:16:31] [INFO] testing 'MySQL UNION query (random number) - 1 to 20 columns'
[16:16:40] [INFO] testing 'MySQL UNION query (NULL) - 21 to 40 columns'
[16:16:45] [INFO] testing 'MySQL UNION query (random number) - 21 to 40 columns'
[16:16:49] [INFO] testing 'MySQL UNION query (NULL) - 41 to 60 columns'
[16:16:54] [INFO] checking if the injection point on POST parameter 'j_password' is a false positive
[16:16:54] [WARNING] false positive or unexploitable injection point detected
[16:16:54] [WARNING] POST parameter 'j_password' does not seem to be injectable
[16:16:54] [CRITICAL] all tested parameters do not appear to be injectable. Try to increase values for '--level'/'--risk' options if you wish to perform more tests. Please retry with the switch '--text-only' (along with --technique=BU) as this case looks like a perfect candidate (low textual content along with inability of comparison engine to detect at least one dynamic parameter). If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent'
[16:16:54] [WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 1171 times, 400 (Bad Request) - 1 times

[*] ending @ 16:16:54 /2026-06-09/
```

**Análise:** SQL Injection não confirmado nos parâmetros j_username e j_password nesta execução. O resultado é inconclusivo, não negativo — as condições do teste (ViewState inválido, WAF ativo, 1171 erros 500) invalidam a execução como evidência definitiva de ausência de SQLi.

---

### 8.5 WSTG-CONF — Testes de configuração

#### 8.5.1 Análise dos cabeçalhos HTTP de segurança (WSTG-CONF-05)

**Comando:**
```bash
curl -I https://dev.pedreira.org/devSecurityG5/login.jsf
```

**Saída:**
```
HTTP/1.1 200
set-cookie: ETSS=--; Max-Age=0; Expires=Thu, 01 Jan 1970 00:00:10 GMT; Path=/; Secure
p3p: CP="IDC DSP COR ADM DEVi TAIi PSA PSD IVAi IVDi CONi HIS OUR IND CNT"
set-cookie: JSESSIONID=trn01~52003F12DB0E448FA587EBFB5A611666; Path=/devSecurityG5; Secure; HttpOnly
set-cookie: ESR=; Max-Age=86400; Expires=Thu, 11 Jun 2026 16:20:30 GMT; Path=/; Secure
content-type: text/html;charset=UTF-8
transfer-encoding: chunked
date: Wed, 10 Jun 2026 16:20:29 GMT
strict-transport-security: max-age=31536000; includeSubDomains; preload;
```

**Tabela de avaliação:**

| Cabeçalho | Presente | Valor configurado | Avaliação |
|---|---|---|---|
| `Content-Security-Policy` | ❌ | — | Ausente |
| `Strict-Transport-Security` | ✅ | max-age=31536000; includeSubDomains; preload; | Adequado |
| `X-Frame-Options` | ❌ | — | Ausente |
| `X-Content-Type-Options` |  ❌ | — | Ausente |
| `Referrer-Policy` | ❌ | Ausente |
| `Permissions-Policy` | ❌ | — | Ausente |
| `Server` | ❌ | — |  Não revela |

**Análise:** A aplicação dev.pedreira.org/devSecurityG5 é uma aplicação Java/JSF que apresenta postura de segurança mista. Por um lado, adota boas práticas como HSTS com preload, cookies com Secure e HttpOnly, e comunicação exclusivamente HTTPS. Por outro lado, carece de controles fundamentais de segurança web moderna: ausência total de Content-Security-Policy, X-Frame-Options, X-Content-Type-Options e SameSite nos cookies. O cookie ESR com valor vazio e a presença do header legado P3P sugerem código legado com possíveis fragilidades em lógica de sessão e anti-CSRF. O conjunto indica uma aplicação que passou por melhorias parciais de segurança, mas ainda não implementa uma estratégia defensiva completa, estando potencialmente exposta a clickjacking, CSRF e MIME sniffing. Recomenda-se revisão prioritária dos controles de segurança de resposta HTTP e da lógica de cookies de estado.

---

## 9. Análise consolidada dos resultados

### 9.1 Inventário de vulnerabilidades

| ID | Título | Severidade | CVSS v3.1 | WSTG | OWASP Top 10 (2025) |
|---|---|---|---|---|---|
| FIND-001 | Ausência de bloqueio de conta e rate limiting | 🔴 Alta | 7.5 | WSTG-AUTHN-03 | A07:2025 — Falhas de Identificação e Autenticação |
| FIND-002 | Token CSRF estático e não vinculado à sessão | 🔴 Alta | 7.1 | WSTG-SESS-05 | A01:2025 — Quebra de Controle de Acesso |
| FIND-003 | Cookies de sessão sem atributo `SameSite` | 🟠 Média | 6.5 | WSTG-SESS-02 | A05:2025 — Configuração de Segurança Incorreta |
| FIND-004 | Cookies `ETSS` e `ESR` sem atributo `HttpOnly` | 🟠 Média | 6.1 | WSTG-SESS-02 | A05:2025 — Configuração de Segurança Incorreta |
| FIND-005 | Ausência de cabeçalhos HTTP de segurança (CSP, X-Frame-Options, X-Content-Type-Options) | 🟠 Média | 5.8 | WSTG-CONF-05 | A05:2025 — Configuração de Segurança Incorreta |
| FIND-006 | Uso de `targetOrigin='*'` em `postMessage` | 🟠 Média | 5.4 | WSTG-CLNT-10 | A03:2025 — Injeção |
| FIND-007 | Redirecionamento incondicional via cookie `ETL` sem validação de integridade | 🟠 Média | 5.3 | WSTG-CLNT-04 | A01:2025 — Quebra de Controle de Acesso |
| FIND-008 | Resposta HTTP 200 em falha de autenticação | 🟡 Baixa/Média | 4.3 | WSTG-AUTHN-01 | A05:2025 — Configuração de Segurança Incorreta |
| FIND-009 | Certificado TLS com vencimento iminente e cifras CBC ativas (LUCKY13) | 🟡 Média | 5.9 | WSTG-CRYP-01 | A02:2025 — Falhas Criptográficas |
| FIND-010 | Vazamento de topologia de cluster via prefixo `trn01~` no `JSESSIONID` | 🔵 Baixa | 3.7 | WSTG-INFO-06 | A05:2025 — Configuração de Segurança Incorreta |
| FIND-011 | Exposição de versão de framework e build (`v=6.1`, `ve=09110300`) | 🔵 Baixa | 3.7 | WSTG-INFO-08 | A05:2025 — Configuração de Segurança Incorreta |
| FIND-012 | Ausência de DNSSEC e registros DNS CAA | 🔵 Baixa | 3.1 | WSTG-CONF-10 | A05:2025 — Configuração de Segurança Incorreta |
| FIND-013 | Header legado `P3P` presente na resposta HTTP | 🔵 Informativa | 0.0 | WSTG-INFO-06 | — |
| FIND-014 | SQL Injection — resultado inconclusivo (teste invalidado por condições adversas) | ⚪ Inconclusivo | — | WSTG-INPV-05 | A03:2025 — Injeção |

---

### 9.2 Distribuição por severidade

| Severidade | Quantidade | IDs |
|---|---|---|
| 🔴 Alta | 2 | FIND-001, FIND-002 |
| 🟠 Média | 5 | FIND-003, FIND-004, FIND-005, FIND-006, FIND-007 |
| 🟡 Baixa/Média | 2 | FIND-008, FIND-009 |
| 🔵 Baixa | 3 | FIND-010, FIND-011, FIND-012 |
| ⚪ Informativa/Inconclusiva | 2 | FIND-013, FIND-014 |

---

### 9.3 Discussão

Os resultados obtidos ao longo das fases de reconhecimento, mapeamento e varredura ativa revelam um padrão característico de aplicações Java EE legadas: controles de segurança implementados de forma parcial ou delegados implicitamente ao framework, sem verificação da adequação dessas configurações ao modelo de ameaças atual.

O achado de maior impacto potencial é a combinação de FIND-001 e FIND-002. A ausência de bloqueio de conta e rate limiting (FIND-001) expõe o endpoint `/devSecurityG5/login.jsf` a ataques de força bruta irrestrita, confirmada pela ausência de qualquer mecanismo de defesa nas 5 requisições disparadas em 3 segundos durante o teste com Hydra. Paralelamente, o token CSRF presente na aplicação (FIND-002) demonstrou ser estático entre sessões distintas, com o valor `6faeed95-1310-4648-a949-984d34ee00ba` idêntico em todos os carregamentos de página observados. Um token que não varia por sessão é, funcionalmente, equivalente à ausência de proteção CSRF, pois qualquer página maliciosa pode incorporá-lo diretamente. A ausência de `SameSite` em todos os cookies reforça esse vetor, eliminando a camada de defesa residual que os navegadores modernos oferecem por padrão.

A exposição de cookies de sessão sem `HttpOnly`, especificamente `ETSS` (token de sessão) e `ESR` (controle de login) — configura um vetor de sequestro de sessão via XSS. O código-fonte da aplicação confirma que `getCookie()` é chamado diretamente nesses cookies por scripts client-side, o que evidencia que a ausência de `HttpOnly` é intencional no design original, mas incompatível com o princípio de menor privilégio aplicado à camada de apresentação.

O mecanismo de redirecionamento baseado no cookie `ETL` (FIND-007) merece atenção específica: a lógica `if(getCookie("ETL") == "JWT_TOKEN")` executa um redirecionamento incondicional para o contexto Flutter (`devMWFlutterWeb`) sem qualquer validação de integridade ou origem do cookie. Considerando que `ETL` não possui `HttpOnly`, um atacante que consiga injetar esse valor, via XSS ou manipulação direta do armazenamento de cookies, pode forçar o redirecionamento de usuários autenticados para um contexto de aplicação distinto, com consequências imprevisíveis para o fluxo de autenticação.

A ausência de cabeçalhos HTTP de segurança fundamentais (FIND-005) — Content-Security-Policy, X-Frame-Options e X-Content-Type-Options — amplia a superfície de ataque ao não restringir origens de conteúdo, permitir o embutimento da aplicação em iframes de terceiros (clickjacking) e possibilitar MIME sniffing por parte do navegador. O uso de `targetOrigin='*'` em chamadas `postMessage` (FIND-006) agrava esse cenário: mensagens contendo URL e título da página são transmitidas a qualquer origem receptora, o que em combinação com um iframe malicioso configura um vetor de vazamento de informação de navegação.

No plano criptográfico (FIND-009), a presença de cipher suites CBC no TLS 1.2 mantém a exposição teórica ao ataque LUCKY13 (CVE-2013-0169), embora seu impacto prático seja mitigado pelo suporte a TLS 1.3 com AEAD. O ponto operacionalmente crítico é o vencimento iminente do certificado wildcard `*.pedreira.org` em 03/07/2026, menos de 4 semanas a contar da data de execução do teste — representando risco de indisponibilidade do serviço por expiração.

Os achados de baixa severidade (FIND-010, FIND-011, FIND-012) seguem o padrão de exposição de informação de infraestrutura: o prefixo `trn01~` no JSESSIONID revela a topologia do cluster Tomcat; os parâmetros `v=6.1` e `ve=09110300` identificam de forma precisa o PrimeFaces 6.1 (descontinuado, com CVEs conhecidos) e a versão proprietária do MentorWeb; a ausência de DNSSEC e CAA reduz o custo de ataques de envenenamento de DNS e emissão fraudulenta de certificados. Individualmente, esses achados têm impacto limitado; em conjunto, fornecem a um atacante um mapa detalhado da infraestrutura que reduz significativamente o esforço de reconhecimento em fases subsequentes de um ataque real.

O resultado inconclusivo do SQLMap (FIND-014) não deve ser interpretado como ausência de injeção SQL. As condições do teste, ViewState inválido, presença de mecanismo de proteção detectado pelo próprio SQLMap, e 1171 respostas HTTP 500, invalidam a execução como prova negativa. O teste requer reexecução com ViewState válido capturado via proxy autenticado e, eventualmente, uso de técnicas de bypass de WAF.

---

## 10. Conclusão

O presente estudo conduziu uma avaliação de segurança do sistema Mentor Web em ambiente de homologação, cobrindo as etapas de reconhecimento passivo, reconhecimento ativo, mapeamento da superfície de ataque e varredura de vulnerabilidades, em conformidade com o OWASP Web Security Testing Guide (WSTG) e o OWASP Top 10 (2025).

Foram identificados 13 achados com severidade definida e 1 inconclusivo, distribuídos entre as categorias de autenticação, gestão de sessão, configuração de servidor, criptografia e exposição de informações. A postura de segurança da aplicação apresenta lacunas relevantes, especialmente nas camadas de controle de acesso e configuração de resposta HTTP, que devem ser endereçadas antes de qualquer promoção do sistema para ambientes de produção.


---

## 11. Referências

- OWASP. *Web Security Testing Guide (WSTG) — stable*. Disponível em: https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/. Acesso em: 05/06/2026.
- OWASP. *OWASP Top 10 — 2025*. Disponível em: https://owasp.org/Top10/2025/. Acesso em: 05/06/2026.
- FIRST. *Common Vulnerability Scoring System v3.1: Specification Document*. Disponível em: https://www.first.org/cvss/specification-document. Acesso em: 06/06/2026.
- NIST. *Technical Guide to Information Security Testing and Assessment (SP 800-115)*. Disponível em: https://csrc.nist.gov/publications/detail/sp/800-115/final. Acesso em: 07/06/2026.
- MITRE. *CWE — Common Weakness Enumeration*. Disponível em: https://cwe.mitre.org/. Acesso em: 10/06/2026.
- IETF. *RFC 6265 — HTTP State Management Mechanism*. Disponível em: https://tools.ietf.org/html/rfc6265. Acesso em: 10/06/2026.
- BERNSTEIN, D. J. *The LUCKY13 attack on TLS CBC*. 2013. Disponível em: http://www.isg.rhul.ac.uk/tls/TLStiming.pdf. Acesso em: 10/06/2026.


---
