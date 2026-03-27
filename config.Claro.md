# 2026-03-27 18:24:15 by RouterOS 7.20.8
# system id = xgu6FgUJZ4E
#
/interface ethernet
set [ find default-name=ether1 ] comment=Link-Tier1 disable-running-check=no \
    name=ether1-Link-Tier1
set [ find default-name=ether2 ] comment=LInk-Hotel disable-running-check=no \
    name=ether2-Hotel
set [ find default-name=ether3 ] comment=Link-LinkSpeed disable-running-check=\
    no name=ether3-LinkSpeed
set [ find default-name=ether4 ] comment=LInk-BH-Net disable-running-check=no \
    name=ether4-BH-Net
/interface vlan
add comment=BGP-Claro->LinkSpeed interface=ether3-LinkSpeed name=vlan1120 \
    vlan-id=1120
add comment=BGP-Claro->BHNet interface=ether4-BH-Net name=vlan1130 vlan-id=\
    1130
/interface list
add name=CGNAT-HOTEL
add name=WAN
/port
set 0 name=serial0
/routing table
add disabled=no fib name=Rota-MG-1
add disabled=no fib name=Rota-MG-1-IPv6
/routing bgp instance
add as=300 disabled=no name=Claro-Instance router-id=3.3.3.3 routing-table=\
    Rota-MG-1
add as=301 disabled=no name=Claro-Instance-IPv6 router-id=3.3.3.4 \
    routing-table=Rota-MG-1-IPv6
/routing bgp template
add as=300 disabled=no name=BGP-Claro output.redistribute=connected,static \
    routing-table=Rota-MG-1
add afi=ipv6 as=301 disabled=no name=BGP-Claro-IPv6 output.filter-chain=\
    bgp-out-v6 .network=Rotas-Anunciaveis-v6 routing-table=Rota-MG-1-IPv6
/snmp community
set [ find default=yes ] disabled=yes
add addresses=192.168.1.27/32 name=SNMP_mikrotik security=private
/system logging action
add disk-file-count=5 disk-file-name=critico.log disk-lines-per-file=10000 \
    name=DIsco target=disk
add disk-file-count=5 disk-file-name=nat.log disk-lines-per-file=10000 name=\
    DIscoNAT target=disk
/interface list member
add interface=ether2-Hotel list=CGNAT-HOTEL
add interface=ether1-Link-Tier1 list=WAN
/interface pppoe-server server
add authentication=chap,mschap1,mschap2 comment=To-Hotel disabled=no \
    interface=ether2-Hotel max-mru=1492 max-mtu=1492 one-session-per-host=yes \
    service-name=PPPoE-Hotel
/ip address
add address=185.184.183.181/30 comment=LinkSpeed interface=ether3-LinkSpeed \
    network=185.184.183.180
add address=75.74.73.73/30 comment=BH-Net interface=ether4-BH-Net network=\
    75.74.73.72
/ip dhcp-client
add interface=ether1-Link-Tier1
/ip firewall address-list
add address=70.70.70.0/24 list=Rotas-divulgaveis
add address=70.70.68.0/22 list=Rotas-divulgaveis
add address=70.70.70.0/23 list=Rotas-divulgaveis
/ip firewall filter
add action=accept chain=input comment="Estabelecidas e relacionadas" \
    connection-state=established,related
add action=accept chain=input comment="Aceita Zabbix" dst-port=161,162 \
    protocol=udp src-address=192.168.1.27
add action=accept chain=input comment="Aceita ping" connection-limit=100,32 \
    protocol=icmp
add action=accept chain=input comment=----------------BGP-Operadora dst-port=\
    179 in-interface=ether1-Link-Tier1 protocol=tcp src-address=192.168.88.1
add action=accept chain=input comment="Aceita ping " protocol=icmp
add action=add-dst-to-address-list address-list=PortScan address-list-timeout=\
    1w chain=input comment="PortScan Detective" in-interface-list=WAN log=yes \
    protocol=tcp psd=21,3s,3,1
add action=add-src-to-address-list address-list=PortScan address-list-timeout=\
    1w chain=input comment="\"Pega malandro\"" dst-port=20-25,3389,5900 log=\
    yes protocol=tcp
add action=drop chain=input comment="Recusa pacotes invalidos" \
    connection-state=invalid in-interface-list=WAN log=yes
add action=drop chain=input comment="DROP GERAL"
/ip firewall nat
add action=masquerade chain=srcnat log=yes log-prefix=NAT out-interface-list=\
    WAN
/ip firewall raw
add action=drop chain=prerouting comment="Derruba os portscans" log=yes \
    src-address-list=PortScan
/ip route
add blackhole comment="BGP - Rota 1" disabled=no dst-address=200.100.10.0/24 \
    gateway="" routing-table=Rota-MG-1 suppress-hw-offload=no
add blackhole comment="BGP - Rota 2" disabled=no distance=1 dst-address=\
    200.100.20.0/24 gateway="" routing-table=Rota-MG-1 suppress-hw-offload=no
add blackhole comment="BGP - Rota 3" disabled=no dst-address=200.100.30.0/24 \
    gateway="" routing-table=Rota-MG-1 suppress-hw-offload=no
add blackhole comment="BGP - Rota 0" disabled=no dst-address=0.0.0.0/0 \
    gateway="" routing-table=Rota-MG-1 suppress-hw-offload=no
add blackhole comment="BGP - Rota 4" disabled=no dst-address=200.100.40.0/24 \
    gateway="" routing-table=Rota-MG-1 suppress-hw-offload=no
add blackhole comment=Rota-divulgavel-1 disabled=no distance=1 dst-address=\
    70.70.68.0/22 gateway="" routing-table=Rota-MG-1 suppress-hw-offload=no
add blackhole comment=Rota-divulgavel-2 disabled=no distance=1 dst-address=\
    70.70.70.0/23 gateway="" routing-table=Rota-MG-1 suppress-hw-offload=no
add blackhole comment=Rota-divulgavel-3 disabled=no distance=1 dst-address=\
    70.70.70.0/24 gateway="" routing-table=Rota-MG-1 suppress-hw-offload=no
/ipv6 address
add address=2804:1::1/125 advertise=no comment=To-LinkSpeed interface=\
    ether3-LinkSpeed
/ipv6 firewall address-list
add address=2804:2::/32 list=Rotas-Anunciaveis-v6
/ppp secret
add local-address=100.64.0.1 name=ClaroHotel remote-address=100.64.0.2 \
    service=pppoe
/routing bgp connection
add as=300 connect=yes disabled=no input.filter=Claro-v4-Import instance=\
    Claro-Instance listen=yes local.address=185.184.183.181 .role=ebgp name=\
    to-LinkSpeed output.filter-chain=Claro-v4-Export .network=\
    Rotas-divulgaveis .redistribute=connected,static remote.address=\
    185.184.183.182/32 .as=100 routing-table=Rota-MG-1 templates=BGP-Claro
add as=300 connect=yes disabled=no instance=Claro-Instance listen=yes \
    local.address=75.74.73.73 .role=ebgp name=to-BHNet output.redistribute=\
    connected,static remote.address=75.74.73.74/32 .as=200 routing-table=\
    Rota-MG-1 templates=BGP-Claro
add as=301 connect=yes disabled=no instance=Claro-Instance-IPv6 listen=yes \
    local.address=2804:1::1 .role=ebgp name=to-LinkSpeed-IPv6 remote.address=\
    2804:1::2/128 .as=101 routing-table=Rota-MG-1-IPv6 templates=\
    BGP-Claro-IPv6
/routing filter rule
add chain=Claro-v4-Import disabled=no rule=\
    "if (dst in 200.2.0.0/22 && dst-len in 22-24) {accept}"
add chain=Claro-v4-Export disabled=no rule=\
    "if (dst in 200.2.0.0/22 && dst-len in 22-24) {accept} "
add chain=Claro-v4-Export disabled=no rule=\
    "if (dst in 200.100.10.0/24) {accept}"
add chain=Claro-v4-Export disabled=no rule=\
    "if (dst in 200.100.20.0/24) {accept}"
add chain=Claro-v4-Export disabled=no rule=\
    "if (dst in 200.100.30.0/24) {accept}"
add chain=Claro-v4-Export disabled=no rule=\
    "if (dst in 200.100.40.0/24) {accept}"
add chain=Claro-v4-Export disabled=no rule=\
    "if (dst in 70.70.70.0/23 && dst-len in 23-24) {accept}"
add chain=Claro-v4-Export disabled=no rule=\
    "if (dst in 70.70.68.0/22) {accept}"
/snmp
set contact=paulopatrick@proton.me enabled=yes location=MG/BR trap-community=\
    SNMP_mikrotik trap-version=3
/system clock
set time-zone-autodetect=no time-zone-name=America/Sao_Paulo
/system identity
set name=Claro
/system logging
add action=DIsco topics=critical
add action=DIsco topics=firewall
add action=DIscoNAT prefix=NAT topics=firewall
/system ntp server
set enabled=yes multicast=yes
/tool graphing interface
add interface=*F00000
/tool romon
set enabled=yes
