# Архитектура WireMesh

## Высокоуровневая схема

```
┌─────────────────┐       ┌──────────────────────┐       ┌─────────────────┐
│   LLM Agents    │◄─────►│   MCP Server         │◄─────►│                 │
│ (Grok, Claude,  │       │   (JSON-RPC 2.0)     │       │  Control Plane  │
│  local agents)  │       └──────────────────────┘       │  (REST + gRPC)  │
└─────────────────┘                                      │                 │
                                                         │  - Nodes        │
┌─────────────────┐       ┌──────────────────────┐       │  - Networks     │
│   Web UI (SPA)  │◄─────►│   HTTP/REST API      │◄─────►│  - Routes       │
└─────────────────┘       └──────────────────────┘       │  - DNS          │
                                                         │  - Exit Nodes   │
                                                         │  - Policies     │
                                                         └────────┬────────┘
                                                                  │
                                                                  │ config / heartbeat
                                                                  ▼
┌─────────────────┐       ┌──────────────────────┐       ┌─────────────────┐
│   Agent Node A  │◄─────►│   WireGuard Mesh     │◄─────►│   Agent Node B  │
│   (wg0 + routes)│       │   (data plane)       │       │   (wg0 + routes)│
└─────────────────┘       └──────────────────────┘       └─────────────────┘
```

## Основные компоненты

### 1. Control Plane (Network Coordination Server)

- Веб-сервер и API, развёрнутый на собственном домене.
- Хранит информацию о узлах, виртуальных сетях, маршрутах, DNS-настройках, exit-узлах.
- Выполняет выдачу конфигурации агентам (WireGuard ключи, список пиров, маршруты, DNS).
- Экспонирует REST/gRPC API и MCP-сервер.
- Рекомендуемый стек MVP: Go (или FastAPI/Node.js) + PostgreSQL (или SQLite для простоты).

### 2. Data Plane (WireGuard-оверлей)

- У каждого узла запускается WireGuard peer с уникальной парой ключей.
- Control plane распределяет публичные ключи и адреса, формируя mesh-топологию.
- Режимы:
  - point-to-point / full mesh
  - subnet router (объявление маршрутов к локальным подсетям)
  - exit node (маршрутизация 0.0.0.0/0 и ::/0 через узел + NAT)

### 3. Agent (Node Client)

- Пакет для Arch Linux: systemd-unit, конфиг, бинарь.
- Функции:
  - Регистрация узла на control plane (по токену/ключу)
  - Генерация и отправка WireGuard-ключей
  - Получение и применение конфигурации (wg-quick / netlink)
  - Heartbeat / статус
  - Обработка обновлений конфигурации (pull или WebSocket/push)

### 4. Web UI (Management Console)

- SPA поверх Control Plane API (React / Vue / Svelte).
- Возможности:
  - Список узлов и статусы
  - Управление маршрутами, DNS, exit-узлами
  - Политики (ACL: кто видит кого)
  - Генерация join-токенов

### 5. MCP-Server Adapter

- Реализует Model Context Protocol поверх API Control Plane.
- Позволяет LLM-клиентам выполнять те же операции, что и администратор в Web UI.

## Поток «install-and-connect» (один шаг)

1. Администратор разворачивает Control Plane на своём домене и настраивает базовую конфигурацию.
2. Пользователь на Arch Linux выполняет:
   ```bash
   pacman -S wiremesh-agent
   wiremesh-agent join https://net.example.com --token <TOKEN>
   ```
3. Agent:
   - Генерирует WireGuard keypair
   - Отправляет публичный ключ + системную информацию на Control Plane
   - Получает конфигурацию (IP, peers, DNS, routes)
   - Применяет: создаёт интерфейс `wg0`, прописывает маршруты, DNS (systemd-resolved / resolv.conf)
4. Узел появляется в Web UI как online. Админ может назначить роли (subnet router / exit node).

## Управление маршрутами, DNS и exit-узлами

### Маршруты

Control Plane формирует WireGuard-конфиг для каждого узла с учётом объявленных маршрутов (`AllowedIPs`).

При изменении маршрута — пересборка конфигов и push/pull обновления агентам.

### DNS

Для каждой виртуальной сети — список DNS-серверов (внутренних или внешних).

Agent при подключении может переопределять DNS (split DNS / full override).

### Exit Nodes

- Админ помечает узел как exit node.
- Другие узлы выбирают его как шлюз.
- На exit-узле: `0.0.0.0/0` в AllowedIPs + MASQUERADE (nftables/iptables).

## Безопасность (базовый уровень MVP)

- Аутентификация агентов по join-токену (одноразовому или долгоживущему).
- Все данные plane — WireGuard (Curve25519 + ChaCha20-Poly1305).
- Control plane — TLS обязателен.
- В будущем: OIDC/SAML, ACL на уровне портов/протоколов, device posture.
