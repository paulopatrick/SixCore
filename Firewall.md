#SixCore #DominandoRedes

# Treinamento de Segurança MikroTik – Firewall

Oferecido pela empresa  
*Controllar Sistemas de Segurança e Acesso LTDA*  

Tutorial do instrutor oficial MikroTik  
**Leonardo Vieira de Albuquerque**  
LinkedIn: http://linkedin.com/in/albuquerqueleonardo/

---

# Índice

- [Equipamento utilizado](#equipamento-utilizado)
- [Topologia da rede](#topologia-da-rede)
- [Configuração INPUT](#configuração-input)
- [Port Knocking](#port-knocking-sequência-de-portas)
- [Configuração FORWARD](#configuração-forward)
- [Regras administrativas](#regras-administrativas)
- [Regra final de segurança](#regra-final-de-segurança)

---

# Equipamento utilizado

| Item | Descrição |
|-----|-----|
| Dispositivo | MikroTik hAP Lite |
| Modelo | RB941-2nD |
| Sistema | RouterOS 6.49.19 (Long-term) |

---

# Topologia da rede

Exemplo simplificado da rede protegida pelo firewall:

```
           INTERNET
               │
               │
          ┌───────────┐
          │  MikroTik │
          │  Firewall │
          └─────┬─────┘
                │
        ┌───────┴────────┐
        │                │
      LAN             Wi-Fi
  192.168.88.0/24     Clients
```

---

# Configuração **INPUT**

## 1 - Bloqueio de conexões inválidas

```bash
/ip firewall filter
add chain=input connection-state=invalid action=drop
```
## 2 - Conexões estabelecidas e relacionadas

```bash
/ip firewall filter
add chain=input connection-state=established,related action=accept
```
## 3 - Obtém IP origem de quem tenta acesso externo ao roteador

```bash
/ip firewall filter
add chain=input protocol=tcp src-port=0-1024,3389,5900 in-interface-list=WAN action=add-src-to-address-list address-list=Hackers timeout=7d
```
## 4 - Obtém IP origem de quem tenta PortScan

```bash
/ip firewall filter
add chain=input protocol=tcp psd=21,3s,3,1 action=add-src-to-address-list address-list=Hackers timeout=7d
```
## 5 - SSH Brute Force – Estágio 1

```bash
/ip firewall filter
add chain=input protocol=tcp dst-port=22 in-interface-list=WAN connection-state=new action=add-src-to-address-list address-list=SSH_estagio_1 timeout=1m
```
## 6 - SSH Brute Force – Estágio 2

```bash
/ip firewall filter
add chain=input protocol=tcp dst-port=22 in-interface-list=WAN connection-state=new src-address-list=SSH_estagio_1 action=add-src-to-address-list address-list=SSH_estagio_2 timeout=1m
```
## 7 - SSH Brute Force – Estágio 3
```bash
/ip firewall filter
add chain=input protocol=tcp dst-port=22 in-interface-list=WAN connection-state=new src-address-list=SSH_estagio_2 action=add-src-to-address-list address-list=Hackers timeout=7d
```
# Port Knocking (Sequência de portas)

## 8 - Fase 1

```bash
/ip firewall filter
add chain=input protocol=tcp dst-port=57153 action=add-src-to-address-list address-list="Sequencia-Portas-Fase1" timeout=10s
```
## 9 - Fase 2

```bash
/ip firewall filter
add chain=input protocol=tcp dst-port=32570 src-address-list="Sequencia-Portas-Fase1" action=add-src-to-address-list address-list="Sequencia-Portas-Fase2" timeout=10s
```
## 10 - Fase 3

```bash
/ip firewall filter
add chain=input protocol=udp dst-port=4851 src-address-list="Sequencia-Portas-Fase2" action=add-src-to-address-list address-list="Sequencia-Portas-Fase3" timeout=10s
```
## 11 - Fase 4

```bash
/ip firewall filter
add chain=input protocol=tcp dst-port=49001 src-address-list="Sequencia-Portas-Fase3" action=add-src-to-address-list address-list="Sequencia-Portas-Fase4" timeout=10s
```
## 12 - Fase 5

```bash
/ip firewall filter
add chain=input protocol=tcp dst-port=61013 src-address-list="Sequencia-Portas-Fase4" action=add-src-to-address-list address-list="Sequencia-Portas-Fase5" timeout=10s
```
## 13 - Liberação final

```bash
/ip firewall filter
add chain=input protocol=tcp dst-port=26012 src-address-list="Sequencia-Portas-Fase5" action=add-src-to-address-list address-list="Sequencia-Portas-Liberados" timeout=4h
```
---

# Configuração **FORWARD**

## 14 - FastTrack para conexões estabelecidas

```bash
/ip firewall filter
add chain=forward connection-state=established,related action=fasttrack-connection
```
## 15 - Bloqueio de conexões inválidas

```bash
/ip firewall filter
add chain=forward connection-state=invalid action=drop
```
## 16 - Aceitar conexões estabelecidas e relacionadas

```bash
/ip firewall filter
add chain=forward connection-state=established,related action=accept
```
## 17 - Captura de IP tentando acessar LAN pela WAN

```bash
/ip firewall filter
add chain=forward in-interface-list=WAN out-interface-list=LAN action=add-src-to-address-list address-list=Hackers timeout=7d
```
## 18 - Permitir acesso LAN → Internet

```bash
/ip firewall filter
add chain=forward in-interface-list=LAN out-interface-list=WAN action=accept
```

---

# Regras administrativas

## 19 - Permitir acesso LAN ao roteador (exceto portas administrativas)

```bash
/ip firewall filter
add chain=input protocol=tcp dst-port=!3596,2498,5124 in-interface-list=LAN action=accept
```
## 20 - Liberar WinBox e WebFig apenas para IP autorizado

```bash
/ip firewall filter
add chain=input protocol=tcp dst-port=3596,2498,5124 src-address-list="Sequencia-Portas-Liberados" in-interface-list=LAN action=accept
```

---

# Regra final de segurança

## 21 - Drop geral

```bash
/ip firewall filter
add chain=input action=drop
```

---

# Observação

Este laboratório tem como objetivo demonstrar técnicas de **hardening de firewall em MikroTik**, incluindo:

- proteção contra **Port Scan**
- proteção contra **Brute Force**
- **Port Knocking**
- bloqueio automático de IPs suspeitos

Novas regras poderão ser adicionadas conforme evolução dos estudos em **segurança de redes**.
