# web-ui

**Ownership:** FrontendOrchestrator  
**Status:** Scaffold (Stage 4)

## Purpose

Web Management Console для администратора WireMesh.

Позволяет:
- Видеть список узлов и их статус (online/offline, last_seen)
- Управлять маршрутами (добавить/удалить subnet)
- Назначать exit-node
- Настраивать DNS виртуальной сети
- Генерировать join-токены
- Базовые ACL / политики (после MVP)

## Boundaries

**Владеет:**
- SPA (frontend only)
- Вызовы REST API Control Plane
- UX для admin-операций

**Не владеет:**
- Бизнес-логикой и хранением (всё на Control Plane)
- Аутентификацией узлов (join-токены выдаёт backend)

## Tech Stack (рекомендация)

- **SvelteKit** (или Vue 3 + Vite / React + Vite) — выбрать один на Stage 4
- Tailwind CSS
- TypeScript
- Fetch / ofetch к Control Plane API
- Опционально: TanStack Query

Для MVP достаточно простого SPA без SSR.

## Key Interfaces

| Provides | Consumes |
|----------|----------|
| Browser UI | Control Plane REST API (`/api/v1/...`) |
| — | OpenAPI types (generated from `shared/openapi/`) |

## Local Development

```bash
cd web-ui
npm install
npm run dev
```

Proxy к Control Plane обычно настраивается в vite/svelte.config.

## MVP Scope (Stage 4)

- [ ] Список узлов + статусы
- [ ] Таблица маршрутов + add/remove
- [ ] Управление DNS
- [ ] Включение/выключение exit-node
- [ ] Генерация join-токена
- [ ] Базовая авторизация admin (token / basic)

## Dependencies

- **control-plane** — стабильные endpoints
- **shared/openapi** — для генерации клиента (желательно)

## Related Docs

- [docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md)
- [docs/API.md](../docs/API.md)
- [Roadmap Stage 4](../docs/ROADMAP.md)
