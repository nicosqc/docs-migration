# docs-migration
Das folgende Beispiel zeigt, wie man Dateien von einem alten Kundenserver (in diesem Fall DataOpt) zu einem neuen Cloud-Server kopiert, der mit Ansible provisioniert ist.
## 1. Verbinden mit dem Server
```bash
eval $(ssh-agent) ; ssh-add ~/.ssh/id_rsa && ssh -oForwardAgent=yes -oKexAlgorithms=+diffie-hellman-group1-sha1 -oHostKeyAlgorithms=+ssh-rsa -oPubkeyAcceptedKeyTypes=+ssh-rsa -p 6789 root@217.86.148.100
```
Aufschlüsselung:

`eval $(ssh-agent)`:

Startet den SSH-Agenten, der für die Verwaltung von SSH-Schlüsseln zuständig ist. Der Befehl wird in der aktuellen Shell ausgeführt und ist für die gesamte Shell-Session aktiv.
`ssh-add ~/.ssh/id_rsa`:

Fügt deinen privaten SSH-Schlüssel (`~/.ssh/id_rsa`) zum SSH-Agenten hinzu, sodass du dich ohne Eingabe des Passworts verbinden kannst.
`ssh -oForwardAgent=yes ... -p 6789 root@217.86.148.100`:

Stellt eine SSH-Verbindung zum Server mit der IP-Adresse `217.86.148.100` auf Port `6789` als Benutzer `root` her.
Optionen:
`-oForwardAgent=yes`: Erlaubt die Weiterleitung des SSH-Agenten (nützlich, wenn du dich von diesem Server aus weiter verbinden möchtest).
`-oKexAlgorithms=+diffie-hellman-group1-sha1`: Fügt einen spezifischen Schlüsselwechselalgorithmus hinzu, der möglicherweise von älteren Servern benötigt wird.
`-oHostKeyAlgorithms=+ssh-rsa`: Akzeptiert SSH-RSA-Host-Schlüssel (eigentlich veraltet).
`-oPubkeyAcceptedKeyTypes=+ssh-rsa`: Akzeptiert RSA-Öffentlichen Schlüssel (eigentlich veraltet).
## 2. Archiv erstellen
```bash
tar -czf ~/docs.tar.gz /var/www/html/mobidas/wws/docs/  /var/www/html/mobidas/wws/public/document/ /var/www/html/mobidas/wws/public/file/ /var/www/html/mobidas/wws/public/item1/ /var/www/html/mobidas/wws/public/item2
```
Aufschlüsselung:

`tar -czf ~/docs.tar.gz ...`:
Erzeugt ein komprimiertes Archiv (.tar.gz) mit dem Namen docs.tar.gz im Home-Verzeichnis (~/).
Optionen:
-c: Erstellen eines neuen Archivs.
-z: Komprimieren des Archivs mit gzip.
-f: Gibt den Dateinamen für das Archiv an.
Pfad-Argumente: Die angegebenen Verzeichnisse werden ins Archiv aufgenommen.
## 3. Archiv kopieren
```bash
scp ~/docs.tar.gz ppl.beveb.com:/root/
```
Aufschlüsselung:

`scp ~/docs.tar.gz ppl.beveb.com:/root/`:
Kopiert die Datei `docs.tar.gz` im Homeverzeichnis des Lokalen SystemS zum Server `ppl.beveb.com` ins Verzeichnis `/root/`.
scp steht für "secure copy" und wird verwendet, um Dateien sicher über SSH zu übertragen.

Anschließend wird die Sitzung durch `exit` oder CTRL+D beendet.
## 4. Archiv entpacken
Zuerst muss sich mit dem Zielrechner verbunden werden:
```bash
ssh root@ppl.beveb.com
```
Anschließend wird das Archiv entpackt und seine Unterordner an die richtige Stelle verschoben:
```bash
tar -xf ~/docs.tar.gz
```
