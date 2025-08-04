# 🛡️ Perimeter Security Lab — OpenWRT → RouterOS → Suricata (Inline IPS)

This lab builds a simple virtualized perimeter where all client traffic passes through:
OpenWRT → RouterOS (MikroTik) → Suricata IPS.  
The goal is to simulate real-world network inspection, NAT, routing, and inline threat prevention.

---

## 📐 Network Architecture

```plaintext
[Clients]
    ↓ (DHCP 10.10.1.0/24)
[OpenWRT Router]
    LAN: 10.10.1.1
    ↓
[RouterOS VM]
    ether1 (WAN): 10.10.1.100
    ether2 (LAN): 10.10.2.1
    ↓
[Suricata VM]
    enp0s8: 10.10.2.2
    ↓
[Internet via Wi-Fi (192.168.3.x)]

💡 IP Address Plan
Device	Interface	IP Address	Description
OpenWRT	br-lan	10.10.1.1/24	Gateway + DHCP server
RouterOS	ether1	10.10.1.100/24	WAN (OpenWRT connection)
RouterOS	ether2	10.10.2.1/24	LAN → Suricata
Suricata VM	enp0s8	10.10.2.2/24	Inline IPS interface

⚙️ RouterOS Configuration

/ip address add address=10.10.1.100/24 interface=ether1
/ip address add address=10.10.2.1/24 interface=ether2

/ip route add dst-address=0.0.0.0/0 gateway=10.10.1.1

/ip firewall nat add chain=srcnat out-interface=ether1 action=masquerade

🛠 Suricata Setup (Inline IPS Mode)
🧩 1. Install Suricata (on Ubuntu/Debian):
sudo apt update
sudo apt install suricata

🌐 2. Configure Interface:
sudo ip addr add 10.10.2.2/24 dev enp0s8
sudo ip link set enp0s8 up
sudo ip route add default via 10.10.2.1

3. Edit /etc/suricata/suricata.yaml
default-rule-path: /etc/suricata/rules
rule-files:
  - local.rules

af-packet:
  - interface: enp0s8
    copy-mode: ips
    disable-promisc: no
    use-mmap: yes
    checksum-checks: no

✅ 4. Test config & run
sudo suricata -T -c /etc/suricata/suricata.yaml
sudo systemctl enable --now suricata

🔥 Suricata Rules (/etc/suricata/rules/local.rules)
drop icmp any any -> any any (msg:"BLOCK ICMP"; sid:1000001; rev:1;)
alert http any any -> any any (msg:"HTTP Detected"; sid:1000002; rev:1;)

Check logs:
tail -f /var/log/suricata/fast.log

👤 Author
📡 HardPoint IT Lab




