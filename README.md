
# PVE: Emulate a Real Physical Machine  
**Debian • Ubuntu • Arch Linux VMs** (Proxmox VE)

[![Star History Chart](https://api.star-history.com/svg?repos=AICodo/pve-emu-realpc&type=Date)](https://www.star-history.com/#AICodo/pve-emu-realpc&Date)

**Languages:** [English-from CN](README.md) | [English](README.en.md)  
*Thanks to [@mk990](https://github.com/mk990) for the translation.*

---

## 🚀 Latest Updates

### 2025-09-06
- Motherboard model & memory serial now fully customizable (no more random)  
- IDE/SATA disk serial fixed with `serial=20-character-string` (random by default)  
- NVIDIA GPU passthrough **Error 43** fixed: use `ssdt.aml` (no battery) **or** `ssdt-battery.aml` (with battery)  
  → Desktop CPUs = no battery  
  → Laptop CPUs = with battery

### 2025-08-05
- ACPI SSDT loading added  
- `ssdt-battery.aml` = virtual battery + CPU/motherboard temp + fan  
- Edit with [Xiasl](https://github.com/ic005k/Xiasl)

### 2025-07-25
- Randomized trio on every VM restart (memory serial, disk serial/firmware, motherboard model)

### 2025-02-28
- **CPU sensor passthrough** (temp, MHz, voltage, power) in Windows  
- Works with CPU-Z, HWInfo, HWMonitor (Intel & AMD)

**Intel Demo**  
![Intel CPU Sensor Passthrough](https://github.com/user-attachments/assets/e5c2f90b-7e65-45d3-bbc4-6825551d421d)

**AMD Demo**  
![AMD CPU Sensor Passthrough](https://github.com/user-attachments/assets/2ebe3ca5-c438-4b98-83d9-4295a7d001b1)

---

## 1. Preparation

1. PVE Web UI → **Datacenter** → **Options** → Set **MAC Address Prefix** to `D8:FC:93`
2. **Recommended**: Passthrough a real SATA/M.2 drive + physical wired NIC (USB NIC works great)

> **⚠️ This project has ZERO restrictions** on SCSI or VirtIO devices.

---

## 2. Installation

Upload these 3 files to `/root` (use WinSCP):

- `pve-qemu-kvm_10.xxx_amd64.deb` (from this repo)
- `pve-edk2-firmware-ovmf_xxx.deb` (from this repo or [AICodo repo](https://github.com/AICodo/pve-emu-realpc_edk2-firmware-ovmf))
- `ssdt.aml` (or `ssdt-battery.aml`)

### Check version
```bash
dpkg -l | grep pve-qemu-kvm
```

### Install
```bash
dpkg -i pve-qemu-kvm_10.*_amd64.deb
dpkg -i pve-edk2-firmware-ovmf_*_amd64.deb
```

**No reboot needed.**

### Restore original packages
```bash
apt reinstall pve-qemu-kvm
apt reinstall pve-edk2-firmware-ovmf
```

* * *

3\. Create VM (Recommended Settings)
------------------------------------

*   **Machine**: OVMF (UEFI) + Q35
*   **Disk**: SATA (≥128 GB — smaller looks fake)
*   **CPU**: host (1 socket, multiple cores)
*   **Network**: e1000
*   **Memory**: 8192 / 16384 MB (ballooning = 0)
*   Avoid all VirtIO devices

### Edit config
```bash
nano /etc/pve/qemu-server/100.conf
```

### Full recommended config (QEMU 9/10)

```conf
args: -acpitable file=/root/ssdt.aml -acpitable file=/root/ssdt-ec.aml -acpitable file=/root/hpet.aml -cpu host,host-cache-info=on,hypervisor=off,vmware-cpuid-freq=false,enforce=false,host-phys-bits=true -smbios type=0,vendor="American Megatrends International LLC.",version=H3.7G,date='02/21/2023',release=3.7 -smbios type=1,manufacturer="Maxsun",product="MS-Terminator B760M",version="VER:H3.7G(2022/11/29)",serial="Default string",sku="Default string",family="Default string" -smbios type=2,manufacturer="Maxsun",product="MS-Terminator B760M",version="VER:H3.7G(2022/11/29)",serial="Default string",asset="Default string",location="Default string" -smbios type=3,manufacturer="Default string",version="Default string",serial="Default string",asset="Default string",sku="Default string" -smbios type=17,serial=DF1EC466,asset="9876543210" -smbios type=4,manufacturer="Intel(R) Corporation",version="12th Gen Intel(R) 0000" -smbios type=9 -smbios type=8 -smbios type=8

balloon: 0
bios: ovmf
boot: order=ide2;sata0;net0
cores: 8
cpu: host
efidisk0: local:102/vm-102-disk-0.raw,efitype=4m,size=528K
ide2: none,media=cdrom
localtime: 1
memory: 16384
net0: e1000=D8:FC:93:56:1D:C7,bridge=vmbr0,firewall=1
numa: 0
ostype: l26
sata0: local:102/vm-102-disk-1.raw,size=128G,ssd=1,serial=0123456789ABCDEF0123
scsihw: virtio-scsi-single
smbios1: uuid=505429c8-a350-41e9-9154-3851c095254e
sockets: 1
usb0: host=0000:3825
usb1: host=258a:002a
vmgenid: 2271babc-cafc-4c68-be8b-2bb3157c9924
```

**Tip**: Change serial=0123456789ABCDEF0123 (exactly 20 chars) and SMBIOS values to customize.

* * *

Testing
-------

Use **AIDA64**, **CPU-Z**, **HWInfo**, **HWMonitor** Advanced tools in the repo’s tools/ folder: al-khaser & pafish64.exe

* * *

**Project for education & experimentation.** Fork and push further!

⭐ Star if it helped you!
