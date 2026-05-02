# AdGuard Home Setup Notes

## Назначение документа

Этот документ фиксирует заметки по настройке AdGuard Home в проекте.

## Planned IP

```text
AdGuard Home: 192.168.10.3
```

## Role

AdGuard Home provides:

- DNS filtering;
- ad blocking;
- tracker blocking;
- local DNS rewrites;
- query log;
- DNS diagnostics.

## Install target

```text
LXC 101: adguard-home
OS: Debian 12
CPU: 1 core
RAM: 512 MB
Disk: 8 GB
Network: vmbr1
```

## OpenWrt DHCP option

OpenWrt should provide AdGuard as DNS to clients:

```text
6,192.168.10.3
```

## Local DNS rewrites

```text
router.home.arpa      -> 192.168.10.1
proxmox.home.arpa     -> 192.168.10.2
adguard.home.arpa     -> 192.168.10.3
bot.home.arpa         -> 192.168.10.4
cloud.home.arpa       -> 192.168.10.5
ap.home.arpa          -> 192.168.10.10
```

## Security

Do not expose AdGuard Home web UI to WAN.

Do not commit real `AdGuardHome.yaml` if it contains sensitive data.

## Backup

After stable setup:

- export settings;
- document filter lists;
- document local DNS rewrites;
- keep sensitive backup outside GitHub.
