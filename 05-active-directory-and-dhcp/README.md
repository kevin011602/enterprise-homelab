# Promozione a Domain Controller, Configurazione di Active Directory e Centralizzazione dei Servizi DHCP

In questa sezione andiamo a realizzare il cuore logico dell'intera identità del nostro laboratorio enterprise indipendente: l'installazione dei **Servizi di dominio Active Directory (AD DS)** su Windows Server 2022 (**DC01**), il join del client Windows 10 e la successiva centralizzazione del ruolo **DHCP**, risolvendo i conflitti di rete monitorati tramite **Zabbix**.

## 1. Installazione del Ruolo Servizi di dominio Active Directory (AD DS)

In questa prima fase andiamo a preparare il server caricando sul disco i file binari necessari per Active Directory. Fino a questo momento la macchina è un semplice server standalone; l'installazione del ruolo è il prerequisito fondamentale per la successiva promozione.

1. Accediamo alla console di Windows Server 2022. Dalla dashboard principale del **Server Manager** (il centro nevralgico di gestione dell'OS), spostiamo lo sguardo in alto a destra, facciamo click sul menu a tendina **Gestione** (*Manage*) e selezioniamo la prima opzione: **Aggiungi ruoli e funzionalità** (*Add Roles and Features*).
   
   ![](assets/01.png)

2. Si apre il wizard di configurazione guidata. Nella schermata **Tipo di installazione**, lasciamo selezionata l'opzione predefinita **Installazione basata su ruoli o basata su funzionalità**. Questa scelta ci permette di aggiungere servizi mirati localmente sul nostro server, scartando le opzioni legate alle infrastrutture desktop remote (VDI). Clicchiamo su **Avanti >**.
   
   ![](assets/02.png)

3. Nella schermata **Selezione del server**, dobbiamo definire su quale macchina applicare le modifiche. Il radio button è impostato su *Selezionare un server dal pool di server*. Identifichiamo l'unica riga presente in tabella, che corrisponde alla nostra macchina locale: verifichiamo che il nome sia **DC01**, che risponda sull'IP `192.168.10.100` e che il sistema operativo sia *Windows Server 2022 Standard Evaluation*. Clicchiamo su **Avanti >**.
   
   ![](assets/03.png)

4. Arriviamo alla schermata cruciale dei **Ruoli server**. Scorriamo la lista dei servizi disponibili, individuiamo la voce **Servizi di dominio Active Directory** (*Active Directory Domain Services*) e andiamo a spuntare la relativa casella di controllo.
   
   ![](assets/04.png)

5. Non appena inseriamo la spunta, Windows Server capisce che il ruolo richiede degli strumenti di management specifici per poter essere configurato e gestito. Si apre quindi un pop-up automatico intitolato *Aggiungi funzionalità necessarie per Servizi di dominio Active Directory?*. Lasciamo tutto così com'è e clicchiamo sul pulsante **Aggiungi funzionalità** per approvare l'installazione degli strumenti di amministrazione remota (RSAT) e della console grafica. Il pop-up si chiuderà riportandoci al wizard principale.
   
   ![](assets/05.png)

6. Nella schermata successiva, dedicata alle **Funzionalità**, il sistema ci mostra i componenti software aggiuntivi. Noteremo che elementi fondamentali come il *.NET Framework 4.8* sono già pre-installati o selezionati automaticamente dal ruolo precedente. In questa fase non dobbiamo aggiungere nient'altro manualmente. Clicchiamo direttamente su **Avanti >** per saltare la schermata.
   
   ![](assets/06.png)

7. Il wizard ci mostra una scheda informativa dedicata ad Active Directory. È una pagina di riepilogo architetturale importante: ci ricorda il comportamento logico di AD DS e ci avvisa che, per funzionare correttamente, l'infrastruttura necessita di un server DNS all'interno della rete (che andremo a configurare nel passaggio successivo). Clicchiamo su **Avanti >**.
   
   ![](assets/07.png)

8. Siamo giunti alla schermata di **Conferma**, l'ultimo controllo prima della scrittura dei file su disco. Per ottimizzare i tempi e automatizzare le operazioni di laboratorio, andiamo a selezionare manualmente la casella di controllo **Riavvia automaticamente il server di destinazione se necessario**. All'apparire del pop-up di avviso che ci chiede l'autorizzazione al reboot immediato senza preavviso, clicchiamo su **Sì**. Verifichiamo nel riepilogo testuale che il ruolo in coda sia effettivamente *Servizi di dominio Active Directory* e facciamo click sul pulsante **Installa**.
   
   ![](assets/08.png)

## 2. Promozione del Server a Domain Controller (Inizializzazione della Foresta)

L'operazione precedente si limita a copiare i binari di Active Directory sul sistema, ma non rende la macchina un Domain Controller. Adesso eseguiamo la promozione vera e propria, definendo la struttura logica del nostro dominio.

1. Una volta che la barra di avanzamento nella schermata **Risultati** ha terminato il suo lavoro, il sistema ci avviserà che la configurazione è parziale con il messaggio: *“Configurazione necessaria. Installazione completata in DC01...”*. Per trasformare la macchina nel punto di riferimento dell'identità aziendale, facciamo click sul link ipertestuale blu **Alza di livello il server a controller di dominio** (*Promote this server to a domain controller*).
   
   ![](assets/09.png)

2. Si apre la *Configurazione guidata Servizi di dominio Active Directory*. Nella prima scheda, **Configurazione distribuzione**, dobbiamo impostare l'operazione di deployment. Trattandosi del primissimo Domain Controller del nostro homelab enterprise, andiamo a selezionare il terzo radio button: **Aggiungi una nuova foresta**. Subito sotto, nel campo di testo **Nome dominio radice**, digitiamo la stringa formale del nostro dominio locale: **`lab.local`**. Clicchiamo su **Avanti >**.
   
   ![](assets/10.png)

3. Nella scheda **Opzioni del controller di dominio**, impostiamo i livelli funzionali e le credenziali di emergenza. Lasciamo i menu a tendina *Livello funzionale della foresta* e *del dominio* impostati sul valore massimo di default disponibile (**Windows Server 2016**). Nella sezione dedicata alle funzionalità, le caselle **Server DNS (Domain Name System)** e **Catalogo globale (GC)** vengono selezionate in automatico (scelta mandatoria per il primo DC). Spostiamo l'attenzione in basso ed immettiamo una password sicura nei campi **Password** e **Conferma password** per la modalità **DSRM** (*Directory Services Restore Mode*). Questa chiave di sicurezza è fondamentale per effettuare attività di ripristino o manutenzione del database offline di Active Directory. Clicchiamo su **Avanti >**.
   
   ![](assets/11.png)

4. Arriviamo alla scheda **Opzioni DNS**. In alto notiamo un banner di avviso giallo con il testo: *“Impossibile creare una delega per il server DNS perché non è possibile trovare la zona radice autorevole corrispondente...”*. Questo comportamento è del tutto normale, atteso e corretto. Stiamo configurando una foresta radice (`lab.local`) completamente isolata e autorevole all'interno del nostro laboratorio privato; non esiste una zona DNS superiore su Internet a cui legarsi. Ignoriamo tranquillamente l'avviso e andiamo avanti cliccando su **Avanti >**.
   
   ![](assets/12.png)

5. Nello step **Opzioni aggiuntive**, il sistema esegue il parsing del dominio radice per assegnare l'identificativo NetBIOS legacy per la retrocompatibilità di rete. Il wizard rimuove automaticamente il suffisso `.local` e popola il campo con la stringa in maiuscolo **`LAB`**. Verificato che non vi siano conflitti di broadcast sulla nostra rete privata, clicchiamo su **Avanti >**.
   
   ![](assets/13.png)

6. Nella schermata **Percorsi**, lasciamo che Windows Server allochi i file strutturali dell'Identity Provider all'interno delle cartelle di sistema predefinite proposte come best practice (la cartella del database `NTDS`, i file di registro dei log e la cartella delle policy di gruppo `SYSVOL` sul disco `C:`). Clicchiamo su **Avanti >**.
   
   ![](assets/14.png)

7. La scheda **Verifica opzioni** funge da riepilogo generale di tutte le scelte effettuate nei passaggi precedenti. Diamo un'occhiata veloce per controllare che tutto sia coerente e facciamo click su **Avanti >** per lanciare la telemetria interna di convalida.
   
   ![](assets/15.png)

8. Nella schermata **Controllo dei prerequisiti**, l'OS esegue i test di conformità hardware e software sulla macchina DC01. Quando in alto compare il rassicurante banner verde con il messaggio d'esito positivo (*“Tutti i controlli dei prerequisiti sono riusciti...”*), siamo pronti. Nel box inferiore verranno mostrati alcuni warning standard sulla sicurezza degli algoritmi legacy (es. Windows NT 4.0) che possiamo ignorare. Facciamo click sul pulsante **Installa** per avviare la promozione a Domain Controller. Al termine della scrittura, il server eseguirà un riavvio automatico.
   
   ![](assets/16.png)
   
   ![](assets/17.png)

9. Al riavvio del server, la schermata di sblocco e logon di Windows ci conferma visivamente il successo dell'operazione: sotto l'icona dell'utente amministrativo non troviamo più il nome del computer locale, bensì la stringa formale del dominio **`LAB\Administrator`**. Inseriamo la password di amministrazione ed effettuiamo l'accesso. La Dashboard del Server Manager si caricherà mostrando i servizi AD DS e DNS perfettamente attivi.
   
   ![](assets/18.png)
   
   ![](assets/19.png)

## 3. Configurazione di Rete e Join al Dominio del Client Windows 10

Per fare in modo che una workstation venga inserita nella gerarchia aziendale e gestita centralmente dall'Identity Provider, dobbiamo configurare la sua interfaccia di rete e associarla esplicitamente al dominio `lab.local`.

1. Spostiamoci sulla macchina virtuale dedicata al client. Apriamo il Pannello di controllo, navighiamo in *Rete e Internet > Connessioni di rete*, facciamo click destro sull'icona della scheda di rete fisica/virtuale **Ethernet0** e selezioniamo *Proprietà*. Da qui, facciamo doppio click sulla voce **Protocollo Internet versione 4 (TCP/IPv4)**.

2. Per permettere alla macchina di trovare il Domain Controller, configuriamo temporaneamente un indirizzo IP fisso coerente con la sottorete del laboratorio:
   
   - Selezioniamo *Utilizza il seguente indirizzo IP* e inseriamo l'IP **`192.168.10.51`**, la subnet mask **`255.255.255.0`** e il gateway predefinito **`192.168.10.1`** (l'IP interno del nostro firewall pfSense).
   
   - Selezioniamo *Utilizza i seguenti indirizzi server DNS* e nel campo **Server DNS preferito** inseriamo l'indirizzo **`192.168.10.100`** (corrispondente all'IP del nostro Domain Controller DC01). **Nota critica**: questo passaggio è fondamentale; senza questo puntamento preciso, il client non sarà mai in grado di risolvere i record DNS interni necessari a localizzare Active Directory. Lasciamo vuoto il campo del DNS alternativo e clicchiamo su **OK** per applicare.
   
   ![](assets/20.png)

3. Prima di procedere con l'associazione formale, verifichiamo la bontà della risoluzione dei nomi. Apriamo il **Prompt dei comandi** sul client Windows 10 ed eseguiamo un test di ping verso il nome del dominio:
   
   DOS
   
   ```
   ping lab.local
   ```
   
   Verifichiamo con soddisfazione che il prompt esegua il parsing corretto traducendo il nome `lab.local` nell'IP del Domain Controller (`192.168.10.100`) con pacchetti interamente ricevuti.
   
   ![](assets/21.png)

4. Adesso passiamo all'azione. Apriamo la finestra delle **Proprietà del sistema** di Windows 10 (posizionandoci sulla scheda *Nome computer*). La schermata ci mostra il nome host attuale della macchina ed il gruppo di lavoro standard (*WORKGROUP*). Facciamo click sul pulsante **Cambia...**.

5. Nella sottofinestra *Cambiamenti dominio/nome computer*, spostiamo il pallino di attivazione nella sezione *Membro di* su **Dominio** e digitiamo all'interno del campo di testo la stringa del nostro dominio aziendale: **`lab.local`**. Facciamo click su **OK**.
   
   ![](assets/22.png)

6. L'invio della richiesta interroga il Domain Controller, il quale apre un box di sicurezza nativo di Windows sovrapposto allo schermo per autorizzare la creazione dell'account computer all'interno del database. Inseriamo le credenziali con privilegi elevati del dominio: nel campo nome utente digitiamo **`Administrator`** e nel campo password inseriamo la master password del laboratorio (es. **`P@ssword2026!`**). Clicchiamo su **OK**.
   
   ![](assets/23.png)

7. Dopo pochi istanti di elaborazione, il sistema convalida l'autenticazione e mostra il pop-up centrale di notifica informativa con il messaggio: *“Benvenuto nel dominio lab.local.”*. Clicchiamo su **OK** per chiudere la notifica. Il sistema ci avviserà che le modifiche diventeranno effettive solo dopo il riavvio obbligatorio della macchina; confermiamo e lasciamo riavviare il client.
   
   ![](assets/24.png)

8. Al riavvio della macchina client Windows 10, ci troviamo davanti alla schermata di blocco. Per evitare l'accesso con l'utente admin locale, facciamo click in basso a sinistra sulla voce **Altro utente**. Al centro compariranno i campi di inserimento credenziali e, cosa fondamentale, sotto la casella della password leggeremo la dicitura chiara: **Accedi a: LAB**. Questa è la conferma che la workstation sta puntando direttamente ad Active Directory. Digitiamo l'identità in formato legacy **`LAB\Administrator`** e la password per effettuare il primo logon centralizzato di test sulla workstation.
   
   ![](assets/25.png)
   
   ![](assets/26.png)

## 4. Gestione degli Oggetti e Creazione di un Utente di Dominio

L'utilizzo dell'account Administrator sui client è una pratica contraria alle policy di sicurezza. Procediamo a creare un utente standard all'interno del database della directory per le attività quotidiane del laboratorio.

1. Ritorniamo sulla console di **DC01**. Dal Server Manager, spostiamo il cursore nell'angolo superiore destro, clicchiamo sul menu a discesa **Strumenti** (*Tools*) e selezioniamo la voce dedicata: **Utenti e computer di Active Directory** (*Active Directory Users and Computers*). Sullo sfondo, la dashboard ci conferma che lo stato di salute di tutti i ruoli installati è nominale (marker verdi).
   
   ![](assets/27.png)

2. Si apre la console MMC di gestione degli oggetti (`dsa.msc`). Nel menu ad albero di sinistra espandiamo il nodo del dominio `lab.local` e selezioniamo il contenitore di sistema predefinito denominato **Users**. Spostiamo il cursore nello spazio di lavoro centrale, facciamo click con il tasto destro in un punto vuoto e selezioniamo il percorso dal menu contestuale: **Nuovo > Utente**.
   
   ![](assets/28.png)

3. Si apre il primo step del pannello guidato *Nuovo oggetto Utente*. Andiamo a valorizzare gli attributi anagrafici fondamentali (Nome e Cognome) e impostiamo la sintassi per le credenziali di rete. Nel campo **Nome accesso utente** (*User logon name*) digitiamo l'identificativo standard scelto per il laboratorio, nel mio caso **`kev.pal`**. Accanto, il menu a tendina assocerà il suffisso completando l'UPN in `kev.pal@lab.local`. Il campo sottostante per i sistemi precedenti a Windows 2000 si autocompilerà come `LAB\kev.pal`. Clicchiamo su **Avanti >**.
   
   ![](assets/29.png)

4. Nel secondo step del wizard andiamo a definire i criteri di sicurezza per la credenziale. Inseriamo una password complessa nei campi **Password** e **Conferma password**. Trattandosi di un account di test inserito in un ambiente di laboratorio controllato, andiamo a **deselezionare** la casella *Cambiamento obbligatorio password all'accesso successivo* e andiamo a selezionare esplicitamente la spunta su **Nessuna scadenza password**, evitando così che l'account scada o si blocchi durante i test futuri. Clicchiamo su **Avanti >** e successivamente su **Fine** nella schermata di riepilogo.
   
   ![](assets/30.png)

5. Per verificare l'effettiva operatività del nuovo oggetto, torniamo sulla macchina client Windows 10. Dalla schermata di logon, assicurandoci di aver selezionato la voce *Altro utente*, inseriamo nel campo di testo il nome di accesso appena creato (**`kev.pal`**) seguito dalla password corrispondente. Premiamo Invio: il client interrogherà il Domain Controller, validerà i privilegi e avvierà l'inizializzazione del nuovo profilo utente di rete sulla workstation.
   
   ![](assets/31.png)

## 5. Installazione e Configurazione del Ruolo Server DHCP

Mantenere gli IP statici sui client richiede un carico amministrativo eccessivo ed espone al rischio di errori manuali. Andiamo ad installare e centralizzare il servizio DHCP direttamente su Windows Server per automatizzare la distribuzione degli indirizzi IP e dei parametri DNS a tutte le macchine del laboratorio.

1. Ritorniamo su **DC01** e rilanciamo il wizard *Aggiungi ruoli e funzionalità* dal Server Manager. Avanziamo fino alla scheda **Ruoli server**, individuiamo nell'elenco la voce **Server DHCP** e andiamo a selezionare la relativa casella di controllo, approvando le funzionalità correlate nel pop-up che compare. Avanziamo nelle schermate successive senza apportare modifiche manuali fino alla fine e clicchiamo su **Installa**.
   
   ![](assets/32.png)

2. Al completamento del processo software, la scheda *Risultati* ci informerà che l'istanza è presente sul disco ma richiede passaggi post-installazione. Facciamo click sul link ipertestuale di avviso evidenziato in rosso: **Completa configurazione DHCP**.
   
   ![](assets/33.png)

3. Si apre la finestra secondaria *Configurazione guidata post-installazione DHCP*, posizionata sullo step logico di **Autorizzazione**. Per fare in modo che il server DHCP sia legittimato a distribuire IP all'interno del dominio Active Directory, dobbiamo autorizzarlo. Lasciamo attiva l'opzione predefinita *Usa credenziali dell'utente seguente* (che mostra già popolato il campo con l'account amministrativo principale **`LAB\Administrator`**) e facciamo click sul pulsante **Commit** in basso a destra. Una volta completato il task con successo, chiudiamo la finestra.
   
   ![](assets/34.png)

4. Torniamo sulla barra superiore del Server Manager, clicchiamo su **Strumenti** (*Tools*) e selezioniamo la voce specifica **DHCP** per lanciare la console amministrativa nativa.
   
   ![](assets/35.png)

5. All'interno della console DHCP, espandiamo il nodo del nostro server autorizzato **`dc01.lab.local`** nel menu ad albero di sinistra, rivelando le opzioni per i protocolli di rete. Facciamo click con il tasto destro sulla voce **IPv4** e, dal menu contestuale comparso al centro dello schermo, selezioniamo la seconda opzione dall'alto: **Nuovo ambito...** (*New Scope*).
   
   ![](assets/36.png)

6. Si avvia la procedura guidata di creazione dell'ambito logico di indirizzamento. Nel primo step operativo, denominato **Nome ambito**, dobbiamo assegnare un identificativo testuale alla rete. All'interno del campo di testo obbligatorio *Nome*, digitiamo una stringa chiara come **`Rete_Lab`**, lasciamo vuoto il campo *Descrizione* e clicchiamo su **Avanti >**.
   
   ![](assets/37.png)

7. Nella scheda **Intervallo indirizzi IP**, dobbiamo andare a recintare il perimetro del nostro pool di distribuzione. Nella sezione *Impostazioni di configurazione per il server DHCP*, compiliamo i limiti esatti della nostra sottorete aziendale:
   
   - **Indirizzo IP iniziale**: `192.168.10.50`
   
   - **Indirizzo IP finale**: `192.168.10.150`
     
     Nella sezione sottostante, dedicata ai parametri della subnet mask, impostiamo il valore di contatore della **Lunghezza** su **`24`**, che autocompilerà il campo *Subnet mask* con il valore standard della classe C: **`255.255.255.0`**. Clicchiamo su **Avanti >**.
   
   ![](assets/38.png)

8. Avanziamo nel wizard superando le schermate di esclusione e di lease-time fino ad arrivare alla scheda **Router (gateway predefinito)**. Questo passaggio permette al DHCP di indicare ai client la strada per uscire dalla rete locale. Nel campo di testo inseriamo l'indirizzo IP interno del nostro firewall pfSense: **`192.168.10.1`**, facciamo click sul pulsante *Aggiungi* per registrarlo nella lista di instradamento e clicchiamo su **Avanti >**.
   
   ![](assets/39.png)

9. Arriviamo alla scheda **Nome dominio e server DNS**. Il wizard eredita automaticamente le impostazioni di Active Directory, mostrando nel campo *Dominio padre* la stringa `lab.local` e inserendo nella lista inferiore l'IP del nostro Domain Controller (**`192.168.10.100`**). Questo assicurerà che ogni client riceva l'IP di DC01 come DNS primario, garantendo l'integrità del join. Clicchiamo su **Avanti >**, selezioniamo l'opzione per attivare l'ambito immediatamente e concludiamo la procedura.
   
   ![](assets/40.png)

## 6. Passaggio del Client a DHCP e Risoluzione dei Conflitti di Rete (Rogue DHCP)

Avendo configurato un server DHCP ufficiale all'interno della Active Directory, dobbiamo riconfigurare il client Windows 10 affinché smetta di usare l'IP fisso temporaneo e ascolti i broadcast della rete. Tuttavia, la presenza simultanea di un altro server DHCP attivo (come quello predefinito su pfSense) genererà un grave conflitto di rete.

1. Spostiamoci sulla macchina virtuale **Windows 10** ed entriamo nuovamente nelle proprietà IPv4 della scheda di rete Ethernet0 per rimuovere l'assegnazione statica. Spostiamo i radio button selezionando **Ottieni automaticamente un indirizzo IP** e **Ottieni indirizzo server DNS automaticamente**. Clicchiamo su **OK** in basso a destra per applicare le modifiche e forzare la scheda di rete in ascolto di broadcast DHCP.
   
   ![](assets/41.png)

2. Per convalidare lo stato del network e verificare i parametri ottenuti in modalità dinamica, apriamo il **Prompt dei comandi** sul client ed eseguiamo l'ispezione visiva tramite il comando:
   
   DOS
   
   ```
   ipconfig /all
   ```
   
   Analizzando l'output a schermo, verifichiamo che la configurazione sia corretta e allineata: la voce *DHCP abilitato* è contrassegnata da un chiaro *Sì*, l'indirizzo IPv4 assegnato è il primo del pool (**`192.168.10.50`**) e i parametri di *Server DHCP* e *Server DNS* puntano stabilmente all'indirizzo del Domain Controller (`192.168.10.100`).
   
   ![](assets/42.png)

3. **Risoluzione del conflitto di rete (Rogue DHCP):** L'infrastruttura presenta ora un'anomalia critica: sia Windows Server sia il firewall pfSense rispondono alle richieste DHCP sulla stessa subnet. Per risolvere questo conflitto, apriamo il browser e colleghiamoci alla Web GUI di pfSense (`https://192.168.10.1`). Navighiamo nel menu superiore seguendo il percorso **Services > DHCP Server** e posizioniamoci sulla scheda d'interfaccia **LAN**. Individuiamo la primissima casella di controllo in alto, denominata **Enable DHCP server on LAN interface**, e **deselezioniamola** (rimuovendo la spunta). Questa scelta sistemistica è fondamentale per spegnere il server DHCP non autorizzato, delegando interamente la distribuzione degli IP a Windows Server. Scorriamo la pagina verso il basso e facciamo click su **Save** per rendere definitive le modifiche.
   
   ![](assets/43.png)

4. **Intercettazione dell'allarme su Zabbix:** Avendo spento il servizio sul firewall, l'infrastruttura di monitoraggio rileva immediatamente il cambiamento di stato. Se andiamo a dare uno sguardo alla nostra console enterprise **Zabbix**, noteremo che nel widget **Current Problems** della dashboard principale è scattato un alert associato alla macchina DC01 (o al nodo pfSense):
   
   > **Problem:** PFSense: DHCP server is not running
   
   ![](assets/44.png)

5. **Disattivazione del trigger obsoleto:** Poiché lo spegnimento del DHCP su pfSense è una scelta architetturale definitiva e desiderata per questo laboratorio, dobbiamo istruire Zabbix affinché non lo consideri più come un problema. Nel menu laterale di Zabbix, navighiamo in **Data Collection > Hosts**.

6. All'interno della tabella, individuiamo la riga corrispondente a **pfSense-Core**. Spostiamo lo sguardo verso destra e clicchiamo sul link ipertestuale **Triggers** associato a questo host.
   
   ![](assets/45.png)

7. Nella lista dei controlli attivi, cerchiamo la regola legata allo stato del servizio DHCP e andiamo a cliccare sullo stato **Enabled**: l'interfaccia lo commuterà in **Disabled**, spegnendo definitivamente il monitoraggio su quel parametro e ripulendo la dashboard principale da alert non necessari.
   
   ![](assets/46.png)

L'allineamento è completato con successo: c'è un solo server DHCP attivo nel dominio, i falsi positivi su Zabbix sono stati azzerati e l'infrastruttura di rete è ora pulita, realistica e centralizzata sotto il controllo del Domain Controller.
