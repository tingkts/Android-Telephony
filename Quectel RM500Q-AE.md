## [Quectel RM500Q-AE](https://www.quectel.com/product/5g-rm500q-ae) by PCI-E interface

### Porting check items

- PCI-E port is ready, often needs to check dts nodes of pcie

- PCI-E MHI driver which embedded QMI network interface

- Vendor RIL &ensp; - &ensp; [*IRadio 1.4*](https://github.com/tingkts/Android-Telephony/tree/main/compatibility_matrix.5.xml%20requires%20IRadio1.4)

- APN

- [device](https://cs.android.com/android/platform/superproject/+/android-10.0.0_r41:device/) configuration





### Runtime @ device

```console
evk_8mp:/ # find | grep -w "rild"
./vendor/etc/init/rild.rc
./vendor/bin/hw/rild

evk_8mp:/vendor # cat ./etc/init/rild.rc
service vendor.ril-daemon /vendor/bin/hw/rild  -l /vendor/lib64/libreference-ril.so -- -d /dev/mhi_DUN
    class main
    user radio
    group radio cache inet misc audio sdcard_rw log readproc wakelock
    capabilities BLOCK_SUSPEND NET_ADMIN NET_RAW


evk_8mp:/vendor # find | grep -w "ril.so"
./lib64/libreference-ril.so
./lib64/libril.so
... omit /lib, it's the same as /lib64 ...

evk_8mp:/ # getprop | grep -i ril
[gsm.version.ril-impl]: [Quectel_Android_RIL_Driver_V3.3.72_v1.4_dirty]
[init.svc.vendor.ril-daemon]: [running]
[init.svc_debug_pid.vendor.ril-daemon]: [509]
[rild.libargs]: [-d /dev/mhi_DUN]
[rild.libpath]: [/vendor/lib64/libreference-ril.so]
[ro.boottime.vendor.ril-daemon]: [14133315125]
[vendor.rild.libargs]: [-d /dev/mhi_DUN]
[vendor.rild.libpath]: [/vendor/lib64/libreference-ril.so]

evk_8mp:/ # getprop | grep -i radio
[persist.radio.multisim.config]: [ssss]

evk_8mp:/ # getprop | grep -i tel
[cache_key.telephony.get_active_data_sub_id]: [-6077550228791884224]
[cache_key.telephony.get_default_data_sub_id]: [-6077550228791884236]
[cache_key.telephony.get_default_sms_sub_id]: [-6077550228791884230]
[cache_key.telephony.get_default_sub_id]: [-6077550228791884232]
[cache_key.telephony.get_slot_index]: [-6077550228791884231]
[gsm.operator.alpha]: [Chunghwa Telecom]
[gsm.sim.operator.alpha]: [中華電信_Chunghwa Telecom]
[gsm.version.ril-impl]: [Quectel_Android_RIL_Driver_V3.3.72_v1.4_dirty]
[ro.telephony.default_network]: [22,20]

evk_8mp:/ # getprop | grep -i phon
[cache_key.telephony.get_active_data_sub_id]: [-6077550228791884224]
[cache_key.telephony.get_default_data_sub_id]: [-6077550228791884236]
[cache_key.telephony.get_default_sms_sub_id]: [-6077550228791884230]
[cache_key.telephony.get_default_sub_id]: [-6077550228791884232]
[cache_key.telephony.get_slot_index]: [-6077550228791884231]
[gsm.current.phone-type]: [1]
[ro.telephony.default_network]: [22,20]


evk_8mp:/ # ps -A | grep -i -E "ril|radio|tel|phon"
radio           509      1 8756800   5800 binder_wait_for_work 0 S rild
radio           892    403 12061180 104396 do_epoll_wait      0 S com.android.phone

```


<br>


### Modem Device Node, Driver Node

```console
evk_8mp:/ # ls -al /dev | grep -i mhi
crw-rw----  1 radio     radio     508,   0 2022-02-08 11:57 mhi_BHI
crw-rw----  1 radio     radio     509,   1 2022-02-08 11:57 mhi_DIAG
crw-rw----  1 radio     radio     509,   3 2022-02-08 11:57 mhi_DUN         // AT command port
crw-rw----  1 radio     radio     509,   0 2022-02-08 11:57 mhi_LOOPBACK
crw-rw----  1 radio     radio     509,   2 2022-02-08 11:57 mhi_QMI0


lspci
01:00.0 Class ff00: 17cb:0306		// rm500q-ae
00:00.0 Class 0604: 16c3:abcd

./sys/bus/pci/devices
./sys/bus/pci/devices/0000:00:00.0
./sys/bus/pci/devices/0000:01:00.0


evk_8mp:/ # ls -al /sys/bus/pci/drivers
total 0
./sys/bus/pci/drivers/ehci-pci
./sys/bus/pci/drivers/ufshcd
./sys/bus/pci/drivers/xhci_hcd
./sys/bus/pci/drivers/wlan_pcie
./sys/bus/pci/drivers/pcieport
./sys/bus/pci/drivers/mhi_q         // QMI_WWAN by pcie
./sys/bus/pci/drivers/serial
./sys/bus/pci/drivers/dwc3-haps


evk_8mp:/ # ifconfig

rmnet_mhi0 Link encap:UNSPEC    Driver mhi_netdev
          inet6 addr: fe80::6d15:1d4c:f416:6469/64 Scope: Link
          UP RUNNING NOARP  MTU:1500  Metric:1
          RX packets:32 errors:0 dropped:0 overruns:0 frame:0
          TX packets:61 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:9012 TX bytes:6144

rmnet_mhi0.1 Link encap:Ethernet  HWaddr 02:50:f4:00:00:00
          inet addr:10.175.188.67  Mask:255.255.255.248
          inet6 addr: fe80::50:f4ff:fe00:0/64 Scope: Link
          UP RUNNING NOARP  MTU:1500  Metric:1
          RX packets:39 errors:0 dropped:0 overruns:0 frame:0
          TX packets:61 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:8687 TX bytes:6497
```



<br>

### Use case of AT command

```console
echo -e 'ati\r\n' > /dev/mhi_DUN
echo -e 'AT+QCFG?\r\n' > /dev/mhi_DUN
echo -e 'AT+QSIMDET?\r\n' > /dev/mhi_DUN
echo -e 'AT+QSIMSTAT?\r\n' > /dev/mhi_DUN
echo -e 'AT+CPIN?\r\n' > /dev/mhi_DUN

echo -e 'AT+CSQ=?\r\n' > /dev/mhi_DUN
echo -e 'AT+CSQ\r\n' > /dev/mhi_DUN
```





<br>
<br>
<br>



Ref. &nbsp;5G module [Quectec 5G RM500Q-AE](https://www.quectel.com/product/5g-rm500q-ae), based on [i.MX 8M Plus](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors/i-mx-8-processors/i-mx-8m-plus-arm-cortex-a53-machine-learning-vision-multimedia-and-industrial-iot:IMX8MPLUS) [Android 11](https://cs.android.com/android/platform/superproject/+/android-11.0.0_r40:)