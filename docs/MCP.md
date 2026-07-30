# MCP-интеграция WireMesh

MCP (Model Context Protocol) используется как стандартный способ предоставления LLM-агентам доступа к управлению виртуальной сетью.

MCP-сервер выступает фасадом над REST/gRPC API Control Plane.

## Транспорт

- Local: stdio (для локальных агентов)
- Remote: Streamable HTTP / SSE (для удалённых клиентов)

## Resources (примеры)

| URI / Name              | Описание                              |
|-------------------------|---------------------------------------|
| `wiremesh://nodes`      | Список всех узлов                     |
| `wiremesh://nodes/{id}` | Детали конкретного узла               |
| `wiremesh://routes`     | Текущие маршруты                      |
| `wiremesh://dns`        | Конфигурация DNS                      |
| `wiremesh://exit-nodes` | Доступные exit-узлы                   |
| `wiremesh://networks`   | Список виртуальных сетей              |

## Tools (примеры)

### nodes.list

Возвращает список узлов с статусами.

### nodes.set_role

```json
{
  "node_id": "uuid",
  "roles": ["exit_node"]
}
```

### routes.add

```json
{
  "node_id": "uuid",
  "cidr": "10.20.30.0/24"
}
```

### routes.remove

```json
{
  "route_id": "uuid"
}
```

### dns.set_servers

```json
{
  "network_id": "uuid",
  "servers": ["100.64.0.1"]
}
```

### exit_nodes.enable / exit_nodes.disable

### networks.create

Создание новой сети (для advanced-сценариев).

## Capability Negotiation

Сервер объявляет поддерживаемые resources и tools при инициализации сессии.

## Примеры использования

LLM-агент может:

- Автоматически поднимать exit-node при определённых условиях нагрузки
- Создавать временные сети для CI/CD или review-окружений
- Аудитить текущее состояние маршрутов и DNS
- Реагировать на offline-узлы

## Реализация

Рекомендуется использовать официальные SDK MCP (TypeScript / Python / Go) и маппить вызовы на внутренний API.

См. также: [спецификация MCP](https://modelcontextprotocol.io).
