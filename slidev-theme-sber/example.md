---
theme: ./
highlighter: shiki
lineNumbers: false
transition: slide-left
title: Моя Презентация
mdc: true
layout: title
variant: 4
---

# Добро пожаловать

Используется локальная тема: `slidev-theme-vzhyx`

<!--
Заметка: Стартовый слайд.
Переход: При клике следующий слайд выедет снизу вверх (slide-up).
-->

---
layout: simple-slide
variant: 4
transition: slide-up
---

# Переход: Slide Up

Этот слайд появился снизу.

<!--
Заметка: Слайд с контентом.
Переход: Следующий слайд "упадет" сверху вниз (slide-down).
-->

---
layout: interjection
variant: 1
transition: slide-down
---

<TextBig>
  Главная мысль
  <br>
  или важная цитата
</TextBig>

<!--
Заметка: Слайд-вставка с крупным текстом.
Переход: Следующий слайд выедет слева направо (slide-right).
-->

---
layout: simple-slide
variant: 5
transition: slide-right
---

# Переход: Slide Right

Этот слайд появился слева (сдвиг вправо).

<!--
Заметка: Еще один контентный слайд.
Переход: Следующий слайд плавно проявится (fade).
-->

---
layout: interjection
variant: 2
transition: fade
---

# FADE IN

<!--
Заметка: Вставка с плавным появлением.
Переход: Этот слайд плавно растворится, открывая следующий (fade-out).
-->

---
layout: simple-slide
variant: 10
transition: fade-out
---

# Переход: Fade Out

Предыдущий слайд растворился, открывая этот.

<!--
Заметка: Контентный слайд.
Переход: Следующий слайд появится с эффектом увеличения (zoom).
-->

---
layout: interjection
variant: 3
transition: zoom
---

# ZOOM!

<!--
Заметка: Вставка с зумом.
Переход: Следующий слайд появится мгновенно без анимации (none).
-->

---
layout: simple-slide
variant: 12
transition: none
---

# Переход: None

Просто резкая смена.

<!--
Заметка: Финальный слайд.
Переход: Конец презентации.
-->

---
layout: two-cols
variant: 4
leftWidth: 50%
transition: slide-left
align: center
---

# Two Cols (Centered)

Слева текст по центру.

::right::

# Right Column

Справа текст тоже по центру.
Используется `align: center` для обеих колонок.

---
layout: two-cols
variant: 5
leftWidth: 30%
transition: slide-up
leftAlign: top
rightAlign: center
---

# Left Top / Right Center

Левая колонка выровнена по верху (`leftAlign: top`).

::right::

# Right Center

Правая колонка выровнена по центру (`rightAlign: center`).

```js
console.log('Vertical Center');
```

---
layout: two-cols
variant: 6
leftWidth: 70%
transition: fade
align: bottom
---

# Both Bottom

Обе колонки выровнены по низу (`align: bottom`).

::right::

# Narrow Right

Узкая колонка для заметок или доп. информации.
Тоже внизу.

```ts{1}{lines:true,startLine:1}
const isAlive = true

```

---
layout: two-cols
variant: 6
leftWidth: 70%
transition: fade
align: bottom
---

````md magic-move
```js
console.log(`Step ${1}`)
```
```js
console.log(`Step ${1 + 1}`)
```
```ts
console.log(`Step ${3}` as string)
```
````


---
layout: two-cols
---

::header::
# Общий заголовок
Контент, который будет растянут на всю ширину сверху.

::default::
# Левая колонка
Текст слева.

::right::
# Правая колонка
Текст справа.




::right::

````md magic-move
```ts{1}
const isSpeacker = isAlive && isZdorov && isTalking
```


````
