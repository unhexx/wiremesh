# WireMesh Agent (Arch Linux)

## Цели MVP

- Установка одной командой через pacman / AUR
- Join одной командой
- Автоматическое применение WireGuard-конфигурации
- systemd-юнит с автозапуском и перезапуском
- Heartbeat и получение обновлений конфигурации

## Установка (целевая)

```bash
# Через AUR (планируется)
yay -S wiremesh-agent

# Или вручную
pacman -U wiremesh-agent-0.1.0-1-x86_64.pkg.tar.zst
```

## Команды CLI

```bash
wiremesh-agent join <control-plane-url> --token <token>
wiremesh-agent status
wiremesh-agent leave
wiremesh-agent logs
wiremesh-agent config show
```

## Поведение при join

1. Генерирует WireGuard keypair (если ещё нет).
2. Отправляет `POST /api/v1/nodes/register`.
3. Получает IP + peers + DNS + routes.
4. Записывает конфиг в `/etc/wiremesh/wg0.conf` (или использует netlink напрямую).
5. Поднимает интерфейс `wg0` через `wg-quick up` или эквивалент.
6. Настраивает DNS (через `resolvectl` / systemd-resolved предпочтительно).
7. Запускает/перезапускает systemd-сервис.
8. Начинает периодический heartbeat.

## Systemd unit (пример)

```ini
[Unit]
Description=WireMesh Agent
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/bin/wiremesh-agent daemon
Restart=on-failure
RestartSec=5
StateDirectory=wiremesh
ConfigurationDirectory=wiremesh

[Install]
WantedBy=multi-user.target
```

## Обновление конфигурации

Варианты (MVP — pull):

- Периодический `GET /nodes/{id}/config` (например, каждые 30–60 с)
- Сравнение checksum / ETag
- При изменении — атомарное применение нового конфига + `wg syncconf`

Позже: WebSocket push.

## Зависимости

- `wireguard-tools`
- `systemd`
- (опционально) `nftables` для exit-node MASQUERADE

## PKGBUILD (заготовка)

Будет добавлен в `packaging/arch/`.
