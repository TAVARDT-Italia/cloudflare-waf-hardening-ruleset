# Cloudflare WAF Hardening Ruleset e Blueprint di Sicurezza di Bordo

[![Cloudflare WAF](https://img.shields.io/badge/Cloudflare-WAF%20Hardened-F38020.svg?logo=cloudflare)](https://cloudflare.com)
[![Diagnostics Suite](https://img.shields.io/badge/TAVARDT-Diagnostics-5b3cf5.svg)](https://tools.tavardt.com/it/analisi-header-sicurezza/)
[![Licence MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

***

Insieme di regole personalizzate per Cloudflare WAF, mitigazione dei bot e blueprint di sicurezza di bordo realizzato da **TAVARDT**, in piena conformità con le Misure Minime di Sicurezza ICT di AgID e il GDPR europeo.

> [!IMPORTANT]
> **Architettura Perimetrale Universale e Agnóstica:** Questo insieme di regole non è limitato a WordPress né ad architetture monolitiche. È stato strutturato per proteggere API RESTful, moderni microservizi, frontend SPA (Next.js, React, Vue) e backend di produzione (Node.js, Python, Go, Laravel, Java). Il blocco tempestivo delle scansioni rivolte a CMS obsoleti funge da igiene perimetrale attiva, arrestando il traffico malevolo direttamente sui data center periferici di Cloudflare in meno di 1ms prima che impatti server e risorse di origine.

***

## Funzionalità Principali e Architettura Difensiva

* **Isolamento Perimetrale delle Credenziali**: Blocca all'istante sulla rete perimetrale l'accesso a file `.env`, repository `.git`, dump `.sql`, `config.json`, `package.json` e `composer.json` prima che raggiungano l'origine.
* **Soppressione dei Tool di Ricognizione**: Intercetta scanner automatici di vulnerabilità (Nikto, SQLMap, Acunetix, Masscan, Zgrab, Nmap, DirBuster, Gobuster) sul perimetro Cloudflare.
* **Igiene Perimetrale Agnóstica dello Stack**: Neutralizza vettori di abuso diretti a CMS legacy (`/xmlrpc.php`, trackback) in meno di 1ms, proteggendo socket TCP, memoria e CPU del server di origine.
* **Sanificazione dei Metodi HTTP**: Rifiuta verbi di debug non standard o pericolosi (`TRACE`, `TRACK`, `DEBUG`) direttamente alla barriera del firewall perimetrale.
* **Sfida Adattiva per Endpoint Amministrativi**: Attiva Sfide Gestite di Cloudflare su percorsi sensibili (`/admin`, `/wp-login.php`, `/administrator`), consentendo il passaggio del traffico autorizzato da ASN interno.

***

## Vettori di Rischio e Impatto Operativo

Pubblicare servizi web e API pubbliche senza controlli perimetrali dichiarativi espone i server di origine a continue scansioni ostili e al consumo incontrollato di risorse. Il ruleset neutralizza cinque vettori principali sulla frontiera CDN:

* **Esposizione di File Sensibili e Credenziali (TAVARDT-WAF-01)**: Sfruttamento di configurazioni errate sui server di origine per scaricare variabili d'ambiente, repository Git e dump di database.
* **Scansioni Automatizzate di Vulnerabilità (TAVARDT-WAF-02)**: Sondaggi intensivi di scanner automatici che esauriscono la banda passante, appesantiscono la CPU e ricercano falle note.
* **Attacchi di Amplificazione e Brute Force su CMS (TAVARDT-WAF-03 e TAVARDT-WAF-05)**: Botnet che tentano attacchi DDoS tramite pingback XML-RPC o attacchi di forza bruta su pagine di autenticazione amministrativa.
* **Abuso del Protocollo tramite Verbi HTTP Non Sicuri (TAVARDT-WAF-04)**: Attori malevoli che impiegano metodi obsoleti per aggirare le cache, tentare cross-site tracing (XST) o forzare funzioni di debug.
* **Saturazione delle Risorse e Blocco del Server di Origine**: Traffico malevolo non filtrato che consuma i processi applicativi, causando rallentamenti e indisponibilità per gli utenti legittimi.

***

## Distribuzione e Automazione Perimetrale

Configurate il ruleset di protezione impiegando le API v4 dichiarative di Cloudflare oppure tramite infrastructure-as-code con Terraform:

### Opzione A: Importazione Diretta tramite Cloudflare API v4

```bash
# Applicare le regole personalizzate di firewall tramite Cloudflare API v4
curl -X PUT "https://api.cloudflare.com/client/v4/zones/YOUR_ZONE_ID/rulesets/phases/http_request_firewall_custom/entrypoint" \
  -H "Authorization: Bearer YOUR_CLOUDFLARE_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d @rules/ruleset.json
```

### Opzione B: Provisioning Dichiarativo con Terraform

```bash
# Accedere alla directory del modulo Terraform
cd terraform

# Inizializzare il provider Cloudflare
terraform init

# Pianificare la configurazione dichiarativa delle regole
terraform plan -var="zone_id=YOUR_ZONE_ID"

# Applicare la configurazione di firewall sulla rete perimetrale
terraform apply -var="zone_id=YOUR_ZONE_ID"
```

***

## Domande Tecniche Frequenti

### Quali vettori di vulnerabilità vengono neutralizzati dal ruleset Cloudflare WAF sulla rete perimetrale?

Il ruleset blocca scanner di ricognizione automatizzati, tentativi di esposizione di file sensibili e variabili d'ambiente, abusi del protocollo XML-RPC e metodi HTTP di debug direttamente sui nodi Cloudflare. Questo filtraggio preventivo arresta il traffico ostile in meno di 1 millisecondo, salvaguardando CPU, memoria e connessioni al database del server di origine.

### Perché questo modello di protezione perimetrale è indicato per API REST e microservizi moderni oltre che per CMS?

Gli scanner di exploit automatizzati scansionano gli endpoint pubblici indiscriminatamente, sia che il servizio sia sviluppato in Node.js, Go, Python o Next.js. Intercettare sul perimetro richieste malevole e tentativi di accesso a file riservati garantisce un ambiente pulito e privo di rumore computazionale.

### In che modo il ruleset previene la fuga accidentale di credenziali e dump di database?

La regola TAVARDT-WAF-01 genera un blocco immediato HTTP 403 per qualsiasi richiesta contenente percorsi come `.env`, `.git`, `.sql`, `composer.json` o `package.json`. Questa protezione deterministica impedisce che errori di configurazione del web server espongano token privati e strutture dati riservate.

### Come opera la protezione delle aree amministrative senza compromettere i servizi interni autorizzati?

La regola TAVARDT-WAF-05 applica Sfide Gestite di Cloudflare ai tentativi di accesso a `/wp-login.php`, `/admin` e `/administrator`. Include un'eccezione esplicita per l'ASN 13335 di Cloudflare, consentendo alle routine interne autorizzate e ai sistemi di monitoraggio di operare senza ostacoli.

### Quali sono le differenze operative tra la distribuzione tramite API v4 di Cloudflare e Terraform?

L'importazione di `rules/ruleset.json` tramite API v4 assicura un'implementazione immediata senza dipendenze software locali nella fase `http_request_firewall_custom`. Il modulo `terraform/main.tf` permette invece la gestione della sicurezza come codice, il tracciamento dei cambiamenti e la replicabilità su zone multiple di produzione.

***

## Link Utili

* **Portale Aziendale:** https://ag.tavardt.com/it/
* **Suite di Diagnostica:** https://tools.tavardt.com/it/
* **Contatto Tecnico:** https://ag.tavardt.com/it/contatto/

***

### Licenza
Distribuito sotto la Licenza MIT. Consultare il file [LICENSE](LICENSE) per tutti i dettagli.

***

© 2026 TAVARDT. Tutti i diritti riservati.
