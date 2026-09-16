---
custom-width: 80
---
>Пополняется после каждой сессии.

## 📌 Top Priority Mistakes (повторяются чаще всего)

### 1. Времена (Tenses) — топ-1 проблема

**Правило:** Рассказывая о прошлом опыте (interview stories), используй **Past Simple**, а не Present.

❌ `I am carefully connect backend` — так нельзя (am + базовая форма глагола = ошибка) ✅ `I carefully connected the backend` (Past Simple) ✅ `I was carefully connecting the backend` (Past Continuous, если подчёркиваешь процесс)

|Неправильно|Правильно|Почему|
|---|---|---|
|I spend more time|I spent more time|рассказ о прошлом|
|I create user token|I created a user token|рассказ о прошлом|
|There are bugs|There were bugs|прошедшее время|
|I checked all|I checked everything|+ lexical fix|
|rewrite code|rewrote the code|прошедшее от "write"|
|all started work|everything started working|нужен gerund|

**Правило "am + verb":** никогда не соединяй `am/is/are` с базовой формой глагола. Либо `am + verb-ing` (Present Continuous), либо просто `verb-ed` (Past Simple).

---

### 2. Артикли (the/a/an)

**Правило:** технические части системы (backend, frontend, database) почти всегда требуют **the**, если речь о конкретном проекте.

❌ `backend part of my projects` ✅ `**the** backend part of my projects`

❌ `part of application` ✅ `part of **the** application`

❌ `frontend part` ✅ `**the** frontend part`

_Общее правило:_ the — когда объект конкретный/уже известный из контекста; a/an — когда упоминается впервые/абстрактно.

---

### 3. Лексика: частые ловушки

| Ошибка              | Верно                                           | Комментарий                                       |
| ------------------- | ----------------------------------------------- | ------------------------------------------------- |
| do your app secure  | **make** your app secure                        | "do" не сочетается с прилагательным-результатом   |
| post an email       | **send** an email                               | "post" — публикация в соцсетях, не отправка почты |
| bugs at credentials | bugs **in** the credentials                     | предлог **in**, не "at"                           |
| rightly connect     | **correctly** connect                           | "rightly" звучит неестественно                    |
| bisnes-logic        | **business logic**                              | орфография                                        |
| nessecary           | **necessary**                                   | орфография                                        |
| chenging            | **changing**                                    | опечатка                                          |
| pet-project         | **personal project** / pet project (без дефиса) | естественность                                    |

---

### 4. Конструкции "нужно/необходимо"

❌ `There was necessary to carefully connect backend` ✅ `**It was** necessary to carefully connect **the** backend **with X**`

Калька с русского "было необходимо..." → в английском конструкция: **It was necessary to + verb**

❌ `need carefully connect` ✅ `need **to** carefully connect` (после need всегда **to + verb**)

---

### 5. "with" вместо "is/are" (калька с русского)

**Правило:** описывая состояние/результат ("ответ успешный", "статус ошибочный") — используй **to be (is/are)**, а не "with".

❌ `If response with success` ✅ `If the response **is** successful`

❌ `response with error` ✅ `there**'s** an error` / the response **is** an error

---

### 6. Third person -s (server/frontend/it + verb)

**Правило:** когда подлежащее — оно (server, frontend, app, it, he/she) — к глаголу в Present Simple добавляется **-s**.

| Неправильно           | Правильно                        |
| --------------------- | -------------------------------- |
| frontend get response | the frontend **gets** a response |
| server send request   | the server **sends** a request   |
| server process        | server **processes** ✅           |

---

### 7. Перечисление причин через "such as"

**Правило:** вместо цепочки "or, or, or" — используй **such as** + список через запятую.

❌ `wrong password or user not found or maybe uncorrectly email` ✅ `**such as** a wrong password, a non-existent user, **or** an incorrect email`

---

### 8. need to / try to — не забывай "to"

❌ `need carefully connect` ✅ `need **to** carefully connect`

❌ `I try to validate` (ок само по себе, но лучше просто Present Simple для описания flow) ✅ `I validate` (если речь про общий процесс, а не про попытку)

---

### 9. Отрицание в Present Simple: don't/doesn't, не "have not"

❌ `Users have not access` ✅ `Users **don't have** access`

**Правило:** в разговорном/техническом английском отрицание строится через **do/does + not + base verb**, а не "have not" (это устаревшая/британская форма).

---

### 10. Passive Voice: Past Participle, не Past Simple

❌ `it was wrote at middleware` ✅ `it was **written** in the middleware`

**Правило:** конструкция **be + Past Participle** (3-я форма глагола: written, done, made), а не Past Simple (wrote, did, made — в обычном предложении).

|Base|Past Simple|Past Participle (для Passive)|
|---|---|---|
|write|wrote|**written**|
|do|did|**done**|

---

### 11. "in the middleware", не "at"

❌ `at middleware` ✅ **in** the middleware — когда логика "находится внутри" чего-то (файла, слоя, функции)

---

## 🔑 Концепт: Authentication vs Authorization (частая путаница!)

⚠️ **Важно:** третий смежный процесс — **registration** (создание аккаунта) — это НЕ authentication и НЕ authorization.

⚠️ **Частая ошибка джунов на собесе:** приводить пример про проверку токена/логина как пример **authorization**. Если у тебя в проекте реализована только проверка "залогинен/не залогинен" — это **authentication**, и лучше честно так и сказать, чем путать термины.

**Хороший честный ответ, если нет продвинутого примера:**

> "I mostly implemented authentication — checking whether the user was logged in via a token in the middleware. I haven't built a full authorization system with different roles yet, but I understand the difference: authorization means checking not just _if_ you're logged in, but _what_ you're allowed to do."

### Пример 3: Authentication vs Authorization (мой опыт)

> My middleware checked whether the user had a token, which basically meant checking if they were logged in or not. That's authentication, not authorization — I haven't yet built a full authorization system with different user roles, but I understand the difference between the two.

---

## 🖥️ Концепт: Что происходит после получения HTML (Rendering)

Этот момент часто теряется в ответах — стоит выучить как отдельный блок.

|Этап|Что происходит|
|---|---|
|**Parsing HTML**|браузер разбирает HTML и строит **DOM** (Document Object Model) — дерево элементов страницы|
|**Parsing CSS**|браузер разбирает CSS и строит **CSSOM** (CSS Object Model)|
|**Render Tree**|DOM + CSSOM объединяются в **render tree** — только видимые элементы|
|**Layout (Reflow)**|браузер вычисляет позицию и размер каждого элемента на странице|
|**Paint**|браузер закрашивает пиксели на экране|
|**JavaScript execution**|если есть `<script>`, браузер выполняет JS — может менять DOM и вызывать повторный reflow/paint|

Это всё вместе называется **Critical Rendering Path**.

**Фраза для собеса (заучить):**

> "After receiving the HTML, the browser parses it to build the DOM, parses the CSS to build the CSSOM, combines them into a render tree, then calculates the layout and paints the page. If there's JavaScript, it gets executed too, which might modify the DOM further."

---

### 12. Подлежащее в английском предложении ОБЯЗАТЕЛЬНО

**Это твоя топ-проблема в этом раунде** — встретилась 4+ раза подряд.

❌ `Then get TCP connection` — нет подлежащего! ✅ `Then **it** establishes a TCP connection`

❌ `browser posting HTTP request` ✅ `**the** browser **sends** an HTTP request`

**Правило:** в отличие от русского, в английском нельзя опускать подлежащее и вспомогательный глагол. Каждое предложение = подлежащее + сказуемое, без исключений (кроме повелительного наклонения: "Check the logs.").

---

### 13. Request vs Response — путаница №3 (закрепить навсегда!)

Это уже третий раз за наши разборы — явно слабое место.

- **Request** = что отправляет **клиент/браузер** → серверу
- **Response** = что отправляет **сервер** → обратно клиенту

❌ `server processing this response` (сервер не может обрабатывать response — он его создаёт) ✅ `the server processes this **request**`

**Мнемоника:** Client re**QUEST**s (спрашивает) → Server re**SPONDS** (отвечает).

---

### 14. Избегай "wanna", "gonna" и т.п. на собесе

❌ `I wanna grow up like full-stack developer` ✅ `I **want to** grow **into a** full-stack developer`

**Правило:** "wanna/gonna/gotta" — casual разговорные сокращения, не подходят для формальной речи (собеседования, деловая переписка).

**Плюс:** `grow up` = буквально "повзрослеть" (о детях), не подходит для профессионального роста. Используй просто **grow** или **develop**.

---

### 15. realize vs see — для "осознал/понял"

❌ `I saw that I was interested` ✅ `I **realized** that I was interested`

**Правило:** "see" — увидеть буквально; для "осознать/понять" нужен **realize**.

---

### 16. Дефис в составных прилагательных

❌ `fullstack projects`, `programming related degree` ✅ `**full-stack** projects`, `**programming-related** degree`

**Правило:** когда два слова вместе описывают существительное как единое прилагательное — нужен дефис. Без дефиса "fullstack" выглядит как отдельное существительное (неверно).

---

### 17. "am" + enjoy — снова ошибка "am + verb"

❌ `I am really enjoy programming` ✅ `I **really enjoy** programming` (Present Simple, без "am")

_(см. правило №1 выше — это тот же паттерн, но теперь с "enjoy")_

---

### 18. "was + verb" — тот же паттерн, что "am + verb", но в прошедшем

❌ `I was create`, `I was really feel`, `I done it` ✅ `I **created**`, `I **really felt**`, `I**'d** done it` (или проще: `I finished it`)

**Правило:** "was" не сочетается с базовой формой глагола напрямую. Варианты:

- `was + verb-ing` → Past Continuous (**I was creating** — процесс в моменте)
- просто `verb-ed` → Past Simple (**I created** — законченное действие)
- `had + verb-3` → Past Perfect (**I had done** — действие до другого момента в прошлом)

Это твой главный грамматический паттерн-ошибка в целом: **am/is/are/was + base verb напрямую — всегда неверно.**

---

### 19. Заглавная буква у "I" и имён собственных

❌ `i done it`, `it was spotify-clone` ✅ `**I**'d done it`, `it was a **Spotify** clone`

**Правило:** местоимение "I" — всегда с большой буквы, бренды/названия (Spotify, Google) — тоже.

---

### 20. Second Conditional — для гипотетических ситуаций

Когда описываешь ситуацию, которой **не было**, но которая могла бы случиться:

**Формула: If + Past Simple, ... would + base verb**

❌ `if it will became first I would listen` ✅ `**if it happened**, I **would listen**`

❌ `If their descision will be better` ✅ `If their decision **were/was** better` (после if — Past Simple, не "will be", даже если речь о будущем)

**Не смешивай типы условных в одном предложении** — либо весь First Conditional (if + Present, will + verb — для реальных будущих ситуаций), либо весь Second Conditional (if + Past, would + verb — для гипотетических).

---

### 21. Passive Voice — не забывай "is/are"

Обратная сторона правила №1: если правило №1 про "am/was + base verb" (лишний вспомогательный глагол), то здесь наоборот — **пропущен** вспомогательный глагол в Passive.

❌ `data stored in tables` (без "is" — не Passive, просто сломано) ✅ `data **is** stored in tables`

❌ `data organized as documents` ✅ `data **is** organized as documents`

**Правило:** Passive Voice = **be (am/is/are/was/were) + Past Participle**. Оба компонента обязательны — нельзя пропустить "be".

---

### 22. Present Simple для общих правил/закономерностей, не Past

Когда описываешь общее правило ("когда выбирать X", "как это работает в принципе"), а не конкретный случай из прошлого — используй **Present Simple**.

❌ `when we needed keys`, `when we had a lot of data` ✅ `when we **need** keys`, `when we **have** a lot of data`

**Как отличить:** "Tell me about a bug you fixed" → Past (конкретный случай). "When would you choose SQL vs NoSQL" → Present (общее правило).

---

### 23. Continuous без вспомогательного глагола — ещё один вариант правила №1

❌ `I trying to find`, `I developing projects` ✅ `I **am** trying to find` / **I try** to find (Present Simple) ✅ `I **am** developing` / **I develop** (Present Simple)

**Правило:** любой Continuous (verb+ing) обязательно требует am/is/are перед собой. Без вспомогательного глагола это не предложение, а сломанная фраза — с ЛЮБЫМ подлежащим (I, he, server, it — неважно).

---

### 24. ⚠️ stack vs stuck — похожие по звучанию, разный смысл!

Частая ловушка именно для разработчиков — "stack" знакомое техническое слово, и мозг путает его с "stuck".

- **stack** /stæk/ — структура данных, "стопка" (существительное)
- **stuck** /stʌk/ — застрял, в тупике (прилагательное/причастие)

❌ `I stack in progress` ✅ `I**'m** **stuck**` / `I **get stuck**`

**Устойчивое выражение:** _"When I get stuck, I search for solutions or ask for help."_

---

### 25. Прилагательное, а не наречие, после "be"

❌ `can be more efficiently` ✅ `can be more **efficient**`

**Правило:** после глагола **to be** (is/are/was/be) нужно прилагательное, не наречие. Наречие (-ly) — только для описания глагола действия: _"work efficiently"_, но _"be efficient"_.

---

### 26. "who" для людей, "that" — для вещей

❌ `developers that know more`, `a developer that know` ✅ `developers **who** know more`, `a developer **who** knows`

**Правило:** формально для людей используют **who**, для вещей/объектов — **that/which**. "That" для людей тоже допустимо в разговорном языке, но "who" звучит естественнее и грамотнее.

---

### 27. "features" ≠ "новые вещи/идеи"

Частая лексическая ошибка — использовать "features" в значении "что-то новое", хотя это слово означает конкретно **функции продукта**.

❌ `learn new features from them`, `bring new features to the team` ✅ `learn new **things/approaches**`, `bring new **ideas/perspectives**`

**Запомнить:** feature = функция приложения (например, "dark mode is a new feature"). Для абстрактных "новых вещей, которые узнаю/привношу" — используй **things, ideas, approaches, perspectives**.

---

### 28. give vs bring — что уместнее в контексте команды

❌ `I can give new things to a team` ✅ `I can **bring** new things **to the team**`

**Правило:** "bring" естественнее для "привносить в команду/проект" (что-то нематериальное — идеи, энергию, свежий взгляд), "give" больше про буквальную передачу.

---

### 29. Помни про Past Simple, когда рассказываешь результат истории

❌ `It helps me to get an offer` (для прошлого события) ✅ `It **helped** me get an offer`

**Ещё раз:** если рассказываешь **завершившуюся историю** (Situation → Action → Result) — весь рассказ должен быть в Past Simple, включая финальный результат. Не переключайся на Present на середине истории.

---

### 30. when vs while/whereas — время vs противопоставление

❌ `var is function-scoped when let and const are block-scoped` ✅ `var is function-scoped, **while** let and const are block-scoped`

**Правило:** "when" — только про **время** ("когда происходит X"). Для **противопоставления** двух фактов ("тогда как", "в то время как") нужно **while** или **whereas**. Частая калька с русского, где "когда" используется в обоих смыслах.

---

### 31. any vs some в отрицательных предложениях

❌ `I don't use some special apps` ✅ `I don't use **any** special apps`

**Правило:** **some** — в утвердительных предложениях (_"I use some apps"_), **any** — в отрицательных и вопросах (_"I don't use any apps"_, _"Do you use any apps?"_).

---

### 32. Артикль перед "schedule"

❌ `work with flexible schedule or fixed schedule` ✅ `work with **a** flexible schedule or **a** fixed one`

---

### 33. other vs others

❌ `SOLID and others principles` ✅ `SOLID and **other** principles`

**Правило:** **other** — прилагательное перед существительным (other principles, other things). **others** — самостоятельное местоимение, без существительного после (_"some like this approach, others prefer that one"_).

---

### 34. Zero Conditional — для общих закономерностей ("если X, то всегда Y")

Третий тип условных (после First и Second) — для **общих истин/правил**, а не гипотез или будущих ситуаций:

**Формула: If + Present Simple, ... Present Simple**

❌ `If I learned something new... this day is a good day` ✅ `If I **learn** something new, that day **is** a good day for me`

**Сравнение всех трёх типов:**

|Тип|Формула|Когда|
|---|---|---|
|Zero|If + Present, ... Present|общая закономерность|
|First|If + Present, ... will + verb|реальная будущая возможность|
|Second|If + Past, ... would + verb|гипотеза/нереальная ситуация|

---

### 35. Past Perfect — когда одно прошлое событие раньше другого

Когда рассказываешь историю с **двумя событиями в прошлом**, и одно произошло **раньше** другого — используй **Past Perfect (had + Past Participle)** для более раннего события.

❌ `I see that I forgot` (оба в неверном времени) ✅ `I **realized** that I **had forgotten**` — сначала забыл (Past Perfect), потом это заметил (Past Simple)

**Логика:** более раннее событие → Past Perfect, более позднее (момент рассказа) → Past Simple.

---

### 36. ask + [кому] + [что] — без предлога "to"

❌ `ask clarifying questions to my mentor` ✅ `**ask my mentor** clarifying questions`

**Правило:** с глаголом "ask" порядок **ask + человек + вопрос**, без предлога. Сравни с "explain something **to** someone" — там "to" нужен, а с "ask" — нет.

---

### 37. "all what" не существует — только "everything that"

❌ `check all what I need`, `I have done all what I want` ✅ `check **everything that** I need`, `I have done **everything I** wanted`

**Правило:** "all what" — калька с русского "всё, что". По-английски правильно: **everything that** (или просто everything).

---

## 💬 Behavioral Questions — заготовки ответов

### "Tell me about yourself" — структура

**Формула:** Present → Past/Path → Goal (20-30 секунд, не длиннее)

1. **Present** — кто ты сейчас (учёба/работа/уровень)
2. **Past** — как пришёл к разработке (что изучал, что зацепило)
3. **Future/Goal** — что ищешь сейчас (плавный переход к позиции)

⚠️ Не дублируй сюда рассказ о конкретном проекте — для этого обычно есть отдельный вопрос ("tell me about a project you're proud of").

### Пример 5: Tell me about yourself (готовый шаблон, можно адаптировать)

> My name is Evgeniy. I'm nineteen years old, and I'm currently pursuing a degree in Computer Science. I started with C-family languages, specifically C#, and during my studies, I realized that I was really interested in creating web applications — building UIs and connecting the backend with business logic. I really enjoy programming, and right now I'm at an internship level. I'm looking for an internship or junior position where I can keep building full-stack projects.

**Заметка:** при устной речи "internship/junior" (слэш) произноси как **"internship or junior"**.

### Пример 6: Why do you want to work as a developer?

> I think being a developer is the most suitable career for me. What motivates me is solving problems, debugging, and creating something new — whether it's new features or optimizing existing programs and applications. I remember when I created one of my first big projects, a Spotify clone: I really felt like I had accomplished something, and I enjoyed the result. That feeling of seeing something I built actually work is what made me realize this is what I want to do.

**Заметка:** не дублируй фразы из "Tell me about yourself" — здесь фокус на **мотивации и эмоции**, а не на статусе/пути.

---

### Пример 7: Disagreement with a teammate (когда реального опыта ещё не было)

**Формула для таких случаев:** 1) честно и коротко признать, что опыта не было → 2) сразу перейти к **конкретному пошаговому плану**, а не общим фразам ("I would listen and understand").

> I haven't had a real disagreement like that yet, since I'm still early in my career and mostly work on my own projects. But if it happened, here's how I'd approach it: first, I'd ask them to explain their reasoning, since there's often a good reason I might be missing. Then I'd share my own perspective and the reasoning behind it. If we still disagreed, I'd suggest looking at documentation, best practices, or even trying a quick proof-of-concept for both approaches to see which works better in practice.

**Ключевая идея:** конкретный план действий (спросить → объяснить → проверить на практике) звучит намного увереннее общих фраз "I would listen and try to understand".

---

## 🧑‍💻 General Questions — заготовки ответов (сессия 2)

### Пример 10: How do you keep your code organized?

> I use SOLID and other principles in my projects, such as DRY, KISS, and YAGNI, and I try to keep the folder structure organized. I also refactor parts of the code after implementing a new feature or adding new files. I use Prettier, which helps me keep all files in one consistent format. As for folder structure, I organize by type — components, hooks, utils, and other folders.

### Пример 11: How do you handle critical feedback on code review?

> I haven't had any real code reviews yet, but I try to see critical feedback as a learning opportunity rather than something negative. I gladly accept it, and if something isn't clear, I ask questions to understand the reasoning behind the feedback. Then I try to fix the issues and apply what I learned to future code as well.

### Пример 12: Where do you see yourself in 2-3 years?

> I want to become a mid-level full-stack developer, working on solid, meaningful projects. I also want to have strong knowledge of the technologies I work with. I believe your company can be a great place for me to grow as a developer — I want to become someone who knows both frontend and backend well and can be helpful to any team.

**Заметка:** на реальном собесе замени общее "your company" на что-то **конкретное** про эту компанию — звучит сильнее.

### Пример 13: What do you do when you feel stuck or unmotivated?

> I always try to stay motivated about the task I'm working on, and I try to keep going even when I feel stuck or unmotivated. I take breaks while I'm working to keep my mind focused, and I break the task down into smaller parts, which helps me feel more organized and see results quickly.

### Пример 14: How do you prioritize tasks?

> I try to stay organized under tight deadlines and work efficiently. I try to do the most urgent or important task first. I think it's better to finish one thing properly than to take on everything and not get anything done.

**Опционально для усиления:** можно упомянуть термин **Eisenhower Matrix** (срочность × важность), если реально пользуешься таким подходом.

### Пример 15: Do you prefer working independently or as a team?

> I think it's better for me to work in a team. Working in a team can be more efficient, especially when you have developers who know more solutions than I do or who are more experienced. I can always learn new things from them and become a more skilled developer. At the same time, I think I can also bring new things to the team — new ideas, new ways of solving problems or fixing bugs, and a fresh perspective.

### Пример 16: Something you struggled with recently

> One thing I struggled with recently was having to learn a lot of theory before interviews. At some point, I felt like I didn't want to just study theory all the time, so I started watching mock interviews online as well — it helped me keep my mind fresh and take a break from pure studying while still learning something useful. In the end, it helped me get an offer.

### Пример 17: let vs const vs var (technical)

> These are three ways of declaring variables. Var is the legacy way — it has hoisting and returns undefined when we access the variable before initialization. Let and const throw a ReferenceError and have a temporal dead zone (TDZ). Const requires initialization at the moment of declaration. Also, var is a function-scoped way of declaring variables, while let and const are block-scoped.

### Пример 18: How do you make sure you don't miss deadlines?

> I always check my deadlines and keep track of time. I'm a responsible developer, and I always try to keep in touch with my mentor or other developers. Since I'm not working right now, I don't use any special apps — I just write down notes about what I need to do.

### Пример 19: Learning something new quickly

> One time, I needed to quickly learn how to write unit tests. It was a very important part of my project, where I needed to check the HTTP methods in my application. I found some examples and asked AI for information on how to write tests, and then I coded them myself. In the end, I covered the required functionality with tests.

### Пример 20: What work environment helps you do your best work?

> I value a quiet, productive environment where I can focus, but I can quickly adapt to different conditions. Working from home or from the office isn't a problem for me, and I'm comfortable with both a flexible schedule and a fixed one.

**Заметка:** первая попытка на этот вопрос содержала фразу "I appreciate it when people adapt to me" — звучит так, будто ожидаешь, что команда подстроится под тебя. Лучше подчёркивать свою **гибкость**, а не ожидания к другим.

### Пример 21: How do you make sure your code is readable for others?

> I use SOLID and other principles, write clear comments where needed, and use Prettier to keep a consistent code style across the project.

**Заметка:** не ссылайся на "as I mentioned" — на реальном собесе каждый вопрос отдельный, включай мысль полностью заново.

### Пример 22: How do you make sure you're always learning?

> I always try to learn something new. I read articles, watch technical videos, and try to stay up to date with the technologies I use. I watch videos from Russian developers every day, where they talk about the situation in IT and cover new features and updates — it helps me stay up to date with everything that's happening in the field.

### Пример 23: Explaining something technical to a non-technical person

> I'm a student, so I've had a lot of situations where I needed to explain technical topics to classmates — for example, how databases work, or the basics of C++ and Python. When I explain something technical, I always try to speak clearly and simply, and I avoid using technical terms when I can. Explaining how something works under the hood, or other complex topics, can be tricky — so I try to simplify my explanation and use everyday language. I've helped classmates with coursework more than once.

**Заметка:** этот ответ собран без единого конкретного случая (не смогли быстро вспомнить один в моменте) — перед реальным собесом стоит один раз спокойно вспомнить и проговорить настоящий пример, иначе может не выдержать уточняющий вопрос "can you give a specific example?".

### Пример 24: What does a good day at work look like for you?

> As for me, a good day is a day when you complete your plans. I feel good when I've done everything I wanted to do. If I learn something new during my daily routine, that day is a good day for me.

### Пример 25: Unclear or changing requirements

> I think it's sometimes an integral part of the work. I try to stay confident and organized when working on a task with unclear or changing requirements. If I don't understand the task, I ask my mentor clarifying questions.

### Пример 26: A mistake you made — Voting App story

> I had one mistake not so long ago. I was working on my Voting App project, and when I started checking the frontend part, I realized that I had forgotten about one piece of functionality — a page with personal votes. After I saw that issue, I switched my plan from frontend to backend and created a service and a route with all the required tests and code for that missing functionality. In the end, I managed to finish it successfully.

**Заметка:** один из сильнейших ответов за всю практику — конкретная история, чёткий Situation → Action → Result, плюс тут впервые правильно понадобился Past Perfect.

### Пример 27: When is code "good enough" to ship?

> I consider different approaches to make the code good enough, and I also create a test file for it to check everything I need. That piece of code should be written following SOLID and other principles, and it should be understandable for other developers.

---

## 🗄️ Концепт: SQL vs NoSQL (частый технический вопрос)

**Фраза для собеса (заучить):**

> "We should choose SQL databases when our data is well-structured and stable, when we need keys to connect data, and when we need strong transactions. We should choose NoSQL databases when data changes often, when it's organized as documents, when we have a lot of unstructured data, and when we need fast reads from the database."

### Пример 8: SQL vs NoSQL

> In SQL databases, all data is stored in tables — for example, MySQL or PostgreSQL. In NoSQL databases, data is stored in other forms, such as documents or graphs. We should choose SQL databases when our data is well-structured and stable, when we need keys to connect data, and when we need strong transactions. We should choose NoSQL databases when data changes often, when it's organized as documents, when we have a lot of unstructured data, and when we need fast reads from the database.

### Пример 9: How do you approach learning a new technology?

> I try to find technical articles, documentation, or learning materials. After that, I develop projects using that technology or framework and try to consolidate my knowledge about it. For example, when I learned Next.js, first of all, I found a course and watched the lessons step by step while practicing and building projects along the way. When I get stuck, I try to find more information or search for solutions, or I ask AI assistants for help.

---

## 🎯 STAR Framework для behavioral-вопросов

Когда рассказываешь историю на собесе — держи структуру:

- **Situation** — контекст (что за проект)
- **Task** — что нужно было сделать
- **Action** — что конкретно делал (это должно быть подробно!)
- **Result** — что получилось, чему научился

⚠️ **Частая ошибка**: рассказывать только про Action ("carefully connected", "carefully debugged"), но не объяснять **в чём была сложность и как её нашёл**. Интервьюеру интересен именно процесс решения проблемы.

⚠️ **Ещё одна ошибка (новая, из практики про читаемость кода)**: ссылаться на "as I mentioned" / "as I said" на реальном собесе. Каждый вопрос там будет **отдельным**, без общего контекста прошлых ответов внутри одного диалога с интервьюером (если это не явное продолжение той же темы) — не полагайся на то, что интервьюер помнит прошлый ответ, включай всю нужную мысль заново.

---

## ✅ Разобранные примеры

### Пример 1: Password-reset flow

> I haven't dealt with a tricky issue in production yet — I'm still at the internship level and mostly work on personal projects. But there's a real issue I solved: implementing a password-reset flow with email notifications. When a user wants to change their password, the system sends them an email with a link. At first, though, emails weren't being sent to users. After I carefully debugged the backend, I found there were bugs in the credentials and the email-sending API. I checked everything, rewrote the code, and after that everything started working correctly.

---

### Пример 2: Login flow (frontend → backend → response)

> First, I validate the data on the frontend. I use Zod to check the data in the forms that users interact with. Then, when the login form is submitted, the data is sent to the backend as a request — from the client to the server. The server processes this data and sends a response back to the client with a status code (success or an error). After that, the frontend gets a response from the server, and we need to check what kind of response it is. If the response is successful, we let the user into the app and store the user data in cookies or in a session. If there's an error, we deny the user access to the app and return a status code along with a message explaining what went wrong — such as a wrong password, a non-existent user, or an incorrect email.

**Ключевой смысловой момент:** не путай **request** (клиент → сервер) и **response** (сервер → клиент).

**Security note:** на реальном собесе лучше не раскрывать точную причину ошибки логина (не говорить прямо "user not found"), а использовать общий message типа "invalid credentials" — это защита от brute-force атак на существующие аккаунты.

---

### Пример 4: URL bar → Enter → Rendered page

> First, the browser checks the IP address using DNS. Then it establishes a TCP connection with the server using that IP address. If the site uses HTTPS, it also establishes a TLS connection. Then the browser sends an HTTP request. The server processes this request and sends back an HTTP response. The browser then receives the response, often HTML, and sends additional requests to fetch other resources — like CSS, JS, or images. After receiving the HTML, the browser parses it to build the DOM, parses the CSS to build the CSSOM, combines them into a render tree, then calculates the layout and paints the page. If there's JavaScript, it gets executed too, which might modify the DOM further.

**Структура (заучить порядок):** DNS → TCP → TLS (если HTTPS) → HTTP Request → Server processing → HTTP Response → Parsing (DOM/CSSOM) → Render Tree → Layout → Paint → JS execution

---

## 📄 Полный подготовленный интервью-скрипт

> Собственная заранее подготовленная версия ответов — более отточенная, чем живые черновики выше. Хорошая база для заучивания и адаптации под конкретную компанию.

**Tell me about yourself and your journey into frontend development.**

> My name is Evgeniy. I am 19 years old and currently studying for a programming-related degree. I started my journey with C languages and gradually became interested in frontend and backend development. During my studies, I discovered that I really enjoy building user interfaces and creating applications that people can interact with directly.

**Tell me about your daily routine.**

> There's nothing really unusual about my daily routine. I go to the gym three to four times a week, and the rest of the time I spend studying, coding, and learning theory.

**Why do you want to work as a Software Developer?**

> I chose software development because I enjoy seeing the results of my work immediately. I like building interfaces that users interact with every day. I also enjoy combining programming with design and user experience. I am improving my skills in both frontend and backend development, but working on the client side feels more engaging to me than focusing mainly on backend.

**Why do you want to join our company?**

> I believe your company would be a great place for me to grow as a developer. I have read positive reviews about your mentoring culture and practical approach to learning. I am looking for an environment where I can improve my skills, learn from experienced developers, and contribute to real projects.

**Tell me about a time when you worked in a team.**

> Most of my teamwork experience comes from academic projects. In those projects, we divided responsibilities, developed different parts of the application, and then integrated everything together. This experience taught me the importance of communication, responsibility, and helping teammates when challenges arise.

**What would you do if you disagreed with a teammate's decision?**

> First, I would listen carefully to my teammate's point of view and try to understand the reasoning behind their decision. I would explain my own perspective and discuss the advantages and disadvantages of both approaches. If their solution turned out to be better, I would gladly accept it and learn from it.

**Why should we hire you?**

> You should hire me because I am motivated, responsible, and eager to learn. I take ownership of my tasks and always try to deliver high-quality results. Although I am still at the beginning of my career, I have already built several real projects using React and Next.js, and I am continuously improving my skills. I believe I can quickly become a valuable member of the team.

**What is your strongest technical skill?**

> My strongest technical skill is building React applications. I am comfortable creating reusable components, managing state, working with APIs, and implementing responsive user interfaces. I have also created full-stack projects that included authentication and backend integration — for example, working with databases and web APIs.

**What frontend technologies are you currently learning?**

> Currently, I am improving my knowledge of React and JavaScript with TypeScript. I spend a lot of time learning best practices, application architecture, and ways to write cleaner and more maintainable code. I also try to stay up to date with new features and improvements in the technologies I use.

**How do you keep your technical skills up to date?**

> I keep my technical skills up to date by following official documentation, reading articles, watching technical videos, and exploring new features in the technologies I use. I also learn by building personal projects because practical experience helps me understand concepts much better.

**Describe one project you are proud of.**

> One project I am particularly proud of is an AI-powered web application built with React. The application included user authentication, password recovery via email, and integration with AI services through API requests. Users could communicate with an AI assistant and analyze different types of data. This project helped me gain experience in both frontend and backend development and taught me how to work with modern web technologies.

**What is your biggest weakness?**

> Since I am still at the beginning of my career, I sometimes need more time when working with unfamiliar technologies. However, I learn quickly and actively improve my skills through practice and personal projects.

**Tell me about a challenging bug you had to solve.**

> This relates to one of my biggest projects. I think the most challenging part of development is the backend, because it's the foundation of the app. I spent most of my time properly integrating AI and user authorization into the app. I didn't encounter any critical bugs there, but the integration itself required a lot of careful work and time.

⚠️ **Заметка:** оригинальная версия этого ответа содержала "I didn't encounter any serious difficulties" — что прямо противоречит вопросу про "a challenging bug". Если реального сложного бага не было — лучше сказать честно и переключиться на что было сложным (как в исправленной версии выше), а не отрицать сложность вопроса целиком.

**How do you handle tight deadlines?**

> When working under tight deadlines, I start working on the tasks immediately and focus on completing them efficiently. The first thing I do is break the task down into smaller parts. I try to stay organized, track my progress, and communicate any risks as early as possible. My goal is always to deliver a high-quality result on time.

**What motivates you to learn programming?**

> What motivates me most is my desire to build a career in software development. I genuinely enjoy programming and solving problems, and I like seeing how my skills improve over time. I also find it rewarding to create applications that people can actually use.

**If you don't know how to solve a task, what do you do?**

> If I don't know how to solve a task, I first try to understand the problem and break it down into smaller parts. Then I search for information in the documentation, technical articles, or other reliable sources. If necessary, I watch tutorials or ask more experienced developers for advice. I also use AI tools to help me understand concepts and explore possible solutions, but I always verify the information before applying it.

**What's the last project you developed?**

> Right now, I'm working on a project called Voting App — it's an application for online voting. So far, I've set up a monorepo, and I'm following Git Flow with pull requests. I've also configured CI, husky pre-commit hooks, ESLint and Prettier checks, and automated tests. On the backend side, I've integrated PostgreSQL with Prisma, and I'm using Docker to containerize and deploy the server. Once I finish setting up the backend, I'm planning to focus more on the frontend.

**Do you have any questions for us?**

> Yes, actually I do have a couple of questions. First, what does the onboarding process look like for new developers? And also, I was wondering what opportunities there are for learning and professional growth within the team.

**What are your salary expectations?**

> I don't have a specific number in mind right now, but I'm looking for a paid internship or entry-level position.

**Tell me about a time you made a mistake or failed at something. What did you learn from it?**

> There have been situations where I set my goals too broadly from the start and forgot about the basics. I always want to do more and do it better, but I've learned that it's important to start small first. Now, when I begin a new project or task, I try to focus on the fundamentals before moving on to more complex or ambitious goals.

---

## 📝 Vocabulary Bank (накопительный список)

- password-reset flow
- email notifications
- redirect the user to
- credentials
- debug / debugged / debugging
- business logic
- secure the application
- validate the data
- submit a form
- request / response (не путать!)
- status code
- store data in cookies / in a session
- deny access
- invalid credentials
- non-existent user
- incorrect email
- brute-force attack
- authentication (не authentification!)
- authorization
- registration
- permissions / roles
- middleware
- logged in / logged out
- token (access token, refresh token)
- admin panel
- regular user
- DNS (Domain Name System)
- IP address
- TCP connection
- TLS connection
- HTTP request / HTTP response
- DOM (Document Object Model)
- CSSOM
- render tree
- layout / reflow
- paint
- critical rendering path
- fetch resources
- accomplish something
- motivation / motivated by
- career (не "work" в этом смысле)
- solving problems
- well-structured / unstructured data
- transactions (ACID)
- documents (NoSQL)
- scalability
- keys / relations
- get stuck (не "stack"!)
- consolidate knowledge
- learning materials
- along the way
- folder structure
- refactor / refactoring
- consistent format
- learning opportunity
- mid-level / full-stack
- meaningful projects
- fresh perspective
- tight deadlines
- take a break / take breaks
- mock interview
- get an offer
- hoisting
- temporal dead zone (TDZ)
- ReferenceError
- function-scoped / block-scoped
- keep track of time
- keep in touch with
- comfortable environment
- adapt to conditions
- consistent code style
- clear comments
- stay up to date
- under the hood
- everyday language (vs jargon)
- clarifying questions
- integral part of the work
- missing functionality
- good enough (code quality)
- required tests
- monorepo
- Git Flow / pull requests
- pre-commit hooks
- CI (continuous integration)
- containerize / deploy
- onboarding process
- entry-level position
- ownership of tasks
- eager to learn
- reusable components
- maintainable code