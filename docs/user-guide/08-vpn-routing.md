# 08. VPN Routing

Этот документ описывает общий подход к настройке VPN-клиента на OpenWrt и выборочной маршрутизации трафика.

Цель этапа: направлять через VPN только те сервисы или устройства, которым это действительно нужно, а остальной трафик оставлять напрямую через провайдера.

## 1. Почему не нужно гнать весь трафик через VPN

Полный VPN для всей домашней сети может привести к проблемам:

- выше задержка;
- ниже скорость;
- больше нагрузка на VPS;
- больше проблем с банками и государственными сервисами;
- больше капч;
- сложнее диагностика;
- YouTube может забивать весь канал VPN.

Поэтому в этом проекте используется selective routing:

```text
Selected services -> VPN
Regular traffic   -> Direct WAN
YouTube/Discord   -> Direct WAN + anti-DPI processing
```

## 2. Что можно направлять через VPN

Примеры сервисов, которые могут маршрутизироваться через VPN:

```text
ChatGPT / OpenAI
Instagram / Meta services
Telegram, if required
other selected services
```

Не рекомендуется направлять через VPN:

```text
banks
government services
games
local services
printers
IoT devices
YouTube, if anti-DPI works without VPN
Discord, if anti-DPI works without VPN
```

## 3. Варианты VPN

| Вариант | Когда использовать |
|---|---|
| WireGuard | Если провайдер не блокирует и не ухудшает работу WireGuard |
| AmneziaWG | Если обычный WireGuard нестабилен из-за DPI |
| OpenVPN | Если нужен совместимый, но более тяжёлый вариант |
| sing-box / xray | Если нужна более гибкая маршрутизация и маскировка |

Для первого этапа проще всего использовать WireGuard или AmneziaWG.

## 4. Где должен работать VPN

VPN-клиент должен работать на OpenWrt, а не на каждом отдельном устройстве.

Так проще:

```text
Home clients -> OpenWrt -> PBR -> VPN or WAN
```

Преимущества:

- не нужно ставить VPN на каждое устройство;
- можно управлять маршрутами централизованно;
- можно направлять через VPN только выбранные устройства или домены;
- Smart TV и IoT можно обслуживать на уровне сети.

## 5. Требования к VPN-серверу

Минимальный VPS для лёгкой выборочной маршрутизации:

```text
1 vCPU
512 MB - 1 GB RAM
10-20 GB SSD
100 Mbps or better network
```

Если через VPN не гонять YouTube, мощный VPS на первом этапе не нужен.

## 6. Базовая логика интерфейсов

После настройки VPN в OpenWrt появится отдельный интерфейс.

Примеры:

```text
wg0  -> WireGuard
awg0 -> AmneziaWG
```

Итоговая логика:

```text
WAN  -> direct ISP route
wg0  -> VPN route
LAN  -> home clients
```

## 7. Policy-Based Routing

Для выборочной маршрутизации используется PBR.

Логика правил:

```text
chatgpt.com        -> VPN
openai.com         -> VPN
instagram.com      -> VPN
cdninstagram.com   -> VPN
telegram.org       -> VPN if required
t.me               -> VPN if required
all other traffic  -> WAN
```

## 8. Пример списка доменов для VPN

Примерный список для ChatGPT / OpenAI:

```text
chatgpt.com
openai.com
auth.openai.com
oaistatic.com
oaiusercontent.com
```

Примерный список для Instagram / Meta:

```text
instagram.com
cdninstagram.com
facebook.com
fbcdn.net
messenger.com
```

Примерный список для Telegram:

```text
telegram.org
t.me
telegram.me
```

Списки нужно корректировать по фактической работе сервисов.

## 9. Устройства вместо доменов

Иногда проще направлять через VPN не отдельные домены, а конкретное устройство.

Пример:

```text
Work laptop -> VPN
Smart TV    -> direct WAN or VPN depending on task
Phone       -> selected domains through VPN
```

Такой подход полезен, если сервис использует много CDN и доменных имён.

## 10. Проверка после настройки VPN

После настройки VPN нужно проверить:

- VPN-интерфейс поднялся;
- OpenWrt видит handshake;
- выбранные сайты открываются;
- обычные сайты идут напрямую;
- банки и локальные сервисы не ушли через VPN;
- DNS работает стабильно;
- нет утечки всего домашнего трафика в VPN без необходимости.

## 11. Проверка маршрута

Для проверки можно использовать:

```bash
traceroute example.com
curl ifconfig.me
```

Но проверять внешний IP нужно аккуратно: для выбранного сервиса IP должен быть VPN, а для обычного трафика — провайдерский.

## 12. Что не публиковать в GitHub

Нельзя публиковать:

- private key;
- реальные `wg0.conf` / `awg0.conf`;
- endpoint, если он должен оставаться приватным;
- токены панели VPN;
- реальные IP-адреса и логины.

В репозиторий можно добавлять только шаблоны:

```text
wg0.conf.example
awg0.conf.example
pbr-rules.example.md
```

## 13. Что не делать

Не рекомендуется:

- включать полный VPN для всей сети без необходимости;
- направлять YouTube через VPN, если anti-DPI работает напрямую;
- добавлять VPN до проверки базовой сети;
- менять сразу VPN, DNS и anti-DPI одновременно;
- хранить реальные ключи в репозитории.

## 14. Контрольный список

Этап считается завершённым, если:

- VPN-интерфейс работает;
- PBR установлен и включён;
- выбранные сервисы идут через VPN;
- обычный трафик идёт напрямую;
- YouTube/Discord не перегружают VPN;
- нет утечки секретов в GitHub.

## Итог

После этого этапа OpenWrt сможет направлять только выбранные сервисы через VPN, сохраняя обычный интернет напрямую через провайдера.

Следующий этап:

```text
docs/user-guide/09-anti-dpi-routing.md
```
