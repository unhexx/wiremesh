# Дорожная карта WireMesh — до первого релиза (MVP v0.1)

**Цель первого релиза (MVP):**  
Два (и более) Arch Linux-хоста успешно присоединяются к самохостируемому Control Plane одной командой, образуют WireGuard mesh, могут пинговать друг друга, поддерживают subnet-router и один exit-node. Управление доступно через минимальный Web UI и базовый MCP-сервер. Есть PKGBUILD/AUR-пакет агента и docker-compose для Control Plane.

**Критерии выхода в v0.1:**
- e2e-сценарий «два узла + subnet + exit» проходит стабильно
- `pacman -S wiremesh-agent && wiremesh-agent join ...` работает
- Control Plane поднимается одной командой (`docker compose up`)
- Есть OpenAPI, базовая документация и changelog
- MCP tools для nodes/routes/dns/exit работают

---

## Оркестраторы (специализированные агенты)

Используется иерархическая модель **Supervisor + Specialists** с параллельными потоками.

| Оркестратор              | Ответственность                                      | Основные навыки / инструменты          |
|--------------------------|------------------------------------------------------|----------------------------------------|
| **SupervisorOrchestrator** | Общая координация, чекпоинты, human gates, prioritization, рефлексия | deterministic-workflow, multi-agent, reflective |
| **SpecOrchestrator**     | Спецификации, OpenAPI, data models, протоколы, docs  | docs, OpenAPI, architecture            |
| **BackendOrchestrator**  | Control Plane (Go), API, storage, config generation, auth | Go, chi/echo, SQLite/Postgres, WireGuard logic |
| **ClientOrchestrator**   | Agent (Go), WireGuard integration, systemd, PKGBUILD | Go, netlink/wg, systemd, packaging     |
| **FrontendOrchestrator** | Web UI (SPA), формы управления маршрутами/DNS/exit   | Svelte/React/Vue, Tailwind, API client |
| **MCPOrchestrator**      | MCP-сервер (resources + tools), адаптер к API        | MCP SDK, JSON-RPC                      |
| **QAOrchestrator**       | e2e-тесты, демо-сценарии, smoke, документация тестов | test scripts, two VMs/containers       |
| **ReleaseOrchestrator**  | Версионирование, docker images, AUR, changelog, tags | gh, docker, packaging, release notes   |

**Параллельные потоки (после Stage 1):**
- **[BACKEND]** + **[CLIENT]** (могут идти почти параллельно)
- **[FRONTEND]** + **[MCP]** (после стабилизации API)
- **[QA]** подключается на каждом крупном чекпоинте
- **[RELEASE]** — финальный

Все задачи оформляются как INVEST (Independent, Negotiable, Valuable, Estimable, Small, Testable) с чёткими acceptance criteria и Evidence requirements.

---

## Stage 0 — Foundation (уже сделано)

- [x] Репозиторий `unhexx/wiremesh`
- [x] README, LICENSE, .gitignore, CONTRIBUTING
- [x] Базовая документация (ARCHITECTURE, SPEC, API, MCP, AGENT, DEPLOYMENT, PROJECT-BRIEF)
- [x] Issues #1–#3

**Чекпоинт 0:** Документация и трекинг готовы.

---

## Stage 1 — Spec Freeze (SpecOrchestrator + Supervisor)

**Цель:** Заморозить контракты до начала кода.

### Декомпозиция (делегируется SpecOrchestrator)

| ID              | Задача                                      | Acceptance Criteria                                      | Evidence                          |
|-----------------|---------------------------------------------|----------------------------------------------------------|-----------------------------------|
| SPEC-001        | OpenAPI 3.0 спецификация (`openapi.yaml`)   | Полное покрытие endpoints из docs/API.md, примеры запросов/ответов | Файл в репо + валидация swagger   |
| SPEC-002        | Точный JSON Schema для `/nodes/{id}/config` | Схема + примеры (minimal, with routes, with exit)        | `docs/schemas/config.schema.json` |
| SPEC-003        | Протокол heartbeat + update (ETag/checksum) | Документирован pull-модель + опциональный WS             | Обновлённый SPEC.md + API.md      |
| SPEC-004        | Модель данных (ER-диаграмма + Go structs)   | Entity-relationship + рекомендуемые Go-типы              | `docs/models.md` или в SPEC.md    |
| SPEC-005        | Join-token lifecycle                        | Создание, TTL, max_uses, revoke                          | Раздел в API.md                   |

**Human Gate после Stage 1:** Review OpenAPI + schema (Supervisor + пользователь).

**Чекпоинт 1:** Контракты зафиксированы. Можно начинать Backend и Client параллельно.

---

## Stage 2 — Control Plane MVP (BackendOrchestrator)

**Цель:** Работающий сервер, который выдаёт валидный WireGuard-конфиг.

### Декомпозиция

| ID              | Задача                                      | Acceptance Criteria                                      | Параллельность |
|-----------------|---------------------------------------------|----------------------------------------------------------|----------------|
| BE-001          | Каркас Go-проекта + chi/echo + конфиг       | `go run` поднимает HTTP на :8080                         | —              |
| BE-002          | SQLite storage + миграции (nodes, networks, tokens) | CRUD для Node/Network/JoinToken                         | после BE-001   |
| BE-003          | `POST /nodes/register`                      | Принимает token+pubkey, выделяет IP, возвращает config   | после BE-002   |
| BE-004          | Генерация peer list (full mesh)             | Config содержит всех остальных peers с правильными AllowedIPs | после BE-003 |
| BE-005          | `GET /nodes/{id}/config` + ETag             | Возвращает актуальный конфиг, поддержка If-None-Match    | после BE-004   |
| BE-006          | `POST /nodes/{id}/heartbeat`                | Обновляет last_seen + endpoints                          | параллельно с BE-005 |
| BE-007          | Join-token API (create / list / revoke)     | Admin может выпускать токены                             | после BE-002   |
| BE-008          | Минимальный admin CLI или простой HTML      | Можно создать сеть и токен без curl                      | после BE-007   |
| BE-009          | docker-compose + Dockerfile                 | `docker compose up` поднимает control plane + volume     | параллельно    |

**Чекпоинт 2:** Control Plane выдаёт рабочий конфиг. Можно тестировать с ручным `wg-quick`.

---

## Stage 3 — Agent MVP (ClientOrchestrator)

**Цель:** Агент, который одной командой поднимает `wg0` и становится online.

### Декомпозиция

| ID              | Задача                                      | Acceptance Criteria                                      |
|-----------------|---------------------------------------------|----------------------------------------------------------|
| CL-001          | Каркас Go-агента + CLI (cobra/urfave)       | `wiremesh-agent version` / `join --help`                 |
| CL-002          | Генерация WireGuard keypair                 | Ключи сохраняются в StateDirectory                       |
| CL-003          | Join flow (register → получить config)      | Успешный register + сохранение node_id + config          |
| CL-004          | Применение конфига (wg-quick или netlink)   | Интерфейс `wg0` up, IP назначен, peers видны             |
| CL-005          | systemd unit + install script               | `systemctl enable --now wiremesh-agent` работает         |
| CL-006          | Heartbeat loop + config pull                | При изменении на сервере агент обновляет peers           |
| CL-007          | PKGBUILD + makepkg                          | Собирается пакет, ставится через pacman                  |
| CL-008          | `status` / `leave` команды                  | Чистый вывод состояния, корректное удаление интерфейса   |

**Параллельно со Stage 2** (после SPEC-002).

**Чекпоинт 3:** `wiremesh-agent join ...` → узел online в Control Plane, `ping` между двумя агентами проходит.

---

## Stage 4 — Routes, DNS, Exit Nodes (Backend + Client + Frontend)

**Цель:** Полноценное управление маршрутизацией.

### Декомпозиция

**BackendOrchestrator:**
- BE-010: Модели Route + ExitNode + CRUD API
- BE-011: Логика пересборки AllowedIPs при изменении маршрутов
- BE-012: DNS config API + прокидывание в agent config
- BE-013: Exit-node flag + документация по MASQUERADE (nftables)

**ClientOrchestrator:**
- CL-009: Применение дополнительных routes (ip route)
- CL-010: DNS override (resolvectl / systemd-resolved)
- CL-011: Exit-node mode (IP forwarding + MASQUERADE setup/teardown)

**FrontendOrchestrator:**
- FE-001: Каркас SPA (SvelteKit / Vue / React — выбрать один)
- FE-002: Список узлов + статусы
- FE-003: Таблица маршрутов + add/remove
- FE-004: Управление DNS и exit-nodes
- FE-005: Генерация join-токенов в UI

**Чекпоинт 4:** Можно через UI добавить subnet route и exit-node, агенты применяют изменения, интернет-трафик идёт через exit.

---

## Stage 5 — MCP Server (MCPOrchestrator)

**Цель:** LLM-агенты могут управлять сетью.

### Декомпозиция

| ID              | Задача                                      | Acceptance Criteria                                      |
|-----------------|---------------------------------------------|----------------------------------------------------------|
| MCP-001         | Каркас MCP-сервера (Go или Python SDK)      | Поднимается, проходит capability negotiation             |
| MCP-002         | Resources: nodes, routes, dns, exit-nodes   | `list` / `read` возвращают актуальные данные             |
| MCP-003         | Tools: nodes.set_role, routes.add/remove, dns.set, exit.enable/disable | Вызовы изменяют состояние на Control Plane |
| MCP-004         | Примеры: CLI-клиент + prompt для Claude/Grok| Документированный working example                        |
| MCP-005         | Интеграция в docker-compose                 | MCP доступен вместе с Control Plane                      |

**Чекпоинт 5:** MCP-клиент может добавить маршрут и включить exit-node.

---

## Stage 6 — QA, Packaging, First Release (QA + Release + Supervisor)

### Декомпозиция

**QAOrchestrator:**
- QA-001: e2e-скрипт «два контейнера/VM + join + ping»
- QA-002: Сценарий subnet-router (доступ к LAN за одним узлом)
- QA-003: Сценарий exit-node (curl ifconfig.me через exit)
- QA-004: Регрессия после изменений конфигов
- QA-005: Документация «How to demo»

**ReleaseOrchestrator:**
- REL-001: Semantic versioning + CHANGELOG.md
- REL-002: Docker images (control-plane, опционально agent)
- REL-003: AUR package (wiremesh-agent) + PKGBUILD в репо
- REL-004: GitHub Release v0.1.0 + binaries (если нужно)
- REL-005: Финальный README с quickstart для self-hosting

**Supervisor:**
- Финальный human gate + review
- Рефлексия по всему MVP (что улучшить в v0.2)

**Чекпоинт 6 / Release Gate:** Все критерии MVP выполнены → тег `v0.1.0`.

---

## Граф зависимостей (упрощённо)

```
Stage 1 (Spec)
    ↓
┌───┴───┐
│       │
Stage 2 Stage 3     ← параллельно
(Backend)(Client)
    ↓       ↓
    └───┬───┘
        ↓
    Stage 4 (Routes/DNS/Exit + Frontend)
        ↓
    Stage 5 (MCP)
        ↓
    Stage 6 (QA + Release)
```

---

## Принципы исполнения

1. **INVEST + narrow slices** — каждая задача должна быть завершаема за 1–3 сессии агента.
2. **Evidence first** — каждый чекпоинт сопровождается коммитами, скриншотами/логами, smoke-маркерами.
3. **Human gates** на границах Stage 1, 3, 4, 6.
4. **Параллелизм** Backend ↔ Client после фиксации схем.
5. **Рефлексия** после каждого Stage (reflective-improvement) — lessons → обновление планов.
6. **Единый источник правды** — этот ROADMAP.md + GitHub Issues + (позже) `.agent/` артефакты, если проект будет вестись в eeagent-стиле.

---

## Следующие конкретные действия (прямо сейчас)

1. **SpecOrchestrator** начинает SPEC-001 (OpenAPI) и SPEC-002 (config schema).
2. Создать GitHub Project / Milestone «MVP v0.1».
3. После Stage 1 — запустить параллельно BackendOrchestrator (BE-001) и ClientOrchestrator (CL-001).

Готов делегировать первую волну задач оркестраторам по команде.
