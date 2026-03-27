# ================================
# DEVICE: R6-SWITCH-CORE
# RouterOS: 7.20.8
# FUNÇÃO: Switch L2 Core (VLAN Segmentation)
# ================================

# === BRIDGE ===
/interface bridge
add name=bridge-core vlan-filtering=yes priority=0 comment="Bridge principal com VLAN"

# === INTERFACES ===
/interface ethernet
set [ find default-name=ether1 ] comment="Access VLAN 200 - Quartos"
set [ find default-name=ether2 ] comment="Access VLAN 100 - ADM"
set [ find default-name=ether3 ] comment="Access VLAN 300 - Segurança"
set [ find default-name=ether11 ] comment="Trunk para roteador (Hotel Primário)"
set [ find default-name=ether12 ] comment="Trunk para roteador backup / expansão"

# === PORTAS NA BRIDGE ===
/interface bridge port

# Access ports
add bridge=bridge-core interface=ether1 pvid=200 comment="Quartos"
add bridge=bridge-core interface=ether2 pvid=100 comment="Administrativo"
add bridge=bridge-core interface=ether3 pvid=300 comment="Segurança"

# Trunks
add bridge=bridge-core interface=ether11 comment="Trunk principal"
add bridge=bridge-core interface=ether12 comment="Trunk secundário"

# Outras portas (default)
/interface bridge port
add bridge=bridge-core interface=ether4
add bridge=bridge-core interface=ether5
add bridge=bridge-core interface=ether6
add bridge=bridge-core interface=ether7
add bridge=bridge-core interface=ether8
add bridge=bridge-core interface=ether9
add bridge=bridge-core interface=ether10

# === VLANs ===
/interface bridge vlan

# VLAN 100 - ADM
add bridge=bridge-core vlan-ids=100 \
    tagged=ether11,ether12 \
    untagged=ether2

# VLAN 200 - QUARTOS
add bridge=bridge-core vlan-ids=200 \
    tagged=ether11,ether12 \
    untagged=ether1

# VLAN 300 - SEGURANÇA
add bridge=bridge-core vlan-ids=300 \
    tagged=ether11,ether12 \
    untagged=ether3

# === GERENCIAMENTO ===
/ip dhcp-client
add interface=bridge-core comment="IP de gerenciamento"

# === SISTEMA ===
/system identity
set name=Switch-Core

/system clock
set time-zone-name=America/Sao_Paulo

/system ntp client
set enabled=yes

/tool romon
set enabled=yes
