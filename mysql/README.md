# Raspi Emby (MediaServer)



installation von [hier](https://emby.media/linux-server.html)  - Version armhf!
```bash
dpkg -i emby-server-deb_4.4.3.0_armhf.deb
sudo systemctl stop emby-server.service
sudo chown -R /var/lib/emby
sudo nano /usr/lib/systemd/system/emby-server.service
----------- SNIP --------------
User=pi
----------- SNIP --------------
sudo systemctl start emby-server.service
```

