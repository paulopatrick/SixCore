# ================================
# ROUTER: R5-HOTEL-BACKUP
# RouterOS: 7.20.8
# FUNÇÃO: Cliente Backup + VPN (Failover do Hotel Primário)
# ================================

# === INTERFACES ===
/interface ethernet
set [ find default-name=ether1 ] name=ether1-LinkSpeed comment="Link LinkSpeed"
set [ find default-name=ether2 ] name=ether2-BHNet comment="Link BHNet"
set [ find default-name=ether3 ] name=ether3-Core comment="Switch Core"
set [ find default-name=ether4 ] disabled=yes comment="Não utilizado"

# === VPN CLIENTS ===

# PPTP (legado)
/interface pptp-client
add name=pptp-HotelPrimario \
    connect-to=150.150.160.162 \
    user=HotelPrimario \
    comment="VPN PPTP (legado) para Hotel Primário"

# SSTP (recomendado)
/interface sstp-client
add name=sstp-HotelPrimario \
    connect-to=150.150.160.162 \
    user=HotelPrimario \
    profile=default-encryption \
    comment="VPN SSTP segura para Hotel Primário"

# === LISTAS ===
/interface list
add name=WAN

/interface list member
add interface=ether1-LinkSpeed list=WAN
add interface=ether2-BHNet list=WAN

# === ENDEREÇAMENTO ===
/ip address
add address=150.150.140.138/29 interface=ether1-LinkSpeed comment="LinkSpeed"
add address=230.230.240.242/29 interface=ether2-BHNet comment="BHNet Backup"

# === ROTAS (FAILOVER) ===
/ip route
add dst-address=0.0.0.0/0 gateway=150.150.140.137 distance=1 comment="Primário LinkSpeed"
add dst-address=0.0.0.0/0 gateway=230.230.240.241 distance=2 comment="Backup BHNet"

# === NAT ===
/ip firewall nat
add chain=srcnat action=masquerade out-interface-list=WAN

# === SISTEMA ===
/system identity
set name=Hotel-Backup

/tool romon
set enabled=yes
