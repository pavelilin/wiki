
# Tor-Pi

## Network Flow:


#### Router RJ45 <--> Ethernet Port on Raspberry <--> TOR <--> Raspberry WIFI AC <--> WIFI CLIENT 


## Preperation

- Download Rasbian Strech Lite from: [raspberrypi.org](https://www.raspberrypi.org/downloads/raspbian/)
- Burn Image to SD-Card.
- Boot Raspberry Pi 3.


```bash
sudo apt-get update && sudo apt-get upgrade && sudo apt-get install git 
sudo raspi-config
```
 - Change User Password within the Config Interface
 - Enable SSH In Interface Options

```bash
sudo reboot
```

## Confirutation
- SSH TO Raspberry Pi 3

```bash
SSH root@raspberry_host
git clone https://github.com/unixabg/RPI-Wireless-Hotspot.git
cd RPI-Wireless-Hotspot
sudo ./install
```

- "Y" to agree to terms
- "Y" to use preconfigured DNS
- "Y" to use Unblock-Us DNS servers
- "N" for WiFi defaults
- Type in a new WiFi password (it will be checked)
- Type in a new SSID
- Type in your desired WiFi channel (1, 6, 11)
- Type "N" when asked - "Are you using a rtl871x chipset?" -- 
- Type "N" for chromecast support (unless you plan to use a chromecast w/RasTor)

```bash
sudo reboot
sudo apt-get update && sudo apt-get upgrade
```bash


#Connect to Your new SSID with a Phone and check if you have full Internet Connection.

sudo apt-get install tor
sudo nano /etc/tor/torrc

# -- Add the following just below the first set of comments:

Log notice file /var/log/tor/notices.log
VirtualAddrNetwork 10.192.0.0/10
AutomapHostsSuffixes .onion,.exit
AutomapHostsOnResolve 1
TransPort 9040
TransListenAddress 192.168.42.1
DNSPort 53
DNSListenAddress 192.168.42.1

# -- iptable routes:

sudo iptables -F && sudo iptables -t nat -F
sudo iptables -t nat -A PREROUTING -i wlan0 -p udp --dport 53 -j REDIRECT --to-ports 53
sudo iptables -t nat -A PREROUTING -i wlan0 -p tcp --syn -j REDIRECT --to-ports 9040

# -- Check your routes:

sudo iptables -t nat -L

# -- if all routs looks like about:

sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"

# -- Create log file:

sudo touch /var/log/tor/notices.log 
sudo chown debian-tor /var/log/tor/notices.log && sudo chmod 644 /var/log/tor/notices.log

# -- Start TOR:

sudo service tor start

# -- Check to see if the service is running:

sudo service tor status

# -- Run TOR Service at Boot:

sudo update-rc.d tor enable
sudo reboot

# -- Test for TOR service is running after reboot, connect to the WIFI. 
# -- DONE -- #


# -- Optional: Install Monit Service to reload Tor Serivce if Down

sudo apt-get install monit
sudo nano /etc/monit/monitrc

# -- add those lines to the end of the config:

check process gdm with pidfile /var/run/tor/tor.pid
   start program = "/etc/init.d/tor start"
   stop program = "/etc/init.d/tor stop"
   
# -- realod and add Monit to startup
sudo monit reaload
sudo update-rc.d monit enable
 ```