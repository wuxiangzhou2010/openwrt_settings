### raspberry pi wifi setting(/etc/network/interfaces)
```
allow-hotplug wlan0
iface wlan0 inet static
address 192.168.1.222
netmask 255.255.255.0
gateway 192.168.1.1
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp
```


/etc/wpa_supplicant/wpa_supplicant.conf is like 
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=GB

#network={
#       ssid="ChinaNGB-xxx"
#       psk="xxx"
#       key_mgmt=WPA-PSK
#}

network={
        ssid="ChinaNet-xxx"
        psk="xxx"
        key_mgmt=WPA-PSK
}


#network={
#       ssid="TP-LINK_xxx"
#       psk="xxx"
#       key_mgmt=WPA-PSK
#       proto=RSN
#       pairwise=CCMP
#       auth_alg=OPEN
#}
```



### raspberry pi3 eth0 setting
```
# Please note that this file is written to be used with dhcpcd
# For static IP, consult /etc/dhcpcd.conf and 'man dhcpcd.conf'

# Include files from /etc/network/interfaces.d:
source-directory /etc/network/interfaces.d

auto lo
iface lo inet loopback

auto eth0
allow-hotplug eth0
iface eth0 inet static
address 192.168.10.1
netmask 255.255.255.0
dns-nameservers (your private DNS server)    
```


### the iptables
```
#echo 1 >> /proc/sys/net/ipv4/ip_forward 
# To make it auto-set this value on boot uncomment this line in /etc/sysctl.conf

#net.ipv4.ip_forward=1
# next up: set up some rules in iptables to perform the natting and forwarding:

# Always accept loopback traffic
iptables -A INPUT -i lo -j ACCEPT

# We allow traffic from the LAN side
iptables -A INPUT -i eth0 -j ACCEPT

######################################################################
#
#                         ROUTING
#
######################################################################

# eth0 is LAN
# wan0 is WAN

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
# Masquerade.
iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
# fowarding
iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
# Allow outgoing connections from the LAN side.
iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT
```
