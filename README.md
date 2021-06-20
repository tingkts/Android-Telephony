
## Overview

<img src="./res/RIL structure.PNG" alt="RIL structure" style="zoom:45%;" />




<br>
<br>


## Telephony App/Service

[packages/services/Telephony](https://cs.android.com/android/platform/superproject/+/android-10.0.0_r30:packages/services/Telephony/;bpv=0)

[packages/services/Telecomm](https://cs.android.com/android/platform/superproject/+/android-10.0.0_r30:packages/services/Telecomm/;bpv=0)



<br>

## Telephony Provider

[packages/providers/TelephonyProvider](https://cs.android.com/android/platform/superproject/+/android-10.0.0_r30:packages/providers/TelephonyProvider/;bpv=0)

<br>

```
// e.g. APN conf file, database

imx8mp_evk:/ # system/etc/apn-conf.xml

imx8mp_evk:/ # data/user_de/0/com.android.providers.telephony/databases
$ ls -al
total 1100
drwxr-xr-x 1 ting 1049089      0 Mar 17 13:06 ./
drwxr-xr-x 1 ting 1049089      0 Mar 17 13:06 ../
-r--r--r-- 1 ting 1049089  20480 Mar 17 12:06 CarrierInformation.db
-r--r--r-- 1 ting 1049089      0 Mar 17 12:06 CarrierInformation.db-journal
-r--r--r-- 1 ting 1049089  57344 Mar 17 12:06 HbpcdLookup.db
-r--r--r-- 1 ting 1049089      0 Mar 17 12:06 HbpcdLookup.db-journal
-r--r--r-- 1 ting 1049089 229376 Mar 17 12:06 carrierIdentification.db
-r--r--r-- 1 ting 1049089      0 Mar 17 12:06 carrierIdentification.db-journal
-r--r--r-- 1 ting 1049089 118784 Mar 17 12:06 mmssms.db
-r--r--r-- 1 ting 1049089      0 Mar 17 12:06 mmssms.db-journal
-r--r--r-- 1 ting 1049089 696320 Mar 17 12:06 telephony.db
-r--r--r-- 1 ting 1049089      0 Mar 17 12:06 telephony.db-journal

```



- [Android：一篇就够！全面&详细解析APN（涉及内容：GGSN，authtype，MVNO，pdp，Apns-conf，supl,hipri,dun）](https://blog.csdn.net/GentelmanTsao/article/details/103234535)


<br>


## Telephony Framework


[frameworks/opt/telephony/](http://androidxref.com/9.0.0_r3/xref/frameworks/opt/telephony/) <br>
[frameworks/base/telephony/](http://androidxref.com/9.0.0_r3/xref/frameworks/base/telephony/)

- The client of HIDL Radio service of [IRadio](http://androidxref.com/9.0.0_r3/xref/hardware/interfaces/radio/)

  [frameworks/opt/telephony/src/java/com/android/internal/telephony/RIL.java](http://androidxref.com/9.0.0_r3/xref/frameworks/opt/telephony/src/java/com/android/internal/telephony/RIL.java)

```java
// frameworks/opt/telephony/src/java/com/android/internal/telephony/RIL.java

39 import android.hardware.radio.V1_0.IRadio;

190    volatile IRadio mRadioProxy = null;

351    /** Returns a {@link IRadio} instance or null if the service is not available. */
352    @VisibleForTesting
353    public IRadio getRadioProxy(Message result) {
354        if (!mIsMobileNetworkSupported) {
355            if (RILJ_LOGV) riljLog("getRadioProxy: Not calling getService(): wifi-only");
356            if (result != null) {
357                AsyncResult.forMessage(result, null,
358                        CommandException.fromRilErrno(RADIO_NOT_AVAILABLE));
359                result.sendToTarget();
360            }
361            return null;
362        }
363
364        if (mRadioProxy != null) {
365            return mRadioProxy;
366        }
367
368        try {
369            mRadioProxy = IRadio.getService(HIDL_SERVICE_NAME[mPhoneId == null ? 0 : mPhoneId],
370                    true);
371            if (mRadioProxy != null) {
372                mRadioProxy.linkToDeath(mRadioProxyDeathRecipient,
373                        mRadioProxyCookie.incrementAndGet());
374                mRadioProxy.setResponseFunctions(mRadioResponse, mRadioIndication);
375            } else {
376                riljLoge("getRadioProxy: mRadioProxy == null");
377            }
378        } catch (RemoteException | RuntimeException e) {
379            mRadioProxy = null;
380            riljLoge("RadioProxy getService/setResponseFunctions: " + e);
381        }
382
383        if (mRadioProxy == null) {
384            // getService() is a blocking call, so this should never happen
385            riljLoge("getRadioProxy: mRadioProxy == null");
386            if (result != null) {
387                AsyncResult.forMessage(result, null,
388                        CommandException.fromRilErrno(RADIO_NOT_AVAILABLE));
389                result.sendToTarget();
390            }
391        }
392
393        return mRadioProxy;
394    }

527    @Override
528    public void getIccCardStatus(Message result) {
529        IRadio radioProxy = getRadioProxy(result);
530        if (radioProxy != null) {
531            RILRequest rr = obtainRequest(RIL_REQUEST_GET_SIM_STATUS, result,
532                    mRILDefaultWorkSource);
533
534            if (RILJ_LOGD) riljLog(rr.serialString() + "> " + requestToString(rr.mRequest));
535
536            try {
537                radioProxy.getIccCardStatus(rr.mSerial);
538            } catch (RemoteException | RuntimeException e) {
539                handleRadioProxyExceptionForRR(rr, "getIccCardStatus", e);
540            }
541        }
542    }
```

- The client of HIDL Radio service of [IRadioConfig](http://androidxref.com/9.0.0_r3/xref/hardware/interfaces/radio/config/)

  [frameworks/opt/telephony/src/java/com/android/internal/telephony/RadioConfig.java](http://androidxref.com/9.0.0_r3/xref/frameworks/opt/telephony/src/java/com/android/internal/telephony/RadioConfig.java)

```java
// frameworks/opt/telephony/src/java/com/android/internal/telephony/RadioConfig.java

26 import android.hardware.radio.config.V1_0.IRadioConfig;

56    private volatile IRadioConfig mRadioConfigProxy = null;

152    /** Returns a {@link IRadioConfig} instance or null if the service is not available. */
153    public IRadioConfig getRadioConfigProxy(Message result) {
154        if (!mIsMobileNetworkSupported) {
155            if (VDBG) logd("getRadioConfigProxy: Not calling getService(): wifi-only");
156            if (result != null) {
157                AsyncResult.forMessage(result, null,
158                        CommandException.fromRilErrno(RADIO_NOT_AVAILABLE));
159                result.sendToTarget();
160            }
161            return null;
162        }
163
164        if (mRadioConfigProxy != null) {
165            return mRadioConfigProxy;
166        }
167
168        try {
169            mRadioConfigProxy = IRadioConfig.getService(true);
170            if (mRadioConfigProxy != null) {
171                mRadioConfigProxy.linkToDeath(mServiceDeathRecipient,
172                        mRadioConfigProxyCookie.incrementAndGet());
173                mRadioConfigProxy.setResponseFunctions(mRadioConfigResponse,
174                        mRadioConfigIndication);
175            } else {
176                loge("getRadioConfigProxy: mRadioConfigProxy == null");
177            }
178        } catch (RemoteException | RuntimeException e) {
179            mRadioConfigProxy = null;
180            loge("getRadioConfigProxy: RadioConfigProxy getService/setResponseFunctions: " + e);
181        }
182
183        if (mRadioConfigProxy == null) {
184            // getService() is a blocking call, so this should never happen
185            loge("getRadioConfigProxy: mRadioConfigProxy == null");
186            if (result != null) {
187                AsyncResult.forMessage(result, null,
188                        CommandException.fromRilErrno(RADIO_NOT_AVAILABLE));
189                result.sendToTarget();
190            }
191        }
192
193        return mRadioConfigProxy;
194    }

240    /**
241     * Wrapper function for IRadioConfig.getSimSlotsStatus().
242     */
243    public void getSimSlotsStatus(Message result) {
244        IRadioConfig radioConfigProxy = getRadioConfigProxy(result);
245        if (radioConfigProxy != null) {
246            RILRequest rr = obtainRequest(RIL_REQUEST_GET_SLOT_STATUS, result, mDefaultWorkSource);
247
248            if (DBG) {
249                logd(rr.serialString() + "> " + requestToString(rr.mRequest));
250            }
251
252            try {
253                radioConfigProxy.getSimSlotsStatus(rr.mSerial);
254            } catch (RemoteException | RuntimeException e) {
255                resetProxyAndRequestList("getSimSlotsStatus", e);
256            }
257        }
258    }
259
```

[frameworks/opt/telephony/src/java/com/android/internal/telephony/RadioConfigResponse.java](http://androidxref.com/9.0.0_r3/xref/frameworks/opt/telephony/src/java/com/android/internal/telephony/RadioConfigResponse.java)

```java
// frameworks/opt/telephony/src/java/com/android/internal/telephony/RadioConfigResponse.java
40    /**
41     * Response function for IRadioConfig.getSimSlotsStatus().
42     */
43    public void getSimSlotsStatusResponse(RadioResponseInfo responseInfo,
44                                          ArrayList<SimSlotStatus> slotStatus) {
45        RILRequest rr = mRadioConfig.processResponse(responseInfo);
46
47        if (rr != null) {
48            ArrayList<IccSlotStatus> ret = RadioConfig.convertHalSlotStatus(slotStatus);
49            if (responseInfo.error == RadioError.NONE) {
50                // send response
51                RadioResponse.sendMessageResponse(rr.mResult, ret);
52                Rlog.d(TAG, rr.serialString() + "< "
53                        + mRadioConfig.requestToString(rr.mRequest) + " " + ret.toString());
54            } else {
55                rr.onError(responseInfo.error, ret);
56                Rlog.e(TAG, rr.serialString() + "< "
57                        + mRadioConfig.requestToString(rr.mRequest) + " error "
58                        + responseInfo.error);
59            }
60
61        } else {
62            Rlog.e(TAG, "getSimSlotsStatusResponse: Error " + responseInfo.toString());
63        }
64    }
```

<br>

## Radio HIDL

[HIDL Interface of Radio](http://androidxref.com/9.0.0_r3/xref/hardware/interfaces/radio/)

```shell
ting@ting-pc:~/aosp/android-9/hardware/interfaces/radio
$ ls -al
total 48
drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 ./
drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 ../
drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 1.0/
drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 1.1/
drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 1.2/
drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 1.3/
drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 1.4/
drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 1.5/
drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 1.6/
drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 config/
drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 deprecated/
```

- [IRadio](http://androidxref.com/9.0.0_r3/xref/hardware/interfaces/radio/)

  [IRadio](http://androidxref.com/9.0.0_r3/xref/hardware/interfaces/radio/) is implemented by [libril](http://androidxref.com/9.0.0_r3/xref/hardware/ril/libril/)

  ```shell
  # e.g. java: android.hardware.radio.1.1.IRadio
  # e.g. c/C++: android::hardware::radio::1.1::IRadio

  ting@ting-pc:~/aosp/android-9/hardware/interfaces/radio/1.0
  $ ls -al
  total 289
  drwxr-xr-x 1 ting 197121     0 十二月 25 17:53 ./
  drwxr-xr-x 1 ting 197121     0 十二月 25 17:53 ../
  -rw-r--r-- 1 ting 197121   413 十二月 25 17:53 Android.bp
  -rw-r--r-- 1 ting 197121 59221 十二月 25 17:53 IRadio.hal
  -rw-r--r-- 1 ting 197121 17849 十二月 25 17:53 IRadioIndication.hal
  -rw-r--r-- 1 ting 197121 92745 十二月 25 17:53 IRadioResponse.hal
  -rw-r--r-- 1 ting 197121  2961 十二月 25 17:53 ISap.hal
  -rw-r--r-- 1 ting 197121  6189 十二月 25 17:53 ISapCallback.hal
  -rw-r--r-- 1 ting 197121 97551 十二月 25 17:53 types.hal
  drwxr-xr-x 1 ting 197121     0 十二月 25 17:53 vts/

  ting@ting-pc:~/aosp/android-9/hardware/interfaces/radio/1.1
  $ ls -al
  total 41
  drwxr-xr-x 1 ting 197121    0 十二月 25 17:53 ./
  drwxr-xr-x 1 ting 197121    0 十二月 25 17:53 ../
  -rw-r--r-- 1 ting 197121  423 十二月 25 17:53 Android.bp
  -rw-r--r-- 1 ting 197121 4899 十二月 25 17:53 IRadio.hal
  -rw-r--r-- 1 ting 197121 2013 十二月 25 17:53 IRadioIndication.hal
  -rw-r--r-- 1 ting 197121 3355 十二月 25 17:53 IRadioResponse.hal
  -rw-r--r-- 1 ting 197121  778 十二月 25 17:53 ISap.hal
  -rw-r--r-- 1 ting 197121 8644 十二月 25 17:53 types.hal
  drwxr-xr-x 1 ting 197121    0 十二月 25 17:53 vts/
  ```

- [IRadioConfig](http://androidxref.com/9.0.0_r3/xref/hardware/interfaces/radio/config/)

  [IRadioConfig](http://androidxref.com/9.0.0_r3/xref/hardware/interfaces/radio/config/) is the optional implementation by modem vendor.

  ```
  ting@ting-pc:~/aosp/android-9/hardware/interfaces/radio/config
  $ ls -al
  total 16
  drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 ./
  drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 ../
  drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 1.0/
  drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 1.1/
  drwxr-xr-x 1 ting 197121 0 十二月 25 17:53 1.2/

  ting@ting-pc:~/aosp/android-9/hardware/interfaces/radio/config/1.0
  $ ls -al
  total 25
  drwxr-xr-x 1 ting 197121    0 十二月 25 17:53 ./
  drwxr-xr-x 1 ting 197121    0 十二月 25 17:53 ../
  -rw-r--r-- 1 ting 197121  427 十二月 25 17:53 Android.bp
  drwxr-xr-x 1 ting 197121    0 十二月 25 17:53 default/
  -rw-r--r-- 1 ting 197121 3875 十二月 25 17:53 IRadioConfig.hal
  -rw-r--r-- 1 ting 197121 1419 十二月 25 17:53 IRadioConfigIndication.hal
  -rw-r--r-- 1 ting 197121 1847 十二月 25 17:53 IRadioConfigResponse.hal
  -rw-r--r-- 1 ting 197121 1905 十二月 25 17:53 types.hal
  drwxr-xr-x 1 ting 197121    0 十二月 25 17:53 vts/
  ```

<br>

## RIL Daemon

◤ For Android versions prior to Android 8.0, RIL daemon use Socket to communicate with Anroid phone framework.

◤ On Android 8.0 or later versions, the communication interface between Android phone framework and ril-daemon service has been changed from socket to HIDL.

```shell
ting@ting-pc:~/aosp/android-9/hardware/ril$ ls -al
total 36
drwxrwxr-x  7 ting ting 4096 Nov 12 21:23 ./
drwxrwxr-x 12 ting ting 4096 Dec 15 17:14 ../
-rw-rw-r--  1 ting ting 2850 Nov 12 21:23 CleanSpec.mk
-rw-rw-r--  1 ting ting  152 Nov 12 21:23 OWNERS
drwxrwxr-x  4 ting ting 4096 Nov 12 21:23 include/
drwxrwxr-x  2 ting ting 4096 Nov 12 21:23 libril/             // Radio HIDL Impl.
drwxrwxr-x  3 ting ting 4096 Nov 12 21:23 librilutils/
drwxrwxr-x  2 ting ting 4096 Nov 12 21:23 reference-ril/      // Vendor RIL
drwxrwxr-x  2 ting ting 4096 Nov 12 21:23 rild/               // RIL Daemon

```

- [rild](http://androidxref.com/9.0.0_r3/xref/hardware/ril/rild/)

  The RIL Daemon talks to the telephony services and dispatches "solicited commands" to the Vendor RIL.

  ```shell
  ting@ting-pc:~/aosp/android-9/hardware/ril/rild$ ls -al
  total 40
  drwxrwxr-x 2 ting ting  4096 Nov 12 21:23 ./
  drwxrwxr-x 7 ting ting  4096 Nov 12 21:23 ../
  -rw-rw-r-- 1 ting ting   788 Nov 12 21:23 Android.mk
  -rw-rw-r-- 1 ting ting     0 Nov 12 21:23 MODULE_LICENSE_APACHE2
  -rw-rw-r-- 1 ting ting 10695 Nov 12 21:23 NOTICE
  -rw-rw-r-- 1 ting ting  7101 Nov 12 21:23 rild.c
  -rw-rw-r-- 1 ting ting   191 Nov 12 21:23 rild.legacy.rc
  -rw-rw-r-- 1 ting ting   198 Nov 12 21:23 rild.rc
  ```

  ``` shell
  ting@ting-pc:~/aosp/android-9/hardware/ril/rild$ cat rild.rc
  service vendor.ril-daemon /vendor/bin/hw/rild
      class main
      user radio
      disabled
      group radio cache inet misc audio log readproc wakelock
      capabilities BLOCK_SUSPEND NET_ADMIN NET_RAW
  ```

  ```c
  ting@ting-pc:~/aosp/android-9/hardware/ril/rild
  $ cat rild.c

  int main(int argc, char **argv) {
      // vendor ril lib path either passed in as -l parameter, or read from rild.libpath property
      const char *rilLibPath = NULL;
      // ril arguments either passed in as -- parameter, or read from rild.libargs property
      char **rilArgv;

  	...

      // ril/socket id received as -c parameter, otherwise set to 0
      const char *clientId = NULL;

      RLOGD("**RIL Daemon Started**");
      RLOGD("**RILd param count=%d**", argc);

  	...

      dlHandle = dlopen(rilLibPath, RTLD_NOW);	// indicate to Vendor RIL, libreference-ril.so
  ```



- [libreference-ril.so](http://androidxref.com/9.0.0_r3/xref/hardware/ril/reference-ril/)

   The Vendor RIL is specific to a particular radio implementation, and dispatches "unsolicited commands" up to the RIL Daemon.

   Modem module vendor needs to implement this shared library that coding modem specific AT commands.

   ```shell
   ting@ting-pc:~/aosp/android-9/hardware/ril/reference-ril
   $ ls -al
   total 198
   drwxr-xr-x 1 ting 197121      0 十二月 25 17:52 ./
   drwxr-xr-x 1 ting 197121      0 十二月 25 17:52 ../
   -rw-r--r-- 1 ting 197121   1157 十二月 25 17:52 Android.mk
   -rw-r--r-- 1 ting 197121   3850 十二月 25 17:52 at_tok.c
   -rw-r--r-- 1 ting 197121   1004 十二月 25 17:52 at_tok.h
   -rw-r--r-- 1 ting 197121  24710 十二月 25 17:52 atchannel.c
   -rw-r--r-- 1 ting 197121   4381 十二月 25 17:52 atchannel.h
   -rw-r--r-- 1 ting 197121   1309 十二月 25 17:52 misc.c
   -rw-r--r-- 1 ting 197121    904 十二月 25 17:52 misc.h
   -rw-r--r-- 1 ting 197121      0 十二月 25 17:52 MODULE_LICENSE_APACHE2
   -rw-r--r-- 1 ting 197121  10885 十二月 25 17:52 NOTICE
   -rw-r--r-- 1 ting 197121    177 十二月 25 17:52 OWNERS
   -rw-r--r-- 1 ting 197121 122720 十二月 25 17:52 reference-ril.c
   -rw-r--r-- 1 ting 197121     26 十二月 25 17:52 ril.h

   ```

   ```c
   ting@ting-pc:~/aosp/android-9/hardware/ril/reference-ril
   $ cat reference-ril.c

   static void requestSignalStrength(void *data __unused, size_t datalen __unused, RIL_Token t)
   {
       ATResponse *p_response = NULL;
       int err;
       char *line;
       int count = 0;
       // Accept a response that is at least v6, and up to v10
       int minNumOfElements=sizeof(RIL_SignalStrength_v6)/sizeof(int);
       int maxNumOfElements=sizeof(RIL_SignalStrength_v10)/sizeof(int);
       int response[maxNumOfElements];

       memset(response, 0, sizeof(response));

       err = at_send_command_singleline("AT+CSQ", "+CSQ:", &p_response);

       if (err < 0 || p_response->success == 0) {
           RIL_onRequestComplete(t, RIL_E_GENERIC_FAILURE, NULL, 0);
           goto error;
       }

       line = p_response->p_intermediates->line;

       err = at_tok_start(&line);
       if (err < 0) goto error;

       for (count = 0; count < maxNumOfElements; count++) {
           err = at_tok_nextint(&line, &(response[count]));
           if (err < 0 && count < minNumOfElements) goto error;
       }

       RIL_onRequestComplete(t, RIL_E_SUCCESS, response, sizeof(response));

       at_response_free(p_response);
       return;

   error:
       RLOGE("requestSignalStrength must never return an error when radio is on");
       RIL_onRequestComplete(t, RIL_E_GENERIC_FAILURE, NULL, 0);
       at_response_free(p_response);
   }
   ```




- [libril.so](http://androidxref.com/9.0.0_r3/xref/hardware/ril/libril/)

   The library of [rild](http://androidxref.com/9.0.0_r3/xref/hardware/ril/rild/) that implement Radio HIDL service, e.g. android::hardware::radio::V1_1::IRadio.

   ```shell
   ting@ting-pc:~/aosp/android-9/hardware/ril/libril
   $ ls -al
   total 560
   drwxr-xr-x 1 ting 197121      0 十二月 25 17:52 ./
   drwxr-xr-x 1 ting 197121      0 十二月 25 17:52 ../
   -rw-r--r-- 1 ting 197121   1138 十二月 25 17:52 Android.mk
   -rw-r--r-- 1 ting 197121      0 十二月 25 17:52 MODULE_LICENSE_APACHE2
   -rw-r--r-- 1 ting 197121  10885 十二月 25 17:52 NOTICE
   -rw-r--r-- 1 ting 197121  49496 十二月 25 17:52 ril.cpp
   -rw-r--r-- 1 ting 197121  11316 十二月 25 17:52 ril_commands.h
   -rw-r--r-- 1 ting 197121  10484 十二月 25 17:52 ril_event.cpp
   -rw-r--r-- 1 ting 197121   1517 十二月 25 17:52 ril_event.h
   -rw-r--r-- 1 ting 197121   3112 十二月 25 17:52 ril_internal.h
   -rw-r--r-- 1 ting 197121 343787 十二月 25 17:52 ril_service.cpp
   -rw-r--r-- 1 ting 197121  34538 十二月 25 17:52 ril_service.h
   -rw-r--r-- 1 ting 197121   4791 十二月 25 17:52 ril_unsol_commands.h
   -rw-r--r-- 1 ting 197121   8980 十二月 25 17:52 RilSapSocket.cpp
   -rw-r--r-- 1 ting 197121   6491 十二月 25 17:52 RilSapSocket.h
   -rw-r--r-- 1 ting 197121   1714 十二月 25 17:52 RilSocket.h
   -rw-r--r-- 1 ting 197121   3906 十二月 25 17:52 rilSocketQueue.h
   -rw-r--r-- 1 ting 197121  39253 十二月 25 17:52 sap_service.cpp
   -rw-r--r-- 1 ting 197121   1093 十二月 25 17:52 sap_service.h

   ```

   ```cpp
   // impl. of android::hardware::radio::V1_1::IRadio

   ting@ting-pc:~/aosp/android-9/hardware/ril/libril
   $ cat ril_service.cpp

   struct RadioImpl : public V1_1::IRadio {
       int32_t mSlotId;
       sp<IRadioResponse> mRadioResponse;
       sp<IRadioIndication> mRadioIndication;
       sp<V1_1::IRadioResponse> mRadioResponseV1_1;
       sp<V1_1::IRadioIndication> mRadioIndicationV1_1;

       Return<void> setResponseFunctions(
               const ::android::sp<IRadioResponse>& radioResponse,
               const ::android::sp<IRadioIndication>& radioIndication);

       Return<void> getIccCardStatus(int32_t serial);
       ...
   ```


### Stop/Start RIL daemon

Usually it needs to stop ril daemon before testing AT command with modem module

```shell

setprop ctl.start vendor.ril-daemon

setprop ctl.stop vendor.ril-daemon

```

### Runtime @ device

```shell
console:/vendor # find | grep -w "rild"
	./etc/init/rild.rc
	./bin/hw/rild

		console:/vendor # cat ./etc/init/rild.rc
		service vendor.ril-daemon /vendor/bin/hw/rild -l /vendor/lib64/libreference-ril.so
			class main
			user radio
			group radio cache inet misc audio sdcard_rw log readproc wakelock
			capabilities BLOCK_SUSPEND NET_ADMIN NET_RAW
		console:/vendor #


console:/vendor # find | grep -w "ril.so"
	./lib64/libreference-ril.so


console:/ # getprop | grep -i -E "ril|radio"

	[gsm.version.ril-impl]: [Quectel_Android_RIL_Driver_V3.1.8]
	[init.svc.vendor.ril-daemon]: [running]
	[persist.radio.multisim.config]: [ssss]
	[persist.rild.nitz_long_ons_0]: []
	[persist.rild.nitz_long_ons_1]: []
	[persist.rild.nitz_long_ons_2]: []
	[persist.rild.nitz_long_ons_3]: []
	[persist.rild.nitz_plmn]: []
	[persist.rild.nitz_short_ons_0]: []
	[persist.rild.nitz_short_ons_1]: []
	[persist.rild.nitz_short_ons_2]: []
	[persist.rild.nitz_short_ons_3]: []
	[persist.vendor.radio.apm_sim_not_pwdn]: [1]
	[persist.vendor.radio.atfwd.start]: [false]
	[persist.vendor.radio.custom_ecc]: [1]
	[persist.vendor.radio.rat_on]: [combine]
	[persist.vendor.radio.sib16_support]: [1]
	[ril.subscription.types]: [NV,RUIM]
	[rild.libargs]: [-d /dev/ttyUSB2]
	[rild.libpath]: [/vendor/lib64/libreference-ril.so]
	[ro.boottime.vendor.ril-daemon]: [17157893326]
	[vendor.rild.libpath]: [/vendor/lib64/libreference-ril.so]


console:/ # ps -A | grep -E -i "radio|ril"

    system         559     1   16180   4824 binder_thread_read  0 S android.hardware.radio.config@1.0-service
    radio         1312     1 2132928   6752 binder_thread_read  0 S rild
    radio         1321     1    5084   1868 __skb_wait_for_more_packets 0 S ssgqmigd
    radio         1526     1   22500   5368 futex_wait_queue_me 0 S ipacm
    radio         2003  1259 3712124  84148 SyS_epoll_wait      0 S com.qualcomm.qti.telephonyservice
    radio         3065  1259 3699680  77996 SyS_epoll_wait      0 S com.qualcomm.qcrilmsgtunnel
    radio         4375  1259 3695368  78876 SyS_epoll_wait      0 S com.qualcomm.qti.lpa
    radio         4390  1259 3697792  81464 SyS_epoll_wait      0 S com.qualcomm.qti.modemtestmode
    radio         4442  1259 3698524  80652 SyS_epoll_wait      0 S com.qualcomm.simcontacts
    radio         5480  1259 3714564  95160 futex_wait_queue_me 0 S com.android.phone
```

<br>

## Modem Device Node, Driver Node

- Take modem module Quectel EC25 for example, it's the USB serial device.

```shell
console:/ #  ls -l /dev/ttyUSB*

	crw-rw---- 1 radio radio 188,   0 1970-01-01 00:00 /dev/ttyUSB0
	crw-rw---- 1 radio radio 188,   1 1970-01-01 00:00 /dev/ttyUSB1
	crw-rw---- 1 radio radio 188,   2 2020-12-02 09:59 /dev/ttyUSB2
	crw-rw---- 1 radio radio 188,   3 1970-01-01 00:00 /dev/ttyUSB3

ls -al /sys/bus/usb/drivers

	drwxr-xr-x 32 root root 0 2020-12-07 08:06 .
	drwxr-xr-x  4 root root 0 1970-01-01 00:00 ..
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 asix
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 ax88179_178a
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 cdc_ether
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 cdc_ncm
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 cdc_subset
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 cdc_wdm
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 hub
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 lvs
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 net1080
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 option
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 qmi_wwan_q
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 snd-usb-audio
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 ums-alauda
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 ums-cypress
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 ums-datafab
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 ums-freecom
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 ums-isd200
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 ums-jumpshot
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 ums-karma
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 ums-onetouch
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 ums-sddr09
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 ums-sddr55
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 ums-usbat
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 usb
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 usb-storage
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 usb_ehset_test
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 usbfs
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 usbhid
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 usbserial
	drwxr-xr-x  2 root root 0 2020-12-07 08:06 zaurus
```

<br>

## Use case of AT command

```shell
console:/ #

	cat /dev/ttyUSB2 &

	echo "ati;+csub" > /dev/ttyUSB2

	echo -e 'AT+CPIN?\r\n' > /dev/ttyUSB2

	echo -e 'at+qsimdet=1,0\r\n' > /dev/ttyUSB2
	echo -e 'AT+QSIMDET=?\r\n' > /dev/ttyUSB2
	echo -e 'AT+QSIMDET?\r\n' > /dev/ttyUSB2

	echo -e 'AT+QSIMSTAT=1\r\n' > /dev/ttyUSB2
	echo -e 'AT+QSIMSTAT=?\r\n' > /dev/ttyUSB2
	echo -e 'AT+QSIMSTAT?\r\n' > /dev/ttyUSB2
```



<br>




## Glossary

[Telephony 术语 缩写 Acronyms](https://blog.csdn.net/djialin0418/article/details/51127162)


<br>

## Appendix

- 中華電信 mcc/mnc 466/92

<br>
<br>
<br>



Ref. module of [Quectel EC25-E](https://github.com/tingkts/Android-Telephony/blob/main/res/Quectel_WCDMA%26LTE_Linux_USB_Driver_User_Guide_V1.8.pdf), based on [AOSP Android 9](http://androidxref.com/9.0.0_r3/) / [Android 10](https://cs.android.com/android/platform/superproject/+/android-10.0.0_r30:)

