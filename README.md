# Perimeter-Lab
# Network Security Perimeter Lab (OpenWRT → RouterOS → Suricata IPS)
This project builds a virtualized perimeter security lab using physical and virtual components.  
The goal is to pass all client traffic through a MikroTik (RouterOS) and Suricata in inline IPS mode.

---

## 🧱 Architecture
[Clients]
↓
[OpenWRT Router]
LAN: 10.10.1.1
↓
[RouterOS VM]
ether1: 10.10.1.100 (WAN side)
ether2: 10.10.2.1 (LAN to Suricata)
↓
[Suricata VM]
enp0s8: 10.10.2.2
↓
[Internet via Wi-Fi (192.168.3.x)]


🖼 See: [`diagrams/network_diagram.png`](./diagrams/network_diagram.png)  
🗂 Editable: [`diagrams/network_diagram.drawio`](./diagrams/network_diagram.drawio)

---

## 💡 IP Plan

| Device       | Interface         | IP Address      | Role                        |
|--------------|-------------------|-----------------|-----------------------------|
| OpenWRT      | br-lan            | 10.10.1.1/24     | DHCP + main gateway         |
| RouterOS     | ether1            | 10.10.1.100/24   | Connects to OpenWRT (WAN)   |
| RouterOS     | ether2            | 10.10.2.1/24     | Internal side to Suricata   |
| Suricata VM  | enp0s8            | 10.10.2.2/24     | Inline interface            |

---

## ⚙️ RouterOS Configuration


/ip address add address=10.10.1.100/24 interface=ether1
/ip address add address=10.10.2.1/24 interface=ether2

/ip route add dst-address=0.0.0.0/0 gateway=10.10.1.1
/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade

Suricata Setup
Install:

sudo apt update && sudo apt install suricata
Set interface IP:


sudo ip addr add 10.10.2.2/24 dev enp0s8
sudo ip link set dev enp0s8 up
Set default gateway:

sudo ip route add default via 10.10.2.1
Update /etc/suricata/suricata.yaml:

default-rule-path: /etc/suricata/rules
rule-files:
  - local.rules

af-packet:
  - interface: enp0s8
    copy-mode: ips
    disable-promisc: no
    use-mmap: yes
    checksum-checks: no
Test config:

sudo suricata -T -c /etc/suricata/suricata.yaml

sudo systemctl enable --now suricata
🔥 Sample Rules (local.rules)
rules

drop icmp any any -> any any (msg:"BLOCK ICMP"; sid:1000001; rev:1;)
alert http any any -> any any (msg:"HTTP Traffic Detected"; sid:1000002; rev:1;)

