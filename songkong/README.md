# Raspberry SongKong
[SongKong](http://www.jthink.net/songkong/) ist eine Appikation, die Musik erkennt und automatisch taggt - drüberschauen schadet aber durchaus nicht! (z.B. mit [Jaikoz](http://www.jthink.net/jaikoz/) - Mir ist aufgefallen, dass der Raspberry ziemlich "schauft", wenn Musik taggt - deshalb hab ich ihm ein eigenes Mainboard spendiert.

Konfigurationsdateien für Raspberry SongKong Station - Schritt für Schritt

## Hardware
Raspberry Pi 4 (4GB)
Crucial BX500 2.5 SSD (120GB)
Orico 2.5 Inch Hard Drive Enclosure (SATA -> USB)

## allgemeine Infos
--> Installtion des Betriebssystems und SSD als Root [siehe](https://github.com/gecompat/Raspi/blob/master/System_Default/README.md) 

IPv6 lasse ich bei diesem Gerät eingeschaltet

## allgemeine "Vorarbeiten"
### Hostname setzen
Da der Hostname bei der "Grundinstallation" ja nicht verändert wurde (raspberripi), hole ich das jetzt für dieses System nach. `sudo raspi-config`  und unter "2 Network Options / Hostname" trage ich `songkong` (analog zu meinem über DHCP vergebenen Hostname) ein.
Danach verlangt raspi-config nach einem Reboot, welches ich ausführe.

Weil ich auf meinen externen Platten erkennen möchte, um welches Sysem es sich handelt, erstelle ich noch im Root Verzeichnis ein File namens SongKong.
```bash
sudo touch /SongKong
```

## NAS-MountPoints fixe MountPoints einrichten
meine MusikSammlung und sonstige benötigte Shares binde ich fix ein (also nicht wie bisher über Automount). Dazu
```bash
sudo systemctl stop NAS-N1-Media-Musik.automount
sudo systemctl disable NAS-N1-Media-Musik.automount
sudo systemctl enable NAS-N1-Media-Musik.mount
sudo systemctl start NAS-N1-Media-Musik.mount
sudo systemctl status NAS-N1-Media-Musik.mount
```
Wenn das alles funktioniert hat, dann sollte man mittels `ls -la /NAS/N1/Media/Musik` auch die Daten sehen.

## Java (Headless
```bash
sudo apt install default-jdk
```

Wenn alles geklappt hat kann sollte `java -version` in etwa so einen Output liefern:
```bash
openjdk version "11.0.7" 2020-04-14
OpenJDK Runtime Environment (build 11.0.7+10-post-Raspbian-3deb10u1)
OpenJDK Server VM (build 11.0.7+10-post-Raspbian-3deb10u1, mixed mode)
```


## SongKong
Java muss installiert sein 
```bash
cd /opt 
http://www.jthink.net/songkong/de/download.jsp Headless! -> Link ermitteln 
wget <Link>
sudo mv <Link> SongKong_6.9.3_Mercury_20200515 
sudo tar -xvzf SongKong_6.9.3_Mercury_20200515 
sudo chown -R pi:pi songkong 
cd /opt/songkong
```

### über systemd starten
Damit SongKong auch automatisch gestartet wird, benötge ich noch folgendes systemd Konfigfile
```bash
sudo nano /etc/systemd/system/songkong.service
-------------- SNIP ----------------
[Unit]
	Description=songkong auto start
	After=multi.user.target

[Service]
	Type=simple
	User=pi
	WorkingDirectory=/opt/songkong
	ExecStart=/bin/sh /opt/songkong/songkongremote.sh
	Restart=always
	RestartSec=60
[Install]
	WantedBy=multi-user.target
-------------- SNIP ----------------
```
oder über Repository
```bash
sudo cp ~/Repo/Raspi/songkong/etc/systemd/system/* /etc/systemd/system/
sudo chmod 644 /etc/systemd/system/songkong.service
```

Aktivieren und starten
```bash
sudo systemctl enable songkong.service
sudo systemctl start songkong.service
```

Jetzt sollte man (nach einer kleinen Vorlaufzeit) mittels eines Browsers auf `http://songkong:4567` zugreifen.


### Sicherung der Config mittels Git
Damit nur das gesichert wird, was wirklich benötigt ist:
```bash
sudo systemctl stop songkong.service
cd /home/pi
mv .songkong <Verzeichnis im Repository>
ln -s <Verzeichnis im Repository> /home/pi/.songkong
```
Um gewisse Unterverzeichnisse vom Git auszuschließen muss man das File .gitignore anlegen und editieren sieh [hier](https://github.com/gecompat/Public_RaspiConf/tree/master/System_Default#sicherung-der-config-files-mittels-git).


### Log und Config am Share "publizieren"
Damit ich auch vom Laptop aus einen Log ansehen kann, einfach meine Einstellungen und Reports sichern kann, leite ich das alles  auf mein lokales Verzeichnis, das mittels Samba freigegeben ist um.
Dazu muss sichergestellt sein, dass `follow symlinks = yes` in der smb.conf aktiviert ist.
```bash
ln -s <Verzeichnis im Repository> /smbshare/songkong
```

