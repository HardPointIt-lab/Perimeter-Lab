# Perimeter-Lab
Perimeter-Lab — a hands-on network security lab with Suricata IPS/IDS, Grafana dashboards, DNS-over-HTTPS on OpenWrt, and RouterOS routing. All client traffic passes through the IPS, while DNS queries are encrypted. Includes configs and dashboards for learning and practice.


/interface ethernet
set [ find default-name=ether1 ] name=LAN
set [ find default-name=ether2 ] name=WAN

/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik

/ip address
add address=192.168.1.2/24 interface=LAN network=192.168.1.0
add address=192.168.1.3/24 interface=WAN network=192.168.1.0

/ip dhcp-client
add disabled=no interface=LAN

/ip firewall nat
add action=masquerade chain=srcnat out-interface=WAN

/ip route
add distance=1 gateway=192.168.1.1



🔒 OpenWrt: Configuring DNS-over-HTTPS with https-dns-proxy

To enable DNS-over-HTTPS on OpenWrt you first install the required packages by running opkg install https-dns-proxy luci-app-https-dns-proxy, which will automatically pull in dependencies such as libcares, libnghttp2, libcurl4, libev and resolveip. Once the installation is complete, the system will configure and start https-dns-proxy and its LuCI interface.

After that, configure https-dns-proxy to use Cloudflare as the DoH provider by typing uci set https-dns-proxy.@https-dns-proxy[0].resolver='cloudflare' and then specify the bootstrap DNS servers with uci set https-dns-proxy.@https-dns-proxy[0].bootstrap_dns='1.1.1.1,1.0.0.1'. Save the configuration using uci commit https-dns-proxy and restart the service with /etc/init.d/https-dns-proxy restart. At this point the proxy will start and dnsmasq will be updated to integrate with it.

Next you need to adjust dnsmasq so it forwards all queries to the local DoH proxy instead of using the system resolvers. Do this by setting uci set dhcp.@dnsmasq[0].noresolv='1' and adding the local proxy address with uci add_list dhcp.@dnsmasq[0].server='127.0.0.1#5053'. Commit the changes using uci commit dhcp and then restart dnsmasq with /etc/init.d/dnsmasq restart. During this restart you might briefly see messages like udhcpc: broadcasting discover or no lease, failing, which are normal and not a sign of misconfiguration.

Finally, test the setup with nslookup google.com. If the configuration is correct, the result will show that the DNS server in use is 127.0.0.1:53, which confirms that queries are being forwarded through dnsmasq to https-dns-proxy and ultimately resolved by Cloudflare over DoH. At this point DNS-over-HTTPS is fully operational on your OpenWrt router.
