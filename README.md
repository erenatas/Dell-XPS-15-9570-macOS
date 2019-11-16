# Introduction

This is a hackintosh EFI built with Clover for the `Dell XPS 9570, i7-8750H, 4K` laptop, designed in order to run Mac OS Mojave 10.14.X and above versions. This configuration integrates the commonly used native sound card driver, graphics driver, camera driver, and can support hot-swappable `HDMI 2.0` and `DisplayPort via Type C` dual-screen `4K@60Hz` output, supporting cold boot. The wireless network card is DW1830 to support wireless network transmission functions such as Wifi and AirDrop, and supports touch gestures. The sleep wakeup should be working normally.

# Hardware Configuration

## Working

* Machine :Dell XPS 9570
* CPU: Intel i7-8750H
* GPU: UHD630 (Integrated GPU)
* RAM: 16GB RAM
* Display: 4K Sharp Display
* SSD: PC401 NVMe SK hynix 512GB SSD
* Audio: Realtek ALC298
* Touchscreen
* WLAN + Bluetooth : DW1830 (replaced)

## not driven

* Killer 1535 (Not supported)
* Goodix fingerpint reader (Does not work in both MacOS and Linux)
* Nvidia Geforce 1050Ti (Nvidia Optimus cards are not supported in MacOS)

# Software Environment

* BIOS: `1.5.0 ~ 1.11.2`
* System: `Mac Mojave + Windows 10 1809` Dual system coexistence
* macOS: `Mac Mojave 10.14.X` & `Mac Catalina 10.15 beta 3~5`

# Instructions
  [bavariancake] (https://github.com/bavariancake/XPS9570-macOS) at
  The basic steps for installing Mojave are described in the `Installation and system updates` section.
  **Assuming you have completed the USB installation media preparation work**, and want to use the configuration released by this warehouse to complete the Black Apple tour, then you need to understand the role of the following three configuration files, clearly when to switch to The configuration file for your model is fully loaded.

## Introduction to the configuration file

### `config_install.plist`

This is the configuration file used during the Mojave installation phase, in order to inject a wrong `ig-platform-id = 0x12345678`, disable the kernel display, so that the system can complete the installation on the desktop.

### `config.plist`

  The difference with `config_install.plist` is that the correct kernel platform id `ig-platform-id = 0x3e9b0000` is injected. This configuration works with the UHD630 core repair patch provided by @FireWolf in WEG to drive the 4K/1080P internal display.
  
  This file is designed to drive the XPS 9570 4K version of the model, where:
  
  `dpcd-max-link-rate` = `<14000000>`

### `confug_1080P.plist`

  This file is specifically designed to drive the XPS 9570 1080P version of the model, which is different from `config.plist` only in:
  
   `dpcd-max-link-rate` = `<0A000000>`

## Steps for usage

* 1.**Replace the installation volume EFI**: Delete the Clover folder that comes with the USB media EFI partition and replace it with Clover of this warehouse.
* 2.**Use `config_install.plist`**: When entering the CLOVER startup interface, switch the configuration used by Clover to `config_install.plist`; or rename `config_install.plist` to `config.plist` first. , replace the original `config.plist` (note backup!) and then enter the interface

* 3.**Complete the initial installation**: Select the installation volume icon, enter the installation interface to complete the installation, if the process fails to install, etc., you can enter the installation interface multiple times (3~5 times)

* 4.** Rebuild Cache**: After entering the desktop, run `sudo kextcache -i /` from the terminal to rebuild the cache

* 5.** Use `config.plist(for 4k)`**: If the machine is a 4k version, use `config.plist` directly. Otherwise, see step 6
  
* 6.** Use `config_1080p.plist`**: If the model is 1080p, rename `config_1080P.plist` to `config.plist` to replace the original config
* 7.**Add three codes to generate**: This warehouse does not fill in the default three yards. If you want to activate Facetime and iMessage, please refer to [bavariancake] (https://github.com/bavariancake/XPS9570-macOS) The activation scheme provided, or the extraction of three yards in the white apple machine is used in this machine. If you do not have activation requirements, you can use `Clover Configruator` to edit `config.plist`. In the `SIMBIOS` item, select the model `MacBookPro15,1` and randomly generate a set of three codes.

* 8.** Verify that UHD630 is driven**: If the display is already driven, the 4K display will automatically turn on HDpi. About the graphics card in the native report is displayed as `Intel UHD Graphics 630 2048 MB`

* 9.** Add some drivers to `L/E` or `S/L/E` (optional)**: Based on this repository configuration, if you want the driver injection method to be closer to the white apple, please Following the driver installation method of [Complete Solution--Remove-v Mode](), it is recommended to keep only `VirtualSmc` in `CLOVE/kexts/Other`, and install the remaining drivers to `L/E` or `S/L /E`, then remove the drivers that have been installed in the `CLOVER/kexts/Ohter` path. Finally, rebuild the cache to restart the computer.
   
**Every time you update, just switch to config_install.plist to complete the installation. The first thing after entering the system is to rebuild the cache!**

# Known issues

* ~~4K version has high fan noise under high load, high temperature, battery life is only 2.5 hours~~
* ~~ When using the battery, Bluetooth is not available after sleep wake up~~
* ~~ Remove the -v mode and restart it randomly when entering the system (see [Complete Solution - Remove -v Mode] (#compromise))~~
* ~~kernel_task The process occupancy has been no less than 20%, or more than 100% (see [Perfect Solution - Solve the Kernel_task Super High Occupancy] (#compromise))~~
* There is a chance that sleep has a kernel crash with wake sleep transition
* ~~ Can't wake up with mouse and keyboard, can only wake up with power and open cover~~
* Lightning three devices do not support hot swap, only support semi-hot swap after cold start

# Not tested yet

* `Mac Catalina 10.15.X`
* Models other than 4k and 8750H

# Improve the plan

## Displaying Solid State Drives

  Boot into the BIOS, change the hard disk mode to AHCI, and switch to the original mode after the system is installed and the driver is completed. Of course, it is normal to keep AHCI work.

 <span id = "compromise"></span>

## Remove -v mode

### Problem Description

 Injecting the touchpad and screen touch driver from `CLOVER/kexts/Other`, removing the `-v` mode will cause the machine to restart randomly.

### solution

 The default configuration of the repository retains the `-v` mode for the following two purposes:

* It is convenient for everyone to debug and record errors when installing and entering the system for the first time.
* Keep the auto-selection of the driver installation location

If you want to remove the `-v` startup parameter and switch the clover mode to normal mode, you need to perform the following steps:

* Use `Kext Utility` to install `VoodooI2C.kext`, `VoodooI2CHID.kext`, `VoodooPS2Controller.kext` in `/CLOVER/kexts/Other/` to `/System/Library/Extensions/` or `/Libary /Extensions` directory
* Use `Kext Utility` or run `sudo kextcache -i /` on the terminal to rebuild the cache
* Edit `config.plist` to remove the `-v` parameter from Boot/Boot Arguments;
* Remove the three installed drivers `/CLOVER/kexts/Other/`

Note that when entering the system for the first time, you can first reserve `-v` to ensure that the machine works properly before performing this optimization.

## Solve the occasional silence of the headphones

### Problem Description 

After the original sound card is successfully driven, insert the earphone to the maximum volume, but only hear the low sound, turn on the sound of the system setting, switch between the two options of the input source, and the volume of the earphone returns to normal. After the system setting is turned off, After about 2 minutes, the sound failed again.

### Solution

 Enable a daemon for ALC298. See [ALC298PlugFix] (https://github.com/jardenliu/ALC298PlugFix)

## Solve the super high occupancy of kernel_task

### Problem Description

 Incorrect Layout Id may cause AppleALC to work abnormally, causing the kernel to be busy and the kernel_taskCPU usage rate is over 100%. For a detailed discussion, see [ISSUE #2] (https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS- Mojave/issues/2), [ISSUE #8] (https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/issues/8), or AppleALC may conflict with certain drivers of).
 
 ### Solution

 ALC298 sound card of different machines may need to use different Layout Id. The default Layout Id of this configuration is 30.
 According to tests some machines need to be injected 72: in

 In the `Devices/Properties/PciRoot(0)/Pci(0x1f,3)` column, change the `layout-id` to 72.

* `layout-id 72 NUMBER`

 ## Drive Dell DA300 NIC

 ### Problem Description

  Mojave does not have a Realtek 8153 NIC driver. If you want to access the Ethernet through the DA300, the port will not receive the corresponding network packet.
### Solution

Additional drivers are required, as discussed in [ISSUE #23] (https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/issues/23).

## Generating frequency conversion information for the CPU
  [one-key-cpufriend](https://github.com/stevezhengshiqi/one-key-cpufriend) One-click on the desktop to generate `CPUFriendDataProvider.kext`, `CPUFriendDataProvider.kext`, then move the two drivers to /CLOVER/kexts/Other/` or install to `L/E`,`S/L/E`

## Thunder 3 device hot swap

### Problem Description

Thunderbolt equipment does not work properly after hot plugging. For discussion, see [ISSUE #4] (https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/issues/4).

### Solution

After using the SSDT-based half-lightning 3 patch in the warehouse, perform the following steps:

* Restart into the BIOS and set `BIOS assist enumeration` to `enabled`
* Set the BIOS lightning interface related settings to open, and set the access rights of the lightning interface to the lowest.
* Download the latest Thunderbolt firmware update from Dell website and install it
* Connect the lightning device under `Windows` system, the connection permission is set to `Always allow`
* Connect the Thunder 3 device before starting the system, then enter the `Mac Mojave` test hot plug to check if it is working properly.

## Solving DW1830 sleep is not available after wake-up

### Problem Description

When using the battery, let the system sleep to sleep, about 1 minute after waking up again, Bluetooth status is not available

### Solution

Remove the DW1830 from the machine and shield the NIC 'Pin 56` and `Pin 54` pins with a cover (such as tape or double-sided tape), see [jaymonkey] (https://www.tonymacx86.com/threads/bluetooth- Random-not-available-after-sleep.266255/post-1862126)


  ![Shielding Pins](https://www.tonymacx86.com/attachments/bcmwifi-jpg.368795)

<!-- # About this machine
![About this machine](https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/blob/master/screenshots/about_my_machine.png)
[Nvme SSD] (https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/raw/master/screenshots/NVME.png)
![集显](https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/raw/master/screenshots/Graphic.png)
![Dual Display Output](https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/raw/master/screenshots/dual_displays.png)
![Output Details](https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/raw/master/screenshots/dual_displays_details.png)
[Touchpad] (https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/raw/master/screenshots/trackpad.png)
![USB3.1](https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/raw/master/screenshots/USB3.1.png)
![Battery](https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/raw/master/screenshots/Battery.png)
[DW1830 Bluetooth] (https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/raw/master/screenshots/DW1830%20Bluetooth.png)
![DW1830 Wifi](https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/raw/master/screenshots/DW1830%20Wifi.png)
![Native Management] (https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/raw/master/screenshots/Extention.png)
![inverter](https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/raw/master/screenshots/Intel%20Turbo%20Boost.png)
 -->
# Update history

## August 8, 2019
* Support `Mac Mojave 10.14.X` and `Mac Catalina Beta 5`
* Use a compromise scheme to disable the touch screen and fix the bug that the `kenerl_task` CPU usage exceeds '10%` (the touch screen has been disabled, don't mind if you don't use it)
* Inject the fake 'PCI' list in `Properties` to complete the system report
* Delete `AppleBacklightInjector.kext` because WEG has been automatically injected
* Replace `VirtualSMC` with the original `FakeSMC` to avoid `Service exited with abnormal code: 1` when `Catalina` is installed.
* Complete non-required hot fixes to complete the ACPI list required by Mac

##July 21, 2019

**Note: This version is based on the `BIOS 1.11.2` update. It is recommended to upgrade the BIOS to this version before upgrading.

* Enable [GPIO Pinning] (https://voodooi2c.github.io/#GPIO%20Pinning/GPIO%20Pinning) interrupt mode for the touchpad with `SSDT-I2C.xml` hot patch
* Fixed a bug where the touchpad "double click to drag" failed
* Added decompiled DSDT file based on `BIOS 1.11.2` extraction
* Use `SSDT-PTWAKE.aml` to call the deep method to shield the independent display
* Inject the DMA controller information of the system default search with `SSDT-DMAC.aml`
* Repair USB potential `Instant wake` with `SSDT-UPRW.am`
* Solve the bug that is not available after DW1830 wake-up by blocking `Pin 54 Pin 56`, thanks to [jaymonkey](https://www.tonymacx86.com/threads/bluetooth-randomly-not-available-after-sleep.266255/ Post-1862126) Provided solution
* Inject layout-id 30 using Clover (**Note: SSDT injection method is deprecated**)
* Updated to `Luli 1.3.7`
* Join `HibernationFixup.kext` to avoid wake up after hibernation

## July 6, 2019

* Added lightning interface 3 semi-hot swap SSDT patch for extensive testing from [ISSUE #4 @andresandiah] (https://github.com/LuletterSoul/Dell-XPS-15-9570-macOS-Mojave/ Issues/4#issuecomment-506429102), thank him!
* Updated to WEG 1.3.X to fix LPSCON driver status get error

## July 5, 2019

* Updated to 10.14.5 (18F203)
* Updated to BIOS 1.11.2 (using this version of XPS has become quieter, it is recommended to update the test)
* Updated to Clover 4979
* Replace FakeSmc with VirtualSmc, the battery display is more accurate
* Fixed an issue where the two-stage Apple Logo size was inconsistent when entering the system in non-啰嗦 mode.
* Fixed the problem of `HDMI 2.0 4k@60Hz 1080P hDpi` mode failing after wake-up
* [@FireWolf Update] (https://github.com/acidanthera/WhateverGreen/pull/24): This PR has added a driver for the LPSCON adapter, and the DP signal is enabled by setting the adapter's operating mode to Protocol Converter Mode. Converted to HDMI 2.0 signal, so UHD630 supports 4k@60Hz output (again, thank you Dawei FireWolf!)
* Test situation: Use the official WEG 1.3.0, set the monitor's working mode to 4k@60hz<-->1080P@60hz hDPi. After entering the system for the first time, HDMI 2.0 supports hot plugging, but sleep wakes up the monitor without signal. Adjust the working mode to 4k@60hz <--> 1080@30hz hDpi with RDM, the display is lit
* Experimental results: When entering the system for the first time, the LPSCON driver provided by the official WEG maximizes the HDMI2.0 port transfer rate (by setting the LPSCON mode to PCON), but after the machine wakes up, the LPSCON register information is Reset to initial value, WEG does not continue to update LPSCON working mode after waking up
* Solution: Modify the kernel extension source, remove the LPSCON driver

### Modify the memory to 2048MB

* Modifying the memory to 2048M via WhateverGreen may solve the problem of some screens.

#声明

The XPS 9570 4K with this configuration seems to work very well in the latest version. There are still many kernel issues and driver issues after continuous testing. My vision is to make the XPS 9570 4K "perfect" on Mac OS.

But time, energy, and ability limit the timeliness of my maintenance of this warehouse. I have no time to answer many questions from ISSUE. The `Mac Catalina 10.15.X` version is coming soon. How is the new version running on the XPS 9570? I don't know, but I hope that friends who have the will and ability can explore and overcome known difficult problems with me, and promptly PR or feedback some cooperative contributions (such as perfecting Chinese and English ReadMe, providing different models). Feedback), or propose more solutions that can be improved, rather than being a passer-by who is fearless and behind closed doors.

I hope this warehouse can continue to provide meaningful updates for everyone, I hope XPS 9570 can achieve the perfect Hackintosh soon!

Finally, I am very supportive of promoting this project, but if you need to post this EFI or reprint it to others, please be sure to indicate my warehouse source, as well as [bavariancake] (https://github.com/bavariancake/XPS9570-macOS) , [Xigtun] (https://github.com/Xigtun/xps-9570-mojave) warehouse address. If you are sure of my work, I am also happy to accept a capital star from a gentleman~~

**Do not use for business! Respect open source and labor! Your cooperation and support is the driving force for our continuous maintenance and sharing. **

# References

* [LuletterSoul] This repo has been forked from LuletterSoul/Dell-XPS-15-9570-macOS-Mojave
* [Apple](https://www.apple.com) for macOS
* [Rehabman] (https://github.com/RehabMan): Provide a large number of black apple drivers, the big black apple forum abroad, pay tribute to Daxie!
* [Lilu](https://github.com/acidanthera/Lilu): Pay tribute to the great reverse engineer and developer of the kernel extension project!
* [WhateverGreen] (https://github.com/acidanthera/WhateverGreen): Thanks to all the great developers who participated in this open source kernel extension project!
* [FireWolf](https://github.com/0xFireWolf/): Provides core open source contributions such as DPCD maximum link rate fix, Intel HDMI infinite loop connection repair, LSPCON driver support, etc., enabling some new machines based on UHD630 Type, especially XPS 9570 provides a very strong technical difficulty, thank you very much!
* [bavariancake] (https://github.com/bavariancake/XPS9570-macOS): Provides a detailed plan for XPS 9570 Hackintosh. His warehouse records every detail of the XPS 9570 driver, which is XPS 9570 Black. Apple's open source pioneer on Github contributed most of the configuration templates for this repository, thanks to his hard work and labor!
* [Xigtun](https://github.com/Xigtun/xps-9570-mojave): Like @bavariancake, a selfless contributor to the early open source XPS 9570 Black Apple configuration, this warehouse was in the early days.
*Based on the deep integration of [bavariancake] (https://github.com/bavariancake/XPS9570-macOS) configuration, improved to get the perfect configuration today, thank you for this selfless colleagues!
* @807133286 : The first [Touchpad Driver Solution] (https://github.com/Xigtun/xps-9570-mojave/issues/23) was introduced in the Xigtun repository, giving the XPS 9570 nearly white apples. The touchpad experience is the inspiration for improving the XPS 9570 touch version!
* [Vision Forum] (http://bbs.pcbeta.com/forum-559-1.html): Thank you for the general tutorial provided by the great gods, so that I can easily get started as a white person!
* [黑果小兵] (https://blog.daliansky.net/): I think that domestic needs more such unselfish, high-level black apple evangelists, thank him!
* Thanks to all the friends who have provided meaningful ISSUE and test feedback on the warehouse. The better the black apple configuration of XPS 9570 is, the more feedback and support you have made this project more meaningful!
