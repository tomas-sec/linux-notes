# WiFi Not Auto-Connecting on Arch KDE

## Problem
After every reboot, WiFi wouldn't automatically reconnect. 
Had to manually run `nmcli dev wifi connect <network> --ask` every time.

## Cause
Using `--ask` flag with nmcli creates a temporary connection 
profile without autoconnect enabled.

## Fix
Delete the old broken profile and reconnect properly:
```bash
nmcli connection delete 
nmcli dev wifi connect  password "yourpassword"
```

This creates a new profile with autoconnect enabled by default.

## Verified on
- OS: Arch Linux
- NetworkManager version: 1.54.1-1

## Date
March 2026
