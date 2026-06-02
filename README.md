# 🌐 Enterprise Homelab & Cyber Security Laboratory

Benvenuto nel repository dedicato al mio laboratorio indipendente di infrastruttura di rete e sicurezza aziendale. Questo ecosistema virtuale è progettato per simulare scenari enterprise reali, integrando servizi di identità centralizzati, monitoraggio proattivo, segregazione del traffico e analisi delle minacce.

L'obiettivo del progetto è la sperimentazione pratica di configurazioni di sistema, hardening, gestione delle policy di dominio e centralizzazione dei servizi di rete.

## 🏗️ Architettura Attuale della Rete

L'infrastruttura è configurata su un segmento di rete isolato (**LAN Segment: "lab"**) sulla sottorete **`192.168.10.0/24`**, con i servizi DHCP e DNS interamente delegati alla componente Windows Server per garantire l'integrità del dominio Active Directory.

### 1. 🛡️ pfSense-Core | Il Gateway di Rete

> **IP Principale:** `192.168.10.1`
> 
> **Sistema Operativo:** FreeBSD (pfSense)

- **Ruolo:** Agisce esclusivamente come Router perimetrale, Firewall e Gateway di uscita verso Internet per l'intero laboratorio.

- **Stato Attuale:** Il server DHCP interno è stato **disattivato** per evitare conflitti di broadcast (Rogue DHCP) sulla sottorete, delegando il compito a DC01.

- **Interfacce di Rete:**
  
  - `WAN`: Configurata in modalità NAT (collegamento verso l'host fisico/Internet).
  
  - `LAN`: Attestata sul segmento isolato "lab".

### 2. 🧠 DC01 | Il Cervello del Dominio (`lab.local`)

> **IP Principale:** `192.168.10.100`
> 
> **Sistema Operativo:** Windows Server 2022 Standard

- **Ruolo:** Domain Controller primario della foresta **`lab.local`** (NetBIOS: `LAB`).

- **Servizi e Configurazione Attivi:**
  
  - **AD DS:** Gestione centralizzata delle identità (es. account `Administrator`, `kev.pal`).
  
  - **DNS Server:** Risoluzione dei nomi interna ed essenziale per il corretto funzionamento di Active Directory.
  
  - **DHCP Server:** Unico distributore ufficiale di indirizzi IP della LAN (Pool attivo: `192.168.10.50` - `192.168.10.150`).
  
  - **Group Policy Management (GPO):** Configurazione e distribuzione di policy di dominio centralizzate. Implementato l'hardening degli endpoint tramite blocco selettivo delle impostazioni di sistema.

### 3. 🖥️ Windows 10 Client | La Workstation Aziendale

> **IP Attuale (DHCP):** `192.168.10.51`
> 
> **Sistema Operativo:** Windows 10 Professional

- **Ruolo:** Client standard utilizzato per testare i privilegi utente e l'applicazione delle configurazioni centralizzate.

- **Stato Attuale:** Macchina inserita ufficialmente in **Join al dominio `lab.local`**. L'interfaccia di rete è dinamica (DHCP automatico) e riceve IP, DNS e Gateway direttamente da DC01. Supporta il login con utente locale o tramite credenziali di rete (`LAB\kev.pal`).

- **Sicurezza & Hardening:** Sulla workstation è applicata attivamente la **GPO di blocco del Pannello di Controllo e delle Impostazioni PC** per gli utenti non amministrativi. Include lo Zabbix Agent pre-installato.

### 4. 📊 Zabbix Server | Il Centro di Monitoraggio

> **IP Principale:** `192.168.10.50`
> 
> **Sistema Operativo:** CentOS Stream 9

- **Ruolo:** Engine di monitoraggio proattivo dell'intera infrastruttura virtuale.

- **Stato Attuale:** Raccoglie telemetria e dati prestazionali dagli host. I trigger di allarme sono stati ottimizzati (tuning manuale) per ignorare lo spegnimento del servizio DHCP su pfSense, garantendo una dashboard pulita e priva di falsi positivis (*PFSense: DHCP server is not running* impostato su *Disabled*).

### 5. 🐳 Lubuntu Server | L'Host Applicativo

> **IP Riservato:** `192.168.10.52`
> 
> **Sistema Operativo:** Lubuntu (Linux leggero)

## 📋 Tabella Riassuntiva dei Dispositivi

| **Dispositivo**       | **Indirizzo IP** | **Ruolo Principale**                  | **Sistema Operativo** |
| --------------------- | ---------------- | ------------------------------------- | --------------------- |
| **pfSense-Core**      | `192.168.10.1`   | Router / Firewall / Gateway           | FreeBSD (pfSense)     |
| **DC01**              | `192.168.10.100` | Domain Controller / DHCP / DNS / GPO  | Windows Server 2022   |
| **Windows 10 Client** | Dynamic (`.51`)  | Postazione Utente Standard (Hardened) | Windows 10 Pro        |
| **Zabbix Server**     | `192.168.10.50`  | Monitoraggio Centrale                 | CentOS Stream 9       |
| **Lubuntu Server**    | `192.168.10.52`  | Server Applicativo                    | Lubuntu Linux         |

## 📂 Struttura del Repository

Il repository è organizzato in cartelle numerate che tracciano l'evoluzione logica del laboratorio. Ogni cartella contiene la documentazione tecnica dettagliata e gli asset grafici delle configurazioni.
