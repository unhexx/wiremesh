# mcp-server

**Ownership:** MCPOrchestrator  
**Status:** Scaffold (Stage 5)

## Purpose

MCP (Model Context Protocol) сервер, который экспонирует ресурсы и инструменты WireMesh для LLM-агентов (Grok, Claude Desktop, локальные агенты и т.д.).

Позволяет агентам:
- Читать состояние сети (nodes, routes, dns, exit-nodes)
- Выполнять управляющие действия (добавить маршрут, включить exit-node, сменить DNS и т.п.)

## Boundaries

**Владеет:**
- MCP protocol surface (resources + tools + capability negotiation)
- Адаптацию вызовов MCP → REST API Control Plane

**Не владеет:**
- Хранением состояния и бизнес-логикой (всё делегируется Control Plane)

Может быть:
1. Отдельным процессом (рекомендуется для MVP и чёткого ownership)
2. Встроенным в control-plane позже (если захотим один бинарь)

## Tech Stack (рекомендация)

- **Go** (официальный / community MCP SDK) **или** Python (anthropic MCP SDK) — выбрать на Stage 5
- Транспорт: stdio (local) + Streamable HTTP / SSE (remote)

## Key Interfaces

| Provides | Consumes |
|----------|----------|
| MCP Resources & Tools | Control Plane REST API |
| Capability negotiation | `shared/` schemas (для типизации) |

Примеры tools (из docs/MCP.md):
- `nodes.list` / `nodes.set_role`
- `routes.add` / `routes.remove`
- `dns.set_servers`
- `exit_nodes.enable` / `disable`

## Local Development

```bash
# пример
go run ./mcp-server/cmd/server --control-plane http://localhost:8080
# или
python -m mcp_server --control-plane-url http://localhost:8080
```

## MVP Scope (Stage 5)

- [ ] Resources: nodes, routes, dns, exit-nodes, networks
- [ ] Tools: основные mutating операции
- [ ] Capability negotiation
- [ ] stdio + хотя бы один remote transport
- [ ] Пример использования с Claude Desktop / локальным MCP-клиентом
- [ ] Документация

## Dependencies

- **control-plane** — все реальные действия идут через его API
- **shared/** — желательно иметь типизированные схемы

## Related Docs

- [docs/MCP.md](../docs/MCP.md)
- [docs/API.md](../docs/API.md)
- [Roadmap Stage 5](../docs/ROADMAP.md)
