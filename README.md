# Hackintosh
EFI for my Hackintosh Machine. 
- OpenCore 0.6.7-Release
- Catalina (10.15)
## OpenCore
- [Install Guide](https://dortania.github.io/OpenCore-Install-Guide)
- [Download OpenCorePkg](https://github.com/acidanthera/OpenCorePkg)

## Utils
- [ProperTree](https://github.com/corpnewt/ProperTree) - edit .plist files
- [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) - For generating our SMBIOS data
- [slowgeek](https://opencore.slowgeek.com) - OpenCore Sanity Checker

## Doc
### Finding your hardware
<details>
<summary>My Hardware</summary>

|||Generation|
|---|---|---|
|Motherboard|ASUSTeK P8H61-M LX3 R2.0||
|CPU|Intel(R) Core(TM) i3-3220 CPU @ 3.30GHz|Ivy Bridge|
|GPU|Intel HD Graphics 2500|
|GPU|Nvidia Colorful GTX650|kepler GK107|
|Audio|ASUSTeK 6Series Chipset HD||
|Audio|Nvidia HDMI||
|Wired|Realtek RTL8168F/8111F ||
||||

</details>

### [USB Creation](https://dortania.github.io/OpenCore-Install-Guide/installer-guide/linux-install.html)

<details><summary>Linux</summary>

- Download BaseSystem or RecoveryImage files

    ```shell
    cd /path/to/OpenCore-0.6.7-Release/Utilities/macrecovery
    # Catalina(10.15)
    python ./macrecovery.py -b Mac-00BE6ED71E35EB86 -m 00000000000000000 download
    ```

    you'll either get BaseSystem or RecoveryImage files.

- Making the installer
    ```shell
    lsblk
    sudo gdisk /dev/<your USB block>
    # send p to print your block's partitions (and verify it's the one needed)
    p
    # send o to clear the partition table and make a new GPT one (if not empty)
    o
    # Hex code or GUID: 0700 for Microsoft basic data partition type
    0700
    w
    y
    # quit gdisk
    # format your USB to FAT32 and named OPENCORE
    sudo mkfs.vfat -F 32 -n "OPENCORE" /dev/<your USB partition block>
    # mount your USB partition
    udisksctl mount -b /dev/<your USB partition block>
    # cd to your USB drive
    mkdir com.apple.recovery.boot
    cp /path/to/OpenCore-0.6.7-Release/Utilities/macrecovery/BaseSystem* /path/to/com.apple.recovery.boot
    ```
</details>

### Gathering files

<details>
<summary>config.plist</summary>

```shell
cp /path/to/OpenCore 0.6.7-Release/Docs/Sample.plist /path/to/your/EFI/OC/config.plist
```

</details>
<details>
<summary>EFI/OC/ACPI</summary>

- [SSDT-EC](https://dortania.github.io/Getting-Started-With-ACPI/ssdt-methods/ssdt-easy.html#so-what-can-t-ssdttime-do) - for For Broadwell desktops and older  
  Fixes the embedded controller
- [SSDT-IMEI](https://dortania.github.io/Getting-Started-With-ACPI/Universal/imei-methods/manual.html)
  Needed to add a missing IMEI device on Ivy Bridge CPU with 6 series motherboards
- [SSDT-PM](https://dortania.github.io/OpenCore-Post-Install/universal/pm.html#sandy-and-ivy-bridge-power-management) - run Pike's [ssdtPRGen.sh](https://github.com/Piker-Alpha/ssdtPRGen.sh) script to generate this file. This will be run in **post install**

</details>
<details>
<summary>EFI/OC/Drivers/</summary>

- [HfsPlus.efi](https://github.com/acidanthera/OcBinaryData/blob/master/Drivers/HfsPlus.efi) - Needed for seeing HFS volumes
- OpenRuntime.efi - used as an extension for OpenCore to help with patching boot.efi for NVRAM fixes and better memory management.
```shell
cp /path/to/OpenCore-0.6.7-Release/X64/EFI/OC/Drivers/{OpenHfsPlus.efi,OpenRuntime.efi} /path/to/EFI/OC/Drivers/
```

</details>
<details>
<summary>EFI/OC/Kexts/</summary>

- [Lilu](https://github.com/acidanthera/Lilu/releases)  
  A kext to patch many processes, required for AppleALC, WhateverGreen, VirtualSMC and many other kexts. Without Lilu, they will not work.  
  Note that Lilu and plugins requires OS X 10.8 or newer to function
- [VirtualSMC](https://github.com/acidanthera/VirtualSMC/releases)  
Emulates the SMC chip found on real macs, without this macOS will not boot
Alternative is FakeSMC which can have better or worse support, most commonly used on legacy hardware.
Requires OS X 10.6 or newer
    - SMCProcessor.kext  
Used for monitoring CPU temperature, doesn't work on AMD CPU based systems
    - SMCSuperIO.kext  
Used for monitoring fan speed, doesn't work on AMD CPU based systems
- [WhateverGreen](https://github.com/acidanthera/WhateverGreen/releases) - Used for graphics patching DRM, boardID, framebuffer fixes, etc, all GPUs benefit from this kext.
- ~~[AppleALC](https://github.com/acidanthera/AppleALC/releases)~~
- VoodooHDA VT1708S.kext
- [RealtekTRL8111](https://github.com/Mieze/RTL8111_driver_for_OS_X/releases)
- [USBInjectAll](https://bitbucket.org/RehabMan/os-x-usb-inject-all/downloads/)
- [VoodooPS2Controller.kext](https://github.com/acidanthera/VoodooPS2/releases) - PS2 keyboard

</details>

### plist
[lvy Bridge](https://dortania.github.io/OpenCore-Install-Guide/config.plist/ivy-bridge.html)

- DeviceProperties
  - Add  
    **PciRoot(0x0)/Pci(0x2,0x0) - iGPU**  
    07006201 - Used when the iGPU is only used for computing tasks and doesn't drive a display
    |key|type|value|
    |---|---|---|
    |AAPL,ig-platform-id|Data|07006201|

    **PciRoot(0x0)/Pci(0x16,0x0) - IMEI**  
    specifically needed to spoof your IMEI device into being supported. Note this property is still required with or without SSDT-IMEI.
    |Key	|Type|	Value|
    |---|---|---|
    |device-id|	Data|	3A1E0000|

    **[PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x1) - Audio-Nvidia](https://dortania.github.io/OpenCore-Post-Install/universal/audio.html)**

    ```shell
    10de:0e1b /PCI0@0/PEG0@1/PEGP@0,1 = PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x1)
    ```

	  |Key|Type|	Value|
    |---|---|---|
    |[external-audio](https://dortania.github.io/OpenCore-Post-Install/universal/audio.html#applealc-not-working-correctly-with-multiple-sound-cards)|	Data|	01|
    |[alctsel](https://dortania.github.io/OpenCore-Post-Install/universal/audio.html#applealc-not-working-from-windows-reboot)|Data|01000000|

- NVRAM
  - Add  
    7C436110-AB2A-4BBB-A880-FE41995C9F82
    |boot-args|Description|
    |---|---|
    |shikigva=40|Swaps boardID to iMac14,2 for better Nvidia support and whitelists patches|

## BIOS

Intel BIOS settings
Note: Most of these options may not be present in your firmware, we recommend matching up as closely as possible but don't be too concerned if many of these options are not available in your BIOS
### Disable
- Fast Boot
- ~~Secure Boot~~
- ~~Serial/COM Port~~
- ~~Parallel Port~~
- ~~VT-d (can be enabled if you set DisableIoMapper to YES)~~
- CSM
- ~~Thunderbolt(For initial install, as Thunderbolt can - cause issues if not setup correctly)~~
- ~~Intel SGX~~
- ~~Intel Platform Trust~~
- ~~CFG Lock (MSR 0xE2 write protection)(This must be off, if you can't find the option then enable AppleCpuPmCfgLock under Kernel -> Quirks. Your hack will not boot with CFG-Lock enabled)~~
### Enable
- ~~VT-x~~
- ~~Above 4G decoding~~
- Hyper-Threading
- Execute Disable Bit
- EHCI/XHCI Hand-off
- ~~OS type: Windows 8.1/10 UEFI Mode~~
- ~~DVMT Pre-Allocated(iGPU Memory): 32MB~~
- SATA Mode: AHC