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

Die Datei ``mycall.call`` enthält die folgende Information und liegt imm Verzeichnis ```/root```: 
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
```

Datei: extensions.conf
```ruby
[general]
autofallthrough=no
static=yes
writeprotect=no
exten => _0X.,1,Dial(SIP/${EXTEN}@alarmanruf,60,r)                   ; die externe NUmmer wird angerufen, die mit 0 beginnt, also 0172xxxx
[asterisk-phones]                                                    ; Bezeichnung aus der Datei mycall.call
exten => 10,1,Answer()
exten => 10,n,Wait(2)
exten => 10,n,Playback(/var/lib/asterisk/sounds/my/alarmmeldung)     ; die WAV Datei wird hier angegeben
exten => 10,n,Wait(2)
exten => 10,n,Hangup()
```
Der Asterisk Server wird mit der Datei ```sip.conf``` konfiguriert und liegt als Vorlage im Verzeichnis ```/etc/asterisk```
```ruby
[general]
allowguest=no
port = 5060                                            ; Standard Port
bindaddr = 0.0.0.0
qualify = no
disable = all
allow = alaw
allow = ulaw
videosupport = yes
dtmfmode = rfc2833
srvlookup = yes
localnet=192.168.1.1/255.255.255.0                     ; IP der FritzBox
directmedia = no
nat = yes
transport = udp
register => alarmanruf:ppppp@192.168.1.1/alarmanruf    ; Für die Registrierung der SIP aus der FritzBox
                                                       ; In der FritzBox habe ich einen SIP mit user: alarmanruf und passwort: ppppp sowie Namen: alarmanruf eingerichtet
                                                       ; ppppp ist das Passwort, das in der Fritz!Box für den SIP hinterlegt ist

[alarmanruf]                                           ; Von der SIP config in der FritzBox
permit=192.168.1.1/255.255.255.0                       ; IP der FritzBox
type = friend
contect=phones
insecure = invite,port
authuser = alarmanruf                                  ; Von der SIP config in der FritzBox
username = alarmanruf                                  ; Von der SIP config in der FritzBox
fromuser = alarmanruf                                  ; Von der SIP config in der FritzBox
fromdomain = fritz.box
secret = ppppp                                         ; Von der SIP config in der FritzBox
host = 192.168.1.1                                     ; FritzBox
```

In der Fritz!Box 6591 Cable ist die folgende Configuration für das Telefon hinterlegt worden:
```ruby
Name:                alarmanruf
Anschluss:           LAN/WAN
Rufnummer ausgehend: 02xxxxxxx1      ; freie ISDN Nummer
Rufnummer ankommend: 02xxxxxxx1      ; freie ISDN Nummer
intern:              **620
```
Sowie den Anmeldedaten:

Verwenden Sie die folgenden Anmeldedaten, um Ihr IP-Telefon an der FRITZ!Box anzumelden.
```ruby
Registrar:     fritz.box oder 192.168.1.1
Benutzername:  alarmanruf
Kennwort:      ppppp                       ; secret = ppppp
```

Wenn keine freie ISDN Nummer mehr vorhanden ist, kann auch über www.SipGate.de eine kostenfreie Lokale Nummer eingerichtet werden.
