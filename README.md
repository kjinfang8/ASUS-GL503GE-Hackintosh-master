# ASUS-FX504GE-Hackintosh
Discussion, necessary configurations and instructions to get [ASUS ROG GL503GE laptop](https://www.asus.com/id/ROG-Republic-Of-Gamers/ROG-Strix-GL503/) working with macOS Mojave 10.14.x. Mojave is strongly recommended. The following should also work with all ASUS GL503GE.

# Notes
1. 128 GB NVMe SSD is used for installing macOS
2. APFS partition format has to be used
3. If you are upgrading from the previous version and your partition is HFS+, better boot the installer, unmount the partition and convert it to APFS

# Pre-installation
Get yourself a Mojave USB installer with Clover installed. Important Clover settings (via Clover Configurator) are:
1. Acpi SSDT `PluginType` checked
2. Graphics: Inject Intel checked
3. Kernel Patches `Kernel LAPIC`, `KernelPM` and `AppleRTC` enabled
4. SMBIOS: MacBookPro15,2
5. UEFI Drivers
 
Kexts installed to `/EFI/CLOVER/kexts/Other`: **FakeSMC**, **VoodooPS2Controller**

# BIOS Settings
1. Secure Boot: Disabled
2. SATA mode: AHCI
3. DVMT-Preallocated: 64MB

# Post Configuration
Any changes made to kexts are to be followed by `sudo kextcache -i /`.
Every external kext mentioned is assumed to be the latest.
- First Steps
1. Lilu and FakeSMC kexts installed to `/Library/Extensions`
2. DSDT files generated by Clover to the EFI partition
3. Drop all _DSM methods
## CPU Power Management
1. Clover ACPI `PluginType` enabled
2. Clover Kernel Patches `Kernel LAPIC`, `KernelPM` and `AppleRTC` enabled
3. ACPIBatteryManager kext installed to `/Library/Extensions`
## ALC295 Realtek Audio
Internal speaker and microphone work. For Headphone output, volume balance has to be either left or right to make the sound normal.
1. `/System/Library/Extensions/AppleGFXHDA.kext` **must be removed** (ID matched but not actually compatible)
2. AppleALC kext installed to `/Library/Extensions`
3. Clover Audio injection `Inject=3` (`ResetHDA` may be enabled)
## PS/2 Keyboard
1. VoodooPS2Controller kext installed to `/Library/Extensions` and `/EFI/CLOVER/kexts/Other` (using keyboard in Recovery mode)
2. Karabiner (to remap your keyboard)
## Intel UHD 630 Graphics
1. Enabled by `device-properties` injection (have Clover's Inject Intel **unchecked**, go with `0x3E9B0000`)
2. WhateverGreen kext (with CFL backlight fix) installed to `/Library/Extensions`
## Sleep and Wake
1. Manual static patching of `USB _PRW 0x6D (instant wake)` for Skylake, focusing on adding `_PRW` to `XDCI` and/or `CNVW` (Special thanks to [MegaStood](https://github.com/MegaStood/Hackintosh-FX504GE-ES72))
2. Alternatively, compare and modify your `DSDT.aml` file with the one provided in this repository
## Backlight Control
Install the latest WhateverGreen. If you use AppleBacklightFixup, remove it.
1. Latest `SSDT-PNLF.aml` and `SSDT-PNLFCFL.aml` installed to `/EFI/Clover/ACPI/patched`

## HDMI Port + Audio
No thorough test on this.
1. Disable WhateverGreen's HDMI injection by adding a boot flag `-igfxnohdmi`
2. `device-properties` combination of `framebuffer-con1-type`, `framebuffer-con1-pipe` and `AAPL01,override-no-connect` based on [this post](https://www.tonymacx86.com/threads/uhd-630-no-hdmi-audio.265490/page-2#post-1858289)
## USB 2.0/3.1 Ports
1. Clover USB injection `Inject=false`
2. USBInjectAll and XHCI-300-series-injector kexts installed to `/Library/Extensions`
3. `SSDT-XHC.aml` installed to `/EFI/Clover/ACPI/patched` for better USB support
4. Disable unused USB ports via `/EFI/Clover/APCI/patched/SSDT-UIAC.aml`
## Realtek LAN
1. [RealtekRTL8111](https://www.insanelymac.com/forum/topic/287161-new-driver-for-realtek-rtl8111/) kext installed to `/System/Library/Extensions` and `/EFI/CLOVER/kexts/Other` (using internet in Recovery mode)
## SATA controller
1. SATA-300-series-unsupported kext installed to `/Library/Extensions`
## I2C ELAN1200 Precision TouchPad (pci8086,a368)
1. Kexts compiled from the source code (as of May 2019), can also be found in this repository under "Trackpad" folder, install the kexts to `/Library/Extensions` and rebuild kext cache. 
2. DSDT editing for trackpad support (More info [here](https://voodooi2c.github.io/#GPIO%20Pinning/GPIO%20Pinning)) (look at `** MODIFIED **`):
```
Device (TPD0)
{
    ...

    Name (SBFG, ResourceTemplate ()
    {
        GpioInt (Level, ActiveLow, ExclusiveAndWake, PullDefault, 0x0000,
            "\\_SB.PCI0.GPI0", 0x15, ResourceConsumer, ,
            )
            {   // Pin list
                0x0015
            }
    })

    ...

    Method (_STA, 0, NotSerialized)  // _STA: Status
    {
        Return (0x0F) // ** MODIFIED **
    }
    Method (_CRS, 0, NotSerialized)  // _CRS: Current Resource Settings
    {
        Return (ConcatenateResTemplate (SBFB, SBFG)) // ** MODIFIED **
    }
```

Trackpad works okay but with minor stuttering.

## Miscellaneous
1. NoTouchID kext installed to `/Library/Extensions` (MacBookPro15,2 has Touch ID)

# Things that do not work
## NVIDIA Geforce 1050 Ti (Optimus)
Discrete graphic, we probably never see the day. For now, use `SSDT-DDGPU.aml` (in `/EFI/Clover/ACPI/patched`) to power it off.
## Intel Wi-Fi AC 9560
Intel built-in Wi-Fi chipset, we again probably never see the day.
## Intel Bluetooth
It never works if Wi-Fi doesn't work.

![Screenshot](GL503GE-SS.png?raw=true)



Thank a lot :
https://www.tonymacx86.com/threads/guide-booting-the-os-x-installer-on-laptops-with-clover.148093/
https://www.tonymacx86.com/threads/guide-patching-laptop-dsdt-ssdts.152573/
https://www.tonymacx86.com/threads/guide-laptop-backlight-control-using-applebacklightfixup-kext.218222/


Source
https://github.com/alexandred/VoodooI2C
https://github.com/acidanthera
https://github.com/RehabMan
https://github.com/PoomSmart/Asus-FX504GE-Hackintosh

