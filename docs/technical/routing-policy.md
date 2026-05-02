# Routing Policy

## Назначение документа

Этот документ описывает политику маршрутизации трафика в проекте `home-server-lab`.

Основная идея: не направлять весь трафик через один маршрут, а разделить его по категориям.

## Основной принцип

```text
YouTube / Discord -> WAN + anti-DPI
Selected blocked services -> VPN
Regular traffic -> direct WAN
Local services -> LAN
```

Такой подход снижает задержку, уменьшает нагрузку на VPN и упрощает эксплуатацию домашней сети.

## Категории трафика

| Category | Route | Reason |
|---|---|---|
| Regular websites | Direct WAN | минимальная задержка |
| Banks / government services | Direct WAN | меньше проблем с антифродом |
| Games | Direct WAN | минимальный ping |
| YouTube | WAN + anti-DPI | не перегружать VPN видеотрафиком |
| Discord | WAN + anti-DPI | снизить задержку в voice |
| ChatGPT / OpenAI | VPN | выбранный защищённый маршрут |
| Instagram / Meta services | VPN | выбранный защищённый маршрут |
| Telegram | VPN only if required | зависит от фактической доступности |
| Local services | LAN | не должны уходить в интернет |

## Почему не full-tunnel VPN

Full-tunnel VPN означает, что весь трафик домашней сети уходит через VPN.

Недостатки:

- выше задержка;
- ниже скорость;
- сильная зависимость от VPS;
- YouTube перегружает VPN-канал;
- Discord voice может иметь высокий jitter;
- банки и государственные сервисы могут чаще требовать проверки;
- сложнее понять, где именно возникла проблема.

## Selective routing

Selective routing позволяет направлять трафик по правилам.

Примеры критериев:

- source IP;
- destination IP;
- domain;
- port;
- protocol;
- interface;
- device group.

На OpenWrt для этого может использоваться PBR.

## VPN route

VPN route должен применяться только к выбранным сервисам или устройствам.

Примерные домены для OpenAI:

```text
chatgpt.com
openai.com
auth.openai.com
oaistatic.com
oaiusercontent.com
```

Примерные домены для Meta/Instagram:

```text
instagram.com
cdninstagram.com
facebook.com
fbcdn.net
messenger.com
```

Примерные домены для Telegram:

```text
telegram.org
t.me
telegram.me
```

## Anti-DPI route

Anti-DPI processing should be applied only where it is required.

Target services:

```text
YouTube
Discord
```

Example YouTube-related domains:

```text
youtube.com
www.youtube.com
m.youtube.com
youtu.be
googlevideo.com
ytimg.com
ggpht.com
youtubei.googleapis.com
```

Example Discord-related domains:

```text
discord.com
discord.gg
discordapp.com
discordapp.net
discordcdn.com
discord.media
```

## Direct WAN route

Direct WAN remains the default route.

It should be used for:

- regular web browsing;
- online games;
- banking;
- government services;
- local ISP resources;
- software updates, unless blocked;
- IoT devices, unless isolated later.

## Local LAN route

Local services must remain inside LAN:

```text
proxmox.home.arpa
router.home.arpa
adguard.home.arpa
bot.home.arpa
cloud.home.arpa
ap.home.arpa
```

They should not be routed to VPN.

## Device-based routing

For some cases, routing by device is simpler than routing by domain.

Example:

| Device | Policy |
|---|---|
| Work laptop | VPN for selected services |
| Smart TV | WAN + anti-DPI for YouTube |
| Phone | selected services through VPN |
| Printer | direct LAN only |
| IoT | direct WAN or isolated VLAN later |

## Failure behavior

It is important to decide what happens if VPN fails.

Possible modes:

1. Fail open: traffic falls back to WAN.
2. Fail closed: traffic is blocked if VPN is down.

For privacy-sensitive services, fail closed may be safer.

For convenience-focused home use, fail open may be more practical.

This project should document the selected behavior for each route.

## Testing policy

After every routing change, test:

- regular website direct WAN;
- selected VPN website;
- YouTube playback;
- Discord voice;
- bank/government site direct access;
- local DNS names;
- DNS query log in AdGuard.

## Risks

Routing policy risks:

- domain list incomplete;
- CDN changes break routing;
- DNS cache causes unexpected route;
- VPN route catches too much traffic;
- anti-DPI rule catches unrelated traffic;
- fallback behavior not documented.

## Итог

Routing policy is based on minimal necessary redirection:

```text
Only route what needs routing.
Keep everything else direct.
```
