# pi-internet-bridge
This is a tutorial on how to use your Raspberry Pi as a WIFI to ethernet bridge.  
I am making this because I ran into issues while following may other tutorials. Issues like `dnsmasq` failing to start on boot.  
If you run into issues, feel free to submit an issue to this GitHub repository.  
_This was last updated on January 1, 2024._

#### Why?
I am using an old PC that does not have built in Wi-Fi, so I use a wireless to ethernet bridge that plugs into the wall.  
Since the plug has issues and constantly disconnects me from the internet, I wondered if it was possible to use my raspberry pi as the bridge.  
It works perfectly and so far has not disconnected my PC from the internet. Finally, I can play on that Minecraft server without being disconnected every 15 minutes.

There are plenty of these (almost the same) tutorials everywhere (they almost seem like copies...) and while following them I ran into issues. I decided, "Why not rewrite it, but with what actually works, so no one has to spend a sleepless night trying to fix errors when all they wanted to do was play some multiplayer Minecraft."

## What you'll need
- Raspberry Pi with Wi-Fi and Ethernet connection capabilities
- Ethernet cable
- A Wi-Fi connection

### Step 1
Connect your Raspberry Pi to the internet and SSH into it or connect it to a display.

Next, update your Raspberry Pi
```
sudo apt-get update
sudo apt-get upgrade
```

### Step 2
Install `dnsmasq`
```
sudo apt-get install dnsmasq
```
Note: This tutorial assumes the `dhcpd` and `iptables` commands work and the libraries are installed for them.

### Step 3
Configure the ethernet connection.  
You need to set a static IP. Open the `dhcpcd.conf` file
```
sudo nano /etc/dhcpcd.conf
```

In this file add
```
interface eth0
static ip_address=192.168.220.1/24
static routers=192.168.220.0
metric 300

interface wlan0
metric 200
```
Save the changes (CTRL + O, then CTRL + X)

### Step 4
Restart `dnsmasq`
```
sudo service dhcpcd restart
```

### Step 5
Before modifying dnsmasq's config make a backup copy
```
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
```

Next open the config file
```
sudo nano /etc/dnsmasq.conf
```

And type the following
```
interface=eth0    # Use interface eth0  
listen-address=192.168.220.1   # Specify the address to listen on
bind-dynamic      # Bind to the interface
server=8.8.8.8    # Use Google DNS
domain-needed     # Don't forward short names
bogus-priv        # Drop the non-routed address spaces.
dhcp-range=192.168.220.50,192.168.220.150,12h   # IP range and lease time
```
Save and exit the editor.  
This is where I ran into issues. All the tutorials I found use `bind-interfaces`, which after some research, I found that it was making `dnsmasq` start the connection before it was ready. To fix the problem, I replaced `bind-interfaces` with `bind-dynamic`.

### Step 6
You now need to configure the Raspberry Pi’s firewall so that it will forward all traffic from the `eth0` connection over to the `wlan0` connection.

To do this, enable ipv4p IP Forwarding in the `sysctl.conf` file.
```
sudo nano /etc/sysctl.conf
```

In the file find the following line
```
#net.ipv4.ip_forward=1
```

and uncomment it:
```
net.ipv4.ip_forward=1
```

Save and exit the file.

### Step 7
You probably don't want to wait until the next reboot for the configuration to load, so run the following command
```
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
```

### Step 8
Next you need to make `wlan0` and `eht0` forward traffic to each other
```
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE  
sudo iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT  
sudo iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT
```

Since the iptables are flushed on every boot, you will need to save them somewhere so they are loaded back in on every boot.
```
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

To make the iptables load in on every boot, first run
```
sudo nano /etc/rc.local
```

Then add the following above the `exit 0`
```
iptables-restore < /etc/iptables.ipv4.nat
```
Save and exit the file

### Step 9
Finally restart `dnsmasq`
```
sudo service dnsmasq start
```
Everything should now be ready. You should have a Raspberry Pi Wi-Fi bridge. To test if its working, plug your device into the Pi's ethernet port. If your device now has an internet connection, you set everything up successfully.

It is recommended that you reboot
```
sudo reboot
```

## Credits
[Article used for the how to](https://pimylifeup.com/raspberry-pi-wifi-bridge/)
