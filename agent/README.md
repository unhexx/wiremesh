# agent

**Ownership:** ClientOrchestrator  
**Status:** Scaffold (Stage 3)

## Purpose

Клиентский агент узла WireMesh.

Одна команда join → генерация ключей → регистрация на Control Plane → поднятие WireGuard-интерфейса (`wg0`) → heartbeat + pull обновлений конфигурации.

Целевая платформа MVP: **Arch Linux** (pacman / AUR).

## Boundaries

**Владеет:**
- CLI (`wiremesh-agent`)
- Генерация и хранение WireGuard keypair
- Применение конфигурации (wg-quick / netlink)
- systemd unit
- Heartbeat loop + config pull/update
- Exit-node side effects (IP forwarding + MASQUERADE)
- Packaging (PKGBUILD)

**Не владеет:**
- Логику распределения IP и peer lists (это Control Plane)
- UI и политики

## Tech Stack (рекомендация MVP)

- Language: **Go 1.22+** (один статический бинарь)
- CLI: cobra или urfave/cli
- WireGuard: `golang.zx2c4.com/wireguard` + `wgctrl` / вызовы `wg` + `wg-quick`
- systemd unit + StateDirectory / ConfigurationDirectory

## Key Interfaces

| Provides | Consumes |
|----------|----------|
| `wg0` interface + routes + DNS | Control Plane REST API |
| Heartbeat | `shared/schemas/` (config format) |
| systemd service | — |

## Local Development

```bash
go run ./agent/cmd/wiremesh-agent join https://net.example.com --token <TOKEN>
go run ./agent/cmd/wiremesh-agent status
```

Требует root / capabilities для поднятия интерфейса и маршрутов.

## Packaging (Arch)

См. `agent/packaging/arch/` (будет добавлено):
- PKGBUILD
- systemd unit
- install script

Target install:
```bash
pacman -S wiremesh-agent
wiremesh-agent join <url> --token <token>
```

## MVP Scope (Stage 3)

- [ ] CLI: join / status / leave
- [ ] Key generation + persistent state
- [ ] Register + apply config (wg0 up)
- [ ] systemd unit
- [ ] Heartbeat + config refresh (pull + ETag)
- [ ] PKGBUILD
- [ ] Basic leave / cleanup

## Dependencies

- **control-plane** — должен уметь register + отдавать config
- **shared/schemas** — формат config JSON

## Related Docs

- [docs/AGENT.md](../docs/AGENT.md)
- [docs/SPEC.md](../docs/SPEC.md)
- [Roadmap Stage 3](../docs/ROADMAP.md)
