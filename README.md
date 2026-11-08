# AlderLake-IrisXe-OpenCore (macOS Ventura on Lenovo IdeaPad 3 15IAU7)

![macOS Ventura](https://img.shields.io/badge/macOS-Ventura%2013.7.8-blue?logo=apple)
![OpenCore](https://img.shields.io/badge/OpenCore-0.8.5-black)
![Alder Lake](https://img.shields.io/badge/Intel-Alder%20Lake%2012th%20Gen-informational)
![Metal 2](https://img.shields.io/badge/Iris%20Xe%20Metal%202-1536%20MB%20VRAM-success)
![Dual Boot](https://img.shields.io/badge/Dual%20Boot-Windows%2011%20%2B%20macOS-success)
![Audio](https://img.shields.io/badge/Audio-ALC257%20Layout%20100-success)
![License](https://img.shields.io/badge/License-BSD%203--Clause-green)

A production-grade, tested, open-source dual-boot OpenCore EFI configuration for running **macOS Ventura 13.7.8 (22H730)** on the **Lenovo IdeaPad 3 15IAU7** (Intel 12th Gen Alder Lake-U) with **Full Metal 2 GPU Acceleration (1536 MB VRAM)**, **Realtek ALC257 Audio (Layout 100)**, **Realtek Wi-Fi 6**, and **I2C Multitouch Trackpad** alongside **Windows 11** on the internal NVMe SSD.

---

## Hardware Specifications

| Component | Hardware Details | macOS Status | Kext / Solution |
| :--- | :--- | :--- | :--- |
| **Model** | Lenovo IdeaPad 3 15IAU7 (Type 82RK) | Fully Supported | OpenCore 0.8.5 |
| **CPU** | Intel® Core™ i5-1235U (10 Cores: 2P + 8E, 12 Threads) | 100% Working | `CpuTopologyRebuild.kext`, `SSDT-PLUG-ALT` |
| **RAM** | 16 GB DDR4 3200 MHz | 100% Working | Dual-channel native |
| **Storage** | Intel SSDPEKNW512GZL 512 GB PCIe NVMe M.2 SSD | 100% Working | `NVMeFix.kext` |
| **Integrated GPU** | Intel® Iris® Xe Graphics (`0x46A8`, Alder Lake-UP3 GT2) | **100% Metal 2 (1536 MB VRAM)** | Patched `WhateverXe.kext` (`-wex-edp-keep`) + `AppleIntelTGLCompat` |
| **Display** | 15.6" Full HD (1920x1080) 60 Hz Anti-glare IPS / TN | 1080p FHD Native | Internal eDP panel |
| **Audio Codec** | Realtek ALC257 (`0x10ec0257`, HDA `pci8086,51c8`) | Speakers, Mic & 3.5mm Jack | `AppleALC.kext` Layout 100 (`alcid=100`, `alcdelay=1000`) |
| **ACPI Controllers** | Intel PCH MCHC & SMBus | Native DMA Arbitration | `SSDT-MCHC.aml`, `SSDT-SBUS.aml` |
| **Wi-Fi** | Realtek RTL8852BE Wi-Fi 6 802.11ax PCIe (`pci10ec,b852`) | 100% Working | `AirportRTW.kext` |
| **Trackpad** | ELAN06FA I2C Precision Touchpad (`pci8086,51e9`) | Full Gestures & Tap | `VoodooI2C.kext` + `VoodooI2CHID` (`-vi2c-force-polling`) |
| **Keyboard** | Standard PS/2 Keyboard | Volume, Brightness & Fn Keys | `VoodooPS2Controller.kext` |
| **Battery & Power** | Embedded Controller Battery & Power Adapter | 100% Working | `ECEnabler.kext` |
| **USB Ports** | 1x USB 2.0, 1x USB 3.2 Gen 1, 1x USB-C 3.2 Gen 1 | 100% Working | `USBToolBox.kext` + `UTBMap.kext` |

---

## What Works

- [x] **Full Metal 2 GPU Acceleration:** Intel Iris Xe (`0x46A8`) with 1536 MB VRAM, smooth window animations, translucent Dock, QuickTime local video playback, and WebKit/Metal hardware acceleration via patched `WhateverXe` (`-wex-edp-keep`).
- [x] **Audio (Layout 100):** Realtek ALC257 stereo internal speakers, built-in microphone, and 3.5mm combo headphone jack. Layout 100 isolates the headphone and speaker amplifiers into separate path maps, preventing hardware amplifier collisions when plugging in headphones.
- [x] **PCI DMA & ACPI Arbitration:** `SSDT-MCHC` and `SSDT-SBUS` enable macOS native `AppleSMBusController` and `AppleSMBusPCI`, eliminating PCI bus DMA latency spikes on the Intel PCH audio bus.
- [x] **Direct SSD Dual-Boot:** Boots directly from the internal NVMe SSD with OpenCore picker without requiring a USB flash drive. Windows 11 partition and `EFI\Microsoft` remain completely untouched and bootable.
- [x] **Wi-Fi 6:** Realtek RTL8852BE runs at full native speed on 2.4 GHz and 5 GHz networks via `AirportRTW.kext`.
- [x] **Trackpad & Gestures:** `ELAN06FA` I2C touchpad operates with smooth cursor tracking, physical clicking, tap-to-click, and multi-touch gestures via `-vi2c-force-polling`.
- [x] **Display Resolution:** Full HD 1920x1080 60 Hz crisp native display on internal eDP panel.
- [x] **Battery & Charging:** Accurate battery percentage, charging status, and power adapter indicators via `ECEnabler.kext`.
- [x] **Sleep & Wake:** Proper power states and lid wake.

---

## OpenCore EFI Structure

```text
EFI/
├── BOOT/
│   └── BOOTx64.efi
└── OC/
    ├── ACPI/
    │   ├── SSDT-AWAC.aml        # RTC/AWAC clock fix for 12th Gen
    │   ├── SSDT-EC-USBX.aml     # Embedded Controller & USB power
    │   ├── SSDT-MCHC.aml        # Memory Controller Hub fix (PCI DMA bus)
    │   ├── SSDT-PLUG-ALT.aml    # Alder Lake CPU power management
    │   ├── SSDT-PNLF.aml        # Backlight support
    │   ├── SSDT-SBUS.aml        # System Management Bus fix (AppleSMBusPCI)
    │   └── SSDT-XOSI.aml        # Windows OSI emulation
    ├── Drivers/
    │   ├── OpenCanopy.efi       # Graphical boot picker
    │   ├── OpenHfsPlus.efi      # HFS+ filesystem driver
    │   ├── OpenRuntime.efi      # NVRAM & runtime services
    │   └── ResetNvramEntry.efi  # NVRAM reset utility
    ├── Kexts/
    │   ├── Lilu.kext
    │   ├── VirtualSMC.kext
    │   ├── SMCProcessor.kext
    │   ├── ECEnabler.kext
    │   ├── CpuTopologyRebuild.kext
    │   ├── CpuTscSync.kext
    │   ├── NVMeFix.kext
    │   ├── AirportRTW.kext       # Realtek RTL8852BE Wi-Fi 6 driver
    │   ├── AppleALC.kext         # Audio codec driver (layout 100)
    │   ├── VoodooPS2Controller.kext
    │   ├── VoodooI2C.kext        # Intel LPSS I2C driver (Alder Lake patched)
    │   ├── VoodooI2CHID.kext     # ELAN06FA Precision Touchpad driver
    │   ├── USBToolBox.kext
    │   ├── UTBMap.kext           # Custom USB port map
    │   ├── RestrictEvents.kext
    │   ├── AMFIPass.kext
    │   └── WhateverXe.kext       # Patched WhateverXe with eDP-keep
    └── config.plist
```

---

## Installation & Setup Instructions

### 1. BIOS Settings
Before booting macOS, enter your BIOS setup (press `F2` or `Fn+F2` on startup) and configure:
- **Configuration -> Storage Controller / SATA Mode:** AHCI (Disable Intel VMD / RST)
- **Security -> Secure Boot:** Disabled
- **Boot -> Fast Boot:** Disabled
- **Boot -> USB Boot:** Enabled

### 2. Generate New Serial Numbers (SMBIOS)
For privacy and iCloud/iMessage functionality, serial numbers in the shared `config.plist` are replaced with `GENERATE_YOUR_OWN`. You must generate your own SMBIOS credentials before signing into Apple services:

1. Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS).
2. Run `GenSMBIOS` and select option `Generate SMBIOS`.
3. Enter model: `MacBookPro16,2`.
4. Open `EFI/OC/config.plist` in [ProperTree](https://github.com/corpnewt/ProperTree) or a text editor.
5. In `PlatformInfo -> Generic`, fill in:
   - `SystemProductName`: `MacBookPro16,2`
   - `SystemSerialNumber`: *(from GenSMBIOS)*
   - `MLB`: *(from GenSMBIOS)*
   - `SystemUUID`: *(from GenSMBIOS)*

### 3. Dual-Boot Deployment on Internal SSD
To boot directly from your SSD without keeping a USB drive plugged in:
1. In Windows or macOS, mount your internal EFI partition (`SYSTEM_DRV` / `disk0s1`).
2. Copy the `EFI/OC` folder and `EFI/BOOT/BOOTx64.efi` into the internal EFI partition.
3. Keep the existing `EFI/Microsoft` folder untouched so Windows 11 continues to boot directly.
4. Add OpenCore as a UEFI boot entry via Windows PowerShell:
   ```powershell
   bcdedit /set "{bootmgr}" path \EFI\OC\OpenCore.efi
   ```

---

## Media & Video Playback Notes

- **Display Resolution:** The laptop panel is native Full HD (1920x1080) at 60 Hz.
- **Hardware Codec Support:** macOS VideoToolbox provides native hardware acceleration for **H.264** and **HEVC (H.265)** on Intel Iris Xe graphics.
- **Recommended Playback:** For zero-buffering, stutter-free 60fps playback, stream video in native 1080p FHD in Safari or Chrome.

---

## Contributing & Open Source

Contributions, issue reports, and pull requests are welcome! If you have optimizations for battery life, power management, or new macOS versions (e.g. Sonoma / Sequoia), feel free to open a PR.

---

## License
This project is licensed under the [BSD 3-Clause License](LICENSE).

## Credits & Acknowledgements
- [Acidanthera](https://github.com/acidanthera) for OpenCorePkg, Lilu, AppleALC, VirtualSMC, and VoodooPS2.
- [VoodooI2C Team](https://github.com/VoodooI2C/VoodooI2C) for I2C input support.
- [b00t0x](https://github.com/b00t0x) for WhateverXe.
- [nickk19](https://github.com/nickk19) for Alder Lake eDP research and PR #43.
- [AirportRTW](https://github.com/williambj1/AirportRTW) for Realtek Wi-Fi 6 macOS support.
