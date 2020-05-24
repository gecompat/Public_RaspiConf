# Raspberry Pi 4 Default Installation
Konfigurationsdateien für Raspberry SongKong Station - Schritt für Schritt

## Hardware
Raspberry Pi 4 (4GB)
Crucial BX500 2.5 SSD (120GB)
Orico 2.5 Inch Hard Drive Enclosure (SATA -> USB)
SanDisk 64GB Ultra

## SD-Card
mittels [Raspberri Pi Imager](https://www.raspberrypi.org/downloads/) Raspian-Lite auf eine SSD schreiben.
ein File Namens `SSH` in der `boot` Partition anlegen, damit man über z.B. [Putty](https://www.putty.org/) via SSH-Connection zugreifen kann.

Ich lege mir zusätzlich immer noch ein File ab, das den hostnamen wiederspiegelt - dann erkenne ich auch gleich, um welche SD-Karte es sich handelt, wenn ich sie mal so "rumliegen" habe `hostname_xyz`

Weiters trage ich gleich in der `config.txt` folgende Zeilen ein.
```bash
max_usb_current=1
gpu_mem=16
```
(Erste Zeile dient dazu, dass die USB Geräte (hoffentlich) genügend Strom bekommen. Die Zweite reduziert den Speicherbedarf für die Grafikkarte - also diese weglassen, wenn man eine grafische Oberfläche haben will ;-)

Da ich auch die externe SSD Festplatte ([Crusial BX500 120GB](https://www.amazon.de/gp/product/B07G3KRZBX/ref=ppx_yo_dt_b_asin_title_o00_s02?ie=UTF8&psc=1) bzw. [240GB](https://www.amazon.de/gp/product/B07G3L3DRK/ref=ppx_yo_dt_b_asin_title_o01_s02?ie=UTF8&psc=1) bzw. [Western Digital WDS100T2B0A](https://www.amazon.de/gp/product/B073SBQMCX/ref=ppx_yo_dt_b_asin_title_o05_s00?ie=UTF8&psc=1)) mittels [Orico 2.5" Hard Drive Enclosure (2139U3)](https://www.amazon.de/gp/product/B07F3475HG/ref=ppx_yo_dt_b_asin_title_o00_s01?ie=UTF8&psc=1) anschließe, erweitere ich noch in `cmdline.txt` die Zeile (darf nur eine Zeile sein!) um folgenden Eintrag:
```bash
usb-storage.quirks=152d:0578:u
```
Sollte man ein anderes Gerät haben, kann man auch probieren das Ganze ohne entsprechenden Eintrag zu starten - bei mir MUSS dieser Quirk eingetragen sein. ([hier die Quelle](https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/) (heute nicht erreichbar, weil Zertifikat abgelaufen - werde somit eine Kopie direkt hier ablegen, damit die Informationen  gesichert sind - sobald erreichbar).

Falls man ein anderes Gerät hat. über `sudo lsusb` findet man die "Nummer" raus
```bash
pi@raspberrypi:~ $ lsusb
Bus 002 Device 002: ID 152d:0578 JMicron Technology Corp. / JMicron USA Technology Corp. JMS567 SATA 6Gb/s bridge
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Wenn das erledigt ist, SD-Card in den Raspberry stecken, mit Netzwerk verbinden und (sicherheitshalber das Erste mal ohne angeschlossener SSD) starten.

Frühestens, wenn sich der Raspberry eine IP-Adresse "geholt" hat, kann man sich über SSH anmelden. `User=pi, Password=raspberry`.

## Geschwindikeitstest 1 - mit SD-Card
Weil es einfach spannender ist, wenn man sieht, dass sich was verbessert:
```bash
cd ~
sudo curl https://raw.githubusercontent.com/TheRemote/PiBenchmarks/master/Storage.sh | sudo bash
```
liefert eine Kennzahl, mit der man recht gut vergleichen kann. (dieser Test dauert noch recht lange - wird aber mit einer SSD dann deutlich schneller ;-)

Ergebnis mit [SanDisk SDSQUAR-064G-GZFMA, Ultra](https://www.amazon.de/SanDisk-Ultra-microSDXC-Speicherkarte-Adapter/dp/B073SB2L3C/ref=sr_1_3?__mk_de_DE=%C3%85M%C3%85%C5%BD%C3%95%C3%91&dchild=1&keywords=SDSQUAR-064g-gzfma&qid=1590254582&s=computers&sr=1-3)
```bash
     Category                  Test                      Result
---------------------------------------------------------------------------     
HDParm                    Disk Read                 40.97 MB/s
HDParm                    Cached Disk Read          39.58 MB/s
DD                        Disk Write                17.4 MB/s
FIO                       4k random read            2807 IOPS (11228 KB/s)
FIO                       4k random write           838 IOPS (3352 KB/s)
IOZone                    4k read                   8966 KB/s
IOZone                    4k write                  2859 KB/s
IOZone                    4k random read            7792 KB/s
IOZone                    4k random write           3287 KB/s

                          Score: 1132
```


## System upgraden und Grundeinstellungen
Da ich mir die SD-Karte nicht nur für einen Raspberry herrichte, sondern eine für alle verwenden will (haben alle die gleiche Hardware), versuche ich wirklich nur das zu installieren, was alle Syteme benötigen. - Also setze ich hier auch keinen Hostnamen, sondern lasse den Default `raspberrypi`.


### Repository SRC aktivieren und System updaten
Kommentar von deb-src entfernen
```bash
sudo nano /etc/apt/sources.list
------------- SNIP -------------
deb http://raspbian.raspberrypi.org/raspbian/ buster main contrib non-free rpi
# Uncomment line below then 'apt-get update' to enable 'apt-get source'
deb-src http://raspbian.raspberrypi.org/raspbian/ buster main contrib non-free rpi
------------- SNIP -------------
```

Für alle, die nano nicht kennen - mittels `CTRL-X` kommt in den Speichern Dialog - mit `Y` wird gespeichert.

```bash
sudo -s
apt-get update && apt-get upgrade && apt-get dist-upgrade && apt-get full-upgrade
exit
```


### raspi-config
`sudo raspi-config` starten und folgende Einstellungen anpassen:
- 1 Change User Password 
- 2 Network Options / Hostname (ich lasse das hier absichtlich aus - wie oben geschrieben dient diese SD-Card als Vorlage für alle Systeme)
- 2 Network Options / Network interface names predictable -> yes
- 3 Boot Options / Desktop / CLI / Console Autologin (ich will automatisch angemeldet werden - falls ich irgendwann ins System muss, kein Passwort mehr weiß, kann ich so einfach an Monitor anschließen und alles ist wieder in Ordnung)
- 3 Boot Options / Wait for Network at Boot -> yes
- 4 Localisation Options / Change Locale / de_DE.UTF-8 UTF-8 aktivieren, und als default locale setzen
- 4 Localisation Options / Change Timezone -> Europe / Vienna 
- 4 Localisation Options / Change WLAN Country -> AT
- 5 Interfacing Options / SSH -> yes (ist bereits gesetzt - File SSH)
- 7 Advanced Options / Expand Filesystem 
- 7 Advanced Options / Memory Split -> 16   (ist bereits gesetzt - Edit von config.txt)

reboot yes


## Git installieren
Die weiteren Files habe ich bereits in meinem Repository abgelegt, deshalb installiere ich mir auch gleich GIT. (Da ich u.U. auch mal größere Files hab, kommt gleich LFS dazu)
```bash
sudo apt-get install git-lfs
```
damit ich mich in Zukunft nicht andauernd authentifizieren muss:
```bash
git config --global credential.helper store
```
und noch meine Email und UserName:
```bash
git config --global user.email "xxx@xxxxxx.xxx"
git config --global user.name "xxxxxxxxxxxx"
```

Jetzt kann ich mir schon das Repository holen, in dem ich die weiteren Konfigurationen gespeichert hab. - Ich lege die Repositories unter `~/Repo/` ab.

```bash
mkdir ~/Repo
cd ~/Repo/
git clone http://<Git-UserName>:<Git-Password>@github.com/gecompat/Raspi ~~ZielVerzeichnis~~
```
wer will, kann auch ein Zielverzeichnis mitgeben -> sonst wird automatisch ein Verzeichnis mit dem Repository Namen erstellt.
```bash
git clone http://<Git-UserName>:<Git-Password>@github.com/gecompat/Raspi ~~ZielVerzeichnis~~
```

Bin mir zwar nicht sicher, ob man das wirklich machen muss, aber ich "installiere" noch lfs.
```
cd Raspi
git lfs install
```
Wenn noch kein .gitattributes vorhanden ist, in dem sowas steht:
```bash
------------- SNIP ----------------
*.db filter=lfs diff=lfs merge=lfs -text
*.iso filter=lfs diff=lfs merge=lfs -text
------------- SNIP ----------------
```
dann muss man noch die Dateitypen registrieren, die über lfs ins Repository kommen sollen.
```bash
git lfs track "*.iso"
git lfs track "*.db"
git add .
git commit -m "lfs aktiviert"
git push
```


## Samba Share erstellen
Damit ich mir Daten leicht von einem Raspi zum anderen kopieren kann - und das auch einfach von meinem Laptop aus über eingebundene Verzeichnisse, lege ich auf jedem Raspi einen Share an.
[Quelle](https://www.elektronik-kompendium.de/sites/raspberry-pi/2007071.htm)

zukünftiges Share Verzeichnis erstellen und Samba installieren
```bash
sudo mkdir /smbshare
sudo chown root:root /smbshare/
sudo chmod 777 /smbshare/
sudo apt-get update
--> ich übergebe mittels DHCP die WINS Server IP-Adresse (meine NAS)
sudo apt-get install dhcp-client
sudo apt-get install samba samba-common smbclient

---> abfrage, ob samba Informationen vom DHCP Server erhalten soll (NetBIOS-Name-Server)

prüfen ob läuft
sudo systemctl status smbd.service
sudo systemctl status nmbd.service

Hinweis:  Q  für exit!




sudo mv /etc/samba/smb.conf /etc/samba/smb.conf_alt
sudo nano /etc/samba/smb.conf
----------------- SNIP ------------------
[global]
workgroup = WORKGROUP
security = user
encrypt passwords = yes
client min protocol = SMB2
client max protocol = SMB3
allow insecure wide links = yes
----------------- SNIP ------------------

-> workgroup anpassen an eigene Umgebung ( bei mir PRIVAT)

check ob Config oK
testparm

sudo service smbd restart
sudo service nmbd restart




Share in smb.conf hinzufügen
sudo nano /etc/samba/smb.conf
----------------- SNIP ------------------
[xx_smbshare]
comment = Samba-Share auf Raspberry xx
path = /smbshare
read only = no
follow symlinks = yes
wide links = yes
----------------- SNIP ------------------


testparm
```

oder über Repository herholen
```bash
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf_orig
sudo cp ~/Repo/Raspi/System_Default/etc/samba/smb.conf /etc/samba/smb.conf
sudo chmod 644 smb.conf
```



```bash
sudo systemctl restart smbd.service
sudo systemctl restart nmbd.service

Samba Passwort einrichten
sudo smbpasswd -a pi
```

Hinweis für deaktivieren von Usern:

Benutzer "pi" für Samba deaktivieren:
`sudo smbpasswd -d pi`

Benutzer "pi" für Samba wieder aktivieren:
`sudo smbpasswd -e pi`


## Zugriff auf NAS einrichten
### Mount Points anlegen
Ich habe mehrere NAS, die ich N1 bis Nx benannt hab. - Für jeden SharePoint, den ich benötige, richte ich mir einen MountPoint ein. Alle nach dem Schema:
NAS/Nx/MountPoint

```
sudo -s
mkdir /NAS
mkdir /NAS/N1
mkdir /NAS/N1/Log
chmod -R 777 /NAS
exit
```

### Server Erreichbarkeit prüfen
#### Allgemeines Prüf-Script
Damit man das ShareVerzeichnis überhaupt mounten kann, muss selbstverständlich der Server (NAS) erreichbar sein - sonst kann nicht gemountet werden. Für diese Prüfung lege ich mir ein allgemeines Script an:

```bash
sudo nano /usr/local/sbin/server_up
--------- SNIP ----------
#!/bin/bash
#==============================================================================================
# serverctl by TomL*thlu.de
#
# Script-Name :  serverctl
# Version     :  1.0
# Date        :  31.07.2017
# Lizenz      :  GNU GPL3
# Description :  Check if given server is reachable
#==============================================================================================

[ -z "$1" ] && Server="8.8.8.8" || Server=$1
echo "active/running   Server=$Server" | systemd-cat -t "thlu:`basename $0`" -p "info"

timeout=85
Diff=0
HomeNetIsConnect=-1

Start=$(date +%s);

while [ true ]; do
   /bin/ping -c1 -W1 -q $Server &>/dev/null
   HomeNetIsConnect=$?

   [ $HomeNetIsConnect -eq 0 ] && break
   /bin/sleep 0.5

   End=$(date +%s);
   Diff=$((End-S
   tart))
   [[ Diff -gt timeout ]] && break
done
rc=0
if [[ $HomeNetIsConnect -eq 0 ]]; then
   echo "Host $Server is reachable! (RC:$HomeNetIsConnect, after $Diff Seconds wait)" | systemd-cat -t "thlu:`basename $0`" -p "info"
else
   echo "Host $Server is not reachable! (RC:$HomeNetIsConnect, after $Diff Seconds wait)" | systemd-cat -t "thlu:`basename $0`" -p "err"
   rc=1
fi

echo "Successful terminated with exitcode=$rc" | systemd-cat -t "thlu:`basename $0`" -p "info"
exit $rc
--------- SNIP ----------

-- root:root müsste es schon sein - nur zur Sicherheit
sudo chown root:root /usr/local/sbin/server_up
sudo chmod 755 /usr/local/sbin/server_up
```


#### Als Dienst aktivieren
```bash
sudo nano /etc/systemd/system/server_N1_up.service
--------- SNIP ----------
[Unit]
Description=server_N1_up.service:   Waiting for Network or Server to be up
After=network.target

[Service]
Type=oneshot
TimeoutStartSec=95
ExecStart=/usr/local/sbin/server_up N1.<FQDN>

[Install]
WantedBy=multi-user.target
--------- SNIP ----------
```
oder über Repository
```bash
sudo cp ~/Repo/Raspi/System_Default/etc/systemd/system/server_* /etc/systemd/system/
```
Rechte richten und Dienst aktivieren

```bash
sudo chmod 644 /etc/systemd/system/server_N*
-- je Serverprüfung
sudo systemctl enable server_N1_up.service
sudo systemctl start server_N1_up.service
sudo systemctl status server_N1_up.service
```

### Credential für Share ablegen
```bash
sudo nano /etc/smbcredentials_N1
--------- SNIP ----------
username=<@user@>
password=<password>
--------- SNIP ----------
```
oder über Repository
```bash
sudo cp ~/Repo/Raspi/System_Default/etc/smbcredentials_* /etc/
```
Rechte richten
```bash
sudo chmod 600 /etc/smbcredentials_*
```

### systemd Automount / Mount
Ich habe jetzt 2 Möglichkeiten die "Shares" zu mounten - entweder fix oder als Automount. Ich trage in jedem System jene Mount's als Automount an, die ich nicht fix für die Funktion des Raspberry benötige - z.B. SongKong greift auf meine MusikSammlung zu -> somit auf diesem Gerät /NAS/N1/Media/Musik als mount und nicht als automount.

Dafür muss ich in /etc/systemd/system/ ein xxx.automount anlegen Filename ist der MountPoint wobei jedes / ein "-" wird (Achtung bei Sonderzeichen und Blank!) - da muss speziell getaggt werden. Mit folgendem Befehl kann man ermitteln, wie getaggt werden muss!
```bash
systemd-escape --suffix=mount --path /NAS/Media/Ein\ langer\ Name/
```

Jetzt für jeden Mount-Point folgende 2 Files erstellen
```bash
nano /etc/systemd/system/NAS-N1-Log.automount

------------------ SNIP ------------------
[Unit]
        Description=cifs mount script
        Requires=network-online.target
        After=network-online.service

[Automount]
        Where=/NAS/N1/Log
        TimeoutIdleSec=10

[Install]
        WantedBy=multi-user.target
------------------ SNIP ------------------
```

```bash
nano /etc/systemd/system/NAS-N1-Log.mount
------------------ SNIP ------------------
[Unit]
        Description=cifs mount /NAS/N1/Log
        Requires=network-online.target server_N1_up.service
        After=network-online.service server_N1_up.service
        Conflicts=shutdown.service
        ConditionPathExists=/NAS/N1/Log
        
[Mount]
        What=//<FQDN>/Log
        Where=/NAS/N1/Log
        Options=credentials=/etc/smbcredentials_N1,uid=pi,vers=3.0,rw
        Type=cifs

[Install]
        WantedBy=multi-user.target
------------------ SNIP ------------------
```
oder über Repository
```bash
sudo cp ~/Repo/Raspi/System_Default/etc/systemd/system/NAS-* /etc/systemd/system/
```
Rechte richten
```bash
sudo chmod 644 /etc/systemd/system/NAS-*
```

Für meine SD-Card Vorlage mounte ich alle als Automount daher für jeden der Erstellten Mount-Points folgende Befehle:
```bash
sudo systemctl enable NAS-N1-Log.automount
sudo systemctl start NAS-N1-Log.automount 
```
Hinweis: wenn man sich das start sparen will, genügt enable und ein reboot ;-)

Hinweis: muss man mal was korrigieren in den systemd "Steuerdateien" muss man folgenden Befehl absetzten:
```bash
sudo systemctl daemon-reload
```


## von SSD booten
### SSD partitionieren
Als erstes muss die SSD partitioniert werden - Ziel ist es eine große Partition über die gesamte Platte anzulegen.
`sudo fdisk -l` zeigt die vorhandenen Disks und Partitionen - fast sicher ist die Festplatte ein `/dev/sda`
**ACHTUNG nicht `/dev/mmcblk0` das ist die SD-Karte!!!**

```bash
sudo fdisk /dev/sda
n new
p primary
default anwenden
x expert modus
i disk identifier
0xd34db33f
r expert modus verlassen
w write to disk
```
**ACHTUNG: wenn man die Festplatten parallel an einen PC anhängt, wird der durcheinander kommen, denn alle haben jetzt den gleichen Identifier - Vorteil ist aber, dass egal an welchen Raspberry man sie anhängt es immer so aussieht, als ob es die bekannte ist **

Den Identifier kann man mittels `sudo blkid` kontrollieren. Die Partitionierung kann man mittels `sudo fdisk -l` anzeigen lassen

### Partition formatieren
Die Partition muss jetzt noch formatiert werden
```bash
sudo mkfs.ext4 /dev/sda1
```

### Root Filesystem erstellen
Dazu mounte ich die Partition in ein neues Verzeichnis und kopiere dann mittels rsync die Daten. - Ich lösche jetzt noch schnell die Repositories, die ich für mich privat noch eingerichtet hab - damit das nicht fix auf meiner SD-Karte vorhanden ist. Achtung vorher nicht vergessen `git add .`, `git status`, `git commit -m "xxx"` und `git push`!
Repository löschen `rm -rf verzeichnis`

```bash
sudo mkdir /newdrive
sudo mount /dev/sda1 /newdrive
sudo rsync -avx / /newdrive
```

### von SSD starten
Damit ich mir sicher sein kann, dass ich auch das Root System von meiner SSD aktiv ist, erstelle ich auf auf der SSD noch ein File als Kennzeichen `sudo touch ROOT_SSD`

Damit die SSD als Root eingerichtet wird, muss das in der /boot/cmdline.txt angegeben werden. Das passiert indem man in der ersten Zeile folgendes zusätzlich einträgt: `root=/dev/sda1 rootfstype=ext4 rootwait`
**ACHTUNG: darauf achten, dass nicht noch ein root=xxxxxx bzw. die anderen Angaben schon vorhanden sind**

dann
```bash
sudo reboot
```

anmelden
```bash
ls -la /
```

wenn man da dann `ROOT_SSD` sieht, ist auch die SSD als root gemountet!

Fehlt nur noch das Aufräumen:
```bash
sudo rm -r /newdrive
sudo rm /ROOT_SSD
```

## Geschwindikeitstest 2 - mit SSD
```bash
cd ~
sudo curl https://raw.githubusercontent.com/TheRemote/PiBenchmarks/master/Storage.sh | sudo bash
```

```bash
     Category                  Test                      Result
HDParm                    Disk Read                 168.26 MB/s
HDParm                    Cached Disk Read          167.68 MB/s
DD                        Disk Write                183 MB/s
FIO                       4k random read            4903 IOPS (19612 KB/s)
FIO                       4k random write           6104 IOPS (24417 KB/s)
IOZone                    4k read                   26011 KB/s
IOZone                    4k write                  22322 KB/s
IOZone                    4k random read            19058 KB/s
IOZone                    4k random write           25946 KB/s

                          Score: 6520
```
Im Vergleich zu vorher:
```bash
     Category                  Test                      Result
---------------------------------------------------------------------------     
HDParm                    Disk Read                 40.97 MB/s
HDParm                    Cached Disk Read          39.58 MB/s
DD                        Disk Write                17.4 MB/s
FIO                       4k random read            2807 IOPS (11228 KB/s)
FIO                       4k random write           838 IOPS (3352 KB/s)
IOZone                    4k read                   8966 KB/s
IOZone                    4k write                  2859 KB/s
IOZone                    4k random read            7792 KB/s
IOZone                    4k random write           3287 KB/s

                          Score: 1132
```


Nicht nur die Geschwindigkeit ist etwas, das überzeugt, sondern auch der Vorteil, dass eine SSD deutlich langlebiger ist als eine SD-Card. Somit sollte mein System sicher sein.





# Sicherung der Config Files mittels GIT
`.gitignore` Inhalt
Alle Files und Unterverzeichnisse ausschließen: `*`
Alle Unterverzeichnisse ausschließen `*/*`

Wenn das File angelegt ist, kann man mittels `git check-ignore -v *` prüfen, ob irgend ein File / Sub-Folder in dem man gerade steht ausgeschlossen wird. [Quelle](https://git-scm.com/docs/git-check-ignore)
