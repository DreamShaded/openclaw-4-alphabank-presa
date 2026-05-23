# 08 — Создаём новый скилл по шагам

Соберём минимальный LangGraph-скилл `example` по образцу `task-scheduler`. Принцип: скопировать
скелет, заменить узлы графа и инструменты.

> Самый быстрый старт — `cp -r task-scheduler example` и переименовать. Ниже — что именно менять.

## Шаг 0. Каркас каталога

```
example/
├── package.json
├── tsconfig.json
├── Dockerfile
├── docker-compose.yml
├── .env.example
├── src/
│   ├── server.ts
│   ├── config.ts
│   ├── llm.ts
│   ├── logger.ts
│   ├── graph/{index.ts, state.ts, prompts.ts, nodes/*.ts}
│   └── clients/    # если нужны внешние сервисы
└── mcp/{package.json, Dockerfile, mcp-server.mjs}
```

## Шаг 1. package.json (точные пины версий)

В этом проекте версии **жёстко зафиксированы** (без `^`/`~`) — копируй из `task-scheduler/package.json`:

```jsonc
{
  "name": "example",
  "type": "module",
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc -p tsconfig.json",
    "start": "node dist/server.js",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "@fastify/cors": "10.0.1",
    "@langchain/core": "0.3.30",
    "@langchain/langgraph": "0.2.39",
    "@langchain/openai": "0.3.17",
    "@modelcontextprotocol/sdk": "1.29.0",
    "dotenv": "16.4.7", "fastify": "5.2.1",
    "pino": "9.5.0", "pino-pretty": "13.0.0", "zod": "3.25.76"
  },
  "devDependencies": { "@types/node": "22.10.5", "tsx": "4.19.2", "typescript": "5.7.3" }
}
```

## Шаг 2. config.ts — разбор env

```ts
import "dotenv/config";
const str = (k: string, d: string) => process.env[k] ?? d;
const int = (k: string, d: number) => Number.parseInt(process.env[k] ?? String(d), 10);
const required = (k: string) => { const v = process.env[k]; if (!v) throw new Error(`${k} required`); return v; };

export const config = {
  port: int("PORT", 7810),
  llm: {
    apiKey: () => required("MOONSHOT_API_KEY"),
    baseUrl: str("MOONSHOT_BASE_URL", "https://api.moonshot.ai/v1"),
    model: str("EXAMPLE_MODEL", "kimi-k2.6"),   // см. урок 09: имя модели важно
    timeoutMs: int("LLM_TIMEOUT_MS", 300_000),  // запас под reasoning + async
  },
};
```

## Шаг 3. llm.ts — копия обёртки Kimi

Скопируй `task-scheduler/src/llm.ts` целиком (`chatText`, `chatJSON`, `isFixedTempModel`). Менять
не нужно — это переиспользуемый слой.

## Шаг 4. Граф (src/graph/)

`state.ts` — состояние с reducer'ами:

```ts
import { Annotation } from "@langchain/langgraph";
export const ExampleStateAnnotation = Annotation.Root({
  input:  Annotation<{ query: string }>(),
  facts:  Annotation<string[]>({ reducer: (a, b) => [...a, ...b], default: () => [] }),
  result: Annotation<string>(),
});
export type ExampleState = typeof ExampleStateAnnotation.State;
```

`nodes/analyze.ts` — LLM-узел:

```ts
import { chatJSON } from "../../llm.js";
import { z } from "zod";
export async function analyzeNode(state) {
  const out = await chatJSON({
    system: "Ты аналитик. Верни JSON {facts: string[]}.",
    user: state.input.query,
    schema: z.object({ facts: z.array(z.string()) }),
    label: "analyze",
  });
  return { facts: out.facts };
}
```

`index.ts` — сборка и запуск:

```ts
import { StateGraph, START, END } from "@langchain/langgraph";
const graph = new StateGraph(ExampleStateAnnotation)
  .addNode("analyze", analyzeNode)
  .addNode("render", renderNode)
  .addEdge(START, "analyze")
  .addEdge("analyze", "render")
  .addEdge("render", END);
const compiled = graph.compile();
export async function runExample(input) { return (await compiled.invoke({ input })); }
```

## Шаг 5. server.ts — Fastify

```ts
import Fastify from "fastify";
import cors from "@fastify/cors";
import { z } from "zod";
import { runExample } from "./graph/index.js";

const app = Fastify();
await app.register(cors, { origin: false });
app.get("/healthz", async () => ({ ok: true, service: "example" }));

const Req = z.object({ query: z.string().min(1) });
app.post("/run", async (req, reply) => {
  const p = Req.safeParse(req.body);
  if (!p.success) { reply.code(400); return { error: "invalid", issues: p.error.issues }; }
  return await runExample(p.data);
});
app.listen({ host: "0.0.0.0", port: config.port });
```

> Долгая работа (>~60с)? Сделай `POST /run` асинхронным (job+poll) по образцу
> `task-scheduler/src/plan-jobs.ts` + `GET /run/:jobId` — см. [урок 06](06-async-i-tajmauty.md).

## Шаг 6. MCP-обёртка (mcp/mcp-server.mjs)

Скопируй из `task-scheduler/mcp/mcp-server.mjs`, поменяй: имя сервера, `BASE_URL`, набор
`registerTool` (`example_run`, `example_healthz`). Скелет — в [уроке 03](03-mcp-obertka.md).

## Шаг 7. Dockerfile + docker-compose.yml

Скопируй `Dockerfile` и `mcp/Dockerfile` из task-scheduler. В `docker-compose.yml` — два сервиса
(`example` + `example-mcp`) и подключение к внешней сети OpenClaw:

```yaml
networks:
  default:
  openclaw:
    external: true
    name: openclaw-4-alphabank-27052026_default
```

Порты прокинь на localhost (`127.0.0.1:7810:7810`, `127.0.0.1:7811:8080`), задай `mem_limit`,
`cap_drop: [NET_RAW, NET_ADMIN]`, `security_opt: [no-new-privileges:true]` — как у соседей.

## Шаг 8. SKILL.md

Создай `.claude/skills/example/SKILL.md` с frontmatter (`name`, `description`, `when_to_use`,
`keywords`) и пошаговым workflow + секцией «Защита от ошибок». Шаблон — [урок 04](04-skill-md.md).

## Шаг 9. Поднять и зарегистрировать

```bash
cd example
cp .env.example .env            # вписать MOONSHOT_API_KEY
docker compose up -d --build
curl -s http://127.0.0.1:7810/healthz | jq

# зарегистрировать MCP-сервер в OpenClaw
docker compose -f ../docker-compose.yml exec openclaw \
  node dist/index.js mcp set example '{"url":"http://example-mcp:8080/mcp","transport":"streamable-http"}'
docker compose -f ../docker-compose.yml restart openclaw
```

## Шаг 10. Проверка

```bash
pnpm typecheck                                  # компилируется
curl -s -X POST http://127.0.0.1:7810/run -H 'content-type: application/json' \
  -d '{"query":"тест"}' | jq                    # сервис считает
docker compose -f ../docker-compose.yml exec openclaw \
  node dist/index.js mcp show example           # OpenClaw видит сервер
```

В GUI спроси то, что описано в `when_to_use` → агент должен активировать скилл и позвать
`example_run`.

## Чеклист готовности

- [ ] `pnpm typecheck` зелёный.
- [ ] `/healthz` отвечает, `/run` считает.
- [ ] MCP-обёртка слушает (`docker compose logs example-mcp` → "listening").
- [ ] `mcp show <name>` показывает сервер; `skills list` — скилл.
- [ ] Версии deps запинены точно (без `^`/`~`).
- [ ] Долгая работа — асинхронная (job+poll), иначе будет `-32001`.
- [ ] В SKILL.md есть «Защита от ошибок».

Дальше — [урок 09](09-podvodnye-kamni.md): грабли, на которых уже поскальзывались.
