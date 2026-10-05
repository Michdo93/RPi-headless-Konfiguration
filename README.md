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

### Neuen Benutzer anlegen (vor dem ersten Boot)

Seit Raspberry Pi OS Bullseye (April 2022) gibt es keinen Standardbenutzer `pi` mit Passwort `raspberry` mehr. Ohne vorkonfigurierten Benutzer startet beim ersten Boot ein Einrichtungsassistent, der headless nicht bedienbar ist. Der Benutzer muss deshalb vorher auf der SD-Karte angelegt werden.

#### Variante 1: `userconf.txt` (Raspberry Pi OS)

Im Root-Verzeichnis der Boot-Partition (unter Windows das sichtbare FAT-Laufwerk) eine Datei `userconf.txt` mit genau einer Zeile ablegen:

```
benutzername:VERSCHLÜSSELTES_PASSWORT
```

Den Passwort-Hash erzeugt man so:

```
openssl passwd -6 NEUES_PASSWORT
```

Beispiel:

```
echo "michael:$(openssl passwd -6 'MeinPasswort')" | sudo tee /mnt/sdcard/boot/firmware/userconf.txt
# bei älteren Images: /mnt/sdcard/boot/userconf.txt
```

Beim ersten Boot wird der Standardbenutzer (UID 1000) auf diesen Namen umbenannt, das Passwort gesetzt und das Home-Verzeichnis `/home/benutzername` angelegt. Die Datei wird danach automatisch gelöscht. Der Benutzer ist in den üblichen Gruppen (`sudo`, `gpio`, `video`, …).

> **Hinweis:** `userconf.txt` legt keinen *zusätzlichen* Benutzer an, sondern ersetzt den ersten Benutzer. Für weitere Benutzer siehe Variante 3.

#### Variante 2: `cloud-init` (Ubuntu)

Ubuntu-Images lesen beim ersten Boot die Datei `user-data` auf der Boot-Partition (`system-boot`). Diese Datei z. B. so anpassen:

```yaml
#cloud-config
users:
  - default                # weglassen, wenn der Benutzer "ubuntu" nicht angelegt werden soll
  - name: michael
    gecos: Michael
    shell: /bin/bash
    groups: [adm, sudo, dialout, video, plugdev]
    lock_passwd: false
    passwd: "$6$DEIN_HASH"   # mit openssl passwd -6 erzeugen
    sudo: "ALL=(ALL) ALL"
    ssh_authorized_keys:
      - ssh-ed25519 AAAA... michael@laptop

ssh_pwauth: true
```

Home-Verzeichnis, Gruppen und SSH-Key werden automatisch angelegt. `cloud-init` läuft nur beim ersten Boot. Soll die Datei nach dem Booten erneut ausgewertet werden, muss vorher `sudo cloud-init clean` ausgeführt werden.

#### Variante 3: Manuell über `/etc/passwd`, `/etc/shadow` und `/etc/group`

Funktioniert unabhängig von der Distribution (auch DietPi). Allerdings nur unter Linux, da die Root-Partition (ext4) gemountet sein muss.

```
ROOT=/mnt/sdcard
USER=michael
ID=1001        # muss frei sein, prüfen mit: cut -d: -f3 $ROOT/etc/passwd | sort -n | tail
HASH=$(openssl passwd -6 'MeinPasswort')
TAGE=$(( $(date +%s) / 86400 ))   # Datum der letzten Passwortänderung

# Benutzer, Passwort und Gruppe eintragen
echo "$USER:x:$ID:$ID:$USER,,,:/home/$USER:/bin/bash" | sudo tee -a $ROOT/etc/passwd
echo "$USER:$HASH:$TAGE:0:99999:7:::"                 | sudo tee -a $ROOT/etc/shadow
echo "$USER:x:$ID:"                                    | sudo tee -a $ROOT/etc/group
echo "$USER:!::"                                       | sudo tee -a $ROOT/etc/gshadow

# Home-Verzeichnis aus /etc/skel anlegen
sudo cp -a $ROOT/etc/skel $ROOT/home/$USER
sudo chown -R $ID:$ID $ROOT/home/$USER
sudo chmod 750 $ROOT/home/$USER

# sudo-Rechte vergeben
echo "$USER ALL=(ALL) ALL" | sudo tee $ROOT/etc/sudoers.d/010_$USER
sudo chmod 440 $ROOT/etc/sudoers.d/010_$USER
```

Optional einen SSH-Key hinterlegen:

```
sudo mkdir -p $ROOT/home/$USER/.ssh
cat ~/.ssh/id_ed25519.pub | sudo tee $ROOT/home/$USER/.ssh/authorized_keys
sudo chown -R $ID:$ID $ROOT/home/$USER/.ssh
sudo chmod 700 $ROOT/home/$USER/.ssh
sudo chmod 600 $ROOT/home/$USER/.ssh/authorized_keys
```

Weitere Gruppen (z. B. `gpio`, `video`, `dialout`) fügt man hinzu, indem man in `$ROOT/etc/group` den Benutzernamen an die jeweilige Zeile anhängt (`gpio:x:997:pi,michael`).

> **Wichtig:** Bei `chown` immer die numerische UID/GID verwenden. Der Benutzername existiert auf dem Host-System nicht oder hat dort eine andere ID.
