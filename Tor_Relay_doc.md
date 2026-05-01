This project sets up a middle relay — the safest type of Tor relay to run from home. Your relay passes encrypted traffic between other Tor nodes but never connects directly to the public internet as an "exit." This means no exit-related legal or abuse concerns.

Hardware: Raspberry Pi 3 Model B

OS: Raspberry Pi OS (Debian Trixie)

Tor Version: 0.4.9.6 (recommended)

Relay Type: Middle (non-exit)

Bandwidth Limit: 100 KB/s average, 200 KB/s burst

Prerequisites
Raspberry Pi 3 B with Raspberry Pi OS installed

SSH access enabled (for headless operation)

Access to your home router's admin page (for port forwarding)

A static local IP or DHCP reservation for your Pi (recommended)

A nickname for your relay and a contact email

Installation
1. Update Your System
bash
sudo apt update && sudo apt upgrade -y
2. Install Tor and Nyx
Nyx is a terminal-based monitor for watching relay activity.

bash
sudo apt install tor nyx -y
3. Configure the Firewall (UFW)
Allow SSH first so you don't lock yourself out, then open the Tor ORPort.

bash
sudo apt install ufw -y
sudo ufw allow 22
sudo ufw allow 443
sudo ufw enable
Verify with:

bash
sudo ufw status verbose
4. Find Your Pi's Local IP
bash
hostname -I
Note this address — you'll need it for port forwarding.

5. Configure Tor
Edit the Tor configuration file:

bash
sudo nano /etc/tor/torrc
Add the following lines at the bottom (replace YourRelayNickname and email):

text
Nickname YourRelayNickname
ContactInfo your-email@example.com

ControlPort 9051
CookieAuthentication 1

ORPort 443
ExitRelay 0
DirPort 9030

RelayBandwidthRate 100 KB
RelayBandwidthBurst 200 KB

ExitPolicy reject *:*
Save with Ctrl+X, Y, Enter.

What these settings do:

Setting	Purpose
Nickname	Your relay's public name on the Tor network
ContactInfo	Email the Tor Project can use to reach you about relay issues
ControlPort 9051	Allows Nyx to connect and monitor Tor
CookieAuthentication 1	Secure authentication for the control port
ORPort 443	The port your relay listens on (443 = standard HTTPS, rarely blocked)
ExitRelay 0	Explicitly prevents your relay from becoming an exit
DirPort 9030	Serves directory information to other relays
RelayBandwidthRate	Average bandwidth cap for sustained traffic
RelayBandwidthBurst	Maximum burst rate for short periods
ExitPolicy reject *:*	Blocks all exit traffic — critical for a middle relay
6. Add Your User to the Tor Group
This allows Nyx to monitor Tor without needing root.

bash
sudo usermod -aG debian-tor $USER
Log out and back in for this to take effect.

7. Restart Tor
bash
sudo systemctl restart tor
8. Set Up Port Forwarding on Your Router
For your relay to be reachable from the Tor network, you must forward port 443 to your Pi.

Log into your router's admin page (typically http://192.168.1.1 or http://192.168.0.1)

Find the "Port Forwarding" or "Virtual Server" section

Create a new rule:

External/Public Port: 443

Internal/Private IP: Your Pi's local IP address (from Step 4)

Internal/Private Port: 443

Protocol: TCP

Save and apply

9. Verify Your Relay is Working
Start Nyx:

bash
nyx
Inside Nyx, press the right arrow key to go to the log page (page 2). Look for:

text
Self-testing indicates your ORPort is reachable from the outside. Excellent. Publishing server descriptor.
This confirms your relay is properly configured and visible to the Tor network. The self-test can take up to 20 minutes after starting Tor.

Monitoring Your Relay
Nyx (Terminal-Based)
bash
nyx
Page 1: Bandwidth graphs, fingerprint, flags

Page 2: Real-time log messages

Page 3: Connections to other relays

Page 4: Tor configuration

Page 5: Python interpreter

Keys: m = menu, p = pause, q = quit

Tor Metrics Website
After your relay has been running for a few hours, you can look it up on Tor Metrics by searching for:

Your relay's nickname

Your relay's fingerprint (349D2968...)

This shows uptime, bandwidth contribution, and when your relay might be promoted to a Guard relay.

Enabling Automatic Start on Boot
bash
sudo systemctl enable tor
Your relay will now start automatically whenever your Pi boots.

Relay Lifecycle
Time Running	Milestone
0–3 hours	Self-tests, appears on Tor Metrics
3–72 hours	Gains "Fast" and "Stable" flags if bandwidth is sufficient
~68 days	Can receive the "Guard" flag (used as entry point for clients)
8+ months	Eligible for full Guard status (most trusted position)
Important Notes
Don't run an exit relay from home. This configuration explicitly prevents it (ExitRelay 0 and ExitPolicy reject *:*).

Bandwidth caps are adjustable. You can increase RelayBandwidthRate if you have a fast, unmetered connection.

Keep your system updated. Run sudo apt update && sudo apt upgrade regularly, especially for Tor updates.

Uptime matters. The longer your relay runs uninterrupted, the more valuable it becomes to the network.

Troubleshooting
"Unable to connect to tor" when running Nyx
Make sure you added ControlPort 9051 and CookieAuthentication 1 to /etc/tor/torrc and restarted Tor. Also ensure your user is in the debian-tor group and you've logged out and back in.

Relay not reachable (self-test fails)
Double-check your router's port forwarding rule

Ensure your Pi has a static local IP or DHCP reservation

Some ISPs block incoming connections — contact your ISP if the issue persists

Check that ufw is allowing port 443

Permission errors in Nyx
Run Nyx without sudo after adding your user to the debian-tor group. If you still have issues, sudo nyx works as a fallback.