# ================================
# ROUTER: R3-BHNET
# RouterOS: 7.20.8
# FUNÇÃO: Provedor / Peering + VPN + Cliente
# ================================

# === INTERFACES ===
/interface ethernet
set [ find default-name=ether4 ] name=ether1-Link-Claro comment="Peering com Claro (AS300)"
set [ find default-name=ether1 ] name=ether2-Link-Hotel comment="Rede Cliente Hotel"
set [ find default-name=ether2 ] name=ether3-LinkSpeed comment="Peering com LinkSpeed (AS100)"
set [ find default-name=ether3 ] name=ether4-Hotel-Backup comment="Link Backup Cliente"

# === WIREGUARD VPN ===
/interface wireguard
add name=Wireguard-Hotel listen-port=45781 mtu=1420 comment="VPN Site-to-Site Hotel"

/interface wireguard peers
add interface=Wireguard-Hotel name=peer-Hotel \
    endpoint-address=220.220.230.226 endpoint-port=45781 \
    allowed-address=0.0.0.0/0,::/0 persistent-keepalive=30s \
    comment="Peer VPN Hotel"

# === LISTAS ===
/interface list
add name=WAN

/interface list member
add interface=ether1-Link-Claro list=WAN
add interface=ether3-LinkSpeed list=WAN

# === ENDEREÇAMENTO ===
/ip address
add address=75.74.73.74/30 interface=ether1-Link-Claro comment="Link Claro"
add address=20.20.30.30/30 interface=ether3-LinkSpeed comment="Link LinkSpeed"
add address=220.220.230.225/29 interface=ether2-Link-Hotel comment="Rede Cliente"
add address=230.230.240.241/29 interface=ether4-Hotel-Backup comment="Backup Cliente"
add address=10.0.0.1/24 interface=Wireguard-Hotel comment="Rede VPN"

# === DHCP (BACKUP LINK) ===
/ip pool
add name=pool-backup ranges=230.230.240.242-230.230.240.246

/ip dhcp-server network
add address=230.230.240.240/29 gateway=230.230.240.241

# === BGP ===
/routing bgp instance
add name=BHNet-Instance as=200 router-id=2.2.2.2

/routing bgp connection
add name=to-Claro instance=BHNet-Instance \
    local.address=75.74.73.74 remote.address=75.74.73.73 \
    remote.as=300 role=ebgp

# === PREFIXOS ANUNCIADOS ===
/ip firewall address-list
add list=IPs-comerciaveis address=5.4.3.0/24
add list=IPs-comerciaveis address=9.8.7.0/24

# === ROTAS ===
/ip route
add dst-address=0.0.0.0/0 gateway=75.74.73.73 comment="Default via Claro"

# Blackhole para anúncio BGP
add dst-address=5.4.3.0/24 type=blackhole
add dst-address=9.8.7.0/24 type=blackhole

# === FILTROS BGP ===
/routing filter rule

# Importação
add chain=bgp-in rule="if (dst in 70.70.68.0/22) {accept}"
add chain=bgp-in rule="if (dst in 70.70.70.0/23 && dst-len in 23-24) {accept}"
add chain=bgp-in rule="if (dst in 200.2.0.0/22 && dst-len in 22-24) {accept}"
add chain=bgp-in rule="if (dst in 200.100.0.0/16) {accept}"

# Exportação
add chain=bgp-out rule="if (dst in 5.4.3.0/24) {accept}"
add chain=bgp-out rule="if (dst in 9.8.7.0/24) {accept}"

# === FIREWALL ===
/ip firewall filter
add chain=input action=accept protocol=tcp dst-port=443 comment="Permite SSTP VPN"

/ip firewall nat
add chain=srcnat action=masquerade out-interface-list=WAN

# === VPN SSTP (opcional) ===
/interface sstp-server server
set enabled=yes authentication=mschap1,mschap2

# === SISTEMA ===
/system identity
set name=BHNet

/system clock
set time-zone-name=America/Sao_Paulo

/system ntp client
set enabled=yes

/system ntp client servers
add address=a.ntp.br

/tool romon
set enabled=yes
