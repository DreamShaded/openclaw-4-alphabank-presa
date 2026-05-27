# 02 — LangGraph и LangChain

LangGraph-скиллы (`task-scheduler`, `meeting-brief`) построены на двух библиотеках:

- **`@langchain/langgraph`** — оркестрация: описываем пайплайн как граф состояния.
- **`@langchain/openai`** — вызов LLM (Kimi через OpenAI-совместимый API Moonshot).

## Зачем граф, а не просто `await` по очереди

Можно было бы написать `const a = await intake(); const b = await decompose(a); ...`. Граф даёт:
- **Параллелизм через топологию**: несколько узлов из одной точки исполняются параллельно,
  а следующий узел ждёт их все (fan-out / fan-in) — без ручного `Promise.all`.
- **Единое типизированное состояние** с правилами слияния (reducers): узлы возвращают
  *кусочки* состояния, граф их сливает.
- **Декларативность**: пайплайн виден как список рёбер, а не как лапша из await.

## StateGraph: сборка графа

`task-scheduler/src/graph/index.ts` — граф собирается из узлов и рёбер:

```ts
const graph = new StateGraph(PlanStateAnnotation)
  .addNode("intake", intakeNode)
  .addNode("decompose", decomposeNode)
  .addNode("gather-busy", gatherBusyNode)
  .addNode("gather-holidays", gatherHolidaysNode)
  .addNode("schedule", scheduleNode)
  .addNode("render", renderNode)
  .addEdge(START, "intake")
  .addEdge("intake", "decompose")        // ┐ из intake расходятся
  .addEdge("intake", "gather-busy")      // ┤ три ветки параллельно
  .addEdge("intake", "gather-holidays")  // ┘
  .addEdge("decompose", "schedule")      // ┐ schedule ждёт все три
  .addEdge("gather-busy", "schedule")    // ┤ (fan-in)
  .addEdge("gather-holidays", "schedule")// ┘
  .addEdge("schedule", "render")
  .addEdge("render", END);

const compiled = graph.compile();
```

Картинка топологии:

```
START → intake ─┬→ decompose ───────┐
                ├→ gather-busy ──────┼→ schedule → render → END
                └→ gather-holidays ──┘
```

`decompose`, `gather-busy`, `gather-holidays` стартуют **одновременно** после `intake`, а
`schedule` запускается только когда **все три** завершились. Это и есть fan-out → fan-in,
заданный одними рёбрами.

Запуск:

```ts
export async function runPlan(input: PlanInput): Promise<PlanResult> {
  const final = (await compiled.invoke({ input })) as PlanState;
  // final содержит всё накопленное состояние всех узлов
  return { planMd: final.planMd, placed: final.placed.map(...), ... };
}
```

## Состояние и reducers (Annotation)

`src/graph/state.ts` описывает форму состояния через `Annotation`. Каждый узел возвращает
**только изменённые поля**, граф их мёржит. Для полей-списков задаётся reducer, который
**накапливает** значения от разных узлов. Пример из meeting-brief (`gaps` собираются со всех
gather-узлов):

```ts
gaps: Annotation<string[]>({
  reducer: (prev, next) => [...prev, ...next],  // накопление
  default: () => [],
}),
```

Узел `gather-gmail` упал → вернул `{ gaps: ["gmail unavailable: ..."] }`; reducer добавит это к
тому, что вернули другие узлы. В итоге `final.gaps` — объединённый список проблем.

Без reducer (скалярные поля типа `planMd`) последнее записанное значение побеждает.

## Узел = чистая (по возможности) функция состояния

Узел принимает текущее состояние и возвращает частичное обновление:

```ts
export function scheduleNode(state: PlanState): Partial<PlanState> {
  // ... читает state.subtasks, state.busy, ...
  return { placed: result.placed, unplaced: result.unplaced, gaps };
}
```

Узлы бывают синхронные (чистая логика, `schedule`/`render`) и асинхронные (LLM/IO,
`intake`/`decompose`/`gather-*`). Граф одинаково ждёт и те, и другие.

## Вызов LLM: `src/llm.ts`

Тонкая обёртка над `ChatOpenAI`, настроенная на Moonshot (Kimi). Полный разбор:

```ts
import { ChatOpenAI } from "@langchain/openai";

function isFixedTempModel(model: string): boolean {
  return /k2\.[56]|thinking/i.test(model);   // reasoning-модели Kimi
}

function chat(): ChatOpenAI {
  const model = config.llm.model;
  const params = {
    apiKey: config.llm.apiKey(),
    model,
    timeout: config.llm.timeoutMs,   // 120с+ (см. урок 06)
    maxRetries: 2,                   // p-retry: всего 3 попытки
    configuration: { baseURL: config.llm.baseUrl }, // https://api.moonshot.ai/v1
  };
  if (isFixedTempModel(model)) params.topP = 0.95;     // reasoning: temperature НЕЛЬЗЯ
  else params.temperature = 0.2;                        // обычные: можно
  return new ChatOpenAI(params);
}
```

Ключевой нюанс: **reasoning-модели Kimi (`kimi-k2.5`, `kimi-k2.6`, `*-thinking`) отвергают
параметр `temperature`** — для них задаём `topP` вместо `temperature`. Если этого не сделать,
API вернёт 400. (Подробнее про модели Moonshot — [урок 09](09-podvodnye-kamni.md).)

### Две функции-помощника

```ts
// произвольный текст (для render-узла)
export async function chatText(args: { system; user }): Promise<string>

// строго JSON по zod-схеме (для intake/decompose/reconcile)
export async function chatJSON<T>(args: {
  system; user; schema: ZodTypeAny; label?
}): Promise<T> {
  const sys = `${args.system}\n\nReturn ONLY a valid JSON object. No prose, no markdown fences.`;
  const raw = await chatText({ system: sys, user: args.user });
  const cleaned = stripFences(raw).trim();      // снимаем ```json ... ```
  const parsed = JSON.parse(cleaned);            // упадёт → понятная ошибка с label
  return args.schema.parse(parsed) as T;         // zod-валидация формы
}
```

`chatJSON` решает вечную проблему «LLM вернул почти-JSON»:
1. в системный промпт добавляется «верни только JSON»;
2. `stripFences` срезает markdown-обёртку ` ```json `;
3. `JSON.parse` + zod-схема — гарантия, что дальше по коду структура валидна, иначе явная ошибка.

## Узел, использующий chatJSON

`decompose` (псевдокод по реальной структуре):

```ts
export async function decomposeNode(state: PlanState): Promise<Partial<PlanState>> {
  const subtasks = await chatJSON<Subtask[]>({
    system: DECOMPOSE_SYSTEM,             // из src/graph/prompts.ts
    user: JSON.stringify(state.classified),
    schema: SubtaskArraySchema,           // zod
    label: "decompose",
  });
  // +25% буфера к оценке (доменное правило)
  return { subtasks: subtasks.map(applyBuffer) };
}
```

Промпты вынесены в `src/graph/prompts.ts` — отдельно от логики узлов, чтобы их можно было
править и сравнивать, не трогая код.

## meeting-brief: тот же приём, шире fan-out

`meeting-brief/src/graph/index.ts`: `classify` → **четыре** параллельных gather
(`gather-rag`, `gather-gmail`, `gather-calendar`, `gather-drive`) → `reconcile` → `render`.

- `classify` (chatJSON) — разбирает запрос: контрагент, проект, диапазон дат, поисковые запросы.
- 4× gather — параллельно опрашивают RAG, Gmail, Calendar, Drive (fail-soft: ошибка → `gaps`).
- `reconcile` (chatJSON) — сводит факты в бакеты `{context, discussed, ours, theirs, open, blockers}`.
- `render` (chatText) — markdown.

Один и тот же скелет: **classify (LLM) → параллельный сбор данных → свод (LLM) → рендер (LLM)**.

## Что важно унести

- Граф = узлы + рёбра; параллелизм задаётся топологией, не кодом.
- Состояние сливается reducer'ами; списки накапливаются (`gaps`).
- LLM зовём через `chatJSON`/`chatText`; reasoning-модели требуют `topP`, а не `temperature`.
- Дорогие LLM-узлы держим в минимуме, доменную логику — чистыми функциями.

Дальше — [урок 03](03-mcp-obertka.md): как сервис превращается в инструмент для агента.

Если коротко — это **локальный граф знаний поверх SQLite (better-sqlite3, WAL)**, который строится параллельно с векторной базой и достраивает контекст там, где векторного поиска не хватает.

## Что внутри

Файл `rag/data/graph.sqlite` · пять таблиц:

| Таблица | Что хранит |
|---|---|
| `entities` | сущности из заметок: `id`, `type`, `name`, `normalized`, `mentions` |
| `entity_aliases` | алиасы, **типизированные**: `person:ada` ≠ `project:ada` |
| `edges` | направленные связи `src → rel → dst` с весом |
| `chunk_entities` | маппинг chunk_id ↔ entity_id (через какой кусок упомянуто) |
| `extractions_cache` | кэш LLM-извлечений, чтобы не дёргать модель повторно |

**Типы сущностей:** `person`, `project`, `tech`, `decision`, `event`, `commitment`.

## Как наполняется

На ingest, **после** того как чанк уже залит в Qdrant, для него вызывается `graph/extractor.ts`:
- идёт в LLM (`moonshot-v1-8k` — дешёвая JSON-mode модель)
- получает массив сущностей и связей
- пишет в `entities` / `edges` / `chunk_entities`
- параллельно крутится 4 таких вызова (`GRAPH_INGEST_CONCURRENCY=4`)
- результат кэшируется — пока чанк не изменился, LLM повторно не дёргается

Порядок «сначала Qdrant, потом граф» — это про crash safety: если упадём на graph-extract, у нас уже есть рабочий векторный поиск, а граф просто достроится при следующей переиндексации.

## Зачем он нужен — graph expansion при поиске

В `retrieve.ts` в режиме `mode='deep'`:

1. векторный поиск даёт top-K кусков
2. для найденных кусков смотрим в `chunk_entities` — какие сущности там упомянуты
3. по `edges` находим связанные сущности (по проекту, людям, технологиям)
4. через `chunk_entities` обратно — поднимаем **дополнительные** чанки про эти сущности
5. всё это идёт в контекст LLM

Это покрывает классический провал чистого RAG: задаёшь вопрос про «проект интеграция», находишь куски где он явно назван — но **не находишь** заметку, где обсуждается решение по нему, потому что само название проекта там не повторяется. Граф вытаскивает её по связи `Максим — owner — integration-module`.

## Ограничения

- **Локальный**: не шарится между несколькими rag-инстансами. Если масштабировать горизонтально — нужен Postgres
- Качество графа = качество LLM-extraction; на дешёвой модели бывают шумные сущности
- Текущий объём на демо: можно посмотреть руками — `sqlite3 rag/data/graph.sqlite "SELECT type, COUNT(*) FROM entities GROUP BY type;"`

Если интересно — могу нарисовать отдельный слайд именно про граф, показать схему таблиц и как graph expansion дотягивает связанные чанки.