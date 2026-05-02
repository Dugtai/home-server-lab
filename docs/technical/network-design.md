# Network Design

## Назначение документа

Этот документ описывает сетевой дизайн проекта `home-server-lab`.

Основная задача: построить домашнюю сеть, в которой mini PC с Proxmox и OpenWrt выполняет роль центрального шлюза, а Wi-Fi access point используется только для подключения клиентов.

## Целевая топология

```text
ISP / ONT / Provider router
        ↓
WAN Ethernet port on mini PC
        ↓
Proxmox vmbr0
        ↓
OpenWrt WAN
        ↓
OpenWrt routing/firewall/NAT
        ↓
OpenWrt LAN
        ↓
Proxmox vmbr1
        ↓
LAN Ethernet port on mini PC
        ↓
Tenda i29 Access Point
        ↓
Home clients
```

## Proxmox bridge design

| Bridge | Role | Physical port | IP on Proxmox |
|---|---|---|---|
| vmbr0 | WAN bridge | NIC 1 | No IP |
| vmbr1 | LAN bridge | NIC 2 | 192.168.10.2/24 |

`vmbr0` предназначен только для подключения WAN-интерфейса OpenWrt.

`vmbr1` является локальным bridge-интерфейсом, через который доступны Proxmox, LXC-контейнеры, VM и домашние клиенты.

## Почему на WAN bridge нет IP

Proxmox не должен иметь IP-адрес на внешней стороне сети.

Причины:

- панель управления Proxmox не должна быть доступна со стороны WAN;
- уменьшается поверхность атаки;
- проще контролировать направление трафика;
- WAN полностью отдаётся OpenWrt VM.

## LAN addressing plan

```text
Network: 192.168.10.0/24

OpenWrt LAN:      192.168.10.1
Proxmox:          192.168.10.2
AdGuard Home:     192.168.10.3
Chatbot:          192.168.10.4
Nextcloud:        192.168.10.5
Access Point:     192.168.10.10
DHCP clients:     192.168.10.100-192.168.10.250
```

## DNS design

AdGuard Home является основным DNS-сервером для клиентов.

```text
Clients -> AdGuard Home -> Upstream DNS
```

DHCP option on OpenWrt:

```text
6,192.168.10.3
```

Локальные DNS-записи:

```text
router.home.arpa      -> 192.168.10.1
proxmox.home.arpa     -> 192.168.10.2
adguard.home.arpa     -> 192.168.10.3
bot.home.arpa         -> 192.168.10.4
cloud.home.arpa       -> 192.168.10.5
ap.home.arpa          -> 192.168.10.10
```

## DHCP design

DHCP должен работать только на OpenWrt LAN.

Access point не должен выдавать IP-адреса.

```text
OpenWrt DHCP: enabled
Access Point DHCP: disabled
Provider router DHCP: outside LAN scope or not used after bridge mode
```

## Access Point design

Tenda i29 используется как L2 access point:

- no NAT;
- no DHCP;
- static management IP;
- Wi-Fi clients are part of OpenWrt LAN;
- routing is handled by OpenWrt.

Management IP:

```text
192.168.10.10
```

## Temporary NAT stage

На первом этапе допускается временная схема с двойным NAT:

```text
Provider router with NAT
        ↓
OpenWrt WAN DHCP
        ↓
OpenWrt LAN
```

Это безопаснее для начальной настройки, потому что основная сеть провайдера остаётся рабочей.

После проверки стабильности можно перейти к bridge mode.

## Bridge mode stage

После стабилизации сети провайдерский роутер или ONT может быть переведён в bridge mode.

Тогда OpenWrt становится основным граничным маршрутизатором.

WAN OpenWrt может использовать:

- DHCP;
- PPPoE;
- static IP.

Фактический режим зависит от провайдера.

## Traffic flow

### Regular traffic

```text
Client -> OpenWrt -> WAN -> ISP
```

### DNS traffic

```text
Client -> AdGuard Home -> upstream DNS
```

### VPN-selected traffic

```text
Client -> OpenWrt PBR -> VPN interface -> Internet
```

### YouTube/Discord anti-DPI traffic

```text
Client -> OpenWrt anti-DPI processing -> WAN -> ISP
```

## Future VLAN design

На первом этапе VLAN не используются.

Возможное развитие:

| VLAN | Role |
|---|---|
| VLAN 10 | Main LAN |
| VLAN 20 | Guest Wi-Fi |
| VLAN 30 | IoT devices |
| VLAN 40 | Lab services |
| VLAN 99 | Management |

Для VLAN потребуется поддержка на access point и, возможно, managed switch.

## Risks

Основные риски сетевого дизайна:

- потеря доступа к Proxmox из-за ошибки bridge;
- конфликт IP-адресов;
- два DHCP-сервера в одной сети;
- случайная публикация Proxmox в WAN;
- некорректная маршрутизация VPN/PBR;
- отказ OpenWrt VM приводит к потере интернета.

## Mitigation

Меры снижения рисков:

- не назначать IP на WAN bridge;
- вести таблицу адресации;
- отключить DHCP на access point;
- иметь локальный доступ к mini PC через монитор/клавиатуру;
- менять сетевые параметры по одному;
- сохранять бэкапы конфигураций перед изменениями.

## Итог

Сетевой дизайн строится вокруг OpenWrt VM как основного маршрутизатора, Proxmox как платформы виртуализации и AdGuard Home как выделенного DNS-фильтра.
