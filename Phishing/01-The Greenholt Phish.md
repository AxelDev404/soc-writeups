# The Greenholt Phish

**Data:** 01-10-2026
**Piattaforma:** TryHackMe
**Categoria:** Phishing
**Difficoltà:** Easy

---

## Contesto

Un responsabile vendite presso Greenholt PLC ha segnalato un'email sospetta ricevuta da un cliente noto. Il messaggio ha sollevato diversi segnali di allarme: un saluto generico, una richiesta inaspettata di trasferimento di denaro e un allegato non richiesto. Secondo il dipendente, questo comportamento non è in linea con il solito stile di comunicazione del cliente. Temendo che l'email possa essere dannosa, il messaggio è stato inoltrato al SOC (Centro Operativo di Sicurezza) per ulteriori indagini.

L'obiettivo è analizzare il campione di email fornito e determinare se è legittimo o parte di un tentativo di phishing.

---

## Obiettivi

- Analizzare l'email fornita per identificare ed estrarre gli elementi chiave.
- Indagare sulla fonte del messaggio per determinarne l'origine e l'autenticità.
- Utilizzare strumenti di analisi per valutare la potenziale natura dannosa dell'email.

---

## Strumenti utilizzati

- **Talos Intelligence** — reputazione di IP e domini
- **EasyDMARC** — analisi dei record SPF e DMARC
- **VirusTotal** — analisi dell'hash SHA256 dell'allegato
- **sha256sum** (Linux) — calcolo dell'hash dell'allegato in sandbox
- **DeadLink-SOC** — tool personale in Python per defangare link, IP, domini ed email

---

## Analisi

### Ispezione dell'email

Ispezionando la mail, in prima fase sembra usare una tecnica di ingegneria sociale sfruttando l'impatto emotivo della vittima, facendosi passare per un suo cliente. La mail tratta una richiesta di trasferimento di denaro avvenuta con successo pari a una somma di 149.650$. Dall'analisi si può notare l'uso di:

- **Email spoofed** — il dominio originale del mittente non corrisponde con quello mostrato
- **Indirizzo IP con reputazione bassa**
- **Allegato sospetto** — `application/octet-stream; name="SWT_#09674321____PDF__.CAB"`

### Analisi SPF e DMARC

Ulteriori analisi sono state condotte sul record SPF del dominio Return-Path identificato nell'intestazione dell'email, e sul DMARC:

v=spf1 include:spf.protection.outlook.com -all

Dal DMARC risulta che le policy specificate nei record mettono in quarantena i messaggi provenienti da questa email:


### Ispezione dell'allegato

Ho scaricato l'allegato in una sandbox ed eseguito il comando per calcolare l'hash, poi l'ho portato su VirusTotal per ulteriori indagini.

**Hash SHA256:**

2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f



**Esito VirusTotal:**
- 49/64 security vendor hanno flaggato il file come malevolo
- `trojan.msil/loki`
- Classificato nelle categorie **trojan-ransomware**

---

## Passaggi

**1. Ispezione dell'email**
Ho aperto l'inspector dell'email per accedere alle fonti principali: email mittente (verifica dello spoofing), dominio, indirizzo IP di origine, informazioni sull'allegato e codice HTML per visualizzare la presenza di link (sia normali che accorciati).

**2. Analisi degli indicatori**
Partendo dalla base, ho controllato l'oggetto in quanto è il cuore dell'ingegneria sociale: ciò che è emerso risultava essere un messaggio d'impatto. Ma non solo l'oggetto conteneva la tecnica citata, bensì tutto il corpo del messaggio. In seguito ho controllato la reputazione dell'indirizzo IP e del dominio, ottenendo anche le informazioni sul proprietario del dominio (HOSTPAPA). Tutti questi indicatori erano sufficienti per trarre le conclusioni, ma mancavano altri pezzi, quindi ho deciso di proseguire con un'ulteriore analisi su SPF e DMARC, i quali non davano un buon risultato: in DMARC il dominio era flaggato in quarantena (SPAM).

**3. Analisi dell'allegato**
Ho scaricato l'allegato in una sandbox per isolarmi dall'ambiente ed evitare la propagazione della minaccia nel sistema. Mi sono limitato all'esecuzione del comando `sha256sum` su una macchina Linux per ottenere l'hash, che ho poi portato su VirusTotal. Il file risultava essere nella categoria Trojan e Ransomware, con uno score molto alto.

**4. Conclusioni**
La conclusione principale dell'indagine è un esito positivo al phishing: l'intento dell'attore era ottenere una via d'accesso al sistema tramite l'esecuzione del trojan allegato. La conclusione si basa su tutti gli indicatori presenti nell'indagine:

- IP con reputazione bassa
- Dominio con reputazione bassa
- Social engineering
- Allegato con estensione sconosciuta al browser
- Esiti delle analisi tramite tool: VirusTotal, EasyDMARC, Talos Intelligence

---

## IOC trovati

| Tipo | Valore (defangato) | Note |
| :--- | :--- | :--- |
| Dominio | `[defangato]` | Registrato su HOSTPAPA, reputazione bassa |
| IP | `[defangato]` | Reputazione bassa |
| Hash SHA256 | `2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f` | Trojan/Ransomware |
| Allegato | `SWT_#09674321____PDF__.CAB` | `application/octet-stream` |

---

## Verdetto

**True Positive** — Email di phishing con allegato malevolo (trojan/ransomware). L'attaccante ha usato tecniche di ingegneria sociale per indurre la vittima ad aprire l'allegato. IOC estratti e verificati tramite Talos Intelligence, EasyDMARC e VirusTotal.

---

## Query utilizzate

Nessuna query SPL utilizzata in questa analisi. Comando usato per il calcolo dell'hash:
```bash
sha256sum SWT_#09674321____PDF__.CAB

Riferimenti

 - TryHackMe — The Greenholt Phish

 - VirusTotal

 - Talos Intelligence

 - EasyDMARC


 ## Pattern appresi

- Social engineering tramite richiesta finanziaria inattesa
- Spoofing del mittente
- Discrepanza tra nome apparente e vera estensione dell'allegato
- IP/domain reputation come indicatori di supporto
- Hash come IOC per identificare un campione
- Correlazione di più indicatori prima del verdict
