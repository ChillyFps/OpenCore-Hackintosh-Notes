# 🍎 OpenCore Hackintosh & Multi-Boot Guide

Technical documentation on building stable `EFI` folders with the **OpenCore** bootloader to run macOS on non-Apple x86 hardware alongside Windows and Linux.

---

## 🏗️ OpenCore `EFI` Directory Structure

```text
EFI
├── BOOT
│   └── BOOTx64.efi
└── OC
    ├── ACPI/          # Compiled .aml SSDT patches (EC, USBX, PLUG, PNLF)
    ├── Drivers/       # UEFI drivers (OpenRuntime.efi, HfsPlus.efi)
    ├── Kexts/         # Kernel extensions (Lilu, VirtualSMC, WhateverGreen, etc.)
    ├── Resources/     # OpenCanopy GUI picker assets & audio
    ├── Tools/         # OpenShell.efi, CleanNvram.efi
    ├── OpenCore.efi   # Main bootloader executable
    └── config.plist   # Master configuration file
