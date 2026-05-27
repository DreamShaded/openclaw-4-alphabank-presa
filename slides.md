---
theme: ./slidev-theme-sber
highlighter: shiki
lineNumbers: false
defaults:
  transition: slide-left
title: "Как я поднял AI-агента и снова стал спать спокойно: OpenClaw, скиллы и корпоративная рутина"
mdc: true
layout: title
variant: 6
contentPos: bottom
contentAlign: left
---

# Как я поднял AI-агента и снова стал спать спокойно: OpenClaw, скиллы и корпоративная рутина

Троицкий Роман, ПАО Сбербанк | SmartCare

<!--
Всем привет! приветище! 
Многострадальный доклад, это уже шестая версия, первые две в апреле. В итоге финальную презу клодычем делал.
Инструмент новый, есть набор и лучших практик, но больше эксперименты - тут разберём мой эксперимент.
Буду оч рад критике, но если дышу в правильную сторону - ещё лучше.
Спойлер - система работает, плюс я показывал похожего агента профильным дата сайнс экспертам - сказали нормально)
Надеюсь нанести пользу большей части.
-->

---
src: ./pages/intro.md
---

---
src: ./pages/bootstrap-files.md
---

---
src: ./pages/skill-creation.md
---

---
layout: simple-slide
variant: 5
transition: slide-left
clicks: 5
---

<!-- Takeaways: 4 skills checked + final "we can build complex skills" -->
<TakeawaysSlide />

<!--
Что у нас в итоге.
Простые навыки — markdown с frontmatter, агент сам подхватит.
RAG — поняли, зачем нужны эмбеддинги, hybrid-поиск и Qdrant.
MCP — stateful сессия, JSON-RPC и registerTool как точка входа для агента.
LangChain и LangGraph — конструктор для сборки цепочек и графов состояний.
И главное — теперь у нас есть всё, чтобы делать сложные навыки: сцеплять LLM с любым нужным API.
-->

---
layout: simple-slide
variant: 5
transition: slide-up
---

<!-- Final slide: thanks + QR to github repo -->
<OutroSlide />

<!--
Спасибо за внимание! Весь код демо и навыков — в репозитории по QR-коду.
Буду рад вопросам и обратной связи.
-->
