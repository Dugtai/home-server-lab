# PBR Rules Example

This file contains sanitized examples of policy-based routing rules.

Actual rules depend on the selected VPN interface, DNS mode and OpenWrt package version.

## Interfaces

Example interface names:

```text
wan  -> direct ISP route
wg0  -> WireGuard VPN route
awg0 -> AmneziaWG VPN route
```

## Example routing policy

| Name | Source | Destination | Interface |
|---|---|---|---|
| OpenAI via VPN | LAN clients | chatgpt.com, openai.com | wg0 |
| Instagram via VPN | LAN clients | instagram.com, cdninstagram.com | wg0 |
| Telegram via VPN | LAN clients | telegram.org, t.me | wg0 |
| YouTube direct | LAN clients | youtube.com, googlevideo.com | wan + anti-DPI |
| Discord direct | LAN clients | discord.com, discordcdn.com | wan + anti-DPI |
| Banks direct | LAN clients | selected bank domains | wan |

## Example domain groups

### OpenAI

```text
chatgpt.com
openai.com
auth.openai.com
oaistatic.com
oaiusercontent.com
```

### Instagram / Meta

```text
instagram.com
cdninstagram.com
facebook.com
fbcdn.net
messenger.com
```

### Telegram

```text
telegram.org
t.me
telegram.me
```

### YouTube anti-DPI group

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

### Discord anti-DPI group

```text
discord.com
discord.gg
discordapp.com
discordapp.net
discordcdn.com
discord.media
```

## Notes

- YouTube and Discord should not be routed through VPN if anti-DPI processing works correctly.
- Banks and government services should usually remain on direct WAN.
- Domain lists require validation because services can change CDN and hostnames.
- Test one rule at a time.
