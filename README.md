# OpenCore Lenovo B330-15IKBR

OpenCore EFI for the **Lenovo B330-15IKBR (81M1)** running **macOS Ventura 13.7.8**, tuned to behave as close as possible to a real MacBook Pro: native power management, Mac-style boot, working sleep, HDMI audio, native trackpad gestures and full battery reporting.

> [!IMPORTANT]
> This EFI ships with **placeholder SMBIOS data**. Before your first boot you must generate your own serial, MLB, UUID and ROM. See [Before you boot](#before-you-boot).

## Hardware

| Component | Model | Notes |
|---|---|---|
| Laptop | Lenovo B330-15IKBR (81M1) | |
| CPU | Intel Core i3-7020U (Kaby Lake, 2C/4T, 2.3 GHz, no Turbo) | See [Other CPU variants](#other-cpu-variants) |
| iGPU | Intel HD Graphics 620 (`0x5916`) | |
| Chipset | Intel Sunrise Point-LP (100 series) | |
| RAM | 2 × 4 GB DDR4-2133 (dual channel) | |
| Storage | SATA SSD (AHCI) | |
| Display | 15.6" 1366 × 768, eDP | |
| External display | HDMI 1.4 | Video and audio |
| Audio | Realtek ALC230 (`0x10EC0230`, subsystem `17AA:3820`) | Reported as ALC236 by Linux |
| Ethernet | Realtek RTL8111GU | |
| Wi-Fi / Bluetooth | Broadcom BCM94360NG (M.2 NGFF) | Replaced the stock card |
| Trackpad | Synaptics SYNA2B49 (I2C, RMI4) | Not a Precision Touchpad |
| Keyboard | PS/2 | |
| Webcam | EasyCamera (USB) | |
| Card reader | Realtek USB (`0BDA:0177`) | |

## Software

| | Version |
|---|---|
| macOS | Ventura 13.7.8 (22H730) |
| OpenCore | 1.0.8 RELEASE |
| SMBIOS | MacBookPro14,1 |

### Kexts

| Kext | Version | Purpose |
|---|---|---|
| Lilu | 1.7.3 | Patching framework |
| VirtualSMC + SMCProcessor + SMCBatteryManager | 1.3.9 | SMC emulation, CPU sensors, battery |
| WhateverGreen | 1.7.2 | iGPU framebuffer, HDMI and RPS control (`rps-control`) |
| AppleALC | 1.9.9 (layout 21 build) | Audio, see [Audio](#audio) |
| CPUFriend + CPUFriendDataProvider | 1.3.0 / 1.0.1 | CPU frequency profile (i3-7020U only): 800 MHz floor, EPP `0x40`, 1600 MHz cap in Low Power Mode |
| ECEnabler | 1.0.6 | Reads 16-bit EC fields for battery info |
| YogaSMC | 1.5.3 | Lenovo EC: battery conservation mode, sensors, hotkeys |
| BrightnessKeys | 1.0.4 | Fn brightness keys |
| VoodooI2C + VoodooRMI (+ RMII2C) | 2.9.1 / 1.4.3 | I2C trackpad with native multitouch gestures |
| VoodooI2CHID | 1.0 | Disabled by default; fallback HID driver for the trackpad |
| VoodooPS2Controller | 2.3.8 | Keyboard only (mouse and trackpad plugins disabled) |
| RealtekRTL8111 | 2.4.2 | Ethernet |
| USBToolBox + UTBMap | 1.2.0 / 1.1 | USB port map (9 ports) |

### ACPI

| SSDT | Purpose |
|---|---|
| SSDT-PLUG | Enables XCPM (`plugin-type`) |
| SSDT-EC, SSDT-USBX | Fake `EC` device for macOS (the native `EC0` is left untouched) and USB power properties |
| SSDT-PNLF | Backlight control |
| SSDT-ALS0 | Fake ambient light sensor (keeps brightness settings) |
| SSDT-GPI0 | Enables the GPIO controller for the I2C trackpad |
| SSDT-XOSI | Windows OSI so the I2C trackpad is exposed |
| SSDT-GPRW | Prevents instant wake from USB/HDA (with `GPRW` to `XPRW` rename) |
| SSDT-SBUS, SSDT-MCHC | SMBus (`BUS0` + `DVL0`, loads AppleSMBusController) and memory controller devices |
| SSDT-HPET | HPET on IRQ 0 and 8, like a real Mac, with `HPET`/`IPIC`/`RTC`/`TIMR` IRQ patches (generated with [SSDTTime](https://github.com/corpnewt/SSDTTime)) |

## Status

### Working
- Full graphics acceleration: Metal, H.264/HEVC hardware encode and decode (Quick Sync)
- Brightness control and Fn brightness keys
- HDMI video **and audio** (see [Troubleshooting](#troubleshooting))
- Internal speakers, headphone jack with automatic switching, internal mic and headset mic
- Native CPU power management (XCPM) with a custom frequency profile (800 MHz to 2.3 GHz)
- Sleep and wake (lid, menu, power button)
- Battery percentage, cycle count, health and Lenovo battery conservation mode
- Trackpad with native macOS multitouch gestures
- Keyboard, Fn keys
- USB 2.0 / 3.0 ports
- Ethernet
- iMessage, FaceTime and App Store (with your own valid SMBIOS)
- SD card reader
- Webcam
- TRIM on SATA SSD
- Mac-like boot: hidden picker, boot chime, `Cmd+R` / `Cmd+V` hotkeys
- SIP enabled (`csr-active-config = 00000000`)

### Not working
- **Intel GuC firmware loading (`igfxfw=2`)**: not possible on this platform. The Management Engine on 100-series chipsets never returns the firmware hash, and forcing it breaks acceleration.

## Before you boot

1. **Generate your SMBIOS** with [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) for `MacBookPro14,1`. In `config.plist > PlatformInfo > Generic`, replace:
   - `SystemSerialNumber` (placeholder `W00000000001`)
   - `MLB` (placeholder `M0000000000000001`)
   - `SystemUUID` (placeholder `00000000-0000-0000-0000-000000000000`)
2. **Set `ROM`** to the MAC address of your built-in Ethernet (`en0`), without colons. The placeholder is `112233445566`. In macOS: `ifconfig en0 | grep ether`.
3. Check the serial on [Apple's coverage page](https://checkcoverage.apple.com). It should be **invalid** (not in use by a real Mac).
4. Copy the `EFI` folder to the EFI partition of your boot disk. For a new installation, follow [Installing macOS](#installing-macos) instead.

Placeholder SMBIOS values are enough to install macOS. Your own serial, MLB, UUID and ROM are only required for iMessage, FaceTime and the App Store, but set them before you sign in with your Apple ID.

### Config files

| File | Use |
|---|---|
| `EFI/OC/config.plist` | Daily boot: hidden picker, boot chime, no verbose, Apple Secure Boot enabled |
| `EFI/OC/config-install.plist` | Installation and troubleshooting: verbose boot, logging, visible picker, UEFI Shell |

OpenCore only reads the file named `config.plist`. To use the installation profile, rename the files as described below. Both files have the same ACPI, kexts, device properties and quirks, so a macOS installed with one boots with the other.

## BIOS settings

Typical settings for this model (option names can vary between BIOS versions):

- Boot mode: **UEFI**
- Secure Boot: **Disabled**
- SATA controller mode: **AHCI**
- Intel Virtualization Technology: either value works (`DisableIoMapper` is enabled)

`AppleXcpmCfgLock` is enabled to cope with the locked MSR `0xE2` (CFG Lock) of the stock BIOS.

## Installing macOS

### What the installation config changes

| Setting | `config.plist` | `config-install.plist` | Why |
|---|---|---|---|
| `boot-args` | `keepsyms=1` | `-v keepsyms=1 debug=0x100` | Verbose boot, and a kernel panic stays on screen instead of rebooting |
| `Misc > Boot > ShowPicker` / `Timeout` | `false` / `5` | `true` / `0` | The picker always shows and waits for you |
| `Misc > Boot > PickerMode` | `External` (OpenCanopy) | `Builtin` | Text picker with no dependency on GUI resources |
| `Misc > Boot > HideAuxiliary` | `true` | `false` | Shows macOS Recovery, Reset NVRAM and the tools |
| `Misc > Debug > Target` | `3` | `67` | Also writes the OpenCore log to the EFI partition |
| `Misc > Debug > AppleDebug` / `ApplePanic` | `false` | `true` | Saves boot.efi and kernel panic logs |
| `Misc > Security > SecureBootModel` | `Default` | `Disabled` | Rules out Apple Secure Boot as a cause of failure |
| `UEFI > Audio > AudioSupport`, `AudioDxe.efi`, `OpenCanopy.efi` | enabled | disabled | Fewer drivers in the early boot |
| `Misc > Tools` | none | UEFI Shell, ControlMsrE2 | Troubleshooting from the picker |

### Steps

1. **Create the installer USB drive** (16 GB or larger).
   - On a Mac: download *Install macOS Ventura* from the App Store and run
     `sudo /Applications/Install\ macOS\ Ventura.app/Contents/Resources/createinstallmedia --volume /Volumes/USB`
   - On Windows or Linux: use `macrecovery.py` from the OpenCore package (`Utilities/macrecovery`):
     `python3 macrecovery.py -b Mac-4B682C642B45593E -m 00000000000000000 download`
     and copy the resulting `com.apple.recovery.boot` folder to the root of a FAT32 USB drive. A recovery installer downloads macOS during setup, so you need Ethernet or Wi-Fi; both work in the installer.
2. **Copy the `EFI` folder** to the EFI partition of the USB drive (the root of the FAT32 drive for a recovery installer).
3. **Switch to the installation profile.** In `EFI/OC` on the USB drive:
   - rename `config.plist` to `config-boot.plist`
   - rename `config-install.plist` to `config.plist`
4. Apply the [BIOS settings](#bios-settings), insert the USB drive and press `F12` (or `Fn+F12`) at power-on to open the boot menu. Select the USB drive.
5. In the OpenCore picker, choose the installer (*Install macOS Ventura* or *macOS Base System*). In the installer, open *Disk Utility* and erase the internal drive as **APFS** with the **GUID Partition Map** scheme, then install.
6. The installer reboots several times. Each time, select **macOS Installer** in the picker until it disappears, then select your drive.

### After the installation

1. Mount the EFI partition of the internal drive:
   `sudo diskutil mount disk0s1` (check the identifier with `diskutil list`)
2. Copy the `EFI` folder from the USB drive to the internal EFI partition.
3. In `EFI/OC` on the internal drive, **restore the daily profile**:
   - delete `config.plist` (the installation profile)
   - rename `config-boot.plist` to `config.plist`
   Keep `config-install.plist` for future troubleshooting.
4. Generate your SMBIOS and set `ROM` (see [Before you boot](#before-you-boot)) in the new `config.plist`.
5. Reboot without the USB drive. `boot-args` is rewritten by OpenCore on every boot (`NVRAM > Delete`), so the verbose flags from the installation profile do not persist. If anything looks off, hold `Option` at power-on and select **Reset NVRAM** once.

### Debugging a failed installation

With `config-install.plist` active:

- **Verbose output**: the last lines before a hang or reboot show where it failed. Take a photo of the screen.
- **OpenCore log**: `opencore-YYYY-MM-DD-HHMMSS.txt` is written to the root of the EFI partition you booted from. For a more detailed log, replace `OpenCore.efi`, `BOOTx64.efi` and the drivers with the **DEBUG** build of OpenCore 1.0.8.
- **Kernel panic**: with `debug=0x100` the panic stays on screen. After a successful boot, the full report is in *Console > Crash Reports* or `/Library/Logs/DiagnosticReports`.
- **CFG Lock**: select **ControlMsrE2** in the picker to check whether MSR `0xE2` is locked.

| Symptom | Likely cause and fix |
|---|---|
| Stops at `EndRandomSeed` or right after selecting the installer | Usually a changed config. Use the files from this repository unchanged, check that the installation profile is named `config.plist`, and that the BIOS uses UEFI with Secure Boot disabled. |
| `Still waiting for root device` | The installer USB is not seen. Use a different USB port (preferably a USB 2.0 port) or a different drive. |
| Black screen after the verbose text ends | Graphics initialization. Close and reopen the lid or connect HDMI. If it persists, add `-igfxvesa` to `boot-args` to finish the installation without acceleration, then remove it. |
| Trackpad does not respond in the installer | Use a USB mouse. The I2C trackpad driver may load late in the installer. |
| Reboots in a loop during the second stage | Keep selecting **macOS Installer** in the picker, not the drive name, until that entry disappears. |

## Audio

Audio uses **AppleALC layout-id 21**, made for this laptop:

- PinConfig taken from the B330 BIOS default pin configuration (speaker `0x14`, headphone `0x21`, internal mic `0x12`, headset mic `0x19`)
- Speaker SoftwareDSP removed. The EQ inherited from the earlier layout attenuated the internal speakers.
- Verified with `alc-verb`: DAC `0x02`/`0x03` at `0x57` (0 dB) and EAPD enabled on the speaker pin

Layout 21 was submitted upstream to [acidanthera/AppleALC](https://github.com/acidanthera/AppleALC). Until it is part of an official release, **keep the AppleALC.kext included in this EFI**. An official AppleALC without layout 21 will fall back to a different layout and audio will not work as expected.

The Linux codec dump is in [`Dumps/ALC230/Lenovo B330-15IKBR.txt`](https://github.com/maiconjs/AppleALC/tree/alc230-b330-layout21/Dumps/ALC230) on the AppleALC fork.

## Post-install

**Enable TRIM** (third-party SATA SSDs only, run **once**):

```
sudo trimforce enable
```

The setting is permanent and survives reboots and macOS updates. From then on macOS trims the free space of the APFS container automatically at every boot and sends TRIM as files are deleted, so there is nothing to repeat periodically. To confirm, check that *System Information > SATA* shows `TRIM Support: Yes`, or run:

```
system_profiler SPSerialATADataType | grep TRIM
```

**Battery conservation mode** (stops charging at about 60%, recommended if you use the laptop plugged in most of the time):

```
ioio -s IdeaVPC ConservationMode true
ioreg -n IdeaVPC -r | grep ConservationMode
```

`ioio` can be built from [RehabMan/OS-X-ioio](https://github.com/RehabMan/OS-X-ioio). The YogaSMC preference pane can also toggle it, but on Ventura it may load blank (see [YogaSMC#189](https://github.com/zhen-zen/YogaSMC/issues/189)).

**Disable hibernation** (Hackintosh sleep is more reliable without it):

```
sudo pmset -a hibernatemode 0 standby 0 autopoweroff 0
sudo rm -f /var/vm/sleepimage
```

**Hide the boot picker**: the picker is hidden by default. Hold `Option` or `Esc` at power-on to show it. Set `Misc > Boot > ShowPicker` to `true` if you want it every time.

## Other CPU variants

The B330-15IKBR was also sold with 8th-gen Kaby Lake-R CPUs (for example i5-8250U, with UHD 620).

- `CPUFriendDataProvider.kext` is specific to the **i3-7020U**. On any other CPU, disable `CPUFriend.kext` and `CPUFriendDataProvider.kext` or generate your own data with [CPUFriendFriend](https://github.com/corpnewt/CPUFriendFriend).
- The framebuffer (`AAPL,ig-platform-id 0x591B0000`, HDMI patched on connector 1) generally works for UHD 620 as well, but this was only tested on HD 620.

## Troubleshooting

**No HDMI audio on a TV or AV receiver**
Some receivers advertise an interlaced mode (1080i) as the preferred timing in their EDID. macOS then picks 1080i, and HDMI audio does not work in that mode. Select a 1080p 60 Hz mode in *System Settings > Displays*. If it does not stick, install a display override with the 1080p mode as the preferred timing under `/Library/Displays/Contents/Resources/Overrides/`.

**Trackpad stops working**
Disable `VoodooRMI.kext`, its `VoodooInput.kext` and `RMII2C.kext`, then enable `VoodooI2C.kext/Contents/PlugIns/VoodooInput.kext` and `VoodooI2CHID.kext`. The trackpad will fall back to VoodooI2CHID.

**Boot hangs with the chime enabled**
Disable `UEFI > Audio > AudioSupport` and the `AudioDxe.efi` driver.

**Instant wake after sleep**
Make sure `SSDT-GPRW.aml` and the `GPRW to XPRW` ACPI rename are both enabled.

## Credits

- [Acidanthera](https://github.com/acidanthera) for OpenCore, Lilu, VirtualSMC, WhateverGreen, AppleALC and CPUFriend
- [VoodooI2C](https://github.com/VoodooI2C), [VoodooSMBus/VoodooRMI](https://github.com/VoodooSMBus/VoodooRMI) and [VoodooPS2](https://github.com/acidanthera/VoodooPS2)
- [zhen-zen](https://github.com/zhen-zen/YogaSMC) for YogaSMC
- [1Revenger1](https://github.com/1Revenger1/ECEnabler) for ECEnabler
- [USBToolBox](https://github.com/USBToolBox)
- [Mieze](https://github.com/Mieze/RTL8111_driver_for_OS_X) for RealtekRTL8111
- [Dortania](https://dortania.github.io/OpenCore-Install-Guide/) for the OpenCore Install Guide

Each bundled component keeps its own license.

## Disclaimer

Provided as-is, without warranty. Back up your data and keep a working USB EFI before replacing the one on your disk.
