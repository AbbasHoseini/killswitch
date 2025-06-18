
# 🔒 Kill Switch for macOS to Prevent IP Leak When VPN Disconnects

This project provides a simple yet effective Kill Switch for macOS users who want to ensure their real IP address is not exposed if the VPN connection drops. The setup uses macOS’s built-in firewall system: **Packet Filter (pf)**.

## 📌 Requirements

- OS: macOS (tested on macOS Catalina and above)
- Administrator privileges
- A text editor (e.g., nano or Visual Studio Code)
- Active VPN connection (typically using interfaces like `utun0`, `utun4`)

## 🧰 Setup Instructions

### 1. Identify the VPN Interface

Run the following command in the terminal to list all interfaces:

```bash
ifconfig
```

Look for interfaces like `utun0`, `utun2`, or `utun4`. These are commonly used by VPNs. In this guide, we assume it’s `utun4`.

### 2. Create Firewall Configuration File

Create or edit the following file:

```bash
sudo nano /etc/pf.anchors/com.myvpn.killswitch
```

Insert the following rules (replace the VPN IP with your actual one):

```pf
# Block all by default
block all

# Allow DNS requests (for domain resolution)
pass out quick on en0 proto udp from any to any port 53 keep state
pass out quick on en0 proto tcp from any to any port 53 keep state

# Allow outgoing traffic to VPN server (replace with your VPN server's IP and port)
pass out quick on en0 proto tcp from any to 123.123.123.123 port 80 keep state
pass out quick on en0 proto udp from any to 123.123.123.123 port 80 keep state

# Allow ICMP traffic only through VPN (optional but recommended)
pass out quick on utun4 inet proto icmp from any to any keep state

# Allow all traffic through VPN interface
pass out quick on utun4 from any to any keep state
```

🔁 Note: `en0` is usually the Wi-Fi interface. If you use Ethernet or another interface, replace accordingly.

### 3. Link the File in `pf.conf`

Edit the main pf config file:

```bash
sudo nano /etc/pf.conf
```

Add the following lines at the top:

```pf
anchor "com.myvpn.killswitch"
load anchor "com.myvpn.killswitch" from "/etc/pf.anchors/com.myvpn.killswitch"
```

### 4. Validate Configuration

Before enabling, verify the config:

```bash
sudo pfctl -n -f /etc/pf.conf
```

### 5. Enable the Firewall

Apply and enable the rules:

```bash
sudo pfctl -f /etc/pf.conf
sudo pfctl -e
```

If pf is already enabled, only the `-f` flag is needed.

### 6. Test the Kill Switch

1. Connect to your VPN.
2. Disconnect the VPN.
3. Try to ping a public address:

```bash
ping google.com
```

✅ If correctly configured, there should be no outgoing traffic when VPN is down.

## 🚫 Disabling the Kill Switch

To temporarily disable:

```bash
sudo pfctl -d
```

To remove it permanently, delete the anchor lines from `/etc/pf.conf` and remove the config file.

## ⚠️ Security Notes

- Ensure the VPN server IP is correct.
- Correctly identify your VPN interface (`utunX`).
- This Kill Switch only controls outgoing traffic at the system level.

## 📄 License

This project is released under the MIT License.
