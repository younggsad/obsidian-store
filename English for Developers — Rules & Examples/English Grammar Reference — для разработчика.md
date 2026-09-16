---
custom-width: 80
---
> Полный справочник по грамматике: собран на основе твоих реальных ошибок + базовые правила. Открывай, когда нужно быстро что-то проверить.

---

## 1. Времена (Tenses)

### 1.1 Present Simple — общие факты, процессы, привычки

**Форма:** base verb (+ **-s** для he/she/it)

Используй для:

- Технических процессов ("the server processes the request")
- Постоянных фактов ("I work as an intern")
- Привычек ("I always test my code before committing")

⚠️ **Не забывай -s в третьем лице:**

|Неправильно|Правильно|
|---|---|
|the server send|the server **sends**|
|frontend get response|the frontend **gets** a response|
|it work correctly|it **works** correctly|

**Отрицание:** do/does + not + base verb (НЕ "have not") ❌ `Users have not access` → ✅ `Users **don't have** access`

---

### 1.2 Past Simple — законченное действие в прошлом

**Форма:** verb + **-ed** (или 2-я форма неправильного глагола)

Используй для рассказа о том, что уже произошло (interview stories, что делал в проекте):

|Base|Past Simple|
|---|---|
|create|created|
|write|wrote|
|spend|spent|
|check|checked|
|find|found|

**Пример:** _"I created a user token and connected the backend with the email API."_

---

### 1.3 ⚠️ Главная ошибка: "am/is/are/was" + base verb — ТАК НЕЛЬЗЯ НИКОГДА

Это твоя топ-1 системная ошибка (встречалась 8+ раз). Три правильных варианта вместо этого:

|Что хотел сказать|❌ Неправильно|✅ Правильно|
|---|---|---|
|Простое прошедшее действие|I am/was create|**I created**|
|Процесс в моменте (Continuous)|I am connect|**I am connecting** / **I was connecting**|
|Действие до другого момента (Perfect)|I was done it|**I had done it**|

**Правило одной фразой:** после **am/is/are/was/were** глагол должен стоять либо в **-ing** форме, либо это должен быть **Passive Voice** (be + Past Participle, например "was written"). Голая базовая форма после этих слов — всегда ошибка.

---

### 1.4 Present Continuous — действие прямо сейчас / временный процесс

**Форма:** am/is/are + verb-**ing**

**Пример:** _"I'm currently working on a new feature."_ (сейчас, в процессе)

Не путай с Present Simple — Continuous для временного/текущего, Simple для постоянного/привычного.

---

### 1.5 Passive Voice — когда важнее действие, чем исполнитель

**Форма:** be (am/is/are/was/were) + **Past Participle** (3-я форма глагола)

❌ `it was wrote at middleware` ✅ `it was **written** in the middleware`

|Base|Past Simple|Past Participle (для Passive)|
|---|---|---|
|write|wrote|**written**|
|do|did|**done**|
|send|sent|**sent**|
|create|created|**created**|

**Когда использовать:** когда неважно/неизвестно, кто выполнил действие, или фокус на объекте: _"The data is validated on the frontend."_ (неважно кем — просто описываешь систему)

---

### 1.6 Conditionals — условные предложения

**First Conditional** — реальная возможная ситуация в будущем: **If + Present Simple, ... will + base verb** _"If the test fails, I will check the logs."_

**Second Conditional** — гипотетическая ситуация (маловероятная или не случившаяся): **If + Past Simple, ... would + base verb** _"If I had more time, I would refactor this code."_

**Zero Conditional** — общая закономерность/истина ("если X, то всегда Y"): **If + Present Simple, ... Present Simple** _"If I learn something new, that day is a good day for me."_

❌ Частая ошибка — смешивание типов в одном предложении: `If it will became first I would listen` — смешаны First и Second Conditional

✅ Выбери один тип: `If it **happened**, I **would** listen...` (весь Second Conditional)

---

### 1.7 Past Perfect — когда одно прошлое событие раньше другого

**Форма:** had + Past Participle

Когда в истории **два события в прошлом**, и одно случилось **раньше** другого — более раннее событие оформляй в Past Perfect, а более позднее (момент рассказа) — в Past Simple.

❌ `I see that I forgot` ✅ `I **realized** that I **had forgotten**` — сначала забыл (раньше, Past Perfect), потом заметил (позже, Past Simple)

---

## 2. Артикли (the / a / an / без артикля)

### 2.1 Базовое правило

|Артикль|Когда использовать|Пример|
|---|---|---|
|**a/an**|Впервые упоминаешь, неконкретный предмет (из класса объектов)|_"I created **a** user token"_|
|**the**|Конкретный, уже известный из контекста объект|_"**The** backend processes **the** request"_|
|**без артикля**|Неисчисляемые существительные в общем смысле, множественное число в общем смысле|_"I like **programming**"_, _"**Servers** process requests"_|

### 2.2 Технические части системы — почти всегда "the"

Когда речь о **конкретном** проекте/системе:

❌ `backend part of my projects` → ✅ `**the** backend part of my projects` ❌ `frontend part` → ✅ `**the** frontend part` ❌ `part of application` → ✅ `part of **the** application`

### 2.3 Практическое правило для разработчика

- Первое упоминание нового объекта → **a/an**: _"I built **a** login form"_
- Все последующие упоминания того же объекта → **the**: _"...**the** form validates the input"_
- Уникальные/единственные в своём роде вещи (в контексте) → **the**: _"**the** database"_, _"**the** server"_ (если он один в проекте)

---

## 3. Подлежащее — ОБЯЗАТЕЛЬНО в каждом предложении

В отличие от русского, в английском **нельзя опустить подлежащее**.

❌ `Then get TCP connection` — нет подлежащего! ✅ `Then **it** establishes a TCP connection`

❌ `browser posting HTTP request` ✅ `**the** browser **sends** an HTTP request`

**Каждое предложение = подлежащее + сказуемое.** Единственное исключение — повелительное наклонение: _"Check the logs."_

---

## 3.5 Прилагательное, а не наречие, после "be"

❌ `can be more efficiently` ✅ `can be more **efficient**`

**Правило:** после глагола **to be** (is/are/was/be) нужно прилагательное, не наречие. Наречие (-ly) — для описания глагола действия: _"work efficiently"_, но _"be efficient"_.

---

## 3.6 "who" для людей, "that/which" — для вещей

❌ `developers that know more`, `a developer that know` ✅ `developers **who** know more`, `a developer **who** knows`

**Правило:** формально для людей используют **who**, для вещей/объектов — **that/which**. "That" для людей тоже встречается в разговорном языке, но "who" звучит грамотнее.

---

## 4. Предлоги (Prepositions) — частые ошибки

|Неправильно|Правильно|Правило|
|---|---|---|
|bugs **at** credentials|bugs **in** the credentials|in — "внутри" чего-то|
|**at** middleware|**in** the middleware|in — логика "внутри" слоя/файла|
|listen **[без предлога]** my teammate|listen **to** my teammate|listen требует "to"|
|goes **on** the backend|goes **to** the backend|направление — to|
|redirect **on** page|redirect **to** the page|направление — to|

---

## 5. Модальные конструкции ("нужно", "нужно, чтобы")

### "It was necessary to..."

❌ `There was necessary to connect backend` ✅ `**It was** necessary **to** connect **the** backend`

### need to / try to — всегда с "to"

❌ `need carefully connect` ✅ `need **to** carefully connect`

---

## 6. Частые лексические ошибки (не грамматика, а выбор слова)

|Ошибка|Верно|Почему|
|---|---|---|
|do your app secure|**make** your app secure|"do" не сочетается с прилагательным-результатом|
|post an email|**send** an email|"post" — публикация, не отправка почты|
|rightly connect|**correctly** connect|"rightly" звучит неестественно|
|I saw that I was interested|I **realized**|"see" = увидеть буквально, не "осознать"|
|I wanna / gonna|I **want to** / **going to**|casual-сокращения не подходят для собеса|
|grow up like a developer|grow **into** a developer|"grow up" = "повзрослеть" (о детях)|
|such situatios / decision (опечатки)|situ**a**tions, dec**i**sion|проверяй орфографию при печати|

---

## 7. Request vs Response — не путать!

- **Request** = что отправляет **клиент/браузер** → серверу
- **Response** = что отправляет **сервер** → обратно клиенту

**Мнемоника:** Client re**QUEST**s (спрашивает) → Server re**SPONDS** (отвечает).

❌ `server processing this response` (сервер не может обрабатывать response — он его создаёт) ✅ `the server processes this **request**`

---

## 8. Составные прилагательные — дефис

❌ `fullstack projects`, `programming related degree` ✅ `**full-stack** projects`, `**programming-related** degree`

**Правило:** два слова вместе описывают существительное как единое прилагательное → нужен дефис.

---

## 9. Заглавные буквы

- Местоимение **"I"** — всегда с большой буквы
- Бренды/названия (Spotify, Google, React) — с большой буквы

---

## 10. Перечисления — "such as" вместо цепочки "or"

❌ `wrong password or user not found or maybe uncorrectly email` ✅ `**such as** a wrong password, a non-existent user, **or** an incorrect email`

---

## 11. Continuous без вспомогательного глагола

❌ `I trying to find`, `I developing projects` ✅ `I **am** trying to find` / **I try** to find (Present Simple)

**Правило:** любой Continuous (verb+ing) обязательно требует am/is/are перед собой — с любым подлежащим.

---

## 12. stack vs stuck — похожие по звучанию

- **stack** /stæk/ — структура данных (существительное)
- **stuck** /stʌk/ — застрял (прилагательное)

❌ `I stack in progress` → ✅ `I**'m** **stuck**` / `I **get stuck**`

---

## 13. when vs while/whereas — время vs противопоставление

❌ `var is function-scoped when let and const are block-scoped` ✅ `var is function-scoped, **while** let and const are block-scoped`

**Правило:** "when" — только про время. Для противопоставления двух фактов — **while/whereas**. Частая калька с русского "когда".

---

## 14. Мелкие, но частые ошибки

|Ошибка|Верно|Правило|
|---|---|---|
|`I don't use some apps`|`I don't use **any** apps`|any — в отрицаниях/вопросах, some — в утверждениях|
|`flexible schedule or fixed schedule`|`**a** flexible schedule or **a** fixed one`|не забывай артикль|
|`SOLID and others principles`|`SOLID and **other** principles`|other — перед существительным, others — самостоятельно|
|`can be more efficiently`|`can be more **efficient**`|прилагательное после "be", не наречие|
|`developers that know`|`developers **who** know`|who — для людей|
|`learn new features from them`|`learn new **things/ideas**`|features = функции продукта, не абстрактные "новые вещи"|
|`give new things to a team`|`**bring** new things **to the team**`|bring — для нематериального вклада в команду|
|`ask questions to my mentor`|`**ask my mentor** questions`|ask + человек + вопрос, без "to"|
|`check all what I need`|`check **everything that** I need`|"all what" не существует — только "everything that"|

---

## 🎯 Быстрая шпаргалка перед собесом (проверь себя по этому списку)

- [ ]  Не соединяю am/is/are/was напрямую с базовой формой глагола
- [ ]  В каждом предложении есть подлежащее
- [ ]  Проверяю -s в третьем лице (he/she/it/server/frontend + verb)
- [ ]  The — для конкретных/уже известных вещей, a/an — для новых
- [ ]  Request (клиент→сервер) vs Response (сервер→клиент) — не путаю
- [ ]  Conditionals не смешиваю (Zero/First/Second — выбираю один тип)
- [ ]  Если в истории два прошлых события — более раннее в Past Perfect
- [ ]  Не использую wanna/gonna
- [ ]  Listen **to**, redirect **to**, bugs **in**