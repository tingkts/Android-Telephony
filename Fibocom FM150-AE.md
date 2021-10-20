
## [Fibocom FM150-AE](https://www.fibocom.com/en/products/5G-FM150-AE.html)

### Porting check items

- USB Serial Option driver

- Qualcomm GobiNet or QMI_WWAN driver&emsp;[🔗](https://blog.csdn.net/qlexcel/article/details/117150901)

- Vendor RIL

- APN

- [device](https://cs.android.com/android/platform/superproject/+/android-10.0.0_r41:device/) configuration

</br>

### Runtime @ device

```console
imx8mp_evk:/vendor # find | grep -w "rild"
./etc/init/rild.rc
./bin/hw/rild

imx8mp_evk:/vendor # cat ./etc/init/rild.rc
service vendor.ril-daemon /vendor/bin/hw/rild -l /vendor/lib64/libreference-ril.so -- -w 2
    class main
    user radio
    group radio cache inet misc audio sdcard_rw log readproc wakelock
    capabilities BLOCK_SUSPEND NET_ADMIN NET_RAW


imx8mp_evk:/vendor # find | grep -w "ril.so"
./lib64/libreference-ril.so
./lib/libreference-ril.so


imx8mp_evk:/vendor # getprop | grep -i -E "ril|radio"
[gsm.version.ril-impl]: [RIL_V1.0.10]
[init.svc.vendor.radio-config-hal-1-0]: [running]
[init.svc.vendor.ril-daemon]: [running]
[persist.radio.multisim.config]: [ssss]
[ril.currentapntype]: [default]
[ril.fibocom.version]: [RIL_V1.0.10]
[rild.libargs]: [-d /dev/ttyUSB2]
[rild.libpath]: [/vendor/lib64/libreference-ril.so]
[ro.boottime.vendor.radio-config-hal-1-0]: [14389603125]
[ro.boottime.vendor.ril-daemon]: [14640582250]
[vendor.rild.libpath]: [/vendor/lib64/libreference-ril.so]


imx8mp_evk:/vendor # ps -A | grep -E -i "radio|ril"
system         332     1   39260   5828 binder_thread_read  0 S android.hardware.radio.config@1.0-service
radio          362     1  115764   7740 binder_thread_read  0 S rild
radio          745   304 3264076 119500 do_epoll_wait       0 S com.android.phone

```


<br>


### Modem Device Node, Driver Node

```console
imx8mp_evk:/ # ls -al /dev/ttyUSB*
crw-rw---- 1 radio radio 188,   0 2000-03-25 03:32 /dev/ttyUSB0
crw-rw---- 1 radio radio 188,   1 2000-03-25 03:32 /dev/ttyUSB1
crw-rw---- 1 radio radio 188,   2 2021-10-13 17:07 /dev/ttyUSB2    // port of AT command
crw-rw---- 1 radio radio 188,   3 2000-03-25 03:32 /dev/ttyUSB3


imx8mp_evk:/ # ls -al /sys/bus/usb/drivers
total 0
drwxr-xr-x 30 root root 0 2021-10-13 17:15 .
drwxr-xr-x  4 root root 0 2021-10-13 17:15 ..
drwxr-xr-x  2 root root 0 2021-10-13 17:15 GobiNet              // GobiNet driver
drwxr-xr-x  2 root root 0 2021-10-13 17:15 aiptek
drwxr-xr-x  2 root root 0 2021-10-13 17:15 asix
drwxr-xr-x  2 root root 0 2021-10-13 17:15 ax88179_178a
drwxr-xr-x  2 root root 0 2021-10-13 17:15 cdc_ether
drwxr-xr-x  2 root root 0 2021-10-13 17:15 cdc_ncm
drwxr-xr-x  2 root root 0 2021-10-13 17:15 cdc_subset
drwxr-xr-x  2 root root 0 2021-10-13 17:15 ftdi_sio
drwxr-xr-x  2 root root 0 2021-10-13 17:15 gtco
drwxr-xr-x  2 root root 0 2021-10-13 17:15 hanwang
drwxr-xr-x  2 root root 0 2021-10-13 17:15 hub
drwxr-xr-x  2 root root 0 2021-10-13 17:15 kbtab
drwxr-xr-x  2 root root 0 2021-10-13 17:15 net1080
drwxr-xr-x  2 root root 0 2021-10-13 17:15 option              // usb serial option driver
drwxr-xr-x  2 root root 0 2021-10-13 17:15 r8152
drwxr-xr-x  2 root root 0 2021-10-13 17:15 snd-usb-audio
drwxr-xr-x  2 root root 0 2021-10-13 17:15 uas
drwxr-xr-x  2 root root 0 2021-10-13 17:15 usb
drwxr-xr-x  2 root root 0 2021-10-13 17:15 usb-storage
drwxr-xr-x  2 root root 0 2021-10-13 17:15 usb_acecad
drwxr-xr-x  2 root root 0 2021-10-13 17:15 usb_ehset_test
drwxr-xr-x  2 root root 0 2021-10-13 17:15 usb_serial_simple
drwxr-xr-x  2 root root 0 2021-10-13 17:15 usbfs
drwxr-xr-x  2 root root 0 2021-10-13 17:15 usbhid
drwxr-xr-x  2 root root 0 2021-10-13 17:15 usbserial_generic
drwxr-xr-x  2 root root 0 2021-10-13 17:15 uvcvideo
drwxr-xr-x  2 root root 0 2021-10-13 17:15 xpad
drwxr-xr-x  2 root root 0 2021-10-13 17:15 zaurus


imx8mp_evk:/ # ifconfig
dummy0    Link encap:Ethernet  HWaddr ee:77:08:3c:b3:f3
          inet6 addr: fe80::ec77:8ff:fe3c:b3f3/64 Scope: Link
          UP BROADCAST RUNNING NOARP  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:8 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 TX bytes:560

eth0      Link encap:Ethernet  HWaddr 1e:71:c2:0d:75:92  Driver imx-dwmac
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 TX bytes:0
          Interrupt:46

usb0      Link encap:Ethernet  HWaddr 00:30:30:30:30:30  Driver GobiNet         // GobiNet NIC network interface
          inet addr:10.35.255.69  Bcast:10.35.255.71  Mask:255.255.255.252
          inet6 addr: fe80::230:30ff:fe30:3030/64 Scope: Link
          UP BROADCAST RUNNING NOARP MULTICAST  MTU:1500  Metric:1
          RX packets:33 errors:28 dropped:0 overruns:0 frame:0
          TX packets:41 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:10133 TX bytes:6336

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope: Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 TX bytes:0
```



<br>

### Use case of AT command

```console
imx8mp_evk:/ # cat /dev/ttyUSB2 &

imx8mp_evk:/ # echo -e 'AT+CGMI?\r' > /dev/ttyUSB2
+CGMI: "Fibocom Wireless Inc."
OK

imx8mp_evk:/ # echo -e 'AT+GTDUALSIM?\r' > /dev/ttyUSB2
+GTDUALSIM: 0,"SUB1","No Service"
OK

imx8mp_evk:/ # echo -e 'AT+GTDUALSIM=?\r' > /dev/ttyUSB2
+GTDUALSIM: (0,1)
OK
```





<br>
<br>
<br>



Ref. module of [Fibocom FM150-AE](https://www.fibocom.com/en/products/5G-FM150-AE.html), based on [i.MX 8M Plus](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors/i-mx-8-processors/i-mx-8m-plus-arm-cortex-a53-machine-learning-vision-multimedia-and-industrial-iot:IMX8MPLUS) [Android 10](https://cs.android.com/android/platform/superproject/+/android-10.0.0_r41:)

