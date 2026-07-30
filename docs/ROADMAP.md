# Дорожная карта WireMesh (MVP и далее)

## Этап 1 — Архитектура и спецификации (текущий)

- [x] Высокоуровневое описание
- [x] Сущности (Node, Network, Route, DNS, ExitNode)
- [x] WireGuard-модель и IPAM
- [x] Черновик REST API
- [x] Черновик MCP resources/tools
- [ ] OpenAPI 3.0 спецификация
- [ ] Детальный протокол agent ↔ control plane (включая формат конфигов)

## Этап 2 — Control Plane (сервер)

- [ ] Базовый HTTP-сервер (предпочтительно Go + chi/echo или FastAPI)
- [ ] Модели + миграции (PostgreSQL / SQLite)
- [ ] `POST /nodes/register`
- [ ] `GET /nodes/{id}/config`
- [ ] Управление routes / dns / exit
- [ ] Join-токены
- [ ] Простой in-memory или файловый бэкап состояния (для самого первого прототипа)
- [ ] Минимальный Web UI (или даже CLI-admin сначала)

## Этап 3 — Agent для Arch Linux

- [ ] Бинарь агента (Go предпочтительно — один статический бинарь)
- [ ] Генерация ключей WireGuard
- [ ] Join-флоу
- [ ] Применение конфига (wg-quick или netlink)
- [ ] systemd unit
- [ ] PKGBUILD + AUR
- [ ] Heartbeat + pull обновлений

## Этап 4 — Управление маршрутами, DNS, exit-узлами

- [ ] Полная логика пересборки peer-конфигов
- [ ] Поддержка subnet router
- [ ] Поддержка exit node + документация по nftables/iptables
- [ ] Web UI формы и таблицы
- [ ] Обновление агентов при изменениях

## Этап 5 — MCP-сервер

- [ ] Реализация MCP-сервера (resources + tools)
- [ ] Capability negotiation
- [ ] Примеры клиентов (CLI + интеграция с LLM)
- [ ] Документация использования с Claude Desktop / локальными агентами

## Этап 6 — Тестирование и демо-сценарии

- [ ] Два Arch-хоста в одной сети
- [ ] Subnet router к локальной LAN
- [ ] Exit node + проверка интернет-трафика
- [ ] Управление через Web UI и MCP

## Дальнейшее развитие (после MVP)

- Кросс-платформенность (Debian, Fedora, macOS, Windows, *BSD)
- OIDC / SAML
- ACL (кто к кому и по каким портам)
- DERP-like relays / NAT traversal улучшения
- CI/CD интеграция (временные сети)
- Prometheus-метрики + Grafana
- Session recording (по аналогии с Tailscale SSH)
- Multi-network на одном агенте
- MagicDNS-подобный сервис

## Приоритеты ближайших задач

1. Зафиксировать OpenAPI + точный формат config JSON.
2. Минимальный Control Plane на Go (register + config + heartbeat).
3. Минимальный Agent на Go, который реально поднимает wg0.
4. Один e2e-сценарий «два узла видят друг друга».
