---
layout: post
title:  "Towards Cyanogen"
date:   2015-11-04 20:15:00 +0530
categories: porting
---
I have some experience in flashing [Nexus One]() with [CyanogenMod](http://www.cyanogenmod.org) years back. However the previous experience was only about using a pre-built Cyanogen ROM for the device without any customization.

> CyanogenMod is a free, community built distribution of Android which greatly extends the capabilities of your device.

In orderi to make my Coolpad Note 3 usable to a certain extent, I think porting Cyanogen for it is the only option. This is also in line with my long pending goal of learning Android internals.

CyanogenMod is a matured project with fairly good [documentation](https://wiki.cyanogenmod.org/w/Development). However porting an operating system, especially the kernel for a new device is usually a non-trivial exercise with multiple unexpected complications. However since both stock Android and CyanogenMod is based on Linux kernel itself, it might be possible to re-use the kernel shipped by the phone vendor while building only the user space initially. Most of the Android devices runs over ARM architecture, have similar hardware and based on similar chipsets due to which porting Cyanogen for a new device *may* be easier than expected. However as the Cyanogen documentation says:

> Porting CyanogenMod to a new device can be ridiculously easy or ridiculously difficult, depending on the device itself, whether it currently runs a recent version of Android or not, and of course your skills as a developer matter too.

In general, based on the [porting guide](https://wiki.cyanogenmod.org/w/Doc:_porting_intro) and general survey, I could identify the follow broad steps towards porting CyanogenMod for my Coolpad Note 3:

* [Development Environment Setup](https://wiki.cyanogenmod.org/w/Build_for_maguro)
* Target Device Hardware Information Gathering
* Building and Testing Recovery
* Obtain Proprietory/Vendor Blobs
* Customize Build
	* May require customization of drivers
* Run Build System


