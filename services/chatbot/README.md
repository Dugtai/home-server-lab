# Chatbot Service

## Назначение

Этот раздел предназначен для будущего чатбота, который будет работать в отдельном LXC-контейнере.

## Planned container

```text
LXC 102: chatbot
IP: 192.168.10.4
CPU: 1-2 cores
RAM: 1-2 GB
Disk: 10-30 GB
```

## Runtime

Возможные варианты:

- Python;
- Node.js;
- systemd service;
- Docker inside LXC only if required.

## Secrets

Real `.env` files must not be committed.

Use:

```text
.env.example
```

Local real file:

```text
.env
```

## Logs

Logs should be stored locally and excluded from Git.

## Autostart

Bot process should be managed by systemd.

Example planned service file:

```text
services/chatbot/systemd-service.example
```
