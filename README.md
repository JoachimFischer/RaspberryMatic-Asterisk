# RaspberryMatic-Asterisk
RaspberryMatic kann Telefonanrufe über die PBX Asterisk ausführen für Alarmmeldungen

Schritt1:
Installation vom Asterisk Server auf dem Raspberry
``root@fipbox:~# su root``
``sudo apt-get install asterisk``

Test der Asterisk installation:
``root@fipbox:~# asterisk -c
Asterisk already running on /var/run/asterisk/asterisk.ctl.  Use 'asterisk -r' to connect.``


Um einen Test Call durchzuführen, muss die Datei  mycall.call in das Verzeichnis  /var/spool/asterisk/outgoing  copieren
cp /root/mycall.call /var/spool/asterisk/outgoing




