##WNDR4300 settings


### first compiling the openwrt 
* install dependencies
```
sudo apt-get update
sudo apt-get -y --no-install-recommends install git-core build-essential libssl-dev libncurses5-dev unzip gawk  subversion mercurial
```
* download the code 
```
git clone https://github.com/openwrt/openwrt.git
```
* install and update luci feed
```
cd openwrt 
./scripts/feeds update packages luci
./scripts/feeds install -a -p luci
```
* isntall shadowsocks
```
git clone https://github.com/shadowsocks/openwrt-shadowsocks.git package/shadowsocks-libev
# choose Network -> shadowsocks-libev
git clone https://github.com/aa65535/openwrt-chinadns.git package/chinadns
# choose Network -> ChinaDNS
git clone https://github.com/aa65535/openwrt-dist-luci.git package/openwrt-dist-luci
# choose LuCI -> 3. Applications
	choose luci chinadns and luci shadowsocks-libev
# maybe need to compile po2lmo
	pushd package/openwrt-dist-luci/tools/po2lmo
	make && sudo make install
	popd
```
* make menuconfig		
	select WNDR4300  settings
	
### optional for WNDR2000v4
```
# disable ipv6
	Global build settings -> remove ipv6
# switch  shadowsocks-libev to polarssl 
	libraries -> SS(libpolarssl)
# delete ppp
	Network-> remove ppp  
		then -> Kernel modules -> Network Support ->remove kmod-ppp
```
## build script for local saved archive
```shell
tar xf openwrt_with_git.tar.xz
cd openwrt
# update folder to the latest
git pull
./scripts/feeds update packages luci
./scripts/feeds install -a -p luci
pushd package/shadowsocks-libev
git pull
popd
pushd package/chinadns
git pull
popd
pushd package/openwrt-dist-luci
git pull
popd
```
	
* flash by luci or by command line 
``` shell
sysupgrade -v /tmp/filename-of-downloaded-sysupgrade.bin
```   
   
### ChinaDNS setting 
```
Enable  yes
Enable Bidirectional Filter   yes
local port 5353
CHNRoute File /etc/chinadns_chnroute
upstream Servers 223.5.5.5,8.8.4.4
```
shadowsocks settings (this is easy, maybe add later)
  
### network-->DHCP and DNS 
	General settings-->DNS forwardings: 127.0.0.1#5353
	Resolv and Hosts Files-->ignore resolve file yes
						  -->ignore /etc/hosts

						  
### network-->interfaces
```
	LAN:
		Protocol: static
		IP v4: 192.168.5.1
		mask: 255.255.255.0
		gateway 192.168.10.1
		broadcast 192.168.5.255
		custom DNS 192.168.10.1
		
	WAN:
		Protocol: static
		IP v4: 192.168.10.2
		mask: 255.255.255.0
		gateway: 192.168.10.1
		broadcast: 192.168.10.255
		custom DNS: 192.168.10.1
```		
		
### update  chinaroute file
```sh
$wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/chinadns_chnroute
```
