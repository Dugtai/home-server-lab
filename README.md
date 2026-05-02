# Home Server Lab

Домашняя серверная инфраструктура на базе мини-ПК Firebat S1 с Intel N100, Proxmox VE, OpenWrt, AdGuard Home, выборочной маршрутизацией трафика, VPN и anti-DPI обработкой для YouTube/Discord.

Проект оформляется как лабораторный стенд для портфолио: с документацией, схемами, примерами конфигураций и поэтапным описанием внедрения.

## Цель проекта

Создать управляемую домашнюю сеть с централизованной фильтрацией DNS, маршрутизацией трафика, изоляцией сервисов и возможностью дальнейшего расширения под self-hosted сервисы.

Основные задачи:

- развернуть Proxmox VE на bare metal;
- использовать OpenWrt как виртуальный основной шлюз;
- вынести DNS-фильтрацию в отдельный LXC-контейнер с AdGuard Home;
- настроить выборочную маршрутизацию трафика;
- направлять YouTube и Discord через anti-DPI обработку без полного VPN;
- направлять отдельные заблокированные сервисы через VPN;
- оставить обычный трафик напрямую через провайдера;
- подготовить основу для чатбота, Nextcloud, мониторинга и резервного копирования.

## Аппаратная платформа

| Компонент | Модель / параметры |
|---|---|
| Mini PC | Firebat S1 |
| CPU | Intel N100 |
| RAM | 16 GB |
| SSD | 512 GB |
| Hypervisor | Proxmox VE |
| Access Point | Tenda i29 AX3000 Wi-Fi 6 |
| Network | 2 × Ethernet |

## Целевая архитектура

```text
Optical terminal / ISP router
        ↓
Bridge mode or temporary NAT mode
        ↓
Firebat S1 / Proxmox VE
        ↓
OpenWrt VM as main gateway
        ↓
Tenda i29 AX3000 as Wi-Fi access point
        ↓
Home devices
```

## Виртуальная инфраструктура

```text
Proxmox VE
├── VM 100: OpenWrt Gateway
│   ├── WAN
│   ├── LAN
│   ├── Firewall
│   ├── DHCP
│   ├── PBR / selective routing
│   ├── VPN client
│   └── anti-DPI routing for YouTube/Discord
│
├── LXC 101: AdGuard Home
│   ├── DNS filtering
│   ├── ad blocking
│   ├── local DNS records
│   └── DNS query statistics
│
├── LXC 102: Chatbot
│   ├── bot runtime
│   ├── systemd service
│   └── environment file
│
└── VM/LXC 103: Nextcloud
    ├── local cloud storage
    ├── LAN/VPN-only access at first stage
    └── external storage and backup later
```

## Разделение трафика

| Тип трафика | Маршрут |
|---|---|
| YouTube | WAN + anti-DPI |
| Discord | WAN + anti-DPI |
| ChatGPT / OpenAI | VPN |
| Instagram / Meta services | VPN |
| Telegram | VPN only if required |
| Banks / government services | Direct WAN |
| Games | Direct WAN |
| IoT devices | Direct WAN or separate VLAN later |
| Other regular traffic | Direct WAN |

## План адресации

```text
Network: 192.168.10.0/24

OpenWrt LAN:      192.168.10.1
Proxmox:          192.168.10.2
AdGuard Home:     192.168.10.3
Chatbot:          192.168.10.4
Nextcloud:        192.168.10.5
Tenda i29 AP:     192.168.10.10

DHCP range:
192.168.10.100 - 192.168.10.250
```

Local DNS names:

```text
router.home.arpa      -> 192.168.10.1
proxmox.home.arpa     -> 192.168.10.2
adguard.home.arpa     -> 192.168.10.3
bot.home.arpa         -> 192.168.10.4
cloud.home.arpa       -> 192.168.10.5
ap.home.arpa          -> 192.168.10.10
```

## Этапы реализации

### Stage 1 — Base infrastructure

- Install Proxmox VE on Firebat S1.
- Configure BIOS options for virtualization and power recovery.
- Configure Proxmox network bridges.
- Create OpenWrt VM.
- Configure WAN/LAN routing.
- Connect Tenda i29 as access point.
- Verify internet access for wired and wireless clients.

### Stage 2 — DNS filtering

- Create Debian LXC container.
- Install AdGuard Home.
- Configure OpenWrt DHCP to provide AdGuard as DNS server.
- Add local DNS rewrites.
- Verify DNS logs and filtering.

### Stage 3 — anti-DPI routing

- Install OpenWrt-compatible anti-DPI tooling.
- Apply anti-DPI processing for YouTube and Discord traffic.
- Keep YouTube and Discord traffic outside VPN to reduce latency.
- Test video streaming, Discord voice, images and screen sharing.

### Stage 4 — VPN and selective routing

- Configure VPN client on OpenWrt.
- Add PBR rules for selected services.
- Route ChatGPT, Instagram and selected Telegram traffic through VPN.
- Keep regular traffic routed directly via ISP.

### Stage 5 — Additional services

- Deploy chatbot container.
- Add systemd service for bot process.
- Add monitoring.
- Prepare backup strategy.
- Deploy test Nextcloud instance after network stabilization.

## Repository structure

Planned structure:

```text
home-server-lab/
├── README.md
├── .gitignore
├── docs/
├── diagrams/
├── proxmox/
├── openwrt/
├── adguard/
├── services/
│   ├── chatbot/
│   └── nextcloud/
└── screenshots/
```

## Security notes

This repository must not contain real secrets or private configuration files.

Do not publish:

- VPN private keys;
- bot tokens;
- `.env` files;
- real WireGuard/AmneziaWG configs;
- PPPoE credentials;
- public IP address if it should remain private;
- MAC addresses;
- Nextcloud database dumps;
- Proxmox VM backups;
- screenshots with sensitive data.

Only sanitized examples should be committed:

```text
.env.example
wg0.conf.example
network.example
pbr-rules.example.md
AdGuardHome.example.yaml
```

## Current status

Project status: planning and initial repository setup.

Completed:

- repository created;
- initial `.gitignore` added;
- base README created.

Next steps:

- add documentation structure;
- document Proxmox installation plan;
- document network topology;
- prepare OpenWrt VM setup notes.
