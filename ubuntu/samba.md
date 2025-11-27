# Samba aka SMB

sudo ufw allow 445 and 139

install and start service for wsdd for Windows Discovery
```
sudo apt-get install wsdd
```

Enable feature for Samba Direct in Windows Feature
Allow app for File Sharing for Samba in private network

Edit /etc/samba/smb.conf
force user = owner username

Add-on config for speed
```
  getwd cache = true
  read raw = Yes
  write raw = Yes
  min receivefile size = 16384
  use sendfile = true
  aio read size = 16384
  aio write size = 16384
```

Restart 
```
sudo systemctl restart smbd.service
```
