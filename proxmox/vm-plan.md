# Proxmox VM/LXC Plan

## Назначение документа

Этот документ фиксирует план виртуальных машин и LXC-контейнеров для проекта.

## Planned resources

| ID | Type | Name | CPU | RAM | Disk | Network | IP |
|---|---|---|---|---|---|---|---|
| 100 | VM | openwrt-gateway | 2 cores | 1024 MB | 4-8 GB | vmbr0 + vmbr1 | 192.168.10.1 |
| 101 | LXC | adguard-home | 1 core | 512 MB | 8 GB | vmbr1 | 192.168.10.3 |
| 102 | LXC | chatbot | 1-2 cores | 1-2 GB | 10-30 GB | vmbr1 | 192.168.10.4 |
| 103 | VM/LXC | nextcloud | 2 cores | 3-4 GB | 100-150 GB test | vmbr1 | 192.168.10.5 |
| 104 | LXC | monitoring | 1 core | 512 MB-1 GB | 8-16 GB | vmbr1 | TBD |

## Critical services

Critical path:

```text
Proxmox -> OpenWrt VM -> LAN/WAN connectivity
```

If OpenWrt VM is down, home internet through this gateway is down.

## Startup order

Recommended startup order:

1. OpenWrt VM.
2. AdGuard Home LXC.
3. Chatbot LXC.
4. Monitoring LXC.
5. Nextcloud VM/LXC.

## Autostart

OpenWrt VM should have autostart enabled.

AdGuard Home should also autostart after OpenWrt.

Application services can start later.

## Notes

- OpenWrt should be a VM, not LXC.
- AdGuard is suitable for LXC.
- Nextcloud should not be deployed before backup planning.
- Chatbot secrets must not be committed to GitHub.
