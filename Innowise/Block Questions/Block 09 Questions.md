---
custom-width: 80
---
## 1. Playwright vs Drupal tests — виды дефектов

**Вопрос:** Какие виды дефектов хорошо обнаруживаются с помощью Playwright end-to-end тестов, а какие лучше покрывать Drupal Unit, Kernel или Functional тестами?

**Question:** What kinds of defects are Playwright end-to-end tests well suited to detect, and which are better covered by Drupal unit, kernel, or functional tests?

**Ответ:** Playwright хорошо проверяет пользовательские сценарии, UI, навигацию, формы, AJAX и permissions. Unit лучше подходит для изолированной PHP-логики, Kernel — для Drupal API и entities, Functional — для Drupal-функциональности на HTTP-уровне.

**Answer:** Playwright is good for user journeys, UI, navigation, forms, AJAX, and permissions. Unit tests are better for isolated PHP logic, Kernel tests for Drupal APIs and entities, and Functional tests for Drupal functionality at the HTTP level.

---

## 2. Risk-based coverage — объём E2E-тестов

**Вопрос:** Почему «покрыть весь проект Playwright» — это цель, основанная на рисках, а не требование воспроизвести каждое поведение Drupal в браузерном тесте?

**Question:** Why is “cover the entire project with Playwright” a risk-based goal rather than a requirement to reproduce every Drupal behavior in a browser test?

**Ответ:** E2E-тесты медленные и дорогие, поэтому их нужно использовать для критических пользовательских сценариев. Внутреннюю Drupal-логику лучше тестировать на более низком уровне.

**Answer:** E2E tests are slow and expensive, so they should focus on critical user journeys. Internal Drupal logic is better tested at a lower level.

---

## 3. Stable fixtures — стабильность тестовых данных

**Вопрос:** Почему стабильная стратегия создания тестовых данных важнее количества сгенерированных тестов?

**Question:** Why does a stable fixture strategy matter more than the number of generated test cases?

**Ответ:** Тестам нужно предсказуемое начальное состояние. Если данные конфликтуют или зависят от других тестов, большое количество тестов только увеличивает нестабильность.

**Answer:** Tests need a predictable initial state. If data conflicts or depends on other tests, more tests only increase instability.

---

## 4. Security — секреты в Playwright

**Вопрос:** Какие риски безопасности возникают при добавлении в Git storage state Playwright, cookies, credentials, traces или screenshots?

**Question:** What are the security risks of committing Playwright storage state, cookies, credentials, traces, or screenshots?

**Ответ:** Они могут содержать cookies, tokens, credentials, sessions или другие чувствительные данные. Их публикация может привести к несанкционированному доступу.

**Answer:** They may contain cookies, tokens, credentials, sessions, or other sensitive data. Committing them can lead to unauthorized access.

---

## 5. Test isolation — независимость тестов

**Вопрос:** Почему каждый тест должен запускаться независимо и не зависеть от порядка выполнения файлов?

**Question:** Why must every test be able to run independently and not rely on the order in which files happen to execute?

**Ответ:** Playwright может запускать тесты параллельно и в разном порядке. Зависимость между тестами делает результаты нестабильными.

**Answer:** Playwright can run tests in parallel and in different orders. Dependencies between tests make results unstable.

---

## 6. Synchronization — `waitForTimeout()`

**Вопрос:** Почему `waitForTimeout()` обычно является источником flaky-тестов и как locator assertions и auto-waiting Playwright обеспечивают лучшую синхронизацию?

**Question:** Why is `waitForTimeout()` usually a source of flakiness, and how do locator assertions and Playwright auto-waiting provide better synchronization?

**Ответ:** Фиксированная задержка не знает, когда операция завершилась. Playwright автоматически ждёт нужного состояния элемента, например `toBeVisible()` или `toHaveText()`.

**Answer:** A fixed delay does not know when an operation is finished. Playwright automatically waits for the required element state, such as `toBeVisible()` or `toHaveText()`.

---

## 7. Locators — роли и labels

**Вопрос:** Почему accessible locators, такие как roles и labels, обычно надёжнее сгенерированных CSS selectors или XPath?

**Question:** Why are accessible locators such as roles and labels generally more robust than generated CSS selectors or XPath expressions?

**Ответ:** Roles и labels основаны на семантике интерфейса и меньше зависят от HTML-структуры и CSS-классов. Они также лучше отражают реальное взаимодействие пользователя.

**Answer:** Roles and labels are based on UI semantics and depend less on HTML structure and CSS classes. They also better represent real user interaction.

---

## 8. `data-testid` — когда использовать

**Вопрос:** Когда использование `data-testid` оправдано, а когда оно может скрыть реальную проблему accessibility или семантической разметки?

**Question:** When is a `data-testid` attribute justified, and when would adding one hide a real accessibility or semantic-markup problem?

**Ответ:** `data-testid` полезен, когда нет стабильного семантического locator. Но его не следует использовать вместо role или label, если это скрывает проблему доступности.

**Answer:** `data-testid` is useful when there is no stable semantic locator. It should not replace a role or label when that would hide an accessibility problem.

---

## 9. Persistence — проверка после reload

**Вопрос:** Почему тест должен проверять сохранённое состояние после reload, а не только изменение DOM сразу после действия?

**Question:** Why should a test verify persisted behavior after reload instead of only checking an immediately updated DOM element?

**Ответ:** Изменение DOM не доказывает, что данные сохранены в Drupal. Reload проверяет, что состояние действительно сохранено и загружается из backend.

**Answer:** A DOM change does not prove that data was saved in Drupal. Reload verifies that the state was actually persisted and loaded from the backend.

---

## 10. Authorization — скрытая кнопка Approve

**Вопрос:** Почему проверка того, что кнопка Approve скрыта, недостаточна для доказательства того, что неавторизованный пользователь не может одобрить Task?

**Question:** Why is checking that the Approve button is hidden insufficient to prove that an unauthorized user cannot approve a Task?

**Ответ:** Скрытая кнопка проверяет только UI. Пользователь всё ещё может напрямую вызвать endpoint, поэтому нужно проверить серверный доступ, например HTTP 403.

**Answer:** A hidden button only checks the UI. A user can still call the endpoint directly, so server-side access must also be tested, for example with HTTP 403.

---

## 11. HTTP 403 — проверка реальной ошибки

**Вопрос:** Как тест может отличить ожидаемый ответ 403 от необработанного exception, который случайно отображается как страница с ошибкой?

**Question:** How can tests distinguish an expected 403 response from an unhandled exception that happens to render an error-looking page?

**Ответ:** Нужно проверять фактический HTTP status code, а при необходимости и содержимое ответа. Одной проверки текста «Access denied» недостаточно.

**Answer:** The test should check the actual HTTP status code and, when necessary, the response content. Checking only “Access denied” text is not enough.

---

## 12. Real interactions — Media Library и AJAX

**Вопрос:** Почему Media Library, moderation workflow и Drupal AJAX behaviors следует тестировать через реальные действия пользователя, а не полностью заменять API setup?

**Question:** Why should Media Library, moderation workflow, and Drupal AJAX behaviors be tested through real user interactions rather than replaced entirely by API setup?

**Ответ:** Эти функции зависят от совместной работы UI, JavaScript, AJAX, Drupal и permissions. Только реальное взаимодействие проверяет весь пользовательский сценарий.

**Answer:** These features depend on the interaction between the UI, JavaScript, AJAX, Drupal, and permissions. Only real interaction verifies the complete user journey.

---

## 13. Fixtures — API/Drush setup

**Вопрос:** Когда допустимо использовать API, Drush или database setup для fixtures и почему это не должно заменять проверяемый business journey?

**Question:** When is direct API/Drush/database fixture setup appropriate, and why should it not replace the business journey being tested?

**Ответ:** API, Drush или database setup удобно использовать для быстрой подготовки тестовых данных. Но само бизнес-действие, которое мы тестируем, должно выполняться через UI.

**Answer:** API, Drush, or database setup is useful for quickly preparing test data. However, the business action being tested should still be performed through the UI.

---

## 14. LLM debugging — данные для AI

**Вопрос:** Какие данные следует предоставить LLM для диагностики failed Playwright test и какие чувствительные данные необходимо удалить?

**Question:** What information should be supplied to an LLM when diagnosing a failed Playwright test, and what sensitive information must be removed first?

**Ответ:** Нужно предоставить код теста, ошибку, stack trace, locator, URL и trace или screenshot. Перед передачей нужно удалить passwords, tokens, cookies, credentials и персональные данные.

**Answer:** Provide the test code, error, stack trace, locator, URL, and trace or screenshot. Remove passwords, tokens, cookies, credentials, and personal data first.

---

## 15. AI-generated tests — бесполезный green test

**Вопрос:** Почему AI-сгенерированный тест может проходить успешно, но при этом не проверять значимое поведение продукта?

**Question:** Why can an AI-generated test pass while asserting no meaningful product behavior?

**Ответ:** AI может создать слабый assertion, например проверить только наличие элемента. Такой тест будет зелёным, но не докажет, что бизнес-операция работает.

**Answer:** AI may create a weak assertion, such as checking only that an element exists. The test can pass without proving that the business operation works.

---

## 16. AI mistakes — типичные ошибки

**Вопрос:** Какие распространённые ошибки встречаются в AI-сгенерированных Playwright тестах в отношении selectors, waits, data isolation и assertions?

**Question:** What common mistakes appear in AI-generated Playwright tests regarding selectors, waits, data isolation, and assertions?

**Ответ:** Частые ошибки — хрупкие selectors, `waitForTimeout()`, общие данные, отсутствие cleanup и слабые assertions. Например, тест проверяет наличие кнопки вместо результата операции.

**Answer:** Common mistakes include fragile selectors, `waitForTimeout()`, shared data, missing cleanup, and weak assertions. For example, a test checks that a button exists instead of checking the operation result.

---

## 17. AI fixes — проверка причины

**Вопрос:** Почему AI-предложенное исправление нужно проверять по trace и воспроизводимой причине, а не принимать только потому, что тест стал зелёным?

**Question:** Why must an AI-suggested fix be validated against a trace and reproducible cause rather than accepted because the test turns green?

**Ответ:** Зеленый тест может означать, что assertion просто стал слабее. Trace помогает убедиться, что исправлена реальная причина ошибки, а не только скрыт её симптом.

**Answer:** A green test may mean that the assertion was simply weakened. A trace helps verify that the real cause was fixed instead of hiding the symptom.

---

## 18. Artifacts — traces, screenshots и video

**Вопрос:** Зачем сохранять traces и screenshots только при failure и когда video может дать дополнительную полезную информацию?

**Question:** What is the purpose of retaining traces and screenshots only on failure, and when might video add useful evidence?

**Ответ:** Это уменьшает количество артефактов и экономит место, сохраняя диагностические данные для ошибок. Video полезно для сложных визуальных и timing-related проблем.

**Answer:** This reduces the number of artifacts and saves storage while keeping diagnostic data for failures. Video is useful for complex visual and timing-related problems.

---

## 19. Retries — диагностика flaky-тестов

**Вопрос:** Почему retries полезны для диагностики, но опасны, если используются для сокрытия flaky-тестов?

**Question:** Why can retries be useful diagnostically but dangerous as a way to hide flaky tests?

**Ответ:** Retry помогает определить, является ли ошибка нестабильной. Но постоянное прохождение со второй попытки скрывает проблему и создаёт ложное ощущение стабильности.

**Answer:** A retry helps identify whether a failure is intermittent. But consistently passing on the second attempt hides the problem and creates a false sense of stability.

---

## 20. Parallel execution — workers и fixtures

**Вопрос:** Как количество workers и параллельное выполнение должны влиять на naming и cleanup fixtures?

**Question:** How should worker count and parallel execution influence fixture naming and cleanup?

**Ответ:** Fixtures должны иметь уникальные имена или ID, чтобы workers не конфликтовали между собой. Cleanup должен удалять только данные конкретного теста или worker.

**Answer:** Fixtures should have unique names or IDs so workers do not conflict with each other. Cleanup should remove only the data belonging to the specific test or worker.

---

## 21. Smoke vs regression — наборы тестов

**Вопрос:** Что должно входить в быструю smoke suite по сравнению с полной regression suite?

**Question:** What belongs in a fast smoke suite compared with a full regression suite?

**Ответ:** Smoke suite должна проверять самые критичные сценарии: login, открытие проекта, создание Task и основные действия. Regression suite должна дополнительно покрывать permissions, edge cases, accessibility и другие функции.

**Answer:** A smoke suite should check the most critical journeys: login, opening a project, creating a Task, and key actions. A regression suite should also cover permissions, edge cases, accessibility, and other features.

---

## 22. Accessibility — automation и manual testing

**Вопрос:** Почему автоматическое accessibility scanning нужно комбинировать с keyboard tests и проверкой человеком?

**Question:** Why should automated accessibility scanning be combined with keyboard tests and human review?

**Ответ:** Автоматические инструменты находят только часть accessibility-проблем. Keyboard tests и human review помогают обнаружить проблемы взаимодействия, UX и контекста.

**Answer:** Automated tools detect only some accessibility problems. Keyboard tests and human review help find interaction, UX, and contextual problems.

---

## 23. Viewports vs visual regression — разница

**Вопрос:** Чем тестирование на разных viewport sizes отличается от screenshot-based visual regression testing?

**Question:** How does testing at multiple viewport sizes differ from screenshot-based visual regression testing?

**Ответ:** Viewport testing проверяет работу и layout на разных размерах экрана. Visual regression сравнивает screenshots с baseline и обнаруживает визуальные изменения.

**Answer:** Viewport testing checks functionality and layout at different screen sizes. Visual regression compares screenshots with baselines to detect visual changes.

---

## 24. Coverage matrix — известные gaps

**Вопрос:** Почему в финальной coverage matrix нужно явно указывать manual checks и известные пробелы вместо заявления о полном покрытии?

**Question:** Why should the final coverage matrix explicitly list manual checks and known gaps instead of claiming complete coverage?

**Ответ:** Полное автоматическое покрытие практически недостижимо. Manual checks и known gaps показывают реальные границы тестирования и предотвращают ложное ощущение полного покрытия.

**Answer:** Complete automated coverage is practically impossible. Manual checks and known gaps show the real limits of testing and prevent a false sense of complete coverage.