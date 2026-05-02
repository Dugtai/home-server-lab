# 04. Proxmox Network

Этот документ описывает базовую сетевую настройку Proxmox VE для схемы с виртуальным OpenWrt.

Цель этапа: подготовить два сетевых bridge-интерфейса:

```text
vmbr0 -> WAN
vmbr1 -> LAN
```

## 1. Общая логика

Mini PC с двумя Ethernet-портами будет использоваться как сервер-шлюз.

Один физический порт будет смотреть в сторону провайдера:

```text
Provider router / ONT -> WAN port -> vmbr0 -> OpenWrt WAN
```

Второй физический порт будет смотреть в домашнюю сеть:

```text
OpenWrt LAN -> vmbr1 -> LAN port -> Access Point / switch / clients
```

## 2. Временная схема на этапе настройки

На первом этапе не нужно сразу переводить роутер провайдера в bridge mode.

Безопаснее начать так:

```text
Existing router
        ↓
Mini PC / Proxmox
        ↓
OpenWrt VM later
        ↓
Access Point later
```

Так можно настроить Proxmox и OpenWrt без риска сразу потерять интернет во всей квартире/доме.

## 3. Найти имена сетевых интерфейсов

В веб-интерфейсе Proxmox открыть:

```text
Node -> System -> Network
```

Там будут видны сетевые интерфейсы.

Они могут называться примерно так:

```text
enp1s0
enp2s0
```

Имена зависят от конкретного железа и BIOS.

## 4. Как определить WAN и LAN порт

Нужно понять, какой физический Ethernet-порт соответствует какому имени.

Простой способ:

1. Подключить кабель только в один Ethernet-порт.
2. В Proxmox посмотреть, у какого интерфейса появился статус `active`.
3. Записать соответствие.
4. Переставить кабель во второй порт.
5. Снова проверить статус.

Пример записи:

```text
Left Ethernet port  -> enp1s0
Right Ethernet port -> enp2s0
```

Фактические названия нужно записать для своего устройства.

## 5. Целевая схема bridge

Пример:

```text
enp1s0 -> vmbr0 -> WAN
enp2s0 -> vmbr1 -> LAN
```

`vmbr0` используется только для WAN OpenWrt. IP-адрес на самом Proxmox для `vmbr0` обычно не нужен.

`vmbr1` используется для локальной сети. На нём будет IP-адрес Proxmox.

## 6. Пример целевой адресации

После настройки OpenWrt локальная сеть будет такой:

```text
Network: 192.168.10.0/24

OpenWrt LAN:  192.168.10.1
Proxmox:      192.168.10.2
AdGuard:      192.168.10.3
Access Point: 192.168.10.10
DHCP range:   192.168.10.100-192.168.10.250
```

## 7. Настройка через веб-интерфейс Proxmox

Открыть:

```text
Node -> System -> Network
```

Проверить существующий bridge, чаще всего это:

```text
vmbr0
```

После установки Proxmox он обычно создан автоматически и содержит IP-адрес Proxmox.

На этапе перестройки сети нужно быть осторожным: неправильная настройка bridge может привести к потере доступа к веб-интерфейсу.

## 8. Рекомендуемая логика настройки

На старте можно использовать такую стратегию:

1. Не менять сразу текущий рабочий bridge, через который доступен Proxmox.
2. Сначала определить имена физических интерфейсов.
3. Создать второй bridge.
4. Подготовить OpenWrt VM.
5. Перенести постоянный IP Proxmox в LAN-сеть только после проверки OpenWrt.

## 9. Пример итоговой конфигурации `/etc/network/interfaces`

Ниже пример. Его нельзя копировать вслепую без проверки имён интерфейсов.

```text
auto lo
iface lo inet loopback

iface enp1s0 inet manual

iface enp2s0 inet manual

auto vmbr0
iface vmbr0 inet manual
        bridge-ports enp1s0
        bridge-stp off
        bridge-fd 0

# WAN bridge for OpenWrt VM. No IP address on Proxmox.

auto vmbr1
iface vmbr1 inet static
        address 192.168.10.2/24
        gateway 192.168.10.1
        bridge-ports enp2s0
        bridge-stp off
        bridge-fd 0

# LAN bridge. Proxmox management interface.
```

Важно: `gateway` должен быть только один. В целевой схеме gateway для Proxmox — это OpenWrt LAN IP `192.168.10.1`.

## 10. Что делать, если потерян доступ к Proxmox

Если после изменения сети веб-интерфейс Proxmox перестал открываться:

1. Подключить монитор и клавиатуру к mini PC.
2. Войти в консоль Proxmox.
3. Проверить файл:

```bash
nano /etc/network/interfaces
```

4. Исправить ошибку.
5. Перезапустить сеть или перезагрузить сервер:

```bash
systemctl restart networking
```

или

```bash
reboot
```

## 11. Что не делать

Не рекомендуется:

- назначать IP на WAN bridge;
- открывать Proxmox со стороны WAN;
- включать bridge mode у провайдера до проверки OpenWrt;
- одновременно менять все сетевые настройки без локального доступа к mini PC;
- использовать один и тот же IP на Proxmox и OpenWrt.

## 12. Контрольный список

Перед переходом к созданию OpenWrt VM нужно зафиксировать:

- имя физического WAN-интерфейса;
- имя физического LAN-интерфейса;
- какой bridge будет WAN;
- какой bridge будет LAN;
- IP Proxmox в будущей LAN-сети;
- есть ли локальный доступ к серверу через монитор и клавиатуру.

## Итог

После этого этапа Proxmox должен быть готов к созданию OpenWrt VM с двумя сетевыми адаптерами:

```text
NIC 1 -> vmbr0 -> WAN
NIC 2 -> vmbr1 -> LAN
```
