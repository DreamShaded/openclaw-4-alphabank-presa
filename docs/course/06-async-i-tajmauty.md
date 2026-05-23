# 06 — Async и таймауты OpenClaw (главный нюанс)

Этот урок отвечает на вопрос «почему планировщик долго/странно отвечает» и почему корень — в
**самом OpenClaw**, а не в нашем коде. Здесь же — как мы это обошли паттерном job + poll.

## Симптом

Запускаешь `plan_task` из GUI — и либо приходит ошибка `MCP error -32001: Request timed out`,
либо агент показывает план, **которого сервис не считал** (выдумал «из головы»). При этом сам
сервис в логах честно дорабатывает и пишет `plan done`.

## Почему так: три независимых потолка

### Потолок 1 — захардкоженный таймаут MCP-вызова в OpenClaw (~60с)

Это и есть **недоработка OpenClaw** (известный баг, issue #61786 — *«MCP tool calls timeout
prematurely (hardcoded 60s limit)»*). В MCP-клиенте OpenClaw используется дефолт
`DEFAULT_REQUEST_TIMEOUT_MSEC` из `@modelcontextprotocol/sdk`, и при вызове инструмента он
**не переопределяется**. Внутренний вызов выглядит так:

```js
// openclaw: pi-bundle-mcp-runtime
async callTool(serverName, toolName, input) {
  ...
  return await session.client.callTool({ name: toolName, arguments: input });
  //                                    ^ НИ опции таймаута, ни resetTimeoutOnProgress
}
```

Раз опция не передаётся — действует дефолт SDK (~60с). Любой инструмент, который думает дольше
минуты, обрывается с `-32001`.

**Важно: этот таймаут НЕ выводится в конфиг.** Проверено по схеме OpenClaw:

| Что пробовали | Что это на самом деле | Чинит -32001? |
|---|---|---|
| `mcp.servers.<name>.connectionTimeoutMs` (issue #57969) | таймаут **установки соединения** (`connectWithTimeout`), не вызова | ❌ |
| `agents.defaults.timeoutSeconds` | бюджет **хода агента** / CLI-бэкенда | ❌ |
| LLM-таймауты агента (issue #46049) | таймаут к **модели агента**, тоже захардкожен | ❌ |
| `requestTimeoutMs` MCP-клиента | **то, что нужно**, но в конфиг **не выведено** | ✅ только патчем кода |

Вывод: конфигом не лечится. Либо патчить бандл OpenClaw (хрупко, слетит при апдейте), либо
**не держать долгий вызов** — то есть сделать инструмент асинхронным.

### Потолок 2 — undici headersTimeout (~300с)

Если попытаться «подождать» в обёртке (держать длинный `fetch` к сервису), упрёшься во второй
потолок: у Node-fetch (undici) дефолтный `headersTimeout = 300с`. Сервис не шлёт заголовки
ответа, пока план не готов, → на 300с `fetch` падает с `TypeError: fetch failed`, даже если твой
`AbortController` выставлен на больше. А подключить `undici` в контейнер обёртки, чтобы поднять
этот лимит, нельзя — модуль там не резолвится.

### Потолок 3 — латентность самой модели

`kimi-k2.6` — reasoning-модель с **сильным разбросом** времени ответа. Замеры одного и того же
`/plan` в этом проекте: **113с → 353с → 504с**. То есть план может считаться 2–8 минут — в разы
больше обоих потолков выше.

## Цепочка отказа (как ломалось в GUI)

```
агент → plan_task ──(60с)──► -32001 (потолок 1)
агент: «тула не ответила» → ВЫДУМЫВАЕТ план сам        ← худшее: фальшивый результат
   ...тем временем сервис досчитывает за 504с и пишет plan done — но агент уже ушёл
```

Был и второй наблюдаемый сценарий: агент бросал поллинг через ~7 минут (упёрся в свой бюджет
хода) и **запускал новый** `plan_task` с новым jobId — старый, уже посчитанный, оставался
невостребованным.

## Решение: асинхронный инструмент (job + poll)

Раз ни один вызов не должен висеть дольше ~60с — делаем так, чтобы **каждый** MCP-вызов
возвращался мгновенно, а тяжёлую работу выносим в фон **на стороне сервиса**.

### Сервис: стартуем job, отдаём id

`task-scheduler/src/plan-jobs.ts` — in-memory стор задач:

```ts
const jobs = new Map();        // jobId → { status, result?, error?, startedAt, finishedAt? }
const TTL_MS = 30 * 60_000;

export function startPlanJob(input) {
  const jobId = randomUUID();
  jobs.set(jobId, { status: "running", startedAt: Date.now() });
  runPlan(input)                                   // НЕ await — фоном
    .then((result) => jobs.set(jobId, { status: "done", result, finishedAt: Date.now(), ... }))
    .catch((err)   => jobs.set(jobId, { status: "error", error: String(err), ... }));
  return jobId;                                    // мгновенно
}
export function getPlanJob(jobId) { return jobs.get(jobId); }
```

`src/server.ts` — два коротких эндпоинта вместо одного долгого:

```ts
app.post("/plan", async (req, reply) => {          // стартует job
  const jobId = startPlanJob(parsed.data);
  return { jobId, status: "running" };             // отвечает за миллисекунды
});

app.get("/plan/:jobId", async (req, reply) => {    // опрос
  const job = getPlanJob(req.params.jobId);
  if (!job) { reply.code(404); return { status: "not_found", jobId }; }
  if (job.status === "running") return { status: "running", jobId, elapsedMs: ... };
  if (job.status === "error")   return { status: "error", jobId, error: job.error };
  return { status: "done", jobId, ...job.result };
});
```

### Обёртка: тонкий прокси (никаких длинных соединений)

`mcp/mcp-server.mjs` — два инструмента, оба возвращаются за миллисекунды:

```js
mcp.registerTool("plan_task", { ... }, async (args) => {
  const r = await postJson("/plan", args);                 // → { jobId, status: "running" }
  return asContent({ ...r.body, poll: "Call plan_result every ~5s until done/error" });
});

mcp.registerTool("plan_result", {
  inputSchema: { jobId: z.string().min(1) },
}, async ({ jobId }) => {
  const r = await getJson(`/plan/${encodeURIComponent(jobId)}`);  // короткий GET
  const status = r.body?.status;
  return { ...asContent(r.body), isError: status === "error" || status === "not_found" };
});
```

Так undici-потолок (300с) не достаётся вообще — нет долгого fetch.

### SKILL.md: агент обязан опрашивать, а не выдумывать

```
plan_task возвращает { jobId, status:"running" } сразу. НЕ выдумывай план:
ОБЯЗАТЕЛЬНО опрашивай plan_result({ jobId }) каждые ~5с, пока status не станет done или error.
Если до done не дошло — у тебя НЕТ плана; не сочиняй слоты и не зови commit_plan.
```

### Сервисный LLM-таймаут — поднять

Так как kimi-k2.6 может думать >120с на один вызов, в `.env` сервиса:

```
LLM_TIMEOUT_MS=300000   # 5 мин на вызов; план опрашивается async, так что это безопасно
```

## Итог: где какой таймаут крутить

| Слой | Параметр | Где |
|------|----------|-----|
| Вызов MCP-tool агентом | ~60с, **захардкожен** | OpenClaw (issue #61786) — не трогаем, обходим async |
| Долгий fetch обёртка→сервис | undici headersTimeout ~300с | не держим долгих fetch (job+poll) |
| Вызов LLM в сервисе | `LLM_TIMEOUT_MS` | `<service>/.env` |
| Один gather/IO-узел | `GATHER_TIMEOUT_MS` + `with-timeout.ts` | `<service>/.env` |

**Правило для любого долгого инструмента в OpenClaw:** если работа может занять >~60с — делай
инструмент асинхронным (старт-задачи + опрос результата) на стороне сервиса, а MCP-обёртку держи
тонкой. Это в память проекта вынесено отдельно — см. `memory/project_openclaw_mcp_timeout.md`.

Дальше — [урок 07](07-tri-servisa.md): разбор трёх сервисов целиком.
