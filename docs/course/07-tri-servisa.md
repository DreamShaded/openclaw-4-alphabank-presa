# 07 — Разбор трёх сервисов

Три реальных скилла. Первые два — близнецы (LangGraph), третий — другой паттерн (RAG-пайплайн).

---

## task-scheduler — планировщик задач

**Что делает:** из текста задачи + дедлайна получает подзадачи с оценкой времени и раскладывает
их по свободным слотам Google Calendar.

**Граф** (`src/graph/index.ts`):

```
intake → decompose ─────────┐
         gather-busy ────────┼→ schedule → render → END
         gather-holidays ────┘
```

- `intake` (LLM) — классификация: заголовок, категория (`work`/`personal`), дедлайн, сложность.
- `decompose` (LLM) — подзадачи + `estimateMinutes` (+25% буфера автоматически).
- `gather-busy` / `gather-holidays` (IO, через gogcli MCP) — занятость и праздники РФ; обёрнуты в
  `withTimeout(..., GATHER_TIMEOUT_MS)`, fail-soft → `gaps`.
- `schedule` (**чистая логика**, `src/scheduler/`) — генерация свободных слотов в окне дня и
  раскладка блоков. LLM не нужен → детерминированно и тестируемо.
- `render` — markdown-таблица плана.

**Доменное ядро (`src/scheduler/`):**
- `slots.ts` — `generateFreeSlots`: по дням режет окно `[windowStart, windowEnd]` на слоты
  `SLOT_MINUTES`, выкидывает выходные/праздники/занятость. Ключ: `cursor + slotMs <= dayEnd` —
  ни один слот не выходит за конец окна (поэтому «закончить к 23:00» работает само).
- `distribute.ts` — `findContiguousBlock`: ищет смежные слоты под длительность подзадачи
  (режим `blocks`) либо размазывает (`even`).
- `time-zone.ts` — конвертация wall-time ↔ UTC в IANA-таймзоне (Europe/Moscow).

**Запись в календарь:** `commit_plan` → `POST /commit` → `calendarCreate` (gogcli).
`gog calendar create` берёт `<calendarId>` **позиционно**, время — `--from/--to` (RFC3339),
таймзону — `--start-timezone/--end-timezone`; ответ приходит как `{ event: { id, htmlLink, ... } }`.

**Особенность:** `plan_task` **асинхронный** (job+poll, см. [урок 06](06-async-i-tajmauty.md)).

---

## meeting-brief — брифинг к встрече

**Что делает:** перед встречей собирает структурированный бриф из 4 источников.

**Граф** (`src/graph/index.ts`):

```
classify ─┬→ gather-rag ──────┐
          ├→ gather-gmail ─────┤
          ├→ gather-calendar ──┼→ reconcile → render → END
          └→ gather-drive ─────┘
```

- `classify` (LLM, chatJSON) — разбирает запрос: контрагент, проект, тема, диапазон дат,
  поисковые запросы, email'ы.
- 4× `gather` (**параллельно**):
  - `gather-rag` → HTTP к сервису `rag`: `ragAsk({mode:"deep",maxHops:3})` + до 4× `ragSearch({rerank:true})`.
  - `gather-gmail` / `gather-calendar` / `gather-drive` → gogcli MCP (`gmail search`,
    `calendar events`, `drive search`). Calendar расширяет окно на **+14 дней вперёд** (грядущие встречи).
  - Все обёрнуты в `withTimeout(..., GATHER_TIMEOUT_MS)`, дедуп по id, **fail-soft** → `gaps`.
- `reconcile` (LLM, chatJSON) — компактит источники (клип текстов, слайс хитов) и раскладывает
  факты по бакетам: `{ context, discussed, ours, theirs, open, blockers }`.
- `render` (LLM, chatText) — markdown-бриф.

**Клиенты:** `src/clients/rag-client.ts` (HTTP к rag: `/search`, `/ask`, `/healthz`) и
`src/clients/gogcli-mcp-client.ts` (MCP к gogcli: gmail/calendar/drive).

**Чему учит:** как переиспользовать **другой скилл** (rag) как зависимость, и как
fan-out на 4 источника со сбором ошибок в `gaps` (бриф строится даже если часть источников отвалилась).

**MCP-tools:** `meeting_brief(...)`, `meeting_brief_healthz()`. Здесь вызов синхронный
(бриф укладывается в окно), но при росте источников логика та же, что у планировщика → async.

---

## rag — поиск и синтез по заметкам

Другой паттерн: не граф-оркестрация одного запроса, а **постоянный пайплайн индексации** +
**гибридный поиск** + **многошаговый синтез (IRCoT)**.

**Индексация** (реактивная):
- `src/sources.ts` — список каталогов заметок (`RAG_NOTE_DIRS`), динамически меняется через `/sources`.
- `src/watcher.ts` — `chokidar` следит за `.md`; на `add/change/unlink` → переиндексация файла;
  плюс периодический `reconcile` (fallback для NFS/rsync/git pull).
- `src/chunker.ts` — `gray-matter` парсит frontmatter, режет по заголовкам H1–H2, чанк ≤
  `CHUNK_MAX_CHARS`. ID чанка = SHA-1(path+pos+heading+text) → детерминирован (safe upsert).
- `src/indexer.ts` — сверяет mtime (`data/index-state.json`), эмбеддит, пишет в Qdrant, кормит граф знаний.

**Эмбеддинги** (`src/embeddings.ts`): отдельный сервис `embeddings` (BGE-M3, локально, бесплатно),
**hybrid** — dense (1024D, cosine) + sparse (BM25-like). Плюс `rerank` (cross-encoder) **снаружи**
Qdrant.

**Векторная БД** (`src/qdrant.ts`, `@qdrant/js-client-rest`): коллекция `notes` с dense + sparse
векторами; поиск — dual prefetch + RRF-фьюжн; фильтр по frontmatter (project/type/tag).

**Граф знаний** (`src/graph/`, `better-sqlite3`): извлечение сущностей/связей из чанков +
personalized PageRank для multi-hop. Включается `GRAPH_ENABLED`.

**Синтез** (`src/ircot.ts`): IRCoT — decompose вопроса → итеративные hop'ы (retrieve → reason) →
synthesize. Модели: дешёвая для extraction/JSON (`moonshot-v1-8k`), дорогая для финального
синтеза (`kimi-k2.6`).

**HTTP API** (`src/server.ts`, Fastify): `/search`, `/ask`, `/ask/stream` (SSE), `/sources`,
`/graph/stats`, `/metrics` (Prometheus), `/reindex`, `/evals/run`, и **веб-GUI** `/ui/`
(`@fastify/static`) — статус, граф, источники, ручной реиндекс.

**MCP-обёртка** (`mcp/mcp-server.mjs`): tools `rag_search` (гибридный поиск + rerank) и `rag_ask`
(IRCoT). Прокси в HTTP с `RAG_FETCH_TIMEOUT_MS=120000` (важно для `/ask`).

**Эвалы** (`src/evals.ts`): JSONL-корпус вопросов, метрики `hitAtK`, `keywordCoverage`,
`answerSimilarity`; baseline/compare/AB — регрессионные проверки качества поиска.

**Чему учит:** скилл может быть не «один запрос → ответ», а сервисом с фоновым состоянием
(индекс), своим GUI и метриками/эвалами. MCP-обёртка и регистрация — те же.

---

## Сравнение

| | task-scheduler | meeting-brief | rag |
|---|---|---|---|
| Оркестрация | LangGraph | LangGraph | свой пайплайн (watcher+IRCoT) |
| LLM | Kimi (intake/decompose) | Kimi (classify/reconcile/render) | Kimi (synth) + moonshot-v1-8k (extract) |
| Хранилище | — (stateless + job-стор) | — | Qdrant + SQLite (граф) |
| Внешние | gogcli | rag + gogcli | embeddings + qdrant |
| Async tool | да (job+poll) | нет | нет (но `/ask` долгий) |
| GUI | нет | нет | да (`/ui/`) |
| Общее | Fastify, MCP-обёртка, SKILL.md, регистрация, `chatJSON`, fail-soft, таймауты |

Дальше — [урок 08](08-novyj-skill.md): собираем свой скилл по шагам.
