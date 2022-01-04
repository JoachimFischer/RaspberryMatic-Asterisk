# RaspberryMatic-Asterisk
RaspberryMatic kann Telefonanrufe über die PBX Asterisk ausführen für Alarmmeldungen

## Schritt 1:
Installation vom Asterisk Server auf dem Raspberry
```ruby
su root
sudo apt-get install asterisk
```

Test der Asterisk installation:
```ruby
root@fipbox:~# asterisk -c
Asterisk already running on /var/run/asterisk/asterisk.ctl.  Use 'asterisk -r' to connect.
```

## Schritt 2:
Verwenden Sie die folgenden Anmeldedaten, um Ihr IP-Telefon (SIP) an der FRITZ!Box anzumelden.
In der Fritz!Box 6591 Cable ist die folgende Configuration für das SIP-Telefon hinterlegt worden:
```ruby
Name:                alarmanruf
Anschluss:           LAN/WAN
Rufnummer ausgehend: 02xxxxxxx1      ; freie ISDN Nummer
Rufnummer ankommend: 02xxxxxxx1      ; freie ISDN Nummer
intern:              **620
```
Sowie den Anmeldedaten:
```ruby
Registrar:     fritz.box oder 192.168.1.1
Benutzername:  alarmanruf
Kennwort:      ppppp                       ; secret = ppppp
```

Wenn keine freie ISDN Nummer mehr vorhanden ist, kann auch über www.SipGate.de eine kostenfreie Lokale Nummer eingerichtet werden.

## Schritt 3:
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

## Schritt 4:
Die Dateien liegen als templete unter ``` /etc/asterisk ```. Hier wird eine weitere Datei mit dem Namen extensions.conf wird agelegt. Diese Datei enthält die Information, welche WAV Datei abgespielt werden soll. In diesem Fall ist es die Datei alarmmeldung, die als Sounddatei alammeldung.wav in das Verzeichnis ```/var/lib/asterisk/sounds/my/``` kopiert wird. 

Eine beliebige Sound Datei kann mit Text to Speech erstellt werden, z.B. hier https://ttsfree.com/text-to-speech/german
Wenn man eine Sounddateie im mp3 Format hat, muss diese umgewandelt werden in ein wav Format mit dem folgenden Befehl auf dem Raspberry:
```ruby
ffmpeg -i /tmp/mySpeech.mp3 -ar 8000 -ac 1 -ab 64k /var/lib/asterisk/sounds/my/mySpeech.wav
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

## Schritt 5:
Wenn die SIP eingerichtet ist, kann der Asterisk Server getestet werden mit den folgenden Befehlen:
```ruby
asterisk -r                                                                                      ; den Befehl absetzen
Asterisk 16.2.1~dfsg-1+deb10u2, Copyright (C) 1999 - 2018, Digium, Inc. and others.
Created by Mark Spencer <markster@digium.com>
Asterisk comes with ABSOLUTELY NO WARRANTY; type 'core show warranty' for details.
This is free software, with components licensed under the GNU General Public
License version 2 and other licenses; you are welcome to redistribute it under
certain conditions. Type 'core show license' for details.
=========================================================================
Connected to Asterisk 16.2.1~dfsg-1+deb10u2 currently running on fipbox (pid = 490)               ; anschließend startet die CLI

fipbox*CLI> sip show registry                                                                     ; im CLI den Befehl absetzen
Host                                    dnsmgr Username       Refresh State                Reg.Time                 
192.168.1.1:5060                        N      alarmanruf         285 Registered           Tue, 04 Jan 2022 19:40:21

fipbox*CLI> sip show peers
Name/username             Host                                    Dyn Forcerport Comedia    ACL Port     Status      Description                      
alarmanruf/alarmanruf     192.168.1.1                                 Yes        Yes         A  5060     Unmonitored                                  
1 sip peers [Monitored: 0 online, 0 offline Unmonitored: 1 online, 0 offline]

```

## Schritt 6:
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




