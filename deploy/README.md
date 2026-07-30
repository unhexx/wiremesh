# deploy

**Ownership:** ReleaseOrchestrator (+ Backend для Dockerfile)  
**Status:** Scaffold

## Purpose

Всё, что нужно для развёртывания WireMesh в self-hosted окружении.

- `docker-compose.yml` — быстрый подъём Control Plane (+ опционально MCP)
- Dockerfiles (или ссылки на них в компонентах)
- Примеры env-файлов
- В будущем: k8s manifests, Helm chart, systemd units для control-plane

## Structure (target)

```
deploy/
├── docker-compose.yml
├── docker-compose.dev.yml
├── .env.example
├── control-plane/           # если Dockerfile живёт здесь
├── k8s/                     # future
└── README.md
```

Рекомендация MVP: Dockerfile лежит рядом с кодом компонента (`control-plane/Dockerfile`), а `deploy/` содержит только compose и примеры конфигурации.

## Quick Start (target)

```bash
cp deploy/.env.example deploy/.env
# отредактировать BASE_URL, secrets и т.д.
docker compose -f deploy/docker-compose.yml up -d
```

После поднятия:
1. Создать Network + Join Token (через API или будущий admin CLI/UI)
2. На Arch-хостах: `wiremesh-agent join https://... --token ...`

## Related Docs

- [docs/DEPLOYMENT.md](../docs/DEPLOYMENT.md)
- [control-plane/README.md](../control-plane/README.md)
