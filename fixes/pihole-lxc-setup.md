# Pihole Setup on Proxmox LXC Container

## Goal
Run Pihole as a lightweight LXC container on Proxmox to filter
ads and trackers network-wide.

## System Info
- **Host:** Proxmox VE (pve01)
- **Container:** Debian 12 LXC (CT ID 100)
- **Pihole IP:** 192.168.68.20
- **DNS Provider:** Cloudflare (1.1.1.1)

## Steps

### 1. Download Debian 12 Template
In Proxmox web UI:
local storage → CT Templates → Templates → search debian-12 → Download

### 2. Create LXC Container
Settings used:
- Hostname: pihole
- Template: debian-12-standard
- Disk: 4GB
- CPU: 1 core
- Memory: 512MB
- IP: 192.168.68.20/24
- Gateway: 192.168.68.1

### 3. Fix DNS on Proxmox Host
Container couldn't download packages — resolv.conf was pointing to 127.0.0.1:
```bash
nano /etc/resolv.conf
```
Changed to:
```
nameserver 1.1.1.1
nameserver 8.8.8.8
```

### 4. Fix Time Sync Issue
apt was throwing "Release file not valid yet" errors due to wrong system time on host.
```bash
apt install chrony -y
systemctl enable --now chronyd
```

### 5. Install Curl
Debian minimal doesn't include curl by default:
```bash
apt update && apt install curl -y
```

### 6. Install Pihole
```bash
curl -sSL https://install.pi-hole.net | bash
```
Selected during install:
- DNS provider: Cloudflare
- Privacy mode: Mode 1
- Query logging: Show everything

### 7. Change Admin Password
Password shown on screen during install so changed immediately:
```bash
pihole setpassword yournewpassword
```

### 8. Point Router to Pihole
In Deco app:
More → Advanced → DHCP Server
- Primary DNS: 192.168.68.20
- Secondary DNS: 1.1.1.1

## Result
Pihole live and filtering network-wide ✅
All devices on network using Pihole as DNS automatically.

## What I Learned
- LXC containers are lighter than full VMs for simple services
- Containers inherit the host clock — time issues must be fixed on the host
- Always set a secondary DNS fallback in case Pihole goes down
- YouTube ads cannot be blocked via DNS — Google serves them from the same domains as video content
