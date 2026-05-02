# 00. User Guide Roadmap

Этот документ задаёт рекомендуемый порядок прохождения пользовательского гайда.

Цель: пройти настройку по шагам и не смешивать несколько сложных задач одновременно.

## Общий принцип

Не нужно сразу настраивать Proxmox, OpenWrt, AdGuard, VPN, anti-DPI и Nextcloud одновременно.

Правильный порядок:

```text
Hardware -> Proxmox -> Network -> OpenWrt -> Access Point -> AdGuard -> VPN -> anti-DPI -> Services
```

## Roadmap

### Step 1 — Hardware

Документ:

```text
docs/user-guide/01-what-to-buy.md
```

На этом этапе нужно понять:

- какое железо требуется;
- зачем нужен mini PC;
- зачем нужны два Ethernet-порта;
- хватит ли 16 GB RAM;
- хватит ли SSD 512 GB;
- что докупить позже.

### Step 2 — First start

Документ:

```text
docs/user-guide/02-first-start.md
```

На этом этапе:

- проверяется комплект;
- включается mini PC;
- проверяется BIOS;
- включается виртуализация;
- отключается Secure Boot;
- готовится флешка с Proxmox VE.

### Step 3 — Install Proxmox VE

Документ:

```text
docs/user-guide/03-install-proxmox.md
```

На этом этапе:

- Windows удаляется;
- Proxmox VE устанавливается на SSD;
- настраивается временный IP;
- проверяется вход в веб-интерфейс Proxmox.

### Step 4 — Proxmox network

Документ:

```text
docs/user-guide/04-proxmox-network.md
```

На этом этапе:

- определяются физические сетевые интерфейсы;
- планируются `vmbr0` и `vmbr1`;
- подготавливается WAN/LAN логика;
- фиксируется будущая адресация.

### Step 5 — Create OpenWrt VM

Документ:

```text
docs/user-guide/05-create-openwrt-vm.md
```

На этом этапе:

- создаётся OpenWrt VM;
- подключаются WAN и LAN bridge;
- настраивается LAN IP;
- проверяется базовая маршрутизация.

### Step 6 — Connect access point

Документ:

```text
docs/user-guide/06-connect-access-point.md
```

На этом этапе:

- Tenda i29 переводится в режим Access Point;
- отключается DHCP/NAT на точке доступа;
- задаётся статический IP;
- проверяется Wi-Fi через OpenWrt.

### Step 7 — Install AdGuard Home

Документ:

```text
docs/user-guide/07-install-adguard.md
```

На этом этапе:

- создаётся LXC для AdGuard;
- устанавливается AdGuard Home;
- OpenWrt начинает выдавать AdGuard как DNS;
- проверяется Query Log.

### Step 8 — VPN routing

Документ:

```text
docs/user-guide/08-vpn-routing.md
```

На этом этапе:

- добавляется VPN-клиент на OpenWrt;
- включается selective routing;
- выбранные сервисы направляются через VPN;
- обычный трафик остаётся напрямую.

### Step 9 — Anti-DPI routing

Документ:

```text
docs/user-guide/09-anti-dpi-routing.md
```

На этом этапе:

- YouTube и Discord обрабатываются отдельно от VPN;
- проверяется YouTube video playback;
- проверяется Discord voice;
- фиксируются рабочие правила.

### Step 10 — Troubleshooting

Документ:

```text
docs/user-guide/10-troubleshooting.md
```

Этот документ использовать при ошибках:

- нет доступа к Proxmox;
- не запускается OpenWrt VM;
- клиенты не получают IP;
- не работает DNS;
- не работает VPN;
- не работает YouTube/Discord.

## Что делать после базовой настройки

После прохождения шагов 1–10 можно переходить к дополнительным сервисам:

- chatbot;
- monitoring;
- Nextcloud;
- backups;
- VLAN;
- remote access.

## Что не делать раньше времени

Не нужно до завершения базовой сети:

- устанавливать Nextcloud;
- открывать сервисы в интернет;
- включать bridge mode у провайдера без проверки;
- добавлять VLAN;
- ставить слишком много фильтров в AdGuard;
- делать полный VPN для всей сети.

## Итог

Рекомендуемый порядок прохождения гайда:

```text
01 -> 02 -> 03 -> 04 -> 05 -> 06 -> 07 -> 08 -> 09 -> 10
```

Каждый этап должен завершаться проверкой результата.
