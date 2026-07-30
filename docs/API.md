# REST API Control Plane (черновик v0.1)

Базовый путь: `/api/v1`

Аутентификация:
- Join-токены для регистрации узлов
- Session / API-token / OIDC для административных операций (Web UI и MCP)

## Nodes

### POST /nodes/register

Регистрация нового узла.

**Request:**
```json
{
  "token": "join-token-here",
  "public_key": "base64-wireguard-public-key",
  "name": "hostname-or-custom",
  "os": "arch",
  "version": "0.1.0",
  "endpoints": ["203.0.113.5:51820"]
}
```

**Response 201:**
```json
{
  "id": "uuid",
  "ip": "100.64.0.5",
  "network_id": "uuid",
  "config": { ... }  // см. Config ниже
}
```

### GET /nodes

Список узлов (admin).

### GET /nodes/{id}

Информация об узле.

### GET /nodes/{id}/config

Актуальная WireGuard-конфигурация + routes + dns.

**Response:**
```json
{
  "interface": {
    "private_key": "...",          // только если агент ещё не имеет
    "address": "100.64.0.5/32",
    "listen_port": 51820
  },
  "peers": [
    {
      "public_key": "...",
      "allowed_ips": ["100.64.0.6/32", "10.0.0.0/24"],
      "endpoint": "203.0.113.10:51820",
      "persistent_keepalive": 25
    }
  ],
  "dns": ["100.64.0.1"],
  "routes": [ ... ]
}
```

### POST /nodes/{id}/heartbeat

```json
{
  "endpoints": ["..."] ,
  "status": "online",
  "metrics": { ... } // опционально
}
```

### PATCH /nodes/{id}/roles

```json
{
  "roles": ["subnet_router", "exit_node"]
}
```

## Routes

### GET /routes

### POST /routes

```json
{
  "node_id": "uuid",
  "cidr": "192.168.10.0/24",
  "type": "subnet"
}
```

### DELETE /routes/{id}

## DNS

### GET /networks/{id}/dns

### PUT /networks/{id}/dns

```json
{
  "servers": ["100.64.0.1", "1.1.1.1"],
  "search_domains": ["internal.example"],
  "override": true
}
```

## Exit Nodes

### POST /nodes/{id}/exit

Включить/выключить режим exit node.

### GET /exit-nodes

Список доступных exit-узлов.

## Networks

### POST /networks

Создание новой виртуальной сети.

### GET /networks

## Auth / Tokens

### POST /tokens/join

Генерация join-токена (admin).

```json
{
  "network_id": "uuid",
  "expires_in": 3600,
  "max_uses": 1
}
```

---

*API будет уточняться по мере реализации. Рекомендуется OpenAPI 3.0 спецификация в `/openapi.yaml`.*
