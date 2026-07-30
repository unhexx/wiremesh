# control-plane

**Ownership:** BackendOrchestrator  
**Status:** Scaffold (Stage 2)

## Purpose

Координационный сервер (Control Plane) WireMesh.

Отвечает за:
- Регистрацию узлов и выдачу WireGuard-конфигураций
- Хранение состояния (nodes, networks, routes, DNS, exit-nodes, join-tokens)
- Генерацию peer lists (full mesh для MVP)
- REST API, используемый Agent, Web UI и MCP-сервером
- Auth (join-tokens → позже session/OIDC)

## Boundaries

**Владеет:**
- HTTP/REST API (`/api/v1/...`)
- Persistent storage (SQLite → PostgreSQL)
- Логика IPAM и пересборки AllowedIPs
- Join-token lifecycle

**Не владеет:**
- Применение WireGuard-конфигов (это `agent/`)
- UI (это `web-ui/`)
- MCP protocol surface (это `mcp-server/`, хотя может быть встроен позже)

## Tech Stack (рекомендация MVP)

- Language: **Go 1.22+**
- Router: chi или echo
- Storage: SQLite (modernc.org/sqlite или gorm) → PostgreSQL
- Config: env + yaml
- Logging: slog / zerolog
- Docker multi-stage build

## Key Interfaces

| Provides | Consumes |
|----------|----------|
| REST API (OpenAPI) | `shared/openapi/`, `shared/schemas/` |
| Config JSON для агента | — |
| Join tokens | — |

См. также:
- [docs/API.md](../docs/API.md)
- [docs/SPEC.md](../docs/SPEC.md)
- [shared/](../shared/)

## Local Development

```bash
# из корня репозитория (после появления go.mod)
go run ./control-plane/cmd/server

# или
cd control-plane && go run ./cmd/server
```

Environment variables (план):
- `DATABASE_URL`
- `BASE_URL`
- `JWT_SECRET` / token signing key
- `LISTEN_ADDR=:8080`
- `DEFAULT_NETWORK_CIDR=100.64.0.0/10`

## Build & Run (target)

```bash
docker build -t wiremesh-control-plane -f control-plane/Dockerfile .
docker compose -f deploy/docker-compose.yml up control-plane
```

## MVP Scope (Stage 2)

- [ ] POST /nodes/register
- [ ] GET /nodes/{id}/config (+ ETag)
- [ ] POST /nodes/{id}/heartbeat
- [ ] Join-token CRUD
- [ ] Full-mesh peer generation
- [ ] SQLite persistence
- [ ] Dockerfile + healthcheck

## Dependencies

- **shared/** — OpenAPI + JSON schemas (must be frozen before serious implementation)
- **deploy/** — docker-compose integration

## Related Docs

- [Architecture](../docs/ARCHITECTURE.md)
- [Roadmap Stage 2](../docs/ROADMAP.md)
- [API draft](../docs/API.md)
