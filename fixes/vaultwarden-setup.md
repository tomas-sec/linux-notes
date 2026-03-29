# Vaultwarden Self-Hosted Password Manager

## Goal
Replace Proton Pass with a self-hosted Vaultwarden instance running
on Proxmox, accessible from any device with a trusted SSL certificate.

## System Info
- **Host:** Proxmox VE (pve01)
- **Container:** Debian 12 LXC (CT ID 101)
- **Vaultwarden IP:** 192.168.68.21
- **Reverse Proxy:** Caddy with Let's Encrypt SSL

## Steps

### 1. Create LXC Container
Settings used:
- Hostname: vaultwarden
- Template: debian-12-standard
- Disk: 8GB
- CPU: 1 core
- Memory: 512MB
- IP: 192.168.68.21/24
- Gateway: 192.168.68.1

### 2. Install curl and Docker
```bash
apt update && apt install curl -y
curl -fsSL https://get.docker.com | sh
```

### 3. Run Vaultwarden
```bash
docker run -d \
  --name vaultwarden \
  --restart unless-stopped \
  -v /vw-data/:/data/ \
  -e SIGNUPS_ALLOWED=false \
  -p 8080:80 \
  vaultwarden/server:latest
```

### 4. Install Caddy Reverse Proxy
Vaultwarden requires HTTPS — Caddy handles SSL automatically:
```bash
apt install -y debian-keyring debian-archive-keyring apt-transport-https
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg
curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | tee /etc/apt/sources.list.d/caddy-stable.list
apt update && apt install caddy -y
```

### 5. Configure Caddyfile
```bash
nano /etc/caddy/Caddyfile
```
Contents:
```
yourdomain.duckdns.org {
    reverse_proxy localhost:8080
}
```
Then Run:
```bash
systemctl restart caddy
```

### 6. Set Up DuckDNS
- Created free subdomain at duckdns.org
- Pointed to public IP for Let's Encrypt cert validation
- Added local DNS record in Pihole:
  - Domain: yourdomain.duckdns.org
  - IP: 192.168.68.21

### 7. Port Forward on Router
In Deco app:
- External port: 443
- Internal IP: 192.168.68.21
- Internal port: 443
- Protocol: TCP

Caddy automatically obtained a trusted Let's Encrypt certificate.

### 8. Connect Bitwarden App
In Bitwarden mobile app:
- Region → Self-hosted
- Server URL: yourdomain.duckdns.org
- Log in with account credentials

## Issues Hit Along the Way
- Self signed certificates rejected by mobile apps — solved with
  DuckDNS + Let's Encrypt via Caddy
- Port 443 conflict — moved Vaultwarden to port 8080 internally
- RFC1918 rejection — solved by pointing DuckDNS to public IP
  and adding local DNS override in Pihole

## Security Notes
- SIGNUPS_ALLOWED=false prevents anyone from registering new accounts
- Port 443 is open to the internet — only existing accounts work
- All vault data is encrypted client-side by Bitwarden

## What I Learned
- Self signed certs work in browsers but mobile apps reject them
- Caddy automatically handles Let's Encrypt certificates
- DuckDNS provides free subdomains for home servers
- Pihole local DNS records override public DNS for local network
- Any internet-exposed service should have registration disabled
- Port forwarding exposes services publicly — always lock down first
