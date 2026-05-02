# Proxmox Storage Plan

## Назначение документа

Этот документ фиксирует базовый план использования SSD 512 GB в mini PC.

## Current storage

```text
Internal SSD: 512 GB
```

## Recommended usage

| Purpose | Approximate size |
|---|---|
| Proxmox system | 32-64 GB |
| OpenWrt VM | 4-8 GB |
| AdGuard LXC | 8 GB |
| Chatbot LXC | 10-30 GB |
| Monitoring LXC | 8-16 GB |
| Nextcloud test | 100-150 GB |
| Local temporary backups | 50-100 GB |
| Free reserve | remaining space |

## Important note about Nextcloud

512 GB is enough for a test Nextcloud instance, but not ideal for long-term storage.

For production-like use, Nextcloud data should be placed on separate storage with backups.

## Backup storage

Recommended later:

```text
External HDD/SSD -> Proxmox backups and Nextcloud backup
```

## What not to store only on internal SSD

Do not keep the only copy of important data on the internal SSD.

Risk examples:

- SSD failure;
- filesystem corruption;
- accidental VM deletion;
- failed update;
- ransomware on client-synced files;
- configuration mistake.

## Итог

The internal 512 GB SSD is enough for the initial lab, but external backup storage is required before storing important data.
