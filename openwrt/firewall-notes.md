# OpenWrt Firewall Notes

## Назначение документа

Этот документ фиксирует базовые принципы firewall-настройки OpenWrt в проекте.

## Zones

Expected zones:

| Zone | Role |
|---|---|
| lan | trusted home network |
| wan | untrusted internet side |
| vpn | VPN tunnel zone, if required |

## Basic policy

Recommended logic:

```text
LAN -> WAN: allowed
WAN -> LAN: denied
LAN -> OpenWrt management: allowed
WAN -> OpenWrt management: denied
LAN -> VPN: allowed by PBR only
```

## Management access

LuCI and SSH should not be exposed to WAN.

Allowed:

```text
LAN -> OpenWrt LuCI
LAN -> OpenWrt SSH
```

Denied:

```text
WAN -> OpenWrt LuCI
WAN -> OpenWrt SSH
WAN -> Proxmox
WAN -> AdGuard
```

## DNS

Clients should use AdGuard Home as DNS:

```text
DNS server: 192.168.10.3
```

Optional future improvement:

- redirect all LAN DNS requests to AdGuard;
- block external DNS bypass;
- document DoH/DoT policy.

## VPN zone

If VPN interface exists:

```text
wg0 or awg0 -> vpn zone
```

PBR should decide what traffic uses VPN.

## Anti-DPI

Anti-DPI processing for YouTube/Discord should be documented separately.

It should not expose management services and should not replace firewall policy.

## What not to do

Do not:

- open Proxmox to WAN;
- open AdGuard admin UI to WAN;
- open OpenWrt LuCI to WAN;
- allow inbound WAN traffic without a documented reason;
- create broad port forwards without notes;
- publish real firewall configs with sensitive IPs.

## Итог

Default firewall posture:

```text
LAN trusted, WAN untrusted, management only from LAN.
```
