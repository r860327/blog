openwork + 3g usb modem(中兴 MF667s)
====================================

安装如下应用包
--------------

First install required packages:

::

   * comgt manpage for comgt
   * Appropriate host controller interface for your USB hardware (precompiled images will most likely already contain the correct one)
       * kmod-usb2 (aka EHCI)
       * kmod-usb-ohci
       * kmod-usb-uhci (for example VIA chips)
   * Support for serial communication; needed to send control commands to the dongle and receive its responses.
   * kmod-usb-serial, and
   * kmod-usb-serial-option, and
   * kmod-usb-serial-wwan, or
   * kmod-usb-acm i.s.o. the last two, depending on dongle/phone hardware.
       * kmod-usb-serial-option is not available for 2.4 kernel, install kmod-usb-serial and put a line equivalent to "usbserial vendor=0x12d1 product=0x1003 maxSize=2048" in /etc/modules.d/60-usb-serial)
   * modeswitching tools, if your modem initially presents itself as a storage device - one of the following, depending on your modem:
       * usb-modeswitch and usb-modeswitch-data (recommended) A mode switching tool for controlling "flip flop" (multiple device) USB gear.
   Note: as of r36812, usb-modeswitch package had been under major overhaul from ordinary draisberghof usb_modeswitch found in many linux distributions. usb-modeswitch-data is included in the package, with the new json format.
   * sdparm - utility to send SCSI commands (needed on Ovation MC935D)
   * luci-proto-3g for proper support in luci in RC6 and later

   opkg update
   opkg install comgt kmod-usb-serial kmod-usb-serial-option kmod-usb-serial-wwan usb-modeswitch usb-modeswitch-data

使用option GSM 驱动usb切换后的串口。
------------------------------------

注意，这里的vid和pid为切换到usb modem模式后的vid和pid。

::

   insmod option #skip this if option driver is loaded already
   echo '<vid> <pid> ff' > /sys/bus/usb-serial/drivers/option1/new_id

使用usb-modeswitch切换usb模式
-----------------------------

1. 创建 /etc/usb_modeswitch.conf 文件，并添加如下类容，这些内容由usb
   modem设备制造商提供

::

   DefaultVendor=0x19d2
   DefaultProduct=0x1588

   TargetVendor=0x19d2
   TargetProduct=0x1589                       MessageContent="55534243123456782000000080000c85010101180101010101000000000000"

2. 执行 usb_modeswitch -W -c /etc/usb_modeswitch.conf -I 命令
   查看dmesg是否有如下输出，并查看/dev下是否有ttyUSB0~ttyUSB3

::

   [426955.836000] option 1-1.2:1.0: GSM modem (1-port) converter detected
   [426955.844000] usb 1-1.2: GSM modem (1-port) converter now attached to ttyUSB0
   [426955.860000] option 1-1.2:1.1: GSM modem (1-port) converter detected
   [426955.868000] usb 1-1.2: GSM modem (1-port) converter now attached to ttyUSB1
   [426955.884000] option 1-1.2:1.2: GSM modem (1-port) converter detected
   [426955.892000] usb 1-1.2: GSM modem (1-port) converter now attached to ttyUSB2
   [426955.908000] option 1-1.2:1.3: GSM modem (1-port) converter detected
   [426955.916000] usb 1-1.2: GSM modem (1-port) converter now attached to ttyUSB3
   [426955.936000] scsi17 : usb-storage 1-1.2:1.4
   [426956.940000] scsi 17:0:0:0: CD-ROM            HSDPA    CDROM Storage    2.31PQ: 0 ANSI: 2
   [426956.948000] scsi 17:0:0:0: Attached scsi generic sg0 type 5
   [426956.960000] scsi 17:0:0:1: Direct-Access     HSDPA    MMC Storage      2.31PQ: 0 ANSI: 2
   [426956.968000] sd 17:0:0:1: Attached scsi generic sg1 type 0
   [426956.992000] sd 17:0:0:1: [sda] Attached SCSI removable disk

::

    同时可以查看 /proc/bus/usb/devices 文件，注意红色的部分。

::

   T:  Bus=01 Lev=01 Prnt=01 Port=00 Cnt=01 Dev#=  3 Spd=480  MxCh= 0
   D:  Ver= 2.00 Cls=02(comm.) Sub=00 Prot=00 MxPS=64 #Cfgs=  1
   P:  Vendor=19d2 ProdID=1589 Rev= 0.00
   S:  Manufacturer=ZTE,Incorporated
   S:  Product=ZTE Mobile Broadband Station
   S:  SerialNumber=1234567890ABCDEF
   C:* #Ifs= 7 Cfg#= 1 Atr=a0 MxPwr=500mA
   A:  FirstIf#= 0 IfCount= 2 Cls=02(comm.) Sub=06 Prot=00
   I:* If#= 0 Alt= 0 #EPs= 1 Cls=02(comm.) Sub=06 Prot=00 Driver=(none)
   E:  Ad=88(I) Atr=03(Int.) MxPS=  64 Ivl=125us
   I:* If#= 1 Alt= 0 #EPs= 0 Cls=0a(data ) Sub=00 Prot=00 Driver=(none)
   I:  If#= 1 Alt= 1 #EPs= 2 Cls=0a(data ) Sub=00 Prot=00 Driver=(none)
   E:  Ad=81(I) Atr=02(Bulk) MxPS= 512 Ivl=0ms
   E:  Ad=01(O) Atr=02(Bulk) MxPS= 512 Ivl=0ms
   I:* If#= 2 Alt= 0 #EPs= 3 Cls=ff(vend.) Sub=ff Prot=ff Driver=option
   E:  Ad=87(I) Atr=03(Int.) MxPS=  64 Ivl=500us
   E:  Ad=82(I) Atr=02(Bulk) MxPS= 512 Ivl=0ms
   E:  Ad=02(O) Atr=02(Bulk) MxPS= 512 Ivl=0ms
   I:* If#= 3 Alt= 0 #EPs= 2 Cls=ff(vend.) Sub=ff Prot=ff Driver=option
   E:  Ad=83(I) Atr=02(Bulk) MxPS= 512 Ivl=0ms
   E:  Ad=03(O) Atr=02(Bulk) MxPS= 512 Ivl=0ms
   I:* If#= 4 Alt= 0 #EPs= 2 Cls=ff(vend.) Sub=ff Prot=ff Driver=option
   E:  Ad=84(I) Atr=02(Bulk) MxPS= 512 Ivl=0ms
   E:  Ad=04(O) Atr=02(Bulk) MxPS= 512 Ivl=0ms
   I:* If#= 5 Alt= 0 #EPs= 2 Cls=ff(vend.) Sub=ff Prot=ff Driver=option
   E:  Ad=85(I) Atr=02(Bulk) MxPS= 512 Ivl=0ms
   E:  Ad=05(O) Atr=02(Bulk) MxPS= 512 Ivl=0ms
   I:* If#= 6 Alt= 0 #EPs= 2 Cls=08(stor.) Sub=06 Prot=50 Driver=usb-storage
   E:  Ad=86(I) Atr=02(Bulk) MxPS= 512 Ivl=0ms
   E:  Ad=06(O) Atr=02(Bulk) MxPS= 512 Ivl=0ms

配置网络接口
------------

::

    编辑 /etc/config/network 文件

::

   config interface wan
   #   option ifname ppp0 # on some carriers enable this line
       option pincode 1234
       option device /dev/ttyUSB0
       option ap n 3gnet
       option service umts
       option proto 3g

配置拨号脚本
------------

::

    修改 /etc/chatscripts/3g.chat 文件，

1. 运行，这个命令可以注册网络。 gcom -d /dev/ttyUSB0
2. 运行‘ifup wan’拨号。要等上个几秒钟

::

   ABORT   BUSY
   ABORT   'NO CARRIER'
   ABORT   ERROR
   REPORT  CONNECT
   TIMEOUT 12
   ""      "AT&F"
   OK      "ATE1"
   OK      'AT+CGDCONT=1,"IP","$USE_APN"'
   ABORT   'NO CARRIER'
   TIMEOUT 15
   OK      "ATD*99#"
   CONNECT ' '

这里dmesg里没有信息。可以查看usb modem卡上的等（蓝色常亮）。

使用ifconfig查看
----------------

::

   3g-wan    Link encap:Point-to-Point Protocol
   inet addr:10.10.243.197  P-t-P:10.64.64.64  Mask:255.255.255.255
   UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  Metric:1
   RX packets:5592 errors:0 dropped:0 overruns:0 frame:0
   TX packets:8693 errors:0 dropped:0 overruns:0 carrier:0
   collisions:0 txqueuelen:3
   RX bytes:2283335 (2.1 MiB)  TX bytes:1330400 (1.2 MiB)

参考 openwork 官方文档
----------------------

http://wiki.openwrt.org/doc/recipes/3gdongle
