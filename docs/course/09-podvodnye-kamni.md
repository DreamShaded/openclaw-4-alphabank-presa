# 09 — Подводные камни

Грабли, на которых уже поскальзывались в этом проекте. Формат: **симптом → причина → как чинить**.

## 1. Имя модели Moonshot/Kimi

**Симптом:** `/plan` падает 500; в логах `404 "Not found the model kimi-k2-turbo-preview or
Permission denied"`, `MODEL_NOT_FOUND`.
**Причина:** имени `kimi-k2-turbo-preview` нет на `api.moonshot.ai`. Доступны (по ключу):
`kimi-k2.6` (свежая), `kimi-k2.5`, плюс `moonshot-v1-{8k,32k,128k}` и vision/auto.
**Чинить:** перед тем как гадать — `GET /v1/models` с ключом, выбрать из списка. Конфиг —
`TASK_SCHEDULER_MODEL` (и дефолт в `src/config.ts`).

## 2. Reasoning-модели Kimi и `temperature`

**Симптом:** 400 от API при вызове `kimi-k2.5/k2.6/*-thinking`.
**Причина:** reasoning-модели **запрещают** `temperature`.
**Чинить:** для них слать `topP` вместо `temperature` — это уже сделано в `src/llm.ts`
(`isFixedTempModel = /k2\.[56]|thinking/i`). Своя модель из этого семейства — проверь, что попадает
под регэксп.

## 3. Reasoning-модель отдаёт пустой `content`

**Симптом:** ответ есть, но `choices[0].message.content === ""`, всё ушло в `reasoning_content`,
`finish_reason: "length"`.
**Причина:** при малом `max_tokens` reasoning съедает бюджет, на финальный ответ не остаётся.
**Чинить:** не зажимать `max_tokens`; в сервисе он не задан → дефолт большой. В юнит-тестах модели
закладывай запас.

## 4. Латентность плана нестабильна (60с потолок OpenClaw)

**Симптом:** `MCP error -32001: Request timed out`, либо агент показывает выдуманный план.
**Причина:** kimi-k2.6 считает 2–8 минут (замеры 113/353/504с), а MCP-вызов в OpenClaw
захардкожен на ~60с (issue #61786).
**Чинить:** инструмент — асинхронный (job+poll), см. [урок 06](06-async-i-tajmauty.md). Конфигом
таймаут не поднять.

## 5. gogcli: `calendar create` — не флаги, а позиционный аргумент

**Симптом:** `unknown flag --start` / `unknown flag --calendar`; событие не создаётся.
**Причина:** `gog calendar create <calendarId>` берёт календарь **позиционно**, время — `--from/--to`
(RFC3339), таймзону — `--start-timezone/--end-timezone`. Флагов `--start/--end/--calendar` нет.
**Чинить:** строить argv как `["calendar","create", calendar ?? "primary", "--summary", s, "--from", iso, "--to", iso, ...]`.
Сверяйся с `gog calendar create --help` на нужной версии бинаря (здесь v0.17.0).

## 6. gogcli: `calendar events` — `--calendars`, не `--calendar`

**Симптом:** `unknown flag --calendar, did you mean "--calendars"?` при чтении календаря
праздников.
**Причина:** у `gog calendar events` нет `--calendar` (ед.ч.) — есть `--calendars` (мн.ч.) или `--cal`.
**Чинить:** одиночный id передавать через `--calendars`.

## 7. gogcli: форма ответа `create`

**Симптом:** `calendar create returned no event`, хотя событие создалось (и дубли при ретрае).
**Причина:** `gog` отдаёт одиночное событие как `{ event: { id, htmlLink, ... } }` —
**объект**, не массив. Наивный парсер, ждущий массив или `{id}` на верхнем уровне, промахивается.
**Чинить:** доставать `out.event` (объект), с фолбэками на bare-`{id}` и `{event:[...]}`.

## 8. gogcli MCP сам инжектит `--json --no-input`

**Симптом:** свой CLI-MCP падает на `JSON.parse(stdout)` — там человеческий текст.
**Причина:** `gog` по умолчанию печатает TSV/текст; JSON только с `--json`. Обёртка gogcli
форсит `["--json","--no-input", ...args]`.
**Чинить:** в своей CLI-обёртке так же форсить машинный вывод; на стороне клиента парсить
`{exit, stdout, stderr}` и кидать ошибку при `exit !== 0`.

## 9. zod `.datetime()` отвергает offset `+03:00`

**Симптом:** `/commit` → 400 `invalid request`, когда слот пришёл в форме `...+03:00`.
**Причина:** `z.string().datetime()` по умолчанию принимает только `Z`, не offset.
**Чинить:** `z.string().datetime({ offset: true })` — принимает и `Z`, и `+03:00`. Поправлено в
`server.ts` и в `mcp/mcp-server.mjs` (схемы должны совпадать!).

## 10. undici headersTimeout (~300с) при долгом fetch

**Симптом:** `TypeError: fetch failed` ровно на ~300с, хотя `AbortController` выставлен больше.
**Причина:** дефолтный `headersTimeout` у Node-fetch; сервис не шлёт заголовки до готовности.
`undici` в контейнер обёртки не импортируется, чтобы поднять лимит.
**Чинить:** не держать длинных fetch — асинхронность в сервисе (job+poll), короткие опросы.

## 11. Окно планировщика: задачи в ночь

**Симптом:** personal-план раскладывается на 00:00–06:00.
**Причина:** окно `personal` было 00:00–23:30; `generateFreeSlots` начинает с самого раннего
слота, `distribute` заполняет с начала → ночь.
**Чинить:** окно `PERSONAL_HOURS_START/END` = 09:00–23:00. «Закончить к 23:00» обеспечивается
само (`slots.ts`: `cursor + slotMs <= dayEnd`, блок не выходит за `windowEnd`).

## 12. fail-soft в gather-узлах, а не «упасть всё»

**Симптом (анти-паттерн):** один недоступный источник роняет весь бриф/план.
**Правильно:** каждый IO/gather-узел в `try/catch` возвращает `{ gaps: ["X unavailable: ..."] }`,
а `gaps` накапливаются reducer'ом. Результат строится из того, что доступно; пробелы показываются
пользователю как ⚠. Плюс `withTimeout(p, GATHER_TIMEOUT_MS, label)` на каждый внешний вызов.

## 13. Точные пины версий зависимостей

**Правило проекта:** в `package.json` и lockfile — точные версии (`"4.4.3"`), без `^`/`~`/`latest`.
Ставить `pnpm add <pkg>` и снимать `^` вручную. (См. память проекта `feedback_dep_versions.md`.)

## 14. Холидеи РФ требуют подписки

**Симптом:** в `gaps` — `holidays_unavailable`.
**Причина:** публичный календарь «Праздники в России»
(`ru.russian#holiday@group.v.calendar.google.com`) не подписан у пользователя.
**Чинить:** подписаться в Google Calendar, либо `includeHolidays: true` (тогда фильтр праздников
выключается). Это деградация, не баг — обработать в SKILL.md.

## Связанные заметки в памяти проекта

- `memory/project_openclaw_mcp_timeout.md` — потолок 60с и паттерн async (грабли 4, 10).
- `memory/project_moonshot_kimi_models.md` — имена и поведение моделей (грабли 1, 2, 3).
- `memory/feedback_dep_versions.md` — точные пины (грабли 13).

На этом курс завершён. Возврат к [оглавлению](README.md).
