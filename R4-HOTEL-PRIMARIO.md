# ================================
# ROUTER: R4-HOTEL-PRIMARIO
# RouterOS: 7.20.8
# FUNÇÃO: Cliente Multi-WAN + VLAN + VPN
# ================================

# === INTERFACES ===
/interface ethernet
set [ find default-name=ether1 ] name=ether1-Link-Claro comment="PPPoE Claro"
set [ find default-name=ether2 ] name=ether2-LinkSpeed comment="Link LinkSpeed"
set [ find default-name=ether3 ] name=ether3-Link-BHNet comment="Link BHNet"
set [ find default-name=ether4 ] name=ether4-Core comment="Switch Core"

# === PPPoE (LINK PRINCIPAL) ===
/interface pppoe-client
add name=pppoe-Claro interface=ether1-Link-Claro \
    user=HotelPrimario add-default-route=yes \
    use-peer-dns=yes comment="Link principal Claro"

# === VLANs (SEGMENTAÇÃO INTERNA) ===
/interface vlan
add name=vlan100-ADM interface=ether4-Core vlan-id=100
add name=vlan200-Quartos interface=ether4-Core vlan-id=200
add name=vlan300-Seguranca interface=ether4-Core vlan-id=300

# === WIREGUARD VPN ===
/interface wireguard
add name=wireguard-Hotel listen-port=45781 mtu=1420 comment="VPN com BHNet"

/interface wireguard peers
add interface=wireguard-Hotel name=peer-BHNet \
    allowed-address=0.0.0.0/0,::/0 persistent-keepalive=30s \
    comment="Peer BHNet"

# === LISTAS ===
/interface list
add name=WAN

/interface list member
add interface=pppoe-Claro list=WAN
add interface=ether2-LinkSpeed list=WAN
add interface=ether3-Link-BHNet list=WAN

# === ENDEREÇAMENTO ===
/ip address
add address=150.150.160.162/29 interface=ether2-LinkSpeed comment="LinkSpeed"
add address=220.220.230.226/29 interface=ether3-Link-BHNet comment="BHNet"

# VLANs internas
add address=192.168.1.1/25 interface=vlan100-ADM comment="ADM"
add address=192.168.1.129/25 interface=vlan300-Seguranca comment="Segurança"
add address=192.168.2.126/25 interface=vlan200-Quartos comment="Quartos"

# VPN
add address=10.0.0.1/24 interface=wireguard-Hotel comment="Rede VPN"

# === DHCP ===
/ip pool
add name=pool-adm ranges=192.168.1.2-192.168.1.126
add name=pool-seguranca ranges=192.168.1.130-192.168.1.254
add name=pool-quartos ranges=192.168.2.1-192.168.2.125

/ip dhcp-server
add name=dhcp-adm interface=vlan100-ADM address-pool=pool-adm
add name=dhcp-seguranca interface=vlan300-Seguranca address-pool=pool-seguranca
add name=dhcp-quartos interface=vlan200-Quartos address-pool=pool-quartos

/ip dhcp-server network
add address=192.168.1.0/25 gateway=192.168.1.1
add address=192.168.1.128/25 gateway=192.168.1.129
add address=192.168.2.0/25 gateway=192.168.2.126

# === ROTAS (FAILOVER) ===
/ip route
add dst-address=0.0.0.0/0 gateway=pppoe-Claro distance=1 comment="Primário"
add dst-address=0.0.0.0/0 gateway=150.150.160.161 distance=2 comment="Backup LinkSpeed"
add dst-address=0.0.0.0/0 gateway=220.220.230.225 distance=3 comment="Backup BHNet"

# === FIREWALL ===
/ip firewall filter
add chain=input action=accept connection-state=established,related comment="Conexões válidas"

# VPN
add chain=input action=accept protocol=udp dst-port=45781 comment="WireGuard"
add chain=input action=accept protocol=tcp dst-port=443 comment="SSTP"
add chain=input action=accept protocol=tcp dst-port=1723 comment="PPTP"
add chain=input action=accept protocol=gre comment="GRE PPTP"

# Proteção básica
add chain=input action=drop connection-state=invalid comment="Drop inválidos"

# Isolamento de VLAN
add chain=forward action=drop \
    src-address=192.168.2.0/25 dst-address=192.168.1.128/25 \
    comment="Isola Quartos da VLAN Segurança"

# === NAT ===
/ip firewall nat
add chain=srcnat action=masquerade out-interface-list=WAN

# === SERVIÇOS (HARDENING) ===
/ip service
set ftp disabled=yes
set telnet disabled=yes
set api disabled=yes
set api-ssl disabled=yes
set ssh port=15748
set winbox port=47511
set www port=26423
set www-ssl port=36589

# === MONITORAMENTO ===
/snmp
set enabled=yes

# === SISTEMA ===
/system identity
set name=Hotel-Primario

/system clock
set time-zone-name=America/Sao_Paulo

/system ntp client
set enabled=yes

/system ntp client servers
add address=a.ntp.br

/tool romon
set enabled=yes
