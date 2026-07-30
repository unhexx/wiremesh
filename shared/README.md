# shared

**Ownership:** SpecOrchestrator (+ все остальные как consumers)  
**Status:** Scaffold (Stage 1)

## Purpose

Единый источник правды для **контрактов** между компонентами.

Содержит:
- OpenAPI 3.0 спецификацию Control Plane
- JSON Schema для конфигурации агента (`/nodes/{id}/config`)
- Общие типы / константы (если появятся Go packages)
- В будущем — protobuf / gRPC definitions (если понадобится)

Именно заморозка контрактов в `shared/` позволяет **параллельную разработку** Control Plane и Agent.

## Boundaries

**Владеет:**
- Канонические схемы и спецификации
- Версионирование API (через OpenAPI)

**Не владеет:**
- Реализацией
- Runtime-логикой

## Structure (target)

```
shared/
├── openapi/
│   └── openapi.yaml          # Canonical OpenAPI 3.0
├── schemas/
│   ├── config.schema.json    # Agent config response
│   ├── node.schema.json
│   └── ...
├── go/                       # optional shared Go packages later
│   └── ...
└── README.md
```

## Key Contracts

1. **OpenAPI** → источник для:
   - Генерации серверных stubs (control-plane)
   - Генерации клиентов (web-ui, mcp-server, agent если нужно)
   - Документации

2. **config.schema.json** → точный формат ответа `GET /nodes/{id}/config`, который Agent обязан понимать.

## Workflow

1. SpecOrchestrator обновляет схемы в `shared/`
2. Human gate / review
3. BackendOrchestrator и ClientOrchestrator реализуют против зафиксированных контрактов
4. Breaking changes — только через версию API

## MVP Scope (Stage 1)

- [ ] `shared/openapi/openapi.yaml` (полный черновик из docs/API.md)
- [ ] `shared/schemas/config.schema.json`
- [ ] Документированный протокол heartbeat + ETag
- [ ] Описание Join-token lifecycle

## Related Docs

- [docs/SPEC.md](../docs/SPEC.md)
- [docs/API.md](../docs/API.md)
- [docs/ROADMAP.md](../docs/ROADMAP.md) (Stage 1)
