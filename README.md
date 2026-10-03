# RPi-headless-Konfiguration

Die SD-Karte unter Windows oder einem anderen Betriebssystem als Laufwerk öffnen und die beiden Dateien ssh und wpa_supplicant.conf im Root-Verzeichnis der SD-Karte ablegen. Die ssh-Datei aktiviert beim Booten der Raspberry Pi SSH ohne dass man über das raspi-config Menü vorher gehen muss. Die wpa_supplicant.conf beinhaltet die Einstellungen für das Netzwerk. Im aktuellen Beispiel müssen für eine WPA2-Verschlüsselung nur die SSID (Netzwerkname) und das Passwort (PSK) eingetragen werden.

Weitere Informationen findet man [hier](https://wiki.ubuntuusers.de/WLAN/wpa_supplicant/)

Per SSH kann man sich mit der Raspberry Pi nach dem Boot-Vorgang verbinden. Die WLAN-Verbindung wird während dem Bootvorgang ins Verzeichnis /etc/network/interfaces geschoben und automatisch gespeichert. Die SSH-Verbindung kann wie folgt aufgebaut werden:

```
ssh <benutzername>@<ip_address>
```

Die IP-Adresse kann man über ein Netzwerkscan, wie bspw. durch den [Advanced IP Scanner](https://www.advanced-ip-scanner.com/de/), herausfinden.

---

## Weitere Konfigurationen

### Passwort ändern

Hier geht man über die `/etc/shadow` Datei:

```
# Neuen Passwort-Hash generieren
openssl passwd -6 NEUES_PASSWORT

# Shadow-Datei bearbeiten
sudo nano /mnt/sdcard/etc/shadow
```

In der Shadow-Datei die Zeile für den Benutzernamen (z.B. `pi`) suchen und den Hash (zwischen erstem und zweitem `:`) ersetzen:

```
pi:$6$DEIN_NEUER_HASH:...:
```

### Hostnamen ändern

```
echo "Hostname" | sudo tee /mnt/sdcard/etc/hostname

# Auch in /etc/hosts anpassen
sudo nano /mnt/sdcard/etc/hosts
# Zeile ändern: 127.0.1.1   Hostname
```

### SSH aktivieren

```
# Leere Datei erstellt SSH-Aktivierung beim ersten Boot
sudo touch /mnt/sdcard/boot/ssh
# oder je nach Pi- oder Ubuntu-Image:
sudo touch /mnt/sdcard/boot/firmware/ssh
```

Könnte auch unter `DietPi` funktionieren.

### WLAN konfigurieren

Für `Ubuntu` (nicht Raspberry Pi OS) wird `netplan` verwendet:

```
network:
  version: 2
  wifis:
    wlan0:
      dhcp4: true
      optional: true
      access-points:
        "Your_WiFi_SSID":
          password: "Your_WiFi_password"
```

### Zeitzone

```
sudo ln -sf /usr/share/zoneinfo/Europe/Berlin /mnt/sdcard/etc/localtime
echo "Europe/Berlin" | sudo tee /mnt/sdcard/etc/timezone
```

### Tastaturlayout

```
sudo nano /mnt/sdcard/etc/default/keyboard
```

```
XKBMODEL="pc105"
XKBLAYOUT="de"
XKBVARIANT=""
XKBOPTIONS=""
```

### HDMI-Konfiguration für Raspberry Pi

```
sudo nano /mnt/sdcard/boot/firmware/config.txt
```

```
# HDMI Ausgabe erzwingen
hdmi_force_hotplug=1

# HDMI-Modus (falls kein Signal erkannt wird)
hdmi_drive=2          # 2 = HDMI-Modus (mit Audio), 1 = DVI-Modus

# Auflösung erzwingen (optional, 1080p Beispiel)
hdmi_group=1          # 1 = CEA (TV), 2 = DMT (Monitor)
hdmi_mode=16          # CEA 16 = 1080p 60Hz

# HDMI Boost (bei schwachem Signal / langen Kabeln)
config_hdmi_boost=4   # Werte: 0-7
```

```
sudo nano /mnt/sdcard/boot/firmware/cmdline.txt
```

Folgende Parameter **in die bestehende Zeile** einfügen (alles muss in **einer Zeile** bleiben!):

```
console=tty1 video=HDMI-A-1:1920x1080@60
```

Beispiel wie die Zeile aussehen könnte:

```
console=serial0,115200 console=tty1 video=HDMI-A-1:1920x1080@60 root=PARTUUID=... rootfstype=ext4 fsck.repair=yes rootwait
```

#### Häufige hdmi_mode Werte

| Modus | Auflösung |
|---|---|
| `hdmi_group=1` `hdmi_mode=4` | 720p 60Hz |
| `hdmi_group=1` `hdmi_mode=16` | 1080p 60Hz |
| `hdmi_group=2` `hdmi_mode=82` | 1080p 60Hz (Monitor) |
| `hdmi_group=2` `hdmi_mode=85` | 720p 60Hz (Monitor) |

> **Tipp:** Bei einem normalen PC-Monitor `hdmi_group=2` verwenden, bei einem TV `hdmi_group=1`.
