# SIXCORE – Estudo técnico patrocinado pela Controllar Sistemas de Segurança e Acesso LTDA.

Nota:
Para fins de documentação, foi utilizada a ferramenta ChatGPT como apoio na descrição e organização dos scripts (backups). Todas as configurações, validações e implementações foram realizadas por mim acompanhando as aulas do professor Leonardo Vieira de Albuquerque (https://www.linkedin.com/in/albuquerqueleonardo/) na plataforma SixCore (https://sixcore.com.br/).

________________________________________________________________________________________

# 🏨 Projeto de Rede Corporativa – Hotel Multi-Operadora (MikroTik / EVE-NG)

## 📌 Visão Geral

Este projeto simula a infraestrutura completa de um hotel corporativo com:

* Multi-homing com BGP
* Segmentação por VLAN
* Redundância de links
* VPN site-to-site
* Hotspot para clientes
* Separação de redes (ADM / Clientes / Segurança)

Ambiente desenvolvido em laboratório utilizando **EVE-NG + MikroTik RouterOS v7**.

---

## 🧠 Arquitetura

### 🌐 Camada ISP

* **R1 – Claro (Core ISP)**
* **R2 – LinkSpeed**
* **R3 – BHNet**

✔️ BGP multi-homing
✔️ Anúncio e filtragem de rotas
✔️ IPv4 + IPv6

---

### 🏨 Borda do Cliente (Hotel)

#### 🔹 R4 – Hotel Primário

* Recebe links dos 3 provedores
* Balanceamento/failover via rotas
* VLANs internas (100/200/300)
* VPN (WireGuard / SSTP)

#### 🔹 R5 – Hotel Backup

* Link redundante
* Failover automático

---

### 🧠 Core de Rede

#### 🔹 R6 – Switch Core

* VLANs:

  * 100 → Administração
  * 200 → Clientes
  * 300 → Segurança
* Distribuição para switches de acesso

---

### 🔌 Camada de Acesso

#### 🔹 R7 – Switch Quartos

* VLAN 200 (Clientes)
* Portas access para quartos
* Uplink trunk para core

#### 🔹 R8 – Switch Segurança

* VLAN 300 (CFTV)
* Câmeras isoladas
* Sem acesso direto à internet

#### 🔹 R9 – Switch Administração

* VLAN 100
* PCs administrativos
* Rede interna protegida

---

### 📶 Camada de Usuário (Edge)

#### 🔹 R10 – APs por Quarto

* VLAN 200
* Hotspot (captive portal)
* NAT local
* Controle de acesso por usuário

---

## 🔐 Segmentação de Rede

| VLAN | Uso           | Subrede          |
| ---- | ------------- | ---------------- |
| 100  | Administração | 192.168.1.0/25   |
| 200  | Clientes      | 192.168.2.0/25   |
| 300  | Segurança     | 192.168.1.128/25 |

✔️ Isolamento entre redes
✔️ Controle de tráfego
✔️ Segurança por design

---

## 🔁 Redundância

* Multi-WAN (3 ISPs)
* Rotas com distância
* Backup via segundo roteador
* VPN entre sites

---

## 🌍 BGP

* Sessões eBGP entre ISPs
* Filtros de import/export
* Anúncio de prefixos próprios
* Uso de tabelas separadas (VRF-like)

---

## 🔐 Segurança

* Firewall com:

  * Drop de inválidos
  * Proteção contra port scan
* VLAN isolando:

  * Clientes
  * Administração
  * CFTV
* CFTV sem acesso direto à internet
* VPN segura entre pontos

---

## 📡 Hotspot

* Rede dedicada: `10.5.50.0/24`
* Autenticação de usuários
* NAT controlado
* Possibilidade de captive portal

---

## 📊 Boas Práticas Aplicadas

✔️ Separação por camadas (Core / Access / Edge)
✔️ Padronização de nomes
✔️ VLAN tagging correto (trunk vs access)
✔️ Redução de configs duplicadas
✔️ Organização para troubleshooting

---

## 🖼️ Diagrama (Draw.io)

Sugestão de organização visual:

* 🔵 Core (centro)
* 🟢 Access (embaixo)
* 🔴 ISPs (topo)
* 🟡 APs (bordas)

Labels importantes:

* “BGP Peering”
* “VLAN Trunk”
* “Client Network (Isolated)”
* “CFTV Network (Restricted)”

---

## 🔗 Estrutura no Repositório (GitHub)

```
lab-eveng-mikrotik/
│
├── R1-Claro/
├── R2-LinkSpeed/
├── R3-BHNet/
├── R4-Hotel-Primario/
├── R5-Hotel-Backup/
├── R6-Switch-Core/
├── R7-Switch-Quartos/
├── R8-Switch-Seguranca/
├── R9-Switch-ADM/
├── R10-AP-Quarto/
│
└── README.md
```

---

## 🚀 Objetivo do Projeto

Demonstrar conhecimento em:

* Redes corporativas
* Infraestrutura ISP
* MikroTik avançado
* BGP na prática
* Segmentação e segurança

---

## 💼 Aplicação Profissional

Este projeto é compatível com funções como:

* NOC (Nível 2 / 3)
* Analista de Redes
* Suporte ISP
* Infraestrutura corporativa
* Integrador de sistemas (CFTV + rede)

---

## 🧠 Conclusão

Este laboratório representa um ambiente realista de produção, integrando:

* Provedor (ISP)
* Cliente corporativo
* Acesso de usuários
* Segurança e segmentação

Projeto completo, organizado e pronto para apresentação técnica.

---
