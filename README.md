# RPi5 HAT+ for GNS AIS and ADSB Modules

# Allgemein
Schaltplan und Platinendesign mit KiCAD 9.0.  
Im Repo zu finden sind die Projektdateien sowie footprints und symbole der Bauteile der Firma GNS.  
Es gibt ebenso eine Adapterplatine um die AIS/ADSB-Platinen mit allen Pins auf eine Pfostenleiste zu bekommen um z.B. einen ESP32/Arduino o.ä. zu verwenden.  


# Disclaimer
Die Designs sind noch ungeprüft und befinden sich zur Zeit in der Platinenfertigung. Es ist daher ein starker 'work in progress' ohne validierte Layouts.  
**Ich bin eine Privatperson und stehe nicht mit Firma GNS, rpi oder anderen in Verbindung**  
**Für Schäden übernehme ich keine Haftung**  

# Beschreibung der Platine
Benutzt werden GNS701 als GPS-Empfänger mit PPS Ausgang, GNS5851 als AIS-Empfänger sowie GNS5894T als ADSB Empfänger.  
Des Weiteren sind ein EEPROM für die HAT+ Erkennung, sowie ein I2C GPIO-Expander (PCF8574AT) vorhanden um die Reset/Steuereingänge anzusteuern.  
Die Adresse des GPIO-Expanders kann über DIP-Schalter frei gewählt werden.  
Das EEPROM (z.B. CAT24C128) hat eine fixe Adresse.  

# Verdrahtung
Der ADSB-Empfänger hängt an UART1. Hier werden die Flusssteuerung über CTS/RTS (GPIO 2-3) mit angesteuert.
Der AIS-Empfänger hängt an UART3.  
Der GPS-Empfänger hängt an UART4, PPS an GPIO26/Pin 37.  

# Konfiguration
die HAT+ Erkennung bewirkt zur Zeit keine funktionalen benefits, daher müssen alle Einstellungen/Programme selbst getätigt/geladen werden


**config.txt**
```
dtoverlay=pps-gpio,gpiopin=26  
dtoverlay=uart1-pi5,ctsrts  
dtoverlay=uart4-pi5  
dtoverlay=uart5-pi5
```




Das HAT sollte HAT+ Spezifikationen erfüllen, da dies aber nicht bestätigt ist, sind im Design keine HAT+ Beschriftungen/Logos.

Linksammlung:

+ https://datasheets.raspberrypi.com/hat/hat-plus-specification.pdf
+ GPS: https://www.gns-electronics.de/product/gns-701/
+ AIS: https://www.gns-electronics.de/product/gns-5851-ais-module/
+ ADS-B: https://www.gns-electronics.de/product/gns-5894-highperformance-ads-b-modul/

