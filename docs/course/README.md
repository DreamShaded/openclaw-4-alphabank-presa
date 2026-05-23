# Курс: создание скиллов для OpenClaw

Практический курс по тому, как в этом проекте устроены и пишутся **доменные скиллы** —
`task-scheduler`, `meeting-brief`, `rag` — и как сделать свой такой же.

Курс построен «по коду»: каждый тезис привязан к конкретным файлам и строкам в репозитории.

## Кому

Тем, кто умеет в TypeScript/Node и хочет научиться расширять OpenClaw собственными
инструментами: бэкенд-сервис на LangGraph/LangChain → MCP-обёртка → `SKILL.md` →
регистрация в OpenClaw.

## Как читать

Уроки пронумерованы — идти по порядку. Первые пять дают общую модель, шестой — главный
нюанс (таймауты OpenClaw), седьмой разбирает три реальных сервиса, восьмой — пошаговая
сборка своего скилла, девятый — собранные грабли.

## Оглавление

| # | Урок | О чём |
|---|------|-------|
| 00 | [Обзор](00-obzor.md) | Что такое «скилл» здесь, как всё связано, карта трёх сервисов |
| 01 | [Архитектура скилла](01-arhitektura-skilla.md) | Анатомия: сервис + MCP-обёртка + SKILL.md + регистрация |
| 02 | [LangGraph и LangChain](02-langgraph-langchain.md) | StateGraph, узлы/рёбра, fan-out/fan-in, ChatOpenAI (Kimi), chatJSON |
| 03 | [MCP-обёртка](03-mcp-obertka.md) | MCP SDK, Streamable-HTTP, `registerTool`, зачем отдельный слой |
| 04 | [SKILL.md](04-skill-md.md) | Формат, frontmatter, интерактивный workflow, защита от ошибок |
| 05 | [Регистрация](05-registraciya.md) | `openclaw mcp set`, `config/openclaw.json`, AGENTS.md / TOOLS.md |
| 06 | [Async и таймауты OpenClaw](06-async-i-tajmauty.md) | Почему планировщик «долго отвечает» — недоработка OpenClaw и обход |
| 07 | [Разбор трёх сервисов](07-tri-servisa.md) | task-scheduler / meeting-brief / rag по коду |
| 08 | [Создаём новый скилл](08-novyj-skill.md) | Пошаговый scaffold + чеклист |
| 09 | [Подводные камни](09-podvodnye-kamni.md) | Реальные грабли: модели Moonshot, gogcli, zod, латентность |

## Что в проекте уже есть

```
openclaw-4-alphabank/
├── docker-compose.yml      # сам OpenClaw (gateway, образ ghcr.io/openclaw/openclaw)
├── config/openclaw.json    # конфиг агента: models, mcp.servers, agents, skills
├── workspace/              # «дом» агента: AGENTS.md, TOOLS.md, MEMORY.md, skills/
├── .claude/skills/<name>/SKILL.md   # инструкции скиллов для агента
├── task-scheduler/         # сервис-скилл: планировщик задач (LangGraph + Kimi)
├── meeting-brief/          # сервис-скилл: брифинг к встрече (LangGraph + Kimi)
├── rag/                    # сервис-скилл: RAG по заметкам (Qdrant + IRCoT)
└── gogcli/                 # MCP-обёртка над `gog` (Google Workspace CLI)
```

Версия материала отражает состояние репозитория на момент написания. Команды OpenClaw —
для образа `ghcr.io/openclaw/openclaw` (OpenClaw 2026.5.x).
