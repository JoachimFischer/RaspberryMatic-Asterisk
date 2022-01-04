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

Die Dateien liegen als templete unter ``` /etc/asterisk ```. Hier wird eine weitere Datei mit dem Namen extensions.conf wird agelegt. Diese Datei enthält die Information, welche WAV Datei abgespielt werden soll. In diesem Fall ist es die Datei alarmmeldung, die als Sounddatei alammeldung.wav in das Verzeichnis ```/var/lib/asterisk/sounds/my/``` kopiert wird. 

Wenn man eine Sounddateie im mp3 Format hat, muss diese umgewandelt werden in wav Format mit dem folgenden Befehl:
```ruby
root@fipbox:~# ffmpeg -i /tmp/mySpeech.mp3 -ar 8000 -ac 1 -ab 64 /var/lib/asterisk/sounds/my/mySpeech.wav
```ruby

Datei: extensions.conf
```ruby
[general]
autofallthrough=no
static=yes
writeprotect=no
exten => _0X.,1,Dial(SIP/${EXTEN}@alarmanruf,60,r)
[asterisk-phones]
exten => 10,1,Answer()
exten => 10,n,Wait(2)
exten => 10,n,Playback(/var/lib/asterisk/sounds/my/alarmmeldung)
exten => 10,n,Wait(2)
exten => 10,n,Hangup()
```



