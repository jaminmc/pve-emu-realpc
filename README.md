Language [<a href="README.md">Chinese</a>] | [<a href="README.en.MD">English</a>] Thanks for https://github.com/mk990 translate.

PVE Debian Ubuntu ArchLinux virtual machine emulates a physical machine


20060228 update: Released 10.1.2-7_amd64_Strong which can dynamically display CPU information such as temperature, MHz, voltage, and power consumption in a Windows VM. Use cpu-z, hwinfo, hwmonitor. Intel and AMD CPU sensor passthrough to VM display.

Intel CPU sensor passthrough demo video<img width="1545" height="1154" alt="inteldemo" src="https://github.com/user-attachments/assets/e5c2f90b-7e65-45d3-bbc4-6825551d421d" />


https://github.com/user-attachments/assets/cf95f3a1-9f47-46a1-94e5-cd68c5f5881c

AMD CPU sensor passthrough demo video<img width="1478" height="1182" alt="amddemo" src="https://github.com/user-attachments/assets/2ebe3ca5-c438-4b98-83d9-4295a7d001b1" />


https://github.com/user-attachments/assets/69a922ea-df2c-4d13-94cc-d40736336b2e

20250906 update: Removed random motherboard model (can be customized), removed random memory serial (can be customized), IDE/SATA disk serial can be set with serial=20-character-serial for fixed customization (defaults to random if not set). For NVIDIA discrete GPU passthrough error 43, choose one of: ssdt.aml (without battery) or ssdt-battery.aml (with virtual battery). Desktop CPUs should use no battery, laptop CPUs should use battery. Once SSDT is loaded and error 43 is resolved, you're done.

20250805 update: Added ACPI SSDT loading feature. ssdt-battery.aml includes a virtual battery (visible), ssdt.aml has no virtual battery, virtual CPU and motherboard temperature (visible), virtual fan (not visible). You can use https://github.com/ic005k/Xiasl to edit ssdt.aml (ssdt.aml==ssdt.dat, just different file extensions) for custom modifications.

20250725 update: Implemented randomized trio effect (just restart the VM for automatic changes): random memory serial, random IDE/SATA disk serial and firmware number, random motherboard model.


1. Preparation:

PVE web UI -> Datacenter -> Options -> Change MAC Address Prefix to D8:FC:93

My personal recommendation is to passthrough a SATA or M.2 drive to the VM, along with a wired physical NIC (USB NIC) for testing.

!!! This project has NO usage restrictions on SCSI or Virtio devices !!!



2. Getting Started

Please upload these 2 deb packages and 1 file:

pve-qemu-kvm_10.xxx_amd64.deb  Download from this project (xxx = your specific version)

pve-edk2-firmware-ovmf_xxx.deb  Download from this project, or from https://github.com/AICodo/pve-emu-realpc_edk2-firmware-ovmf

ssdt.aml

Upload these 3 files to /root using WinSCP


3. Check currently installed KVM package version

dpkg -l|grep pve-qemu-kvm

4. If running version 10.x, directly install these 2 anti-detection packages:

dpkg -i pve-qemu-kvm_10.xxx_amd64.deb  (xxx = your specific version)

dpkg -i pve-edk2-firmware-xxx.deb


If your QEMU version is not the latest, upgrade your system and install the latest package from this project:

apt update

apt install pve-qemu-kvm

dpkg -i pve-qemu-kvm_10.xxx_amd64.deb (xxx = your specific version)

dpkg -i pve-edk2-firmware-ovmf_xxx.deb


No reboot required after installation


To restore official packages, just run these commands:

apt reinstall pve-qemu-kvm

#If reinstall fails, force reinstall a specific version: apt install pve-qemu-kvm=10.0.2-4 or apt reinstall pve-qemu-kvm=10.0.2-4

apt reinstall pve-edk2-firmware-ovmf or apt reinstall pve-edk2-firmware-ovmf=4.2025.02-4

5. Create a new virtual machine

Use OVMF+Q35 (recommended) or OVMF+i440fx. In the configuration, make sure to select SATA disk (at least 128GB - 50GB/80GB are too small to look like a real physical disk, don't be stingy with disk size; avoid SCSI and Virtio disk/optical/network devices), IDE or SATA optical drive, display set to Standard initially (switch to GPU passthrough for dedicated/integrated/vGPU later), CPU set to host (1 socket with multiple cores - this is important), network adapter set to e1000 (watch out for MAC address issues to avoid VM detection), avoid all Virtio devices (SCSI disks/optical drives, Virtio NIC, VirtioBlock disks, Virtio-GPU, etc.), and modify the VM args parameters to match mine. Memory should be 8192, 16384, or 4096 with ballooning disabled (to look more like physical machine memory), corresponding to 8GB, 16GB, 4GB. Do not use other sizes (too suspicious, too VM-like).

Only one principle: disk size, memory size, and NICs must look like a real physical machine configuration!!

Use the following command to edit VM configuration:

nano /etc/pve/qemu-server/100.conf

My complete VM configuration (for QEMU 9 and 10; for QEMU 7 and 8 see supplementary notes):

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

meta: creation-qemu=9.0.2,ctime=1724320553

name: win10

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

Supplementary: In QEMU 10, the args parameters above are all that's needed (other settings are hidden/built-in). Just use the args shown above. To fix disk serial randomization, set serial=0123456789ABCDEF0123 (20-character string) for IDE/SATA. Check aida64 to see what hardware is present (fans, temperature, voltage, etc.)

For QEMU 7 and 8, use the following args:

args: -acpitable file=/root/ssdt.aml -cpu host,host-cache-info=on,hypervisor=off,vmware-cpuid-freq=false,enforce=false,host-phys-bits=true -smbios type=0,vendor="American Megatrends International LLC.",version=H3.7G,date='02/21/2023',release=3.7 -smbios type=1,manufacturer="Maxsun",product="MS-Terminator B760M",version="VER:H3.7G(2022/11/29)",serial="Default string",sku="Default string",family="Default string" -smbios type=2,manufacturer="Maxsun",product="MS-Terminator B760M",version="VER:H3.7G(2022/11/29)",serial="Default string",asset="Default string",location="Default string" -smbios type=3,manufacturer="Default string",version="Default string",serial="Default string",asset="Default string",sku="Default string" -smbios type=17,loc_pfx="Controller0-ChannelA-DIMM",manufacturer="KINGSTON",speed=3200,serial=DF1EC466,part="SED3200U1888S",bank="BANK 0",asset="9876543210" -smbios type=4,sock_pfx="LGA1700",manufacturer="Intel(R) Corporation",version="12th Gen Intel(R) Core(TM) i7-12700",max-speed=4900,current-speed=3800,serial="To Be Filled By O.E.M.",asset="To Be Filled By O.E.M.",part="To Be Filled By O.E.M." -smbios type=8,internal_reference="CPU FAN",external_reference="Not Specified",connector_type=0xFF,port_type=0xFF -smbios type=8,internal_reference="J3C1 - GMCH FAN",external_reference="Not Specified",connector_type=0xFF,port_type=0xFF -smbios type=8,internal_reference="J2F1 - LAI FAN",external_reference="Not Specified",connector_type=0xFF,port_type=0xFF -smbios type=11,value="Default string"

6. Additional content is in the tools directory of this project, including VM detection tools. For advanced detection, al-khaser and pafish64.exe are the gold standards for VM environment detection.


This project is meant to inspire further exploration. Feel free to fork and continue experimenting!!!
## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=AICodo/pve-emu-realpc&type=Date)](https://www.star-history.com/#AICodo/pve-emu-realpc&Date)
