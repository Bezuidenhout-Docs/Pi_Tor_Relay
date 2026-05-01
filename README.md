# Tor Relay on Raspberry Pi 3 B

A low-cost, low-power Tor middle relay built with a Raspberry Pi 3 B. This project contributes bandwidth to the Tor network to help strengthen anonymous communication worldwide.

## 📡 My Relay

| Detail | Value |
|---|---|
| **Nickname** | `YourRelayNickname` |
| **Fingerprint** | `XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX` |
| **IP Address** | Public IP (auto-detected by Tor) |
| **ORPort** | `443` |
| **DirPort** | `9030` |
| **Relay Type** | Middle (non-exit) |
| **Flags** | None (awaiting testing and consensus) |
| **Status** | Running — pending reachability self-test |

## 🛠 Hardware & Software

| Component | Detail |
|---|---|
| **Device** | Raspberry Pi 3 Model B |
| **Architecture** | ARM64 (arm64) |
| **OS** | Raspberry Pi OS (Debian Trixie) |
| **Kernel** | Linux 6.12.75+rpt |
| **Tor Version** | 0.4.9.6 (recommended) |
| **Nyx Version** | 2.1.0 |
| **Memory Usage** | ~279 MB (30.8% of total) |

## ⚙️ Configuration

### Tor Configuration (`/etc/tor/torrc`)
Nickname YourRelayNickname
ContactInfo youremail@example.com

ControlPort 9051
CookieAuthentication 1

ORPort 443
ExitRelay 0
DirPort 9030

RelayBandwidthRate 100 KB
RelayBandwidthBurst 200 KB

ExitPolicy reject :

text

> **Replace `YourRelayNickname` and `youremail@example.com` with your own choices before running.**

### Firewall (UFW) Rules
22/tcp ALLOW IN Anywhere # SSH
443/tcp ALLOW IN Anywhere # Tor ORPort

text

### Port Forwarding

| Setting | Value |
|---|---|
| **Protocol** | TCP |
| **External Port** | 443 |
| **Internal Port** | 443 |
| **Internal IP** | Your Raspberry Pi's local IP |

> **Find your Pi's local IP:** `hostname -I`

## 📦 Installed Packages

| Package | Purpose |
|---|---|
| `tor` | Tor daemon |
| `nyx` | Terminal-based Tor monitor |
| `tor-geoipdb` | GeoIP database for Tor |
| `torsocks` | SOCKS proxy wrapper |
| `libtorsocks` | Library for torsocks |
| `python3-stem` | Python Tor controller library |
| `ufw` | Uncomplicated Firewall |

## 🏁 Relay Lifecycle Progress

- [x] System updated
- [x] Tor and Nyx installed
- [x] Firewall configured
- [x] Tor configuration applied
- [x] Control port enabled for Nyx
- [x] User added to `debian-tor` group
- [x] Port forwarding configured on router
- [ ] Reachability self-test passed
- [ ] Relay published in Tor consensus
- [ ] "Fast" and "Stable" flags earned
- [ ] "Guard" flag earned (8+ months)

## 📊 Monitoring

### Nyx (Terminal Monitor)

```bash
nyx
Page	Content
1	Bandwidth graphs, fingerprint, flags
2	Real-time log messages
3	Active connections to other relays
4	Tor configuration summary
5	Python interpreter
Key commands: m = menu, p = pause, h = help, q = quit

Tor Metrics
Once the relay is established, search for it at metrics.torproject.org using your relay's nickname or fingerprint.

Tor Logs
bash
sudo journalctl -u tor@default -f
🔧 Management Commands
bash
sudo systemctl restart tor     # Restart Tor
sudo systemctl enable tor      # Auto-start on boot
sudo systemctl status tor      # Check service status
sudo nyx                       # Monitor relay (with root fallback)
nyx                            # Monitor relay (as normal user)
🚨 Important Notes
This is a middle relay, not an exit relay. It passes encrypted traffic only. ExitRelay 0 and ExitPolicy reject *:* are explicitly set.

Your IP will be publicly listed as part of the Tor network. This is normal and expected for any relay.

Keep the system updated with sudo apt update && sudo apt upgrade -y.

Uptime is critical. The longer and more consistently your relay runs, the more trust it earns and the more valuable it becomes to the network.

Bandwidth limits (100 KB/s average, 200 KB/s burst) can be adjusted in /etc/tor/torrc if you have more capacity.

🧹 Troubleshooting
Problem	Solution
Nyx shows "Unable to connect to tor"	Ensure ControlPort 9051 and CookieAuthentication 1 are in /etc/tor/torrc, then restart Tor
Relay not reachable (self-test fails)	Check router port forwarding rule for TCP port 443 pointing to Pi's local IP
Permission errors in Nyx	Run sudo nyx or ensure user is in debian-tor group and session is refreshed
Tor won't start	Check logs with sudo journalctl -u tor@default -f
📚 Resources
Tor Project: Relay Setup Guide

Tor Metrics

Nyx Documentation

Good Bad ISPs for Tor Relays

📄 License
This project and documentation are dedicated to the public domain. Use it freely for any purpose.

Running a Tor relay is a meaningful contribution to internet freedom, privacy, and anti-censorship. Thank you for being part of the network.
