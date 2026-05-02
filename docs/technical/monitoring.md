# Monitoring

## Назначение документа

Этот документ описывает базовый подход к мониторингу домашней серверной инфраструктуры.

На первом этапе мониторинг должен быть простым и понятным. Не нужно сразу строить сложный observability stack.

## Что нужно мониторить

| Component | What to monitor |
|---|---|
| Mini PC | uptime, temperature, disk health |
| Proxmox | CPU, RAM, storage, VM/LXC status |
| OpenWrt | WAN status, LAN status, DHCP, firewall logs |
| AdGuard Home | DNS query rate, blocked queries, upstream availability |
| VPN | tunnel status, handshake, latency |
| Chatbot | process status, logs, restart count |
| Nextcloud | service status, storage, database, HTTP status |

## Basic monitoring tools

Минимальный набор:

- Proxmox dashboard;
- OpenWrt LuCI status pages;
- AdGuard Home dashboard;
- systemd service status;
- simple ping checks;
- log review.

## Optional tools

Позже можно добавить:

- Uptime Kuma;
- Grafana;
- Prometheus;
- Node Exporter;
- smartmontools;
- Telegram notifications.

## Suggested first monitoring container

Для первого этапа удобно использовать Uptime Kuma.

Планируемая роль:

```text
LXC 104: monitoring
```

Проверки:

```text
OpenWrt web UI
Proxmox web UI
AdGuard DNS port
Chatbot endpoint or process
Nextcloud HTTP endpoint
External internet check
VPN endpoint check
```

## Health checks

Примеры проверок:

| Check | Expected result |
|---|---|
| ping 192.168.10.1 | OpenWrt reachable |
| ping 192.168.10.2 | Proxmox reachable |
| nslookup google.com 192.168.10.3 | AdGuard resolves DNS |
| HTTP check Proxmox | Web UI reachable from LAN |
| HTTP check AdGuard | Web UI reachable from LAN |
| VPN handshake | recent handshake |

## Logs

Полезные источники логов:

Proxmox:

```bash
journalctl -xe
systemctl status pveproxy
systemctl status networking
```

OpenWrt:

```bash
logread
logread -f
```

AdGuard:

```bash
systemctl status AdGuardHome
journalctl -u AdGuardHome
```

Chatbot:

```bash
systemctl status chatbot
journalctl -u chatbot -f
```

## Alerting

На первом этапе достаточно ручной проверки.

Позже можно добавить уведомления:

- Telegram bot;
- email;
- webhook;
- Uptime Kuma notifications.

## What not to overcomplicate

На начальном этапе не нужно сразу внедрять:

- Kubernetes;
- сложный ELK stack;
- Grafana для всего подряд;
- distributed tracing;
- внешние SaaS-мониторинги с публикацией данных.

## Итог

Минимальный мониторинг должен отвечать на вопросы:

```text
Сервер включён?
OpenWrt работает?
Интернет есть?
DNS работает?
VPN поднят?
Диск не забит?
Критичные сервисы живы?
```
