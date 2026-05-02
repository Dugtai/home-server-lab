# Backup Strategy

## Назначение документа

Этот документ описывает стратегию резервного копирования для проекта `home-server-lab`.

Главная идея: домашний сервер не должен быть единственной точкой хранения важных данных. Любой SSD, mini PC, контейнер или виртуальная машина могут выйти из строя.

## Что нужно резервировать

| Компонент | Что резервировать | Приоритет |
|---|---|---|
| Proxmox VE | настройки сети, список VM/LXC, заметки по storage | высокий |
| OpenWrt VM | backup конфигурации OpenWrt | высокий |
| AdGuard Home | настройки, фильтры, DNS rewrites | средний |
| Chatbot | код, `.env` локально, systemd service | высокий |
| Nextcloud | config, database, user data | критический |
| Documentation | GitHub repository | высокий |

## Что не является полноценным бэкапом

Не считается полноценным резервным копированием:

- хранить данные только на SSD mini PC;
- держать копию рядом на том же диске;
- надеяться на RAID без отдельного бэкапа;
- хранить только конфиг без данных;
- хранить только данные без базы данных;
- держать бэкапы без проверки восстановления.

## Уровни бэкапа

### Level 1 — Documentation backup

GitHub-репозиторий хранит:

- инструкции;
- архитектурные решения;
- обезличенные примеры конфигов;
- чек-листы;
- схемы.

GitHub не должен хранить реальные секреты.

### Level 2 — Configuration backup

Периодически сохранять:

- OpenWrt backup archive;
- AdGuard sanitized config;
- список установленных пакетов;
- Proxmox VM/LXC plan;
- systemd service files;
- `.env.example`, но не реальный `.env`.

### Level 3 — VM/LXC backup

Для Proxmox использовать встроенные backup-механизмы VM/LXC.

Рекомендуемый подход:

```text
OpenWrt VM -> backup before major network changes
AdGuard LXC -> weekly backup
Chatbot LXC -> before updates
Nextcloud -> only with database + data backup consistency
```

### Level 4 — Data backup

Для Nextcloud и пользовательских данных нужен отдельный носитель.

Рекомендуемо:

```text
System SSD -> VM/LXC/system services
External HDD/SSD -> backups and data copies
Cloud/offsite copy -> critical documents only
```

## Правило 3-2-1

Классическая схема:

```text
3 copies of important data
2 different media
1 offsite copy
```

Для домашнего проекта можно начать проще:

```text
Main data on server
Local backup on external disk
Important documents copy outside home
```

## Backup schedule

Пример расписания:

| Объект | Частота |
|---|---|
| OpenWrt config | перед каждым крупным изменением |
| Proxmox VM/LXC | еженедельно или перед обновлениями |
| AdGuard config | после изменения фильтров/DNS |
| Chatbot code | через Git |
| Chatbot secrets | локальный защищённый backup |
| Nextcloud data | ежедневно/еженедельно, зависит от важности |
| Documentation | каждый коммит в GitHub |

## Restore testing

Бэкап ценен только тогда, когда его можно восстановить.

Минимальные проверки:

- открыть архив;
- проверить список файлов;
- восстановить тестовый LXC;
- проверить импорт OpenWrt config;
- проверить, что документация содержит актуальные IP и роли.

## Security of backups

Бэкапы часто содержат секреты.

Нельзя публиковать:

- Proxmox VM dumps;
- OpenWrt backup archives без очистки;
- AdGuardHome.yaml с реальными данными;
- Nextcloud database dumps;
- реальные `.env` файлы;
- VPN-конфиги с private keys.

## Что сделать на первом этапе

До установки Nextcloud достаточно:

1. Вести документацию в GitHub.
2. Хранить реальные секреты локально.
3. Делать backup OpenWrt перед крупными изменениями.
4. Делать backup LXC AdGuard после настройки.
5. Сохранить список VM/LXC и IP-адресов.

## Итог

Минимально допустимый подход:

```text
Documentation in GitHub
Secrets outside GitHub
OpenWrt config backup before changes
Proxmox VM/LXC backup after stable setup
External storage before Nextcloud production use
```
