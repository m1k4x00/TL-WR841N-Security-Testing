![image](https://github.com/user-attachments/assets/1f15b6f7-b746-4057-8d21-5159e9b52b87)
## Details

## Target
Manufacturer: TP-Link

Part Number: TL-WR841N(EU)

Firmware version: 14.0

Serial Number: 22487Q7006381

## Equipment
Multimeter

Logic Analyzer: AZDelivery Logic Analyzer

USB to UART Adapter: Hailage CP2102

Flash ROM Programmer: APKLVSR CH341A

Environment: VMWare Kali Linux

Software:
* Sigrok Pulsview
* NMAP
* Firefox

## Initial Recon

### 1. Visual Inspection
Inspection of the external plastic cover of the router revealed a label that contained the FCC ID of the device

FCC ID: 2AXJ4WR841NV14

![Label](https://github.com/user-attachments/assets/d6d04eba-7be9-4cf4-bfe2-7bc98678748d)
Image 1. Label of the router

![Motherboard_Labeled](https://github.com/user-attachments/assets/5022a262-95de-4ad9-89e9-8cc9715de46d)
Image 2. Picture of motherboard


### 2. On Board Testing
By inspecting the motherboard, it can be noted that 4 through holes labeled VCC, GND, RX and TX can be found on the top right corner of the PCB shown as label E in Image 2. This would most likely indicate an UART connection.

The operating voltage was measured as 9V by measuring the voltage drop across the input jack using a multimeter.

A ground connection P1 was found shown as label B.

The UART connection was verified by taking the following steps:
* VCC: Measured the voltage drop between ground and the VCC as 3.3V
* GND: Verified that the GND pin is connected to ground by checking connectivity between GND and P1 (label B) using a multimeter
* TX: Measured voltage drop between ground and TX  as 3.3V during standard operation
* RX: Measured voltage drop between ground and RX as 0V. Additionally tested connection to ground. No ground connection was found

After confirming the UART connection the voltage drop between ground and TX was measured during bootup. The voltage drop was measured varying between 1v-3.3V indicating active data transmission.

Additional pins were soldered to the UART connection to allow easier access to the pins.

The bootup data transmission were caputred using a logic analyzer

![image](https://github.com/user-attachments/assets/b3a65311-4803-4b9c-a5b2-19bb30c47e97)
Image 3. Screenshot of the capture from PulseView.

The baud rate was measured as 125 kHz. However, when compared to standard baud rates, it is likely the real rate is actually 115200.

An UART decoder was applied based on the measurements.

![image](https://github.com/user-attachments/assets/8c3d6a15-bea9-4398-b17d-e14ecd54fdc2)

Image 4. Screenshot or UART decoder settings

### 3. OSINT and Online Recone
Using the previously obtained FCC ID a lookup was done through fccid.io [https://fccid.io/2AXJ4WR841NV14](https://fccid.io/2AXJ4WR841NV14).

The documents show that the previous FCC ID before a change was TE7WR841NV14

![image](https://github.com/user-attachments/assets/c8f0699b-01d9-4c39-8e4c-249d934beb24)
Image 5. Screenshot of the FCC filing indicating changed FCC ID

Lookup using the old FCC ID revelas detailed photos of the components. The circuit diagram, block diagram and operational description are marked as confidential and not visible to the public.

![image](https://github.com/user-attachments/assets/9c5e9699-55d3-4517-9e18-b50c02440a46)
Image 6. Detailed pictures of chips

Chip A from Image 2 identified as Zentel A3S56D40GTP -50L.

Chip C from Image 3 identfied as MEDIATEK MT7628NN.

Details of chip D could not be found from the documents.

The chip was identified by taking a detailed picture of the physical device.
![image](https://github.com/user-attachments/assets/2632655c-3e3b-467b-9794-40bc7026c3d8)

Image 7. Picture of chip D

Chip D was identified as cFeon QH32B-104HIP.

Data sheets were located online for each of the chips.

MEDIATEK MT7628NN: System On Chip containing CPU. Ppurpose-built SOC for N300 routers. The CPU is MIPS24KEc, which supports Linux 2.6.36 SDK and Linux 3.10 SDK. Interfaces with flash memory via SPI.

Product Page: https://www.mediatek.com/products/home-networking/mt7628k-n-a

Data Sheet: https://files.seeedstudio.com/products/114992470/MT7628_datasheet.pdf

Zentel A3S56D40GTP -50L: DRAM 32Kb

Data Sheet: https://www.mouser.ca/datasheet/2/1130/DSA3S56D340GTPF_02-1984099.pdf

cFeon QH32B-104HIP: Id'd as full part number EN25Q32B-104HIP. Noted as Flash ROM that communicates via SPI.

Data Sheet: https://www.alldatasheet.com/datasheet-pdf/pdf/675622/EON/EN25QH32A.html

Searching for the firmware found it supplied through the official support page for the router.

https://www.tp-link.com/ca/support/download/tl-wr841n/#Firmware


### Initial Network Recon

After connecting to the network hosted by the router a nmap scan was run

![image](https://github.com/user-attachments/assets/f171c339-0045-4fa5-89af-b80b7206a3ec)

Image 8. Nmap TCP scan of the router at 192.168.0.1

The scan revelaed that ports 22 and 80 are open

![image](https://github.com/user-attachments/assets/2abf1d2d-211b-47bb-a99c-b22d9c21a5ec)

Image 9. nmap UDP scan of the router at 192.168.0.1

The UDP scan didn't show any clearly open ports.

### Previous CVEs
A common theme of previous CVEs is a lack of proper user input validation on forms and inputs in the web portal that led to either buffer overflow or command injection of an underlying function or service being called.

https://www.opencve.io/cve?vendor=tp-link&product=tl-wr841n

https://blog.viettelcybersecurity.com/1day-to-0day-on-tl-link-tl-wr841n/

https://ktln2.org/2020/03/29/exploiting-mips-router/


![image](https://github.com/user-attachments/assets/71f540d5-919e-49cf-b0af-b7cdfbf7a3e1)

## Enumeration with UART Connection

### Bootlogs

```
[04080B0B][04080B0D][7E7D0000][26263938][0026263B]
DU Setting Cal Done


U-Boot 1.1.3 (Nov 19 2023 - 18:30:02)

Board: Ralink APSoC DRAM:  32 MB
relocate_code Pointer at: 81fc0000
flash manufacture id: 1c, device id 70 16
Warning: un-recognized chip ID, please update bootloader!
============================================
Ralink UBoot Version: 4.3.0.0
--------------------------------------------
ASIC 7628_MP (Port5<->None)
DRAM component: 256 Mbits DDR, width 16
DRAM bus: 16 bit
Total memory: 32 MBytes
Flash component: SPI Flash
Date:Nov 19 2023  Time:18:30:02
============================================
icache: sets:512, ways:4, linesz:32 ,total:65536
dcache: sets:256, ways:4, linesz:32 ,total:32768

 ##### The CPU freq = 580 MHZ ####
 estimate memory size =32 Mbytes
RESET MT7628 PHY!!!!!!
continue to starting system.                                                                                                                                            0
disable switch phyport...

3: System Boot system code via Flash.(0xbc010000)
do_bootm:argc=2, addr=0xbc010000
## Booting image at bc010000 ...
   Uncompressing Kernel Image ... OK
No initrd

Starting kernel ...

Linux version 2.6.36 (zhang@ubuntu) (gcc version 4.6.3 (Buildroot 2012.11.1) ) #1 Sun Nov 19 18:31:57 PST 2023

 The CPU feqenuce set to 575 MHz

 MIPS CPU sleep mode enabled.
CPU revision is: 00019655 (MIPS 24Kc)
Software DMA cache coherency
Determined physical RAM map:
 memory: 02000000 @ 00000000 (usable)
this command line:  test_mode=disable
Initrd not found or empty - disabling initrd
Zone PFN ranges:
  Normal   0x00000000 -> 0x00002000
Movable zone start PFN for each node
early_node_map[1] active PFN ranges
    0: 0x00000000 -> 0x00002000
Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 8128
Kernel command line: console=ttyS1,115200 root=/dev/mtdblock2 rootfstype=squashfs init=/sbin/init test_mode=disable
PID hash table entries: 128 (order: -3, 512 bytes)
Dentry cache hash table entries: 4096 (order: 2, 16384 bytes)
Inode-cache hash table entries: 2048 (order: 1, 8192 bytes)
Primary instruction cache 64kB, VIPT, , 4-waylinesize 32 bytes.
Primary data cache 32kB, 4-way, PIPT, no aliases, linesize 32 bytes
Writing ErrCtl register=0001a9df
Readback ErrCtl register=0001a9df
Memory: 29228k/32768k available (2180k kernel code, 3540k reserved, 583k data, 156k init, 0k highmem)
NR_IRQS:128
console [ttyS1] enabled
Calibrating delay loop... 386.04 BogoMIPS (lpj=772096)
pid_max: default: 4096 minimum: 301
Mount-cache hash table entries: 512
NET: Registered protocol family 16
bio: create slab <bio-0> at 0
Switching to clocksource Ralink Systick timer
NET: Registered protocol family 2
IP route cache hash table entries: 1024 (order: 0, 4096 bytes)
TCP established hash table entries: 1024 (order: 1, 8192 bytes)
TCP bind hash table entries: 1024 (order: 0, 4096 bytes)
TCP: Hash tables configured (established 1024 bind 1024)
TCP reno registered
NET: Registered protocol family 1
squashfs: version 4.0 (2009/01/31) Phillip Lougher
msgmni has been set to 57
io scheduler noop registered
io scheduler deadline registered (default)
Ralink gpio driver initialized
Serial: 8250/16550 driver, 2 ports, IRQ sharing enabled
serial8250: ttyS0 at MMIO 0x10000d00 (irq = 21) is a 16550A
serial8250: ttyS1 at MMIO 0x10000c00 (irq = 20) is a 16550A
brd: module loaded
flash manufacture id: 1c, device id 70 16
Warning: un-recognized chip ID, please update SPI driver!
EN25QX64A(1c 71171c71) (8192 Kbytes)
mtd .name = raspi, .size = 0x00800000 (8M) .erasesize = 0x00010000 (64K) .numeraseregions = 0
Creating 5 MTD partitions on "raspi":
0x000000000000-0x000000010000 : "boot"
0x000000010000-0x000000100000 : "kernel"
0x000000100000-0x0000003e0000 : "rootfs"
mtd: partition "rootfs" set to be root filesystem
0x0000003e0000-0x0000003f0000 : "config"
0x0000003f0000-0x000000400000 : "radio"
Register flash device:flash0
PPP generic driver version 2.4.2
PPP MPPE Compression module registered
NET: Registered protocol family 24
Mirror/redirect action on
u32 classifier
    Actions configured
Netfilter messages via NETLINK v0.30.
nf_conntrack version 0.5.0 (456 buckets, 1824 max)
ip_tables: (C) 2000-2006 Netfilter Core Team, Type=Linux
TCP cubic registered
NET: Registered protocol family 10
ip6_tables: (C) 2000-2006 Netfilter Core Team
IPv6 over IPv4 tunneling driver
NET: Registered protocol family 17
Ebtables v2.0 registered
802.1Q VLAN Support v1.8 Ben Greear <greearb@candelatech.com>
All bugs added by David S. Miller <davem@redhat.com>
VFS: Mounted root (squashfs filesystem) readonly on device 31:2.
Freeing unused kernel memory: 156k freed
starting pid 29, tty '': '/etc/init.d/rcS'
mount: mounting devpts on /dev/pts failed: No such device
cp: can't stat '/etc/SingleSKU_FCC.dat': No such file or directory
rdm_major = 253
spiflash_ioctl_read, Read from 0x003ff100 length 0x6, ret 0, retlen 0x6
Read MAC from flash(  3ff100) ffffffdc-62-79-11-ffffffaa-14
GMAC1_MAC_ADRH -- : 0x0000dc62
GMAC1_MAC_ADRL -- : 0x7911aa14
Ralink APSoC Ethernet Driver Initilization. v3.1  256 rx/tx descriptors allocated, mtu = 1500!
NAPI enable, Tx Ring = 256, Rx Ring = 256
spiflash_ioctl_read, Read from 0x003ff100 length 0x6, ret 0, retlen 0x6
Read MAC from flash(  3ff100) ffffffdc-62-79-11-ffffffaa-14
GMAC1_MAC_ADRH -- : 0x0000dc62
GMAC1_MAC_ADRL -- : 0x7911aa14
PROC INIT OK!
add domain:tplinkwifi.net
add domain:tplinkap.net
add domain:tplinkrepeater.net
add domain:tplinklogin.net
tp_domain init ok
L2TP core driver, V2.0
PPPoL2TP kernel driver, V2.0
Set: phy[0].reg[0] = 3900
Set: phy[1].reg[0] = 3900
Set: phy[2].reg[0] = 3900
Set: phy[3].reg[0] = 3900
Set: phy[4].reg[0] = 3900
Set: phy[0].reg[0] = 3300
Set: phy[1].reg[0] = 3300
Set: phy[2].reg[0] = 3300
Set: phy[3].reg[0] = 3300
Set: phy[4].reg[0] = 3300
resetMiiPortV over.
Set: phy[0].reg[4] = 01e1
Set: phy[0].reg[0] = 3300
Set: phy[1].reg[4] = 01e1
Set: phy[1].reg[0] = 3300
Set: phy[2].reg[4] = 01e1
Set: phy[2].reg[0] = 3300
Set: phy[3].reg[4] = 01e1
Set: phy[3].reg[0] = 3300
Set: phy[4].reg[4] = 01e1
Set: phy[4].reg[0] = 3300
turn off flow control over.

Please press Enter to activate this console. [ util_execSystem ] 185:  ipt_init cmd is "/var/tmp/dconf/rc.router"

[ dm_readFile ] 2061:  can not open xml file /var/tmp/pc/reduced_data_model.xml!, about to open file /etc/reduced_data_model.xml
spiflash_ioctl_read, Read from 0x003e0000 length 0x10000, ret 0, retlen 0x10000
spiflash_ioctl_read, Read from 0x003e0000 length 0xa1c0, ret 0, retlen 0xa1c0
===>Enter Routerspiflash_ioctl_read, Read from 0x003ff100 length 0x6, ret 0, retlen 0x6
 mode
[ oal_sys_readMaspiflash_ioctl_read, Read from 0x003ff200 length 0x4, ret 0, retlen 0x4
cFlash ] 2061:  spiflash_ioctl_read, Read from 0x003ff300 length 0x4, ret 0, retlen 0x4
 3ff100 set flasspiflash_ioctl_read, Read from 0x003ff400 length 0x10, ret 0, retlen 0x10
h mac : DC:62:79spiflash_ioctl_read, Read from 0x003ff500 length 0x29, ret 0, retlen 0x29
:11:AA:14.
spiflash_ioctl_read, Read from 0x003ff600 length 0x21, ret 0, retlen 0x21
spiflash_ioctl_read, Read from 0x003ff700 length 0x10, ret 0, retlen 0x10
spiflash_ioctl_read, Read from 0x003ff700 length 0x10, ret 0, retlen 0x10
spiflash_ioctl_read, Read from 0x00010000 length 0x1d0, ret 0, retlen 0x1d0
spiflash_ioctl_read, Read from 0x003ff100 length 0x6, ret 0, retlen 0x6
[ oal_sys_readMacFlash ] 2061:   3ff100 set flash mac : DC:62:79:11:AA:14.
[ util_execSystem ] 185:  oal_startDynDns cmd is "dyndns /var/tmp/dconf/dyndns.conf"

Get SNTP new config
[ util_execSystem ] 185:  oal_startNoipDns cmd is "noipdns /var/tmp/dconf/noipdns.conf"

[ util_execSystem ] 185:  oal_startCmxDns cmd is "cmxdns /var/tmp/dconf/cmxdns.conf"

ioctl: No such device
[ util_execSystem ] 185:  oal_br_addBridge cmd is "brctl addbr br0;brctl setfd br0 0;brctl stp br0 off"

[ util_execSystem ] 185:  oal_ipt_addLanRules cmd is "iptables -t filter -A INPUT -i br+ -j ACCEPT
"

[ util_execSystem ] 185:  oal_intf_setIntf cmd is "ifconfig br0 192.168.0.1 netmask 255.255.255.0 up"

[ util_execSystem ] 185:  oal_util_setProcLanARaeth v3.1 (ddr cmd is "echoNAPI
 "br0 16820416,",SkbRecycle > /proc/net/con)
ntract_LocalAddr
phy_tx_ring = 0x00ca2000, tx_ring = 0xa0ca2000
"

[ util_exec
phy_rx_ring0 = 0x00ca3000, rx_ring0 = 0xa0ca3000
System ] 185:  o[fe_sw_init:5357]rt305x_esw_init.
al_intf_enableIntf cmd is "ifconfig eth0 up"

disable switch phyport...
GMAC1_MAC_ADRH -- : 0x0000dc62
GMAC1_MAC_ADRL -- : 0x7911aa14
RT305x_ESW: Link Status Changed
[ rsl_getUnusedVlan ] 1110:  GET UNUSED VLAN TAG 1 : [3]
[ rsl_getUnusedVlan ] 1110:  GET UNUSED VLAN TAG 2 : [4]
[ rsl_getUnusedVlan ] 1110:  GET UNUSED VLAN TAG 3 : [5]
[ rsl_getUnusedVlan ] 1110:  GET UNUSED VLAN TAG 4 : [6]
[ util_execSystem ] 185:  oal_addVlanTagIntf cmd is "vconfig add eth0 3"

[ util_execSystem ] 185:  oal_intf_enableIntf cmd is "ifconfig eth0.3 up"

set if eth0.3 to *not wan dev
[ util_execSystem ] 185:  oal_addVlanTagIntf cmd is "vconfig add eth0 4"

[ util_execSystem ] 185:  oal_intf_enableIntf cmd is "ifconfig eth0.4 up"

set if eth0.4 to *not wan dev
[ util_execSystem ] 185:  oal_addVlanTagIntf cmd is "vconfig add eth0 5"

[ util_execSystem ] 185:  oal_intf_enableIntf cmd is "ifconfig eth0.5 up"

set if eth0.5 tdevice eth0.3 entered promiscuous mode
o *not wan dev
device eth0 entered promiscuous mode
[ util_execSystebr0: port 1(eth0.3) entering forwarding state
m ] 185:  oal_adbr0: port 1(eth0.3) entering forwarding state
dVlanTagIntf cmd is "vconfig add eth0 6"

[ util_execSystem ] device eth0.4 entered promiscuous mode
185:  oal_intf_ebr0: port 2(eth0.4) entering forwarding state
nableIntf cmd isbr0: port 2(eth0.4) entering forwarding state
 "ifconfig eth0.6 up"

set if eth0.6 to *not wan dev
[ util_edevice eth0.5 entered promiscuous mode
xecSystem ] 185:br0: port 3(eth0.5) entering forwarding state
  oal_addVlanTagbr0: port 3(eth0.5) entering forwarding state
Intf cmd is "vconfig add eth0 2"

[ util_execSystem ] 185:  oadevice eth0.6 entered promiscuous mode
l_intf_enableIntbr0: port 4(eth0.6) entering forwarding state
f cmd is "ifconfbr0: port 4(eth0.6) entering forwarding state
ig eth0.2 up"

set if eth0.2 to wan dev
[ vlan_addLanPortsIntoBridge ] 606:  add lan Port 255 from br0
[ util_execSystem ] 185:  oal_br_addIntfIntoBridge cmd is "brctl addif br0 eth0.3"

[ util_execSystem ] 185:  oal_br_addIntfIntoBridge cmd is "brctl addif br0 eth0.4"

[ util_execSystem ] 185:  oal_br_addIntfIntoBridge cmd is "brctl addif br0 eth0.5"

[ util_execSystem ] 185:  oal_br_addIntfIntoBridge cmd is "brctl addif br0 eth0.6"

[ util_execSystem ] 185:  rsl_initIPv6CfgObj cmd is "echo 1 > /proc/sys/net/ipv6/conf/all/disable_ipv6"

[ util_execSystem ] 185:  oal_eth_setIGMPSnoopParam cmd is "for i in /sys/devices/virtual/net/*/bridge/multicast_snooping;do echo 1 > $i ; done"

[ util_execSystem ] 185:  oal_wlan_ra_setCountryRegion cmd is "cp /etc/SingleSKU_CE.dat /var/Wireless/RT2860AP/SingleSKU.dat"

[ util_execSystem ] 185:  oal_wlan_ra_setCountryRegion cmd is "iwpriv ra0 set CountryRegion=1"

ra0       no private ioctls.

[ util_execSystem ] 185:  oal_wlan_ra_loadDriver cmd is "insmod /lib/modules/kmdir/kernel/drivers/net/wireless/mt_wifi_ap/mt_wifi.ko"

ADDRCONF(NETDEV_CHANGE): eth0.4: link becomes ready
ADDRCONF(NETDEV_CHANGE): eth0.5: link becomes ready
ADDRCONF(NETDEV_CHANGE): eth0.6: link becomes ready
ADDRCONF(NETDEV_CHANGE): eth0.2: link becomes ready
mt_wifi: module license 'Proprietary' taints kernel.
Disabling lock debugging due to kernel taint
[ util_execSystem ] 185:  oal_wlan_ra_initWlan cmd is "ifconfig ra0 up"

[RTMPReadParametersHook:297]wifi read profile faild.
efuse_probe: efuse = 10000012
exec!
spiflash_ioctl_read, Read from 0x003f0000 length 0x400, ret 0, retlen 0x400
eeFlashId = 0x7628!
tssi_1_target_pwr_g_band = 36
[ util_execSystem ] 185:  oal_wlan_ra_initWlan cmd is "echo 1 > /proc/tplink/led_wlan_24G"

[ util_execSystem ] 185:  oal_wlan_ra_initWlan cmd is "iwpriv ra0 set ed_chk=0"

[ util_execSystem ] 185:  oal_wlan_ra_setStaNum cmd is "iwpriv ra0 set MaxStaNudevice ra0 entered promiscuous mode
m=32"

[ util_br0: port 5(ra0) entering forwarding state
execSystem ] 185br0: port 5(ra0) entering forwarding state
:  oal_br_addIntfIntoBridge cmd is "brctl addif br0 ra0"

[ util_execSystem ] device apcli0 entered promiscuous mode
185:  oal_br_addIntfIntoBridge cmd is "brctl addif br0 apcli0"
device ra1 entered promiscuous mode

[ util_execSystem ] 185:  oal_br_addIntfIntoBridge cmd is "brctl addif br0 ra1spiflash_ioctl_read, Read from 0x003f0000 length 0x2, ret 0, retlen 0x2
"

[ util_execSystem ] 185:  oal_wlan_ra_initEnd cmd is "wlNetlinkTool &"

[ util_execSystem ] 185:  oal_wlan_ra_initEnd cmd is "killall -q wscd"

[ util_execSystem ] 185:  oal_wlan_ra_initEnd cmd is "wscd -i ra0 -m 1 -w /var/tmp/wsc_upnp/ &"

[ util_execSystem ] 185:  rsl_initLanWlanObj cmd is "echo 0 > /proc/tplink/wl_mode"

WLAN-Start wlNetlinkTool
Waiting for Wireless Events from interfaces...
swWlanChkAhbErr: netlink to do
[ oal_wlan_ra_loadDriver ] 2148:  no 5G chip.


[ rsl_initLanWlanObj ] 9639:  perror:1
wscd: SSDP UDP PORT = 1900
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2030 error
sendto: No such file or directory
pid 78 send 2004 error
[ util_execSystem ] 185:  oal_startDhcps cmd is "dhcpd /var/tmp/dconf/udhcpd.conf"

[ util_execSystem ] 185:  oal_lan6_startDhcp6s cmd is "dhcp6s -c /var/tmp/dconf/dhcp6s_br0.conf -P /var/run/dhcp6s_br0.pid br0 &"

[ util_execSystem ] 185:  oal_lan6_startRadvd cmd is "radvd -C /var/tmp/dconf/radvd_br0.conf -p /var/run/radvd_br0.pid &"

iptables: Bad rule (does a matching rule exist in that chain?).
[ rsl_initEwanObj ] 298: Initialize EWAN, enable(1)!
[ rsl_setEwanObj ] 208: Get Ethernet's stack!
[ rsl_setEwanObj ] 262: enable ethernet interface now!
[ oal_ewan_enable ] 469: pEwan->ifName(eth0.2)
[ util_execSystem ] 185:  oal_br_delIntfFromBridge cmd is "brctl delif br0 eth0.2"

brctl: bridge br0: Invalid argument
[ rsl_setEwanObj ] 268: EWAN.ifname(eth0.2)!
[ wan_conn_wanIpConn_getConnectionInfo ] 906: GET MAC(DC:62:79:11:AA:15) successfully!
[ util_execSystem ] 185:  oal_intf_setIfMac cmd is "ifconfig eth0.2 down"

[ util_execSystem ] 185:  oal_intf_setIfMac cmd is "ifconfig eth0.2 hw ether DC:62:79:11:AA:15 up"

[ util_execSystem ] 185:  oal_intf_enableIntf cmd is "ifconfig eth0.2 up"

[ rsl_initWanPppConnObj ] 398: into rsl_initWanPppConnObj!
[ rsl_initWanPppConnObj ] 515: rsl_initWanPppConnObj successed!
[ rsl_initWanPppConnObj ] 398: into rsl_initWanPppConnObj!
[ rsl_initWanPppConnObj ] 515: rsl_initWanPppConnObj successed!
radvd starting
[Jan 01 00:00:07] radvd: no linklocal address configured for br0
[Jan 01 00:00:08] radvd: error parsing or activating the config file: /var/tmp/dconf/radvd_br0.conf
[ rsl_initAppObj ] 1068:  ==> start dhcp client

[ util_execSystem ] 185:  oal_ipt_fwDdos cmd is "iptables -D FORWARD -j FIREWALL_DDOS
"

iptables: No chain/target/match by that name.
[ util_execSystem ] 185:  oal_ipt_forbidLanPing cmd is "iptables -t filter -D INPUT -i br+ -p icmp --icmp-type echo-request -j DROP
iptables -t filter -D FORWARD -i br+ -p icmp --icmp-type echo-request -j DROP"

iptables: Bad rule (does a matching rule exist in that chain?).
iptables: Bad rule (does a matching rule exist in that chain?).
[ util_execSystem ] 185:  oal_ddos_delPingRule cmd is "iptables -t filter -D INPUT ! -i br+ -p icmp --icmp-type echo-request -j ACCEPT
"

iptables: Bad rule (does a matching rule exist in that chain?).
[ util_execSystem ] 185:  oal_ddos_delPingRule cmd is "iptables -t filter -D INPUT ! -i br+ -p icmp --icmp-type echo-request -d 192.168.0.1 -j DROP
"

iptables: Bad rule (does a matching rule exist in that chain?).
[ util_execSystem ] 185:  oal_ipt_setDDoSRules cmd is "iptables -F FIREWALL_DDOS"

[ util_execSystem ] 185:  ddos_clearAll cmd is "rm -f /var/tmp/dosHost"

sh: diagTool: not found
[ util_execSystem ] 185:  oal_initFirewallObj cmd is "ebtables -N FIREWALL"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -F"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -X"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -P INPUT ACCEPT"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -P FORWARD DROP"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -P OUTPUT ACCEPT"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -N FIREWALL"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -N FWRULE"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -N SETMSS"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -A INPUT -i lo -p ALL -j ACCEPT -m comment                                   --comment "loop back""

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -A INPUT  -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -A INPUT -i br+ -p tcp --dport 23 -j ACCEPT"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -A INPUT -p tcp --dport 23 -j DROP"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -A INPUT -i br+ -p tcp --dport 22 -j ACCEPT"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -A INPUT -p tcp --dport 22 -j DROP"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -A INPUT -i br+ -p icmpv6 --icmpv6-type echo-request -j ACCEPT"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -A INPUT -p icmpv6 --icmpv6-type echo-request -j DROP"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -A FORWARD -o br+ -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -A FORWARD -i br+ -j ACCEPT"

[ util_execSystem ] 185:  oal_initIp6FirewallObj cmd is "ip6tables -I FORWARD 1 -j SETMSS"

[ util_execSystem ] 185:  oal_fw6_setFwEnabeld cmd is "ip6tables -D FIREWALL -j ACCEPT"

ip6tables: Bad rule (does a matching rule exist in that chain?).
[ util_execSystem ] 185:  oal_fw6_setFwEnabeld cmd is "ip6tables -F FIREWALL"

[ util_execSystem ] 185:  oal_fw6_setFwEnabeld cmd is "ip6tables -A FIREWALL -j ACCEPT"

[ rsl_initWanL2tpConnObj ] 245: L2TP Connection(ewan_l2tp) is not enable.

[ rsl_initWanL2tpConnObj ] 245: L2TP Connection() is not enable.

[ rsl_initWanPptpConnObj ] 239: PPTP Connection(ewan_pptp) is not enable.

[ rsl_initWanPptpConnObj ] 239: PPTP Connection() is not enable.

[ util_execSystem ] 185:  setupModules cmd is "insmod /lib/modules/kmdir/kernel/net/netfilter/nf_conntrack_ftp.ko"

[ util_execSystem ] 185:  setupModules cmd is "insmod /lib/modules/kmdir/kernel/net/ipv4/netfilter/nf_nat_ftp.ko"

[ util_execSystem ] 185:  oal_openAlg cmd is "iptables -D FORWARD_VPN_PASSTHROUGH  -p udp --dport 500 -j DROP"

iptables: Bad rule (does a matching rule exist in that chain?).
[ util_execSystem ] 185:  setupModules cmd is "insmod /lib/modules/kmdir/kernel/net/ipv4/netfilter/nf_nat_proto_gre.ko"

[ util_execSystem ] 185:  setupModules cmd is "insmod /lib/modules/kmdir/kernel/net/ipv4/netfilter/nf_nat_pptp.ko"

[ util_execSystem ] 185:  oal_openAlg cmd is "iptables -D FORWARD_VPN_PASSTHROUGH  -p tcp --dport 1723 -j DROP"

iptables: Bad rule (does a matching rule exist in that chain?).
[ util_execSystem ] 185:  oal_openAlg cmd is "iptables -D FORWARD_VPN_PASSTHROUGH  -p udp --dport 1701 -j DROP"

iptables: Bad rule (does a matching rule exist in that chain?).
[ util_execSystem ] 185:  setupModules cmd is "insmod /lib/modules/kmdir/kernel/net/netfilter/nf_conntrack_tftp.ko"

[ util_execSystem ] 185:  setupModules cmd is "insmod /lib/modules/kmdir/kernel/net/ipv4/netfilter/nf_nat_tftp.ko"

[ util_execSystem ] 185:  setupModules cmd is "insmod /lib/modules/kmdir/kernel/net/netfilter/nf_conntrack_h323.ko"

[ util_execSystem ] 185:  setupModules cmd is "insmod /lib/modules/kmdir/kernel/net/ipv4/netfilter/nf_nat_h323.ko"

[ util_execSystem ] 185:  setupModules cmd is "insmod /lib/modules/kmdir/kernel/net/netfilter/nf_conntrack_sip.ko"

[ util_execSystem ] 185:  setupModules cmd is "insmod /lib/modules/kmdir/kernel/net/ipv4/netfilter/nf_nat_sip.ko"

[ util_execSystem ] 185:  setupModules cmd is "insmod /lib/modules/kmdir/kernel/net/netfilter/nf_conntrack_rtsp.ko"

[ util_execSystem ] 185:  setupModules cmd is "insmod /lib/modules/kmdir/kernel/net/ipv4/netfilter/nf_nat_rtsp.ko"

nf_nat_rtsp v0.6.21 loading
[ cos_init ] 480:  don't start tenable switch phyport...
ddp at user mode

Set: phy[0].reg[0] = 3900
Set: phy[1].reg[0] = 3900
Set: phy[2].reg[0] = 3900
Set: phy[3].reg[0] = 3900
Set: phy[4].reg[0] = 3900
Set: phy[0].reg[0] = 3300
Set: phy[1].reg[0] = 3300
Set: phy[2].reg[0] = 3300
Set: phy[3].reg[0] = 3300
Set: phy[4].reg[0] = 3300
resetMiiPortV over.
Set: phy[0].reg[4] = 01e1
Set: phy[0].reg[0] = 3300
Set: phy[1].reg[4] = 01e1
Set: phy[1].reg[0] = 3300
Set: phy[2].reg[4] = 01e1
Set: phy[2].reg[0] = 3300
Set: phy[3].reg[4] = 01e1
Set: phy[3].reg[0] = 3300
Set: phy[4].reg[4] = 01e1
Set: phy[4].reg[0] = 3300
turn off flow control over.
[ util_execSystem ] 185:  prepareDropbear cmd is "dropbearkey -t rsa -f /var/tmp/dropbear/dropbear_rsa_host_key"

Will output 1024 bit rsa secret key to '/var/tmp/dropbear/dropbear_rsa_host_key'
Generating key, this may take a while...
[ util_execSystem ] 185:  prepareDropbear cmd is "dropbearkey -t dss -f /var/tmp/dropbear/dropbear_dss_host_key"

Will output 1024 bit dss secret key to '/var/tmp/dropbear/dropbear_dss_host_key'
Generating key, this may take a while...
[ util_execSystem ] 185:  prepareDropbear cmd is "dropbear -p 22 -r /var/tmp/dropbear/dropbear_rsa_host_key -d /var/tmp/dropbear/dropbear_dss_host_key -A /var/tmp/dropbear/dropbearpwd"

start ntp_request
[ oal_sys_getOldTZInfo ] 609:  Open TZ file error!
[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

[ ntp_start ] 504:  ntp connect failed, return.

[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

Get SNTP start config
start ntp_request
[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

[ ntp_start ] 504:  ntp connect failed, return.

[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

Get SNTP start config
start ntp_request
[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

[ ntp_start ] 504:  ntp connect failed, return.

[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

Get SNTP start config
start ntp_request
[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

[ ntp_start ] 504:  ntp connect failed, return.

[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

Get SNTP start config
start ntp_request
[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

[ rsl_setWanIpConnObj ] 674: Into wan_conn_wanIpConn_handleStatusSetOpt!
[ wan_conn_wanIpConn_handleStatusSetOpt ] 712: pNewObj->connectionStatus = Disconnected
[ wan_conn_wanIpConn_handleStatusSetOpt ] 713: pCurrObj->connectionStatus = Unconfigured
[ ntp_start ] 504:  ntp connect failed, return.

[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

Get SNTP start config
start ntp_request
[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

[ ntp_start ] 504:  ntp connect failed, return.

[ util_execSystem ] 185:  oal_sys_unsetTZ cmd is "echo "" > /etc/TZ"

[ rsl_setWanIpConnObj ] 674: Into wan_conn_wanIpConn_handleStatusSetOpt!
[ wan_conn_wanIpConn_handleStatusSetOpt ] 712: pNewObj->connectionStatus = Disconnected
[ wan_conn_wanIpConn_handleStatusSetOpt ] 713: pCurrObj->connectionStatus = Disconnected
[ rsl_setWanIpConnObj ] 674: Into wan_conn_wanIpConn_handleStatusSetOpt!
[ wan_conn_wanIpConn_handleStatusSetOpt ] 712: pNewObj->connectionStatus = Disconnected
[ wan_conn_wanIpConn_handleStatusSetOpt ] 713: pCurrObj->connectionStatus = Disconnected
```

At the time of testing in 2025 It smees that the EU version of the router doesn't spawn a shell during the boot even after removing the R18 SMD resistor located closely to the RX pin of the UART connection. It is possible the that RX pin is disabled by design.

The German (TPD) version gives a shell after removing the resistor

[https://tounsec.re/en/posts/2024/05/extracting-system-files-from-a-tl-wr841n-v14/](https://tounsec.re/en/posts/2024/05/extracting-system-files-from-a-tl-wr841n-v14/)

On the other hand, TP-Link TL-WR840N(EU) seem to spawn a shell.
[https://martinhogberg.se/gaining-a-uart-root-shell-on-the-tp-link-tl-wr840n/](https://martinhogberg.se/gaining-a-uart-root-shell-on-the-tp-link-tl-wr840n/)

## Generating additional log messages

By setting up the router as an access point with a password (Quick setup --> Access Point) the following log messages are promted

```
[ util_execSystem ] 185:  oal_wlan_ra_setWlanAdvCfg cmd is "iwpriv ra0 set DtimPeriod=1"

[ util_execSystem ] 185:  oal_wlan_ra_setWlanAdvCfg cmd is "iwpriv ra0 set HtGi=1"

[ util_execSystem ] 185:  oal_wlan_ra_setWlanAdvCfg cmd is "iwpriv ra0 set HideSSID=0"

[ util_execSystem ] 185:  oal_wlan_ra_setWlanAdvCfg cmd is "iwpriv ra0 set WmmCapable=1"

[ util_execSystem ] 185:  oal_wlan_ra_setWlanAdvCfg cmd is "iwpriv ra0 set NoForwarding=0"

[ util_execSystem ] 185:  oal_wlan_ra_setSec cmd is "iwpriv ra0 set AuthMode=WPA2PSK"

[ util_execSystem ] 185:  oal_wlan_ra_setSec cmd is "iwpriv ra0 set EncrypType=AES"

[ util_execSystem ] 185:  oal_wlan_ra_setSec cmd is "iwpriv ra0 set IEEE8021X=0"

[ util_execSystem ] 185:  oal_wlan_ra_setSec cmd is "iwpriv ra0 set DefaultKeyID=2"

[ util_execSystem ] 185:  oal_wlan_ra_setSec cmd is "iwpriv ra0 set SSID='T''P''-''L''i''n''k''_''A''A''1''4'"

[ util_execSystem ] 185:  oal_wlan_ra_setSec cmd is "iwpriv ra0 set RekeyInterval=0"

[ util_execSystem ] 185:  oal_wlan_ra_setSec cmd is "iwpriv ra0 set WPAPSK='P''a''s''s''w''o''r''d''1''2''3'"

[ util_execSystem ] 185:  oal_wlan_ra_updateWlanCfg cmd is "iwpriv ra0 set AccessPolicy=0"

[ util_execSystem ] 185:  oal_wlan_ra_updateWlanCfg cmd is "iwpriv ra0 set SSID='T''P''-''L''i''n''k''_''A''A''1''4'"

[ util_execSystem ] 185:  oal_wlan_ra_updateWlanCfg cmd is "ifconfig ra1 down"

[ util_execSystem ] 185:  oal_wlan_ra_updateWlanCfg cmd is "ifconfig ra0 down; "
```

A function called `util_execSystem` is executed repeatedly and seems to allow system coommands to be run. 

It seems that the user-supplied input is validated by escaping each charected individually with quotes.

### Interesting findings

Version of the secondary bootloader: `U-Boot 1.1.3 (Nov 19 2023 - 18:30:02), Ralink UBoot Version: 4.3.0.0`

Linux version: Linux version 2.6.36 `(zhang@ubuntu) (gcc version 4.6.3 (Buildroot 2012.11.1) ) #1 Sun Nov 19 18:31:57 PST 2023`

Bootloader default: `3: System Boot system code via Flash.(0xbc010000)`

The bootload sequence could be interrupted

CPU: CPU revision is: `00019655 (MIPS 24Kc)`

Boot partitions:

```
EN25QX64A(1c 71171c71) (8192 Kbytes)
mtd .name = raspi, .size = 0x00800000 (8M) .erasesize = 0x00010000 (64K) .numeraseregions = 0
Creating 5 MTD partitions on "raspi":
0x000000000000-0x000000010000 : "boot"
0x000000010000-0x000000100000 : "kernel"
0x000000100000-0x0000003e0000 : "rootfs"
mtd: partition "rootfs" set to be root filesystem
0x0000003e0000-0x0000003f0000 : "config"
0x0000003f0000-0x000000400000 : "radio"
```

Config files: `can not open xml file /var/tmp/pc/reduced_data_model.xml!, about to open file /etc/reduced_data_model.xml`

Possible writable path: `[ util_execSystem ] 185:  prepareDropbear cmd is "dropbearkey -t rsa -f /var/tmp/dropbear/dropbear_rsa_host_key"`

## Interrupting Bootloader

The bootloader can be interrupted by sending the string "tlp" to the RX pin during boot up.

```
[04080B0B]04080B0D][7E7D0000][2flash manufacture id:  ID, please update bootloader!
============================================ 
Ralink UBoot Version: 4.3.0.0
-------------------------------------------- 
ASIC 7628_MP (Port5<->None)
DRAM component: 256 Mbits DDR, width 16
DRAM bus: 16 bit
Total memory: 32 MBytes
Flash component: SPI Flash
Date:Nov 19 2023  Time:18:30:02
================================ 
icacal:65536
dcache: sets:2O, Read MAC A Flash

swptpltptpl 
Unkno
MT7628 #
MT7628 # help
Unknown command 'help' - try 'help'
MT7628 # 

```

However, the shell is limited and the only command available is `tftpboot`

# Firmware Extraction

## Extraction from ROM

The datasheet of the EN25Q32B-104HIP chip specifies that SPI is used to communicate with the processor. This was confirmed using a logic analyzer.

![image](https://github.com/user-attachments/assets/519cac38-65d8-4de0-a34d-d53e49cc8c3d)

The firmware was extracted using Rasberry pi's SPI pins with the following command

`sudo flashrom -r TP_LINK.img -p linux_spi:dev=/dev/spidev0.0,spispeed=8000`

![image](https://github.com/user-attachments/assets/ab92f69e-efd3-4bec-80a0-0cf66b46a28b)

## Firmware Analysis

Running binwalk on the firmware extraction showed three partitions, matching the ones found previously.

![image](https://github.com/user-attachments/assets/24bb0acc-8b2e-4215-a70f-9df5139b62ac)


Using binwalk to extract the partitions

![image](https://github.com/user-attachments/assets/23a9ec70-a105-48af-a55a-b7530332b9ed)

Binwalk was not able to automatically extract the U-boot partition

Extracting manually using dd

![image](https://github.com/user-attachments/assets/f5b160f3-f8d5-4cb8-98ee-03f98adb224e)

Looking through the filesystem two encrypted xml files can be found

![image](https://github.com/user-attachments/assets/9b04467d-64f2-47f8-92b5-6143e2b55acf)

Encrypted files imply that there is a decrypt function in the firmare

Both the XML files are referenced in libcmm.o located in the lib folder

![image](https://github.com/user-attachments/assets/d9d2474b-eb4f-4215-a0d5-ee65628931cf)

## Decryption Function

Using Ghidra a decryptFile function was located in licmm.so library file

```
int dm_decryptFile(uint param_1,undefined4 param_2,uint param_3,int param_4)
{
  int iVar1;
  undefined auStack_28 [8];
  int local_20;
  
  memcpy(auStack_28,&DAT_000c9f90,8);
  if (param_3 < param_1) {
    cdbg_printf(8,"dm_decryptFile",0xbcf,
                "Buffer exceeded, decrypt buf size is %u, but dm file size is %u",param_3,param_1);
    local_20 = 0;
  }
  else {
    local_20 = cen_desMinDo(param_2,param_1,param_4,param_3,auStack_28,0);
    iVar1 = local_20;
    if (local_20 == 0) {
      cdbg_printf(8,"dm_decryptFile",0xbd6,"DES decrypt error\n");
    }
    else {
      do {
        local_20 = iVar1;
        if (((undefined *)(param_4 + local_20))[-1] != '\0') break;
        iVar1 = local_20 + -1;
      } while (local_20 != 0);[ util_execSystem ] 185:  prepareDropbear cmd is "dropbearkey -t rsa -f /var/tmp/dropbear/dropbear_rsa_host_key"
      *(undefined *)(param_4 + local_20) = 0;
    }
  }
  return local_20;
}
```
Based on the print function it seems DES encryption is used. DES uses 8x8 bytes keys and there appears to be a buffer auStack_28[8] which mathces the size.

The function copies contents from the pointer &DAT_000c9f90. 

![image](https://github.com/user-attachments/assets/7634542d-c049-45d7-8865-175883523176)

DES Encryption Key: 478da50ff9e3d2cb   

Decrypt using openssl

`openssl enc -d -des-ecb -K 478DA50FF9E3D2CB -nopad -in default_config.xml > default_config_dec.xml `

However, there were no notable findings in the decrypted xml files

`openssl enc -d -des-ecb -K 478DA50FF9E3D2CB -nopad -in reduced_data_model.xml > reduzed_data_model_dec.xml`

## Checking for Possible Command Injection

With Ghidra the following decompiled source code for the `util_execSystem` function can be found:

```
int util_execSystem(int param_1,char *param_2,undefined4 param_3,undefined4 param_4)

{
  char *pcVar1;
  __pid_t _Var2;
  uint uVar3;
  undefined4 uVar4;
  int iVar5;
  int iVar6;
  undefined4 local_res8;
  undefined4 local_resc;
  char *__s;
  uint local_234;
  char acStack_230 [512];
  int local_30;
  
  __s = acStack_230;
  local_234 = 0;
  local_res8 = param_3;
  local_resc = param_4;
  memset(__s,0,0x200);
  local_30 = vsnprintf(__s,0x1ff,param_2,&local_res8);
  cdbg_printf(8,"util_execSystem",0xb9,"%s cmd is \"%s\"\n",param_1,__s);
  iVar5 = 1;
  if (0 < local_30) {
    while( true ) {
      local_234 = system(acStack_230);
      uVar3 = local_234 & 0x7f;
      if ((int)local_234 < 0) {
        if (local_234 == 0xffffffff) {
          cdbg_printf(8,"util_execSystem",0xcd,"system fork failed.",param_1,__s);
        }
        else {
          perror("util_execSystem call error:");
        }
      }
      else if (uVar3 == 0) {
        iVar6 = (int)(local_234 & 0xff00) >> 8;
        if (iVar6 == 0) {
          return 0;
        }
        if (iVar6 != 4) {
          return iVar6;
        }
        pcVar1 = strstr(acStack_230,"iptable");
        if (pcVar1 == (char *)0x0) {
          return 4;
        }
        sleep(1);
      }
      else {
        if ((int)((uVar3 + 1) * 0x1000000) >> 0x19 < 1) {
          if ((local_234 & 0xff) == 0x7f) {
            uVar4 = 0xe6;
            pcVar1 = "process stopped, signal number = %d\n";
            uVar3 = (int)(local_234 & 0xff00) >> 8;
          }
          else {
            uVar4 = 0xe8;
            pcVar1 = "Oh,no possible here. status = %d\n";
            uVar3 = local_234;
          }
        }
        else {
          uVar4 = 0xe4;
          pcVar1 = "abnormal termination, signal number = %d\n";
        }
        cdbg_printf(8,"util_execSystem",uVar4,pcVar1,uVar3,__s);
        while (_Var2 = waitpid(-1,(int *)&local_234,1), 0 < _Var2) {
          cdbg_printf(8,"util_execSystem",0xee,"get a zombie process %d",_Var2);
        }
      }
      if (iVar5 == 3) break;
      param_1 = iVar5;
      cdbg_printf(8,"util_execSystem",199,"system execute again, and %d times",iVar5);
      iVar5 = iVar5 + 1;
    }
  }
  return -1;
}

```

By looking at the code it can be noted that the function executes systemcalls based on the parameter2 argument. The util_execSystem itself does not validate the input. The only formatting done is by the vsnprint function which creates the command string. This means that the validation of user input is done by functions calling the util_execSystem function.

User input is validated by escaping strings given by the user. This is done to prevent possible command injections from happening. For example, an user might input a semicolon (;) and execute additonal commands. Now this is does not work since quotes are added to each character.

The string "iwpriv %s set SSID=%s" can be found in the same libcmm.so file 

![image](https://github.com/user-attachments/assets/81841a70-5010-4d42-b602-8fdb6b2de739)

The validation function can be found from the references

```

/* WARNING: Switch with 1 destination removed at 0x000aa304 */
/* WARNING: Switch with 1 destination removed at 0x000aa3d4 */
/* WARNING: Exceeded maximum restarts with more pending */

int FUN_000aa208(int param_1,undefined4 param_2,char *param_3,int param_4,undefined4 param_5)

{
  int iVar1;
  undefined4 uVar2;
  char *pcVar3;
  int local_1c0;
  int local_1bc;
  char acStack_1b8 [12];
  char acStack_1ac [20];
  undefined auStack_198 [32];
  undefined auStack_178 [32];
  undefined auStack_158 [100];
  undefined auStack_f4 [196];
  char *local_30;
  
  local_1bc = 1;
  local_1c0 = 1;
  cstr_strncpy(auStack_178,&DAT_000c1e5c,0xc);
  cstr_strncpy(auStack_198,&DAT_000c3040,4);
  iVar1 = oal_wlan_getSecMode(param_1,&local_1bc,&local_1c0);
  if (iVar1 == 0) {
    if (local_1bc - 1U < 0xb) {
                    /* WARNING: Could not find normalized switch variable to match jumptable */
                    /* WARNING: This code block may not be properly labeled as switch case */
      strcpy(acStack_1ac,"OPEN");
    }
    uVar2 = 2;
    if (local_1c0 - 1U < 5) {
                    /* WARNING: Could not find normalized switch variable to match jumptable */
                    /* WARNING: This code block may not be properly labeled as switch case */
      uVar2 = 2;
      strcpy(acStack_1b8,"NONE");
    }
    else if (local_1c0 == 2) {
      uVar2 = *(undefined4 *)(param_1 + 0x17c);
    }
    iVar1 = strcmp("Up",param_3);
    if (iVar1 != 0) {
      return 0;
    }
    util_execSystem("oal_wlan_ra_setSec","iwpriv %s set AuthMode=%s",param_2,acStack_1ac);
    util_execSystem("oal_wlan_ra_setSec","iwpriv %s set EncrypType=%s",param_2,acStack_1b8);
    util_execSystem("oal_wlan_ra_setSec","iwpriv %s set IEEE8021X=0",param_2);
    if (local_1c0 == 2) {
      if (*(char *)(param_1 + 0x180) != '\0') {
        FUN_000aa15c(auStack_f4,param_1 + 0x180);
        util_execSystem("oal_wlan_ra_setSec","iwpriv %s set Key1=%s",param_2,auStack_f4);
      }
      if (*(char *)(param_1 + 0x210) != '\0') {
        FUN_000aa15c(auStack_f4,param_1 + 0x210);
        util_execSystem("oal_wlan_ra_setSec","iwpriv %s set Key2=%s",param_2,auStack_f4);
      }
      if (*(char *)(param_1 + 0x2a0) != '\0') {
        FUN_000aa15c(auStack_f4,param_1 + 0x2a0);
        util_execSystem("oal_wlan_ra_setSec","iwpriv %s set Key3=%s",param_2,auStack_f4);
      }
      if (*(char *)(param_1 + 0x330) != '\0') {
        FUN_000aa15c(auStack_f4,param_1 + 0x330);
        util_execSystem("oal_wlan_ra_setSec","iwpriv %s set Key4=%s",param_2,auStack_f4);
      }
    }
    util_execSystem("oal_wlan_ra_setSec","iwpriv %s set DefaultKeyID=%d",param_2,uVar2);
    if (local_1c0 - 1U < 2) {
      return 0;
    }
    FUN_000aa15c(auStack_158,param_5);
    util_execSystem("oal_wlan_ra_setSec","iwpriv %s set SSID=%s",param_2,auStack_158);
    util_execSystem("oal_wlan_ra_setSec","iwpriv %s set RekeyInterval=%d",param_2,
                    *(undefined4 *)(param_1 + 0xf0));
    if ((((local_1bc - 6U < 2) || (local_1bc == 9)) || (local_1bc == 10)) || (local_1bc == 0xb)) {
      FUN_000aa15c(auStack_f4,param_1 + 0xae);
      util_execSystem("oal_wlan_ra_setSec","iwpriv %s set WPAPSK=%s",param_2,auStack_f4);
      return 0;
    }
    iVar1 = oal_wlan_getBrNamebyIfName(param_4,param_4,auStack_198);
    if (iVar1 == 0) {
      iVar1 = oal_wlan_getIfAddr(auStack_198,auStack_178);
      if (iVar1 == 0) {
        local_30 = "PppConn_RemoveSubObj";
        iVar1 = strcmp("2.4GHz",(char *)(param_4 + 1099));
        if (iVar1 == 0) {
          pcVar3 = "killall -q  -SIGINT rt2860apd";
        }
        else {
          pcVar3 = "killall -q  -SIGINT rtinicapd";
        }
        util_execSystem("oal_wlan_ra_setSec",pcVar3);
        util_execSystem("oal_wlan_ra_setSec","iwpriv %s set IEEE8021X=0",param_2);
        util_execSystem("oal_wlan_ra_setSec","iwpriv %s set SSID=%s",param_2,auStack_158);
        util_execSystem("oal_wlan_ra_setSec","iwpriv %s set RADIUS_Server=%s",param_2,param_1 + 0xf4
                       );
        util_execSystem("oal_wlan_ra_setSec","iwpriv %s set RADIUS_Port=%d",param_2,
                        *(undefined4 *)(param_1 + 0x134));
        FUN_000aa15c(auStack_f4,param_1 + 0x138);
        util_execSystem("oal_wlan_ra_setSec","iwpriv %s set RADIUS_Key=%s",param_2,auStack_f4);
        util_execSystem("oal_wlan_ra_setSec","iwpriv %s set EAPifname=%s",param_2,auStack_198);
        util_execSystem("oal_wlan_ra_setSec","iwpriv %s set own_ip_addr=%s",param_2,auStack_178);
        util_execSystem("oal_wlan_ra_setSec","iwpriv %s set SSID=%s",param_2,auStack_158);
        sleep(4);
        iVar1 = strcmp("2.4GHz",(char *)(param_4 + 1099));
        if (iVar1 == 0) {
          pcVar3 = "rt2860apd &";
        }
        else {
          pcVar3 = "rtinicapd &";
        }
        util_execSystem("oal_wlan_ra_setSec",pcVar3);
        return 0;
      }
      uVar2 = 0xbce;
    }
    else {
      uVar2 = 0xbc9;
    }
  }
  else {
    uVar2 = 0xb2c;
  }
  cdbg_perror("oal_wlan_ra_setSec",uVar2,iVar1);
  return iVar1;
}


```

The input (param_5) is validated by the `FUN_000aa15c(auStack_158,param_5);` call before being passed on to the util_execSystem function.


```
void FUN_000aa15c(int param_1,char *param_2)

{
  char cVar1;
  size_t sVar2;
  undefined *puVar3;
  int iVar4;
  int iVar5;
  
  sVar2 = strlen(param_2);
  iVar4 = 0;
  for (iVar5 = 0; puVar3 = (undefined *)(param_1 + iVar4), iVar5 < (int)sVar2; iVar5 = iVar5 + 1) {
    if (*param_2 == '\'') {
      *puVar3 = 0x5c;
      iVar4 = iVar4 + 2;
      puVar3[1] = *param_2;
    }
    else {
      *puVar3 = 0x27;
      cVar1 = *param_2;
      iVar4 = iVar4 + 3;
      puVar3[2] = 0x27;
      puVar3[1] = cVar1;
    }
    param_2 = param_2 + 1;
  }
  *puVar3 = 0;
  return;
}
```

The escaping appears to be properly implemented and functions as intended.

However, according to Ghidra there are 510 external references to the `util_execSystem` function.

![image](https://github.com/user-attachments/assets/2526086f-b547-413d-9a06-220ba150d506)

There is a possiblity that there is an instance where the `util_execSystem` function is called without the first escaping the input with the validtion function. Therefore, the usage of the validation function should be verified for all the instances.

## Extracting config partition

After applying SSID and PSK we can see the following config entries:

```
spiflash_ioctl_erase, erase to 0x003f0000 length 0x0, nerase done
spiflash_ioctl_write, Write to 0x003e0000 length 0x10000, ret 0, retlen 0x10000
```
This indicates that the config is at that location

Boot partitions:

```
EN25QX64A(1c 71171c71) (8192 Kbytes)
mtd .name = raspi, .size = 0x00800000 (8M) .erasesize = 0x00010000 (64K) .numeraseregions = 0
Creating 5 MTD partitions on "raspi":
0x000000000000-0x000000010000 : "boot"
0x000000010000-0x000000100000 : "kernel"
0x000000100000-0x0000003e0000 : "rootfs"
mtd: partition "rootfs" set to be root filesystem
0x0000003e0000-0x0000003f0000 : "config"
0x0000003f0000-0x000000400000 : "radio"
```

Extracting config partitino using dd

![image](https://github.com/user-attachments/assets/0a643957-da45-4a29-816f-e9d7e099bce2)

Running backup and restore to possibly trigger decryption of the config partition

![image](https://github.com/user-attachments/assets/579aee42-03b9-4780-b411-969ecb6fa7d7)


Nothing notable output from UART

Running a corrupted config file to try and trigger a error message

![image](https://github.com/user-attachments/assets/50365122-de58-4678-8a80-4126144e9381)

```
[ rsl_sys_restoreCfg ] 2167:  Config file MD5 check fail

[ rdp_restoreCfg ] 342:  perror:4501
```

The rsl_sys_restoreCfg fucntion can be found in the libcmm.so file

```
undefined4 rsl_sys_restoreCfg(void *param_1,int param_2)

{
  bool bVar1;
  int iVar2;
  uint uVar3;
  int iVar4;
  uint __n;
  int *piVar5;
  undefined4 uVar6;
  int *piVar7;
  char *pcVar8;
  int iVar9;
  void *pvVar10;
  undefined4 uVar11;
  uint uVar12;
  
  iVar2 = cmem_getSharedBuffSize();
  pvVar10 = (void *)((int)param_1 + param_2);
  memset(pvVar10,0,iVar2 - param_2);
  uVar3 = cen_desMinDo(param_1,param_2,pvVar10,iVar2 - param_2,&DAT_000f01f8,0);
  if (uVar3 == 0) {
    uVar6 = 0x852;
    pcVar8 = "DES decrypt error\n";
LAB_0002c8fc:
    uVar11 = 1;
    cdbg_printf(8,"rsl_sys_restoreCfg",uVar6,pcVar8);
  }
  else {
    if (uVar3 < 0x11) {
      cdbg_printf(8,"rsl_sys_restoreCfg",0x866,"File size %d is too small\n",uVar3);
    }
    else {
      uVar3 = uVar3 - 0x10;
      uVar12 = iVar2 - 0x41830;
      if (uVar12 < uVar3) {
        uVar6 = 0x870;
LAB_0002ca04:
        cdbg_printf(8,"rsl_sys_restoreCfg",uVar6,
                    "Compress data is too long, available size is %d bytes, now is %d bytes",uVar12,
                    uVar3);
        return 0x1194;
      }
      iVar4 = cen_md5VerifyDigest(pvVar10,(void *)((int)pvVar10 + 0x10),uVar3);
      if (iVar4 == 0) {
        uVar6 = 0x877;
        pcVar8 = "Config file MD5 check fail\n";
      }
      else {
        memcpy(param_1,(void *)((int)pvVar10 + 0x10),uVar3);
        pvVar10 = (void *)((int)param_1 + uVar3);
        memset(pvVar10,0,iVar2 - uVar3);
        __n = cen_uncompressBuff(param_1,pvVar10,iVar2 - uVar3);
        if (__n != 0) {
          iVar4 = dm_restoreCfg(pvVar10,__n,1);
          if (iVar4 == 0) {
            dm_cleanupCfg(8);
            iVar4 = dm_restoreCfg(pvVar10,__n,0);
            if (iVar4 == 0) {
              if (__n < 0xfff1) {
                memset(param_1,0,uVar3);
                *(uint *)((int)pvVar10 + -0x10) = __n;
                *(undefined4 *)((int)pvVar10 + -0xc) = 0x98765432;
                *(undefined4 *)((int)pvVar10 + -4) = 0;
                *(undefined4 *)((int)pvVar10 + -8) = 0;
                *(undefined *)((int)pvVar10 + __n) = 0;
LAB_0002ca98:
                piVar5 = (int *)((int)pvVar10 + -0x10);
                iVar4 = 0;
                uVar3 = 0;
                piVar7 = piVar5;
                while (bVar1 = uVar3 != *piVar5 + 0x10U >> 2, uVar3 = uVar3 + 1, bVar1) {
                  iVar9 = *piVar7;
                  piVar7 = piVar7 + 1;
                  iVar4 = iVar4 + iVar9;
                }
                *(int *)((int)pvVar10 + -8) = -iVar4;
                iVar2 = oal_sys_writeCfgFlash(piVar5,*piVar5 + 0x10U,iVar2);
                if (iVar2 == 0) {
                  return 0;
                }
                cdbg_printf(8,"rsl_sys_restoreCfg",0x925,"Write user config to flash error.");
                return 0;
              }
              memcpy(param_1,pvVar10,__n);
              pvVar10 = (void *)((int)param_1 + __n);
              memset(pvVar10,0,iVar2 - __n);
              uVar3 = cen_compressBuff(param_1,__n,(int)param_1 + iVar2 + -0x8000,pvVar10);
              if (uVar3 != 0) {
                if (uVar12 < uVar3) {
                  uVar6 = 0x8f1;
                  goto LAB_0002ca04;
                }
                memset(param_1,0,__n);
                *(undefined4 *)((int)pvVar10 + -0xc) = 0x98765432;
                *(uint *)((int)pvVar10 + -0x10) = uVar3;
                *(undefined4 *)((int)pvVar10 + -4) = 1;
                *(undefined4 *)((int)pvVar10 + -8) = 0;
                *(undefined *)((int)pvVar10 + uVar3) = 0;
                goto LAB_0002ca98;
              }
              uVar6 = 0x8e9;
              pcVar8 = "compress data error!";
              goto LAB_0002c8fc;
            }
            uVar6 = 0x8c8;
          }
          else {
            uVar6 = 0x8bf;
          }
          pcVar8 = "Set config into DM error\n";
          goto LAB_0002c8fc;
        }
        uVar6 = 0x896;
        pcVar8 = "uncompress data error!\n";
      }
      cdbg_printf(8,"rsl_sys_restoreCfg",uVar6,pcVar8);
    }
    uVar11 = 0x1195;
  }
  return uVar11;
}
```

Here the oal_sys_writeCfgFlash function is used to do the encryption.


```

undefined4 oal_sys_writeCfgFlash(int *param_1,int param_2)

{
  int iVar1;
  undefined4 uVar2;
  uint uVar3;
  int local_60;
  undefined auStack_5c [36];
  undefined auStack_38 [44];
  
  local_60 = 0;
  memset(auStack_38,0,0x21);
  memset(auStack_5c,0,0x21);
  memcpy(auStack_38,gKey,0x20);
  memcpy(auStack_5c,gIv,0x20);
  local_60 = param_2 + -0x10;
  iVar1 = aes_cbc_encrypt_intface_bypart(param_1 + 4,&local_60,auStack_38,auStack_5c);
  if (iVar1 < 0) {
    cdbg_printf(8,"oal_sys_writeCfgFlash",0x707,"Encrypt flash error %d",iVar1);
  }
  param_1[3] = param_1[3] + 2;
  *param_1 = local_60;
  uVar3 = local_60 + 0x10;
  if (uVar3 < 0x10001) {
    iVar1 = FUN_000983c0(0x3e0000,uVar3,param_1);
    uVar2 = 0;
    if (iVar1 < 0) {
      cdbg_printf(8,"oal_sys_writeCfgFlash",0x717,"Write config error\n");
      uVar2 = 0x232a;
    }
  }
  else {
    cdbg_printf(8,"oal_sys_writeCfgFlash",0x711,"Config file length too long - %d\n",uVar3);
    uVar2 = 0x1194;
  }
  return uVar2;
}

```

aes cvc encryption is used and the gKey and gIV are saved in memory

![image](https://github.com/user-attachments/assets/bb16c516-18dd-423d-8538-13a4f389eb63)

IV = 1528632946736539
Key = 1528632946736109

Decrypt using CyberChef

Convert binary to hex that can be copied

![image](https://github.com/user-attachments/assets/42a024a8-6573-4f30-a589-86753714640b)

AES decrypt

![image](https://github.com/user-attachments/assets/912792ca-10ef-46fe-b3ec-1eda88729872)


