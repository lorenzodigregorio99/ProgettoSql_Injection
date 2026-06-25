# Linux Lab: SQL Injection (Data Exfiltration) & Infrastructure Hardening

Questo repository contiene un progetto di laboratorio orientato alla cybersecurity e alla gestione infrastrutturale. Lo scenario simula un attacco di SQL Injection (SQLi) mirato all'estrazione di dati sensibili da un database, seguito dall'implementazione di contromisure sistemistiche su piattaforma Linux.

⚠️ **Disclaimer:** *Questo progetto è stato realizzato esclusivamente a scopo didattico in un ambiente virtuale isolato e controllato.*

---

## 📐 Architettura di Rete e Infrastruttura

L'ambiente è stato interamente virtualizzato su **VMware** ed è composto da due server Linux Ubuntu configurati con **IP statici sulla stessa sottorete** per garantire la comunicazione diretta:


*   **Server Target (Ubuntu Server + XAMPP):** Funge da Web Server dell'infrastruttura. Ospita il servizio Apache, il DBMS MySQL e l'applicazione web di autenticazione (`login.php`).
*   **Client Attaccante (Ubuntu Server):** Macchina remota utilizzata per testare la sicurezza del Web Server e sferrare l'attacco attraverso la rete locale.

---

## 🔓 Analisi della Vulnerabilità e Tecniche di Attacco

La pagina `login.php` sul server target accettava l'input dell'utente (il campo email/username) e lo passava direttamente al database MySQL senza alcuna pulizia, validazione o sanitizzazione dei parametri. Nel laboratorio sono state testate due metodologie di attacco:

### 1. Attacco UNION-based (Data Exfiltration)
Per estrarre i dati tramite l'operatore `UNION`, l'iniezione deve presentare lo stesso numero esatto di colonne della query `SELECT` originale.
*   **Riconoscimento delle colonne (`ORDER BY`):** Sono stati effettuati dei tentativi incrementali iniettando la clausola `ORDER BY 1`, `ORDER BY 2`, `ORDER BY 3`, ecc. Incrementando il numero progressivamente, l'applicazione risponde correttamente fino a quando il valore non supera il numero reale di colonne presenti nella tabella, mandando il sistema in errore. Identificato così il numero esatto di campi, è stata costruita la `UNION SELECT`.
*   <img width="879" height="663" alt="image" src="https://github.com/user-attachments/assets/d7cb09c6-a8de-4e8c-b826-440cba3362b5" />

*   <img width="1165" height="747" alt="image" src="https://github.com/user-attachments/assets/4ac6f24b-272d-4b95-8cf0-6cafa5b3df26" />

*   **Risultato:** L'iniezione ha forzato il database a stampare a schermo informazioni strutturali cruciali: il **nome esatto del database**, l'identità dell'utente di sistema (scoprendo che il servizio girava pericolosamente come **`root`**) e l'elenco completo di **username e hash delle password** degli utenti.
*   <img width="1191" height="780" alt="image" src="https://github.com/user-attachments/assets/2a79804a-3f04-4bb2-88cd-1a104f0e42c9" />
*   <img width="1189" height="769" alt="image" src="https://github.com/user-attachments/assets/ba54f281-c84b-49db-bbe7-3dffba98a9fe" />




### 2. Attacco Blind SQL Injection (Time-Based)
Nello scenario in cui l'applicazione fosse stata configurata per non stampare a video gli errori o l'output dei dati (rendendo l'attacco *invisibile* o *cieco*), è stata testata la tecnica della **Blind SQL Injection basata sul tempo**.
*   **Verifica della vulnerabilità:** È stato iniettato il comando `SLEEP(5)`. Notando che il server impiegava esattamente 5 secondi in più a rispondere, è stata confermata la presenza della falla.
*   **Estrazione dei dati alla cieca:** Per indovinare le informazioni senza vederle, si interroga il database ponendo domande binarie (Vero/Falso). Ad esempio, iniettando una logica condizionale del tipo: *"Se la password dell'amministratore inizia con la lettera 'A', allora vai in `SLEEP(5)`"*. Se il server ritarda la risposta, la condizione è vera, permettendo di ricostruire stringhe complesse carattere per carattere semplicemente misurando i tempi di risposta del server.

---

## 🛡️ Approccio Sistemistico e Contromisure (Hardening)

Se da un lato la risoluzione definitiva del problema richiede la correzione del codice PHP (tramite *Prepared Statements*), dall'altro il compito di un **sistemista** è limitare i privilegi e contenere i danni qualora un'applicazione web venga compromessa. 

Nel laboratorio sono state analizzate e applicate le seguenti contromisure infrastrutturali:

### 1. Hardening dei Privilegi del Database (Eliminazione dell'utente Root)
L'attacco ha rivelato che l'applicazione web comunicava con il database usando l'utente `root`. Questa è una grave vulnerabilità di configurazione sistemistica.
*   **Soluzione applicata:** È stato rimosso l'accesso a `root` per l'applicazione web. È stato configurato un utente MySQL dedicato e non privilegiato (`web_app_user`), limitando i suoi permessi alle sole tabelle necessarie e inibendo funzioni di sistema pericolose.

### 2. Configurazione del Firewall per la Sicurezza del DBMS
Il DBMS non deve mai essere esposto sulla rete, specialmente se contiene dati sensibili.
*   **Soluzione applicata:** Configurazione del firewall di Ubuntu (`UFW`) per bloccare qualsiasi tentativo di connessione esterna sulla porta standard di MySQL (`3306`), forzando il database ad accettare connessioni esclusivamente in locale (`127.0.0.1`) provenienti dal web server Apache locale.

### 3. Hardening di PHP e Gestione dei Log (`php.ini`)
Per impedire che gli errori del database stampassero a schermo informazioni utili all'attaccante (come i nomi delle colonne scoperti con l'ORDER BY):
*   Disattivazione della direttiva `display_errors = Off` nel file `php.ini`.
*   Attivazione del logging centralizzato (`log_errors = On`) per deviare tutti gli errori del database all'interno dei file di log di sistema (`/opt/lampp/logs/error_log`), permettendo al sistemista di monitorare i tentativi di attacco in modo silente.

---

## 🛠️ Requisiti di Replica del Laboratorio

1.  **Hypervisor:** VMware Workstation / Player.
2.  **OS:** 2x Ubuntu Server installati e configurati con IP statici coerenti.
3.  **Stack:** XAMPP per Linux avviato sul server target.
4.  **Database:** Dump SQL della tabella utenti (disponibile nella cartella `/sql` di questo archivio).
