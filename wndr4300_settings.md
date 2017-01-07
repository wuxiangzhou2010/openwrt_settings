## WNDR4300 settings


### first compiling the openwrt 
* install dependencies
```
sudo apt-get update
sudo apt-get -y --no-install-recommends install git-core build-essential libssl-dev libncurses5-dev unzip gawk subversion mercurial
```
* download the code 
```
git clone https://github.com/openwrt/openwrt.git

# or for 15.05 branch
git clone https://github.com/openwrt/openwrt.git -b chaos_calmer
```
or if internet is slow 
```
git clone --depth=1 https://github.com/openwrt/openwrt.git
```
* install and update luci feed
```
cd openwrt 
./scripts/feeds update packages luci
./scripts/feeds install -a -p luci
```
* install shadowsocks
```
#  shadowsocks-libev (maintainer  wongsyrone， 软件包只包含 shadowsocks-libev 的可执行文件, 可与 luci-app-shadowsocks 搭配使用)
git clone https://github.com/shadowsocks/openwrt-shadowsocks.git package/shadowsocks-libev
choose Network -> shadowsocks-libev

# luci-app-shadowsocks (maintainer  aa65535， 本软件包是 shadowsocks-libev 的 LuCI 控制界面, 方便用户控制和使用「透明代理」「SOCKS5 代理」「端口转发」功能.)
git clone https://github.com/shadowsocks/luci-app-shadowsocks  package/luci-app-shadowsocks
choose LuCI -> 3. Applications ->luci-app-shadowsocks

# chinaDNS(可执行文件)
git clone https://github.com/aa65535/openwrt-chinadns.git package/chinadns
choose Network -> ChinaDNS

# openwrt-dist-luci(luci app for chinaDNS)
git clone https://github.com/aa65535/openwrt-dist-luci.git package/openwrt-dist-luci
choose LuCI -> 3. Applications -> luci-app-chinadns
```

* maybe need to compile po2lmo
```
	pushd package/openwrt-dist-luci/tools/po2lmo
	make && sudo make install
	popd
```
* make menuconfig	
```	
# select WNDR4300  settings
# select luci -> 1. collection -> luci 
		 2. Modules -> luci-base
# search and enable iptables-mod-tproxy
```
* option for WNDR2000v4, which has smaller RAM and ROM 
```
# disable ipv6
	Global build settings -> remove ipv6
# remove  PACKAGE_libip6tc
# switch to polarssl (mbedtls has some problem) 
	libraries -> SSL(polarssl)
# disable ppp
	Network-> remove ppp  
	Kernel modules -> Network Support ->remove kmod-ppp
...
...
use the configfile in  openwrt_settings\compile_config
```
* open .config and open ip setting to "CONFIG_PACKAGE_ip=y" or add ip in openwrt-shadowsocks/Makefile
```
DEPENDS:=$(3) +libpcre +libpthread +ip
```
* build script for local saved archive
```shell
#tar xf openwrt_with_git.tar.xz
#cd openwrt
# update folder to the latest

sudo apt-get -y --no-install-recommends install git-core \
build-essential libssl-dev libncurses5-dev unzip gawk subversion mercurial
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
pushd package/luci-app-shadowsocks
git pull
popd

```
	
* flash by luci or by command line 
``` shell
# for routers like wndr2000v4 use
sysupgrade -v /tmp/openwrt-ar71xx-generic-wnr2000v4-squashfs-sysupgrade.bin.bin 

# for wndr4300 use TFTP　
(openwrt-ar71xx-nand-wndr4300-ubi-factory)
or upgrade using the luci
(openwrt-ar71xx-nand-wndr4300-squashfs-sysupgrade.tar)
```   
   
### ChinaDNS setting 
```
Enable  yes
Enable Bidirectional Filter   yes
local port 5353
CHNRoute File /etc/chinadns_chnroute
upstream Servers 114.114.114.114,127.0.0.1#5300
```
### shadowsocks settings
* general  	
	Transparent Proxy 	server: hk
	SOCKS5 Proxy 		Disable
	Port Forward  		server: hk 	port: 5300	destination: 8.8.4.4:53
* server Manage
	add your server
* Access control
	ChinaRoute
### network-->DHCP and DNS 
	General settings-->DNS forwardings: 127.0.0.1#5353
	Resolv and Hosts Files-->ignore resolve file (yes)
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
		custom DNS: 114.114.114.114
```			
### update chinaroute file
``` shell
wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/chinadns_chnroute
```

###  note 
* if git protocol is blocked, you can use https to fetch the repo
```
# hostapd
openwrt/package/network/services/hostapd/Makefile, around line 15 
PKG_SOURCE_URL:=git://w1.fi/srv/git/hostap.git
change to http://w1.fi/hostap.git

#odhcp
package/network/services/odhcpd/Makefile
PKG_SOURCE_URL:=git://github.com/openwrt/odhcpd.git
change to https://github.com/openwrt/odhcpd.git

```
* if mbedtls has compiling error open these macro (configure: ... error: xxx required)
```
vi staging_dir/target-mips_34kc_musl-1.1.15/usr/include/mbedtls/config.h

MBEDTLS_CIPHER_MODE_CFB 
MBEDTLS_ARC4_C
MBEDTLS_BLOWFISH_C
MBEDTLS_CAMELLIA_C
```

### install the lateset ss on server
```
# from https://github.com/shadowsocks/shadowsocks/pull/575
pip install git+https://github.com/shadowsocks/shadowsocks.git@<tag>
pip install git+https://github.com/shadowsocks/shadowsocks.git@master
pip install git+https://github.com/shadowsocks/shadowsocks.git@2.9.0

```