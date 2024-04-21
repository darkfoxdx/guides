# How to set up Wireguard
## Ubuntu Desktop 20.04
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
