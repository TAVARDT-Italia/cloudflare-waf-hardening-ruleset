# Cloudflare WAF Hardening Ruleset & Guida alla Sicurezza Edge | TAVARDT Italia

[![Licence MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Cloudflare WAF](https://img.shields.io/badge/Cloudflare-WAF%20Hardened-F38020.svg?logo=cloudflare)](https://cloudflare.com)
[![Language](https://img.shields.io/badge/Language-Italiano-green.svg)](README.md)
[![Diagnostics Suite](https://img.shields.io/badge/TAVARDT-Diagnostics-5b3cf5.svg)](https://tools.tavardt.com/it/analisi-header-sicurezza/)

Regole avanzate di protezione WAF e sicurezza perimetrale per Cloudflare, conformi ai requisiti del RGPD e alle misure minime di sicurezza dell'ACN.

Sviluppato e gestito dal team di ingegneria della sicurezza perimetrale di **[TAVARDT Italia](https://ag.tavardt.com/it/sicurezza-informatica/)**.

> [!IMPORTANT]
> **Architettura Universale e Agnostica:** Questo repository **non è limitato a WordPress**. È stato concepito per proteggere **API RESTful, microservizi moderni, frontend (Next.js, React, Vue) e backend aziendali (Node.js, Python, Go, Laravel, Java)**. Il blocco delle sonde CMS agisce come un **livello di igiene perimetrale attiva**, scartando il traffico cieco delle botnet direttamente sulla rete periferica di Cloudflare (< 1ms) prima che possa saturare connessioni TCP, memoria o CPU sui server di origine.

---

### Matrice di Protezione Multi-Stack

| Livello di Difesa | Vettori Neutralizzati | Applicazioni e Stack Protetti |
| :--- | :--- | :--- |
| **Isolamento delle Credenziali** | Blocco immediato di `.env`, `.git`, `.sql`, `package.json`, `composer.json` | **API, Next.js, Node.js, Python, Go, Laravel, React** |
| **Mitigazione di Scanner** | Blocco di Nikto, SQLMap, Acunetix, Masscan, Gobuster, Nmap | **Qualsiasi applicazione web, microservizio o gateway API** |
| **Igiene dei Metodi HTTP** | Rifiuto dei verbi di debug non sicuri (`TRACE`, `TRACK`, `DEBUG`) | **API RESTful e microservizi di produzione** |
| **Controllo degli Accessi** | Challenge gestito sulle rotte di autenticazione (`/admin`, `/login`) | **Pannelli di amministrazione, dashboard ed ERP web** |
| **Igiene del Traffico Cieco** | Scarto perimetrale delle richieste verso `/xmlrpc.php` e trackback | **Azzeramento del carico inutile sui server di produzione** |

---

## 1. Panoramica dell'Architettura di Sicurezza Perimetrale

Questo repository fornisce regole dichiarative pronte per l'applicazione a livello perimetrale su Cloudflare, per mitigare attacchi mirati e prevenire la fuga di dati sensibili:

* **Isolamento di File Riservati e Configurazioni:** Blocco perimetrale istantaneo per richieste verso file `.env`, directory `.git`, dump di database `.sql` e manifest di dipendenze (`composer.json`, `package.json`).
* **Mitigazione di Scanner e Sonde Automatizzate:** Rilevamento e blocco in tempo reale dei principali scanner di vulnerabilità (Nikto, SQLMap, Acunetix, Masscan, Nmap, Gobuster, DirBuster).
* **Protezione Perimetrale dei Sistemi CMS:** Blocco immediato dei vettori di abuso su `/xmlrpc.php` e `/wp-trackback.php`.
* **Rifiuto dei Metodi HTTP Pericolosi:** Intercettazione dei verbi di debug non sicuri (`TRACE`, `TRACK`, `DEBUG`) direttamente alla frontiera CDN.
* **Challenge Gestito per Endpoint Amministrativi:** Protezione ad attivazione automatica sulle schermate di accesso riservate (`/wp-login.php`, `/admin`).

---

## 2. Diagnosi Preventiva di Sicurezza

Prima dell'applicazione nei sistemi di produzione, verificate la corretta configurazione degli header HTTP e la resilienza del dominio con gli strumenti TAVARDT:

* **Audit Header di Sicurezza e Certificati SSL:** [Analizzatore Header di Sicurezza TAVARDT](https://tools.tavardt.com/it/analisi-header-sicurezza/)
* **Suite Completa di Diagnostica:** [TAVARDT Tools Italia](https://tools.tavardt.com/it/)

---

## 3. Guida all'Implementazione Rapida

### Opzione A: Distribuzione Diretta tramite API Cloudflare v4

Utilizzate la definizione dichiarativa in [`rules/ruleset.json`](rules/ruleset.json):

```bash
curl -X PUT "https://api.cloudflare.com/client/v4/zones/IL_TUO_ZONE_ID/rulesets/phases/http_request_firewall_custom/entrypoint" \
     -H "Authorization: Bearer IL_TUO_CLOUDFLARE_API_TOKEN" \
     -H "Content-Type: application/json" \
     --data @rules/ruleset.json
```

### Opzione B: Automazione tramite Terraform

Utilizzate il file IaC posizionato in [`terraform/main.tf`](terraform/main.tf):

```bash
cd terraform
terraform init
terraform plan -var="zone_id=IL_TUO_ZONE_ID"
terraform apply -var="zone_id=IL_TUO_ZONE_ID"
```

---

## 4. Consulenza Specialistica e Supporto SOC Continuo

Se la vostra organizzazione necessita di audit approfonditi, conformità alle normative europee (RGPD / NIS 2) o protezione dedicata per agenti autonomi di IA:

* **Portale Aziendale:** [TAVARDT Italia](https://ag.tavardt.com/it/sicurezza-informatica/)
* **Canale di Contatto Tecnico:** [Richiedi Supporto Ingegneristico](https://ag.tavardt.com/it/contatti/)

---

## Licenza

Distribuito sotto licenza open-source MIT. Consultare il file [`LICENSE`](LICENSE) per ulteriori dettagli.  
Copyright (c) 2026 TAVARDT Italia.
