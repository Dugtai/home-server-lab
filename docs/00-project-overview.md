# Project Overview

## Назначение проекта

`home-server-lab` — это проект домашней серверной инфраструктуры на базе мини-ПК Firebat S1, Proxmox VE, OpenWrt, AdGuard Home, выборочной маршрутизации трафика, VPN и дополнительных self-hosted сервисов.

Проект ведётся в двух форматах:

1. **User Guide** — пошаговая инструкция для повторения проекта с нуля.
2. **Technical Documentation** — профессиональное описание архитектуры для портфолио.

Коммерческие материалы, прайс-листы и клиентские шаблоны в репозиторий не добавляются.

## Цели проекта

Основные цели:

- собрать домашний сервер-шлюз на базе Proxmox VE;
- использовать OpenWrt как основной виртуальный маршрутизатор;
- централизовать DNS-фильтрацию через AdGuard Home;
- настроить выборочную маршрутизацию трафика;
- разделить обычный трафик, VPN-трафик и anti-DPI обработку;
- подготовить основу для чатбота, Nextcloud, мониторинга и резервного копирования;
- документировать проект так, чтобы его можно было повторить и показать как портфолио.

## Аппаратная база

| Компонент | Значение |
|---|---|
| Mini PC | Firebat S1 |
| CPU | Intel N100 |
| RAM | 16 GB |
| SSD | 512 GB |
| Hypervisor | Proxmox VE |
| Router VM | OpenWrt |
| Access Point | Tenda i29 AX3000 Wi-Fi 6 |

## Логика сети

```text
ISP / Optical terminal
        ↓
Provider router in bridge mode or temporary NAT mode
        ↓
Firebat S1 with Proxmox VE
        ↓
OpenWrt VM as main gateway
        ↓
Tenda i29 AX3000 as access point
        ↓
Home clients
```

## Логика сервисов

```text
Proxmox VE
├── OpenWrt VM
│   ├── routing
│   ├── firewall
│   ├── DHCP
│   ├── PBR / selective routing
│   ├── VPN client
│   └── anti-DPI routing for YouTube/Discord
│
├── AdGuard Home LXC
│   └── DNS filtering and local DNS records
│
├── Chatbot LXC
│   └── bot runtime and service management
│
└── Nextcloud VM/LXC
    └── local cloud storage after backup planning
```

## Разделение трафика

| Категория | Маршрут |
|---|---|
| YouTube | Direct WAN + anti-DPI processing |
| Discord | Direct WAN + anti-DPI processing |
| ChatGPT / OpenAI | VPN |
| Instagram / Meta services | VPN |
| Telegram | VPN only if required |
| Банки и государственные сервисы | Direct WAN |
| Игры | Direct WAN |
| Обычные сайты | Direct WAN |

## Документационные уровни

### User Guide

Раздел `docs/user-guide/` должен отвечать на вопрос:

> Как обычному пользователю повторить проект с нуля?

Там должны быть:

- пошаговые действия;
- скриншоты;
- проверки после каждого этапа;
- типовые ошибки;
- простые объяснения терминов.

### Technical Documentation

Раздел `docs/technical/` должен отвечать на вопрос:

> Почему архитектура сделана именно так и какие инженерные решения применены?

Там должны быть:

- архитектура;
- сетевой дизайн;
- маршрутизация;
- безопасность;
- резервное копирование;
- мониторинг;
- ограничения и риски.

## Принципы безопасности

В репозиторий нельзя добавлять:

- реальные VPN-ключи;
- токены ботов;
- `.env` файлы;
- реальные конфиги WireGuard/AmneziaWG;
- PPPoE-логины и пароли;
- публичные IP-адреса, если они не предназначены для публикации;
- MAC-адреса;
- дампы баз данных;
- резервные копии виртуальных машин;
- скриншоты с чувствительными данными.

Допускаются только обезличенные примеры:

```text
.env.example
wg0.conf.example
network.example
pbr-rules.example.md
AdGuardHome.example.yaml
```

## Текущий статус

Проект находится на этапе начального проектирования и подготовки репозитория.

Выполнено:

- создан репозиторий;
- добавлен `.gitignore`;
- подготовлен базовый README;
- создана структура документации;
- разделены уровни документации: User Guide и Technical Documentation.

Ближайшие задачи:

- описать аппаратную платформу;
- подготовить схему сети;
- описать установку Proxmox VE;
- подготовить план настройки OpenWrt VM.
