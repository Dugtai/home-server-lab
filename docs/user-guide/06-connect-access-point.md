# 06. Connect Access Point

Этот документ описывает подключение Wi-Fi access point к OpenWrt-сети.

Цель этапа: использовать Tenda i29 или аналогичное устройство только как точку доступа, без функций роутера, NAT и DHCP.

## 1. Роль access point

В этой архитектуре маршрутизацией занимается OpenWrt VM.

Access point нужен только для Wi-Fi:

```text
OpenWrt VM -> LAN port -> Access Point -> Wi-Fi clients
```

Access point не должен выполнять роль отдельного роутера.

## 2. Целевая схема

```text
Provider router / ONT
        ↓
Mini PC WAN port
        ↓
OpenWrt VM
        ↓
Mini PC LAN port
        ↓
Tenda i29 AX3000 Access Point
        ↓
Home Wi-Fi clients
```

## 3. Настройки access point

Рекомендуемые параметры:

```text
Mode: Access Point
DHCP Server: Disabled
NAT: Disabled
Static IP: 192.168.10.10
Gateway: 192.168.10.1
DNS: 192.168.10.3 later, or 192.168.10.1 at first stage
```

Если в веб-интерфейсе точки доступа есть отдельный режим `AP Mode`, нужно выбрать именно его.

## 4. Подключение кабеля

Кабель подключается так:

```text
Mini PC LAN port -> Access Point Ethernet port
```

Если access point питается по PoE, потребуется:

```text
Mini PC LAN port -> PoE injector/switch -> Access Point
```

## 5. Настройка Wi-Fi

Рекомендуется создать минимум одну основную Wi-Fi сеть.

Пример:

```text
SSID: Home
Security: WPA2/WPA3-Personal
Password: strong password
Band: 2.4 GHz + 5 GHz
```

Если точка доступа позволяет разделить диапазоны, можно использовать:

```text
Home-2G
Home-5G
```

Для большинства пользователей удобнее один общий SSID, если устройство корректно управляет band steering.

## 6. Гостевая сеть

Гостевую сеть лучше не включать на первом этапе.

Сначала нужно проверить:

- основная сеть работает;
- клиенты получают IP от OpenWrt;
- интернет стабилен;
- DNS работает корректно.

Гостевую сеть и VLAN лучше добавлять позже.

## 7. Проверка клиента

Подключить телефон или ноутбук к Wi-Fi и проверить параметры сети.

Клиент должен получить адрес:

```text
IP: 192.168.10.x
Gateway: 192.168.10.1
DNS: 192.168.10.1 at first stage or 192.168.10.3 after AdGuard setup
```

Если клиент получает адрес вида `192.168.0.x` или `192.168.1.x`, значит DHCP выдаёт не OpenWrt, а другое устройство.

## 8. Типовые ошибки

### Клиенты не получают IP

Проверить:

- включён ли DHCP на OpenWrt LAN;
- отключён ли DHCP на access point;
- правильно ли подключён кабель;
- активен ли LAN bridge в Proxmox;
- запущена ли OpenWrt VM.

### Интернет есть по кабелю, но нет по Wi-Fi

Проверить:

- режим access point;
- настройки SSID;
- шифрование WPA2/WPA3;
- получает ли Wi-Fi клиент IP;
- не включён ли изолированный guest mode.

### Access point недоступен по IP

Проверить:

- задан ли статический IP `192.168.10.10`;
- нет ли конфликта IP;
- подключено ли устройство к LAN-сегменту;
- не изменился ли IP после сброса настроек.

## 9. Что не делать

Не рекомендуется:

- оставлять DHCP включённым на access point;
- использовать access point как второй роутер;
- включать NAT на access point;
- создавать гостевую сеть до проверки основной;
- менять одновременно настройки OpenWrt и access point.

## 10. Контрольный список

Этап считается завершённым, если:

- access point подключён к LAN-порту mini PC;
- access point имеет статический IP;
- DHCP на access point отключён;
- Wi-Fi клиенты получают IP от OpenWrt;
- интернет через Wi-Fi работает;
- OpenWrt и Proxmox доступны из Wi-Fi сети.

## Итог

После этого этапа домашняя сеть должна работать через OpenWrt, а Wi-Fi должен раздаваться через access point.

Следующий этап:

```text
docs/user-guide/07-install-adguard.md
```
