# Спецификация сущностей и WireGuard-модели

## Основные сущности

### Network

Виртуальная сеть (аналог Tailscale tailnet / ZeroTier network).

| Поле            | Тип          | Описание                                      |
|-----------------|--------------|-----------------------------------------------|
| id              | UUID         | Уникальный идентификатор                      |
| name            | string       | Человекочитаемое имя                          |
| cidr            | CIDR         | Адресное пространство (например 100.64.0.0/10)|
| dns_servers     | []string     | Список DNS (IP или hostname)                  |
| created_at      | timestamp    |                                               |
| updated_at      | timestamp    |                                               |

### Node

Узел (устройство) в сети.

| Поле              | Тип          | Описание                                      |
|-------------------|--------------|-----------------------------------------------|
| id                | UUID         | Внутренний ID                                 |
| network_id        | UUID         | Принадлежность к Network                      |
| name              | string       | hostname / пользовательское имя               |
| public_key        | string       | WireGuard public key (base64)                 |
| private_key_enc   | string?      | (только на агенте)                            |
| ip                | IP           | Выделенный IP из CIDR сети                    |
| endpoints         | []Endpoint   | Известные endpoints (ip:port)                 |
| status            | enum         | online / offline / unknown                    |
| last_seen         | timestamp    |                                               |
| roles             | []Role       | normal / subnet_router / exit_node            |
| os                | string       | linux / arch / windows / darwin / ...         |
| version           | string       | версия агента                                 |

**Идентификатор узла в WireGuard:** `public_key`.

### Route

Объявленный маршрут.

| Поле            | Тип          | Описание                                      |
|-----------------|--------------|-----------------------------------------------|
| id              | UUID         |                                               |
| network_id      | UUID         |                                               |
| node_id         | UUID         | Узел-holder (router)                          |
| cidr            | CIDR         | Анонсируемая подсеть                          |
| type            | enum         | host / subnet / exit                          |
| enabled         | bool         |                                               |

### DNSConfig

| Поле            | Тип          | Описание                                      |
|-----------------|--------------|-----------------------------------------------|
| network_id      | UUID         |                                               |
| servers         | []string     |                                               |
| search_domains  | []string     | опционально                                   |
| override        | bool         | полностью перезаписывать DNS агента           |

### ExitNode

Специализация Route с `cidr = 0.0.0.0/0` или `::/0` + флаг `is_exit = true`.

На стороне агента-exit требуется:

- IP forwarding
- MASQUERADE / SNAT для трафика из mesh

## WireGuard-модель

### Адресация (IPAM)

Рекомендуемый диапазон по умолчанию: `100.64.0.0/10` (CGNAT, как у Tailscale) или пользовательский приватный CIDR.

Каждому узлу выделяется один IPv4 (и опционально IPv6) из пространства Network.

### Конфигурация peer'а (пример)

```ini
[Interface]
PrivateKey = <node private key>
Address = 100.64.0.5/32
DNS = 100.64.0.1

[Peer]
PublicKey = <peer public key>
AllowedIPs = 100.64.0.6/32, 192.168.10.0/24
Endpoint = 203.0.113.10:51820
PersistentKeepalive = 25
```

Для full mesh Control Plane генерирует полный список peers для каждого узла.

Для больших сетей в будущем — DERP-like relay или иерархия (не в MVP).

### Протокол общения Agent ↔ Control Plane

Минимальный набор:

1. `POST /api/v1/nodes/register` — регистрация (token + public_key + info)
2. `GET  /api/v1/nodes/{id}/config` — получение актуальной конфигурации
3. `POST /api/v1/nodes/{id}/heartbeat` — статус + endpoints
4. (опционально) WebSocket `/api/v1/nodes/{id}/events` — push обновлений

Формат конфигурации: JSON (предпочтительно) или YAML.

## Идентификаторы

- Node ID: UUID v4 (внутренний)
- WireGuard identity: public key
- Network ID: UUID v4
