# Linux Lab: SQL Injection (Data Exfiltration) & Infrastructure Hardening

Questo repository contiene un progetto di laboratorio orientato alla cybersecurity e alla gestione infrastrutturale. Lo scenario simula un attacco di SQL Injection (SQLi) mirato all'estrazione di dati sensibili da un database, seguito dall'implementazione di contromisure sistemistiche su piattaforma Linux.

⚠️ **Disclaimer:** *Questo progetto è stato realizzato esclusivamente a scopo didattico in un ambiente virtuale isolato e controllato.*

---

## 📐 Architettura di Rete e Infrastruttura

L'ambiente è stato interamente virtualizzato su **VMware** ed è composto da due server Linux Ubuntu configurati con **IP statici sulla stessa sottorete** per garantire la comunicazione diretta:

*   **Server Target (Ubuntu Server + XAMPP):** Funge da Web Server dell'infrastruttura. Ospita il servizio Apache, il DBMS MySQL e l'applicazione web di autenticazione (`login.php`).
*   **Client Attaccante (Ubuntu Server):** Macchina remota utilizzata per testare la sicurezza del Web Server e sferrare l'attacco attraverso la rete locale.

---

## 🔓 Analisi della Vulnerabilità e Dinamica dell'Attacco

La pagina `login.php` sul server target accettava l'input dell'utente (il campo email/username) e lo passava direttamente al database MySQL senza alcuna pulizia, validazione o sanitizzazione dei parametri.

### Impatto dell'Attacco (Data Exfiltration)
Sfruttando la mancata validazione dell'input nel campo dell'username, è stato inserito un payload SQL ad hoc che ha permesso di manipolare la query originale. Invece di limitarsi a bloccare o bypassare il login, l'iniezione ha forzato il database a restituire a schermo informazioni strutturali e di sistema cruciali.

Tramite questo attacco è stato possibile estrarre:
1.  Il **nome esatto del database** in uso.
2.  L'identità dell'utente di sistema del DBMS (scoprendo che il servizio girava come utente **`root`**).
3.  L'elenco completo di tutti gli **username e le password** (hash) presenti nelle tabelle degli utenti.

---

## 🛡️ Approccio Sistemistico e Contromisure (Hardening)

Se da un lato la risoluzione definitiva del problema richiede la correzione del codice PHP (tramite *Prepared Statements*), dall'altro il compito di un **sistemista** è limitare i privilegi e contenere i danni qualora un'applicazione web venga compromessa. 

