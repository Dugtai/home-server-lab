# 05. Create OpenWrt VM

Этот документ описывает создание виртуальной машины OpenWrt в Proxmox VE.

Цель этапа: подготовить OpenWrt VM, которая будет выполнять роль основного маршрутизатора домашней сети.

## 1. Роль OpenWrt в проекте

OpenWrt будет выполнять функции:

- WAN/LAN routing;
- NAT;
- firewall;
- DHCP server;
- DNS forwarding;
- VPN client;
- selective routing / PBR;
- anti-DPI processing for selected traffic.

## 2. Рекомендуемые ресурсы VM

Для домашнего сервера достаточно:

| Параметр | Значение |
|---|---|
| VM ID | 100 |
| Name | openwrt-gateway |
| CPU | 2 cores |
| RAM | 1024 MB |
| Disk | 4-8 GB |
| NIC 1 | vmbr0 / WAN |
| NIC 2 | vmbr1 / LAN |

## 3. Скачать OpenWrt image

Нужен образ OpenWrt для x86-64.

Обычно используется вариант:

```text
x86/64 combined ext4 image
```

Для Proxmox удобнее использовать raw/img образ и импортировать его как диск VM.

## 4. Создать VM без диска

В Proxmox:

```text
Create VM
```

Основные параметры:

```text
VM ID: 100
Name: openwrt-gateway
OS: Do not use any media
System: default
Disk: удалить или не создавать диск, если Proxmox позволяет
CPU: 2 cores
Memory: 1024 MB
Network 1: vmbr0
```

После создания VM добавить второй сетевой адаптер:

```text
Hardware -> Add -> Network Device -> vmbr1
```

Итог:

```text
net0 -> vmbr0 -> WAN
net1 -> vmbr1 -> LAN
```

## 5. Импортировать диск OpenWrt

Общий порядок через shell Proxmox:

1. Загрузить или скопировать образ OpenWrt на Proxmox.
2. Распаковать образ, если он в `.gz`.
3. Импортировать диск в VM.
4. Назначить диск как загрузочный.

Примерная логика команд:

```bash
# Example only. File name must be replaced with actual image name.
gunzip openwrt-x86-64-generic-ext4-combined.img.gz
qm importdisk 100 openwrt-x86-64-generic-ext4-combined.img local-lvm
```

После импорта в Proxmox:

```text
VM 100 -> Hardware -> Unused Disk -> Edit -> Add
```

Потом в настройках VM:

```text
Options -> Boot Order -> enable imported disk
```

## 6. Первый запуск OpenWrt

Запустить VM:

```text
VM 100 -> Start
```

Открыть консоль:

```text
VM 100 -> Console
```

После загрузки OpenWrt обычно доступна консоль без пароля на первом старте.

## 7. Определить интерфейсы внутри OpenWrt

В OpenWrt интерфейсы могут называться:

```text
eth0
eth1
```

Обычно порядок такой:

```text
eth0 -> WAN
eth1 -> LAN
```

Но это нужно проверить.

## 8. Настроить LAN IP

Целевая LAN-сеть проекта:

```text
OpenWrt LAN: 192.168.10.1/24
```

Если OpenWrt по умолчанию использует `192.168.1.1`, это нужно будет изменить на `192.168.10.1`, чтобы не конфликтовать с текущим роутером провайдера.

## 9. Настроить WAN

На первом этапе WAN можно оставить как DHCP:

```text
WAN protocol: DHCP client
```

Тогда OpenWrt получит интернет от существующего роутера провайдера.

Позже, если провайдерский роутер будет переведён в bridge mode, WAN нужно будет настроить в зависимости от типа подключения:

```text
DHCP
PPPoE
Static IP
```

## 10. Настроить DHCP на LAN

OpenWrt должен выдавать адреса домашним клиентам:

```text
DHCP range: 192.168.10.100 - 192.168.10.250
Gateway: 192.168.10.1
DNS: later AdGuard Home 192.168.10.3
```

На первом этапе DNS можно оставить OpenWrt, а после установки AdGuard заменить DNS на `192.168.10.3`.

## 11. Первый тест

После настройки OpenWrt нужно проверить:

- OpenWrt доступен по `192.168.10.1`;
- клиент получает IP из диапазона `192.168.10.100-250`;
- клиент видит gateway `192.168.10.1`;
- интернет работает через OpenWrt;
- Proxmox доступен по LAN IP.

## 12. Что не делать на этом этапе

Пока не нужно:

- устанавливать VPN;
- включать anti-DPI;
- настраивать PBR;
- переводить провайдера в bridge mode;
- подключать Nextcloud;
- делать сложные firewall rules.

Сначала нужно добиться стабильной базовой маршрутизации.

## 13. Контрольный список

Этап считается завершённым, если:

- OpenWrt VM запускается автоматически или вручную без ошибок;
- у VM есть два сетевых интерфейса;
- WAN получает интернет;
- LAN выдаёт IP клиентам;
- веб-интерфейс OpenWrt доступен;
- интернет через OpenWrt работает.

## Итог

После этого этапа OpenWrt VM готова выполнять роль основного шлюза домашней сети.

Следующий этап:

```text
docs/user-guide/06-connect-access-point.md
```
