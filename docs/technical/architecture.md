# Architecture

## Назначение архитектуры

Проект `home-server-lab` строится как домашняя серверная инфраструктура с разделением ролей между гипервизором, виртуальным маршрутизатором и отдельными контейнерами сервисов.

Основная идея: mini PC используется не как обычный домашний ПК, а как компактная серверная платформа.

## Основные компоненты

| Компонент | Роль |
|---|---|
| Firebat S1 | Физическая серверная платформа |
| Proxmox VE | Гипервизор для VM и LXC |
| OpenWrt VM | Основной шлюз, firewall, DHCP, routing |
| AdGuard Home LXC | DNS-фильтрация и локальные DNS-записи |
| Chatbot LXC | Изолированное окружение для чатбота |
| Nextcloud VM/LXC | Локальное облако, планируется после бэкапов |
| Tenda i29 AX3000 | Wi-Fi access point |

## Архитектурный принцип

Каждый сервис должен выполнять одну основную роль:

```text
Proxmox  -> virtualization
OpenWrt  -> routing and firewall
AdGuard  -> DNS filtering
Chatbot  -> application service
Nextcloud -> file storage service
```

Такой подход упрощает диагностику, обновление, резервное копирование и восстановление.

## Высокоуровневая схема

```text
ISP / Optical terminal
        ↓
Provider router in bridge mode or temporary NAT mode
        ↓
Firebat S1
        ↓
Proxmox VE
        ↓
OpenWrt VM
        ↓
LAN bridge
        ↓
Tenda i29 Access Point
        ↓
Home clients
```

## Виртуальная схема

```text
Proxmox VE
├── vmbr0: WAN bridge
│   └── OpenWrt WAN interface
│
├── vmbr1: LAN bridge
│   ├── OpenWrt LAN interface
│   ├── Proxmox management IP
│   ├── AdGuard Home LXC
│   ├── Chatbot LXC
│   ├── Nextcloud VM/LXC
│   └── Access Point / LAN clients
```

## Разделение WAN и LAN

WAN и LAN разделяются на уровне Proxmox bridges:

```text
Physical NIC 1 -> vmbr0 -> WAN
Physical NIC 2 -> vmbr1 -> LAN
```

На WAN bridge не назначается IP-адрес Proxmox. Это снижает риск случайной публикации панели управления Proxmox во внешнюю сеть.

Управление Proxmox должно быть доступно только из LAN.

## Роль OpenWrt

OpenWrt является центральным элементом сетевой логики.

Функции OpenWrt:

- маршрутизация между WAN и LAN;
- NAT;
- firewall;
- DHCP server;
- DNS forwarding;
- VPN client;
- selective routing / PBR;
- anti-DPI processing for selected traffic.

## Роль AdGuard Home

AdGuard Home не маршрутизирует трафик. Он выполняет DNS-фильтрацию.

Функции:

- обработка DNS-запросов клиентов;
- блокировка рекламы и трекеров;
- локальные DNS-записи;
- журнал DNS-запросов;
- диагностика доменов, используемых клиентами.

## Роль VPN

VPN используется не как универсальный туннель для всего трафика, а как отдельный маршрут для выбранных сервисов.

Логика:

```text
Selected services -> VPN interface
Regular traffic   -> WAN interface
YouTube/Discord   -> WAN + anti-DPI processing
```

Такой подход снижает нагрузку на VPN-сервер и уменьшает задержку для чувствительных сервисов.

## Роль anti-DPI

Anti-DPI обработка применяется только к определённым типам трафика.

Целевые сервисы:

- YouTube;
- Discord.

Причина: эти сервисы чувствительны к скорости и задержке. При рабочей anti-DPI обработке их нецелесообразно направлять через VPN.

## Изоляция сервисов

Сервисы выносятся в отдельные VM/LXC:

```text
OpenWrt  -> отдельная VM
AdGuard  -> отдельный LXC
Chatbot  -> отдельный LXC
Nextcloud -> отдельная VM/LXC
```

Преимущества:

- меньше взаимных зависимостей;
- проще обновлять;
- проще бэкапить;
- проще отключать и восстанавливать;
- ниже риск, что сбой одного сервиса повлияет на остальные.

## Расширяемость

В будущем архитектура может быть расширена:

- VLAN для guest/IoT сетей;
- monitoring stack;
- backup server;
- reverse proxy;
- VPN server для удалённого доступа домой;
- отдельное хранилище для Nextcloud;
- UPS integration.

## Ограничения

Текущая аппаратная база имеет ограничения:

- один SSD без RAID;
- ограниченный объём 512 GB;
- mini PC не является полноценным сервером с ECC RAM;
- отказ mini PC приведёт к остановке маршрутизации и сервисов;
- требуется аккуратная стратегия резервного копирования.

## Итог

Архитектура проекта ориентирована на баланс между простотой, управляемостью и возможностью дальнейшего развития.

Ключевое решение — использовать OpenWrt VM как основной шлюз, а все дополнительные сервисы выносить в отдельные контейнеры или виртуальные машины.
