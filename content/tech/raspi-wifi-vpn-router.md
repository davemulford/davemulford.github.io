---
title: "Creating a wireless VPN router with a Raspberry Pi"
slug: "rpi-vpn-router"
date: "2017-09-04"
tags:
  - "raspberry-pi"
  - "vpn"
---

Learn how to create a VPN router with a Raspberry Pi!

<!--more-->

Here's the scenario: you've made sure to book a hotel that has complimentary wi-fi, checked in, and BAM! You realize the wi-fi is horribly slow and entering the access code they gave you is nearly impossible on their desktop website that you're accessing from your phone. How are you supposed to cure your boredom while laying in that hotel bed?

Most hotels also have an ethernet cable in each room. This is great if you're working from the hotel room on your laptop, but seriously, who does that?! You have options!

You can buy the [Anonabox Wi-Fi Tor & VPN Router](http://a.co/6hAd7zc), or you can make your own. This post focuses on the second option.

Let's start by downloading Raspbian. Specifically, I downloaded Raspbian Stretch LITE from https://www.raspberrypi.org/downloads/raspbian/. I chose the LITE version because a desktop isn't needed here.

If you're on Linux and have your MicroSD card plugged in, you can use the `dd` command to write the Raspbian image to your card. Here is the sample command I used, but note that you may have to change `/dev/sdb` to something else.

~~~
sudo dd if=2017-08-16-raspbian-stretch-lite.img of=/dev/sdb bs=8M status=progress
~~~

Now insert the MicroSD card into the pi and connect HDMI, a keyboard, and the AC adapter. Once you're prompted to login, the username is *pi* and the password is *raspberry*. **Because of the security risk on hotel networks, I highly recommend you change the default password for this account.**

There are five main pieces of software we'll be integrating for this project:

- **openvpn** - connects to a VPN for privacy reasons (optional, but highly recommended).
- **hostapd** - provides wireless router functionality.
- **dnsmasq** - gives a client connecting to your wireless router an IP address.
- **iptables** - routes traffic from one network interface to another, e.g. forwarding client traffic from the wireless chip to the VPN tunnel.
- **dhcpcd** - controls local network cards.

Let's install the packages that are needed! On the pi, run the commands below.

~~~
sudo apt-get install openvpn
sudo apt-get install iptables-persistent
sudo apt-get install hostapd
sudo apt-get install dnsmasq
~~~

We'll now configure each piece of software. To start, you need to determine the name of your network interfaces. Use the `ifconfig` command to do this; below is some sample output. I've removed any interfaces that aren't relevant, like loopback. My interface names are `enxb827ebcbaaf2` and `wlan0`.

~~~
$ ifconfig
enxb827ebcbaaf2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.250  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::e4b2:5f30:5972:b90b  prefixlen 64  scopeid 0x20<link>
        ether b8:27:eb:cb:aa:f2  txqueuelen 1000  (Ethernet)
        RX packets 2054  bytes 179432 (175.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 933  bytes 137337 (134.1 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlan0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.220.1  netmask 255.255.255.0  broadcast 192.168.220.255
        inet6 fe80::ba27:ebff:fe9e:ffa7  prefixlen 64  scopeid 0x20<link>
        ether b8:27:eb:9e:ff:a7  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 46  bytes 6234 (6.0 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
~~~

Let's use this information to configure dhcpcd correctly. I've chosen to configure both interfaces with a static IP address for easier access via SSH. This is optional and only used for troubleshooting. The only requirement here is configuring wlan0 with a static IP address and `192.168.220.1` is a random address I chose to avoid any address conflicts.

Add this to the bottom of `/etc/dhcpcd.conf`.
~~~
interface enxb827ebcbaaf2
static ip_address=192.168.1.250/24
static routers=192.168.1.1
static domain_name_servers=8.8.8.8 8.8.4.4

interface wlan0
static ip_address=192.168.220.1/24
~~~

Next up is dnsmasq and we'll start by making a backup of the default configuration file.

~~~
sudo cp /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
~~~

Now replace the entire contents of `/etc/dnsmasq.conf` with what's below. If you changed wlan0's IP address in the steps above, you'll need to change the `listen-address` line.

~~~
interface=wlan0       # Use interface wlan0
listen-address=192.168.220.1   # Specify the address to listen on
bind-interfaces      # Bind to the interface
server=8.8.8.8       # Use Google DNS
domain-needed        # Don't forward short names
bogus-priv           # Drop the non-routed address spaces.
dhcp-range=192.168.220.50,192.168.220.150,12h # IP range and lease time
~~~

Configuring hostapd is a bit more complex and involves editing a startup script. This is unfortunate, but at least the edits are minor.

First, we'll create `/etc/hostapd/hostapd.conf` with the information about the wireless router. This will include things such as the SSID that's listed when browsing and the "wi-fi password" used to connect. Add the following text to `/etc/hostapd/hostapd.conf` relacing the `ssid` and `wpa_passphrase` with your values.

Note that if you're using the Raspberry Pi 3, you won't have to change the `driver` setting.

~~~
interface=wlan0
driver=nl80211

hw_mode=g
channel=6
ieee80211n=1
wmm_enabled=1
ht_capab=[HT40][SHORT-GI-20][DSSS_CCK-40]
macaddr_acl=0
ignore_broadcast_ssid=0

# Use WPA2
auth_algs=1
wpa=2
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP

# This is the name of the network
ssid=mypiwifi
# The network passphrase
wpa_passphrase=mypiwifi
~~~

The final service to setup is openvpn. I use a [privateinternetaccess VPN](https://www.privateinternetaccess.com), so these steps may change slightly if you're using a different provider. PIA provides the following files:

- Multiple `.ovpn` for connecting to various endpoints they have (like Seattle vs California).
- A client certificate and key named `ca.crt` and `crl.pem`, respectively.
- A credentials file with a `.creds` extension.

You shouldn't touch the certifiate and key files at all. Those are created by PIA to verify your identity. The only files you should have to edit are those ending in `.ovpn`. Here is an example of one:

~~~
client
dev tun
proto udp
remote us-california.privateinternetaccess.com 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
tls-client
remote-cert-tls server
auth-user-pass pia.creds # <---- you may need to add this line if it doesn't already exist
comp-lzo
verb 1
reneg-sec 0
crl-verify crl.pem
script-security 2 # <---- you may need to add this line if it doesn't
up /etc/openvpn/update-resolv-conf # <---- you may need to add this line if it doesn't
down /etc/openvpn/update-resolv-conf # <---- you may need to add this line if it doesn't
~~~

We'll use systemd's openvpn templates, which will allow you to quickly switch between VPN endpoints using the `systemctl` command. First, let's copy the required certificate, key, and credentials files to `/etc/openvpn/client`.

~~~
sudo cp pia.creds ca.crt crl.pem /etc/openvpn/client
sudo cp ca.crt /etc/openvpn/client
sudo cp crl.pem /etc/openvpn/client
~~~

Now you'll want to copy all of the `.ovpn` files you want to that directory. My recommendation is to use lowercase filenames containing no spaces. You'll also need to change the extension from `.ovpn` to `.conf`. For instance, PIA's California VPN file is named `US California.ovpn`, so I named it `california.conf`.

~~~
sudo cp "US California.ovpn /etc/openvpn/client/california.conf"
~~~

To start or stop a VPN connection, you can now use the `systemctl` command. For example, if I wanted to stop Seattle and start California, I'd do this:

~~~
sudo systemctl stop openvpn@seattle.service
sudo systemctl start openvpn@california.service
~~~

You now need to start the services we just configured and set them to start when the pi boots. The commands below do this.

~~~
sudo systemctl restart dhcpcd.service
sudo systemctl restart dnsmasq.service
sudo systemctl restart hostapd.service

sudo systemctl enable dhcpcd.service
sudo systemctl enable dnsmasq.service
sudo systemctl enable hostapd.service

# Start the desired VPN at system boot
sudo systemctl enable openvpn@california.service
~~~

Once all the services have started, final thing to do is configure `iptables` to forward the traffic between the various interfaces we've setup.

The commands below will route packets between the wireless/wired interface and the VPN interface (named `tun0`) .

~~~
sudo iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
sudo iptables -A FORWARD -i tun0 -o enxb827ebcbaaf2 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i enxb827ebcbaaf2 -o tun0 -j ACCEPT

sudo iptables -A FORWARD -i tun0 -o wlan0 -m state --state RELATED,ESTABLISHED
sudo iptables -A FORWARD -i wlan0 -o tun0 -j ACCEPT
~~~

Now make these routes persistent:

~~~
sudo netfilter-persistent save
~~~

That's it! You should now see a new wireless router with the name you chose. And any traffic is encrypted over the VPN. Have fun with **MUCH** better internet at your hotel!
