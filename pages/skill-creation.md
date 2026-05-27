---
layout: simple-slide
variant: 5
transition: slide-left
---

<!-- Slide: Docker Compose launch setup -->
<DockerComposeSlide />

<!--
Давайте уже переходить к практике и небольшой демонстрации. В конце доклада будет куар код на мой репозиторий, это отдельная демка,
которую я вытащил из своего реального клошика, дальше всё будет строиться вокруг моего кода.
пока что я предлагаю запускать агента в изолированном окружении, через докер. 
Официальный образ из документации, простой компоуз, где ключевое - мы указываем директории воркспейс и конфиг, я их прям в гит добавил,
по образцу каждого дот энв экзампл создаём дот энв без экзампл, где указываем ключ нужной модели и юэрэл апи. 
Я использую Кими ка два шесть, это по мнению реддиторов одна из оптимальнейших моделей по соотношению цена и качество. 
Мы монтировали директорию вокрспейс - давайте в ней создадим простейший навык
-->

---
layout: simple-slide
variant: 5
transition: slide-left
---

<!-- Slide: SKILL.md prompt structure -->
<BootstrapFileSlide
  filename="SKILL.md"
  desc="промпт навыка"
  :example="'---\nname: no-junk\ndescription: Отговорить от джанка.\n  Выяснить: голод или триггер?\n---\n«хочу энергетик» → активировать'"
  :blocks="[
    { label: '⚡ Триггеры активации',     color: '#f97316' },
    { label: '🚫 Когда НЕ включать',      color: '#ef4444' },
    { label: '📋 Протокол выполнения',    color: '#3b82f6' },
    { label: '💬 Few-shot диалоги',       color: '#10b981' },
  ]"
/>

<!--
А простейшим является маркдаун файл с текстом на русском или английском языке. 
Считается, что на английском сильно меньше токенов потребляет. Я вот это сильно не заметил,
плюс мы в России - писал на русском языке. 
Мы должны описать по сути классический скилл, как для консольных утилит - фронтмэттер, когда активировать, когда не надо, что делать, и пара примеров диалогов. Вы наверняка уже описывали такие в своих харнессах, ну а если совсем нет - научитесь очень быстро

-->

---
layout: simple-slide
variant: 5
transition: slide-left
---

<!-- Slide: Skills pipeline — auto-discovery on session start -->
<SkillPipelineSlide />

<!--
и тут главное то, что по сути агенты даже через докер получают привычный нам хот релоад, даже хот модуль реплейсмент. 
Написали, открываем сессию - всё, можно пользоваться. Новая сессия на старте читает фронтметтер всех навыков и добавляет их в системный промпт. 

Далее, когда срабатывает триггер - в нашем случае "пользователь хочет энергетика бахнуть" - агент уже прочитает тело навыка и придумает, что с этим делать. Давайте посмотрим, как это выглядит. ДЕМО
-->

---
layout: interjection
variant: 4
transition: slide-up
---

<TextBig>
  ДЕМО
</TextBig>

---
layout: simple-slide
variant: 5
transition: slide-left
---

<DevopsImageSlide />

<!--
Как вы поняли, раньше шутили про YAML-программистов, теперь мы с вами будем программировать Markdown-файлики! Но не всё так просто.
Мы же хотим инструмент, способный дёргать за любую ниточку доступной нам машины, доступных сервисов. Тут мдшкой не обойтись, идём глубже.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 4
---

<!-- Slide: Architecture overview — OpenClaw → MCP → Skills (Docker) -->
<ComplexSkillsArchSlide />

<!--
Когда нам нужен полноценный инструмент, а не просто текстовый навык, мы пишем скилл как отдельный сервис.
Агент OpenClaw общается с каждым скиллом по единому протоколу — MCP. Каждый навык живёт в своём Docker-контейнере.
Кликами показываем: сначала появляется агент, затем протокол, затем скиллы один за другим.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 3
---

<!-- Slide: Each skill = two containers (MCP wrapper + service) -->
<SkillTwoContainersSlide />

<!--
Внутри каждый скилл — это два контейнера. Тонкая MCP-обёртка принимает команды от агента и передаёт их по HTTP реальному сервису.
Агент понятия не имеет, что там внутри — он видит только MCP-интерфейс. Это даёт нам свободу менять реализацию без изменения агента.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 3
---

<!-- Slide: MCP vs REST — stateful long-lived session with bidirectional events -->
<McpVsRestSlide />

<!--
Что такое MCP? Если знакомы с REST — забудьте на минуту. REST это "запрос-ответ-закрыли соединение", одноразовая история.
MCP — это долгоживущая stateful сессия: клиент и сервер один раз договорились, а потом гоняют сообщения в обе стороны.
Сервер сам может пушить уведомления клиенту. Плюс встроенный набор понятий: Tool, Resource, Prompt — REST о таком не знает.
Транспорт под капотом — HTTP + SSE, формат — JSON-RPC 2.0.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 3
---

<!-- Slide: JSON-RPC 2.0 — three message types: Request / Response / Notification -->
<McpMessagesSlide />

<!--
Все сообщения — это JSON-RPC 2.0, всего три типа.
Request — клиент просит что-то сделать и ждёт ответа по id.
Response — сервер отвечает на конкретный id.
Notification — fire-and-forget, без id, без ответа. Это и есть тот самый "server push".
Никакой бинарщины, только JSON — читать и отлаживать легко.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 4
---

<!-- Slide: MCP session lifecycle — Initialize → Discovery → Operation → Shutdown -->
<McpLifecycleSlide />

<!--
Сессия живёт в четыре этапа. Сначала рукопожатие — initialize, обязательное.
Потом discovery — клиент узнаёт, какие инструменты, ресурсы и промпты сервер умеет давать.
Дальше operation — основная работа, может длиться сколь угодно долго.
И в конце graceful shutdown — закрыть может любая сторона.
Между этими шагами сессия живёт минуты, часы, дни — без переподключений.
-->

---
layout: simple-slide
variant: 4
transition: slide-left
clicks: 5
---

<!-- Slide: Skill registration in OpenClaw via docker-compose + config -->
<SkillRegistrationSlide />

<!--
Зарегистрировать навык просто: описываем контейнер в docker-compose, указываем его MCP-адрес в конфиге OpenClaw —
и при следующем старте агент автоматически подключит новый навык. Никакого перезапуска, никакой ручной регистрации.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 3
---

<!-- Slide: Shared infrastructure — one AI model, vector DB, Google API for all skills -->
<SharedInfraSlide />

<!--
Самое вкусное — общая инфраструктура. AI-модель, база знаний, Google API запущены один раз в Docker-сети
и доступны всем скиллам одновременно. Не нужно дублировать — просто подключаемся из каждого контейнера.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 2
---

<!-- Slide: Sample markdown note with frontmatter -->
<NoteExampleSlide />

<!--
Что мы кладём в эту базу знаний? Возьмём пример из демо — обычная markdown-заметка.
Сверху frontmatter — теги, дата, проект. Это метаданные, по ним можно фильтровать.
Снизу — обычный текст в markdown. По нему будем искать по смыслу.
Тысячи таких файлов — и у вас личная база знаний.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 2
---

<!-- Slide: Why RAG — LLM alone vs LLM + retrieved context -->
<RagWhySlide />

<!--
Зачем RAG? Сама по себе LLM не знает про наши заметки — обучали её не на них.
Спросишь — она честно скажет «не знаю» или, что хуже, выдумает.
RAG — Retrieval-Augmented Generation — добавляет шаг: сначала ищем релевантные куски в базе,
потом подсовываем их LLM как контекст. И она отвечает уже по нашим данным.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 2
---

<!-- Slide: Two RAG pipelines — offline indexing + online query -->
<RagPipelinesSlide />

<!--
RAG состоит из двух пайплайнов.
Offline — индексация. Делается заранее: берём заметки, режем на куски, превращаем в векторы, складываем в БД.
Online — на каждый вопрос пользователя: переводим вопрос в вектор, ищем похожие куски, отдаём LLM как контекст.
БД общая для обоих контуров.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 2
---

<!-- Slide: Embeddings — text/meaning encoded as fixed-length vectors -->
<EmbeddingsSlide />

<!--
Что такое эмбеддинг? Это представление текста в виде массива из ~1000 чисел. Любой текст — слово, абзац, страница —
превращается в вектор одной длины.
Главное свойство: близкие по смыслу тексты дают близкие векторы. «Кошка» и «котёнок» окажутся рядом, «кошка» и «налоговый кодекс» — далеко.
Поиск по смыслу = поиск ближайших векторов. Силён в синонимах, слабее в точных кодах и артикулах.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 4
---

<!-- Slide: Three search modes — Dense / Sparse / Hybrid + Qdrant -->
<SearchTypesSlide />

<!--
Три способа искать.
Dense — векторы смысла, ловит синонимы, но мажет по точным кодам.
Sparse — старый-добрый BM25 по словам, наоборот: точные совпадения держит крепко, смысл не понимает.
Hybrid — комбинация двух, рекомендуемый дефолт. Покрывает оба класса запросов.
Поэтому выбираем Qdrant — он поддерживает hybrid из коробки, написан на Rust и работает self-hosted в Docker.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 4
---

<!-- Slide: Project RAG ingestion — markdown → chunker → embeddings → Qdrant + Graph -->
<ProjectRagIngestionSlide />

<!--
Теперь как это устроено у нас в проекте.
Watcher раз в 5 минут проверяет папку с заметками — что изменилось, то переиндексируется.
Файл режется по заголовкам H1/H2 на куски до 3200 символов.
Каждый кусок прогоняется через BGE-M3 — получаем dense вектор 1024 и sparse вектор сразу.
Это всё уходит в Qdrant вместе с метаданными из frontmatter — дата, теги, проект.
Параллельно LLM вытаскивает из текста сущности и связи — это идёт в SQLite-граф.
На демо у меня ~60 файлов, это 502 чанка в Qdrant.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 5
---

<!-- Slide: Project RAG retrieval — hybrid search → rerank → graph → IRCoT synthesis -->
<ProjectRagRetrievalSlide />

<!--
Теперь обратная сторона — что происходит, когда я задаю вопрос.
Сначала Hybrid search в Qdrant: dense + sparse, объединяем через RRF, получаем top-K кусков.
Потом rerank — точная пересортировка кросс-энкодером, чтобы наверх всплыло самое релевантное.
Graph expansion — лезем в SQLite-граф, поднимаем связанные сущности и их чанки. Это даёт нам контекст за пределами прямого совпадения.
IRCoT — если вопрос сложный, разбиваем на под-вопросы и делаем до трёх hops.
Финально Kimi синтезирует ответ из собранного контекста и возвращает с цитатами.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 2
---

<!-- Slide: LangChain overview — building blocks (Agent/Tool/Retriever/Memory) + Runnable -->
<LangchainOverviewSlide />

<!--
Чем мы всё это пишем? LangChain — Python/JS фреймворк, в котором уже готовы кирпичи под типовые паттерны LLM-приложений.
Agent — LLM сам решает, какой инструмент вызвать (паттерн ReAct: Thought → Action → Observation).
Tool — функция с описанием, аналог MCP Tool, только локально в процессе.
Retriever — интерфейс к векторному стору, любому: Chroma, Qdrant, PGVector.
Memory — история диалога между вызовами.
И ключевое — все они реализуют единый интерфейс Runnable, поэтому их можно соединять через pipe.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 2
---

<!-- Slide: LangChain vs LangGraph — linear chain vs state machine -->
<LangchainVsGraphSlide />

<!--
Часто путают LangChain и LangGraph — это разные слои.
LangChain — линейные цепочки и простые агенты: A → B → C. Идеально для RAG-пайплайна или одного агента с инструментами.
LangGraph — это уже граф состояний поверх LangChain: узлы-функции, рёбра-переходы, циклы, ветвления, human-in-the-loop, персистентное состояние.
Грубо: LangChain — это цепочка, LangGraph — полноценный FSM. Для серьёзной многошаговой агентной системы — берёшь LangGraph.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 3
---

<!-- Slide: LCEL — pipe-operator composition for Runnable components -->
<LcelSlide />

<!--
Сборка цепочки в LangChain — это LCEL, LangChain Expression Language. По сути pipe-оператор как в shell.
prompt | llm | parser — три блока, одна цепочка, можно вызвать через invoke.
Поскольку все блоки реализуют Runnable, любой можно подменить — модель, парсер, шаблон.
Добавить retriever или memory — просто ещё одно звено через pipe. Это и делает LangChain удобным конструктором, а не очередной обёрткой над requests.
-->

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 4
---

<!-- Slide: How a skill is exposed to the agent via MCP — registerTool with name/description/inputSchema/handler -->
<McpRegisterToolSlide />

<!--
Теперь самое главное в обёртке — как сервис превращается в навык для агента. Это одна функция: registerTool.
Первый аргумент — имя инструмента, его видит агент.
Второй — описание. И это критически важно: description — это промпт для LLM, не для человека. По нему агент решает, какой инструмент звать. Пишешь скупо — агент промахивается.
Третий — inputSchema на zod. SDK сам валидирует аргументы и публикует схему агенту, чтобы он знал, что передавать.
И обработчик — он просто проксирует во внутренний HTTP-сервис и возвращает content. Флаг isError даёт агенту понять, что вызов сломался.
Всё остальное — транспорт, сессии, /mcp endpoint — SDK берёт на себя. Обёртка остаётся тонкой.
-->

---
layout: interjection
variant: 4
transition: slide-up
---

<TextBig>
  ДЕМО
</TextBig>

<!--
Теперь, когда все кирпичи на столе — давайте посмотрим, как это работает вживую.
-->
