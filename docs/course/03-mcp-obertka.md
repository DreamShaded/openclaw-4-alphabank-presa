# 03 — MCP-обёртка

MCP (Model Context Protocol) — протокол, которым агент OpenClaw общается с инструментами.
Каждый скилл выставляет себя наружу как **MCP-сервер** через тонкую обёртку
(`<service>/mcp/mcp-server.mjs`). Используется `@modelcontextprotocol/sdk`.

## Зачем отдельный слой (а не MCP прямо в сервисе)

1. **Изоляция протокола от логики.** Сервис — обычный HTTP API (его удобно дёргать curl'ом,
   тестировать, переиспользовать из других сервисов: так `meeting-brief` ходит в `rag` по HTTP).
   Обёртка переводит HTTP ↔ MCP.
2. **Разные жизненные циклы.** Обёртка лёгкая (256 МБ в compose), сервис тяжёлый (512 МБ).
3. **Тонкость как фича.** Обёртка не содержит бизнес-логики — только описание инструментов и
   проксирование. Её легко читать и менять контракт инструментов, не трогая ядро.

## Сервер MCP: `mcp/mcp-server.mjs`

Полный скелет (по `task-scheduler/mcp/mcp-server.mjs`):

```js
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StreamableHTTPServerTransport } from "@modelcontextprotocol/sdk/server/streamableHttp.js";
import { isInitializeRequest } from "@modelcontextprotocol/sdk/types.js";
import { z } from "zod";
import { createServer } from "node:http";
import { randomUUID } from "node:crypto";

const BASE_URL = (process.env.TASK_SCHEDULER_URL || "http://task-scheduler:7801").replace(/\/$/, "");

// короткий универсальный proxy в HTTP сервиса
async function request(method, path, body) {
  const res = await fetch(`${BASE_URL}${path}`, {
    method,
    headers: body ? { "content-type": "application/json" } : undefined,
    body: body ? JSON.stringify(body) : undefined,
  });
  const text = await res.text();
  return { ok: res.ok, status: res.status, body: text ? JSON.parse(text) : null };
}
const postJson = (p, b) => request("POST", p, b);
const getJson  = (p)    => request("GET", p);

function asContent(payload) {
  return { content: [{ type: "text", text: JSON.stringify(payload, null, 2) }] };
}
```

### Регистрация инструмента: `registerTool`

```js
function buildMcpServer() {
  const mcp = new McpServer({ name: "task-scheduler", version: "0.1.0" });

  mcp.registerTool(
    "plan_task",                                  // имя tool'а (его видит агент)
    {
      title: "...",
      description: "...подробно: что делает, что вернёт, КАК пользоваться...",
      inputSchema: {                              // zod-схема аргументов
        rawRequest: z.string().min(1).describe("Свободный текст запроса..."),
        deadlineIso: z.string().datetime({ offset: true }).optional().describe("..."),
        category: z.enum(["work", "personal"]).optional(),
        distribution: z.enum(["even", "blocks"]).optional(),
        // ...
      },
    },
    async (args) => {                              // обработчик
      const r = await postJson("/plan", args);
      if (!r.ok) return { ...asContent(r.body), isError: true };
      return asContent({ ...r.body, poll: "Call plan_result ..." });
    },
  );

  // ещё tools: plan_result, commit_plan, plan_healthz
  return mcp;
}
```

Три вещи, которые делают tool полезным для агента:
- **`description`** — это инструкция для LLM, а не для человека. Пиши развёрнуто: что делает,
  что вернёт, в каком порядке звать, чего НЕ делать. Агент выбирает инструмент по описанию.
- **`inputSchema` (zod)** — SDK сам валидирует аргументы и публикует их схему агенту.
- **Обработчик** возвращает `{ content: [...], isError? }`. `isError: true` сигналит агенту, что
  вызов неуспешен (агент должен среагировать, а не считать результат валидным).

### HTTP-транспорт и сессии

Streamable-HTTP MCP работает поверх одного endpoint `/mcp` с сессиями:

```js
const transports = new Map();   // sessionId → transport

const httpServer = createServer(async (req, res) => {
  if (!req.url?.startsWith("/mcp")) { res.writeHead(404).end("not found"); return; }
  const body = req.method === "POST" ? await readJsonBody(req) : undefined;

  const sessionId = req.headers["mcp-session-id"];
  let transport = sessionId ? transports.get(sessionId) : undefined;

  if (!transport) {
    // новая сессия создаётся только на initialize-запрос
    if (req.method !== "POST" || !isInitializeRequest(body)) {
      res.writeHead(400).end("session not initialized"); return;
    }
    transport = new StreamableHTTPServerTransport({
      sessionIdGenerator: () => randomUUID(),
      onsessioninitialized: (id) => transports.set(id, transport),
    });
    await buildMcpServer().connect(transport);
  }
  await transport.handleRequest(req, res, body);
});

httpServer.listen(8080, "0.0.0.0");
```

Логика: первый запрос клиента — `initialize`, на него заводится сессия и отдаётся `mcp-session-id`;
последующие запросы несут этот заголовок и попадают в свой transport.

## MCP-клиент: ходить В чужой MCP

Сервис может быть и **клиентом** MCP. `task-scheduler` читает/пишет календарь через MCP-сервер
`gogcli`. См. `src/clients/gogcli-mcp-client.ts`:

```ts
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StreamableHTTPClientTransport } from "@modelcontextprotocol/sdk/client/streamableHttp.js";

let _client = null;
async function getClient() {                       // singleton + lazy connect
  if (_client) return _client;
  const transport = new StreamableHTTPClientTransport(new URL(config.gogcliMcpUrl));
  const client = new Client({ name: "task-scheduler", version: "0.1.0" });
  await client.connect(transport);
  _client = client;
  return client;
}

async function callGog(argv) {
  const client = await getClient();
  const res = await client.callTool(
    { name: "google_workspace", arguments: { args: argv } },
    undefined,
    { signal, timeout: config.gogcliTimeoutMs },
  );
  const payload = JSON.parse(res.content[0].text);  // { exit, stdout, stderr }
  if (payload.exit !== 0) throw new Error(`gog ... exit=${payload.exit}: ${payload.stderr}`);
  return JSON.parse(payload.stdout);                // gog отдаёт JSON
}
```

Важная деталь про `gogcli`: его обёртка (`gogcli/mcp/mcp-server.mjs`) **сама добавляет
`--json --no-input`** к каждому вызову `gog`:

```js
const finalArgs = ["--json", "--no-input", ...args];   // → spawn("gog", finalArgs)
```

Поэтому `callGog` всегда получает JSON в `stdout` и может его распарсить. Если будешь делать свою
CLI-обёртку — закладывай такой же приём (форсить машинный вывод), иначе клиент будет давиться
человеческим текстом.

## Контракт ответа MCP-tool

Агент получает массив `content`. Конвенция в проекте — один текстовый блок с JSON:

```js
function asContent(payload) {
  return { content: [{ type: "text", text: JSON.stringify(payload, null, 2) }] };
}
```

Плюс флаг `isError: true` для неуспеха. Агент парсит этот JSON и действует по `SKILL.md`.

## Что важно унести

- Обёртка = MCP-сервер (`registerTool` + Streamable-HTTP + сессии), без бизнес-логики.
- `description` инструмента — это промпт для агента; пиши подробно.
- `inputSchema` на zod — валидация + публикация схемы.
- Сервис может быть и MCP-клиентом (ходит в gogcli); машинный вывод CLI форсим флагами.

Дальше — [урок 04](04-skill-md.md): как агент понимает, *когда* и *как* звать эти инструменты.
