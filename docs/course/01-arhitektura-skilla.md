# 01 — Архитектура скилла

Один доменный скилл = четыре слоя. Разберём на `task-scheduler` (структура у всех LangGraph-скиллов одинаковая).

## Слои

```
1. SKILL.md            .claude/skills/task-scheduler/SKILL.md
   (инструкция агенту: когда активироваться, какой workflow, какие MCP-tools звать)
        │
2. MCP-обёртка         task-scheduler/mcp/mcp-server.mjs
   (Streamable-HTTP MCP-сервер; экспонирует tools; проксирует в HTTP сервиса)
        │
3. Бэкенд-сервис       task-scheduler/src/server.ts (Fastify)
   ├─ src/graph/       LangGraph: узлы пайплайна + состояние
   ├─ src/clients/     клиенты к внешним сервисам (gogcli MCP)
   ├─ src/scheduler/   чистая доменная логика (слоты, распределение, таймзоны)
   ├─ src/llm.ts       обёртка над ChatOpenAI (Kimi)
   └─ src/config.ts    разбор env
        │
4. Регистрация         config/openclaw.json → mcp.servers.task_scheduler
   (+ docker-compose, + AGENTS.md/TOOLS.md как документация)
```

## Раскладка файлов

```
task-scheduler/
├── package.json            # deps (см. урок 00), scripts: dev/build/start/typecheck
├── tsconfig.json
├── Dockerfile              # сборка сервиса
├── docker-compose.yml      # два сервиса: task-scheduler + task-scheduler-mcp
├── .env.example            # шаблон конфига (без секретов)
├── README.md
├── src/
│   ├── server.ts           # Fastify: POST /plan, GET /plan/:jobId, POST /commit, /healthz
│   ├── config.ts           # env → типизированный config
│   ├── llm.ts              # ChatOpenAI(Kimi): chatText / chatJSON
│   ├── logger.ts           # pino
│   ├── plan-jobs.ts        # async job-стор (см. урок 06)
│   ├── graph/
│   │   ├── index.ts        # StateGraph: сборка графа + runPlan()
│   │   ├── state.ts        # Annotation-схема состояния
│   │   ├── prompts.ts      # системные промпты для LLM-узлов
│   │   └── nodes/          # узлы: intake, decompose, gather-busy, gather-holidays, schedule, render
│   ├── clients/
│   │   └── gogcli-mcp-client.ts   # MCP-клиент к gogcli (Calendar read/write)
│   └── scheduler/          # чистая логика без I/O
│       ├── slots.ts        # генерация свободных слотов в окне дня
│       ├── distribute.ts   # раскладка подзадач по слотам
│       └── time-zone.ts    # работа с таймзонами (IANA)
└── mcp/
    ├── package.json
    ├── Dockerfile
    └── mcp-server.mjs      # MCP Streamable-HTTP обёртка
```

Принцип, по которому файлы так нарезаны (см. правила проекта `.claude/rules/development-rules.md`):
**файлы < 200 строк, kebab-case, имя говорит о назначении** — чтобы LLM по имени файла понимал,
что внутри, без чтения.

## Поток одного вызова

На примере «распланируй задачу X»:

```
агент (OpenClaw)
  → MCP tool  plan_task(...)                      [mcp/mcp-server.mjs]
     → POST http://task-scheduler:7801/plan       [src/server.ts]
        → startPlanJob()                           [src/plan-jobs.ts]
           → runPlan() = compiled LangGraph.invoke [src/graph/index.ts]
              intake → decompose ─┐
                       gather-busy ┼→ schedule → render
                       gather-holidays┘
                 ├─ LLM-узлы (intake/decompose)  → Kimi   [src/llm.ts]
                 ├─ gather-узлы                   → gogcli [src/clients/]
                 └─ schedule                      → чистая логика [src/scheduler/]
     ← возвращает jobId
  → MCP tool plan_result(jobId)  (поллинг)        → готовый план
  → MCP tool commit_plan(...)    (после подтверждения) → создаёт события в Calendar
```

Три типа узлов в графе, которые стоит различать:
- **LLM-узлы** (`intake`, `decompose`) — зовут Kimi, медленные и недетерминированные.
- **I/O-узлы** (`gather-busy`, `gather-holidays`) — ходят во внешние сервисы, обёрнуты в таймаут.
- **Чистые узлы** (`schedule`, `render`) — детерминированная логика, без сети; их легко юнит-тестировать.

Это разделение — главный приём: дорогую недетерминированную часть (LLM) держим в минимуме,
а раскладку по слотам делаем чистой функцией, которую можно проверить без LLM и без сети.

## Разделение ответственности по слоям

| Слой | Отвечает за | НЕ отвечает за |
|------|-------------|----------------|
| SKILL.md | UX-сценарий: вопросы пользователю, порядок вызовов, защита | бизнес-логику |
| MCP-обёртка | контракт инструментов, проксирование, формат ответа | вычисления |
| Сервис (graph) | оркестрацию шагов, вызовы LLM и I/O | как агент это покажет |
| scheduler/* | доменную логику (слоты, время) | внешний мир |

Дальше — [урок 02](02-langgraph-langchain.md): как именно устроен граф и вызов LLM.
