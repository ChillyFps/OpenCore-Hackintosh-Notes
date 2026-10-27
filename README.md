# 🚀 Lenovo Legion R9000P 2023 (ARX8) - macOS Sequoia OpenCore EFI

Optimized OpenCore EFI configuration for **Lenovo Legion R9000P ARX8** (and Legion Pro 5 / 7 Gen 8 AMD) featuring the **AMD Ryzen 9 7945HX** (16-Core Zen 4) and **NVIDIA GeForce RTX 4060 Laptop GPU** running **macOS 15 Sequoia** powered by the **NullMoth NVIDIA Driver** translation stack.

---

## 💻 Hardware Specifications

| Component | Hardware Specification | macOS Support / Notes |
| :--- | :--- | :--- |
| **Model** | Lenovo Legion R9000P 2023 ARX8 / Legion Pro 5 16ARX8 | Full Chassis Support (YogaSMC) |
| **CPU** | AMD Ryzen 9 7945HX (16 Cores / 32 Threads, Zen 4) | ✅ Working (AMD Vanilla 16-Core Patched) |
| **dGPU** | NVIDIA GeForce RTX 4060 Laptop GPU (8GB GDDR6, AD107) | ✅ Metal 3 Accelerated via NullMoth Driver |
| **iGPU** | AMD Radeon 610M (RDNA 2, 2 CUs) | ❌ Disabled via BIOS MUX Switch (dGPU Mode) |
| **RAM** | 16 GB DDR5 5200MHz | ✅ Full Dual-Channel Working |
| **Storage** | 1TB NVMe PCIe 4.0 SSD | ✅ Working (`NVMeFix.kext` enabled) |
| **Display** | 16" 2.5K WQXGA (2560x1600), 240Hz, 16:10 | ✅ Working (`NVRMFB` Direct Head) |
| **Audio** | Realtek ALC3306 (ALC287 variant) + Harman Speakers | ✅ Working (`AppleALC.kext` `alcid=13`) |
| **Ethernet** | Realtek RTL8111 Gigabit Ethernet | ✅ Working (`RealtekRTL8111.kext`) |
| **Wi-Fi / BT**| MediaTek MT7922 / Realtek RTL8852CE (OEM) | ⚠️ Swap recommended (Intel AX210 or BCM94360NG) |
| **Trackpad** | I2C Precision Multi-Touch Touchpad | ✅ Working (`VoodooI2C` + `VoodooI2CHID`) |
| **Keyboard** | Lenovo Legion TrueStrike RGB Keyboard | ✅ Working (`VoodooPS2Keyboard` + Fn shortcuts) |
| **Battery** | 80Wh Battery + 300W Adapter | ✅ Working (`SMCBatteryManager` + `YogaSMC`) |

---

## ⚙️ Mandatory BIOS Settings

Before booting OpenCore or macOS, enter BIOS Setup (**F2** or **Fn+F2** on startup) and set the following:

### 🔴 Critical Settings
* **Graphics Device**: **Discrete Graphics (dGPU Only / MUX Switch Enabled)**
  > ⚠️ **MANDATORY**: Modern NVIDIA mobile drivers require direct ownership of display framebuffers. If set to Dynamic/Optimus, the display will route through the unsupported AMD Radeon 610M iGPU, leading to a black screen.
* **Secure Boot**: **Disabled**
* **Fast Boot**: **Disabled**
* **Above 4G Decoding**: **Enabled**
* **CSM / Legacy Support**: **Disabled (Pure UEFI)**
* **AMD IOMMU (AMD-Vi)**: Can remain Enabled (`DisableIoMapper=True` is configured in OpenCore)
* **AMD SVM (Virtualization)**: Enabled

---

## 🛠️ Optimizations Applied in this EFI

This EFI was heavily refactored from generic OpenCore Simplify output to eliminate freezes, stutter, and driver clashes:

1. **NVRAM Runtime Writes Restored (`DisableVariableWrite = false`)**:
   * Fixed critical bug where NVRAM runtime writes were locked. macOS can now save display brightness, sound volume, SIP status, and startup disk selection across reboots.
2. **Trackpad Race Condition & Micro-Stutter Eliminated**:
   * Disabled duplicate `VoodooPS2Trackpad.kext` and `VoodooPS2Mouse.kext`. 
   * Internal multi-touch gestures are handled solely by `VoodooI2C` + `VoodooI2CHID`, while the keyboard is routed via `VoodooPS2Keyboard.kext`.
3. **SMBIOS Upgraded to `MacBookPro16,1`**:
   * Upgraded from `MacBookPro16,2` (13" quad-core Intel iGPU-only) to `MacBookPro16,1` (16" high-core laptop with dedicated discrete GPU). This aligns macOS power management (`AGPM`), graphics device policies (`AGDP`), and thermal throttling with the 16-Core Zen 4 + RTX 4060 hardware.
4. **Redundant Intel ACPI Cleaned Up**:
   * Disabled `SSDT-PLUG-ALT.aml` (which injected fake Intel Alder Lake CPU objects into AMD namespace).
   * Disabled `SSDT-USB-Reset.aml` (which targeted Intel 400-series PCH RHUB).
5. **AMD Zen 4 16-Core Vanilla Patches**:
   * Kernel core-count patches (`algrey - Force cpuid_cores_per_package`) configured for **16 physical cores** (`0x10` hex) for macOS Sequoia (15.x / Darwin 24.x).
6. **IOMMU Protection (`DisableIoMapper = true`)**:
   * Prevents AppleVTD memory mapping panics on boot when AMD-Vi is enabled in Legion UEFI.
7. **Audio Reliability**:
   * Injected `alcid=13` directly into `boot-args` for Realtek ALC3306/287 codec support.
8. **NullMoth Driver Integration Arguments**:
   * `nvfb=1 nvaccel=1 nvfbheads=4 -nvkmsnosmooth amfi_get_out_of_my_way=0x1 amfi=0x80 ipc_control_port_options=0 gfxrst=4` included for full Metal 3 hardware acceleration.

---

## 🚀 Installation Guide

### Step 1: Prepare OpenCore EFI & Generate Serials
1. Copy this `EFI` folder to your USB drive's EFI partition (`ESP`).
2. Open `EFI/OC/config.plist` with [ProperTree](https://github.com/corpnewt/ProperTree).
3. If using the GitHub release template (`config.plist.template`), rename it to `config.plist`.
4. Run [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) to generate unique serial numbers:
   * Select `MacBookPro16,1`
   * Fill in `MLB`, `SystemSerialNumber`, and `SystemUUID` under `PlatformInfo -> Generic`.

### Step 2: Install macOS Sequoia
1. Boot from your OpenCore USB stick and choose **Install macOS Sequoia**.
2. Complete the standard macOS installation (the system will reboot several times).

### Step 3: Install NullMoth NVIDIA Driver Stack
1. Once on the macOS desktop, download the **1401 Companion App** or driver package from [nullmoth/nvidia-macos-driver](https://github.com/nullmoth/nvidia-macos-driver).
2. Run the driver installer to place the GPU bundle and kernel extensions into `/Library/GPUBundles/` and `/Library/Extensions/`.
3. Rebuild the auxiliary kernel collection (AKC) when prompted.
4. Perform a **full system restart** (do not simply log out).
5. Verify hardware acceleration in **About This Mac** and **System Information -> Graphics/Displays**.

---

## 📶 Wi-Fi & Bluetooth Note

Lenovo Legion R9000P / Pro 5 laptops typically ship from the factory with either a **MediaTek MT7922 (RZ616)** or **Realtek RTL8852CE** Wi-Fi 6E card. Neither chipset is supported by macOS.

* **Recommended Solution**: Swap the internal M.2 2230 Wi-Fi card with an **Intel AX210** (use `AirportItlwm.kext` / `IntelBluetoothFirmware.kext`) or a native Broadcom card (e.g. `BCM94360NG`).
* **Alternative**: Use USB Wi-Fi dongles supported by Wireless USB OC-BIG-SUR-Adapter or an external USB-C Ethernet adapter.

---

## ⚖️ Credits & Acknowledgements

* **Apple**: macOS
* **Acidanthera**: OpenCore Bootloader, Lilu, AppleALC, VirtualSMC, WhateverGreen, VoodooPS2
* **NullMoth Systems**: NullMoth NVIDIA macOS Driver & 1401 installer tool
* **AMD-OSX Community & Algrey**: AMD Vanilla kernel patches for macOS
* **Alexandred & VoodooI2C Team**: VoodooI2C and VoodooI2CHID
* **zhen-zen**: YogaSMC (Lenovo Legion platform support)
