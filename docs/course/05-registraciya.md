# 05 — Регистрация скилла в OpenClaw

Готовый скилл (сервис + обёртка + SKILL.md) нужно подключить к агенту. Регистрация состоит из:

1. **MCP-сервер** → запись в `config/openclaw.json → mcp.servers.<name>` (командой или вручную).
2. **SKILL.md** → положить туда, где агент его видит.
3. **AGENTS.md / TOOLS.md** → документация скилла для агента (необязательно, но полезно).
4. **Сеть** → обёртка и OpenClaw в одной docker-сети.
5. **Перезапуск** агента, чтобы он подхватил новый MCP-сервер.

## 1. Регистрация MCP-сервера — командой

OpenClaw умеет управлять MCP-конфигом из CLI. Подкоманды:

```bash
# внутри контейнера openclaw (или через docker compose exec openclaw ...)
node dist/index.js mcp list                 # список настроенных MCP-серверов
node dist/index.js mcp show [<name>]        # показать один сервер или весь mcp-конфиг
node dist/index.js mcp set <name> '<json>'  # добавить/изменить сервер из JSON-объекта
node dist/index.js mcp unset <name>         # удалить сервер
```

Регистрация нашего планировщика:

```bash
docker compose exec openclaw node dist/index.js mcp set task_scheduler \
  '{"url":"http://task-scheduler-mcp:8080/mcp","transport":"streamable-http"}'
```

Проверить:

```bash
docker compose exec openclaw node dist/index.js mcp show task_scheduler
```

## 1-bis. Или вручную в config/openclaw.json

`mcp set` просто пишет в `config/openclaw.json`. Можно отредактировать файл напрямую — что и
сделано в этом проекте:

```jsonc
{
  "mcp": {
    "servers": {
      "gogcli":         { "url": "http://gogcli-mcp:8080/mcp",          "transport": "streamable-http" },
      "rag":            { "url": "http://rag-mcp:8080/mcp",             "transport": "streamable-http" },
      "meeting_brief":  { "url": "http://meeting-brief-mcp:8080/mcp",   "transport": "streamable-http" },
      "task_scheduler": { "url": "http://task-scheduler-mcp:8080/mcp",  "transport": "streamable-http" }
    }
  }
}
```

Схема одного сервера (`McpServerSchema` в OpenClaw) принимает поля:
`command`, `args`, `env`, `cwd`, `url`, `transport` (`"sse"` | `"streamable-http"`), `headers`.
Для нашего случая (удалённый сервис в docker-сети) хватает `url` + `transport`.

> Имя сервера в конфиге (`task_scheduler`) — это «пространство имён» его инструментов. Агент видит
> их как `task_scheduler__plan_task`, `task_scheduler__plan_result` и т.д.

## 2. SKILL.md

Положить `SKILL.md` в каталог скиллов: `.claude/skills/<name>/SKILL.md`. Список и проверку
скиллов даёт CLI:

```bash
docker compose exec openclaw node dist/index.js skills list     # все доступные скиллы
docker compose exec openclaw node dist/index.js skills info <name>
docker compose exec openclaw node dist/index.js skills check    # что готово / чего не хватает
```

(`skills install/search/update` — для скиллов из ClawHub; наши доменные скиллы локальные.)

## 3. AGENTS.md и TOOLS.md

Это файлы «дома» агента в `workspace/`:

- **`workspace/AGENTS.md`** — операционные инструкции и протокол памяти агента (как он
  стартует сессию, что помнит, красные линии). Сюда дописывают усвоенные уроки про скилл:
  «при таймауте plan_task — опрашивай plan_result, не выдумывай план».
- **`workspace/TOOLS.md`** — инвентарь доступных инструментов: что есть в образе, чего нет
  намеренно, как добавить новый инструмент. Сюда добавляют запись про новый скилл-инструмент.

Эти файлы агент **читает** при работе (см. секцию «Session Startup» в AGENTS.md). Их можно
править руками или попросить агента дописать («запиши в TOOLS.md, что появился task_scheduler»).
Они не исполняемые — это контекст/память, дополняющая `SKILL.md` и `description` инструментов.

> Связка: `mcp.servers` даёт агенту *возможность* звать инструмент; `SKILL.md` — *сценарий*;
> `AGENTS.md`/`TOOLS.md` — *долговременная память и инвентарь*. Все три уровня дополняют друг друга.

## 4. Сеть (docker-compose)

Обёртка скилла должна быть в одной сети с контейнером OpenClaw. В `task-scheduler/docker-compose.yml`
подключение к внешней сети основного compose-проекта:

```yaml
networks:
  default:
  openclaw:
    external: true
    name: openclaw-4-alphabank-27052026_default   # имя сети основного проекта
```

Тогда OpenClaw достучится до `http://task-scheduler-mcp:8080/mcp` по имени контейнера. Имя внешней
сети узнаётся через `docker network ls | grep openclaw`.

## 5. Перезапуск

Изменения в `config/openclaw.json` подхватываются перезапуском агента:

```bash
docker compose restart openclaw      # или: docker compose up -d openclaw
```

После этого `mcp list` и `skills list` покажут новый скилл, а агент сможет звать его инструменты.

## Шпаргалка по проверке

```bash
# сервис жив?
curl -s http://127.0.0.1:7801/healthz | jq

# MCP-обёртка отвечает? (через тот же healthz-проброс)
docker compose logs --tail=3 task-scheduler-mcp        # "listening on ..."

# OpenClaw видит сервер и инструменты?
docker compose exec openclaw node dist/index.js mcp show task_scheduler
docker compose exec openclaw node dist/index.js skills list | grep task-scheduler
```

Дальше — [урок 06](06-async-i-tajmauty.md): главный нюанс, из-за которого «планировщик долго
отвечает», и почему это недоработка самого OpenClaw.
