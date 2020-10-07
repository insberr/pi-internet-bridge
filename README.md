# pi-internet-bridge
How to use your raspberry pi as a WIFI to ethernet bridge
I am making this because I ran into issues while following may other tutorials.  
Issues like `dnsmasq` failing to start on boot.

## What you'll need
- Raspberry pi 4B (What I used for this, but any rpi with a wifi and ethernet connection will do)
- Ethernet cable
- A WiFI connection

### Step 1
Update your raspberry pi
```
sudo apt-get update
sudo apt-get upgrade
```

### Step 2
Install `dnsmasq`
```
sudo apt-get install dnsmasq
```

### Step 3
Configure the ethernet connection  
You need to set a static IP. Open the `dhcpcd.conf` file
```
sudo nano /etc/dhcpcd.conf
```
In this file add
```
interface eth0
static ip_address=192.168.220.1/24
static routers=192.168.220.0
```
Save the changes (CTRL + O, then CTRL + X)

### Step 4

NEXT STEPS COMING SOON
