# How to set up Wireguard
Table of content

A. [Wireguard server on Ubuntu Desktop 20.04](#a-wireguard-server-on-ubuntu-desktop-2004)
B. [Wireguard Android App](#b-wireguard-android-app)
C. [Wireguard Mac App](c-wireguard-mac-app)
D. [Setup the server to allow the client peer](#d-setup-the-server-to-allow-the-client-peer)
[Other useful commands](#other-useful-commands)
[Sources](#sources)

## A. Wireguard server on Ubuntu Desktop 20.04
### Step 1: Install Wireguard on ubuntu
Update 
```
sudo apt update
```
Install
```
sudo apt install wireguard -y
```
### Step 2: Generate Wireguard Server Keys
Generate and print the private key

Note: Copy this key, you'll be using it later for a config file
```
wg genkey | sudo tee /etc/wireguard/private.key
```
Generate the public key based on the private key and print the public key
```
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```
Secure the private key file permission (optional)
```
sudo chmod go= /etc/wireguard/private.key
```
### Step 3: Configue the Wireguard config file
Create and edit the wg0.config
```
sudo nano /etc/wireguard/wg0.conf
```
The file would look like this
```
[Interface]
PrivateKey = servers_generated_private_key
Address = 192.168.66.1/32
ListenPort = 51820
SaveConfig = true
PostUp = ufw route allow in on wg0 out on eth0
PostUp = iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
PostUp = ip6tables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
PreDown = ufw route delete allow in on wg0 out on eth0
PreDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
PreDown = ip6tables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```
Explaination
<pre>
  PrivateKey -> This is the key we recently created and copied. We can paste that in the config file.
  Address -> This is the virtual network the client and server will be on when the VPN is active. 
    Any IP address can be used here, however the peers need to be on the same range.
  ListenPort -> This can be any open port on the server.
  PostUp/PreDown -> To edit the iptables 
    The Post UP command starts the VPN virtual tunnel on the WireGuard server, and the PreDown rules disable it.
    Also, Stopping transfer rules and hiding the VPN interface is done by the PreDown rule.
</pre>
### Step 4: Port Forwarding configuration

Edit the sysctl.conf file
```
sudo nano /etc/sysctl.conf
```

Add or uncomment these two lines, one is for IPv4 and one is for IPv6
<pre>
  net.ipv4.ip_forward=1
  net.ipv6.conf.all.forwarding=1
</pre>

To save and check if it's correct, it will print the configuration for you as above
```
sudo sysctl -p
```

### Step 5: WireGuard server firewall configuration (Optional)

To find out the public network interface
```
ip route list default
```
In the output of this command, you will find the public interface string after the word “dev“. For example:
<pre>default via x.x.x.x dev enp0s31f6 proto static metric 100</pre>

Open the UDP default port
```
sudo ufw allow 51820/udp
```

Open OpenSSH port
```
sudo ufw allow OpenSSH
```

Save changes to the firewall
```
sudo ufw disable
```
```
sudo ufw enable
```

Note: When I did this for my server, all my docker services couldn't communicate with each other. I found out it was because I haven't enabled ufw yet. So make sure all the ports for your docker containers are also open so they can communicate with each other

Check and verify UFW rules
```
sudo ufw status
```

### Step 6: Start the Wireguard service
Setup autostart Wireguard on boot
```
sudo systemctl enable wg-quick@wg0.service
```
Run the service
```
sudo systemctl start wg-quick@wg0.service
```
Check the service status
```
sudo systemctl status wg-quick@wg0.service
```
## B. Wireguard Android App
### Step 1: Install the Wireguard app
Download from https://play.google.com/store/apps/details?id=com.wireguard.android

### Step 2: Add the tunnel
Click on the + button
Select "Create from scratch"
Click on the Refresh icon on the right of Private key field to generate a Private key and Public key
Fill in the following for Interface:
```
Name: VPN
Private key: app_generated_private_key
Public key: app_generated_public_key
Addresses: 192.168.66.8/32
DNS servers:
```
Explanation
<pre>
  Name: You can fill with any name you want
  Addresses: The address of the peer, you can fill it following the Address configuration
  of the server
  DNS server: If you have your own DNS server or something like Cloudflare or Google DNS
</pre>

Click on "Add Peer" and fill in the following for Peer
```
Public key: servers_generated_public_key
Allowed IPs: 0.0.0.0/32, ::/0
Endpoint: PublicIP:ChosenPort
```
Explanation 
<pre>
  Public key: The public key generate from your server in Ubuntu. 
  You can use this to retrieve it: sudo cat /etc/wireguard/public.key
  Allowed IPs: The allowed IP address, 0.0.0.0/0 means all IP address 
  (Note: Seems like 0.0.0.0/0 didn't allow access to Internet, only local, 0.0.0.0/32, ::/0 gave me access to Internet)
  Endpoint: Your public IP address that can be obtained with https://ipinfo.io plus
    the port you selected from the server
</pre>

Go to step D

## C. Wireguard Mac App
### Step 1: Install the Wireguard app
Download from https://apps.apple.com/us/app/wireguard/id1451685025

### Step 2: Add the tunnel
Click on the + button
Select "Add Empty Tunnel"
Fill in the following for Interface and save:
```
Name: VPN
Public key: app_generated_public_key
On-Demand: select ethernet or Wi-Fi

[Interface]
Private key = app_generated_private_key
Address = 192.168.66.10/32
DNS = 1.1.1.1, 1.0.0.1

[Peer]
PublicKey =
AllowedIPs = 0.0.0.0/32, ::/0
Endpoint = PublicIP:ChosenPort
```
Explanation
<pre>
  Name: You can fill with any name you want
  Public key: Public key generated by app
  On-Demand: On which network do you want the VPN to apply to

  [Interface] - settings by the client
  Private key: Private key generated by app
  Address: The address of the peer, you can fill it following the Address configuration
  of the server
  DNS: Any DNS if there is

  [Peer] - settings to the server
  Public key: The public key generate from your server in Ubuntu. 
  You can use this to retrieve it: sudo cat /etc/wireguard/public.key
  Allowed IPs: The allowed IP address, 0.0.0.0/0 means all IP address 
  (Note: Seems like 0.0.0.0/0 didn't allow access to Internet, only local, 0.0.0.0/32, ::/0 gave me access to Internet)
  Endpoint: Your public IP address that can be obtained with https://ipinfo.io plus
    the port you selected from the server
</pre>

Allow access for your Mac

Go to step D

## D. Setup the server to allow the client peer

Go back to your Ubuntu Desktop

Edit the wg0.config
```
sudo nano /etc/wireguard/wg0.conf
```

Add in the following:
```
[Peer]
PublicKey = app_generated_public_key
AllowedIPs = 192.168.66.8/32
```
Explanation
<pre>
  PublicKey: The public key generated by the Android app
  AllowedIPs: The IP address set in the Android app
  Note - it needs to be seperated per peer, so if you have two peers, it'll look like
    [Peer]
    ...

    [Peer]
    ...
</pre>

Then you may turn on the Wireguard server with
```
sudo wg-quick up wg0 
```
### Tip 1: Make sure your Private DNS settings in your Android app is disabled

### Tip 2: Make sure you close the server before editing the configuration
To turn off the Wireguard server, use
```
sudo wg-quick down wg0 
```
## Other useful commands
### Check connections and server settings
To check the connections to Wireguard and server settings, use the following
```
sudo wg show
```
It'll look like this
<pre>
  interface: wg0
    public key: server_generated_public_key
    private key: (hidden)
    listening port: 51820
  
  peer: app_generated_public_key
    endpoint: public_ip:36473
    allowed ips: 192.168.66.8/32
    latest handshake: 3 minutes, 45 seconds ago
    transfer: 5.83 KiB received, 124 B sent
</pre>

## Sources
1. https://operavps.com/docs/install-wireguard-on-ubuntu/
2. https://www.reddit.com/r/WireGuard/comments/fue9hg/how_to_setup_wireguard_on_ubuntu_server_ubuntu/
