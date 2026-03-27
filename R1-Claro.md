# ================================
# ROUTER: R1-CLARO
# RouterOS: 7.20.8
# FUNÇÃO: BGP Edge (Dual Stack)
# ================================

# === INTERFACES ===
/interface ethernet
set [ find default-name=ether1 ] name=ether1-Link-Tier1 comment="Upstream Tier1"
set [ find default-name=ether2 ] name=ether2-Hotel comment="Rede Hotel (PPPoE)"
set [ find default-name=ether3 ] name=ether3-LinkSpeed comment="Operadora LinkSpeed"
set [ find default-name=ether4 ] name=ether4-BH-Net comment="Operadora BHNet"

# === VLANs ===
/interface vlan
add name=vlan1120 interface=ether3-LinkSpeed vlan-id=1120 comment="BGP Claro -> LinkSpeed"
add name=vlan1130 interface=ether4-BH-Net vlan-id=1130 comment="BGP Claro -> BHNet"

# === BGP ===
/routing bgp instance
add name=Claro-Instance as=300 router-id=3.3.3.3 routing-table=Rota-MG-1
add name=Claro-Instance-IPv6 as=301 router-id=3.3.3.4 routing-table=Rota-MG-1-IPv6

# === PEERS ===
/routing bgp connection
add name=to-LinkSpeed remote.address=185.184.183.182/32 as=100 \
    local.address=185.184.183.181 role=ebgp

add name=to-BHNet remote.address=75.74.73.74/32 as=200 \
    local.address=75.74.73.73 role=ebgp

# === FIREWALL (INPUT HARDENING) ===
/ip firewall filter
add chain=input action=accept connection-state=established,related comment="Permite conexões válidas"
add chain=input action=accept protocol=icmp comment="Permite ping"
add chain=input action=drop comment="Drop geral"

# === NAT ===
/ip firewall nat
add chain=srcnat action=masquerade out-interface-list=WAN

# === ROTAS ANUNCIADAS ===
/ip firewall address-list
add list=Rotas-divulgaveis address=70.70.68.0/22
add list=Rotas-divulgaveis address=70.70.70.0/23

# === IDENTIDADE ===
/system identity
set name=Claro
