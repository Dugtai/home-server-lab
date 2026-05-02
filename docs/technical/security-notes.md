# Security Notes

## Назначение документа

Этот документ фиксирует базовые требования безопасности для проекта `home-server-lab`.

Проект является домашней инфраструктурой, но должен проектироваться с учётом принципов минимизации рисков.

## Основные принципы

- не публиковать административные панели в интернет;
- не хранить секреты в GitHub;
- разделять роли сервисов;
- ограничивать доступ к управлению;
- регулярно обновлять компоненты;
- делать резервные копии до изменений;
- документировать сетевые изменения.

## Proxmox security

Proxmox должен быть доступен только из LAN или через защищённый удалённый доступ.

Не рекомендуется:

- открывать порт `8006` в интернет;
- назначать IP на WAN bridge;
- использовать слабый пароль root;
- хранить пароль root в открытом виде;
- использовать один пароль для всех сервисов.

Рекомендуется:

- использовать сложный пароль;
- ограничить доступ к панели управления;
- обновлять Proxmox;
- делать резервные копии VM/LXC;
- использовать отдельные учётные записи при необходимости.

## OpenWrt security

OpenWrt является основным сетевым шлюзом.

Рекомендации:

- не открывать LuCI в WAN;
- не включать лишние службы на WAN;
- проверять firewall zones;
- отключать ненужные пакеты;
- документировать PBR и VPN правила;
- сохранять бэкап конфигурации перед изменениями.

## AdGuard Home security

AdGuard Home должен быть доступен только из LAN.

Не рекомендуется:

- публиковать веб-интерфейс AdGuard в интернет;
- использовать слабый пароль;
- публиковать `AdGuardHome.yaml` без очистки;
- включать слишком много фильтров без диагностики.

## Secrets management

В GitHub нельзя публиковать:

- `.env`;
- private keys;
- VPN configs with real keys;
- bot tokens;
- API tokens;
- database passwords;
- PPPoE credentials;
- real public IP if it should remain private;
- DDNS names if they should remain private.

В репозитории допускаются только шаблоны:

```text
.env.example
wg0.conf.example
awg0.conf.example
pbr-rules.example.md
AdGuardHome.example.yaml
```

## Git security

`.gitignore` снижает риск случайной публикации файлов, но не является абсолютной защитой.

Если секрет уже попал в Git history, простое удаление файла не решает проблему.

В таком случае нужно:

1. считать секрет скомпрометированным;
2. заменить ключ/токен/пароль;
3. очистить историю Git при необходимости;
4. проверить репозиторий на наличие утечек.

## Network exposure

Компоненты, которые не должны быть доступны из интернета:

| Service | External access |
|---|---|
| Proxmox Web UI | No |
| OpenWrt LuCI | No |
| AdGuard Web UI | No |
| LXC SSH | No direct WAN |
| Nextcloud | Only after hardening/reverse proxy/backups |
| Chatbot admin endpoints | No direct WAN |

## Nextcloud security

Nextcloud не должен публиковаться наружу до настройки:

- HTTPS;
- резервного копирования;
- обновлений;
- сильных паролей;
- ограничений доступа;
- проверки reverse proxy;
- мониторинга логов.

На первом этапе Nextcloud лучше держать только в LAN или через VPN.

## Backup security

Бэкапы могут содержать секреты.

Нельзя публиковать:

- VM dumps;
- LXC backups;
- database dumps;
- Nextcloud data;
- OpenWrt backup archives;
- AdGuard full config with credentials.

Бэкапы должны храниться отдельно от основного SSD.

## Operational safety

Перед изменениями:

- сохранить текущий конфиг;
- сделать скриншот или запись параметров;
- менять один параметр за раз;
- проверять результат после каждого изменения;
- иметь локальный доступ к mini PC через монитор и клавиатуру.

## Итог

Главный принцип безопасности проекта:

```text
Management interfaces stay inside LAN.
Secrets stay outside Git.
Backups are mandatory before risky changes.
```
