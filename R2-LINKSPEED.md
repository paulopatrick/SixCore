# ================================
# ROUTER: R2-LINKSPEED
# RouterOS: 7.20.8
# FUNÇÃO: Provedor / Upstream (BGP Transit)
# ================================

# === INTERFACES ===
/interface ethernet
set [ find default-name=ether1 ] name=ether1-Link-Claro comment="Peering com Claro (AS300)"
set [ find default-name=ether2 ] name=ether2-Link-Hotel comment="Rede Cliente Hotel"
set [ find default-name=ether3 ] name=ether3-Link-BHNet comment="Peering com BHNet"
set [ find default-name=ether4 ] name=ether4-Hotel-Backup comment="Link Backup Hotel"

# === LISTAS ===
/interface list
add name=WAN

/interface list member
add interface=ether1-Link-Claro list=WAN
add interface=ether3-Link-BHNet list=WAN

# === ENDEREÇAMENTO IPv4 ===
/ip address
add address=185.184.183.182/30 interface=ether1-Link-Claro comment="Link Claro"
add address=20.20.30.29/30 interface=ether3-Link-BHNet comment="Link BHNet"
add address=150.150.160.161/29 interface=ether2-Link-Hotel comment="Cliente Hotel"
add address=150.150.140.137/29 interface=ether4-Hotel-Backup comment="Backup Hotel"

# === ENDEREÇAMENTO IPv6 ===
/ipv6 address
add address=2804:1::2/125 interface=ether1-Link-Claro comment="Peering IPv6 Claro"

# === BGP INSTANCES ===
/routing bgp instance
add name=LinkSpeed-v4 as=100 router-id=1.1.1.1
add name=LinkSpeed-v6 as=101 router-id=1.1.1.2

# === BGP PEERS ===
/routing bgp connection
add name=to-Claro instance=LinkSpeed-v4 \
    local.address=185.184.183.182 remote.address=185.184.183.181 \
    remote.as=300 role=ebgp

add name=to-Claro-v6 instance=LinkSpeed-v6 \
    local.address=2804:1::2 remote.address=2804:1::1 \
    remote.as=301 role=ebgp

# === PREFIXOS ANUNCIADOS ===
/ip firewall address-list
add list=Meu-Prefixo address=200.2.0.0/22
add list=Meu-Prefixo address=200.2.0.0/23
add list=Meu-Prefixo address=200.2.0.0/24

# === ROTAS ===
/ip route
add dst-address=0.0.0.0/0 gateway=185.184.183.181 comment="Default via Claro"

# Blackhole (boa prática para anúncio BGP)
add dst-address=200.2.0.0/22 type=blackhole
add dst-address=200.2.0.0/23 type=blackhole
add dst-address=200.2.0.0/24 type=blackhole

# === FILTROS BGP ===
/routing filter rule
add chain=Claro-v4-Export rule="if (dst in 200.2.0.0/22 && dst-len in 22-24) {accept}"

# Importa rotas específicas da Claro
add chain=Claro-v4-Import rule="if (dst == 200.100.10.0/24) {accept}"
add chain=Claro-v4-Import rule="if (dst == 200.100.20.0/24) {accept}"
add chain=Claro-v4-Import rule="if (dst == 200.100.30.0/24) {accept}"
add chain=Claro-v4-Import rule="if (dst == 200.100.40.0/24) {accept}"
add chain=Claro-v4-Import rule="if (dst == 0.0.0.0/0) {accept}"

# Importa rotas do cliente (Hotel)
add chain=Claro-v4-Import rule="if (dst in 70.70.68.0/22) {accept}"
add chain=Claro-v4-Import rule="if (dst in 70.70.70.0/23 && dst-len in 23-24) {accept}"

# === FIREWALL ===
/ip firewall filter
add chain=input action=accept protocol=tcp dst-port=179 comment="Permite BGP"

# === NAT ===
/ip firewall nat
add chain=srcnat action=masquerade out-interface-list=WAN

# === SISTEMA ===
/system identity
set name=LinkSpeed

/system clock
set time-zone-name=America/Sao_Paulo

/system ntp client
set enabled=yes

/system ntp client servers
add address=a.ntp.br
add address=b.ntp.br

/tool romon
set enabled=yes
