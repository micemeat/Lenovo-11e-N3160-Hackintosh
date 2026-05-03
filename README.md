# Lenovo 11e Gen 5 (Celeron N3160) - OpenCore Hackintosh EFI

## Hardware
| Component | Spec |
|-----------|------|
| CPU | Intel Celeron N3160 (Braswell, 4C/4T, 1.6GHz / 2.24GHz burst) |
| GPU | Intel HD Graphics 400 (Gen8 LP) |
| Display | 11.6" Touchscreen (1366x768) |
| RAM | 4GB DDR3L |
| Storage | eMMC or M.2 SATA SSD (varies) |
| Wi-Fi | Intel Dual Band Wireless-AC 8260 |
| Bluetooth | Intel 8260 |
| Ethernet | Intel I218-LM |
| Audio | Realtek ALC298 (or ALC283) |
| Trackpad | I2C HID |
| Touchscreen | I2C HID |
| Chipset | Intel SoC (Braswell - on-package PCH) |

## ⚠️ Important: macOS Version Limit

**This build targets macOS 10.15 Catalina as the maximum supported version.**

The Intel HD Graphics 400 (Gen8 LP) in the N3160 does **NOT** have Metal support, which is required for macOS 11 Big Sur and later. There is no workaround for this — it's a hardware limitation.

**Supported versions:**
| Version | Support |
|---------|---------|
| macOS 10.11 El Capitan | ✅ |
| macOS 10.12 Sierra | ✅ |
| macOS 10.13 High Sierra | ✅ |
| macOS 10.14 Mojave | ✅ |
| macOS 10.15 Catalina | ✅ (Recommended) |
| macOS 11 Big Sur+ | ❌ (No Metal on HD 400) |

## What Works
- ✅ CPU power management (AppleLPC native on Braswell)
- ✅ Intel HD Graphics 400 with QE/CI
- ✅ Internal display at native resolution
- ✅ Backlight control (SSDT-PNLF)
- ✅ Audio (Realtek ALC298, layout-id=28)
- ✅ Ethernet (Intel I218 via IntelMausi)
- ✅ Wi-Fi (Intel AC 8260 via AirportItlwm — native Wi-Fi menu)
- ✅ Bluetooth (IntelBluetoothFirmware + IntelBluetoothInjector + IntelBTPatcher)
- ✅ Keyboard (VoodooPS2Controller)
- ✅ Touchscreen/Trackpad (VoodooI2C + VoodooI2CHID)
- ✅ Battery status (SMCBatteryManager)
- ✅ USB ports
- ✅ Sleep/Wake

## What Doesn't Work
- ❌ macOS 11+ (no Metal support on HD 400 iGPU)
- ❌ HDMI — may need additional framebuffer patching
- ❌ Continuity features (AirDrop, Handoff) — Intel WiFi limitation
- ❌ Sidecar (requires Metal, not available on HD 400)
- ❌ iServices — need valid SMBIOS serials (regenerate before use!)

## OpenCore Version
- **OpenCore 1.0.7** (March 2026)

## Key Differences from Kaby Lake Build (i3-7100U)
This EFI is specifically built for the **Braswell** Celeron N3160. Do NOT use the Kaby Lake EFI on this machine:

| Setting | Kaby Lake (i3-7100U) | Braswell (N3160) |
|---------|----------------------|-------------------|
| SMBIOS | MacBookPro14,1 | MacBookAir7,1 |
| ig-platform-id | 0x59160000 (HD 620) | 0x19160000 (HD 400) |
| SSDT-PLUG | ✅ Required | ❌ Not needed |
| SSDT-PMC | ✅ Required | ❌ Not needed |
| CPUFriend | ✅ Useful | ❌ Not useful |
| NvmExpressDxe | ❌ Not needed | ✅ Required |
| WiFi kext | itlwm + HeliPort | AirportItlwm (native) |
| BT Injector | ❌ Not on Monterey | ✅ IntelBluetoothInjector |
| Max macOS | Ventura/Sonoma | Catalina |

## Kexts Included
| Kext | Version | Purpose |
|------|---------|---------|
| Lilu | 1.7.2 | Kernel patcher (required by all) |
| VirtualSMC | 1.3.7 | SMC emulator (required) |
| SMCProcessor | 1.3.7 | CPU sensor |
| SMCBatteryManager | 1.3.7 | Battery status |
| SMCSuperIO | 1.3.7 | Fan sensor |
| WhateverGreen | 1.6.9 | GPU patches for HD 400 |
| AppleALC | 1.9.3 | Audio for ALC298 |
| IntelMausi | 1.0.8 | Ethernet I218 |
| AirportItlwm | 2.3.0 | WiFi Intel AC 8260 (native, Catalina build) |
| IntelBluetoothFirmware | 2.4.0 | BT firmware upload |
| IntelBluetoothInjector | 2.4.0 | BT injector (Catalina and below) |
| IntelBTPatcher | 2.4.0 | BT patches |
| VoodooPS2Controller | 2.3.7 | Keyboard input |
| VoodooI2C | 2.9.1 | I2C bus driver |
| VoodooI2CHID | 2.9.1 | I2C HID touchscreen |
| RestrictEvents | 1.1.5 | SMBIOS restriction patches |

## SSDTs Included
| SSDT | Purpose |
|------|---------|
| SSDT-EC-USBX | Embedded controller + USB power properties |
| SSDT-PNLF | Backlight control for internal display |

## Drivers Included
| Driver | Purpose |
|--------|---------|
| OpenHfsPlus.efi | HFS+ filesystem |
| OpenRuntime.efi | Runtime services (required) |
| OpenVariableRuntimeDxe.efi | NVRAM support (required) |
| OpenCanopy.efi | Boot picker GUI |
| ResetNvramEntry.efi | Reset NVRAM utility |
| NvmExpressDxe.efi | NVMe boot for pre-Skylake (Braswell needs this) |

## Installation

### 1. Flash EFI to USB
```bash
# Format USB as FAT32, then copy EFI folder to root
cp -r EFI /Volumes/INSTALLER/
```

### 2. BIOS Settings
Enter BIOS (F2 or Fn+F2 on boot):
- **Secure Boot**: Disabled
- **Boot Mode**: UEFI Only
- **CFG Lock**: Disabled (if option exists; otherwise AppleCpuPmCfgLock quirk handles it)
- **VT-d**: Disabled (or leave enabled with DisableIoMapper quirk)
- **Fast Boot**: Disabled
- **USB Configuration**: XHCI

### 3. Install macOS Catalina
1. Boot from USB, select OpenCore picker
2. Select "Install macOS" from the recovery image
3. Format your storage as APFS (or HFS+ for eMMC)
4. Install macOS Catalina
5. After first boot, install to SSD's EFI

### 4. Post-Install

#### Wi-Fi
AirportItlwm provides **native Wi-Fi** — shows up in the Wi-Fi menu just like a real Mac. No HeliPort needed!

#### Touchscreen (I2C)
VoodooI2C + VoodooI2CHID should work out of the box. If it doesn't:
1. Boot with `-liludbg -v` to see I2C debug output
2. Check `IOReg` or `log show --predicate 'process == "kernel"'` for I2C device info
3. May need a custom SSDT-I2C with your specific device's ACPI path

#### Audio
Layout-id 28 is set in boot-args (`alcid=28`). If no sound:
- For ALC298: Try `alcid=15`, `alcid=65`, or `alcid=92`
- For ALC283: Try `alcid=3`, `alcid=11`, or `alcid=21`
- Check AppleALC supported codecs: https://github.com/acidanthera/AppleALC/wiki

#### HDMI Output
If HDMI doesn't work on HD 400:
1. The HD 400 framebuffer is limited — external display may need specific patches
2. Try adding `framebuffer-con1-enable=01000000` and `framebuffer-con1-type=00080000` to DeviceProperties

#### Battery Status
Should work with SMCBatteryManager. If not showing:
- Add `bat0` to `acpi>add` or create a custom SSDT-BAT

## Boot Args Reference
| Arg | Purpose |
|-----|---------|
| `-v` | Verbose boot (remove once working) |
| `debug=0x100` | Debug kernel on panic |
| `keepsyms=1` | Keep panic symbols |
| `alcid=28` | Audio layout ID for ALC298 |
| `-liludbg` | Lilu debug logging |
| `-wegnoegpu` | Disable external GPU |

## SMBIOS Info
- **Model**: MacBookAir7,1
- **Serial**: C02Q31J2G8MC
- **MLB**: C02721904QXH2VK1C
- **UUID**: A1B2C3D4-E5F6-7890-ABCD-EF1234567890

⚠️ **Regenerate serials before using iMessage!** Use GenSMBIOS or macserial:
```bash
# Use GenSMBIOS Python script
pip install genSMBIOS
genSMBIOS -sp MacBookAir7,1
```

## Performance Notes
The Celeron N3160 is a low-power SoC (6W TDP):
- **4 cores** but very slow (Celeron-class, ~1.6-2.24GHz)
- **4GB RAM** — don't expect miracles, keep apps minimal
- Catalina will run but be sluggish compared to a real Mac
- Recommended: disable visual effects, use Safari over Chrome
- If your storage is eMMC, performance will be very slow — upgrade to M.2 SATA SSD if possible

## Credits
- [Acidanthera](https://github.com/acidanthera) - OpenCore, Lilu, VirtualSMC, WhateverGreen, AppleALC, etc.
- [Dortania](https://github.com/dortania) - OpenCore Install Guide, SSDTs
- [OpenIntelWireless](https://github.com/OpenIntelWireless) - AirportItlwm, IntelBluetoothFirmware
- [VoodooI2C](https://github.com/VoodooI2C) - VoodooI2C touchscreen support