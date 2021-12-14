![macOS](https://badgen.net/github/checks/node-formidable/node-formidable/master/macos) ![License](https://badgen.net/badge/license/MIT/blue)

# macOS Monterey or Big Sur with OpenCore 0.7.6 on Z390 Aorus Elite motherboard (AMD RX580 or Intel UHD 630)

<table>
<tr><td align=center width=272px height=272px><img src="Apple12.png" alt="Monterey HDD"></td></tr>
<tr><td>Installing Big Sur and Monterey has been easy on my system. The same setup I use with Big Sur has worked for Monterey. This is a (short) guide using OpenCore 0.7.6. Settings for AMD dGPU as main card or iGPU as single card. EFI folder available.</td></tr>
</table>

### Hardware

<table>
<tr><td>Gigabyte Z390 Aorus Elite motherboard</td></tr>
<tr><td>Realtek ALC1220 audio</td></tr>
<tr><td>Intel I219V Ethernet</td></tr>
<tr><td>Intel i7 9700 CPU</td></tr>
<tr><td>Intel UHD Graphics 630 iGPU</td></tr>
<tr><td>AMD Radeon RX580 8GB dGPU</td></tr>
<tr><td>Fenvi FV-T919 wifi and Bluetooth</td></tr>
</table>

### What works well?

<table>
<tr><td>Radeon RX580 (VDA decoder fully supported)</td></tr>
<tr><td>Shutdown, restart and sleep</td></tr>
<tr><td>Audio (ALC1220 and HDMI)</td></tr>
<tr><td>USB ports (USBMap.kext for this motherboard)</td></tr>
<tr><td>Airdrop, iMessage</td></tr>
</table>

### BIOS settings (version F10h)

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

### OpenCore 0.7.6

I have used the latest OpenCore version, 0.7.6, with the same settings that I use for Big Sur. For the installation to be successful, 3 parameters related to security must be set:

- `SecureBootModel=j160` or `SecureBootModel=Default` in config.plist (Apple secure boot `j160` corresponds to MacPro7,1 and `Default` sets the same model as in SMBIOS)
- SIP enabled (`csr-active-config=00000000` in config.plist)
- Gatekeeper enabled (`sudo spctl --master-enable` in Terminal).

These security options can be changed after installation as they do not are required for Monterey to run.

### SMBIOS

SMBIOS model that works best on my Monterey system is MacPro7,1. This Mac model requires:

- AMD RX580 dGPU as main card
- Intel 630 iGPU disabled in BIOS
- RestrictEvents.kext to avoid RAM misconfiguration warnings.

### CPUFriend.kext

Although the CPU is well detected with MacPro's SMBIOS, my guess is that it does not run at low frequency as often as it does with iMac19.1. For this reason, I have generated a CPUFriendDataProvider.kext extension from the CPUFriendFriend command to accompany CPUFriend.kext. With these 2 kexts (CPUFriendDataProvider.kext + CPUFriend.kext) the CPU shows correct power management and frequency drops to 800 MHz at system idle.

### SSDTs, drivers and tools

**SSDTs**

- SSDT-AWAC-DISABLE: to fix errors with system clock on Z390, B460, Z490 motherboards
- SSDT-EC-USBX: fake Embedded Controller on Skylake and later, also fix USB power
- SSDT-PLUG: power management on Haswell and newer CPUs; to configure the plugin-type=1 parameter on the first processor
- SSDT-PMC: native NVRAM support on systems that lack it, for example Z390 chipsets
- SSDT-USBW: to wake from sleep with a single mouse or keyboard touch (this SSDT works with USBWakeFixup.kext) >> very likely not needed if USB device in DeviceProperties has `acpi-wake-type=01`.

**Drivers**

- CrScreenshotDxe.efi: Screenshots in OpenCore
- HfsPlus.efi: to recognize HFS+ devices
- OpenCanopy.efi: graphical picker with themes
- OpenRuntime.efi: essential driver to run macOS.

**Tools**

- OpenShell.efi: UEFI shell to perform command line tasks from OpenCore

### config.plist

Settings are generally the same as for Big Sur. Some significant details:

- DeviceProperties >> Add >> PciRoot(0x0)/Pci(0x14,0x0): acpi-wake-type as data=01, to improve wake from sleep
- Misc >> Boot >> PickerAttributes=144 to enable Flavours system
- NVRAM> 7C436110-AB2A-4BBB-A880-FE41995C9F82> boot-args: alcid=13 for audio (you can also try 11, both layout-id work fine)
- Misc >> Security >> AllowToggleSip=True to show in the picker the ToggleSIP tool that allows to easily switch between SIP enabled and SIP disabled for the current boot.

### Intel UHD 630

I prefer to use MacPro7,1 SMBIOS, it requires iGPU to be disabled in BIOS. This configuration is the one in the *EFI-macpro* folder.
If you don't have an external graphics card and need to use the integrated one, you have to use iMac19,1 SMBIOS model and *EFI-intel630* folder that has these modifications:

<table>
<tr><td>Enable iGPU in BIOS (and set it as main card)</td></tr>
<tr><td>Removed RestrictEvents.kext, CPUFriendDataProvider.kext and CPUFriend.kext</td></tr>
<tr><td>Added in config.plist >> boot-args >> igfxonln=1</td></tr>
<tr><td>Added in config.plist >> DeviceProperties >> code to patch the framebuffer so that the iGPU is well detected</td></tr>
</table>

       <key>PciRoot(0x0)/Pci(0x2,0x0)</key>
        <dict>
            <key>AAPL,ig-platform-id</key>
            <data>BwCbPg==</data>
            <key>framebuffer-patch-enable</key>
            <data>AQAAAA==</data>
            <key>framebuffer-con0-enable</key>
            <data>AQAAAA==</data>
            <key>framebuffer-con1-enable</key>
            <data>AQAAAA==</data>
            <key>framebuffer-con2-enable</key>
            <data>AQAAAA==</data>
            <key>framebuffer-con0-alldata</key>
            <data>AQAJAAAEAADHAwAA</data>
            <key>framebuffer-con1-alldata</key>
            <data>AgAKAAAEAADHAwAA</data>
            <key>framebuffer-con2-alldata</key>
            <data>AwQIAAAIAADHAwAA</data>
            <key>framebuffer-stolenmem</key>
            <data>AAAwAQ==</data>
            <key>hda-gfx</key>
            <string>onboard-1</string>
            <key>name</key>
            <string>Intel UHD Graphics 630</string>
        </dict>
      
The config.plist file in *EFI-intel630* folder is already set in this way.
Note: don't forget to rename the EFI folder from *EFI-macpro* or *EFI-intel630* to *EFI*.

### Installing Big Sur or  Monterey

The process is almost the same for installation and for update:

- You need a working EFI folder
- Download macOS from Software Update or create USB installer; I don't comment about creating USB installer because there are a lot of sites with this info
- Run Install macOS Big Sur / Monterey app from the Desktop or the setup program from the booted USB
- The process has 2 reboots booting from Macintosh HD and a third reboot booting from the target disk with Monterey.

### Important!

1. Do ResetNVRAM the first time you boot a new EFI.
2. Press spacebar to show auxiliary entries in the picker.
