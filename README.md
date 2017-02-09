# docs

## Raspberry PI + OS images

### Clone image over to SD card in OSX

- `diskutil list` - to check the name of your SD card, eg: `/dev/disk3`
- `df -h` - to see if SD card partitions are mounted (will error with `dd: /dev/rdisk3: Resource busy` if still mounted)
- `diskutil unmountDisk disk3` - to unmount SD card partitions if necessary
- `sudo su` - switch to root
- `dd bs=1m if=/PathToImgFile | pv -s 8g | dd bs=1m of=/dev/rdisk3` - copy backed up bootable OS image to the SD card, here `pv -s 8g` helps to see progress when image size is known in advance, otherwise `pv` can be used. `/dev/rdisk3` is attached SD card, `rdisk3` is used instead of `disk3` as it is going to be faster. This can be shortened if `pv` is not used - `dd bs=1m if=/PathToImgFile of=/dev/rdisk3`

### Backup SD card in OSX

- `diskutil list` - to check the name of your SD card, eg: `/dev/disk3`
- `sudo dd if=/dev/disk3 bs=1m of=/pathToFolder/fileName.img.gz` - without compression and progress indicator, see below for alternative
- `sudo dd if=/dev/disk3 bs=1m | pv -s 16g | gzip > /pathToFolder/fileName.img.gz` - uses compression and reduces image size as otherwise images are going to be the same size as SD card size, compression will collapse free space in cards. Here `-s 16g` is the size of SD card.

### ssh into raspberry

Raspberry PI should be on the same network as your computer you are `ssh`ing from. Provided that you know IP address of RPi you can connect by `ssh pi@192.168.2.2` where `pi` is username of the account set up on RPi image and `192.168.2.2` is the IP address of RPi on your network.

#### Username/Password

Following convention my Linux images will have same username/password as Raspbian has, where:
- username - `pi`
- password - `raspberry`

#### Ethernet cable

When RPi is turned on for the first time it does not connect to WiFi as it does not know SSID and Authentication details so usually you would connect using Ethernet cable.

*make sure on OSX*

- System Preferences > Network > Ethernet > Location [Automatic]
- System Preferences > Network > Ethernet > Configure IPv4 [Using DHCP]
- System Preferences > Sharing > Internet Sharing > Share your connection from [Wi-Fi]
- System Preferences > Sharing > Internet Sharing > To computers using [Ethernet]
- System Preferences > Sharing > Internet Sharing : Should be checked

*OSX check IP of ethernet connection*

By running `ifconfig` you will see current connections, the list will contain WiFi, Ethernet, FireWire, etc. We are looking for a Bridge between OSX and Linux, it is going to look like:

```
bridge100: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	options=3<RXCSUM,TXCSUM>
	ether 12:40:f3:f9:f3:64 
	inet 192.168.2.1 netmask 0xffffff00 broadcast 192.168.2.255
	inet6 fe80::1040:f3ff:fef9:f364%bridge100 prefixlen 64 scopeid 0xa 
	Configuration:
		id 0:0:0:0:0:0 priority 0 hellotime 0 fwddelay 0
		maxage 0 holdcnt 0 proto stp maxaddr 100 timeout 1200
		root id 0:0:0:0:0:0 priority 0 ifcost 0 port 0
		ipfilter disabled flags 0x2
	member: en0 flags=3<LEARNING,DISCOVER>
	        ifmaxaddr 0 port 5 priority 0 path cost 0
	nd6 options=1<PERFORMNUD>
	media: autoselect
	status: active
```

Most importantly bridge shows IP address that is assigned to OSX side, that is `192.168.2.1` in current example, so RPi will be somewhere on `192.168.2.XXX`. Once we know subnet it is possible to just scan it with [nmap](https://nmap.org/book/inst-macosx.html).

*Scan network for connected devices*

- `nmap -sn 192.168.2.0/24` - scan everything under `192.168.2.` example output shows RPi IP is `192.168.2.2` because OSX is on `192.168.2.1`:
```
Starting Nmap 7.40 ( https://nmap.org ) at 2017-02-09 13:54 GMT
Nmap scan report for 192.168.2.1
Host is up (0.00096s latency).
Nmap scan report for 192.168.2.2
Host is up (0.0011s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 6.81 seconds
```

#### Wi-Fi

Wi-Fi is almost the same as Ethernet, you have to identify current IP address (eg: `192.168.1.5`) and then scan subnet (eg: `192.168.1.`) for connected devices using nmap:

```
$ nmap -sn 192.168.1.0/24

Starting Nmap 7.40 ( https://nmap.org ) at 2017-02-09 14:01 GMT
Nmap scan report for 192.168.1.1
Host is up (0.049s latency).
Nmap scan report for 192.168.1.2
Host is up (0.078s latency).
Nmap scan report for 192.168.1.3
Host is up (0.00055s latency).
Nmap scan report for 192.168.1.254
Host is up (0.0046s latency).
Nmap done: 256 IP addresses (4 hosts up) scanned in 9.40 seconds
```

Although it will be trickier in WiFi network as usually it has more devices connected.
