# Raspi Headphones
Dieses Repository zeigt, wie man Headphones am Raspberry Pi installiert

## Grundinstallation analog z.B. [Songkong]

## Headphones

```bash
git clone https://github.com/rembo10/headphones /opt/headphones
sudo chown -R pi:pi /opt/headphones
```

### automatischer Start über systemd
als Vorlage für das Init Script dient das unter `/opt/headphones/init-scripts` liegende File `init.fedora.centos.systemd`

```bash
sudo cp /opt/headphones/init-scripts/init.fedora.centos.systemd /etc/systemd/system/
sudo chmod 644 /etc/systemd/system/init.fedora.centos.systemd
sudo mv /etc/systemd/system/init.fedora.centos.systemd /etc/systemd/system/headphones.service
sudo nano /etc/systemd/system/headphones.service
-------------- SNIP -----------------
[Unit]
Description=Headphones - Automatic music downloader for SABnzbd

[Service]
ExecStart=/opt/headphones/Headphones.py --daemon --config /home/pi/.headphones/config.ini --datadir /home/pi/.headphones/data --nolaunch --quiet
GuessMainPID=no
Type=forking
User=pi
Group=pi

[Install]
WantedBy=multi-user.target

-------------- SNIP -----------------
sudo systemctl enable headphones.service
sudo systemctl start headphones.service
sudo systemctl status headphones.service
```

## automatische Sicherung über GIT
Vorgehensweise analog Anleitung Medusa
```bash
----  .gitignore -----
*
!config.ini
----  .gitignore -----
```

