# Snapped Phish-ing Line

**Data:** 01-10-2026
**Piattaforma:** TryHackMe
**Categoria:** Phishing
**Difficoltà:** Easy

---

## Contesto

In qualità di membro del reparto IT di SwiftSpend Financial, sei responsabile dell'assistenza ai dipendenti per problemi tecnici. Quella che inizialmente sembrava una giornata di routine si è rapidamente trasformata in un'emergenza quando diversi dipendenti di vari reparti hanno segnalato di aver ricevuto un'e-mail sospetta. Diversi utenti hanno notato caratteristiche insolite nel messaggio e, purtroppo, alcuni avevano già inserito le proprie credenziali e non sono più in grado di accedere ai propri account. Data la potenziale compromissione di un sistema più ampio, l'incidente è stato segnalato per un'indagine.

L'obbiettivo è analizzare le prove disponibili, determinare la portata dell'attacco e scoprire come ha operato l'aggressore.

---

## Obiettivi

* Analizza i campioni di email forniti per identificare gli elementi chiave.
* Indagare sugli URL di phishing per comprendere il reindirizzamento
* Recuperare ed esaminare il kit di phishing utilizzato nell'attacco
* Utilizzo di strumenti CTI per raccogliere informazioni sull'avversario
* Analizza il kit di phishing per scoprire ulteriori indicatori

---

## Strumenti utilizzati

- **VirusTotal** — analisi dell'hash SHA256 dell'allegato
- **sha256sum** (Linux) — calcolo dell'hash dell'allegato in sandbox
- **DeadLink-SOC** — tool personale in Python per defangare link, IP, domini ed email
- **Talos Intelligence** — reputazione di IP e domini

---

## Analisi

### Ispezione delle email

 Le email mandate agli utenti erano le stesse il solito pattern dell'ingengeria sociale, in questo caso ognua di esse avevano un allegato in formato html il quale una volta aperto portava ad una pagina fittizia di login. In questo caso la pagina era un clone di accesso Microsoft, navigando tra i path del sito fittizzio mi sono imbattuto nei log che raccoglieva dalle interazioni con il login, esso conteneva tentavi di accesso che raccoglievano :

- informazioni host
- username
- email

diversi utenti hanno tentato di fare l'accesso più volte ignari della situazione. Tramite i path del sito ho scaricato la cartella che ne conteneva la struttura risalendo all'indirizzo email al quale venivano inviati i log.txt contenteti credenziali d'accesso dei dipendenti dell'azienda.

---

## Passaggi

**1. Ispezione dell'email**
Ho aperto l'inspector dell'email per accedere alle fonti principali: email mittente (verifica dello spoofing), dominio, indirizzo IP di origine, informazioni sull'allegato e codice HTML per visualizzare la presenza di link.

**2. Analisi degli indicatori**
Tutte le email contenevano un file `.html` che reindirizzava al dominio dell'attaccante.

**3. Analisi dell'allegato**
Ho scaricato l'allegato in una sandbox per isolarmi dall'ambiente ed evitare la propagazione delle minacce. Ho eseguito il comando `sha256sum` su una macchina Linux per ottenere l'hash, che ho poi portato su VirusTotal. Il file risultava essere classificato come Trojan, con uno score relativamente basso.

**4. Analisi del Phishing Kit**

Ho estratto dal sito, tramite la navigazione dei percorsi in `/data`, il Phishing Kit che conteneva la struttura e il codice sorgente. Ispezionandolo ho trovato l'email sorgente dove venivano reindirizzati i log con gli accessi degli utenti.

**6. Conclusioni**
La conclusione principale dell'indagine è un esito positivo al phishing: l'intento dell'attore era ottenere una via d'accesso al sistema tramite gli accessi degli utenti che hanno eseguito il login sul portale fittizio di Microsoft.

- Stessa mail a più utenti
- Stesso allegato a tutti gli utenti
- Social engineering
- reinderizzamento ad un portale che impersonificava un vendor fidato in questo caso Mircosoft
- Esiti delle analisi tramite tool: VirusTotal

---

## IOC trovati

| Tipo           | Valore (defangato)                                               | Note                                                        |
| :------------- | :--------------------------------------------------------------- | :---------------------------------------------------------- |
| File Zip       | hxxps[://]kennaroads/login/data                                  | Kit di phishing                                             |
| email          | Accounts[.]Payable@groupmarketingonline[.]icu                    | fittizzia azienda di marketing                              |
| Hash SHA256    | ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686 | Trojan, Score 30/60                                         |
| dominio        | kennaroads[.]buzz                                                | Basso rate su Talos Intelligence                            |
| email sorgente | m3npat[@]yandex[.]com                                            | scovata tra i file del percosrso /data del portale fittizio |

---

## Verdetto

**True Positive** — Email di phishing con allegato malevolo (trojan). L'attaccante ha usato tecniche di ingegneria sociale, login fraudolenti per ottenere accessi alla rete interna dell'azienda. IOC estratti e verificati tramite Talos Intelligence, VirusTotal e navigazione nei path del portale.

---

## Query utilizzate

Nessuna query SPL utilizzata in questa analisi. Comando usato per il calcolo dell'hash:

```bash
hxxps[://]kennaroads/login/data

Riferimenti

 - TryHackMe — Snapped Phish-ing Line

 - VirusTotal

 - Talos Intelligence

 - CLI Linux
```

## Cosa ho imparato

Ho imparato a navigare nei path di un sito di phishing per recuperare il kit e l'email sorgente. Ho capito che l'analisi non si ferma all'email, ma si estende all'infrastruttura dell'attaccante.