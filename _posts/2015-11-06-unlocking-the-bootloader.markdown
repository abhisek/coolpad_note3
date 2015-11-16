---
layout: post
title:  "Unlocking the Bootloader"
date:   2015-11-06 00:14:00 +0530
categories: porting
---
### Why?

The purpose of unlocking the bootloader is to allow the bootloader software to flash (update) specific partitions in the eMMC. In other words, we would li
ke to perform raw eMMC I/O with the bootloader providing us the interface to the device storage. While there are chipset specific mode that allows special
 USB/UART based flashing while the device is in specific mode, similar capability can be achieved in multiple phones if it has a bootloader with supported
 capability. For example, Nexus based devices support the flashboot mode with unlockable bootloader. Once unlocked, it is possible to perform flashing usi
ng the fastboot protocol while the device is in fastboot mode.

### The Conventional Approach

Almost all Android based device comes with a locked bootloader. The locked bootloader ensures that the OS cannot be flashed using software means, at least
 without first unlocking the bootloader. Over the years, the Android boot process has become more complex with cryptographic means to establish trust for 
bootloader and maintain a chain of trust for subsequent components involved in the boot process. Android developer documentation explains [verified boot process](https://source.android.com/devices/tech/security/verifiedboot/verified-boot.html) in greater detail.

However, Android has always been developer friendly i.e. most Android based smart phones from different manufacturer allow an user to flash the different 
part of the system with their own modified software, ofcourse at their own risk. Technically this means there are well defined specification and tools for
 easily unlocking the bootloader.

> Note: Unlocking the bootloader voids manufacturer warranty in almost all phones.

The bootloader unlocking process is described in [official google documentation](https://source.android.com/devices/tech/security/verifiedboot/verified-bo
ot.html), it can be roughly summarized to:

* Reboot phone in fastboot mode
   * adb reboot bootloader
* Ensure fastboot is working
   * fastboot devices
   * fastboot -i 0x1ebf devices
      * Get USB Vendor Id from lsusb if not automatically detected.
* Unlock the bootloader
   * fastboot oem unlock
   * fastboot flashing unlock

> Note: Some phones, such as Coolpad Note 3 requires "OEM Unlock" to be enabled from Developer Settings before rebooting to fastboot mode.

However, things may not go as expected, especially in my case with the Coolpad Note 3:

{% highlight bash %}
$ adb reboot bootloader
$ fastboot -i 0x1ebf devices
ZTBAWCGU7949EQWG     fastboot
$ fastboot -i 0x1ebf oem unlock
...
FAILED (remote: unknown command)
finished. total time: 0.002s
$ fastboot -i 0x1ebf flash recovery /tmp/cm_recovery.img 
target reported max download size of 134217728 bytes
sending 'recovery' (16384 KB)...
OKAY [  1.683s]
writing 'recovery'...
FAILED (remote: unknown command)
finished. total time: 1.686s
{% endhighlight %}

For Mediatek based devices, the fasboot mode is handled by the [Little Kernel](https://sturmflut.github.io/mediatek/2015/07/05/mediatek-details-little-kernel/). However, it turns out that the [LK does not support OEM unlock](https://sturmflut.github.io/ubuntu/touch/2015/07/04/hacking-ubuntu-touch-part-8-fastboot/).

### Flashing The Recovery

So far it appears that Mediatek LK based bootloaders do not support the OEM unlock command, hence it is not possile to flash the recovery using fastboot through an unlocked bootloader. The next option was to flash the recovery using the [SP Flash Tool](http://www.theandroidhow.com/2014/06/how-to-flash-stock-custom-recovery-sp-flash-tool.html). I built a basic [CM recovery](http://xda-university.com/as-a-developer/porting-clockworkmod-recovery-to-a-new-device) based on the boot.img that I could extract from the device. The process for building recovery is pretty straight forward once you have the Cyanogen source code and managed to extract the boot image from your target device:

{% highlight bash %}
$ cd /android/system
$ . build/envsetup.sh
$ build/tools/device/mkvendor.sh coolpad rel /tmp/boot.img
{% endhighlight %}

The *vendor/coolpad/rel/BoardConfig.mk* file requires customization, particularly partition sizes. The required information was obtained from */proc/partinfo* in a running device.

Building recovery:
{% highlight bash %}
$ make -j4 otatools
$ lunch cm_rel-eng
$ make -j4 recoveryimage
{% endhighlight %}

I tried to flash the generated recovery.img into the device with SP Flash Tool, the [process is pretty straight](http://www.theandroidhow.com/2014/06/how-to-flash-stock-custom-recovery-sp-flash-tool.html) forward if you have a working environment for SP Flash Tool. However I was greeted with error: **PMT changed for the ROM; it must be downloaded**.

### References:

* https://source.android.com/devices/tech/security/verifiedboot/
* https://source.android.com/devices/tech/security/verifiedboot/verified-boot.html
* http://www.androidenea.com/2009/06/android-boot-process-from-power-on.html
* http://javigon.com/2012/08/27/the-bootloader-understanding-modifying-building-and-installing/
* https://sturmflut.github.io/ubuntu/touch/2015/07/04/hacking-ubuntu-touch-part-8-fastboot/
* https://sturmflut.github.io/mediatek/2015/07/05/mediatek-details-little-kernel/

