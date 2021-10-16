
# Raspberry Pi Zero v1.3 Headless WiFi Setup For Pihole

The following instructions will ultimately enable your Pi Zero to automatically connect the WiFi  
to the router during every boot up, for complete headless management.  
***
**Note**:  
Follow this guide only if your WiFi chipset is not natively supported by the Raspian OS.

***
&nbsp;
## Appendix
&nbsp;1. Prerequisite  
&nbsp;2. Enabling USB ethernet gadget mode on Pi
&nbsp;3. Things to do once connected to Pi via USB  
        &nbsp; &nbsp; &nbsp; 3.1. Enabling Internet access in Raspberry Pi  
        &nbsp; &nbsp; &nbsp; 3.2. SSH into Raspberry Pi  
        &nbsp; &nbsp; &nbsp; 3.3. Change locale settings  
        &nbsp; &nbsp; &nbsp; 3.4. Update the OS  
&nbsp;4. Installing WiFi Driver  
&nbsp;5. Installing Pihole  
&nbsp;6. Installing Unbound  
&nbsp;  


## 1. Prerequisite

a) Raspberry Pi Zero v1.3  
b) USB data cable  
c) Raspbian OS  
d) A charger  
e) Ubuntu 20.04 lts  
f) Raspberry Pi Imager tool  
g) SD card  
h) [WiFi adapter](https://robu.in/product/rtl8188-mini-usb-wireless-network-card-150mbps-wifi-dongle/)

&nbsp;
## 2.Enabling USB ethernet gadget mode on Pi

To enable the gadget mode, edit ```config.txt``` and ```cmdline.txt``` files in 
the```boot```partition of the SD card.\
&nbsp;

Add ```dtoverlay=dwc2``` to ```config.txt``` and\
Add ```add modules-load=dwc2,g_ether``` after the ```rootwait``` in ```cmdline.txt```\
&nbsp;  
IMPORTANT: leave a space before and after the ```add modules-load=dwc2,g_ether``` line.\
&nbsp;  
Make an empty ```ssh``` file inside the same partition to enable SSH on boot.  
&nbsp;  
Now connect the USB data cable to the USB port of the Pi and plug it in the PC's   
USB port.

&nbsp;
## 3. Things to do once connected to Pi via USB
### 3.1 Enabling Internet access in Raspberry Pi

Before enabling the internet, make sure that the Pi is connected as a USB gadget.\
Check ```sudo dmesg``` to confirm it. Look for lines similar to 
```
.....(other things we don't care about right now).....
[ 1943.306812] usb 1-12: new high-speed USB device number 14 using xhci_hcd
[ 1943.494324] cdc_ether 1-12:1.0 usb0: register 'cdc_ether'  
               at usb-0000:00:14.0-12, CDC Ethernet Device, 2e:2f:d7:3d:10:88
[ 1943.526965] cdc_ether 1-12:1.0 enp0s20f0u12: renamed from usb0
```  
***
**Note**:  
Give at least 30 seconds or more before trying the ```dmesg```  
***

If the last line says something like
```
cdc_ether 1-12:1.0 usb0: unregister 'cdc_ether' ....
```

that means the USB gadget mode was enabled and for some reason it got disconnected.  
If you get this, check  whether the USB cable was connected to the USB port on the Pi, not   
on the power port.

Once confirmed that everything is working fine, goto gnome-network-manager gui and change  
the USB ethernet IPv4 setting to ```shared to other computers```.  

Disconnect and the reconnect the USB ethernet in the gui. ( Might be needed to dis/connect the
PCI ethenet, which is your PC's internet connection,  also ).  

Now you should be able to SSH into the internet ready Pi.  
&nbsp;
### 3.2 SSH into Raspberry Pi

```
ssh pi@raspberrypi.local
```
```
password: raspberry
```  
&nbsp;
### 3.3 Change locale settings

1.&nbsp; To eliminate perl: LC errors during package installations;  

```
sudo dpkg-reconfigure locales
```
&nbsp; &nbsp; select the appropriate locale. 
For example; en_US.UTF-8

&nbsp;2. &nbsp;Set timezone  
```
sudo ln -sf /usr/share/timezone/Country/Region /etc/localtime
```
&nbsp;3. Expand filesystem   
&nbsp;  
&nbsp;  &nbsp; expand the filesystem by using 
```
sudo raspi-config
```

&nbsp;  Now its time to shutdown the Pi to take effect the changes.  

&nbsp;  Reboot seems to be breaking the USB ethernet gadget for some reason. Hence you need to shutdown  
&nbsp; the system.  
***
&nbsp; **Note** :  
&nbsp; Check ```dmesg``` again . If you see some messages saying something about kernel, reboot the host.   
&nbsp; Only then re-connect the Pi.
***

&nbsp; Then follow the instructions on 3.1.  
&nbsp;  
### 3.4 Update the OS
```
sudo apt Update
```
```
sudo apt upgrade
```
&nbsp;  Shutdown and un/replug the Pi. Follow 3.1.  
&nbsp;  
## 4 Installing WiFi Driver
If you bought a WiFi adapter like [this one](https://robu.in/product/rtl8188-mini-usb-wireless-network-card-150mbps-wifi-dongle/)  which does not have a native driver support in the OS, you need to install it.  
For that you need to know the device id of the WiFi adapter. To know the device id,
 just plugin the WiFi adapter to your PC and run ```lsusb```. It will show the device 
 id like 
 ```
 Bus 001 Device 002: ID 0bda:f179 Realtek Semiconductor Corp.
 ```
 Here ```0bda:f179``` is the important part. Now you need to install a driver that
 supports this device id. For that goto
 ```
 http://downloads.fars-robotics.net/wifi-drivers/install-wifi
 ```

and search whether that page has your WiFi ID present. For example, if you search for ```0bda:f179```
you will see that it is present there. 
```
elif cat .lsusb | grep -i '0BDA:F179' ; then
		driver=8188fu
```
This  means that for the device id ```0bda:f179``` you need to install the driver ```8188fu```.  
Once you made sure that the driver is available, run the following commands;
```
sudo wget http://downloads.fars-robotics.net/wifi-drivers/install-wifi -O /usr/bin/install-wifi
sudo chmod +x /usr/bin/install-wifi
```
To check whether the driver is available for your current installed kernel, run
```
sudo install-wifi -c "your driver version"
```
eg; ```sudo install-wifi -c 8188fu```  

If it says, 
```
Checking for a 8188fu wifi driver module for your current kernel.
There is a driver module available for this kernel revision.
```
then you can install the WiFi driver by
```
sudo install-wifi "your driver"
```
eg; ```sudo install-wifi 8188fu```  
&nbsp;  
Now we need to setup the WiFi to automatically connect to the access point during the next reboot.  
&nbsp;   
###  WiFi configuration for auto connecting to the access point
Add these line to ```/etc/dhcpcd.conf```;
```
interface wlan0
        static ip_address=192.168.X.X/24
        static routers=192.168.X.X
        static domain_name_servers=192.168.X.X     # probably your router ip
        env wpa_supplicant_conf=/etc/wpa_supplicant/wpa_supplicant.conf
```
&nbsp;

Add these lines to ```/etc/wpa_supplicant/wpa_supplicant.conf```;  
```
country="your country code"

network={
    scan_ssid=1
    ssid="wifi name"
    psk="wifi password"
}
```
Now shutdown the Pi. Connect the WiFi adapter and power cable then turn it on.  

## 5. Installing Pi-hole
To install the pihole run,
```
curl -sSL https://install.pi-hole.net | bash
```
or 

```
wget -O basic-install.sh https://install.pi-hole.net
sudo bash basic-install.sh
```
&nbsp;
#### Optimise for solid state drives
If Pi-hole is running on a solid state drive (SD card, SSD etc..) it is recommended 
to uncomment the ```DBINTERVAL``` value and change it to at least ```60.0``` to minimize 
writes to the database.  
Add these lines to the ```/etc/pihole/pihole-FTL.conf```;
```
## Database Interval
## How often do we store queries in FTL's database -minutes-?
## See: https://docs.pi-hole.net/ftldns/database/
## Options: number of minutes

DBINTERVAL=60.0
```

Restart pihole;
```
sudo systemctl restart pihole-FTL.service
```   
&nbsp;
## 6. Installing Unbound
```sudo apt install unbound```   

#### Configure Unbound
Add these lines to ```/etc/unbound/unbound.conf.d/pi-hole.conf```;
```
server:
    # If no logfile is specified, syslog is used
    # logfile: "/var/log/unbound/unbound.log"
    verbosity: 0

    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes

    # May be set to yes if you have IPv6 connectivity
    do-ip6: no

    # You want to leave this to no unless you have *native* IPv6. With 6to4 and
    # Terredo tunnels your web browser should favor IPv4 for the same reasons
    prefer-ip6: no

    # Use this only when you downloaded the list of primary root servers!
    # If you use the default dns-root-data package, unbound will find it automatically
    #root-hints: "/var/lib/unbound/root.hints"

    # Trust glue only if it is within the server's authority
    harden-glue: yes

    # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
    harden-dnssec-stripped: yes

    # Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
    # see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
    use-caps-for-id: no

    # Reduce EDNS reassembly buffer size.
    # Suggested by the unbound man page to reduce fragmentation reassembly problems
    edns-buffer-size: 1472

    # Perform prefetching of close to expired message cache entries
    # This only applies to domains that have been frequently queried
    prefetch: yes

    # One thread should be sufficient, can be increased on beefy machines. In reality for most users running on small networks or on a single machine, it should be unnecessary to seek performance enhancement by increasing num-threads above 1.
    num-threads: 1

    # Ensure kernel buffer is large enough to not lose messages in traffic spikes
    so-rcvbuf: 1m

    # Ensure privacy of local IP ranges
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10
``` 
&nbsp;

Restart the Unbound  
```
sudo service unbound restart
```
&nbsp;  
#### Configure pihole
Finally, configure Pi-hole to use your recursive DNS server by specifying ```127.0.0.1#5335``` as the  Custom DNS (IPv4)  
&nbsp;
#### Configure network
Set the ```static domain_name_servers``` in   ```/etc/dhcpcd.conf``` to ```127.0.0.1```;
```
interface wlan0
        
        static domain_name_servers=127.0.0.1
        
```
Restart dhcp service;
```
sudo systemctl restart dhcpcd.service
```