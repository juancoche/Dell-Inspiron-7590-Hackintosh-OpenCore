# Dell Inspiron 7590 / 7591 Hackintosh (OpenCore)

[![OpenCore](https://img.shields.io/badge/OpenCore-1.0.x-blue.svg)](https://github.com/acidanthera/OpenCorePkg)
[![macOS](https://img.shields.io/badge/macOS-Ventura%2013.7.x-success.svg)](https://www.apple.com/macos/ventura/)
[![SMBIOS](https://img.shields.io/badge/SMBIOS-MacBookPro16%2C1-orange.svg)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Completely optimized, stable, and daily-driver-ready OpenCore EFI for the **Dell Inspiron 7590** (and compatible **Inspiron 7591**).

Designed and tested on **macOS 13.7.x Ventura** (the optimal sweet spot for 9th Gen Intel Coffee Lake Refresh laptops with Intel Wi-Fi and Bluetooth).

---

## 💻 Hardware Specifications

| Component | Model / Specs | Status in macOS |
| :--- | :--- | :--- |
| **CPU** | Intel Core i7-9750H (6 cores, 12 threads @ 2.6 GHz - 4.5 GHz) | ✅ Fully Supported (`CPUFriend` LFM 800 MHz) |
| **iGPU** | Intel UHD Graphics 630 (Device ID `0x3e9b`) | ✅ Full Metal 3 acceleration (2048 MB VRAM) |
| **dGPU** | NVIDIA GeForce GTX 1650 (4 GB GDDR5) | ⚡ Completely disabled via SSDT (zero battery drain) |
| **Audio** | Realtek ALC295 / ALC3254 | ✅ Speakers, Headphones, & Jack sensing (Layout `75` + `alc-verbs`) |
| **Wi-Fi** | Intel Wireless-AC 9560 160MHz (CNVi) | ✅ Native AirPort menu (`AirportItlwm`) |
| **Bluetooth** | Intel Bluetooth 5.0 (CNVi `0x8087:0x0aaa`) | ✅ Working (`IntelBTPatcher` + `BlueToolFixup`, AirPods supported) |
| **Trackpad** | Dell Precision Touchpad (I2C-HID) | ✅ Native Apple Multitouch gestures (`VoodooI2C`) |
| **Card Reader** | Realtek USB Card Reader | ✅ Working (Internal USB `HS06` mapped + `RealtekCardReader`) |
| **Sensors & Fans** | Dual Fan Controller + Motherboard Sensors | ✅ Working (`SMCDellSensors` + Macs Fan Control support) |
| **Storage** | M.2 NVMe SSDs (Samsung 970 EVO + Toshiba) | ✅ Supported with power management (`NVMeFix`) |
| **Camera** | Realtek HD Webcam (USB `HS05`) | ✅ Native (FaceTime, Photo Booth, Zoom) |
| **Microphone** | Internal Digital Mic (Intel SST DSP) | ⚠️ Not supported in macOS (use AirPods / 3.5mm jack / USB) |
| **Fingerprint** | Goodix Fingerprint Reader (USB) | ❌ Unsupported on macOS |

---

## ✨ Features & Optimizations Included

1. **Dual Fan & Thermal Management:**
   - Includes **`SMCDellSensors.kext` (v1.3.8)** to expose Dell SMM hardware sensors and dual fans to macOS.
   - Compatible with **Macs Fan Control** for custom fan curves (idle temps around **44 °C – 48 °C**).
2. **Audio Combo Jack Fix:**
   - Configured with `layout-id = 75` and initialized with `VerbStub.kext` (`alc-verbs = <01000000>`, `alc-delay = 500`).
   - Seamless headphone jack insertion sensing and mic switching without popping noises.
3. **MicroSD Card Reader Fix:**
   - The internal MicroSD card reader on the Inspiron 7590 is wired internally to USB port `HS06`.
   - Included in `UTBMap.kext` as `Internal (255)` and supported via `RealtekCardReader.kext`.
4. **Power Management & Battery Life:**
   - Tailored `CPUFriendDataProvider.kext` enabling the i7-9750H to step down to **800 MHz** in idle.
   - NVIDIA dGPU disabled via SSDT to prevent overheating and maximize battery runtime.
   - Sleep configured for pure S3 RAM suspension (`hibernatemode 0`), preventing battery drain when the lid is closed.
5. **Polished OpenCore Boot Picker:**
   - Uses `PickerMode = External` with the **GoldenGate** OpenCanopy theme.
   - Custom macOS (Apple) and Windows 11 icons.
   - Duplicate EFI boot entries hidden via `.contentVisibility`.
6. **iServices & Continuity:**
   - FaceTime, iMessage, iCloud Drive, iCloud Keychain, and iPhone Cellular Calls working.

---

## 🛠️ Required BIOS Settings

Reboot your Dell laptop and tap **F2** to enter BIOS Setup:

* **System Configuration:**
  - **SATA Operation:** Set to **AHCI** (Crucial! macOS will not detect drives if set to RAID).
* **Security:**
  - **Secure Boot:** **Disabled**.
  - **Intel SGX:** **Disabled**.
* **Power Management:**
  - **Wake on AC:** **Disabled**.
  - **USB Wake Support:** **Disabled**.
* **POST Behavior:**
  - **Fastboot:** Set to **Thorough** or **Minimal** (Disable Windows "Fast Startup" in Windows Control Panel).

---

## 🚀 Quick Setup Guide

> [!IMPORTANT]
> **DO NOT boot directly with this EFI without generating your own Serial Numbers.**
> The `config.plist` in this repository has been sanitized (`GENERATE_YOUR_OWN`) to protect privacy and prevent Apple ID account blacklists.

### Step 1: Generate your unique SMBIOS serials
1. Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS).
2. Run `GenSMBIOS.command`.
3. Select option **1** to download MaciASL/MacSerial (if needed).
4. Select option **2** to select your `EFI/OC/config.plist`.
5. Select option **3** (Generate SMBIOS) and enter:
   ```text
   MacBookPro16,1
   ```
6. GenSMBIOS will automatically inject a unique `SystemSerialNumber`, `SystemUUID`, and `MLB` into your `config.plist`.

### Step 2: Set your ROM for iServices (FaceTime & iMessage)
For FaceTime, iMessage, and iPhone call routing to work reliably, your `ROM` must match the MAC address of your primary network interface (`en0`):
1. Open your `EFI/OC/config.plist` using ProperTree or OpenCore Configurator.
2. In `PlatformInfo -> Generic -> ROM`, replace `000000000000` with the MAC address of your `en0` interface (without colons or spaces, 12 hexadecimal characters).
3. Save the file.

### Step 3: Copy to your EFI partition
1. Mount your internal SSD's EFI partition using Terminal or MountEFI:
   ```bash
   sudo diskutil mount diskXs1
   ```
2. Copy the `EFI` folder to the root of your EFI partition.

---

## ⚡ Recommended Post-Install Optimizations

### 1. Optimize Sleep / Wake (Prevent battery drain in backpack)
Open **Terminal** and run:
```bash
# 1. Configure pure S3 sleep in RAM and disable network background wakeups
sudo pmset -a hibernatemode 0
sudo pmset -a standby 0
sudo pmset -a autopoweroff 0
sudo pmset -a tcpkeepalive 0
sudo pmset -a proximitywake 0
sudo pmset -a powernap 0

# 2. Reclaim disk space by deleting the 16 GB hibernation sleepimage
sudo rm -f /var/vm/sleepimage
sudo touch /var/vm/sleepimage
sudo chflags uchg /var/vm/sleepimage
```

### 2. Set Up Fan Curves in Macs Fan Control
1. Download and install [Macs Fan Control](https://crystalidea.com/macs-fan-control).
2. Thanks to `SMCDellSensors.kext`, both **Left side** and **Right side** fans will be detected.
3. Click on **Custom (Personalizar)** for each fan:
   - **Control based on sensor:** Select `CPU Core (average)`.
   - **Temperature to ramp up:** `55 °C` or `60 °C`.
   - **Maximum temperature:** `80 °C` or `82 °C`.
4. This ensures proactive cooling during heavy loads while keeping the laptop silent at idle (~45 °C).

---

## 👤 Known Limitations & Workarounds

* **Internal Microphone:** The built-in microphone array is a Digital Microphone (DMIC) connected directly to the Intel SST DSP (Smart Sound Technology cAVS), which lacks macOS drivers.
  * *Workaround:* Use AirPods/Bluetooth headphones, any 3.5mm headset with an in-line mic, or an external USB microphone.
* **Apple Watch Unlock / AirDrop:** Apple's AWDL protocol requires proprietary Broadcom Wi-Fi chipsets. Intel Wi-Fi works wonderfully for internet, hotspot, and local networks, but does not support AirDrop.
  * *Workaround:* Use [LocalSend](https://localsend.org/) for fast cross-platform file transfers between iPhone, Mac, and PC.

---

## 👏 Credits & Acknowledgements

* [Acidanthera](https://github.com/acidanthera) for OpenCorePkg, Lilu, VirtualSMC, AppleALC, WhateverGreen, CPUFriend, and VoodooPS2.
* [OpenIntelWireless](https://github.com/OpenIntelWireless) for AirportItlwm and IntelBluetoothFirmware.
* [0xFireWolf](https://github.com/0xFireWolf) for RealtekCardReader and RealtekCardReaderFriend.
* [alexandred](https://github.com/alexandred) and [VoodooI2C team](https://github.com/VoodooI2C/VoodooI2C) for precision touchpad support.
* [Dortania](https://dortania.github.io/) for their comprehensive OpenCore installation guides.
