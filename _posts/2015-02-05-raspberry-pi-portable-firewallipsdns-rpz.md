---
layout: post
title: RaspberryPi Portable Firewall/IPS/DNS-RPZ
date: '2015-02-05T23:33:00.001-08:00'
author: Kevin L
thumbnail: /images/post_raspifirewall/thumb_2015-02-06-014348_484x151_scrot.png
---

Connecting to unsecured networks can be a risk. You really don't know what could be running on the network. We can use the raspberry pi to act as a firewall for our laptops.

For class, my group was assigned a project to create a portable firewall using a raspberry pi. The pi would connect via wifi to any hotspot (xfinitywifi, starbucks, etc...) and have the host computer attached be completely secure.

![Network Diagram](/images/post_raspifirewall/2015-02-06-014348_484x151_scrot.png)


##Setup

####Prerequisites
 - 1x raspberry pi
 - 1x 4gb sd card (raspbian installed)
 - 1x usb wifi adapter

The default login for raspbian is pi:raspberry

###Misc Configs
```
pi@raspberrypi ~ $ sudo su -
root@raspberrypi:~# apt-get update && apt-get upgrade
root@raspberrypi:~# apt-get install isc-dhcp-server bind9 snort wpasupplicant wireless-tools openssh-server vim tmux
```


Set a static IP address for eth0
We can define the pi's ethernet address as 10.11.12.13/24 since that subnet is rarely used, and it's easier to remember!

Edit ```[/etc/networks/interfaces]```

```
auto lo
iface lo inet loopback
auto eth0
iface eth0 inet static
    address 10.11.12.13
    netmask 255.255.255.0
allow-hotplug wlan0
iface wlan0 inet dhcp
```

SSH only on laptop  
Edit ```[/etc/ssh/sshd_config]```

```
ListenAddress 10.11.12.13
```

We are going to need to give out ip addresses for the clients first!  
Edit ```[/etc/dhcp/dhcpd.conf]```

```
default-lease-time 600;
max-lease-time 7200;
subnet 10.11.12.0 netmask 255.255.255.0 {
      range 10.11.12.100 10.11.12.200;
     option domain-name-servers 10.11.12.13;
     option routers 10.11.12.13;
}
```

##Router / Firewall - IPtables
The pi is going to act as the router for the laptop.  
Now let's route some traffic from wlan0 to eth0!

```
iptables -A FORWARD -i wlan0 -o eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT
iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
```
That will route traffic from the wifi to the laptop, and block all attempts to connect to the actual pi.

We will script this up later :)

##DNS-RPZ - Bind9
DNS-RPZ allows us to maintain a blocklist for bad domains.

We use malware-domains.com for this, since that site provides a decently up-to-date file in bind9 format. You can easily supply your own list here.

Edit ```[cron-update-dns.sh]```

```sh
#!/bin/bash
wget http://malware-domains.com/files/malwaredomains.zones.zip
unzip malwaredomains.zones.zip
sed -e 's/\/etc\/namedb\/blockeddomain.hosts/\/etc\/bind\/rpz\/blocked.zone/g' malwaredomains.zones > blocked-local.conf
service bind9 restart
rm malwaredomains.zones.zip
rm malwaredomains.zones
echo "Blocklist updated: $(date)" >> /var/log/blocklist
```

Edit ```[/etc/bind/named.conf.options]```

```
recursion yes;
forwarders {
     8.8.8.8;
     8.8.4.4;
};
```

Edit ```[/etc/bind/named.conf.local]```

```
include "/etc/bind/blocked-local.conf";
```
Edit ```[/etc/bind/rpz/blocked.zone]```

```
    $TTL 600        ; 10 minutes
    @                       SOA     server.mydomain.com. root.mydomain.com. (
                                   42         ; serial
                                   900        ; refresh (15 minutes)
                                   60         ; retry (1 minute)
                                   604800     ; expire (1 week)
                                   43200      ; minimum (12 hours)
                                   )
                           NS      server.mydomain.com.
                           MX      1 server.mydomain.com.
    $TTL 302400     ; 3 days 12 hours
    @               IN      A       10.11.12.13
    *               IN      A       10.11.12.13

```

Run that bash script once to kick it off, then add it to crontab if you want.

##IPS - Snort
Good ol' snort. You can pull in your own rules or use the default rules that come with the install. The default settings work decently, but can be tweaked.

Edit ```[/etc/snort/snort.debian.conf]```

```
DEBIAN_SNORT_INTERFACE="wlan0" # Change interface to wlan0
```


Now we gotta connect to wifi. I wish webmin would supply a module for this, or an easier way for that matter.

Basically, I just use

```
$ iwlist wlan0 scan | less
$ iwconfig wlan0 essid "xfinitywifi" channel 11 mode Managed
```

Tie it all together
Ssh login to the pi, connect to wifi using iwconfig commands. Then run this script!
{% gist 04ce09ce803c40028037 %}

  
  
![Working Lab](/images/post_raspifirewall/20150206_012435.jpg)
