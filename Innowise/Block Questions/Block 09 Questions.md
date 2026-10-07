---
custom-width: 80
---
## 1. Playwright vs Drupal tests — виды дефектов

**Вопрос:** Какие виды дефектов хорошо обнаруживаются с помощью Playwright end-to-end тестов, а какие лучше покрывать Drupal Unit, Kernel или Functional тестами?

**Question:** What kinds of defects are Playwright end-to-end tests well suited to detect, and which are better covered by Drupal unit, kernel, or functional tests?

**Ответ:** Playwright хорошо подходит для проверки реальных пользовательских сценариев. Он может обнаружить проблемы с навигацией, формами, кнопками, AJAX, permissions, отображением данных и сохранением результата после reload. Unit-тесты лучше использовать для изолированной PHP-логики, Kernel-тесты — для Drupal API, entities и интеграции с базой данных, а Functional-тесты — для проверки Drupal-функциональности на HTTP- и application-уровне.

**Answer:** Playwright is well suited for testing real user journeys. It can detect problems with navigation, forms, buttons, AJAX, permissions, displayed data, and persisted data after a reload. Unit tests are better for isolated PHP logic, Kernel tests for Drupal APIs, entities, and database integration, while Functional tests are useful for testing Drupal functionality at the HTTP and application level.

---

## 2. Risk-based coverage — объём E2E-тестов

**Вопрос:** Почему «покрыть весь проект Playwright» — это цель, основанная на рисках, а не требование воспроизвести каждое поведение Drupal в браузерном тесте?

**Question:** Why is “cover the entire project with Playwright” a risk-based goal rather than a requirement to reproduce every Drupal behavior in a browser test?

**Ответ:** E2E-тесты запускаются медленнее и требуют больше ресурсов, чем Unit или Kernel-тесты. Поэтому нет смысла проверять через браузер каждую внутреннюю функцию Drupal. Нужно выбирать сценарии с наибольшим бизнес-риском: создание проекта, создание Task, изменение статуса, permissions и другие критические действия. Остальную логику лучше покрывать более подходящим уровнем тестирования.

**Answer:** E2E tests are slower and require more resources than Unit or Kernel tests. Therefore, it does not make sense to test every internal Drupal behavior through the browser. We should focus on scenarios with the highest business risk, such as creating a project, creating a Task, changing a status, and checking permissions. Other logic should be covered at a more appropriate testing level.

---

## 3. Stable fixtures — стабильность тестовых данных

**Вопрос:** Почему стабильная стратегия создания тестовых данных важнее количества сгенерированных тестов?

**Question:** Why does a stable fixture strategy matter more than the number of generated test cases?

**Ответ:** Каждый тест должен начинаться с предсказуемого состояния. Если fixtures создаются нестабильно, используют одинаковые имена или зависят от данных предыдущего теста, тесты могут конфликтовать друг с другом. В результате большое количество тестов не повышает качество, а увеличивает количество случайных failures и усложняет debugging.

**Answer:** Every test should start from a predictable state. If fixtures are created inconsistently, use the same names, or depend on data from previous tests, tests can interfere with each other. As a result, having many tests does not necessarily improve quality; it can increase random failures and make debugging harder.

---

## 4. Security — секреты в Playwright

**Вопрос:** Какие риски безопасности возникают при добавлении в Git storage state Playwright, cookies, credentials, traces или screenshots?

**Question:** What are the security risks of committing Playwright storage state, cookies, credentials, traces, or screenshots?

**Ответ:** Playwright storage state и cookies могут содержать активные сессии и authentication tokens. Credentials могут содержать логины и пароли. Traces и screenshots также могут содержать персональные данные, внутренние URL или информацию, которую не должен видеть внешний пользователь. Поэтому такие файлы нельзя коммитить в repository без проверки и очистки.

**Answer:** Playwright storage state and cookies may contain active sessions and authentication tokens. Credentials may contain usernames and passwords. Traces and screenshots can also contain personal data, internal URLs, or information that should not be exposed. Therefore, these files should not be committed to the repository without proper sanitization.

---

## 5. Test isolation — независимость тестов

**Вопрос:** Почему каждый тест должен запускаться независимо и не зависеть от порядка выполнения файлов?

**Question:** Why must every test be able to run independently and not rely on the order in which files happen to execute?

**Ответ:** Playwright может выполнять тесты параллельно и изменять порядок их выполнения. Если один тест создаёт данные, которые затем использует другой тест, результат становится зависимым от порядка запуска. Это приводит к flaky behavior. Независимый тест должен сам создавать необходимые данные или использовать изолированный fixture.

**Answer:** Playwright can run tests in parallel and change their execution order. If one test creates data that another test uses, the result becomes dependent on execution order. This can cause flaky behavior. An independent test should create its own required data or use an isolated fixture.

---

## 6. Synchronization — `waitForTimeout()`

**Вопрос:** Почему `waitForTimeout()` обычно является источником flaky-тестов и как locator assertions и Playwright auto-waiting обеспечивают лучшую синхронизацию?

**Question:** Why is `waitForTimeout()` usually a source of flakiness, and how do locator assertions and Playwright auto-waiting provide better synchronization?

**Ответ:** `waitForTimeout()` просто ждёт заданное количество миллисекунд и не знает, когда операция действительно завершилась. Если приложение работает медленнее, тест может продолжить выполнение слишком рано. Если оно работает быстрее, тест просто теряет время. Playwright лучше использовать через assertions и auto-waiting, например `expect(locator).toBeVisible()` или `toHaveText()`, потому что framework ждёт фактического состояния элемента.

**Answer:** `waitForTimeout()` only waits for a fixed number of milliseconds and does not know when the operation has actually finished. If the application is slower, the test may continue too early. If it is faster, the test only wastes time. Playwright should instead use assertions and auto-waiting, such as `expect(locator).toBeVisible()` or `toHaveText()`, because the framework waits for the actual state of the element.

---

## 7. Locators — роли и labels

**Вопрос:** Почему accessible locators, такие как roles и labels, обычно надёжнее сгенерированных CSS selectors или XPath?

**Question:** Why are accessible locators such as roles and labels generally more robust than generated CSS selectors or XPath expressions?

**Ответ:** Roles и labels основаны на семантике интерфейса, например button, textbox или link. Они меньше зависят от конкретной HTML-структуры и CSS-классов, которые могут измениться во время redesign. Кроме того, такие locators проверяют интерфейс с точки зрения пользователя и помогают одновременно поддерживать accessibility.

**Answer:** Roles and labels are based on UI semantics, such as a button, textbox, or link. They depend less on specific HTML structures and CSS classes, which may change during a redesign. They also test the interface from the user's perspective and help support accessibility at the same time.

---

## 8. `data-testid` — когда использовать

**Вопрос:** Когда использование `data-testid` оправдано, а когда оно может скрыть реальную проблему accessibility или семантической разметки?

**Question:** When is a `data-testid` attribute justified, and when would adding one hide a real accessibility or semantic-markup problem?

**Ответ:** `data-testid` оправдан, когда элемент не имеет стабильного пользовательского или семантического идентификатора. Например, сложный компонент может не иметь подходящего role или accessible name. Но если кнопку можно найти через `getByRole('button', { name: 'Save' })`, добавлять `data-testid` только для удобства теста не нужно. Иначе тест может скрыть проблему с неправильной HTML-разметкой или accessibility.

**Answer:** A `data-testid` is justified when an element has no stable user-facing or semantic identifier. For example, a complex component may not have a suitable role or accessible name. However, if a button can be located with `getByRole('button', { name: 'Save' })`, adding a `data-testid` only for convenience is unnecessary. Otherwise, the test may hide an accessibility or semantic markup problem.

---

## 9. Persistence — проверка после reload

**Вопрос:** Почему тест должен проверять сохранённое состояние после reload, а не только изменение DOM сразу после действия?

**Question:** Why should a test verify persisted behavior after reload instead of only checking an immediately updated DOM element?

**Ответ:** Изменение DOM показывает только то, что frontend сейчас отображает определённое состояние. Оно не гарантирует, что данные были успешно сохранены в Drupal. После reload страница должна снова получить данные из backend. Поэтому проверка после reload позволяет убедиться, что операция действительно сохранилась в базе данных и будет видна пользователю позже.

**Answer:** A DOM change only shows that the frontend currently displays a certain state. It does not guarantee that the data was successfully saved in Drupal. After a reload, the page must load the data again from the backend. Therefore, checking the state after reload verifies that the operation was actually persisted and will still be visible later.

---

## 10. Authorization — скрытая кнопка Approve

**Вопрос:** Почему проверка того, что кнопка Approve скрыта, недостаточна для доказательства того, что неавторизованный пользователь не может одобрить Task?

**Question:** Why is checking that the Approve button is hidden insufficient to prove that an unauthorized user cannot approve a Task?

**Ответ:** Скрытие кнопки является только UI-защитой. Пользователь может вручную вызвать соответствующий URL или HTTP endpoint, не используя интерфейс. Поэтому настоящий security test должен проверять server-side access control. Например, запрос unauthorized пользователя должен получить HTTP 403, а состояние Task не должно измениться.

**Answer:** Hiding the button is only a UI-level protection. A user can manually call the corresponding URL or HTTP endpoint without using the interface. Therefore, a real security test must verify server-side access control. For example, an unauthorized request should receive HTTP 403, and the Task state should remain unchanged.

---

## 11. HTTP 403 — проверка реальной ошибки

**Вопрос:** Как тест может отличить ожидаемый ответ 403 от необработанного exception, который случайно отображается как страница с ошибкой?

**Question:** How can tests distinguish an expected 403 response from an unhandled exception that happens to render an error-looking page?

**Ответ:** Нужно проверять фактический HTTP status code, а не только текст страницы. Для unauthorized action ожидается именно 403. Если сервер возвращает 500, 200 или другой status code, тест должен считать это ошибкой, даже если на странице отображается текст вроде «Access denied». При необходимости также можно проверять response body и логи.

**Answer:** The test should verify the actual HTTP status code, not only the page text. An unauthorized action should return 403. If the server returns 500, 200, or another status code, the test should treat it as a failure even if the page contains text such as “Access denied”. The response body and logs can also be checked when necessary.

---

## 12. Real interactions — Media Library и AJAX

**Вопрос:** Почему Media Library, moderation workflow и Drupal AJAX behaviors следует тестировать через реальные действия пользователя, а не полностью заменять API setup?

**Question:** Why should Media Library, moderation workflow, and Drupal AJAX behaviors be tested through real user interactions rather than replaced entirely by API setup?

**Ответ:** Эти функции зависят от совместной работы Drupal backend, JavaScript, AJAX, forms и permissions. Если создать Media или изменить moderation state напрямую через API, можно проверить только backend-состояние, но не работу пользовательского интерфейса. Например, Media Library должна корректно открыть диалог, выбрать файл и вернуть выбранное значение в форму.

**Answer:** These features depend on the interaction between the Drupal backend, JavaScript, AJAX, forms, and permissions. If we create a Media entity or change a moderation state directly through an API, we may verify the backend state but not the user interface. For example, Media Library should open the dialog, allow the user to select a file, and return the selected value to the form.

---

## 13. Fixtures — API/Drush setup

**Вопрос:** Когда допустимо использовать API, Drush или database setup для fixtures и почему это не должно заменять проверяемый business journey?

**Question:** When is direct API/Drush/database fixture setup appropriate, and why should it not replace the business journey being tested?

**Ответ:** Прямой API, Drush или database setup подходит для подготовки данных, которые не являются частью проверяемого сценария. Например, перед тестом permissions можно заранее создать Project и Task через API. Но если цель теста — проверить создание Task пользователем, сам Task должен быть создан через UI. Иначе тест не проверяет реальный business journey.

**Answer:** Direct API, Drush, or database setup is appropriate for preparing data that is not part of the journey being tested. For example, before a permissions test, we can create a Project and Task through an API. However, if the purpose of the test is to verify Task creation by a user, the Task itself should be created through the UI. Otherwise, the test does not cover the real business journey.

---

## 14. LLM debugging — данные для AI

**Вопрос:** Какие данные следует предоставить LLM для диагностики failed Playwright test и какие чувствительные данные необходимо удалить?

**Question:** What information should be supplied to an LLM when diagnosing a failed Playwright test, and what sensitive information must be removed first?

**Ответ:** Для диагностики полезно предоставить код теста, точный error message, stack trace, locator, URL, шаг, на котором произошла ошибка, и trace или screenshot. Также полезно описать ожидаемое и фактическое поведение. Перед передачей информации необходимо удалить passwords, tokens, cookies, authentication state, API keys и персональные данные.

**Answer:** Useful information includes the test code, exact error message, stack trace, locator, URL, failing step, and a trace or screenshot. It is also useful to describe the expected and actual behavior. Before sharing this information, remove passwords, tokens, cookies, authentication state, API keys, and personal data.

---

## 15. AI-generated tests — бесполезный green test

**Вопрос:** Почему AI-сгенерированный тест может проходить успешно, но при этом не проверять значимое поведение продукта?

**Question:** Why can an AI-generated test pass while asserting no meaningful product behavior?

**Ответ:** AI может создать синтаксически правильный тест со слишком слабым assertion. Например, тест может открыть страницу и проверить, что существует заголовок, хотя реальная задача состоит в проверке создания Task и его сохранения. Такой тест будет green, но он почти не даёт уверенности в бизнес-функциональности.

**Answer:** AI can create a syntactically correct test with a very weak assertion. For example, a test may open a page and check that a heading exists, while the real requirement is to verify Task creation and persistence. Such a test can be green while providing very little confidence in the actual business functionality.

---

## 16. AI mistakes — типичные ошибки

**Вопрос:** Какие распространённые ошибки встречаются в AI-сгенерированных Playwright тестах в отношении selectors, waits, data isolation и assertions?

**Question:** What common mistakes appear in AI-generated Playwright tests regarding selectors, waits, data isolation, and assertions?

**Ответ:** Частые проблемы — использование хрупких CSS/XPath selectors, `waitForTimeout()`, общих fixtures, фиксированных имён данных и слабых assertions. AI также может забыть cleanup или создать зависимость между тестами. Ещё одна проблема — проверка только UI без проверки backend persistence.

**Answer:** Common problems include fragile CSS/XPath selectors, `waitForTimeout()`, shared fixtures, fixed data names, and weak assertions. AI may also forget cleanup or create dependencies between tests. Another common problem is checking only the UI without verifying backend persistence.

---

## 17. AI fixes — проверка причины

**Вопрос:** Почему AI-предложенное исправление нужно проверять по trace и воспроизводимой причине, а не принимать только потому, что тест стал зелёным?

**Question:** Why must an AI-suggested fix be validated against a trace and reproducible cause rather than accepted because the test turns green?

**Ответ:** Зеленый тест не всегда означает исправленную проблему. Например, AI может заменить строгий assertion на более слабый или увеличить timeout. Тест начнёт проходить, но настоящая проблема останется. Поэтому нужно посмотреть trace, воспроизвести failure и убедиться, что исправление устраняет root cause.

**Answer:** A green test does not always mean that the problem has been fixed. For example, AI may replace a strict assertion with a weaker one or increase the timeout. The test will pass, but the real problem will remain. Therefore, we should inspect the trace, reproduce the failure, and verify that the fix addresses the root cause.

---

## 18. Artifacts — traces, screenshots и video

**Вопрос:** Зачем сохранять traces и screenshots только при failure и когда video может дать дополнительную полезную информацию?

**Question:** What is the purpose of retaining traces and screenshots only on failure, and when might video add useful evidence?

**Ответ:** Сохранение артефактов только при failure уменьшает объём данных и не перегружает CI artifacts. При ошибке trace показывает действия, network activity, DOM и состояние страницы, а screenshot показывает визуальный результат. Video может быть полезно, когда проблема связана с последовательностью действий, анимациями, timing или сложным UI-взаимодействием.

**Answer:** Keeping artifacts only on failure reduces the amount of data and avoids unnecessary CI artifacts. When a test fails, a trace can show actions, network activity, the DOM, and page state, while a screenshot shows the visual result. Video can be useful when the problem is related to action sequences, animations, timing, or complex UI interactions.

---

## 19. Retries — диагностика flaky-тестов

**Вопрос:** Почему retries полезны для диагностики, но опасны, если используются для сокрытия flaky-тестов?

**Question:** Why can retries be useful diagnostically but dangerous as a way to hide flaky tests?

**Ответ:** Retry может показать, что failure нестабилен: например, первый запуск падает, а второй проходит. Это важный сигнал для диагностики timing или race condition. Но если просто оставить retries и считать тест стабильным, реальная проблема остаётся. В итоге CI может показывать успешный build, хотя тест иногда ломается.

**Answer:** A retry can show that a failure is intermittent: for example, the first run fails while the second passes. This is an important signal for diagnosing timing issues or race conditions. However, if we simply keep retries and consider the test stable, the real problem remains. As a result, CI may show a successful build even though the test still fails sometimes.

---

## 20. Parallel execution — workers и fixtures

**Вопрос:** Как количество workers и параллельное выполнение должны влиять на naming и cleanup fixtures?

**Question:** How should worker count and parallel execution influence fixture naming and cleanup?

**Ответ:** При параллельном запуске разные workers могут одновременно создавать и изменять данные. Поэтому fixtures должны использовать уникальные имена или IDs, например включать worker или test identifier. Cleanup также должен быть изолированным и не удалять данные другого worker. Это особенно важно для Projects, Tasks и других сущностей, которые используют общее окружение Drupal.

**Answer:** During parallel execution, different workers may create and modify data at the same time. Therefore, fixtures should use unique names or IDs, for example by including a worker or test identifier. Cleanup must also be isolated and must not remove data belonging to another worker. This is especially important for Projects, Tasks, and other entities in the shared Drupal environment.

---

## 21. Smoke vs regression — наборы тестов

**Вопрос:** Что должно входить в быструю smoke suite по сравнению с полной regression suite?

**Question:** What belongs in a fast smoke suite compared with a full regression suite?

**Ответ:** Smoke suite должна содержать небольшое количество самых важных тестов, которые быстро показывают, что приложение в целом работает. Например: login, открытие Project, создание Task и базовое изменение Task. Regression suite должна быть значительно шире и включать permissions, moderation, Media Library, edge cases, accessibility, разные viewport sizes и другие критические сценарии.

**Answer:** A smoke suite should contain a small number of important tests that quickly show whether the application basically works. For example: login, opening a Project, creating a Task, and changing a Task. A regression suite should be much broader and include permissions, moderation, Media Library, edge cases, accessibility, different viewport sizes, and other critical scenarios.

---

## 22. Accessibility — automation и manual testing

**Вопрос:** Почему автоматическое accessibility scanning нужно комбинировать с keyboard tests и проверкой человеком?

**Question:** Why should automated accessibility scanning be combined with keyboard tests and human review?

**Ответ:** Автоматический scanner может найти технические проблемы, например отсутствие label или неправильный contrast, но он не понимает весь пользовательский контекст. Keyboard testing позволяет проверить focus, tab order и возможность выполнить действия без мыши. Human review нужен для проверки реального UX и проблем, которые автоматический инструмент не способен распознать.

**Answer:** An automated scanner can find technical problems, such as missing labels or incorrect contrast, but it does not understand the complete user context. Keyboard testing checks focus, tab order, and whether actions can be completed without a mouse. Human review is needed for real UX problems and issues that automated tools cannot detect.

---

## 23. Viewports vs visual regression — разница

**Вопрос:** Чем тестирование на разных viewport sizes отличается от screenshot-based visual regression testing?

**Question:** How does testing at multiple viewport sizes differ from screenshot-based visual regression testing?

**Ответ:** Тестирование разных viewport sizes проверяет, что приложение остаётся функциональным и корректно адаптируется к разным размерам экрана. Например, Task Board должен работать на desktop и mobile viewport. Visual regression testing использует screenshots и сравнивает их с baseline, чтобы обнаружить визуальные изменения, например неправильные размеры, отступы или позиционирование.

**Answer:** Testing multiple viewport sizes verifies that the application remains functional and adapts correctly to different screen sizes. For example, the Task Board should work on both desktop and mobile viewports. Visual regression testing uses screenshots and compares them with a baseline to detect visual changes such as incorrect sizes, spacing, or positioning.

---

## 24. Coverage matrix — известные gaps

**Вопрос:** Почему в финальной coverage matrix нужно явно указывать manual checks и известные пробелы вместо заявления о полном покрытии?

**Question:** Why should the final coverage matrix explicitly list manual checks and known gaps instead of claiming complete coverage?

**Ответ:** Набор автоматических тестов никогда не гарантирует абсолютно полное покрытие. Некоторые проверки требуют human judgment или ещё не автоматизированы. Поэтому coverage matrix должна показывать, что именно покрывается автоматически, что проверяется вручную и какие gaps остаются. Это даёт реалистичное понимание качества тестирования и помогает планировать следующие задачи.

**Answer:** An automated test suite can never guarantee complete coverage. Some checks require human judgment or may not be automated yet. Therefore, the coverage matrix should show what is covered automatically, what is checked manually, and which gaps remain. This provides a realistic view of test coverage and helps plan future work.