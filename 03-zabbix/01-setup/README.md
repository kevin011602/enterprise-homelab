# Installazione di Zabbix Server su CentOS 9

In questa sezione viene documentata l'installazione del sistema di monitoraggio centralizzato **Zabbix 7.0 LTS** su sistema operativo **CentOS Stream 9**, utilizzando **MariaDB** come Database relazionale.

---

## 1. Predisposizione dell'Ambiente e Repository

Accedere alla console di CentOS, passare all'utente root e scaricare i repository ufficiali di Zabbix.

```bash
# Passare ai privilegi di root
sudo su

# Installare il repository ufficiale di Zabbix 7.0 LTS per CentOS 9
rpm -Uvh https://repo.zabbix.com/zabbix/7.0/centos/9/x86_64/zabbix-release-latest.el9.noarch.rpm

# Pulire la cache dei pacchetti
dnf clean all
```

## 2. Installazione dei Pacchetti Richiesti

Installare il server Zabbix, l'interfaccia web, i componenti per Apache/PHP, le policy di SELinux e il server database MariaDB.

```bash
dnf install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent mariadb-server --disablerepo=epel -y
```

## 3. Configurazione dell'Infrastruttura Database (MariaDB)

Abilitare il servizio database, accedere alla console di MySQL e creare lo schema e l'utente per Zabbix.

```bash
# Abilitare e avviare MariaDB al boot
systemctl enable --now mariadb

# Accedere alla console del DBMS
mysql -u root
```

All'interno della shell di MySQL/MariaDB, eseguire le seguenti query:

```SQL
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'ZabbixPassword2026!';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
QUIT;
```

### Importazione dello Schema Iniziale

Importare le tabelle e i dati iniziali forniti da Zabbix all'interno del database appena creato. Quando richiesto, inserire la password del database (`ZabbixPassword2026!`).

```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
# password: ZabbixPassword2026!
```

Disattivare la funzione di creazione funzioni subito dopo l'importazione:

```bash
mysql -u root -e "SET GLOBAL log_bin_trust_function_creators = 0;"
```

## 4. Configurazione del Server Zabbix

Modificare il file di configurazione principale di Zabbix Server per indicare la password di connessione al database.

```bash
nano /etc/zabbix/zabbix_server.conf
```

1. Premere `CTRL + W` per cercare la stringa.
2. Localizzare la riga `# DBPassword=`.
3. Decommentare la riga (rimuovendo il `#`) e impostare la password del laboratorio:

```plaintext
DBPassword=ZabbixPassword2026!
```

4. Salvare ed uscire (`CTRL + O`, `Invio`, `CTRL + X`).

## 5. Configurazione del Firewall Locale (CentOS)

Aprire le porte necessarie per il traffico Web (HTTP/HTTPS) e per la comunicazione tra gli agenti e il server Zabbix (porte 10050 e 10051).

```bash
firewall-cmd --add-service={http,https} --permanent
firewall-cmd --add-port={10050/tcp,10051/tcp} --permanent
firewall-cmd --reload
```

## 6. Avvio dei Servizi e Verifica di Rete

Avviare tutti i servizi necessari e impostarli per l'avvio automatico ad ogni boot del server.

```bash
systemctl restart zabbix-server zabbix-agent httpd php-fpm
systemctl enable zabbix-server zabbix-agent httpd php-fpm
```

### Verifica dell'indirizzo IP del Server

Verificare l'indirizzo IP statico o dinamico associato all'interfaccia di rete principale (es. `ens33`):

```bash
ip a
```

Nel nostro scenario, l'indirizzo IP rilevato sotto la voce `inet` dell'interfaccia `ens33` è: `192.168.10.50`

## 7. Primo Accesso alla Web GUI

Spostarsi su una macchina client (es. Windows 10 x64 attestata sul medesimo LAN Segment) ed aprire il browser web.

- URL di configurazione iniziale: `http://192.168.10.50/zabbix`

### Credenziali di default per il primo login:

- Username: `Admin` (Nota: la "A" deve essere maiuscola)
- Password: `zabbix`