# Home Server Lab

Домашняя серверная инфраструктура на базе мини-ПК Firebat S1 с Intel N100, Proxmox VE, OpenWrt, AdGuard Home, выборочной маршрутизацией трафика, VPN, anti-DPI обработкой для YouTube/Discord и дополнительными self-hosted сервисами.

Проект оформляется в двух форматах:

1. **User Guide** — пошаговый гайд для повторения проекта с нуля.
2. **Technical Documentation** — техническое описание архитектуры для портфолио.

Коммерческие материалы, прайс-листы и клиентские шаблоны в репозиторий не добавляются.

## Цель проекта

Создать управляемую домашнюю сеть с централизованной DNS-фильтрацией, виртуальным маршрутизатором, выборочной маршрутизацией трафика, изоляцией сервисов и возможностью дальнейшего расширения.

Основные задачи:

- развернуть Proxmox VE на bare metal;
- использовать OpenWrt VM как основной домашний шлюз;
- настроить WAN/LAN-сегментацию через Proxmox bridges;
- вынести DNS-фильтрацию в отдельный LXC с AdGuard Home;
- использовать selective routing вместо полного VPN для всей сети;
- направлять YouTube и Discord через anti-DPI обработку без перегрузки VPN;
- направлять отдельные сервисы через VPN;
- оставить обычный трафик напрямую через провайдера;
- подготовить основу для чатбота, мониторинга, Nextcloud и резервного копирования;
- документировать проект как воспроизводимый инфраструктурный стенд.

## Аппаратная платформа

| Компонент | Модель / параметры |
|---|---|
| Mini PC | Firebat S1 |
| CPU | Intel N100 |
| RAM | 16 GB |
| SSD | 512 GB |
| Hypervisor | Proxmox VE |
| Router VM | OpenWrt |
| Access Point | Tenda i29 AX3000 Wi-Fi 6 |
| Network | 2 × Ethernet |

## Целевая архитектура

```text
ISP / Optical terminal / Provider router
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
├── VM/LXC 103: Nextcloud
│   ├── local cloud storage
│   ├── LAN/VPN-only access at first stage
│   └── external storage and backup later
│
└── LXC 104: Monitoring
    ├── uptime checks
    ├── service status checks
    └── optional notifications
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
| Local services | LAN only |
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

## Документация

### User Guide

Раздел для пошагового повторения проекта с нуля.

```text
docs/user-guide/
├── 00-roadmap.md
├── 01-what-to-buy.md
├── 02-first-start.md
├── 03-install-proxmox.md
├── 04-proxmox-network.md
├── 05-create-openwrt-vm.md
├── 06-connect-access-point.md
├── 07-install-adguard.md
├── 08-vpn-routing.md
├── 09-anti-dpi-routing.md
└── 10-troubleshooting.md
```

Рекомендуемый порядок прохождения:

```text
01 -> 02 -> 03 -> 04 -> 05 -> 06 -> 07 -> 08 -> 09 -> 10
```

### Technical Documentation

Раздел для технического описания архитектуры и инженерных решений.

```text
docs/technical/
├── architecture.md
├── network-design.md
├── routing-policy.md
├── security-notes.md
├── backup-strategy.md
├── service-isolation.md
└── monitoring.md
```

## Репозиторий

Текущая структура:

```text
home-server-lab/
├── README.md
├── CHANGELOG.md
├── .gitignore
├── docs/
│   ├── 00-project-overview.md
│   ├── user-guide/
│   └── technical/
├── proxmox/
│   ├── README.md
│   ├── vm-plan.md
│   └── storage-plan.md
├── openwrt/
│   ├── README.md
│   └── firewall-notes.md
├── adguard/
│   ├── README.md
│   └── setup.md
├── configs/
│   ├── openwrt/
│   │   └── pbr-rules.example.md
│   └── wireguard/
│       └── wg0.conf.example
├── services/
│   ├── README.md
│   └── chatbot/
│       ├── README.md
│       ├── .env.example
│       └── systemd-service.example
├── diagrams/
│   └── README.md
└── screenshots/
    └── README.md
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

### Stage 3 — VPN and selective routing

- Configure VPN client on OpenWrt.
- Add PBR rules for selected services.
- Route selected services through VPN.
- Keep regular traffic routed directly via ISP.

### Stage 4 — anti-DPI routing

- Install OpenWrt-compatible anti-DPI tooling.
- Apply anti-DPI processing for YouTube and Discord traffic.
- Keep YouTube and Discord traffic outside VPN to reduce latency.
- Test video streaming, Discord voice, images and screen sharing.

### Stage 5 — Additional services

- Deploy chatbot container.
- Add systemd service for bot process.
- Add monitoring.
- Prepare backup strategy.
- Deploy test Nextcloud instance after network stabilization and backup planning.

## Security notes

This repository must not contain real secrets or private configuration files.

Do not publish:

- VPN private keys;
- bot tokens;
- real `.env` files;
- real WireGuard/AmneziaWG configs;
- PPPoE credentials;
- public IP address if it should remain private;
- MAC addresses;
- Nextcloud database dumps;
- Proxmox VM/LXC backups;
- OpenWrt full backups without cleanup;
- AdGuardHome.yaml with sensitive data;
- screenshots with sensitive data.

Only sanitized examples should be committed:

```text
.env.example
wg0.conf.example
network.example
pbr-rules.example.md
AdGuardHome.example.yaml
```

`.gitignore` reduces the risk of accidental commits, but it does not make secret handling automatic. Real secrets must remain outside the repository.

## Current status

Project status: repository prepared, practical deployment pending.

Completed:

- repository created;
- `.gitignore` added;
- main README updated;
- User Guide structure added;
- Technical Documentation structure added;
- Proxmox planning documents added;
- OpenWrt notes added;
- AdGuard setup notes added;
- VPN/PBR examples added;
- chatbot templates added;
- backup, monitoring and security notes added;
- changelog added.

Next practical steps:

- prepare Proxmox VE installation media;
- configure Firebat S1 BIOS;
- install Proxmox VE;
- identify physical network interfaces;
- configure Proxmox bridges;
- create OpenWrt VM.
