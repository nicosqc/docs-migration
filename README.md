# docs-migration
Das folgende Beispiel zeigt, wie man Dateien von einem alten Kundenserver (in diesem Fall DataOpt) zu einem neuen Cloud-Server kopiert, der mit Ansible provisioniert ist.
## 1. Verbinden mit dem Server
```bash
eval $(ssh-agent) ; ssh-add ~/.ssh/id_rsa && ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa -p 6789 root@217.86.148.100
```
Aufschlüsselung:

`eval $(ssh-agent)`:

Startet den SSH-Agenten, der für die Verwaltung von SSH-Schlüsseln zuständig ist. Der Befehl wird in der aktuellen Shell ausgeführt und ist für die gesamte Shell-Session aktiv.
`ssh-add ~/.ssh/id_rsa`:

Fügt deinen privaten SSH-Schlüssel (`~/.ssh/id_rsa`) zum SSH-Agenten hinzu, sodass du dich ohne Eingabe des Passworts verbinden kannst.
`ssh -o [...] -p 6789 root@217.86.148.100`:

Stellt eine SSH-Verbindung zum Server mit der IP-Adresse `217.86.148.100` auf Port `6789` als Benutzer `root` her.
Optionen:
- `-oKexAlgorithms=+diffie-hellman-group1-sha1`: 
  Legt das Schlüsselaustauschverfahren fest, das SSH verwenden soll. Hier wird `diffie-hellman-group1-sha1` explizit zugelassen, da dieses ältere, möglicherweise unsichere Verfahren in einigen Umgebungen erforderlich ist.

- `-oHostKeyAlgorithms=+ssh-rsa`: 
  Erlaubt die Verwendung des `ssh-rsa`-Algorithmus für die Host-Schlüsselüberprüfung. Dies ist nützlich, wenn der Server ältere Verschlüsselungsprotokolle verwendet.

- `-oPubkeyAcceptedKeyTypes=+ssh-rsa`: 
  Legt fest, dass der SSH-Befehl `ssh-rsa` als akzeptierte Schlüsseltyp zur Authentifizierung zulässt. Auch dies ist notwendig, wenn das entfernte System ältere Algorithmen benötigt.

## 2. Archiv erstellen
```bash
tar -czf ~/docs.tar.gz /var/www/html/mobidas/wws/docs/  /var/www/html/mobidas/wws/public/document/ /var/www/html/mobidas/wws/public/file/ /var/www/html/mobidas/wws/public/item1/ /var/www/html/mobidas/wws/public/item2
```
Aufschlüsselung:

`tar -czf ~/docs.tar.gz /var/www/html/Pfad1 /var/www/html/Pfad2 /var/www/html/Pfad3`:
Erzeugt ein komprimiertes Archiv (.tar.gz) mit dem Namen `docs.tar.gz` im Home-Verzeichnis (`~/`).
Optionen:
- -c: Erstellen eines neuen Archivs.
- -z: Komprimieren des Archivs mit gzip.
- -f: Gibt den Dateinamen für das Archiv an.
- Pfad-Argumente: Die angegebenen Verzeichnisse werden ins Archiv aufgenommen.

Anschließend wird die Sitzung durch `exit` oder CTRL+D beendet. Die Datei befindet sich im home-Verzeichnis des root-Benutzers `/root/docs.tar.gz`.

## Archiv vom Remote-Server herunterladen
Zuerst wird sich mit dem Zielserver verbunden, um nicht durch die Übertragungsgeschwindigkeit der lokalen Maschine gedrosselt zu sein. Wir leiten den SSH-Agent an den vertrauenswürdigen Server unter unserer Kontrolle weiter, um uns von dort aus mit dem alten Server verbinden zu können, ohne unseren SSH-Schlüssel übertragen zu müssen:
```bash
ssh -oForwardAgent=yes ppl.beveb.com
```
Anschließend verbinden wir uns vom Zielserver mit dem Ursprungsserver und kopieren die Datei auf den Server:
```bash
scp -oKexAlgorithms=+diffie-hellman-group1-sha1 -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa -P 6789 root@217.86.148.100:/root/docs.tar.gz ~/
```

Aufschlüsselung:

`scp`: Der Befehl für "Secure Copy Protocol" (SCP) überträgt Dateien zwischen einem lokalen und einem Remote-System unter Verwendung des SSH-Protokolls.

### Optionen:

- `-oKexAlgorithms=+diffie-hellman-group1-sha1`: 
  Legt das Schlüsselaustauschverfahren fest, das SCP verwenden soll. Hier wird `diffie-hellman-group1-sha1` explizit zugelassen, da dieses ältere, möglicherweise unsichere Verfahren in einigen Umgebungen erforderlich ist.

- `-oHostKeyAlgorithms=+ssh-rsa`: 
  Erlaubt die Verwendung des `ssh-rsa`-Algorithmus für die Host-Schlüsselüberprüfung. Dies ist nützlich, wenn der Server ältere Verschlüsselungsprotokolle verwendet.

- `-oPubkeyAcceptedKeyTypes=+ssh-rsa`: 
  Legt fest, dass der SCP-Befehl `ssh-rsa` als akzeptierte Schlüsseltyp zur Authentifizierung zulässt. Auch dies ist notwendig, wenn das entfernte System ältere Algorithmen benötigt.

- `-P 6789`: 
  Gibt den Port an, über den die Verbindung hergestellt wird. Hier wird Port `6789` statt des Standard-SSH-Ports (22) verwendet.

### Verbindungsdetails:

- `root@217.86.148.100`: 
  Der Benutzername `root` und die IP-Adresse des Remote-Servers (`217.86.148.100`).

- `/root/docs.tar.gz`: 
  Der vollständige Pfad zur Datei `docs.tar.gz` auf dem Remote-Server. Sie befindet sich im Home-Verzeichnis des `root`-Benutzers.

- `~/`: 
  Das Zielverzeichnis auf dem lokalen Rechner, in das die Datei kopiert wird. `~/` verweist auf das Home-Verzeichnis des aktuell angemeldeten Benutzers.

## 4. Archiv entpacken und Berechtigungen anpassen
Anschließend wird das Archiv entpackt, seine Unterordner an die richtige Stelle verschoben und die Berechtigungen repariert:
```bash
tar -xf ~/docs.tar.gz && cp -r --parents /root/var/www/html/mobidas/wws/docs /root/var/www/html/mobidas/wws/public/item1 /root/var/www/html/mobidas/wws/public/item2 /root/var/www/html/mobidas/wws/public/document /root/var/www/html/mobidas/wws/public/file /var/www/ppl.beveb.com/public/wws && chown -R www-data:www-data /var/www/ppl.beveb.com/public/wws && find /var/www/ppl.beveb.com/public/wws -type f -exec chmod 644 {} \;
```

Aufschlüsselung:

### 1. Archiv extrahieren
- `tar -xf ~/docs.tar.gz`: Entpackt das Archiv `docs.tar.gz` im Home-Verzeichnis (`~`).
  - `-x`: Extrahieren des Archivs.
  - `-f`: Gibt die Archivdatei (`docs.tar.gz`) an.

### 2. Kopieren der extrahierten Dateien
- `cp -r --parents /root/var/www/html/mobidas/wws/docs /root/var/www/html/mobidas/wws/public/item1 /root/var/www/html/mobidas/wws/public/item2 /root/var/www/html/mobidas/wws/public/document /root/var/www/html/mobidas/wws/public/file /var/www/ppl.beveb.com/public/wws`:
  - `cp`: Kopiert die angegebenen Verzeichnisse und Dateien an den Zielort.
  - `-r`: Kopiert rekursiv (inkl. aller Unterverzeichnisse und Dateien).
  - `--parents`: Bewahrt die ursprüngliche Verzeichnisstruktur im Zielverzeichnis bei.
  - Die Verzeichnisse unter `/root/var/www/html/mobidas/wws/` werden in das Zielverzeichnis `/var/www/ppl.beveb.com/public/wws` kopiert.

### 3. Benutzerrechte anpassen
- `chown -R www-data:www-data /var/www/ppl.beveb.com/public/wws`: 
  - Ändert den Besitzer der Dateien im Verzeichnis `/var/www/ppl.beveb.com/public/wws` auf den Benutzer `www-data` und die Gruppe `www-data`.
  - `-R`: Anwenden der Rechteänderung rekursiv auf alle Dateien und Unterverzeichnisse.

### 4. Verzeichnisberechtigungen setzen
- `find /var/www/ppl.beveb.com/public/wws -type d -exec chmod 755 {} \;`: Setzt die Berechtigungen für alle Verzeichnisse auf `755` (Lesen, Schreiben und Ausführen für den Eigentümer; Lesen und Ausführen für Gruppe und andere Benutzer).

### 5. Dateiberechtigungen setzen
- `find /var/www/ppl.beveb.com/public/wws -type f -exec chmod 644 {} \;`: Setzt die Berechtigungen für alle Dateien auf `644` (Lese- und Schreibrechte für den Eigentümer, nur Leserechte für Gruppe und andere Benutzer).
