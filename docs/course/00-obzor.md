# 00 — Обзор: что такое «скилл» в этом проекте

## Два разных значения слова «скилл»

В OpenClaw слово «skill» перегружено. Важно различать:

1. **SKILL.md-скилл** — markdown-файл с инструкциями для агента (в `.claude/skills/<name>/SKILL.md`
   или `workspace/skills/<name>/SKILL.md`). Это *подсказка агенту*: когда активироваться и какой
   сделать workflow. Сам по себе кода не исполняет.

2. **Доменный сервис-скилл** — то, что мы изучаем в курсе. Это полноценное приложение
   (`task-scheduler/`, `meeting-brief/`, `rag/`): бэкенд на Node + MCP-обёртка, которое
   реально что-то делает (ходит в LLM, в Google Calendar, в векторную базу). Агент дёргает его
   через **MCP-инструменты**, а `SKILL.md` объясняет агенту, как и когда это делать.

«Создать скилл» в этом проекте = собрать связку **сервис + MCP-обёртка + SKILL.md + регистрация**.

## Общая картина

```
┌─────────────────────────────────────────────────────────────┐
│ OpenClaw (агент, образ ghcr.io/openclaw/openclaw)            │
│  модель агента: moonshot/kimi-k2.6  (config/openclaw.json)    │
│  читает: SKILL.md, AGENTS.md, TOOLS.md                        │
│  вызывает MCP-инструменты ────────────────┐                  │
└───────────────────────────────────────────┼─────────────────┘
                                             │ MCP (Streamable-HTTP)
                  ┌──────────────────────────┼───────────────────────┐
                  ▼                           ▼                       ▼
        ┌──────────────────┐      ┌──────────────────┐    ┌──────────────────┐
        │ task-scheduler-  │      │ meeting-brief-   │    │ rag-mcp          │
        │ mcp (обёртка)    │      │ mcp (обёртка)    │    │ (обёртка)        │
        └────────┬─────────┘      └────────┬─────────┘    └────────┬─────────┘
                 │ HTTP                     │ HTTP                  │ HTTP
        ┌────────▼─────────┐      ┌─────────▼────────┐    ┌────────▼─────────┐
        │ task-scheduler   │      │ meeting-brief    │    │ rag              │
        │ Fastify+LangGraph│      │ Fastify+LangGraph│    │ Fastify+Qdrant   │
        └────────┬─────────┘      └────────┬─────────┘    └────────┬─────────┘
                 │                          │                       │
                 ▼                          ▼                       ▼
          Kimi (Moonshot)           Kimi + RAG + gogcli      Qdrant + embeddings
          + gogcli (Calendar)       (Gmail/Cal/Drive)        + Kimi (IRCoT)
```

Ключевые наблюдения:
- **Каждый скилл — два контейнера**: сам сервис (HTTP) + тонкая MCP-обёртка. Зачем разделять —
  в [уроке 03](03-mcp-obertka.md).
- **Связь агент↔скилл — только через MCP**. Агент не знает про внутренний HTTP сервиса.
- **Общая инфраструктура**: `gogcli` (Google), `rag` (база знаний), `embeddings`, `qdrant` —
  переиспользуются между скиллами через docker-сети.

## Карта трёх сервисов

| Сервис | Что делает | Стек ядра | Внешние зависимости |
|--------|------------|-----------|---------------------|
| **task-scheduler** | Разбивает задачу на подзадачи и раскладывает по свободным слотам календаря до дедлайна | LangGraph + LangChain (Kimi) | gogcli (Calendar) |
| **meeting-brief** | Собирает брифинг к встрече из 4 источников | LangGraph + LangChain (Kimi) | rag, gogcli (Gmail/Cal/Drive) |
| **rag** | Гибридный поиск + синтез ответов по личным заметкам | Qdrant + IRCoT (Kimi) | embeddings (BGE-M3), qdrant |

task-scheduler и meeting-brief — близнецы по архитектуре (LangGraph-граф). rag — другой зверь
(пайплайн индексации + векторный поиск), но MCP-обёртка и принцип регистрации те же.

## Общий технологический стек (для LangGraph-скиллов)

Из `task-scheduler/package.json` и `meeting-brief/package.json` — одинаковый набор:

- `fastify` + `@fastify/cors` — HTTP-сервер.
- `@langchain/langgraph` — граф состояния (оркестрация шагов).
- `@langchain/openai` + `@langchain/core` — вызов LLM (Kimi через OpenAI-совместимый API Moonshot).
- `@modelcontextprotocol/sdk` — и клиент (ходить в gogcli), и сервер (MCP-обёртка).
- `zod` — валидация входа/выхода.
- `pino` + `pino-pretty` — логи.
- `tsx` (dev) — запуск TS без сборки; `typescript` — typecheck.

rag вместо LangGraph использует `@qdrant/js-client-rest`, `better-sqlite3` (граф знаний),
`chokidar` (watcher файлов), `gray-matter` (frontmatter).

## Что дальше

В [уроке 01](01-arhitektura-skilla.md) разберём анатомию одного скилла по слоям.
