This is the buildsystem for the OpenWrt Linux distribution.

You need to have installed the following tools
```
$ sudo apt-get install gcc binutils bzip2 flex python perl 
$ sudo apt-get isntall make findutils grep diffutils unzip gawk subversion zlib1g-dev build-essential
```
Run to get all the latest package definitions defined in feeds.conf / feeds.conf.default respectively, and install symlinks of all of them into
package/feeds/.
```
$ ./scripts/feeds update -a
$ ./scripts/feeds install -a 
```
Use "make menuconfig" to configure your image. 'intorobot-default.config' is provided for you reference
```
$ make defconfig & make prereq
$ cp intorobot-default.config .config
$ make menuconfig
```

Simply running "make" will build your firmware. It will download all sources, build the cross-compile toolchain, 
the kernel and all choosen applications. **Note: Please make sure you can access google, cause some source files need to be downloaded (翻翻翻）**
```
$ make
```


**If you have difficulty to download files, please tell us  (chy at molmc dot com)**

You can use "scripts/flashing/flash.sh" for remotely updating your embedded
system via tftp.

The OpenWrt system is documented in docs/. You will need a LaTeX distribution
and the tex4ht package to build the documentation. Type "make -C docs/" to build it.

To build your own firmware you need to have access to a Linux, BSD or MacOSX system
(case-sensitive filesystem required). Cygwin will not be supported because of
the lack of case sensitiveness in the file system.


Sunshine!
	Your OpenWrt Project
	http://openwrt.org

*******************************************************************************************
##How to Customize Your IntoRobot-Atom OpenWrt
###  1.Set ssh as the default login mode.  

```
      $ vim package/network/services/dropbear/files/dropbear.config 
      1 config dropbear 
      2    option PasswordAuth 'on' 
      3    option RootPasswordAuth 'on' 
      4    option Port '22' 
```
       Note: openwrt has no password by default, and can only accept telnet login initially. So we need to modify the file "package/base-files/files/etc/shadow" by adding passwd e.g., "abc.123"
   
```
      $ vim package/base-files/files/etc/shadow
        1 root:$1$E.GIRCCH$WHE1ArCEXwl3FkhsOHIVN1:16441:0:99999:7:::
      
      if you don not need any default passwd, just modify the above as 
        
        1 root::0:0:99999:7:::
```

### 2. Add necessary libs and apps, i.e.,
```
     python (in Languages->Python)
     wget bzip2 grep usleep (in busybox, in 
           Base system->busybox
                      ->Customize busybox options
                             ->Core utilities->usleep, 
                             ->Networking Utilities->wget
                             ->Finding Utilities->grep
                             ->Archival Utilities->bzip2), 
     gnupg, rng-tools, luci-lib-json (for webpanel, in Utilities, in LuCI->Libraires)
     avahi-nobus-daemon (for Arduino-IDE, in Network->IP Addresses and Names)
     stm32flash (for stm32, in Utilities)
```

### 3. Change default lan ipaddress (Here we change from 192.168.1.1 to 192.168.8.1)
```
     $ vim package/base-files/files/bin/config_generate
       87 lan) ipaddr="192.168.8.1" ;
```
   
###  4. Add customized network configuration in target/linux/ramips/base-files/etc/board.d/02_network

###  5. Add default-built image with custmized name
```
   $ vim target/linux/ramips/image/Makefile
```
   
###  6. Add customized dts file (ATOM.dts) to the dir of target/linux/ramips/dts/. Here we change the default serial baud rate to 115200 in the file mt7620n.dtsi
```
   $ vim target/linux/ramips/dts/mt7620n.dtsi
   13 bootargs = "console=ttyS0,115200";
```

###  7. Add customized board name in target/linux/ramips/base-files/lib/ramips.sh, and add image check info in target/linux/ramips/base-files/lib/upgrade/platform.sh

###  8. Customize the hostname in package/base-files/files/etc/config/system
```
   $ vim package/base-files/files/etc/config/system
     2 option hostname Atom
```

###  9. Modify the file package/kernel/mac80211/files/lib/wifi/mac80211.sh to enable wifi
```
   $ vim package/kernel/mac80211/files/lib/wifi/mac80211.sh
   121  option disabled 0 

        config wifi-iface
            option device   radio$devidx
            option network  lan
            option mode     ap
            option ssid     IntoRobot-Atom$(cat /sys/class/ieee80211/${dev}/macaddress|awk -F ":" '{print $5""$6 }'| tr a-z A-Z)
            option encryption psk2
            option key intorobot 
```
