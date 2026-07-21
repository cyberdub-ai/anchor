# Anchor — stage deployment (cyberdub-ai)

Upstream: https://github.com/ZhFahim/anchor (offline-first, self-hostable note-taking app, AGPL-3.0).
Full upstream docs (features, OIDC, mobile app): [README.upstream.md](README.upstream.md).

## Что это и зачем нам

Из радара (reddit, v0.14.0): импорт заметок из Google Keep + offline-first заметки с
шифрованием-free self-host, rich-text, теги, вложения, шаринг заметок между пользователями,
OIDC-вход (Authelia/Authentik/Pocket ID — у нас уже есть Authelia/Authentik в стеке).
Кандидат на личный notes-сервис вместо облачных заметочников.

## Как поднято (STAGE, не прод)

- Образ: `ghcr.io/zhfahim/anchor:latest` (pre-built, тот же тег что и релиз v0.14.0 — см. `/api/health`)
- Compose: `docker-compose.stage.yml` (отдельный от upstream `docker-compose.yml`, который билдит из исходников)
- Порт: `127.0.0.1:3023 -> 3000` — **не публично**, без Caddy-блока, только localhost на .44
- Секреты: `/srv/secrets/anchor.env` (JWT_SECRET, PG_* — сгенерированы `openssl rand`), подключены через `env_file:`
- Данные: `/srv/data/anchor` (embedded Postgres + attachments внутри контейнера, `PG_HOST` не задан)
- Регистрация: `USER_SIGNUP=disabled` (stage — только ручное создание пользователя владельцем через admin panel/CLI)

Запуск:
```bash
cd ~/stack/anchor
docker compose -f docker-compose.stage.yml up -d
```

## Проверено так

```bash
docker ps --filter name=anchor-stage --format '{{.Names}}\t{{.Status}}'
# anchor-stage   Up ... (healthy)

curl -s http://127.0.0.1:3023/api/health
# {"status":"ok","app":"anchor","version":"0.14.0","timestamp":"..."}

curl -s -o /dev/null -w "%{http_code}\n" http://127.0.0.1:3023/
# 200
```

## Как откатить

```bash
cd ~/stack/anchor
docker compose -f docker-compose.stage.yml down     # останавливает и убирает контейнер+сеть
sudo rm -rf /srv/data/anchor                          # удаляет данные (заметки/вложения/embedded PG) — необратимо, только если точно не нужно
sudo rm /srv/secrets/anchor.env                       # удаляет секреты
# и убрать блок "anchor.env" из /srv/secrets/REGISTRY.md
```

Ничего в проде/Caddy не менялось — откат полностью локален для этого stage-контейнера.
