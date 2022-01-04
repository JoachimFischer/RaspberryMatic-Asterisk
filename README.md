# RaspberryMatic-Asterisk
RaspberryMatic kann Telefonanrufe über die PBX Asterisk ausführen für Alarmmeldungen

Schritt1:
Installation vom Asterisk Server auf dem Raspberry
su root
sudo apt-get install asterisk

Test der Asterisk installation:
root@devdeb ~ # asterisk -c
Asterisk Ready.
*CLI>

Um einen Test Call durchzuführen, die Datei  mycall.call in das Verzeichnis  /var/spool/asterisk/outgoing  copieren
cp /root/mycall.call /var/spool/asterisk/outgoing




