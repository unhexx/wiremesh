# WireMesh

**Самохостируемый сервис виртуальной mesh-сети** (аналог Tailscale / ZeroTier) с собственным доменом, control plane, Web UI и полной интеграцией MCP.

WireMesh строит безопасную L3-сеть поверх существующей инфраструктуры с помощью WireGuard в качестве data plane. Полностью самохостируемый: свой control plane, свой домен, без зависимости от внешних SaaS.

## Ключевые возможности

- **Mesh-сеть на WireGuard** — прямые peer-to-peer туннели, zero-trust, end-to-end шифрование
- **Собственный Control Plane** — регистрация узлов, выдача ключей, маршрутов, DNS, exit-nodes
- **Один шаг подключения** — `pacman -S wiremesh-agent && wiremesh-agent join https://net.example.com --token ...`
- **Управление через Web UI** — маршруты, DNS, exit-узлы, ACL, статусы узлов
- **MCP-сервер** — LLM-агенты (Grok, Claude, локальные) могут управлять сетью через стандартный Model Context Protocol
- **Первая платформа** — Arch Linux (AUR/pacman), далее Linux, macOS, Windows, *BSD

## Структура репозитория (monorepo для параллельной разработки)

```
wiremesh/
├── control-plane/     # BackendOrchestrator — координационный сервер, REST API, storage, config generation
├── agent/             # ClientOrchestrator  — узел (Go CLI + daemon + WireGuard + systemd + PKGBUILD)
├── web-ui/            # FrontendOrchestrator — Admin SPA
├── mcp-server/        # MCPOrchestrator     — MCP resources & tools для LLM-агентов
├── shared/            # SpecOrchestrator    — OpenAPI, JSON Schemas, общие контракты
├── deploy/            # Release / Infra     — docker-compose, примеры конфигурации
├── scripts/           # Вспомогательные скрипты и e2e
└── docs/              # Высокоуровневая документация проекта
```

Каждый компонент имеет собственный `README.md` с описанием границ ответственности, tech stack, интерфейсов, ownership и MVP-скоупа. Это позволяет вести разработку параллельными потоками после заморозки контрактов в `shared/`.

| Компонент | Ownership | Зависит от |
|-----------|-----------|------------|
| [control-plane](control-plane/) | BackendOrchestrator | shared |
| [agent](agent/) | ClientOrchestrator | shared + control-plane API |
| [web-ui](web-ui/) | FrontendOrchestrator | control-plane API |
| [mcp-server](mcp-server/) | MCPOrchestrator | control-plane API |
| [shared](shared/) | SpecOrchestrator | — |
| [deploy](deploy/) | ReleaseOrchestrator | control-plane |

## Быстрый старт (целевое состояние)

```bash
# Установка агента (Arch)
pacman -S wiremesh-agent

# Подключение к своему control plane
wiremesh-agent join https://net.example.com --token <join-token>
```

После join узел появляется в Web UI и автоматически получает конфигурацию WireGuard, маршруты и DNS.

## Архитектура (кратко)

| Компонент              | Назначение                                      |
|------------------------|-------------------------------------------------|
| **Control Plane**      | Координационный сервер + REST + (позже gRPC)    |
| **Data Plane**         | WireGuard mesh (point-to-point / full mesh)     |
| **Agent**              | Клиент узла (systemd, ключи, heartbeat)         |
| **Web UI**             | SPA для администратора                          |
| **MCP Server**         | Адаптер для LLM-агентов и автоматизации         |

Подробности: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)

## Документация

**Высокоуровневая (docs/):**
- [Архитектура](docs/ARCHITECTURE.md)
- [Спецификация сущностей и WireGuard-модели](docs/SPEC.md)
- [REST API](docs/API.md)
- [MCP-интеграция](docs/MCP.md)
- [Агент (Arch Linux)](docs/AGENT.md)
- [Развёртывание](docs/DEPLOYMENT.md)
- [Дорожная карта MVP](docs/ROADMAP.md)
- [Исходное видение](docs/PROJECT-BRIEF.md)

**Компонентная:**
- [control-plane/README.md](control-plane/README.md)
- [agent/README.md](agent/README.md)
- [web-ui/README.md](web-ui/README.md)
- [mcp-server/README.md](mcp-server/README.md)
- [shared/README.md](shared/README.md)
- [deploy/README.md](deploy/README.md)

## Статус

Проект находится на этапе проектирования и подготовки структуры. Код появится в следующих итерациях согласно [ROADMAP](docs/ROADMAP.md).

Текущий фокус: **Stage 1 — Spec Freeze** (OpenAPI + schemas в `shared/`).

## Лицензия

MIT License — см. [LICENSE](LICENSE).

---

*WireMesh — часть экосистемы инструментов для самохостируемых агентных систем.*
