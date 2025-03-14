# Homelab: Secure Wireless Network & Access Point

## Overview
This homelab project is designed to enhance **network security, segmentation, and control** by implementing multiple SSIDs. The key objectives are:

1. **Subnetting and Network Segmentation**: Creating a guest network that is fully isolated from the main network.
2. **Centralized Network Traffic Management**: Ensuring that all devices route through a dedicated server before reaching the router.
3. **Self-Hosted Cloud Storage**: Setting up a Google Drive alternative with remote access.
4. **Remote Desktop Access**: Enabling seamless, worldwide access to a home PC via a Chromebook.

## Tech Stack
- **Hardware:**
  - Router: `[Model]`
  - Server: `[Laptop/Raspberry Pi]`
  - WiFi Adapter: TP-Link Archer T2U
- **Software:**
  - Ubuntu 22.04
  - Hostapd (Wi-Fi Access Point Management)
  - Dnsmasq (DHCP & DNS Server)
  - Netplan (Network Configuration)
  - Iptables/UFW (Firewall Rules)

---

## Network Diagram
![Network Diagram](network_topology.png)
*(Use [draw.io](https://app.diagrams.net/) or [Excalidraw](https://excalidraw.com/) to create a diagram and save it as `network_topology.png`.)*

---

## Objectives
- **Secure Wi-Fi Network with Multiple SSIDs**  
- **Guest Network Isolation**  
- **All Traffic Routed Through the Server**  
- **Static IP for Server, Dynamic for Clients**  
- **Self-Hosted Cloud Storage & Remote Desktop Access**  

---

## Setup & Configuration
### 1. Hostapd - Wi-Fi Access Point
Install and enable Hostapd:
```bash
sudo apt install hostapd
sudo systemctl enable hostapd
```

#### `/etc/hostapd/hostapd.conf`
```ini
interface=wlp2s0
ssid=MainNetwork
hw_mode=g
channel=7
wpa=2
wpa_passphrase=YourStrongPassword
bss=wlp2s0_40
ssid=GuestNetwork
wpa_passphrase=GuestPassword
```

Restart Hostapd:
```bash
sudo systemctl restart hostapd
```

---

### 2. Dnsmasq - DHCP & DNS Server
Install Dnsmasq:
```bash
sudo apt install dnsmasq
```

#### `/etc/dnsmasq.conf`
```ini
interface=wlp2s0
# DHCP for Guest Network
dhcp-range=192.168.40.2,192.168.40.100,255.255.255.0,12h
dhcp-option=3,192.168.40.1
dhcp-option=6,8.8.8.8,8.8.4.4

# DNS settings
server=8.8.8.8
server=8.8.4.4
no-resolv
bind-interfaces
```

Restart Dnsmasq:
```bash
sudo systemctl restart dnsmasq
```

---

### 3. Firewall & NAT Configuration
Allow traffic from the guest network through the server:
```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o wlp2s0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlp2s0 -o eth0 -j ACCEPT
```

Make iptables rules persistent:
```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

---

## Troubleshooting & Debugging
### Check if the Access Point is Running
```bash
sudo systemctl status hostapd
sudo systemctl status dnsmasq
```
### Check Network Interfaces
```bash
ip a
```
### View Firewall Rules
```bash
sudo iptables -L -v -n
```

---

## Lessons Learned & Future Plans
- Understanding **how multiple SSIDs work** without VLANs  
- Configuring **DHCP and DNS for subnet isolation**  
- Managing **firewall rules for security**  

### Next Steps:
- Implement **Self-Hosted Cloud Storage with Remote Access**
- Enable **Remote Desktop for Seamless Chromebook-to-PC Access**
- Experiment with **OpenWRT for Advanced AP Control**

---

## References & Further Reading
- [Hostapd Documentation](https://w1.fi/hostapd/)
- [Dnsmasq Documentation](http://www.thekelleys.org.uk/dnsmasq/doc.html)
- [Iptables Guide](https://netfilter.org/documentation/HOWTO/)

---

## Author
*Created & maintained by Kade van der Merwe.*  
Email: *kadevdm@gmail.com*  



