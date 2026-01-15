# PiHole with Tailscale on Raspberry Pi Zero 2W

A comprehensive guide for setting up PiHole network-wide ad blocking with Tailscale VPN on a Raspberry Pi Zero 2W.

## Table of Contents
- [Overview](#overview)
- [Hardware Requirements](#hardware-requirements)
- [Prerequisites](#prerequisites)
- [Installation Steps](#installation-steps)
  - [1. Raspberry Pi OS Setup](#1-raspberry-pi-os-setup)
  - [2. Initial Configuration](#2-initial-configuration)
  - [3. Installing PiHole](#3-installing-pihole)
  - [4. Installing Tailscale](#4-installing-tailscale)
  - [5. Network Configuration](#5-network-configuration)
- [Configuration](#configuration)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)
- [Maintenance](#maintenance)
- [References](#references)

## Overview

This project documents the installation and configuration of:
- **PiHole**: Network-wide ad blocking
- **Tailscale**: Secure VPN for remote access
- **Raspberry Pi Zero 2W**: Low-power, compact hardware platform

Together, these provide a portable, energy-efficient ad-blocking DNS server accessible from anywhere via VPN.

## Hardware Requirements

- **Raspberry Pi Zero 2W** (or Raspberry Pi Zero W/WH)
- **MicroSD Card**: 8GB minimum (16GB+ recommended)
- **Power Supply**: 5V 2.5A USB power adapter with micro-USB cable
- **Optional**: Case for protection and cooling

## Prerequisites

Before starting, ensure you have:
- A computer with SD card reader
- Internet connection
- Basic command-line knowledge
- Tailscale account (free at [tailscale.com](https://tailscale.com))

## Installation Steps

### 1. Raspberry Pi OS Setup

#### Download Raspberry Pi Imager
Download and install the official Raspberry Pi Imager from [raspberrypi.com/software](https://www.raspberrypi.com/software/).

#### Flash the OS
1. Insert your microSD card into your computer
2. Open Raspberry Pi Imager
3. Click **"Choose OS"** → **"Raspberry Pi OS (other)"** → **"Raspberry Pi OS Lite (64-bit)"**
   - Lite version is recommended for headless operation
4. Click **"Choose Storage"** and select your microSD card
5. Click the **gear icon** (⚙️) to access advanced options:
   - Enable SSH
   - Set username and password
   - Configure WiFi (SSID and password)
   - Set locale settings
6. Click **"Write"** and wait for completion

### 2. Initial Configuration

#### Boot and Connect
1. Insert the microSD card into your Raspberry Pi Zero 2W
2. Power on the device
3. Wait 2-3 minutes for first boot
4. Find the Pi's IP address:
   - Check your router's connected devices, or
   - Use: `ping raspberrypi.local`

#### SSH Connection
```bash
ssh <username>@raspberrypi.local
# or
ssh <username>@<IP_ADDRESS>
```
Replace `<username>` with the username you configured in step 5 during OS imaging.

#### Update the System
```bash
sudo apt update
sudo apt upgrade -y
sudo reboot
```

#### Set Static IP (Optional but Recommended)
```bash
sudo nano /etc/dhcpcd.conf
```

Add at the end (adjust for your network):
```
interface wlan0
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

Save and reboot:
```bash
sudo reboot
```

### 3. Installing PiHole

#### Run the Installation Script
```bash
curl -sSL https://install.pi-hole.net | bash
```

#### Installation Wizard
Follow the on-screen prompts:
1. **Welcome Screen**: Press Enter
2. **Free and Open Source**: Acknowledge
3. **Static IP Needed**: Confirm your static IP setup
4. **Select Upstream DNS Provider**: Choose one (e.g., Google, Cloudflare, or OpenDNS)
5. **Block Lists**: Accept default lists
6. **Admin Web Interface**: Yes (recommended)
7. **Web Server**: Yes, install lighttpd
8. **Query Logging**: Yes (recommended)
9. **Privacy Mode**: Select desired level

#### Save Admin Password
At the end of installation, you'll see:
```
The install log is in /etc/pihole/install.log
View the web interface at http://pi.hole/admin or http://<IP>/admin
Your Admin password is: <random_password>
```

**Save this password!** Or change it:
```bash
pihole -a -p
```

#### Access the Web Interface
Open in browser: `http://<PI_IP_ADDRESS>/admin`

### 4. Installing Tailscale

#### Install Tailscale
```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

#### Authenticate
```bash
sudo tailscale up
```

Follow the URL displayed to authenticate with your Tailscale account.

#### Enable Exit Node (Optional)
To route all traffic through the Pi:
```bash
sudo tailscale up --advertise-exit-node
```

Then enable in the Tailscale admin panel.

#### Configure as Subnet Router (Optional)
To access your local network remotely:
```bash
sudo tailscale up --advertise-routes=192.168.1.0/24
```

Enable in Tailscale admin panel under machine settings.

### 5. Network Configuration

#### Configure Devices to Use PiHole

**Option 1: Router-Level (Recommended)**
1. Access your router's admin panel
2. Find DNS settings (usually under LAN or DHCP settings)
3. Set Primary DNS to your Pi's IP address
4. Save and reboot router

**Option 2: Per-Device**
Configure DNS manually on each device to point to the Pi's IP address.

**Option 3: Via Tailscale**
1. Get your Pi's Tailscale IP: `tailscale ip -4` (typically in 100.64.0.0/10 range)
2. In Tailscale admin panel, go to DNS settings
3. Add nameserver: your Pi's Tailscale IP from step 1
4. Enable "Override local DNS"

## Configuration

### PiHole Configuration

#### Add Additional Blocklists
1. Log into web interface
2. Navigate to **Group Management** → **Adlists**
3. Add URLs from [firebog.net](https://firebog.net/)
4. Update gravity: **Tools** → **Update Gravity**

#### Whitelist Domains
```bash
pihole -w example.com
```

Or via web interface: **Whitelist** → Add domain

#### Blacklist Domains
```bash
pihole -b ads.example.com
```

### Tailscale Configuration

#### View Tailscale Status
```bash
tailscale status
```

#### View Tailscale IP
```bash
tailscale ip -4
```

#### Disable Key Expiry
In Tailscale admin panel:
1. Select your Pi device
2. Click **"..."** → **"Disable key expiry"**

## Usage

### Accessing PiHole Remotely
With Tailscale connected on your device:
1. Get your Pi's Tailscale IP: `tailscale ip -4` on the Pi
2. Access web interface: `http://<TAILSCALE_IP>/admin` (e.g., `http://100.64.1.2/admin`)

### Monitoring
- View blocking stats in web interface dashboard
- Check query log for specific domain lookups
- Monitor long-term statistics in web interface

### Command Line Management
```bash
# View status
pihole status

# Enable/Disable blocking
pihole enable
pihole disable

# Update blocklists
pihole -g

# Tail query log
pihole -t

# View help
pihole -h
```

## Troubleshooting

### PiHole Not Blocking Ads
1. Verify DNS settings on router/device
2. Clear browser cache and DNS cache
3. Check if domains are whitelisted
4. Update gravity: `pihole -g`

### Cannot Access Web Interface
```bash
# Restart PiHole services
pihole restartdns

# Restart web server
sudo service lighttpd restart
```

### Tailscale Connection Issues
```bash
# Check Tailscale status
sudo tailscale status

# Restart Tailscale
sudo systemctl restart tailscaled

# Re-authenticate
sudo tailscale up
```

### High Memory Usage
The Raspberry Pi Zero 2W has limited RAM (512MB). To optimize:
```bash
# Reduce log retention
sudo nano /etc/pihole/pihole-FTL.conf
# Add: MAXDBDAYS=7
```

### WiFi Connectivity Issues
```bash
# Check WiFi status
iwconfig

# Reconnect to WiFi
sudo wpa_cli -i wlan0 reconfigure

# View WiFi logs
sudo journalctl -u wpa_supplicant
```

## Maintenance

### Regular Updates
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Update PiHole
pihole -up

# Update Tailscale
sudo tailscale update
```

### Backup Configuration
```bash
# Backup PiHole settings
pihole -a -t

# Backup to remote location
scp /home/pi/pihole-backup-*.tar.gz user@remote:/backup/
```

### Monitor System Resources
```bash
# View CPU temperature
vcgencmd measure_temp

# View memory usage
free -h

# View disk usage
df -h
```

## References

- [PiHole Official Documentation](https://docs.pi-hole.net/)
- [Tailscale Documentation](https://tailscale.com/kb/)
- [Raspberry Pi Documentation](https://www.raspberrypi.com/documentation/)
- [Raspberry Pi Zero 2W Specifications](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/)

---

**Note**: This documentation is based on installation on a Raspberry Pi Zero 2W. Steps may vary slightly for other Raspberry Pi models.