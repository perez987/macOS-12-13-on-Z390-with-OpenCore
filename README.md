# macOS Ventura / Monterey / Sonoma with OpenCore on Z390 Aorus Elite motherboard (AMD RX 6600 or Intel UHD 630)

<table>
<tr><td align=center width=272px height=272px><img src="macOS13.png" alt="Monterey HDD"></td></tr>
<tr><td><b><ul>
	<li>Guide using OpenCore for Ventura / Monterey / Sonoma on Gigabyte Z390 Aorus Elite motherboard</li>
	<li>Settings for AMD dGPU as main card or iGPU as single card</li>
	<li>EFI folder available for different SMBIOS.</li>
	</ul></b></td></tr>
</table>

### Hardware

<table>
       <tr><td>Motherboard</td><td>Gigabyte Z390 Aorus Elite</td></tr>
       <tr><td>CPU</td><td>Intel i7 9700</td></tr>
       <tr><td>iGPU</td><td>Intel UHD Graphics 630</td></tr>
       <tr><td>dGPU</td><td>AMD Radeon RX 6600 8GB</td></tr>
       <tr><td>Sound</td></td><td>Realtek ALC1220</td></tr>
       <tr><td>Ethernet</td><td>Intel I219V7</td></tr>
       <tr><td>Wifi + BT</td><td>Fenvi FV-T919 BCM94360CD</td></tr>
</table>

### What works well?

<table>
<tr><td>Radeon graphics card (VDA decoder fully supported)</td></tr>
<tr><td>Shutdown, restart and sleep</td></tr>
<tr><td>Audio (ALC1220 and HDMI)</td></tr>
<tr><td>USB ports (USB ports map for this motherboard)</td></tr>
<tr><td>Airdrop, iMessage (wifi + Bluetooth)</td></tr>
</table>

### BIOS settings (version F10)

<table>
<tr><td>CFG Lock: Disabled</td></tr>
<tr><td>CSM: Disabled</td></tr>
<tr><td>VT-d: Disabled</td></tr>
<tr><td>Fast Boot: Disabled</td></tr>
<tr><td>OS Type: Windows 8/10 WHQL</td></tr>
<tr><td>Platform Power Management: Disabled</td></tr>
<tr><td>XHCI Hand-Off: Enabled</td></tr>
<tr><td>Network Stack: Disabled</td></tr>
<tr><td>Wake on LAN: Disabled</td></tr>
<tr><td>Secure Boot: Disabled</td></tr>
<tr><td>DVMT Pre-Allocated: 256M or higher</td></tr>
<tr><td>Integrated Graphics: Disabled / Enabled (according to SMBIOS)</td></tr>
</table>

-----

### Sonoma beta notes (June 2023)

- OTA updates (incremental updates from Software Update) don't work (the update doesn't even show up in the Software Update panel) except with iMac19,1 SMBIOS, probably related to Hackintosh missing Apple T2 chip since iMac19,1 lacks T2 chip but iMacPro1,1 and MacPro7,1 both have T2 chip.
- If you use iMacPro1,1 or MacPro7,1 SMBIOS, there is another possibility to have incremental updates without changing the SMBIOS: add RestrictEvents extension and `revpatch=sbvmm` argument in boot args.
- Both changes (SMBIOS and RestrictEvents with boot arg) can be reverted after the update.
- Updating via the full installer package works with any of the 3 SMBIOS.
- Broadcom BCM4360 series Wi-Fi do not work in Sonoma because the system does not include the drivers. Therefore, the Fenvi T919 card, which has worked very well OOTB so far, in Sonoma only provides Bluetooth. This is a serious inconvenience because the different USB wifi dongles that work in Sonoma lose some of the functionality of the Apple ecosystem like AirDrop, iPhone camera, etc.
- Otherwise, Sonoma beta works fine on this PC, with few and minor bugs detected so far.

### Fenvi T919 wifi back on Sonoma (July 2023)

macOS Sonoma has removed support for all Broadcom Wi-Fi fitted in pre-2017 Macs:

1. AirPortBrcmNIC
- 14E4,43BA >> BCM43602
- 14E4,43A3 >> BCM4350
- 14E4,43A0 >> BCM4360

2. AirPortBrcm4360
- 14E4,4331 >> BCM94331
- 14E4,4353 >> BCM943224.

Fenvi T919 has a BCM94360CD (14E4,43A0 >> BCM4360) so the wifi doesn't work on Sonoma. Bluetooth part works as before.

OCLP developers have been working on this and have released a Sonoma-specific beta version that fixes the problem and brings Wi-Fi back to what it was in Monterey and Ventura. I know that it is not the ideal situation, many of us want to have the system as vanilla as possible but what the OCLP team has achieved is amazing. You have the instructions in this link (look for **Hackintosh notes**):

[Early preview of macOS Sonoma support now available!](https://github.com/dortania/OpenCore-Legacy-Patcher/pull/1077#issuecomment-1646934494)

You can download the latest version of this bracnh of OCLP by looking for the link with this text:

* Latest builds for the sonoma-development branch can be found below:
	* Nightly.link: OpenCore-Patcher.app (Sonoma Development).

**Note**: OCLP developers prefer that you download OCLP beta from the link they post themselves, for this reason I do not put a direct link here. It is a way to take users to the original post.

My wifi is Fenvi T919, so I have tested this OCLP preview version. I have followed TO THE LETTER the instructions and they have worked well. I have wifi and Airdrop back in Sonoma. Note that [khronokernel](https://github.com/khronokernel)'s instructions must be followed EXACTLY.

Don't forget to enable (`Enabled=True`) 3 kexts to be added and 1 kext to be blocked, they are disabled by default.

For anyone having trouble, check out these important details:

* `csr-active-config | data | 03080000`
* `boot-args | string | amphi=0x80`
* `com.apple.iokit.IOSkywalkFamily `blocked
* `IOSkywalk.kext`, `IO80211FamilyLegacy.kext` and `AirPortBrcmNIC.kext` added in this order (Kexts folder and config.plist).

I guess incremental updates are lost with this setup, updates can be done from Software Update but the full installer package is downloaded and not the delta package that only contains the incremental changes.

In summary, the OCLP approach works, at least for me. As a preliminary fix, maybe it doesn't work on some systems.
  
**Important**: `com.apple.iokit.IOSkywalkFamily` blocking must have `Enabled=True` and `Strategy=Exclude`. Otherwise you get kernel panic at boot.

<details>
<summary><b>Image: Wifi back in Sonoma</b></summary>
<br>
<img src="Wifi active again.png">
</details>

-----

### OpenCore

For the installation / update to be successful, 3 parameters related to security must be set:

- `SecureBootModel=j137` or `j160`or `Default` in config.plist (Apple secure boot `j137` corresponds to iMacPro1,1, `j160` to MacPro7,1 and `Default` sets the same model as in SMBIOS)
- SIP enabled (`csr-active-config=00000000` in config.plist)
- Gatekeeper enabled (`sudo spctl --master-enable` in Terminal).

These security options can be changed after installation as they do not are required for Monterey / Ventura to run.

### SMBIOS

iMac19,1 SMBIOS requires:

- AMD dGPU as main card
- Intel 630 iGPU enabled in BIOS
- iGPU code for headless mode in config.plist.

iMacPro1,1 or MacPro7,1 SMBIOS require:

- AMD dGPU as main card
- Intel 630 iGPU disabled in BIOS (this Mac models lack integrated GPU, they have only AMD GPU)
- RestrictEvents.kext to avoid RAM misconfiguration warnings (only MacPro7,1)
- CPUFriend.kext: not mandatory but in my opinion improves CPU Power Management (only MacPro7,1).

### CPUFriend.kext

Although the CPU is well detected with MacPro's SMBIOS, my guess is that it does not run at low frequency as often as it does with iMac19.1 or iMacPro1,1. For this reason, I have generated a CPUFriendDataProvider.kext extension by the CPUFriendFriend command to accompany CPUFriend.kext. Remember that this kext is specific to my CPU: i7 9700, try building your own if yours differs. With these 2 kexts (CPUFriendDataProvider.kext + CPUFriend.kext) the CPU shows correct power management and frequency drops to 800 MHz at system idle.

### SSDTs, drivers and tools

**SSDTs**

- SSDT-AWAC-DISABLE: to fix errors with system clock on Z390, B460, Z490 motherboards
- SSDT-EC-USBX: fake Embedded Controller on Skylake and later, also fix USB power
- SSDT-PLUG: power management on Haswell and newer CPUs; to configure the plugin-type=1 parameter on the first processor
- SSDT-PMC: native NVRAM support on systems that lack it, for example Z390 chipsets
- SSDT-USBW: to wake from sleep with a single mouse or keyboard touch (this SSDT works with USBWakeFixup.kext) >> very likely not needed if USB device in DeviceProperties has `acpi-wake-type=01` >> removed with OpenCore 0.8.9 release, using `acpi-wake-gpe=6D` in USB DeviceProperties instead.

**Drivers**

- HfsPlus.efi: to recognize HFS+ devices
- OpenRuntime.efi: essential driver to run macOS
- OpenCanopy.efi: graphical picker with themes
- CrScreenshotDxe.efi: Screenshots in OpenCore.

**Tools**

- OpenShell.efi: UEFI shell to perform command line tasks from OpenCore

### config.plist

Some settings:

- DeviceProperties >> Add >> PciRoot(0x0)/Pci(0x14,0x0): acpi-wake-type as data=01, to improve wake from sleep
- Misc >> Boot >> PickerAttributes=144 to enable Flavours system
- NVRAM> 7C436110-AB2A-4BBB-A880-FE41995C9F82> boot-args: `alcid=7 agdpmod=pikera`
	(`agdpmod=pikera` not needed with RX 580 or Polaris card, only with RX 6600 and Navi cards)
- Misc >> Security >> AllowToggleSip=True to show in the picker the ToggleSIP tool that allows to easily switch between SIP enabled and SIP disabled for the current boot.

-----

### Intel UHD 630 headless mode

- iGPU and dGPU must be enabled in BIOS with dGPU as primary
- There should be no cable between iGPU HDMI port and any type of display
- Lilu and WhatEverGreen properly installed
- SMBIOS iMac19.1.

You have to add in `DeviceProperties >> Add`:

``` xml
<key>PciRoot(0x0)/Pci(0x2,0x0)</key>
<dict>
	<key>AAPL,ig-platform-id</key>
	<data>AwCRPg==</data>
	<key>device-id</key>
	<data>mz4AAA==</data>
	<key>enable-metal</key>
	<data>AQAAAA==</data>
</dict>
```

This code has data values in Base64, in plist editors they can be seen as hexadecimal, e.g. `AwCRPg==` in Base64 (_AAPL,ig-platform-id_) = `0300913E` in hexadecimal.

To check if the VDA Decoder function is activated you can get Hackintool app (_Fully Supported_ or _Failed_ in the first System tab).

Notes:

- `AAPL,ig-platform-id=07009B3E` is mandatory to detect the iGPU as Coffee Lake
- `device-id=9B3E000` to be displayed as `Intel UHD Graphics 630` instead of `Kabylake Unknown`
- `enable-metal=01` to enable Metal 3 in Ventura

<details>
<summary><b>Image: iGPU as secondary card</b></summary>
<br>
<img src="iGPU as secondary card.png">
</details>

-----

### Intel UHD 630 as single GPU

If you don't have an external graphics card and need to use the iGPU as single card, you have to use iMac19,1 SMBIOS with code in config.plist to patch the framebuffer and other properties so that the iGPU is well detected:

``` xml
<key>PciRoot(0x0)/Pci(0x2,0x0)</key>
<dict>
	<key>AAPL,ig-platform-id</key>
	<data>BwCbPg==</data>
	<key>device-id</key>
	<data>mz4AAA==</data>
	<key>device_type</key>
	<string>VGA compatible controller</string>
	<key>enable-hdmi20</key>
	<data>AQAAAA==</data>
	<key>enable-metal</key>
	<data>AQAAAA==</data>
	<key>framebuffer-con0-busid</key>
	<data>AAAAAA==</data>
	<key>framebuffer-con0-enable</key>
	<data>AQAAAA==</data>
	<key>framebuffer-con0-pipe</key>
	<data>EgAAAA==</data>
	<key>framebuffer-con1-busid</key>
	<data>AAAAAA==</data>
	<key>framebuffer-con1-enable</key>
	<data>AQAAAA==</data>
	<key>framebuffer-con1-pipe</key>
	<data>EgAAAA==</data>
	<key>framebuffer-con2-busid</key>
	<data>BAAAAA==</data>
	<key>framebuffer-con2-enable</key>
	<data>AQAAAA==</data>
	<key>framebuffer-con2-pipe</key>
	<data>EgAAAA==</data>
	<key>framebuffer-con2-type</key>
	<data>AAgAAA==</data>
	<key>framebuffer-patch-enable</key>
	<data>AQAAAA==</data>
	<key>framebuffer-stolenmem</key>
	<data>AAAwAQ==</data>
	<key>hda-gfx</key>
	<string>onboard-1</string>
	<key>force-online</key>
	<data>AQAAAA==</data>
	<key>rps-control</key>
	<data>AQAAAA==</data>
</dict>
```

Note:

- `force-online=01` to force online status on all displays (mandatory).

<details>
<summary><b>Image: iGPU as main card</b></summary>
<br>
<img src="iGPU as main card.png">
</details>

-----

### AMD RX 6600 on Ventura with MacPro or iMacPro SMBIOS

AMD Navi cards run fine on Ventura when using iMac SMBIOS with `agdpmod=pikera` in boot args as the only needed setting. But when using MacPro or iMacPro SMBIOS a lot of users have reported black screen. The simplest way to fix this is to add in DeviceProperties of config.plist properties that set Henbury framebuffer for each of the 4 ports of this GPU.

By default, Radeon framebuffer (`ATY,Radeon`) is loaded. But, in AMDRadeonX6000Framebuffer.kext >> Contents >> Info.plist we can see that AMDRadeonNavi23Controller has `ATY,Henbury` and 6600 series are Navi 23. This is why this framebuffer is selected.

The patch is added in this way:

``` xml
<key>DeviceProperties</key>
    <dict>
        <key>Add</key>
        <dict>
            <key>PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)</key>
            <dict>
                <key>@0,name</key>
                <string>ATY,Henbury</string>
                <key>@1,name</key>
                <string>ATY,Henbury</string>
                <key>@2,name</key>
                <string>ATY,Henbury</string>
                <key>@3,name</key>
                <string>ATY,Henbury</string>
            </dict>
        </dict>
        <key>Delete</key>
        <dict/>
    </dict>
```
Notes:

- PCI path to the GPU may be the same on your system but it is convenient to check it with Hackintool (app) or gfxutil (Terminal utility).
- This is not needed for Monterey.
- This is not needed for Polaris cards (RX 580).
- Henbury patch drops down significantly GeekBench 5 Metal scores.

If needed for other Navi cards, the framebuffers to be loaded are different for each family:

<table>
<tr><td>5500</td><td>ATY,Python</td></tr>
<tr><td>5700</td><td>ATY,Adder</td></tr>
<tr><td>6600</td><td>ATY,Henbury</td></tr>
<tr><td>6800</td><td>ATY,Belknap</td></tr>
<tr><td>6900</td><td>ATY,Carswell</td></tr>
</table>

**1. Alternative method to avoid black screen with MacPro or iMacPro SMBIOS** (thanks [@dreamwhite](https://github.com/dreamwhite))

Using SSDT-BRG0.aml fixes black screen on Ventura with SMBIOS models lacking iGPU. This SSDT allows to define a missing `pci-bridge` device. With it, the Henbury patch is no longer necessary.

As noted above, the Henbury patch drops down the GeekBench 5 scores, however with SSDT-BRG0 expected scores are got, in line with those got by many users with this graphics card.

SSDT-BRG0:

```c++
/*
 * This table provides an example of creating a missing ACPI device
 * to ensure early DeviceProperty application. In this example
 * a GPU device is created for a platform having an extra PCI
 * bridge in the path - PCI0.PEG0.PEGP.BRG0.GFX0:
 * PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)/Pci(0x0,0x0)
 * Such tables are particularly relevant for macOS 11.0 and newer.
 */

DefinitionBlock ("", "SSDT", 2, "ACDT", "BRG0", 0x00000000)
{
    External (_SB_.PCI0.PEG0.PEGP, DeviceObj)

    Scope (\_SB.PCI0.PEG0.PEGP)
    {
        /*
         * This is a PCI bridge device present on PEGP.
         * Normally seen as pci-bridge in I/O Registry.
         */
        Device (BRG0)
        {
            Name (_ADR, Zero)
            Method (_STA, 0, NotSerialized)  // _STA: Status
            {
                If (_OSI ("Darwin"))
                {
                    Return (0x0F)
                }
                Else
                {
                    Return (Zero)
                }
            }

            /*
             * This is an actual GPU device present on the bridge.
             * Normally seen as display in I/O Registry.
             */
            Device (GFX0)
            {
                Name (_ADR, Zero)  // _ADR: Address
            }
        }
    }
}
```
Be sure of the PCI path to your dGPU, mine is `/PCI0@0/PEG0@1/PEGP@0/GFX0@0` so the SSDT has these lines: 
```
External (_SB_.PCI0.PEG0.PEGP, DeviceObj)

Scope (\_SB.PCI0.PEG0.PEGP)
```
Modify the SSDT (if needed) to be set within your system. How to chek the PCI path?

- gfxutil utility in Terminal, look for the line containing GFX0 at the end
- Hackintool >> PCIe tab >> pointer over Navi 23 line >> copy IOReg patch.

**2. Alternative method to avoid black screen with MacPro or iMacPro SMBIOS**

There are some more advanced SSDT configurations such as SSDT-VEGA.aml which, in addition to creating devices, does so by mimicking the way real Macs are configured (EGP0 and EGP1 devices instead of BRG0).
 
SSDT-VEGA has the same benefits as SSDT-BRG0: Henbury patch not needed and good GeekBench 5 scores.
 
It also requires checking the PCI path to your graphics card. In the example code I keep the same path as in the previous SSDT-BRG0.
```c++
DefinitionBlock ("", "SSDT", 2, "HACK", "VEGA", 0x00000000)
{
    External (_SB_.PCI0, DeviceObj)
    External (_SB_.PCI0.PEG0, DeviceObj)
    External (_SB_.PCI0.PEG0.PEGP, DeviceObj)

    Scope (\_SB)
    {
        Scope (PCI0)
        {
            Scope (PEG0)
            {
                Scope (PEGP)
                {
                    Method (_STA, 0, NotSerialized)  // _STA: Status
                    {
                        If (_OSI ("Darwin"))
                        {
                            Return (Zero)
                        }
                        Else
                        {
                            Return (0x0F)
                        }
                    }
                }

                Device (EGP0)
                {
                    Name (_ADR, Zero)  // _ADR: Address
                    Method (_STA, 0, NotSerialized)  // _STA: Status
                    {
                        If (_OSI ("Darwin"))
                        {
                            Return (0x0F)
                        }
                        Else
                        {
                            Return (Zero)
                        }
                    }

                    Device (EGP1)
                    {
                        Name (_ADR, Zero)  // _ADR: Address
                        Device (GFX0)
                        {
                            Name (_ADR, Zero)  // _ADR: Address
                        }

                        Device (HDAU)
                        {
                            Name (_ADR, One)  // _ADR: Address
                        }
                    }
                }
            }
        }
    }
}  
```
-----

### Installing Monterey / Ventura

The process is almost the same for installation and for update:

- You need a working EFI folder
- Download macOS from Software Update or create USB installer; I don't comment about creating USB installer because there are a lot of sites with this info
- Run Install macOS Monterey / Ventura app or the setup program from the booted USB
- The process has 2 reboots booting from Install macOS disk and a third reboot booting from the target disk with Monterey.

----

### SMBIOS and config.plist files

There are different configuration files. Variants are included for 4 possible SMBIOS:

- iMac19,1 with AMD dGPU + iGPU headless mode
- iMac19,1 with iGPU as main card without dGPU
- MacPro7,1 with dGPU + iGPU disabled
- iMacPro1,1 with dGPU + iGPU disabled.

List of config.plist files:

- config-13-imac-amd.plist: iMac19,1 + dGPU AMD + iGPU enabled in BIOS
- config-13-imac-intel.plist: iMac19,1 + iGPU enabled in BIOS as main card
- config-13-imacpro.plist: iMacPro1,1 + dGPU AMD + iGPU disabled in BIOS
- config-13-macpro.plist: MacPro7,1 + dGPU AMD + iGPU disabled in BIOS.

Notes

- rename selected config file to config.plist
- current GPU is AMD RX 6600 XT; for RX 580 and other Polaris cards remove `agdpmod=piker`a from boot-args and don't use the framebuffer patch
- add serial numbers for the SMBIOS model
- USB ports map is specific for muy motherboard: Z390 Aorus Elite.

### Important!

1. Don't forget to rename the selected config file to config.plist.
2. Do ResetNVRAM the first time you boot a new EFI.
3. Press spacebar to show auxiliary entries in the picker.

-----

### Credits

<table>
<tr><td><a href=https://github.com/acidanthera target=_blank>Acidanthera</a></td><td>OpenCore and kexts</td></tr>
<tr><td><a href=https://dortania.github.io target=_blank>Dortania</a></td><td>OpenCore guides</td></tr>
<tr><td><a href=https://www.insanelymac.com/forum/ target=_blank>InsanelyMac</a></td><td>Hackintosh forum</td></tr>
<tr><td><a href=https://www.tonymacx86.com target=_blank>tonymacx86</a></td><td>Hackintosh forum</td></tr>
<tr><td><a href=https://github.com/dortania/OpenCore-Legacy-Patcher) target=_blank>OCLP</a></td><td>OpenCore Legacy Patcher</td></tr>
</table>
