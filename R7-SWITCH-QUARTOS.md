# ================================
# DEVICE: R7-SWITCH-QUARTOS
# RouterOS: 7.20.8
# FUNÇÃO: Switch de Acesso (Clientes / Quartos)
# ================================

# === BRIDGE ===
/interface bridge
add name=bridge-quartos vlan-filtering=yes comment="Bridge VLAN 200 (Quartos)"

# === INTERFACES ===
/interface ethernet
set [ find default-name=ether1 ] comment="Uplink (Trunk para Core)"
set [ find default-name=ether2 ] comment="Quarto 1"
set [ find default-name=ether3 ] comment="Quarto 2"
set [ find default-name=ether4 ] comment="Quarto 3"
set [ find default-name=ether5 ] comment="Quarto 4"
set [ find default-name=ether6 ] comment="Quarto 5"
set [ find default-name=ether7 ] comment="Quarto 6"
set [ find default-name=ether8 ] comment="Quarto 7"
set [ find default-name=ether9 ] comment="Quarto 8"
set [ find default-name=ether10 ] comment="Quarto 9"
set [ find default-name=ether11 ] comment="Quarto 10"
set [ find default-name=ether12 ] comment="Quarto 11"

# === PORTAS NA BRIDGE ===

# Uplink (TRUNK)
add bridge=bridge-quartos interface=ether1 comment="Trunk para Switch-Core"

# Access ports (clientes)
add bridge=bridge-quartos interface=ether2 pvid=200
add bridge=bridge-quartos interface=ether3 pvid=200
add bridge=bridge-quartos interface=ether4 pvid=200
add bridge=bridge-quartos interface=ether5 pvid=200
add bridge=bridge-quartos interface=ether6 pvid=200
add bridge=bridge-quartos interface=ether7 pvid=200
add bridge=bridge-quartos interface=ether8 pvid=200
add bridge=bridge-quartos interface=ether9 pvid=200
add bridge=bridge-quartos interface=ether10 pvid=200
add bridge=bridge-quartos interface=ether11 pvid=200
add bridge=bridge-quartos interface=ether12 pvid=200

# === VLAN ===
/interface bridge vlan
add bridge=bridge-quartos vlan-ids=200 \
    tagged=ether1 \
    untagged=ether2,ether3,ether4,ether5,ether6,ether7,ether8,ether9,ether10,ether11,ether12

# === GERENCIAMENTO ===
/ip dhcp-client
add interface=bridge-quartos comment="IP de gerenciamento"

# === SISTEMA ===
/system identity
set name=Switch-Quartos

/tool romon
set enabled=yes
