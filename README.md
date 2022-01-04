# RaspberryMatic-Asterisk
RaspberryMatic kann Telefonanrufe über die PBX Asterisk ausführen für Alarmmeldungen

Schritt1:
Installation vom Asterisk Server auf dem Raspberry
```ruby
root@fipbox:~# su root
sudo apt-get install asterisk
```

Test der Asterisk installation:
```ruby
root@fipbox:~# asterisk -c
Asterisk already running on /var/run/asterisk/asterisk.ctl.  Use 'asterisk -r' to connect.
```


Um einen Test Call durchzuführen, muss die Datei  mycall.call in das Verzeichnis  /var/spool/asterisk/outgoing  copieren:

```ruby
root@fipbox:~# cp /root/mycall.call /var/spool/asterisk/outgoing
```

Die Datei ``mycall.call`` enthält die folgende Information:
```ruby
Channel: SIP/0171HANDYNUMMER@alarmanruf
MaxRetries: 2
RetryTime: 60
WaitTime: 30
Context: asterisk-phones
Extension: 10
Callerid:2000
```




