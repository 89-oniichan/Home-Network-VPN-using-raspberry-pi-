# Home VPN Server (WireGuard via PiVPN)

<div align="center">

![VPN Shield](https://img.shields.io/badge/VPN-WireGuard-00BCD4?style=for-the-badge&logo=wireguard&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Raspberry%20Pi-A22846?style=for-the-badge&logo=raspberry-pi&logoColor=white)
![Status](https://img.shields.io/badge/Status-Production%20Ready-00C853?style=for-the-badge)

</div>

A comprehensive guide for setting up a secure, self-hosted VPN server on a Raspberry Pi using WireGuard and PiVPN. This setup enables remote access to your home network from anywhere in the world.

---

<div align="center">

**⭐ For the best reading experience, view the interactive documentation on GitHub Pages ⭐**

[![OPEN DOCUMENTATION](https://img.shields.io/badge/OPEN-Interactive%20Documentation-00FF88?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCIgdmlld0JveD0iMCAwIDI0IDI0IiBmaWxsPSJub25lIiBzdHJva2U9ImN1cnJlbnRDb2xvciIgc3Ryb2tlLXdpZHRoPSIyIiBzdHJva2UtbGluZWNhcD0icm91bmQiIHN0cm9rZS1saW5lam9pbj0icm91bmQiPjxwYXRoIGQ9Ik0xOCAxM0g2VjEyYTIgMiAwIDAgMSAyLTJoOGMyIDAgMiAyIDIgMiAzIDAgMyAzIDMgM3YxYzAgMiAyIDIgMiAycy0yIDItMiAybS02LTJoN20tNSA4SDZWMjEiLz48cGF0aCBkPSJNNSAxMWgyIDItMiAySDVWMzJNMjMgMTFoLTJoLTVhMiAyIDAgMCAwLTIgMmgtMnoiLz48L3N2Zz4=)]([https://YOUR_USERNAME.github.io/YOUR_REPO_NAME/](https://github.com/89-oniichan/Home-Network-VPN-using-raspberry-pi-.git)

*(Interactive terminal, live diagrams, and enhanced navigation)*

</div>

## Table of Contents

- [Overview](#overview)
- [Pros & Cons](#pros--cons)
- [Network Topology](#network-topology)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
  - [Phase 1: Install Raspberry Pi OS](#phase-1-install-raspberry-pi-os)
  - [Phase 2: Install WireGuard VPN via PiVPN](#phase-2-install-wireguard-vpn-via-pivpn)
  - [Phase 3: Create VPN Profiles](#phase-3-create-vpn-profiles)
  - [Phase 4: Router Port Forwarding](#phase-4-router-port-forwarding)
  - [Phase 5: Server Configuration](#phase-5-server-configuration)
  - [Phase 6: Client Configuration](#phase-6-client-configuration)
  - [Phase 7: IP Forwarding & NAT Rules](#phase-7-ip-forwarding--nat-rules)
  - [Phase 8: Verify & Test](#phase-8-verify--test)
- [AllowedIPs & Routing Modes](#allowedips--routing-modes)
- [Troubleshooting](#troubleshooting)
- [Operations & Day-2 Tips](#operations--day-2-tips)
- [Glossary](#glossary)

---

## Overview

We built a **remote-access VPN** so your phone, PC, and any systems can reach any `192.168.1.x` device at home from anywhere. The Raspberry Pi terminates WireGuard on `wg0`, then forwards traffic to your home LAN via `wlan0` with NAT masquerading. Each client receives unique cryptographic keys and a dedicated VPN IP.

**Configuration Summary:**
- Server LAN: `192.168.1.x`
- Server VPN: `10.155.159.1`
- Phone: `10.155.159.2`
- PC: `10.155.159.3`
- UDP Port: `51820`

---

## Pros & Cons

### Advantages

- **Minimal latency** — WireGuard is significantly faster than OpenVPN
- **Simple configuration** — PiVPN automates most setup steps
- **End-to-end encrypted** — Modern cryptography (ChaCha20, Poly1305)
- **Low maintenance** — Runs as a systemd service with auto-restart
- **Cross-platform** — Works on iOS, Android, Windows, macOS, Linux

### Considerations

- **Port forwarding required** — Some ISPs block or restrict this
- **Uptime dependency** — Must maintain stable power and internet
- **Single point of failure** — If Pi goes down, VPN is unavailable
- **Dynamic IP challenges** — May need DDNS for changing IPs

---

## Network Topology

```
Internet (Public IP)
     |
     | UDP:51820
     v
Home Router (NAT + Port Forwarding)
     |
     | Forward to 192.168.1.x
     v
Raspberry Pi 5 — WireGuard Server
├── wlan0 (LAN Interface): 192.168.1.x
├── wg0 (VPN Interface): 10.155.159.1
└── NAT: iptables MASQUERADE on wlan0
     |
     | Forwarding
     v
Home LAN Devices (192.168.1.0/24)
├── Desktop PC
├── NAS
└── Smart Home IoT
```

**Remote VPN Clients:**
- Mobile Phone → `10.155.159.2`
- Windows PC → `10.155.159.3`
- MacBook → `10.155.159.4`
- Endpoint: `PUBLIC_IP:51820`

---

## Prerequisites

![Requirements](https://img.shields.io/badge/-Requirements-00BCD4?style=flat-square)

- **Raspberry Pi 5** (or compatible model)
- **Raspberry Pi OS Lite** (64-bit recommended)
- **MicroSD card** (32GB+)
- **Stable internet connection**
- **Router with port forwarding capability**
- **SSH access** to the Pi
- **Basic command-line knowledge**

---

![Setup](https://img.shields.io/badge/-Setup%20Instructions-00BCD4?style=flat-square)

---

## Installation

### Phase 1: Install Raspberry Pi OS ![Pi](https://img.shields.io/badge/-Pi%20OS-A22846?style=flat-square&logo=raspberry-pi)

1. **Download Raspberry Pi Imager**
   - Install [Raspberry Pi Imager](https://www.raspberrypi.com/software) from raspberrypi.com

2. **Choose your configuration**
   - **Device:** Raspberry Pi 5 (or your model)
   - **OS:** Raspberry Pi OS (Lite 64-bit) — desktop is optional
   - **Storage:** Your microSD card

3. **Configure advanced settings** (gear icon)
   - Set username: `pivpn`
   - Set secure password
   - Enable SSH with password authentication
   - Configure Wi-Fi credentials or use Ethernet

4. **Write, eject, and boot**
   - Write the image → Safely eject the card → Insert into Pi → Power it up

5. **Connect via SSH**
   ```bash
   ssh pivpn@192.168.1.x
   ```
   Replace `192.168.1.x` with your Pi's actual IP (check your router's DHCP list)

6. **Update the system**
   ```bash
   sudo apt update && sudo apt full-upgrade -y
   sudo reboot
   ```

---

### Phase 2: Install WireGuard VPN via PiVPN ![WireGuard](https://img.shields.io/badge/-WireGuard-88171A?style=flat-square&logo=wireguard)

PiVPN is a convenient installer that automates WireGuard setup, profile generation, and firewall rules.

**Run the installer:**
```bash
curl -L https://install.pivpn.io | bash
```

**Installation prompts:**

| Step | What to Choose |
|------|----------------|
| VPN Type | **WireGuard** |
| Static IP | Auto-detect or set manually (e.g., `192.168.1.89`) |
| Port | Default `51820` (UDP) |
| DNS Provider | **Cloudflare** (1.1.1.1) or **Quad9** (9.9.9.9) |
| Unattended Upgrades | Enable (recommended for security patches) |

**After installation completes:**
```bash
sudo reboot
```

---

### Phase 3: Create VPN Profiles ![Profiles](https://img.shields.io/badge/-Profiles-00BCD4?style=flat-square)

After the Pi reboots, create individual profiles for each device that will connect to your VPN.

**Add a new client:**
```bash
pivpn add
```
Give it a simple name (e.g., `phone` or `laptop`). PiVPN generates the config file at:
```bash
/home/pivpn/configs/phone.conf
```

**Display QR code for mobile setup:**
```bash
pivpn -qr
```
Scan this QR code with the WireGuard mobile app (iOS/Android) to import the profile instantly.

**Pro Tip:** Create separate profiles for each device to maintain individual key pairs and enable granular revocation if a device is lost or compromised.

---

### Phase 4: Router Port Forwarding ![Router](https://img.shields.io/badge/-Router-4285F4?style=flat-square&logo=router)

**Why?** You must allow VPN traffic from the internet to reach your Pi. Without port forwarding, external clients cannot establish a connection.

**Steps:**
1. Log into your home router (typically `192.168.1.1` or `192.168.0.1` in a browser)
2. Find the **Port Forwarding**, **Virtual Server**, or **NAT** section
3. Add a new forwarding rule:

| Field | Value | Notes |
|-------|-------|-------|
| Name / Service | WireGuard | Easy label for identification |
| Protocol | **UDP** | WireGuard uses UDP exclusively |
| WAN Start Port | `51820` | External port for incoming connections |
| WAN End Port | `51820` | Same as start port |
| LAN Host IP | `192.168.1.x` | Your Pi's static IP |
| LAN Host Port | `51820` | Same as WAN port |

**After saving the rule:** Reboot the router

**Security Warning:** Only expose UDP 51820. Never forward SSH (22), RDP (3389), or other services directly to the internet — access them securely through the VPN tunnel instead.

---

### Phase 5: Server Configuration ![Config](https://img.shields.io/badge/-Server%20Config-00BCD4?style=flat-square)

Server config file location: `/etc/wireguard/wg0.conf`

This file is **auto-generated** by PiVPN when you add clients. Here's what a typical configuration looks like:

```ini
[Interface]
PrivateKey = <SERVER_PRIVATE_KEY>
Address = 10.155.159.1/24
ListenPort = 51820
MTU = 1420

# Phone (10.155.159.2)
[Peer]
PublicKey = <MOBILE_PUBLIC_KEY>
PresharedKey = <PSK>
AllowedIPs = 10.155.159.2/32

# PC (10.155.159.3)
[Peer]
PublicKey = <PC_PUBLIC_KEY>
PresharedKey = <PSK>
AllowedIPs = 10.155.159.3/32
```

**Critical:** Make sure you **do NOT** include `192.168.1.0/24` in the server's `AllowedIPs` field. The server config defines which IPs belong to each peer, not which destinations they can reach. Adding your LAN range here will break routing.

---

### Phase 6: Client Configuration ![Client](https://img.shields.io/badge/-Client%20Config-00BCD4?style=flat-square)

Each client needs a profile that tells WireGuard how to connect and what traffic to route through the tunnel.

**Mobile Client (Phone)**
```ini
[Interface]
PrivateKey = <MOBILE_PRIVATE_KEY>
Address = 10.155.159.2/24
DNS = 9.9.9.9, 149.112.112.112

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
PresharedKey = <PSK>
Endpoint = <YOUR_PUBLIC_IP>:51820
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
```

**Windows PC Client**
Download [WireGuard for Windows](https://www.wireguard.com/install/) and import this config:
```ini
[Interface]
PrivateKey = <PC_PRIVATE_KEY>
Address = 10.155.159.3/24
DNS = 9.9.9.9, 149.112.112.112

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
PresharedKey = <PSK>
Endpoint = <YOUR_PUBLIC_IP>:51820
AllowedIPs = 0.0.0.0/0, ::/0, 192.168.1.0/24
PersistentKeepalive = 25
```

**Why the difference?** Adding `192.168.1.0/24` to the PC's `AllowedIPs` ensures proper LAN discovery for apps like Remote Desktop, SMB shares, and network printers. Mobile clients often work without this because their VPN stack automatically routes LAN subnets under `0.0.0.0/0`.

---

### Phase 7: IP Forwarding & NAT Rules ![NAT](https://img.shields.io/badge/-NAT%20Forwarding-00BCD4?style=flat-square)

These settings allow your Pi to act as a router, forwarding VPN traffic to your LAN and back.

**Enable IP Forwarding:**
```bash
sudo nano /etc/sysctl.conf
```
Ensure these lines are present and **uncommented** (remove the `#` if present):
```ini
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```
Save the file (Ctrl+O, Enter, Ctrl+X) and apply the changes:
```bash
sudo sysctl -p
```

**Configure NAT & Traffic Forwarding**

These iptables rules allow bidirectional traffic between `wg0` (VPN interface) and `wlan0` (Wi-Fi interface). If your Pi uses Ethernet, replace `wlan0` with `eth0`.

**For Wi-Fi (wlan0):**
```bash
sudo iptables -A FORWARD -i wg0 -j ACCEPT
sudo iptables -A FORWARD -o wg0 -j ACCEPT
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
```

**For Ethernet (eth0):**
```bash
sudo iptables -A FORWARD -i wg0 -j ACCEPT
sudo iptables -A FORWARD -o wg0 -j ACCEPT
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

**Verify your interface name:**
```bash
ip a
```

**Make Rules Persistent:**
```bash
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```

**Restart WireGuard:**
```bash
sudo systemctl restart wg-quick@wg0
```

---

### Phase 8: Verify & Test ![Test](https://img.shields.io/badge/-Verification-00BCD4?style=flat-square)

**On the Raspberry Pi:**
```bash
sudo wg
```
Look for recent handshakes and non-zero transfer counters:
```
interface: wg0
  peer: <PC_PUBLIC_KEY>
    endpoint: ***.***.***.***:xxxxx
    allowed ips: 10.155.159.3/32
    latest handshake: 12 seconds ago
    transfer: 2.51 KiB received, 1.83 KiB sent
```

**Check both IP addresses are assigned:**
```bash
hostname -I
```
Should show: `192.168.1.x 10.155.159.1`

**From a VPN Client:**
```bash
ping 10.155.159.1
```
Tests connectivity to the Pi via VPN tunnel.

```bash
ping 192.168.1.x
```
Tests LAN IP accessibility through tunnel.

```bash
curl ifconfig.me
```
Should return your home's public IP (confirms full-tunnel routing).

**Optional: Static Route for True LAN Visibility**

If you want devices on your home LAN to directly see and initiate connections to VPN clients (e.g., ping `10.155.159.2` from a LAN PC), add a static route in your router:
- **Destination:** `10.155.159.0`
- **Subnet mask:** `255.255.255.0`
- **Gateway:** `192.168.1.x` (your Pi's IP)

Look for this in your router's **Advanced → Static Routing** section.

---

## AllowedIPs & Routing Modes ![Routing](https://img.shields.io/badge/-Routing%20Modes-FF6B35?style=flat-square)

WireGuard decides which traffic goes through the VPN using the `AllowedIPs` field in your client config. Understanding this setting is crucial for proper operation.

### Full-Tunnel Mode (`AllowedIPs = 0.0.0.0/0, ::/0`)

All network traffic — internet and LAN — is sent through the VPN tunnel to your Raspberry Pi. This provides complete encryption and makes your device appear to browse from your home IP.

```
Your Device → WireGuard Tunnel (Encrypted) → Pi VPN Server → Internet + LAN
```

**Advantages:**
- Maximum security and privacy
- Your IP appears as your home network
- Protects all traffic on untrusted networks
- Bypasses geographic restrictions

**Trade-offs:**
- Higher latency for all internet traffic
- Full dependency on Pi uptime
- Uses more home bandwidth
- May be slower on mobile data

### Split-Tunnel with LAN (`AllowedIPs = 0.0.0.0/0, ::/0, 192.168.1.0/24`)

This hybrid mode routes all traffic through the VPN (like full-tunnel) but explicitly includes the home LAN subnet. On Windows, adding `192.168.1.0/24` ensures proper routing table priority for LAN services.

```
Your Device → WireGuard Tunnel (Hybrid) → Pi + LAN Access
```

**Advantages:**
- Complete LAN device access (RDP, SMB, printers)
- Still encrypted and secure
- Works around Windows routing quirks
- Network discovery enabled

**Trade-offs:**
- Can cause routing conflicts if misconfigured
- Slightly more complex troubleshooting
- Still routes all internet through VPN

### LAN-Only Mode (`AllowedIPs = 10.155.159.0/24, 192.168.1.0/24`)

For users who only need access to home devices (not full internet through VPN), this mode keeps your regular internet connection local while tunneling only VPN and LAN traffic.

**Use Case Recommendations:**

| Use Case | Recommended Mode |
|----------|-----------------|
| Remote desktop + secure browsing on public Wi-Fi | Full-tunnel (0.0.0.0/0, ::/0) |
| Access NAS/IoT with home IP visible | Split-tunnel (0.0.0.0/0, ::/0, 192.168.1.0/24) |
| Mobile device with automatic routing | Split-tunnel (works without explicit LAN) |
| Only need home LAN access, keep internet local | LAN-only (10.155.159.0/24, 192.168.1.0/24) |

---

## Troubleshooting ![Troubleshoot](https://img.shields.io/badge/-Troubleshooting-FF3D00?style=flat-square)

**Common Issues & Solutions:**

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| VPN connects but can't reach internet | NAT not configured | Check iptables MASQUERADE rule on wlan0/eth0 |
| Can reach VPN IP but not LAN devices | Missing LAN subnet in AllowedIPs | Add 192.168.1.0/24 to client config |
| Handshake never completes | Port forwarding issue | Verify router forwards UDP 51820 to Pi's IP |
| Ping shows "General failure" on Windows | Routing table conflict | Ensure client AllowedIPs includes LAN subnet |
| Client can't connect at all | Public IP changed | Update Endpoint in client config with current public IP |
| Connection drops after few minutes | NAT timeout | Add PersistentKeepalive = 25 to client config |

**Troubleshooting Decision Tree:**

1. **Connects but no LAN access?**
   - Check server forwarding: `ip_forward=1` + NAT on wlan0
   - Verify client routes include LAN subnet
   - Ensure unique client IPs/keys
   - Windows "General failure"? Add LAN to AllowedIPs
   - Still blocked? Check Windows Firewall ICMP and verify iptables rules on Pi

**Most issues stem from:**
- Routing (`AllowedIPs` configuration)
- Forwarding (`sysctl` settings)
- Duplicate client IPs

---

## Operations & Day-2 Tips ![Ops](https://img.shields.io/badge/-Operations-00BCD4?style=flat-square)

### Common Management Commands

```bash
# Add new client profile
pivpn add

# Show QR code for mobile
pivpn -qr

# List all peers
pivpn list

# Revoke a client
pivpn remove

# Show handshakes & traffic
sudo wg

# Per-peer data transfer
sudo wg show wg0 transfer

# Restart WireGuard
sudo systemctl restart wg-quick@wg0

# Save iptables rules
sudo netfilter-persistent save
```

### Security Hardening

- Disable password SSH once key-based auth is configured
- Only expose UDP 51820; never forward SSH/RDP to internet
- Use `PersistentKeepalive = 25` for mobile clients behind NAT
- Regularly update: `sudo apt update && sudo apt upgrade`
- Monitor logs: `sudo journalctl -u wg-quick@wg0 -f`
- Use `fail2ban` for additional SSH protection

### Performance Monitoring

```bash
# Check data transfer per peer
sudo wg show wg0 transfer

# Live WireGuard logs
sudo journalctl -u wg-quick@wg0 -f

# Monitor CPU/RAM usage
htop

# Network statistics for wg0 interface
vnstat -i wg0
```

---

## Glossary ![Glossary](https://img.shields.io/badge/-Glossary-9C27B0?style=flat-square)

| Term | Definition |
|------|-----------|
| **AllowedIPs** | On client: which destinations route through the tunnel. On server: which IPs belong to each peer. |
| **MASQUERADE** | iptables NAT rule that rewrites source IP so LAN devices can reply to VPN clients. |
| **Handshake** | WireGuard key exchange event; must update periodically for a healthy tunnel (every 2 min by default). |
| **wg0** | The WireGuard network interface on your Pi (virtual tunnel adapter). |
| **wlan0 / eth0** | Physical network interface — wlan0 for Wi-Fi, eth0 for Ethernet. |
| **PersistentKeepalive** | Sends periodic packets to keep NAT mappings alive (essential for mobile clients). |
| **Endpoint** | The server's public IP:port that clients connect to. |
| **PSK (PresharedKey)** | Optional extra layer of symmetric encryption for quantum resistance. |
| **ChaCha20-Poly1305** | Modern authenticated encryption cipher used by WireGuard. |
| **Curve25519** | Elliptic curve used for key exchange in WireGuard. |

---

<div align="center">

**Last updated:** 2025

</div>


