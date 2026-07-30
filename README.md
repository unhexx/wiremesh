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
| **Control Plane**      | Координационный сервер + REST/gRPC + MCP        |
| **Data Plane**         | WireGuard mesh (point-to-point / full mesh)     |
| **Agent**              | Клиент узла (systemd, ключи, heartbeat)         |
| **Web UI**             | SPA для администратора                          |
| **MCP Server**         | Адаптер для LLM-агентов и автоматизации         |

Подробности: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)

## Документация

- [Архитектура](docs/ARCHITECTURE.md)
- [Спецификация сущностей и WireGuard-модели](docs/SPEC.md)
- [REST API](docs/API.md)
- [MCP-интеграция](docs/MCP.md)
- [Агент (Arch Linux)](docs/AGENT.md)
- [Развёртывание Control Plane](docs/DEPLOYMENT.md)
- [Дорожная карта (MVP)](docs/ROADMAP.md)
- [Исходное видение проекта](docs/PROJECT-BRIEF.md)

## Статус

Проект находится на этапе проектирования и подготовки документации. Код control plane и агента появится в следующих итерациях согласно [ROADMAP](docs/ROADMAP.md).

## Лицензия

MIT License — см. [LICENSE](LICENSE).

---

*WireMesh — часть экосистемы инструментов для самохостируемых агентных систем.*
