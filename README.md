# AlderLake-IrisXe-OpenCore (macOS Tahoe & Ventura on Lenovo IdeaPad 3 15IAU7)

![macOS Tahoe](https://img.shields.io/badge/macOS-Tahoe%2026.7.1-blue?logo=apple)
![macOS Ventura](https://img.shields.io/badge/macOS-Ventura%2013.7.8-informational?logo=apple)
![OpenCore](https://img.shields.io/badge/OpenCore-1.0.8-black)
![Alder Lake](https://img.shields.io/badge/Intel-Alder%20Lake%2012th%20Gen-informational)
![Iris Xe](https://img.shields.io/badge/Iris%20Xe-Metal%20Acceleration-success)
![Audio](https://img.shields.io/badge/Audio-ALC257%20%2B%20SimpleHDA-success)
![Bluetooth](https://img.shields.io/badge/RTL8852BE-Wi--Fi%206%20%2B%20BT-success)
![License](https://img.shields.io/badge/License-BSD%203--Clause-green)

Unified **OpenCore 1.0.8** dual-macOS + Windows 11 EFI configuration for the **Lenovo IdeaPad 3 15IAU7 (Type 82RK, Intel 12th Gen Alder Lake-U)** supporting **macOS Tahoe 26.7.1 (`25G241` / Darwin `25.6.0`)** and **macOS Ventura 13.7.8 (`22H730` / Darwin `22.6.0`)** from a single shared EFI via `MinKernel`/`MaxKernel` gating.

---

## Verified Hardware Specifications

| Component | Hardware Details | Status | Solution / Notes |
| :--- | :--- | :--- | :--- |
| **Model** | Lenovo IdeaPad 3 15IAU7 (Type 82RK) | Supported | OpenCore `1.0.8` (`MacBookPro16,2` SMBIOS) |
| **CPU** | Intel Core i5-1235U (10C: 2P + 8E, 12T) | Working | `SSDT-PLUG-ALT.aml`, `CpuTscSync.kext` (`1.1.2`), `CpuTopologyRebuild.kext` (`<= 23.99.99`) |
| **RAM** | 16 GB DDR4 3200 MHz | Working | Native dual-channel |
| **Storage** | Intel SSDPEKNW512GZL 512 GB PCIe NVMe | Working | `NVMeFix.kext` (`1.1.3`) |
| **Integrated GPU** | Intel Iris Xe Graphics (`0x46A8`, Alder Lake-UP3 GT2) | Metal Acceleration | `WhateverXe.kext` (`0.7.0`, `-allow3d -igfxdvmt -wex-edp-keep`) + `WhateverXe-Graphics` (`/Library/Extensions`) |
| **Display & Backlight** | 15.6" FHD (1920x1080) 60 Hz eDP | Working (`F5`/`F6`) | `SSDT-PNLF.aml` + `BrightnessKeys.kext` (`1.0.4`, `>= 25.0.0`) |
| **Audio Codec** | Realtek ALC257 (`0x10ec0257`, `alcid=100`) | Speakers, Mic & Jack | `AppleALC.kext` (`1.9.8`) + `SimpleHDA.kext` (`1.0.1` on Tahoe) |
| **Wi-Fi 6** | Realtek RTL8852BE PCIe (`0x10ec:0xb852`) | Working | `AirportRTW.kext` (`<= 23.99.99`) / `AirportRTW-Tahoe.kext` (`>= 25.0.0`) |
| **Bluetooth** | Realtek RTL8852BE USB (`0x0bda:0x4853`, `rtl8852bu`) | Working | `RealtekBluetoothFirmware.kext` (`1.0.2`) + `BlueToolFixup.kext` (`2.7.3`) + `-btlfxboardid` + NVRAM BT props |
| **Trackpad** | ELAN06FA I2C Precision Touchpad (`pci8086,51e9`) | Full Gestures & Tap | `VoodooI2C.kext` (`2.9.1`) + `VoodooI2CHID.kext` (`-vi2c-force-polling`) |
| **Keyboard** | Standard PS/2 Keyboard | Keys & Brightness | `VoodooPS2Controller.kext` (`2.3.8`) |
| **Webcam** | SunplusIT Integrated Camera (`0x5986:0x2155`) | Working | Native UVC via `USBToolBox.kext` + `UTBMap.kext` |
| **Battery & Power** | Celxpert `L20C3PF0` (`BAT0`) + `ADP0` AC | Working | `SSDT-BAT0.aml` (`BAT0._STA -> XSTA`) + `ECEnabler.kext` (`1.0.6`) + `SMCBatteryManager.kext` (`1.3.8`) |
| **ACPI Controllers** | Intel PCH MCHC, SMBus & EC | Working | `SSDT-AWAC.aml`, `SSDT-EC-USBX.aml`, `SSDT-MCHC.aml`, `SSDT-SBUS.aml` |
| **USB Ports** | 1x USB 2.0, 1x USB 3.2 Gen 1, 1x USB-C 3.2 Gen 1 | Working | `USBToolBox.kext` (`1.2.0`) + `UTBMap.kext` (`1.1`) |

---

## Tahoe vs. Ventura Compatibility & Highlights

- **Iris Xe Acceleration (`0x46A8`):**
  - OpenCore loads `WhateverXe.kext` (`0.7.0`) from EFI (`-allow3d -disablegfxfirmware -igfxdvmt -wex-edp-keep`), paired with `WhateverXe-Graphics` installed in `/Library/Extensions/` (`org.b00t0x.WhateverXe.Tahoe` `0.4.1` -> `AppleIntelTGLCompat.kext` `0.8.0` + `AppleIntelTGLGraphics*` on Tahoe; `0.3.2` on Ventura).
  - Smooth window server animations, video decoding, and verified glitch-free 1080p YouTube playback on internal speakers.
- **Tahoe Bluetooth (`RTL8852BE` `0x0bda:0x4853`):**
  - Requires `RealtekBluetoothFirmware.kext` + `BlueToolFixup.kext` together with `-btlfxboardid` in `boot-args`, plus NVRAM `bluetoothExternalDongleFailed` (`<00>`) and 14-byte zeroed `bluetoothInternalControllerInfo` under `7C436110-AB2A-4BBB-A880-FE41995C9F82`.
- **IdeaPad 3 `BAT0` Battery & AC Adapter Fix (`SSDT-BAT0.aml`):**
  - OEM `BAT0._STA` reads `Local0` uninitialized when `ECAV == 0` during early ACPI probing (`AE_AML_UNINITIALIZED_LOCAL`), breaking battery state detection unless patched.
  - `SSDT-BAT0.aml` + the `BAT0._STA -> XSTA` ACPI rename guards `_OSI ("Darwin")` cleanly while calling `XSTA ()` on Windows/Linux.
- **Audio (`ALC257`, `alcid=100` + `SimpleHDA.kext` on Tahoe):**
  - Works out of the box for internal speakers, mic, and 3.5mm jack (`alcid=100`, `alcdelay=1000`, `shda-lang=en`).
  - *Note:* With `alcid=100` (which separates Speaker and Headphone path groups), macOS Tahoe can sometimes default initial output to **Built-in Headphone**. Simply switch output to **Built-in Speaker** in Control Center / Sound Settings (or test `alcid=99` if you prefer a single combined output device).
- **Sleep / Wake Note:**
  - Verified on Ventura; S3/darkwake behavior on Tahoe is still being validated (this snapshot keeps `HibernateMode = None` and debug logging enabled).

---

## OpenCore EFI Structure

```text
EFI/
├── BOOT/
│   └── BOOTx64.efi
└── OC/
    ├── ACPI/
    │   ├── SSDT-AWAC.aml / .dsl      # RTC/AWAC clock fix for 12th Gen
    │   ├── SSDT-BAT0.aml / .dsl      # IdeaPad 3 15IAU7 BAT0._STA ECAV==0 fix
    │   ├── SSDT-EC-USBX.aml / .dsl   # Embedded Controller & USB power properties
    │   ├── SSDT-MCHC.aml             # Memory Controller Hub fix
    │   ├── SSDT-PLUG-ALT.aml / .dsl  # Alder Lake CPU topology
    │   ├── SSDT-PNLF.aml / .dsl      # Internal eDP backlight control
    │   ├── SSDT-RHUB.aml / .dsl      # Optional XHC root hub reset (Disabled)
    │   ├── SSDT-SBUS.aml             # System Management Bus fix
    │   └── SSDT-XOSI.aml             # Optional _OSI table (Disabled)
    ├── Drivers/
    │   ├── OpenHfsPlus.efi           # HFS+ filesystem driver
    │   ├── OpenRuntime.efi           # NVRAM & runtime services (OC 1.0.8)
    │   └── ResetNvramEntry.efi       # NVRAM reset entry
    ├── Kexts/
    │   ├── Lilu.kext                 # 1.7.3
    │   ├── VirtualSMC.kext           # 1.3.9
    │   ├── WhateverXe.kext           # 0.7.0 (Iris Xe eDP-keep)
    │   ├── WhateverGreen.kext        # Disabled fallback (24.0.0+)
    │   ├── NVMeFix.kext              # 1.1.3
    │   ├── ECEnabler.kext            # 1.0.6
    │   ├── SMCBatteryManager.kext    # 1.3.8
    │   ├── RestrictEvents.kext       # 1.0.9 (<= 23.99.99)
    │   ├── CpuTscSync.kext           # 1.1.2
    │   ├── CpuTopologyRebuild.kext   # 2.0.2 (20.0.0–23.99.99)
    │   ├── AMFIPass.kext             # 1.4.1
    │   ├── AirportRTW.kext           # 2.3.1 (<= 23.99.99)
    │   ├── AirportRTW-Tahoe.kext     # 2.3.1 (25.0.0+)
    │   ├── VoodooPS2Controller.kext  # 2.3.8
    │   ├── VoodooI2C.kext            # 2.9.1
    │   ├── VoodooI2CHID.kext         # 1.0 (-vi2c-force-polling)
    │   ├── AppleALC.kext             # 1.9.8
    │   ├── USBToolBox.kext           # 1.2.0
    │   ├── UTBMap.kext               # 1.1 (IdeaPad 3 15IAU7 USB map)
    │   ├── SimpleHDA.kext            # 1.0.1 (25.0.0–25.99.99)
    │   ├── BrightnessKeys.kext       # 1.0.4 (25.0.0+)
    │   ├── BlueToolFixup.kext        # 2.7.3 (25.0.0+)
    │   └── RealtekBluetoothFirmware.kext # 1.0.2 (25.0.0+)
    ├── Tools/
    │   └── OpenShell.efi
    ├── OpenCore.efi                  # OpenCore 1.0.8
    └── config.plist                  # Validated with ocvalidate 1.0.8 (0 issues)
```

---

## Installation & Setup Instructions

### 1. BIOS Settings
Enter Lenovo BIOS setup (`F2` or `Fn+F2` on boot) and configure:
- **Configuration -> Storage Controller Mode:** `AHCI` (disable Intel VMD / RST)
- **Security -> Secure Boot:** `Disabled`
- **Boot -> Fast Boot:** `Disabled`
- **Boot -> USB Boot:** `Enabled`

### 2. Generate Your Own SMBIOS (`config.plist`)
For privacy and Apple Services (iMessage / FaceTime / iCloud), serial numbers in `EFI/OC/config.plist` are sanitized (`GENERATE_YOUR_OWN` / zeroed UUID / dummy ROM `001122334455`). Generate your own before signing in:

1. Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS).
2. Run `GenSMBIOS` and generate values for **`MacBookPro16,2`**.
3. Open `EFI/OC/config.plist` in [ProperTree](https://github.com/corpnewt/ProperTree) and update `PlatformInfo -> Generic`:
   - `SystemProductName`: `MacBookPro16,2`
   - `SystemSerialNumber`: *(from GenSMBIOS)*
   - `MLB`: *(from GenSMBIOS)*
   - `SystemUUID`: *(from GenSMBIOS)*
   - `ROM`: *(6-byte MAC address or generated value)*

### 3. Install `WhateverXe-Graphics` Package in macOS
`WhateverXe` uses companion kexts in `/Library/Extensions/` (requires `csr-active-config = 130a0000` + `AMFIPass.kext`, both preconfigured in `config.plist`):
1. Boot macOS using this EFI (initially in software/VESA mode if fresh install).
2. Install the **WhateverXe Graphics** `.pkg` matching your macOS version (**Tahoe** `0.4.1+` or **Ventura** `0.3.2+`), rebuild kext cache / reboot, and Metal acceleration + 1536 MB VRAM will activate.

### 4. Dual-Boot Deployment on Internal SSD
1. Mount your EFI System Partition (or create/use a dedicated OpenCore ESP if dual-booting Windows 11).
2. Copy `EFI/BOOT/BOOTx64.efi` and `EFI/OC/` to the EFI partition without modifying `EFI/Microsoft/`.
3. Once your install is stable, you may optionally remove `-v` / `-btlfxdbg` from `boot-args` and set `Misc -> Debug -> Target = 0` (`AppleDebug = False`) for silent production boots.

---

## License
This project is licensed under the [BSD 3-Clause License](LICENSE).

## Credits & Acknowledgements
- [Acidanthera](https://github.com/acidanthera) for OpenCorePkg, Lilu, AppleALC, VirtualSMC, BrcmPatchRAM (`BlueToolFixup`), BrightnessKeys, and VoodooPS2.
- [b00t0x](https://github.com/b00t0x) for `WhateverXe` and `CpuTopologyRebuild`.
- [nickk19](https://github.com/nickk19) for Alder Lake eDP research and PR #43.
- [williambj1](https://github.com/williambj1/AirportRTW) / `senmiko` for `AirportRTW` Realtek RTL8852BE Wi-Fi 6 support.
- [laobamac](https://github.com/laobamac) for `SimpleHDA` on macOS Tahoe.
- [thegwchr](https://github.com/thegwchr) / contributors for `RealtekBluetoothFirmware`.
- [1Revenger1](https://github.com/1Revenger1/ECEnabler) for `ECEnabler`.
- [VoodooI2C Team](https://github.com/VoodooI2C/VoodooI2C) for `VoodooI2C` & `VoodooI2CHID`.
- [dhinakg](https://github.com/dhinakg) for `AMFIPass` & `USBToolBox`.
- [corpnewt](https://github.com/corpnewt) for `GenSMBIOS` & `ProperTree`.
