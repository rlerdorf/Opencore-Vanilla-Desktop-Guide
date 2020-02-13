# Converting common properties from Clover to Opencore

Last edited: January 27, 2020

So this little(well not so little as I reread this...) page is for users who are having issues migrating from Clover to OpenCore as some of their legacy quirks are required or the Configuration.pdf isn't well suited for laptop users.  

# Kexts and Firmware drivers

See [Kexts and Firmware drivers](https://github.com/khronokernel/Opencore-Vanilla-Desktop-Guide/blob/master/clover-conversion/clover-efi.md).

# Acpi

**ACPI Renames**:

So with the transition from Clover to OpenCore we should start removing unneeded patches you may have carried along for some time:
* EHCI Patches: Recommeneded to power off the controller with [SSDT-EHCx_OFF](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-EHCx_OFF.dsl). Skylake and newer users do not have an EHCI controller so no need for this.
   * change EHC1 to EH01
   * change EHC2 to EH02
* XHCI Patches: Not needed anymore, recommended to make an [Injector kext](https://github.com/corpnewt/USBMap) instead
   * change XHCI to XHC
   * change XHC1 to XHC
* SATA patches: Purely cosmetic in macOS now
   * change SAT0 to SATA
* IMEI Patches: Handled by WhateverGreen
   * change HECI to IMEI
   * change MEI to IMEI
* GFX patches: Handled by WhateverGreen
   * change GFX0 to IGPU
   * change PEG0 to GFX0
* EC Patches: See here on best solution: [Fixing Embedded Controllers](https://khronokernel.github.io/EC-fix-guide/)
   * change EC0 to EC
   * change H_EC to EC
   * change ECDV to EC
      


**Fixes**:

* **FixIPIC**:
   * CorpNewt's [SSDTTime](https://github.com/corpnewt/SSDTTime) to make the proper SSDT, `FixHPET - Patch out IRQ Conflicts`

* **FixSBUS**:
   * [SSDT-SBUS-MCHC](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-SBUS-MCHC.dsl)

* **FixShutdown**:
   * [FixShutdown-USB-SSDT](https://github.com/khronokernel/Opencore-Vanilla-Desktop-Guide/blob/master/extra-files/FixShutdown-USB-SSDT.dsl)
   * [__PTS to ZPTS Patch](https://github.com/khronokernel/Opencore-Vanilla-Desktop-Guide/blob/master/extra-files/FixShutdown-Patch.plist)
   * This will not harm Windows or Linux installs as this is just adding missing methods that should've been there to start with. *Blame the firmware writers*

* **FixDisplay**:
   * Manual framebuffer patching, WhaterGreen does most of the work already

* **AddMCHC**:
   * [SSDT-SBUS-MCHC](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-SBUS-MCHC.dsl)

* **FixHDA**:
   * Handled by AppleALC
* **FixHPET**:
   * CorpNewt's [SSDTTime](https://github.com/corpnewt/SSDTTime) to make the proper SSDT, `FixHPET - Patch out IRQ Conflicts`
* **FixSATA**:
   * `Kernel -> Quriks -> ExternalDiskIcons -> YES`

* **FixADP1**:
   * Renames device AC0 to ADP1, see [Rename-SSDT](https://github.com/khronokernel/Opencore-Vanilla-Desktop-Guide/blob/master/extra-files/Rename-SSDT.dsl) for an example

* **FixRTC**:
   * CorpNewt's [SSDTTime](https://github.com/corpnewt/SSDTTime) to make the proper SSDT, `FixHPET - Patch out IRQ Conflicts`
* **FixTMR**:
   * CorpNewt's [SSDTTime](https://github.com/corpnewt/SSDTTime) to make the proper SSDT, `FixHPET - Patch out IRQ Conflicts`

* **AddPNLF**:
   * See Rehabman's [SSDT-PNLF](https://github.com/RehabMan/OS-X-Clover-Laptop-Config/blob/master/hotpatch/SSDT-PNLF.dsl)
* **AddIMEI**:
  * [SSDT-SBUS-MCHC](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-SBUS-MCHC.dsl)

* **FixIntelGfx**:
   * WhateverGreen handles this

* **AddHDMI**:
   * WhateverGreen handles this

**SSDT**:
* **PluginType**:
   * [SSDT-PLUG](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-PLUG.dsl)
   * Do note that this SSDT is made for systems where AppleACPICPU attaches CPU0, though some ACPI tables have theirs starting at PR00 so adjust accordingly. CorpNewt's [SSDTTime](https://github.com/corpnewt/SSDTTime) can help you with this though HEDT systems will need to manually make theirs. See [Getting started with ACPI](../extras/acpi.md) for more details
# Boot

**Boot Argument**
* `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> boot-args`

**NeverHibernate**: 
* `Misc -> Boot -> HibernateMode -> None`

# Boot Graphics

# Cpu

# Devices

**USB**:
* FixOwnership: `UEFI -> Quirk -> ReleaseUsbOwnership`
* ClockID: `DeviceProperties -> Add -> PCIRoot... -> AAPL,clock-id`

**Audio**:

For the following, you will need to know your PCIRoot for your audio controller and its name(commonly known as HDEF but also HDAS, HDAU and such), this can be found with [gfxutil](https://github.com/acidanthera/gfxutil/releases):
```
path/to/gfxutil -f HDEF
```
* Inject: `DeviceProperties -> Add -> PCIRoot... -> layout-id`
* AFGLowPowerState: `DeviceProperties -> Add -> PCIRoot... -> AFGLowPowerState -> <01000000>`
* ResetHDA: [CodecCommander](https://bitbucket.org/RehabMan/os-x-eapd-codec-commander/downloads/)

**Add Properties**: 
* No equivalent, need to specify with a PCIRoot path

**Properties**:
* `DeviceProperties -> Add`

**FakeID**:
For the following, you will need to know your PCIRoot for your device and apply their properties with `DeviceProperties -> Add`, PCIRoot can be found with [gfxutil](https://github.com/acidanthera/gfxutil/releases)
* **USB**
   * `device-id`
   * `device_type`
   * `device_type`
* **IMEI**
   * `device-id`
   * `vendor-id`
   
* **WIFI**

   * `name`
   * `compatible`
   
* **LAN**

   * `device-id`
   * `compatible`
   * `vendor-id`

* **XHCI**

   * `device-id`
   * `device_type: UHCI`
   * `device_type: OHCI`

device_type: EHCI

   * `device-id`
   * `AAPL,current-available`
   * `AAPL,current-extra`
   * `AAPL,current-available`
   * `AAPL,current-extra`
   * `AAPL,current-in-sleep`
   * `built-in`

device_type: XHCI

   * `device-id`
   * `AAPL,current-available`
   * `AAPL,current-extra`
   * `AAPL,current-available`
   * `AAPL,current-in-sleep`
   * `built-in`

# Disable Drivers

# Gui

# Graphics

**InjectIntel**:
* `Vendor`
* `deviceID`

**InjectAti**:
* `deviceID`
* `Connectors`

**InjectNvidia**:
* `DeviceID`
* `Family`


**FakeIntel**:
* `device-id`
* `vendor-id`

**FakeAti**:
* `device-id`
* `ATY,DeviceID`
* `@0,compatible`
* `vendor-id`
* `ATY,VendorID`

**RadeonDeInit**:
* [Radeon-Denit-SSDT](https://github.com/khronokernel/Opencore-Vanilla-Desktop-Guide/blob/master/extra-files/Radeon-Deinit-SSDT.dsl)
   * Do note that this is meant for gfx0, adjust for your system

# Kernel and Kext Patches

**KernelPm**: 
* `Kernel -> Quirks -> AppleXcpmCfgLock -> YES`

**AppleIntelCPUPM**:
* `Kernel -> Quirks -> AppleCpuPmCfgLock -> YES`

**DellSMBIOSPatch**:
* `Kernel -> Quirks -> CustomSMBIOSGuid -> YES`
* `PlatformInfo -> UpdateSMBIOSMode -> Custom`

**KextsToPatch**:
* `Kernel -> Patch`

**KernelToPatch**:
* `Kernel -> Patch`

**Kernel LAPIC**:
* `Kernel -> Quirks -> LapicKernelPanic -> YES`

**KernelXCPM**:
* `Kernel -> Quirks -> AppleXcpmExtraMsrs -> YES`

For an extensive list of patches, please compare [OpenCore's `CommonPatches.c`](https://github.com/acidanthera/OcSupportPkg/blob/b2b0fa3c060403fdf0d42d319bd0902df62959f0/Library/OcAppleKernelLib/CommonPatches.c) with [Clover's `kernel_patcher.c` ](https://github.com/CloverHackyColor/CloverBootloader/blob/master/rEFIt_UEFI/Platform/kernel_patcher.c). Some patches are not transfered over so if you're having issues this is the section to check, example is converting the [`KernelIvyBridgeXCPM()`](https://github.com/CloverHackyColor/CloverBootloader/blob/master/rEFIt_UEFI/Platform/kernel_patcher.c#L1134-L1216) to Opencore:

```
Base: _xcpm_bootstrap
Comment: _xcpm_bootstrap (Ivy Bridge) 10.15
Count: 1
Enabled: YES
Find: 8D43C43C22
Identifier: kernel
Limit: 0
Mask: FFFF00FF
MinKernel: 19.
MaxKernel: 19.99.99
Replace: 8D43C63C22
ReplaceMask: 0000FF0000
Skip: 0
```
[Source](https://github.com/khronokernel/Opencore-Vanilla-Desktop-Guide/issues/32)

For Low end Haswell+ like Celerons, please see here for recommended patches: [Bugtracker Issues 365](https://github.com/acidanthera/bugtracker/issues/365)

**USB Port Limit Patches**:
* `Kernel -> Quirks -> XhciPortLimit -> YES`

**External Icons Patch**:
* `kernel -> Quirks -> ExternalDiskIcons -> YES`
* Used for when you interneal disk are seen as external on macOS

**AppleRTC**
* This has been turned into a kext patch, this is needed anytime you have either BIOS reset or safe mode issues.
* Under `Kernel -> patch`:

|Enabled|String|YES|
|:-|:-|:-|
|Comment|String|Disable RTC checksum update on poweroff|
|Count|Number|1|
|Base|String|__ZN8AppleRTC14updateChecksumEv|
|Identifier|String|com.apple.driver.AppleRTC|
|Limit|Number|0|
|Find|Data||
|Replace|Data|c3|

Alternative would be to use [RTCMemoryFixup](https://github.com/acidanthera/RTCMemoryFixup)


# Rt Variables

**ROM**:
* No direct translation for `UseMacAddr0` as you need to provide your hadware ROM, can be found in `System Preferences -> Network -> Advanced -> Hardware`
* Also verify your En0 is still built-in when running OpenCore, this can break iMessage and icloud when there's no `built-in` property.

**MLB**: 
* `PlatformInfo -> Generic -> MLB`

**BooterConfig**: 

`NVRAM -> Add -> 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14-> UIScale`

* 0x28: `01`
* 0x2A: `02`

**CsrActiveConfig**:
`NVRAM -> Add -> csr-active-config`

* 0x0: `00000000`
* 0x3: `03000000`
* 0x67: `67000000`
* 0x3E7: `E7030000`

# SMBIOS

**Product Name**:
* `PlatformInfo -> Generic -> SystemProductName`

**Serial Number**:
* `PlatformInfo -> Generic -> SystemSerialNumber`

**Board Serial Number**:
* `PlatformInfo -> Generic -> MLB`

**SmUUID**:
* `PlatformInfo -> Generic -> SystemUUID`

# System Parameters

**NvidiaWeb**: 

What this does is apply ```sudo nvram nvda_drv=1``` on every boot. To get a similar effect you can find it under the following path:
* `NVRAM -> Add -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> nvda_drv: <31>`

