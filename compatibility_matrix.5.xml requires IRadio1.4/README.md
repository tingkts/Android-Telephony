Can't use IRadio 1.0 as usual in Android 11 (Linux kernel 5.10.52) for the matched compatibility_matrix.5.xml, should upgrade to IRado 1.4.

[build break of declare IRadio 1.0 in manifest.xml](https://github.com/tingkts/Android-Telephony/blob/main/compatibility_matrix.5.xml%20requires%20IRadio1.4/make_log_requires_IRadio1.4.txt)
``` xml
    <hal format="hidl">
        <name>android.hardware.radio</name>
        <transport>hwbinder</transport>
        <version>1.0</version>
        <interface>
            <name>IRadio</name>
            <instance>slot1</instance>
        </interface>
    </hal>
```

should declare IRadio 1.4 in manifest.xml, and use vendor implementation's IRadio 1.4 of libril.so
``` xml
    <hal format="hidl">
        <name>android.hardware.radio</name>
        <transport>hwbinder</transport>
        <fqname>@1.4::IRadio/slot1</fqname>
        <fqname>@1.2::ISap/slot1</fqname>
    </hal>
```
use vendor's prebuilt libril.so
``` Makefile
LOCAL_MODULE_SUFFIX := .so
LOCAL_MODULE:= libril
LOCAL_MODULE_CLASS := SHARED_LIBRARIES

LOCAL_SRC_FILES_arm := libril.so/armeabi/libril.so
LOCAL_SRC_FILES_arm64 := libril.so/arm64-v8a/libril.so

LOCAL_SHARED_LIBRARIES := \
    android.hardware.radio.deprecated@1.0 \
    android.hardware.radio@1.0 \
    android.hardware.radio@1.1 \
    android.hardware.radio@1.2 \
    android.hardware.radio@1.3 \
    android.hardware.radio@1.4 \
    libc++ \
    libc \
    libcutils \
    libdl \
    libhardware_legacy \
    libhidlbase \
    libhidltransport \
    libhwbinder \
    liblog \
    libm \
    librilutils \
    libutils

LOCAL_MODULE_TARGET_ARCHS := arm arm64
LOCAL_MULTILIB := both
include $(BUILD_PREBUILT)
```

</br></br>
Ref.&nbsp;:&nbsp;&nbsp;[Compatibility Matrixes  |  Android Open Source Project](https://source.android.google.cn/devices/architecture/vintf/comp-matrices)