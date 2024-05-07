Notes

To find out if amd64, arm64, amdhf
```
dpkg --print-architecture
```

Downloading mergerfs
Release pages https://github.com/trapexit/mergerfs/releases
Mine is amd64
```
sudo wget https://github.com/trapexit/mergerfs/releases/download/2.40.2/mergerfs_2.40.2.ubuntu-jammy_amd64.deb
sudo dpkg -i mergerfs_2.40.2.ubuntu-jammy_amd64.deb
sudo rm mergerfs*.deb
```

Edit fstab
```
sudo nano /etc/fstab
```
Policy
Can be read here https://github.com/trapexit/mergerfs?tab=readme-ov-file#policies
```
/mnt/disk* /mnt/storage fuse.mergerfs cache.files=partial,dropcacheonclose=true,ignorepponrename=true,allow_other,use_ino,category.create=mfs,minfreespace=10G,fsname=mergerfs,defaults 0 0
```

Mount it
```
sudo mount /mnt/storage
```
