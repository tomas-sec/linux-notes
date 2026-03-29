# Proxmox VE Installation & Network Setup

## Goal
Replace Ubuntu Server with Proxmox VE on a dual-drive machine and
access the web UI from another device on the home network.

## System Info
- **OS:** Proxmox VE
- **Hostname:** pve01.homelab.local
- **Hardware:** Dual drive (small drive for Proxmox OS, 1TB for storage)
- **Web UI:** https://192.168.68.10:8006

## Steps

### 1. Create Bootable USB
Installed Ventoy on CachyOS:
```bash
yay -S ventoy-bin
sudo ventoyweb
```
Dropped Proxmox ISO onto the USB via file manager.

### 2. Install Proxmox
Booted from USB, installed Proxmox onto the smaller drive.
Set hostname to `pve01.homelab.local` and static IP during install.

### 3. Boot Issues
Machine kept booting into Ubuntu GRUB instead of Proxmox.

**Cause:** Two drives, Ubuntu on 1TB was still first in boot order.

**Fix:** Changed boot order in BIOS to prioritize the smaller drive.
GRUB still loaded but Proxmox wasn't in the menu — had to manually
boot it via GRUB command line:
```bash
set root=(lvm/pve-root)
ls /boot/
linux /boot/vmlinuz-6.17.2-1-pve root=/dev/mapper/pve-root
initrd /boot/initrd.img-6.17.2-1-pve
boot
```

### 4. Network Not Reachable
Proxmox installed with IP `192.168.100.2` but home network is
`192.168.68.x` — different subnets, web UI unreachable.

**Fix:** Edited `/etc/network/interfaces`:
```bash
nano /etc/network/interfaces
```
Changed vmbr0 config to:
```
iface vmbr0 inet static
        address 192.168.68.10/24
        gateway 192.168.68.1
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0
```
Then restarted networking:
```bash
systemctl restart networking
```

### 5. NIC Link Down
After network change, NIC was dropping link intermittently.

**Cause:** Was plugged into secondary router on wrong subnet.

**Fix:** Switched ethernet cable to main router port.

## Result
Proxmox web UI accessible at `https://192.168.68.10:8006` ✅

## What I Learned
- Proxmox installs like any OS and wipes the target drive
- FQDN format is required for hostname during install
- bridge-ports must match the exact NIC name shown in `ip a`
- Always plug into the main router, not a secondary switch on a different subnet
- Physical layer issues (wrong cable/port) cause the most confusing errors
