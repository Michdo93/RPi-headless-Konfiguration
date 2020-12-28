# RPi-headless-Konfiguration

Die SD-Karte unter Windows oder einem anderen Betriebssystem als Laufwerk öffnen und die beiden Dateien ssh und wpa_supplicant.conf im Root-Verzeichnis der SD-Karte ablegen. Die ssh-Datei aktiviert beim Booten der Raspberry Pi SSH ohne dass man über das raspi-config Menü vorher gehen muss. Die wpa_supplicant.conf beinhaltet die Einstellungen für das Netzwerk. Im aktuellen Beispiel müssen für eine WPA2-Verschlüsselung nur die SSID (Netzwerkname) und das Passwort (PSK) eingetragen werden.

Weitere Informationen findet man [hier](https://wiki.ubuntuusers.de/WLAN/wpa_supplicant/)

Per SSH kann man sich mit der Raspberry Pi nach dem Boot-Vorgang verbinden. Die WLAN-Verbindung wird während dem Bootvorgang ins Verzeichnis /etc/network/interfaces geschoben und automatisch gespeichert. Die SSH-Verbindung kann wie folgt aufgebaut werden:

```
ssh <benutzername>@<ip_address>
```

Die IP-Adresse kann man über ein Netzwerkscan, wie bspw. durch den [Advanced IP Scanner](https://www.advanced-ip-scanner.com/de/), herausfinden.
