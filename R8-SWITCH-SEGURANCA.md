# ================================
# DEVICE: R8-SWITCH-SEGURANCA
# RouterOS: 7.20.8
# FUNÇÃO: Switch de Segurança (CFTV / VLAN 300)
# ================================

# === BRIDGE ===
/interface bridge
add name=bridge-seguranca vlan-filtering=yes comment="Bridge VLAN 300 (CFTV)"

# === INTERFACES ===
/interface ethernet
set [ find default-name=ether1 ] comment="Uplink (Trunk para Core)"
set [ find default-name=ether2 ] comment="Camera 1"
set [ find default-name=ether3 ] comment="Camera 2"
set [ find default-name=ether4 ] comment="Camera 3"
set [ find default-name=ether5 ] comment="Camera 4"
set [ find default-name=ether6 ] comment="Camera 5"
set [ find default-name=ether7 ] comment="Camera 6"
set [ find default-name=ether8 ] comment="Camera 7"
set [ find default-name=ether9 ] comment="Camera 8"
set [ find default-name=ether10 ] comment="Camera 9"
set [ find default-name=ether11 ] comment="Camera 10"
set [ find default-name=ether12 ] comment="Camera 11 / NVR"

# === PORTAS NA BRIDGE ===

# Uplink (TRUNK)
add bridge=bridge-seguranca interface=ether1 comment="Trunk para Switch-Core"

# Access ports (CFTV)
add bridge=bridge-seguranca interface=ether2 pvid=300
add bridge=bridge-seguranca interface=ether3 pvid=300
add bridge=bridge-seguranca interface=ether4 pvid=300
add bridge=bridge-seguranca interface=ether5 pvid=300
add bridge=bridge-seguranca interface=ether6 pvid=300
add bridge=bridge-seguranca interface=ether7 pvid=300
add bridge=bridge-seguranca interface=ether8 pvid=300
add bridge=bridge-seguranca interface=ether9 pvid=300
add bridge=bridge-seguranca interface=ether10 pvid=300
add bridge=bridge-seguranca interface=ether11 pvid=300
add bridge=bridge-seguranca interface=ether12 pvid=300

# === VLAN ===
/interface bridge vlan
add bridge=bridge-seguranca vlan-ids=300 \
    tagged=ether1 \
    untagged=ether2,ether3,ether4,ether5,ether6,ether7,ether8,ether9,ether10,ether11,ether12

# === GERENCIAMENTO ===
/ip dhcp-client
add interface=bridge-seguranca comment="IP de gerenciamento"

# === DHCP (OPCIONAL PARA CFTV) ===
/ip pool
add name=pool-cftv ranges=192.168.1.131-192.168.1.254

/ip dhcp-server
add name=dhcp-cftv interface=bridge-seguranca address-pool=pool-cftv disabled=yes

/ip dhcp-server network
add address=192.168.1.128/25 gateway=192.168.1.129 comment="Rede CFTV"

# === SISTEMA ===
/system identity
set name=Switch-Seguranca

/tool romon
set enabled=yes
