# Развёртывание Control Plane

## Требования

- Домен с валидным TLS-сертификатом (Let's Encrypt / свой CA)
- Сервер с публичным IP (или доступный через reverse proxy)
- PostgreSQL (рекомендуется) или SQLite для MVP
- Docker / Docker Compose (предпочтительно) или бинарь

## Минимальный docker-compose (черновик)

```yaml
version: "3.8"

services:
  control-plane:
    image: ghcr.io/unhexx/wiremesh-control:latest   # будет
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgres://wiremesh:secret@db:5432/wiremesh
      - JWT_SECRET=change-me
      - BASE_URL=https://net.example.com
    depends_on:
      - db
    volumes:
      - ./data:/data

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: wiremesh
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: wiremesh
    volumes:
      - pgdata:/var/lib/postgresql/data

  # Опционально: reverse proxy + certbot
  # caddy / traefik / nginx

volumes:
  pgdata:
```

## Переменные окружения (план)

| Переменная          | Описание                              |
|---------------------|---------------------------------------|
| `DATABASE_URL`      | Строка подключения к БД               |
| `BASE_URL`          | Публичный URL control plane           |
| `JWT_SECRET`        | Секрет для токенов                    |
| `LISTEN_ADDR`       | `:8080`                               |
| `DEFAULT_NETWORK_CIDR` | `100.64.0.0/10`                   |

## Первый запуск

1. Поднять control plane.
2. Создать первую Network через API или bootstrap CLI.
3. Сгенерировать join-токен.
4. Подключить первый агент.
5. Открыть Web UI (будет на `/` или отдельном порту).

## Безопасность

- Обязательный TLS (терминировать на reverse proxy или внутри).
- Join-токены с ограниченным сроком и числом использований.
- Rate limiting на register / heartbeat.
- Регулярные бэкапы БД.
