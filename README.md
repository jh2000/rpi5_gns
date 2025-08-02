# RPi5 HAT+ for GNS AIS and ADS-B Modules

# Allgemein
Schaltplan und Platinendesign mit KiCAD 9.0.  
Im Repo zu finden sind die Projektdateien sowie footprints und symbole der Bauteile der Firma GNS.  
Die RPi HAT Footprints und Symbole stammen aus einem anderen Repo, siehe Links.  
Es gibt ebenso eine Adapterplatine um die AIS/ADSB-Platinen mit allen Pins auf eine Pfostenleiste zu bekommen um z.B. einen ESP32/Arduino o.ä. zu verwenden.  
GNS bietet verschiedene Stufen von Features, die Platinen sind Pinkompatibel, somit lassen sich alternativ zum GNS5894T auch GNS5894TAC oder der abgespeckte GNS 5892R verwenden.  
Das GPS-Modul ist nicht Pinkompatibel mit den anderen. Für AIS gibt es zur Zeit nur ein Modul.  
Anstelle eines RPi5 kann weitestgehend auch ein RPi4 verwendet werden. Es gibt aber im Detail Unterschiede zu beachten, auf die ich hier erst mal nicht näher eingehe.


# Pro&Contra
+ \+ CPU quasi idle
+ \+ dadurch wesentlich energieeffizienter gegenüber SDR/DVB-Sticks
+ \+ mehr Platz für SDR für Schiff-&Flugfunk
+ \- hoher Preis
+ \- nicht alle Feeder kompatibel

# Disclaimer
Die Designs sind noch ungeprüft und befinden sich zur Zeit in der Platinenfertigung. Es ist daher ein starker 'work in progress' ohne validierte Layouts.  
**Ich bin eine Privatperson und stehe nicht mit Firma GNS, rpi, jlcpcb oder Anderen in kommerzieller Verbindung**  
**Für Schäden übernehme ich keine Haftung**  

# Beschreibung der Platine
Benutzt werden GNS701 als GPS-Empfänger mit PPS Ausgang, GNS5851 als AIS-Empfänger sowie GNS5894T als ADSB Empfänger.  
Des Weiteren sind ein EEPROM für die HAT+ Erkennung, sowie ein I2C GPIO-Expander (PCF8574AT) vorhanden um die Reset/Steuereingänge anzusteuern.  
Die Adresse des GPIO-Expanders kann über DIP-Schalter frei gewählt werden.  
Das EEPROM zur automatschen HAT Erkennung (z.B. CAT24C128) hat Lötpads um den schreibschutz und die Adresse zu steuern. Standardmäßig ist 0x50. Gem. spezifikation würde auch 0x51 gehen.
Sollte das HAT zusammen mit anderen Verwendet werden muss geprüft werden, dass es keinen Konflikt der Adresse oder Pins gibt.
Aus Platz- und Komplexitätsgründen habe ich auf viele Bauteile verzichtet. Entsprechend fehlen Stütz-/Filterkapazitäten, Pegelbegrenzer usw.  
Es lassen sich theoretisch 2 der AIS Empfänger Chips Daisy-Chainen um auf Kanal A und B zeitgleich zu lauschen. Mit mehr Durchkontaktierungen und den Signalteilern/Filtern/Verstärkern dürfte es aber auch noch drauf passen. 

# Verdrahtung
Der ADSB-Empfänger hängt an UART1. Hier werden die Flusssteuerung über CTS/RTS (GPIO 2-3) mit angesteuert.  
Der AIS-Empfänger hängt an UART3.  
Der GPS-Empfänger hängt an UART4, die PPS Funktion an GPIO26/Pin 37.  

# Konfiguration
die HAT+ Erkennung hat zur Zeit keine funktionalen Benefits, daher müssen alle Einstellungen/Programme selbst getätigt/geladen werden.  
Sobald die Hardware lauffähig getestet ist lassen sich zumindest die overlay Anteile und Pinsteuerungen so automatisieren.
Ich verwende Debian 12.

**/boot/config.txt**
```
dtoverlay=pps-gpio,gpiopin=26  
dtoverlay=uart1-pi5,ctsrts  
dtoverlay=uart4-pi5  
dtoverlay=uart5-pi5
```
**/etc/default/gpsd**
```
DEVICES="/dev/ttyAMA4 /dev/pps0"
GPSD_OPTIONS="-s 9600 -n"
```

**/etc/ntpsec/ntp.conf***
```
server 127.127.22.0 minpoll 4 maxpoll 4
fudge 127.127.22.0 refid PPS
fudge 127.127.22.0 flag3 1 # enable kernel PLL/FLL clock discipline

server 127.127.28.0 minpoll 4 maxpoll 4 prefer # PPS requires at least one preferred peer
fudge 127.127.28.0 refid GPS
fudge 127.127.28.0 time1 +0.130 # coarse offset due to the UART delay

```
**/etc/ser2net.yaml**
```
connection: &adsb1
    accepter: tcp,4000
    enable: on
    connector: serialdev,
      /dev/ttyAMA5,
              921600n81,local

connection: &ais1
    accepter: tcp,2001
    enable: on
    options:
      kickolduser: true
    connector: serialdev,
              /dev/ttyS0,
              115200n81,local

```

Die Module geben über die UART-Schnittstelle standardmäßig (lesbaren) Text aus. Es kann für ADS-B aber auch das HULC/Beast Binärformat anstelle von AVR benutzt werden.
Alle üblichen Programme können die Daten entsprechend lesen, bis auf modesdeco2, welches nur funkrohdaten verarbeiten kann.  

Erfolgreich getestete feeder für AIS mit dem GNS5851:  
+ sxfeeder
+ AIS-Catcher

Erfolgreich getestete feeder für ADS-B mit dem GNS5894T:  
+ rbfeeder (radarbox)
+ feed-adsbx (adsbexchange)
+ mlat-client (adsbexchange)
+ fr24feed (flightradar)

Das GPS-Modul bietet GPS und PPS Daten für z.B. gpsd. Die Daten werden ebenso an den ADS-B Empfänger gegeben.  


In der Zielarchitektur verwende ich gpsd, ser2net und alle Feeder der jeweiligen Platformen.  
Dabei werden die AIS und ADS-B rohdaten via ser2net vom jeweiligen primärprogramm (AIS-Catcher, rbfeeder) gelesen und stellen diese Daten dann an sekundärfeeder (sxfeeder, feed-adsb usw...) bereit.
ser2net hat sich als praktisch erwiesen, da es sich einfach debuggen lässt indem man z.b. per nc oder telnet verbinden kann.  
gpsd und ntpd stellen dem System die genauen Zeitdaten bereit. Wer es richtig übertreiben will, kann das Design anpassen und einen RTK fähigen GPS Empfänger einbauen.

Das HAT sollte HAT+ Spezifikationen erfüllen, dies ist aber noch nicht bestätigt, im Design ist testweise ein HAT+ Logos angebracht.

Im aktuellen Design gegenüber dem referenzentwurf fehlt der Schlitz um an einen der CSI Ports zu kommen. Da dies wohl Bestandteil der HAT+ Hardwareforderungen ist, muss ich die Bauteile noch so schieben, dass auch dies gegeben ist.

# Linksammlung:

+ https://datasheets.raspberrypi.com/hat/hat-plus-specification.pdf
+ GPS: https://www.gns-electronics.de/product/gns-701/
+ AIS: https://www.gns-electronics.de/product/gns-5851-ais-module/
+ AIS Daisy-Chaining: https://www.gns-electronics.de/wp-content/uploads/2025/05/GNS5851_AppNote_2ch-Receiver-V0.9.pdf
+ ADS-B: https://www.gns-electronics.de/product/gns-5894-highperformance-ads-b-modul/
+ RPi HAT Template: https://github.com/devbisme/RPi_Hat_Template
+ Prototypenfertiger: https://jlcpcb.com/
