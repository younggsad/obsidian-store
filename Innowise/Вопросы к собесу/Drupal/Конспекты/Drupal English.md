---
custom-width: 80
---
---
# Drupal: Complete Theory Notes (DrupalJira)

A single set of notes for interview preparation. Blocks 1–15 are the main topics, blocks 16–19 are extensions (handlers, forms, changing someone else's code, settings.php), and at the end there is an appendix with basic topics and a general cheat sheet.

## Map: what is responsible for what

| Topic                            | What it is responsible for                      | Key concepts                                                  |
| -------------------------------- | ----------------------------------------------- | ------------------------------------------------------------- |
| 1. Multilingual                  | interface, content, and configuration languages | language negotiation, translatable fields, `t()`              |
| 2. REST / JSON:API               | access to data over HTTP                        | REST Resource, JSON:API, serialization, CORS                  |
| 3. Configuration Management      | transferring settings between environments      | `cex`/`cim`, sync storage, config schema, Config Split        |
| 4. Services & DI                 | reusable logic and dependencies                 | service container, `services.yml`, `create()`                 |
| 5. Performance                   | cache, queries, scaling                         | cache contexts/tags/max-age, N+1, BigPipe                     |
| 6. Testing                       | verifying behavior and access                   | Unit, Kernel, Functional, `BrowserTestBase`                   |
| 7. Security                      | access, CSRF, XSS, SQL                          | Entity Access, CSRF token, Twig escaping                      |
| 8. Multisite                     | several sites on one codebase                   | `sites/`, `settings.php`, `sites.php`                         |
| 9. Events API                    | reacting to events without tight coupling       | `event_dispatcher`, subscribers                               |
| 10. Entity API                   | data CRUD, bundles, query                       | storage, Entity Query, `accessCheck`, revisions, entity keys  |
| 11. Plugin API                   | interchangeable components                      | Plugin Manager, attributes, derivatives                       |
| 12. Batch / Cron / Queue         | long-running and background operations          | `$sandbox`, `hook_cron`, QueueWorker                          |
| 13. Migration API                | transferring data from external sources         | source/process/destination, ID map                            |
| 14. Field API                    | structure of entity data                        | field type/storage/config, widget, formatter, computed fields |
| 15. Authentication API           | who is making the request                       | Authentication Provider, Basic, OAuth, CSRF                   |
| 16. Entity Handlers              | entity roles: list, form, access, routes        | `handlers`, ListBuilder, AccessControlHandler                 |
| 17. Form API                     | forms and their processing                      | FormBase, ConfigFormBase, `#states`, AJAX                     |
| 18. Changing someone else's code | changing Core/contrib without editing it        | alter hooks, preprocess, Twig, RouteSubscriber                |
| 19. settings.php and the DB      | environment parameters and secrets              | `$databases`, `settings.local.php`, `getenv()`                |

---

## Block 1. Multilingual

> [!abstract] Summary 
> Multilingual support consists of three independent layers: interface language (system strings), content language (translations of entities and fields), and configuration language (field labels, content type names). The request language is determined by language negotiation: URL prefix, domain, user settings, the `Accept-Language` header. Some fields can be shared across all translations. The cache must take language into account.

### 1.1. General concept

- **Interface language** — buttons, messages, system labels.
- **Content language** — nodes, user entities, translatable fields.
- **Configuration language** — translatable configuration values (field labels and descriptions, block and View titles).

Core modules:

|Module|Purpose|
|---|---|
|`Language`|managing languages, determining the language context|
|`Interface Translation` (`locale`)|translating interface strings|
|`Content Translation`|translating content|
|`Configuration Translation`|translating supported configuration values|

Minimal setup: enable the four modules → add languages → configure language negotiation → enable translation for the needed entity types and bundles (_Content language and translation_) and mark the translatable fields. Enabling the modules alone does nothing without these steps.

### 1.2. Language negotiation

The mechanism that determines the request language. Sources: URL prefix (`/en/about`), domain (`en.example.com`), user preferences, `Accept-Language`, the default language. Methods and priority are configured separately for each language type, so the interface language and the content language can differ. Choosing the request language does not create a translation and does not guarantee that one exists.

```php
$lm = \Drupal::languageManager();
$ui = $lm->getCurrentLanguage(LanguageInterface::TYPE_INTERFACE)->getId();
$content = $lm->getCurrentLanguage(LanguageInterface::TYPE_CONTENT)->getId();
```

In classes, `language_manager` is injected via DI.

### 1.3. Entity translation

A translation lives at the entity and field level: these are not separate nodes, but language versions of one entity with a shared ID. Translatability is set on the field (`translatable`); the value of a non-translatable field is shared across all translations. A path alias can be different for each language.

```php
if ($node && $node->hasTranslation('en')) {
  $en = $node->getTranslation('en');
  $title = $en->label();
}
if ($node && !$node->hasTranslation('en')) {
  $node->addTranslation('en', ['title' => 'Implement task workflow']);
  $node->save();
}
```

Methods: `isTranslatable()`, `isDefaultTranslation()`, `hasTranslation()`, `getTranslation()` (does not create a translation, throws an exception if there is none), `addTranslation()`.

Do not confuse: the request language, the language of the source entity, the language of a specific translation, the translatability of a field.

### 1.4. Interface translation

Translatable strings are wrapped with `$this->t()` (`StringTranslationTrait`):

```php
$this->t('Task @title has been saved.', ['@title' => $title]);
```

Placeholders: `@value` — escaping; `%value` — escaping + emphasis (`<em>`); `:value` — a URL for `href`/`src`. Plural: `formatPlural($count, '1 task', '@count tasks')`. In Twig: `{{ 'Text'|t }}` and `{% trans %}...{% endtrans %}`.

Strings are not assembled from pieces: the translator must be able to change the word order. `t()` does not replace escaping. Calling `t()` merely marks a string as translatable; the translation itself comes from language packs, `.po` files, or the UI.

### 1.4a. Configuration translation

Not all values are translated: a field **label** is translated, a **machine name** is not. What is translatable is determined by the config schema (types `label`, `text`, etc. are translatable, `integer` is not). Do not confuse: Configuration Management transfers settings between environments, Configuration Translation translates the values of settings.

### 1.5. Language fallback

Choosing an alternative value when the needed translation is missing. The behavior depends on the data type and the API; you cannot assume that the default language will always be substituted. The absence of an entity translation and the absence of a field translation are considered separately.

### 1.6. Multilingual and cache

The rendering result depends on the language, so the cache contexts `languages:language_interface` and `languages:language_content` are needed. They are chosen by actual dependency rather than adding all of them. A context is added in `#cache` or bubbles up from nested elements.

> [!example] Practice
> Enable the four modules, configure negotiation, mark the translatable bundles and fields. In code, all visible strings go through `t()`, and language-dependent components get language cache contexts. Values such as statuses are stored as machine values (`in_progress`, `done`), and the labels are translated.

> [!warning] Common mistakes
> 
> - Mixing up the interface language and the content language.
> - Not checking `hasTranslation()` before `getTranslation()`.
> - Expecting separate values from a non-translatable field.
> - Leaving messages without `t()` or concatenating translatable strings.
> - Forgetting language cache contexts.
> - Translating machine names.
> - Creating separate independent nodes instead of translating one entity.
> - Assuming that interface translation will translate the content.

> [!question] Interview questions 
> **Interface language vs. content language?** The first is the language of system strings and the UI, the second is the language of entity content. They are determined separately and can differ.
> 
> **What happens on `getTranslation()` for a non-existent translation?** An exception; the translation is not created. First `hasTranslation()`, and to create one `addTranslation()`.
> 
> **Translatable vs. non-translatable field?** Translatability is set at the field level. The value of a non-translatable field is shared across all translations (external ID, date).
> 
> **Configuration Translation vs. Configuration Management?** The first translates the values of settings, the second transfers settings between environments.
> 
> **Why language cache contexts?** So that a user does not get a page in another language from the cache.

---

## Block 2. REST API and JSON:API

> [!abstract] Summary 
> REST in Drupal is not just "return JSON". There are three paths: the core REST module with resource plugins, JSON:API for standard entity operations, and custom routes with controllers. In every case you need authentication, authorization, validation, correct HTTP codes, and cache metadata.

### 2.1. Resources and HTTP

REST is a style where the client works with resources over HTTP (for example, a React frontend fetches tasks and changes their status).

|Method|Purpose|
|---|---|
|`GET`|retrieve a resource|
|`POST`|create a resource or perform an operation|
|`PATCH`|partially modify|
|`PUT`|create or fully replace|
|`DELETE`|delete|

|Code|Meaning|
|---|---|
|200 / 201 / 204|success / created / no body|
|400|bad request|
|401|no suitable authentication|
|403|access denied|
|404|not found or not disclosed to the client|
|405|method not supported|
|409|state conflict|
|422|validation failed|

### 2.2. Ways to build an API

- **Core REST module** (REST Resource plugins).
- **JSON:API** (core) — a separate implementation of the specification, not to be confused with REST Resource.
- **Custom routes** with `JsonResponse` / `CacheableJsonResponse`.
- **Views REST export** — a list in JSON/XML without code.
- contrib: GraphQL.

REST infrastructure: Resource plugins, Serialization API, Authentication API, Access API.

### 2.3. JSON:API

```http
GET /jsonapi/node/task?filter[field_status]=in_progress&include=field_project&page[limit]=20
Accept: application/vnd.api+json
```

A resource is identified by **UUID**, not a numeric ID. Response: `type` (`node--task`), `id`, `attributes`, `relationships`. Filter support depends on the fields. An enabled JSON:API allows writing by default: for a read-only API it is configured separately. Permissions and access to related entities are checked as usual.

### 2.4. REST Resource plugin

The plugin extends `ResourceBase`:

```php
#[RestResource(id: 'example_tasks', label: new TranslatableMarkup('Example tasks'),
  uri_paths: ['canonical' => '/api/tasks'])]
final class TasksResource extends ResourceBase {
  public function get(): ResourceResponse {
    return new ResourceResponse(['data' => [['id' => 1, 'title' => 'Prepare notes']]]);
  }
}
```

(in older versions an `@RestResource` annotation is used instead of the attribute). The class does not open the endpoint by itself: the resource must be **enabled** via `rest.resource.*` configuration (methods, formats, authentication) and permissions of the form `restful get example_tasks` must be granted. A `ResourceResponse` is cacheable: add `addCacheableDependency()`.

### 2.5. REST Resource vs. JSON:API

|REST Resource|JSON:API|
|---|---|
|custom plugins|standardized entity API|
|the structure is defined by the developer|structure per the specification|
|for special operations|for typical entity operations|

For standard data, evaluate JSON:API first; for a special business operation — a REST Resource or a custom route (the controller coordinates the request/response, calculations go in a service).

### 2.6. Serialization

- **Normalization** — object → array.
- **Encoding** — array → format (JSON, XML).

Order: request → route → authentication and access → processing → serialization → response.

### 2.7. Authentication, authorization, CSRF

Authentication — who, Authorization — is it allowed for them, CSRF — protection against request forgery through the session. With cookie authentication, the token is obtained from `/session/token` and passed in `X-CSRF-Token`. HTTPS does not replace authorization and CSRF.

### 2.8. Validation and errors

Check the format and types, required fields, allowed values (the status must be an allowed one and the transition permitted), existence of related entities, permissions, business constraints. Errors are returned in a consistent format, without stack traces and secrets.

### 2.9. Caching and CORS

Account for query parameters, headers, language, permissions, personal data. Personalized content is not put into a shared cache without contexts. For a frontend on another domain — `cors.config` in `services.yml` (`enabled`, `allowedOrigins`, `allowedMethods`, `allowedHeaders`, `supportsCredentials`). CORS does not replace authentication and CSRF.

> [!example] Practice 
> For standard data take JSON:API (core, specification, filters, include, pagination). For a special operation — a REST Resource or a route. Lists without code — Views REST export. For a separate frontend: authentication, access checks for each entity, CORS, CSRF for cookies. DrupalJira example: `/jsonapi/node/task` with a filter for a React client.

> [!warning] Common mistakes
> 
> - Operations without access checks; considering authentication sufficient.
> - Ignoring CSRF with cookies.
> - Not validating input.
> - Confusing REST Resource and JSON:API.
> - Exposing internal errors.
> - Using `GET` to change data.
> - Caching personal responses without contexts.
> - Assuming that a route with `_entity_access: node.view` will check the bundle itself.

> [!question] Interview questions 
> **JSON:API vs. the REST module?** JSON:API is a ready-made standardized entity API; REST is built on plugins, and the structure and logic are set by the developer.
> 
> **What is needed for a custom REST Resource to work?** The plugin, an enabled resource `rest.resource.*`, and granted permissions.
> 
> **When 401, 403, 404?** 401 — no suitable authentication, 403 — the user is known but access is denied, 404 — the resource does not exist or is hidden.
> 
> **How to protect cookie requests from CSRF?** The token from `/session/token` in the `X-CSRF-Token` header, plus mandatory permission checks.
> 
> **What identifies an entity in JSON:API?** UUID.

---

## Block 3. Configuration Management

> [!abstract] Summary 
> Settings and the structure of the site are stored in YAML and transferred between environments via Git. Configuration includes content types, fields, Views, roles, and permissions. Content (tasks, projects, time records) is not included. Secrets and infrastructure parameters are set separately on each environment.

### 3.1. Configuration, content, code

- **Code** — PHP, Twig, CSS, JS.
- **Configuration** — content types, fields, Views, roles, workflows, module and display settings.
- **Content** — an article, a task, a project, a time record.

|What|Where|Transferred via CM?|
|---|---|---|
|Config|active config in the DB + YAML|yes|
|Content|entity tables|no (migrations, `default_content`, DB copy)|
|State|the `state` service (time of the last cron)|no|
|Settings|`settings.php`, secrets, paths|no, set on the environment|

Classification depends on the entity type: a taxonomy term is content, a vocabulary is configuration.

### 3.2. Active configuration and sync storage

**Active configuration** — the settings of the running site (the `config` table). **Sync storage** — a directory of YAML files (`$settings['config_sync_directory']`), usually in Git. Process: change → export → check the YAML → commit → deploy → import.

File names: `system.site.yml`, `node.type.task.yml`, `field.storage.node.field_status.yml`, `field.field.node.task.field_status.yml`, `core.extension.yml`, `views.view.*.yml`.

### 3.3. Drush

```bash
drush config:export   # cex
drush config:import   # cim
drush config:status   # cst: differences between active and sync
drush config:get system.site
drush config:set system.site name 'DrupalJira'
drush cim --preview=diff
```

Deployment: `drush updb` → `drush cim` → `drush cr` (or `drush deploy`). Before importing, look at the differences: an import creates, changes, and **deletes** objects.

### 3.4. Machine name, label, UUID

The machine name (`field_status`) is used in code and dependencies, the label (`Status`) is the display name, the UUID is a unique identifier. The **site** UUID in sync must match the target, otherwise a full import is rejected (the sites must originate from the same installation; usually an environment is created from a DB copy).

### 3.5. Config API: Config, Immutable, Editable

- `config.factory` (`ConfigFactoryInterface`): `get('name')` → `ImmutableConfig` (read, taking settings.php overrides into account); `getEditable('name')` → a mutable object without overrides.
- Methods: `get('key')`, `set('key', $v)`, `save()`, `clear('key')`, `delete()`.
- In `ConfigFormBase`/`ControllerBase` the short form `$this->config('name')`.

```php
$limit = $factory->get('mymodule.settings')->get('hours_limit');
$factory->getEditable('mymodule.settings')->set('hours_limit', 8)->save();
```

### 3.6. Simple config and config entities

- **Simple configuration** — a single record (`mymodule.settings`), schema `config_object`.
- **Configuration entity** — many records of the same kind (content types, roles, views), class `ConfigEntityBase`, name `module.type.id`.

### 3.7. config schema

Describes the structure and types of values: helps with validation, translation, and correct export. A missing or incorrect schema produces schema errors in tests.

```yaml
# config/schema/example.schema.yml
example.settings:
  type: config_object
  label: 'Example settings'
  mapping:
    api_endpoint: { type: uri, label: 'API endpoint' }
    timeout: { type: integer, label: 'Timeout' }
    enabled: { type: boolean, label: 'Enabled' }
```

### 3.8. `config/install` and `config/optional`

- `config/install` — installed when the module is installed; if dependencies are not met, installation will fail.
- `config/optional` — installed only if all dependencies are present.

A file in `config/install` **does not update** already existing sites. For them you need a configuration import or an update hook.

### 3.9. Configuration dependencies

In YAML there is a `dependencies` block (`config`, `module`, `theme`, `content`). For a config entity they are calculated automatically (`calculateDependencies()`), hard ones are set in `enforced`. Drupal imports in the correct order and does not let you delete something that other things depend on.

```yaml
dependencies:
  config: [field.storage.node.field_status, node.type.task]
  module: [mymodule]
```

Check: whether modules are enabled, whether objects exist, whether machine names and schemas are correct, whether an object that is referenced is being deleted.

### 3.10. CM and update hooks

CM is declarative transfer of settings. An update hook is a programmatic transformation of an existing installation (data, structure). New structure on old sites: `hook_update_N()` + `drush cex` + commit.

```php
function example_update_10001(): void {
  \Drupal::configFactory()->getEditable('example.settings')->set('enabled', TRUE)->save();
}
```

### 3.11. Environments, overrides, Config Split

Application settings go into configuration, secrets and infrastructure parameters go into the environment (`settings.php`, environment variables, secret management).

- **Overrides**: `$config['system.site']['name'] = 'Dev';` — on-the-fly substitution, does not end up in the export.
- **Config Split** — different sets of configuration per environment (dev modules only locally).
- **Config Ignore** — excludes objects from import/export.
- `$settings['config_exclude_modules']` — exclude modules from synchronization.

### 3.12. Why import does not transfer content

The following are not included: nodes, users, comments, terms, files, `time_log` records. Content is transferred by migrations, `default_content`, or a DB copy. Conversely: if you delete a content type from configuration, the existing records are deleted separately.

### 3.13. Diagnosing errors

`drush config:status`, `drush cim --preview=diff`, the page `/admin/config/development/configuration`, logs `/admin/reports/dblog`. Common causes: different site UUID, a module from `core.extension` is not enabled, a record's dependency is broken, a record does not match the schema, a merge conflict in Git, `updb` was not run, part of the settings is excluded from synchronization.

> [!example] Practice 
> Change locally → `drush cex` → YAML in Git → on another environment deploy the code, `updb`, `cim`, `cr`. Before the import, `cst`. In DrupalJira the Project and Task types, fields, roles, and Views go into Git; the tasks themselves are content and are not included in the export.

> [!warning] Common mistakes
> 
> - Changing configuration on production without transferring it to Git.
> - Assuming that `config/install` updates existing sites.
> - Ignoring dependencies, deleting objects without checking references.
> - Confusing content with configuration.
> - Replacing export/import with update hooks without a reason.
> - Importing without `drush cst`.
> - Storing secrets in the repository (and in `config/sync`).
> - Not adding an update hook when existing data requires transformation.

> [!question] Interview questions 
> **Configuration vs. content?** Configuration describes the structure and settings of the site and is transferred via YAML; content is working data and is not transferred via CM.
> 
> **The site UUID does not match?** A full import is rejected: the sites must originate from the same installation.
> 
> **Different settings for environments?** `$config` overrides and secrets in `settings.php`, Config Split.
> 
> **Will `config/install` update existing sites?** No, only when the module is installed. An import or an update hook is needed.
> 
> **`get()` vs. `getEditable()`?** The first returns an immutable object taking overrides into account, the second returns a mutable one without overrides, for writing.

---

## Block 4. Services and Dependency Injection

> [!abstract] Summary 
> A service is an object with a defined task, registered in the service container. DI means a class receives its dependencies from outside (usually through the constructor) rather than creating them itself. The code becomes reusable, loosely coupled, and testable.

### 4.1. Service and container

Examples of services: `entity_type.manager`, `current_user`, `logger.factory`, `database`, `cache.default`, `language_manager`, `config.factory`, `http_client`, `request_stack`, `event_dispatcher`, `queue`.

The Service Container (Symfony DependencyInjection) stores definitions, creates objects, and injects dependencies. Drupal collects all `MODULE.services.yml` files, builds the container, and caches it; a service is created on first access, usually once per container lifetime (but not every service is a global singleton). After editing `services.yml`, `drush cr` is needed.

### 4.2. DI

Without DI: `\Drupal::entityTypeManager()->getStorage('node')` — a hidden dependency on global state, hard to test. With DI:

```php
final class TaskManager {
  public function __construct(private readonly EntityTypeManagerInterface $entityTypeManager) {}
  public function loadTask(int $id): ?NodeInterface {
    $e = $this->entityTypeManager->getStorage('node')->load($id);
    return $e instanceof NodeInterface && $e->bundle() === 'task' ? $e : NULL;
  }
}
```

It is better to specify a dependency by **interface** rather than by a concrete class: the implementation can be replaced (a decorator, a mock in a test).

### 4.3. Registering a service

```yaml
services:
  example.task_manager:
    class: Drupal\example\TaskManager
    arguments: ['@entity_type.manager']
```

`@` is a reference to a service. Additionally: `tags` (`event_subscriber`, `access_check`, `normalizer`, `authentication_provider`, `cache.bin`), `calls` (call a method after creation), `decorates` (replacing/wrapping a service), `parent`, `lazy`, `public`, `autowire`.

```yaml
  example.task_manager.logged:
    class: Drupal\example\LoggedTaskManager
    decorates: example.task_manager
    arguments: ['@example.task_manager.logged.inner', '@logger.factory']
```

Autowiring in Drupal depends on the version and requires interface aliases; explicit registration is preferable.

### 4.4. How different classes get services

|Where|How|
|---|---|
|another service|`arguments` in `services.yml`|
|controller, form|`create(ContainerInterface $container)` + constructor|
|plugin|`ContainerFactoryPluginInterface`, `create($container, $configuration, $plugin_id, $plugin_definition)`|
|entity handler|`createInstance($container, $entity_type)`|

Plugins are created by the Plugin Manager, so you cannot simply add an argument to the constructor.

```php
final class TaskController extends ControllerBase {
  public function __construct(private readonly EntityTypeManagerInterface $entityTypeManager) {}
  public static function create(ContainerInterface $container): static {
    return new static($container->get('entity_type.manager'));
  }
}
```

### 4.5. `\Drupal::service()`

Acceptable where DI is unavailable: `.module`, `.install`, procedural hooks, update hooks. In classes (services, forms, controllers, plugins) it is avoided: dependencies are hidden, testing is harder.

### 4.6. Service, plugin, handler, plain class

- **Service** — a single object with a permanent role, retrieved by name.
- **Plugin** — one of many interchangeable variants, selected by id.
- **Handler** — the performer of one role for an entity type.
- **Plain class** — `new`, for data and utilities without dependencies.

### 4.7. Testability

Unit test: dependencies are replaced with mock/stub in the constructor. But to verify real integration with Drupal, Kernel/Functional is better. If a class itself calls `\Drupal::service()`, it is hard to isolate.

### 4.8. When a service is not needed

A service is justified when the logic is used in several places, has its own responsibility, requires DI or isolated testing, or is a separate business operation. For a small local operation it is excessive.

> [!example] Practice 
> Business logic (statistics calculation) is extracted into a service, described in `services.yml`, and used in a controller, form, block, Drush command. In DrupalJira: `TaskStatService` is used by a block, a controller, and an update hook.

> [!warning] Common mistakes
> 
> - `\Drupal::service()` everywhere.
> - "God" services.
> - Forgetting to register a service or clear the cache.
> - The order/number of `arguments` does not match the constructor, a typo in the service name.
> - Incorrect injection into plugins.
> - Depending on concrete classes instead of interfaces.
> - Assuming that all Drupal objects are created the same way.

> [!question] Interview questions 
> **What is a service container?** A Symfony DI component: it stores definitions, creates objects, injects dependencies.
> 
> **Why is DI better than `\Drupal::service()`?** Dependencies are explicit, coupling is lower, and they can be replaced in tests.
> 
> **How to inject a service into a plugin?** `ContainerFactoryPluginInterface` and `create()`, because the plugin is created by the manager, not by the container.
> 
> **Tags and decorators?** A tag marks a service for discovery by a subsystem (`event_subscriber`), a decorator replaces or extends a service without editing its code.
> 
> **Where is `\Drupal::service()` acceptable?** In procedural code: `.module`, `.install`, hooks, update hooks.

---

## Block 5. Performance and Scaling

> [!abstract] Summary 
> Performance is speed under a given load, scalability is the ability to grow along with the load. The foundation is caching: cache contexts (what the result depends on), cache tags (which data it depends on), max-age (how long it lives). The second group of problems is excess DB queries (N+1) and heavy work inside the HTTP request. Start with measurements.

### 5.1. Sources of problems and measurement

Excessive DB queries, repeated entity loading, incorrect caching, heavy operations on every request, large render arrays, external APIs, bad indexes, long background operations run synchronously. Tools: Webprofiler (Devel), Xdebug, Blackfire/XHProf, slow SQL log, APM. Profiling is not enabled on production without understanding the load.

### 5.2. Cache levels

- **Internal Page Cache** — whole responses for anonymous requests.
- **Dynamic Page Cache** — pages with dynamic parts, takes contexts into account, useful for authenticated users too.
- **Render Cache** — render results of components.

For data outside the render system — the Cache API (`cache.default`, `cache.data`, custom cache bins). The cache backend (DB, Redis) determines storage but does not fix incorrect metadata.

### 5.3. Cache contexts

Determine under what conditions a separate variant of the result is needed: `user`, `user.roles`, `user.permissions`, `languages:language_interface`, `languages:language_content`, `url.path`, `url.query_args`.

```php
return ['#markup' => $message,
  '#cache' => ['contexts' => ['user.roles', 'languages:language_interface']]];
```

Extra contexts multiply cache variants and reduce cache efficiency.

### 5.4. Cache tags

Describe which data the result depends on; when an entity changes, Drupal invalidates the cache by its tags.

```php
'#cache' => ['tags' => $task->getCacheTags()]   // node:123
```

For lists — `node_list`, `node_list:task`, or custom ones. Tags are responsible for dependence on data, not for variation by user or language.

### 5.5. Max-age

Lifetime in seconds. `0` — do not cache; `Cache::PERMANENT` — unlimited (tag invalidation remains). Note: `max-age: 0` in a nested element bubbles up and disables the cache of the whole page.

|Mechanism|Question|
|---|---|
|contexts|for whom or under what conditions is a separate variant needed?|
|tags|which data changes make the result stale?|
|max-age|how long is the result valid?|

### 5.6. Cache bubbling

The metadata of nested elements passes up to the parents: contexts and tags are merged, the smallest max-age is taken. Metadata is set at the component level rather than recalculated manually. Access objects also carry cacheability (`AccessResult::allowedIf(...)->addCacheContexts([...])`).

### 5.7. N+1 and entity loading

N+1: a list of entities, then a separate query for each one (100 tasks and a separate project load for each). Solutions: Entity Query for IDs, `loadMultiple()` instead of a `load()` loop, preloading related entities, Views, caching reused data, aggregating SQL for statistics (sums of hours). For counting use `->count()` rather than loading all entities. Real SQL is measured: not every sequence of calls is a problem (the storage has a static cache).

### 5.8. Indexes

They speed up filtering, sorting, joins; but they take up space, slow down writes, and do not guarantee a speedup. Analyze `EXPLAIN` rather than adding them at random.

### 5.9. BigPipe, placeholders, lazy builders

BigPipe sends part of the HTML earlier and delivers the dynamic parts later. Placeholders defer the rendering of elements. Personalized parts use lazy builders:

```php
$build['summary'] = [
  '#lazy_builder' => ['drupaljira_timelog.task_stat:buildSummary', [$task_id]],
  '#create_placeholder' => TRUE,
];
```

The callback must be trusted (`TrustedCallbackInterface`). This does not replace caching and query optimization.

### 5.10. Infrastructure

OPcache (PHP bytecode), Redis/Memcache (external cache), CDN, reverse proxy (Varnish, Nginx). Mere presence does not speed things up without correct configuration and invalidation (tags → purge). On production: CSS/JS aggregation, image styles, dev modules disabled.

### 5.11. Scaling

Vertical — more resources on a single server. Horizontal — several instances: a shared DB, shared files (shared storage/S3), shared caches and locks, consistent configuration, cron and queues run once, session storage, limits of the DB and external APIs. Queues remove heavy work from the request, but the work still has to be done.

> [!example] Practice 
> Correct metadata in render arrays: contexts for variations, tags for invalidation, max-age for time limits. Slow personalized parts — lazy builders and BigPipe. For queries: `loadMultiple`, aggregates, indexes based on `EXPLAIN`, heavy work into a queue. The DrupalJira project page calculates the sum of hours in one query instead of loading records one by one.

> [!warning] Common mistakes
> 
> - Disabling the cache instead of fixing the configuration.
> - Forgetting tags and contexts; `max-age` as the only invalidation.
> - N+1; indexes without analysis.
> - Heavy processing in the HTTP request.
> - Scaling PHP while ignoring the DB.
> - Caching personal data without contexts.
> - Heavy Views without pagination and indexes.

> [!question] Interview questions 
> **What does cacheability consist of?** Contexts, tags, max-age.
> 
> **What is cache bubbling?** The passing of cache metadata from nested elements to parents. This is why `max-age: 0` in one element disables the cache of the whole page.
> 
> **How to avoid N+1?** Batch loading (`loadMultiple`), related data in one query or Views, aggregating queries.
> 
> **Internal Page Cache vs. Dynamic Page Cache?** The first caches whole responses for anonymous users, the second caches pages with dynamic parts taking contexts into account, including for authenticated users.
> 
> **How does `Cache::PERMANENT` differ from "never updated"?** There is no time limit, but tag invalidation works.

---

## Block 6. Testing

> [!abstract] Summary 
> Four levels: Unit (isolated logic), Kernel (core and a test DB), Functional (HTTP, `BrowserTestBase`), FunctionalJavascript (a browser with JS). Test at the simplest level that confirms correctness. Always test access permissions, not just the success scenario.

### 6.1. Why

Modules are tightly coupled to the Entity API, services, routing, configuration, access, rendering. Tests catch business logic errors, access violations, DB and configuration problems, route and form errors, regressions.

### 6.2. Types of tests

|Type|Base|What it checks|
|---|---|---|
|Unit|`UnitTestCase`|logic without Drupal, dependencies are mocks/stubs|
|Kernel|`KernelTestBase`|services, entities, configuration, hooks on a test DB|
|Functional|`BrowserTestBase`|routes, forms, permissions via HTTP|
|FunctionalJavascript|`WebDriverTestBase`|behavior that depends on client-side JS|
|E2E (Playwright, Cypress/Nightwatch)|external|end-to-end scenarios|

Kernel tests require modules and schemas to be connected explicitly:

```php
protected static $modules = ['system', 'user', 'node', 'field', 'text', 'drupaljira_timelog'];
protected function setUp(): void {
  parent::setUp();
  $this->installEntitySchema('user');
  $this->installEntitySchema('node');
  $this->installConfig(['node', 'drupaljira_timelog']);
}
```

Functional: `$modules` and the mandatory `$defaultTheme` (for example, `stark`).

```php
public function testAnonymousCannotEdit(): void {
  $this->drupalGet('/node/1/edit');
  $this->assertSession()->statusCodeEquals(403);
}
```

### 6.3. The pyramid

Many fast unit tests, enough kernel tests for integration, functional tests for important scenarios, few heavy browser tests. This is a guideline, not a hard rule.

### 6.4. Mocks and stubs

A mock sets expected calls and verifies interaction, a stub returns predefined values. An excess of mocks produces tests of implementation details. For real integration with the Drupal API — kernel/functional.

### 6.5. Testing access

Check both the allowed and the denied user, as well as route access and entity access. For DrupalJira: an administrator edits; a project manager does what is permitted; a regular user does not change other people's tasks; an anonymous user gets nothing restricted; an inaccessible entity is not revealed via the API.

### 6.6. Test data

Created in the test itself; deterministic, independent of the state of the environment, minimal, isolated between tests.

### 6.7. Regressions

Especially when changing permissions and access handlers, workflow, entity definitions, configuration, API, migrations, caching. Run in CI before merging. Migration tests check the number of records, transformations, relations, re-running; queue tests check idempotency and the absence of duplicated side effects.

### 6.8. Running and static analysis

```bash
./vendor/bin/phpunit -c web/core web/modules/custom/drupaljira_timelog/tests
./vendor/bin/phpunit -c web/core --filter testLoggedHours web/modules/custom/drupaljira_timelog
```

You need `phpunit.xml` and the variables `SIMPLETEST_DB`, `SIMPLETEST_BASE_URL`. Additionally: PHPCS/PHPCBF (the `Drupal`/`DrupalPractice` standard), PHPStan (`phpstan-drupal`).

> [!example] Practice 
> Unit for logic without Drupal, Kernel for services and entities, Functional for routes, forms, and permissions, FunctionalJavascript only when JS is needed. Data is created in the test, code style and analysis run in CI. For DrupalJira statistics a Kernel test fits, for access to time records — a Functional one.

> [!warning] Common mistakes
> 
> - Only the success scenario.
> - Too many mocks.
> - Tests that depend on the data of the site.
> - Not checking access permissions.
> - A Functional test for every trifle.
> - Ignoring flaky tests.
> - Testing the implementation rather than the behavior.
> - Not running tests after changing configuration or schema.

> [!question] Interview questions 
> **What types of tests are there?** Unit, Kernel, Functional (`BrowserTestBase`), FunctionalJavascript. Choose the simplest suitable level.
> 
> **Kernel vs. Functional?** Kernel boots the core and the DB without a browser, modules and schemas are connected manually; Functional works over HTTP on a fully installed site.
> 
> **How to test access?** Both the allowed and the denied user, an anonymous user, someone else's entity; route access and entity access.
> 
> **Why `$modules` and `$defaultTheme`?** `$modules` — the modules enabled for the test, `$defaultTheme` is required for `BrowserTestBase`.

---

## Block 7. Security

> [!abstract] Summary 
> Security is multi-layered: authentication, authorization, access control, input validation, output escaping, CSRF, protection against SQL injection. Do not trust data and check access on the server, not in the interface. A permission does not mean access to any entity: access to a specific task is checked separately.

### 7.1. Principles

Authentication — who; Authorization — is the action allowed; Access control — access to entities, routes, operations; Validation — checking input; Output escaping — protection against XSS; CSRF; SQL injection protection. Valid data does not mean the right to change it, and vice versa.

### 7.2. Permissions and roles

A permission is an allowed action, a role is a set of permissions, a user can have several roles. In DrupalJira: Project Manager, Authenticated User, Administrator.

```yaml
# drupaljira_timelog.permissions.yml
manage project tasks:
  title: 'Manage project tasks'
  restrict access: true
```

Declaring does not grant the permission: roles are assigned it separately. Check: `$account->hasPermission('manage project tasks')` (in classes — via DI `current_user`).

### 7.3. Route access

```yaml
requirements:
  _permission: 'access content'     # or _role, _user_is_logged_in, _entity_access: 'node.update', _custom_access, _csrf_token
```

Complex checks — `_custom_access` or a service with the `access_check` tag. A hidden button is not protection: the check is always on the server.

### 7.4. Entity access (Entity Access API)

```php
$access = $node->access('update', NULL, TRUE);
if ($access->isAllowed()) { /* ... */ }
```

The third argument `TRUE` returns an `AccessResultInterface` with cacheability. The result is three-valued: Allowed, Forbidden, Neutral; Neutral is not permission. Access is checked separately for each operation (`view`, `update`, `delete`). Data queries take `accessCheck` into account. If access depends on the user, roles, or the state of the entity, this is reflected in the cache contexts of the result (`cachePerPermissions()`, `cachePerUser()`).

### 7.5. CSRF

Form API protects forms automatically (form token). For your own state-changing route:

```yaml
requirements:
  _entity_access: 'node.update'
  _csrf_token: 'TRUE'
methods: [POST]
```

The token is expected in the `token` parameter; for REST/JSON:API with cookies — `/session/token` and the `X-CSRF-Token` header. A CSRF token does not replace permission checks. State-changing actions are not done via `GET`.

### 7.6. XSS

Twig escapes automatically; `|raw` — only for trusted HTML. In PHP: `Html::escape()`. In the Render API: `#plain_text` is fully escaped, `#markup` is filtered against a whitelist (`Xss::filterAdmin`). User HTML goes through text formats with filters. Filtering is not escaping: escaping makes data safe for the output context.

### 7.7. SQL injection

The Database API with parameters:

```php
$uid = $connection->select('users_field_data', 'u')->fields('u', ['uid'])
  ->condition('name', $username)->execute()->fetchField();
// or $connection->query('... WHERE name = :name', [':name' => $username]);
```

For entities the Entity API is better. Dynamic table and column names require separate control; safe SQL does not replace checking access to the data.

### 7.8. Validation

Levels: field and entity constraints (Entity Validation API, constraints `NotBlank`, `Range`, `Length`), form validation, checks in services, validation of API request structure. Client-side checking is a convenience, server-side is mandatory. Violations are shown to the user or returned in the API (422).

### 7.9. Files

Check extensions and size, use safe schemes (public/private), do not trust the name and MIME type from the client, do not execute uploaded content, check access when serving a file (the private scheme does not check permissions by itself).

### 7.10. Practices

Least privilege; access for each entity and operation; do not trust URLs, forms, API, JS; Entity and Database API instead of manual SQL; HTTPS and `trusted_host_patterns`; secrets and `hash_salt` outside the repository; do not leave debug in production; update core and contrib (`composer audit`, security advisories); flood control; tests for allowed and denied cases.

### 7.11. Example: changing the status of someone else's task

A safe implementation of `/task/123/status` checks: the method (`POST`), the CSRF token, authorization, the right to change **this particular** task, the validity of the new status and transition, saving through the Drupal API. A request is not safe just because the button is visible to the manager.

> [!example] Practice 
> Standard APIs: Twig, Render API, Entity and Database API with parameters, Form API with tokens, route requirements, Entity Access API. Follow updates, grant the minimum of permissions, test denied scenarios. Changing a DrupalJira task status goes through CSRF, entity access, and a workflow check.

> [!warning] Common mistakes
> 
> - Hiding a button instead of a server-side check.
> - Checking only the permission, not access to the entity.
> - `|raw` and unchecked `#markup` for user input.
> - Concatenation in SQL.
> - Ignoring CSRF; extra permissions "for convenience".
> - Secrets and debug output in production.
> - Treating Neutral as permission.

> [!question] Interview questions 
> **How does Drupal protect against XSS?** Twig escapes automatically, `#plain_text` is fully escaped, `#markup` is filtered by a tag list. `raw` — only for trusted HTML.
> 
> **How to work with the DB safely?** The Entity API or the Database API with parameters, without substituting values into the SQL string.
> 
> **How to protect a route from CSRF?** `_csrf_token: 'TRUE'` and a method restriction; permissions are checked separately.
> 
> **Permission vs. entity access?** A permission is a general right for a type of action; entity access decides for a specific entity and depends on the owner, state, and other conditions.
> 
> **What does Neutral in AccessResult mean?** The check gave neither permission nor denial; it is not considered permission.

---

## Block 8. Multisite

> [!abstract] Summary 
> Several sites on one codebase: shared core, modules, and themes. Each has its own directory in `sites/`, its own `settings.php`, usually its own DB, configuration, and files. Convenient for similar sites, but an error or update in the shared code affects all the sites.

### 8.1. Structure

```text
web/
  core/  modules/  themes/
  sites/
    sites.php                 # mapping of domains to directories (optional)
    default/settings.php
    docs.example.com/settings.php
    support.example.com/settings.php
```

Drupal chooses a directory by the domain, port, and path of the request; if there is no suitable one — `sites/default`. If the directory name does not match the domain: `$sites['docs.localhost'] = 'docs.example.com';` in `sites.php`. Multisite does not automatically mean a shared DB or shared users.

### 8.2. Shared and separate

|Component|How|
|---|---|
|core, module and theme files|usually shared|
|configuration, content (with different DBs)|separate|
|users|depends on storage and connection|
|files|`sites/<site>/files`|
|cache|depends on infrastructure|

### 8.3. Setup

Install the code → create the directories → configure `settings.php` and the DB connection → point the domains at the shared document root → install each site. Drush: `drush --uri=docs.example.com status|cr|cim`. Cron, queues, and `updb` are run **separately for each site**; each has its own sync directory (`config_sync_directory`).

### 8.4. Pros and risks

Pros: one codebase, less duplication, centralized updates. Risks: an update affects all sites, compatibility must be tested on each, requirement conflicts, separate backups, a large blast radius, data isolation depends on configuration.

### 8.5. When to choose it

When the sites are similar and maintained together. If you need independent updates, different module versions, or processes — separate installations (possibly with a shared Composer template). Alternative: a single site with the contrib module **Domain** (several domains, one DB).

> [!example] Practice 
> A directory in `sites/` with `settings.php`, an entry in `sites.php` if needed, Drush with `--uri`. Cron, `updb`, and configuration import per site separately.

> [!warning] Common mistakes
> 
> - Assuming the DB and users are shared.
> - Updating shared code without tests on each site.
> - Forgetting `updb`/cron for each site.
> - One sync directory for different sites.
> - `settings.php` secrets in the repository.
> - Multisite for sites with different life cycles.

> [!question] Interview questions 
> **How does Drupal choose a site?** It matches the domain and path to a directory in `sites/` (the name or an entry in `sites.php`) and loads its `settings.php`.
> 
> **What is shared, what is separate?** Shared: core, modules, themes; separate: `settings.php`, DB, configuration, files.
> 
> **The main risks?** Shared code affects all sites; test each one, `updb` and cron separately.
> 
> **Drush command for a specific site?** `drush --uri=docs.example.com cr`.

---

## Block 9. Events API

> [!abstract] Summary 
> One component reports an event, others react without knowing about each other. An event is an object dispatched via `event_dispatcher`; handlers are event subscribers, services with the `event_subscriber` tag. A subscriber declares events and priorities in `getSubscribedEvents()`. This reduces coupling.

### 9.1. Concepts

- **Event** — an object with data about the event.
- **Event dispatcher** — sends the event to handlers (Symfony EventDispatcher).
- **Event subscriber** — a class subscribed to events.
- **Event listener** — a handler of a specific event.
- **Event name** — an identifier (a string or a class).

Core events: `KernelEvents::REQUEST`, `RESPONSE`, `EXCEPTION`, `TERMINATE`, `VIEW`, `ConfigEvents::SAVE`, `RoutingEvents::ALTER`. List: `drush debug:event`. Not everything in Drupal has a Symfony event: the entity life cycle is more often intercepted by hooks and entity methods.

### 9.2. Subscriber

```php
final class ExampleSubscriber implements EventSubscriberInterface {
  public static function getSubscribedEvents(): array {
    return [KernelEvents::REQUEST => ['onRequest', 30]];   // higher priority — earlier
  }
  public function onRequest(RequestEvent $event): void { /* ... */ }
}
```

The value is a method name, a `[method, priority]` pair, or a list of pairs. An event can be stopped: `$event->stopPropagation()`.

```yaml
services:
  drupaljira_timelog.example_subscriber:
    class: Drupal\drupaljira_timelog\EventSubscriber\ExampleSubscriber
    arguments: ['@current_user']
    tags: [{ name: event_subscriber }]
```

After adding — `drush cr`.

### 9.3. Custom event

```php
final class TaskEvents { public const STATUS_CHANGED = 'drupaljira_timelog.task_status_changed'; }

final class TaskStatusChangedEvent extends Event {
  public function __construct(private readonly NodeInterface $task,
    private readonly string $oldStatus, private readonly string $newStatus) {}
  // getters getTask(), getOldStatus(), getNewStatus()
}

// dispatching (EventDispatcherInterface via DI), after the actual save
$task->set('field_status', $new)->save();
$this->eventDispatcher->dispatch(new TaskStatusChangedEvent($task, $old, $new), TaskEvents::STATUS_CHANGED);
```

The name in `dispatch()` must match the key in `getSubscribedEvents()`. The event is not dispatched before the change has actually happened.

### 9.4. Events vs. Hooks

|Events API|Hooks|
|---|---|
|event dispatching, an event object|named Drupal extension points|
|subscriber services with a tag|`hook_*` functions or OOP hooks `#[Hook]` (Drupal 11.1+)|
|suited to event-driven architecture|widely used in the Drupal API|

Do not replace all hooks with events: use what is provided at the given extension point.

### 9.5. Events and Queue

A dispatcher call is synchronous: a slow subscriber delays the request. Scheme: save the status → dispatch the event → the subscriber puts a notification into the queue → the worker sends it later. An event does not provide asynchrony, delivery guarantees, or transactional consistency.

### 9.6. When to use

When several components react to one action and the main code must not depend on them: notifications, audit, integrations. Do not create an event for every trivial call.

> [!example] Practice 
> The task status change event in DrupalJira triggers an audit and puts a notification into the queue. The subscriber is a service with the `event_subscriber` tag, dependencies via the constructor.

> [!warning] Common mistakes
> 
> - Forgetting the `event_subscriber` tag or clearing the cache.
> - A mismatch in the event name or signature.
> - Dispatching an event before saving.
> - Heavy work in a subscriber (better Queue).
> - Replacing all hooks with events; an event without a real extension point.
> - Mixing up priorities (a higher one runs earlier).

> [!question] Interview questions 
> **Events vs. hooks?** Events are objects handled by subscriber services; hooks are named Drupal extension points implemented by functions or OOP methods.
> 
> **How to create a subscriber?** A class with `EventSubscriberInterface` and `getSubscribedEvents()`, a service with the `event_subscriber` tag, then a cache clear.
> 
> **What does priority mean?** The higher it is, the earlier the call; an event can be stopped with `stopPropagation()`.
> 
> **How to dispatch your own event?** An event class and `dispatch()` on `event_dispatcher` (via DI) after the real change.

---

## Block 10. Entity API

> [!abstract] Summary 
> An entity is a unit of Drupal data. An entity type (`node`, `user`, `taxonomy_term`) sets the common behavior, a bundle (`task`, `project`) is a subtype with its own fields. Entities are content (working data) or configuration (settings). The Entity API is the standard way to create, load, search, delete, and check access without direct SQL.

### 10.1. Content and Configuration Entity

- **Content Entity** — site data: Node, User, Comment, Term, Media, File, your own `time_log`. Fields, revisions, translations; the class extends `ContentEntityBase`.
- **Configuration Entity** — settings: a content type, a role, a vocabulary, a View. Stored as config, exported to YAML, no fields; `ConfigEntityBase`.

Choice: if it is transferred between environments as a setting — Config Entity; if it is created by users and processes — Content Entity.

### 10.2. Entity Type, Bundle, instance

An entity type is a kind (`node`), a bundle is a subtype (`article`, `task`) with its own fields, an instance is a specific record. Bundles are often config entities themselves (for node — `node_type`, parameter `bundle_entity_type`). If there are no subtypes, the bundle matches the type id. In DrupalJira, project and task are node bundles, `time_log` is a separate entity.

### 10.3. Loading and saving

```php
$storage = $this->entityTypeManager->getStorage('node');
$node = $storage->load(123);                 // NULL if it does not exist
$nodes = $storage->loadMultiple($ids);
$node = $storage->create(['type' => 'task', 'title' => 'Implement workflow', 'status' => 1]);
$node->save();                               // insert or update
$node->delete();
```

Here `status` is the **publication** status, not the custom `field_status` and not the moderation state. Before saving, check required fields, permissions, business rules; `$node->validate()` for constraints. Direct SQL against entity tables bypasses fields, revisions, hooks: do not use it.

### 10.4. Working with fields

```php
$status = $node->get('field_status')->value;
$node->set('field_status', 'in_progress')->save();
$project = $node->get('field_project')->entity;     // NULL if none
$id = $node->get('field_project')->target_id;
```

The value must match the field settings. Multi-value fields — Block 14.

### 10.5. Entity Query

```php
$ids = $storage->getQuery()
  ->accessCheck(TRUE)
  ->condition('type', 'task')->condition('field_status', 'in_progress')
  ->sort('created', 'DESC')->range(0, 10)->execute();
$tasks = $storage->loadMultiple($ids);
```

Returns IDs, not objects. `accessCheck()` is mandatory in Drupal 10/11: `TRUE` takes permissions into account, `FALSE` is only for system operations with a justification. `->count()` for counting.

### 10.6. Entity Access

`$node->access('view')`, `$node->access('update', NULL, TRUE)` → `AccessResultInterface`. Decided by the entity's access control handler, the hooks `hook_entity_access()`/`hook_ENTITY_TYPE_access()`, and for node — node grants. Access to individual fields — `hook_entity_field_access`. For creation — `createAccess()` on the handler. Permissions — in `permissions.yml` or `permission_callbacks`; `admin_permission` in the type description.

### 10.7. Entity Reference

An `entity_reference` field: `target_type`, `handler` (`default:node`), `handler_settings` (`target_bundles`). The DB stores `target_id`, `->entity` returns the object. The reverse relationship (a project's tasks) is found with a query on the reference field. Check existence, accessibility, allowed target types.

### 10.8. Revisions

Separate versions of an object: history and rollback.

```php
$node->setNewRevision(TRUE);
$node->setRevisionLogMessage('Status changed to in_progress');
$node->save();
```

Support depends on the entity type, configuration, and modules (Content Moderation).

### 10.9. Entity API and Render API

The Entity API is data, the Render API is display. For output use view builders and modes: `$viewBuilder->view($node, 'teaser')`. Unchecked user data is not inserted into HTML.

### 10.10. Custom Content Entity

When node does not fit (own structure, many records, a separate table), a custom content entity (`time_log`) is created. Components:

1. A class in `src/Entity/` (`ContentEntityBase`) with the `#[ContentEntityType]` attribute (an annotation in older versions), an interface.
2. `baseFieldDefinitions()`.
3. Handlers (Block 16).
4. Permissions and links (`links`), menu items/buttons.
5. Tables are created when the module is installed; changing the structure of an existing entity — an update hook (`EntityDefinitionUpdateManager`).

Parameters of `#[ContentEntityType]`:

- `id`, `label`, `label_collection`, `label_singular`, `label_plural`;
- `handlers`;
- `base_table`, `data_table` (per language), `revision_table`;
- `admin_permission`;
- `entity_keys`;
- `links` (canonical, add-form, edit-form, delete-form, collection);
- `field_ui_base_route` (Field UI attaches "Manage fields" and "Manage display" to it; without it there will be no field interface);
- `translatable`, `bundle_entity_type`, `bundle_label`, `show_revision_ui`, `constraints`.

Additional capabilities are attached via interfaces: `EntityChangedInterface` (modification date), `EntityOwnerInterface` (author), `EntityPublishedInterface`.

### 10.11. Entity keys

A mapping "role → field name", so that `$entity->id()`, `label()` work uniformly: `id`, `uuid`, `label`, `langcode`, `bundle`, `uid`/`owner`, `status`/`published`, `revision`.

```php
entity_keys: ['id' => 'id', 'uuid' => 'uuid', 'label' => 'title', 'owner' => 'uid'],
```

### 10.12. Base fields

```php
public static function baseFieldDefinitions(EntityTypeInterface $entity_type) {
  $fields = parent::baseFieldDefinitions($entity_type);
  $fields['hours'] = BaseFieldDefinition::create('decimal')
    ->setLabel(t('Hours'))->setRequired(TRUE)
    ->setDisplayOptions('form', ['type' => 'number', 'weight' => 0])
    ->setDisplayConfigurable('form', TRUE)->setDisplayConfigurable('view', TRUE);
  $fields['task'] = BaseFieldDefinition::create('entity_reference')
    ->setLabel(t('Task'))->setSetting('target_type', 'node')->setRequired(TRUE);
  return $fields;
}
```

Base fields are the same for all bundles of the type; configurable fields are created via Field UI for specific bundles and stored in configuration.

### 10.13. How entities are stored

- `base_table` — the main record (id, uuid, bundle, language, simple base fields).
- `data_table` — per-language data, `revision_table` and `revision_data_table` — revisions.
- Fields from Field UI — separate tables `type__field_name` (`node__field_status`) and `type_revision__field_name`, columns `name_value`, `name_target_id`.

If the field description changed after installation, an update hook is needed; otherwise the status report shows the warning "Mismatched entity and/or field definitions".

### 10.14. Life cycle hooks

`hook_entity_presave()`, `hook_entity_insert()`, `hook_entity_update()`, `hook_entity_delete()` and variants `hook_ENTITY_TYPE_presave()`. The same in the entity class: `preSave()`, `postSave()`, `preDelete()`.

> [!example] Practice 
> Loading and saving via storage, searching via Entity Query with an explicit `accessCheck`, checking `access()`, fields and revisions. SQL against entity tables is not written. When node does not fit — a custom content entity: in DrupalJira project and task are node bundles, time records are `time_log`.

> [!warning] Common mistakes
> 
> - Confusing entity type and bundle.
> - Confusing `status` (publication), `field_status`, and moderation state.
> - Forgetting `accessCheck()`.
> - Not checking for `NULL` after `load()` and for references.
> - Loading in a loop (N+1).
> - SQL instead of the Entity API.
> - Changing the schema of a custom entity without an update hook.
> - Not setting `field_ui_base_route` and being surprised there is no Field UI.

> [!question] Interview questions 
> **Entity type vs. bundle?** The type sets the general kind (node, user), the bundle is a subtype with its own set of fields (article, page).
> 
> **Content entity vs. configuration entity?** The first stores working data, the second describes settings and is exported to YAML.
> 
> **Why `accessCheck` in Entity Query?** Whether to take permissions into account in the selection; in Drupal 10/11 it is specified explicitly, `FALSE` only deliberately.
> 
> **Base fields vs. configurable fields?** The first are declared in code and exist on all bundles, the second are created via UI or configuration for specific bundles.
> 
> **What are entity keys?** A mapping of roles (id, uuid, label, bundle, owner) to field names.
> 
> **Where are fields from Field UI stored?** In separate tables `type__field_name`.

---

## Block 11. Plugin API

> [!abstract] Summary 
> A plugin is an interchangeable implementation of a known contract: a block, a field widget, a queue worker, a migration process plugin. The Plugin Manager finds plugins (by attributes or annotations), stores definitions, and creates instances. A plugin is needed for integration with a Drupal subsystem; business logic is better extracted into a service.

### 11.1. Basic concepts

- **Plugin type** — a kind (Block, FieldFormatter, QueueWorker).
- **Plugin Manager** — the service `plugin.manager.<type>`: knows the catalog (`Plugin/Block`), the interface, caches definitions, creates `createInstance('id', $config)`.
- **Plugin ID** — unique within a type.
- **Plugin Definition** — metadata: id, name, category, class.
- **Instance configuration** — the settings of a specific use (different from the definition).

Plugins are used for blocks, authentication methods, migration sources, widgets and formatters, queue handlers, Actions, conditions, text filters, image effects, Views plugins. Entity types are also discovered as plugins.

### 11.2. Discovery

- **Attribute discovery** — PHP attributes `#[Block(...)]` (10.2/10.3+, preferred in 11).
- **Annotation discovery** — annotations in a docblock (`@Block(...)`), becoming obsolete, found in older code.
- **YAML discovery** — menus, local tasks, `*.migrate_drupal.yml`.
- **Hook discovery** — `hook_*_info`.

Order: the manager scans `src/Plugin/<Type>/` in all modules, reads attributes, collects definitions. After adding a plugin — `drush cr` (definition cache). The specific mechanism depends on the plugin type.

### 11.3. Block Plugin

```php
#[Block(
  id: 'task_summary',
  admin_label: new TranslatableMarkup('Task Summary'),
  category: new TranslatableMarkup('DrupalJira'),
)]
final class TaskSummaryBlock extends BlockBase {
  public function build(): array {
    return ['#theme' => 'item_list', '#items' => [$this->t('Backlog')],
      '#cache' => ['tags' => ['node_list:task']]];
  }
}
```

`build()` returns a render array, not an HTML string: this way Drupal will process nested elements, libraries, cache metadata, and access. Block methods: `blockForm()`, `blockSubmit()` (settings), `blockAccess()`, `defaultConfiguration()`.

### 11.4. Other types

- Field Formatter (`FormatterBase`): `viewElements()`.
- Field Widget (`WidgetBase`): `formElement()`.
- Queue Worker (`QueueWorkerBase`): `processItem($data)`, `cron: ['time' => 60]`.
- Field Type (`FieldItemBase`).

### 11.5. DI in plugins

```php
final class TaskSummaryBlock extends BlockBase implements ContainerFactoryPluginInterface {
  public function __construct(array $configuration, $plugin_id, $plugin_definition,
    private readonly EntityTypeManagerInterface $entityTypeManager) {
    parent::__construct($configuration, $plugin_id, $plugin_definition);
  }
  public static function create(ContainerInterface $container, array $configuration, $plugin_id, $plugin_definition): static {
    return new static($configuration, $plugin_id, $plugin_definition, $container->get('entity_type.manager'));
  }
}
```

The signatures of base classes depend on the Drupal version.

### 11.6. Custom plugin type

1. An interface and a base class.
2. An attribute class (extends `Drupal\Component\Plugin\Attribute\Plugin`) with description fields.
3. A Plugin Manager (`DefaultPluginManager`): the constructor takes the folder, interface, and attribute class; `alterInfo()`, `setCacheBackend()`.
4. The service `plugin.manager.<type>` (`parent: default_plugin_manager`) and implementations in `src/Plugin/<Type>/`.

For variants generated from data (one block per project) — **derivatives**.

### 11.7. Plugin vs. hook, service, handler

A hook intervenes in an existing extension point; a plugin is a separate implementation of a known type. Not every extension has to be a plugin. A service is a single object with a permanent role, `plugin.manager.block` is a service that manages block plugins.

> [!example] Practice 
> Plugins where Drupal expects interchangeable implementations. A thin plugin integrates with the subsystem, a service holds the logic. The statistics block in DrupalJira builds a render array, and a service calculates the data.

> [!warning] Common mistakes
> 
> - Arguments in a plugin constructor without `ContainerFactoryPluginInterface`.
> - Not clearing the cache; wrong namespace/directory.
> - Business logic in a plugin.
> - Returning an HTML string instead of a render array.
> - Confusing the definition and the instance configuration.
> - A plugin where a hook or service would suffice.
> - The same ID for two plugins of one type.

> [!question] Interview questions 
> **What is a Plugin Manager?** A service that finds the plugins of a type, stores definitions, and creates instances.
> 
> **Annotation vs. attribute discovery?** The definition is in a docblock or in a PHP attribute; in newer versions (10.2/10.3+, 11) attributes are preferred.
> 
> **How to get services in a plugin?** `ContainerFactoryPluginInterface` and `create()`.
> 
> **When a plugin, when a hook?** A plugin is a separate configurable implementation of a known type; a hook is an intervention at an existing point.
> 
> **What does a custom plugin type consist of?** An interface, an attribute, a manager, registration as a service, implementations in `Plugin/<Type>`.

---

## Block 12. Batch API, Cron, and Queue API

> [!abstract] Summary 
> Long operations cannot be executed in a single HTTP request. The Batch API splits the work into steps with progress (started by the user). Cron performs periodic tasks. The Queue API stores individual jobs that are processed by workers. They are often combined.

### 12.1. Why

Importing thousands of records, mass processing, mailings, deleting outdated data, synchronization: in a single request PHP limits, timeouts, and overload are possible.

### 12.2. Batch API

```php
$builder = (new BatchBuilder())
  ->setTitle(t('Updating tasks'))
  ->setFinishCallback(['\Drupal\drupaljira_timelog\Batch\TaskBatch', 'finished'])
  ->addOperation(['\Drupal\drupaljira_timelog\Batch\TaskBatch', 'process'], [$ids]);
batch_set($builder->toArray());
```

(the old variant is the array `$batch = ['title', 'operations', 'finished']`). The operation:

```php
public static function process(array $ids, array &$context): void {
  if (!isset($context['sandbox']['progress'])) {
    $context['sandbox']['progress'] = 0;
    $context['sandbox']['max'] = count($ids);
  }
  $chunk = array_slice($ids, $context['sandbox']['progress'], 50);
  // process $chunk
  $context['sandbox']['progress'] += count($chunk);
  $context['finished'] = $context['sandbox']['max'] ? $context['sandbox']['progress'] / $context['sandbox']['max'] : 1;
}
```

- `$context['sandbox']` — state between calls of one operation;
- `$context['results']` — results for the next steps and the finish callback;
- `$context['message']` — progress text;
- `$context['finished']` — a fraction 0…1, the operation repeats until it reaches 1.

In an update hook it is the same: `hook_update_N(&$sandbox)` and `$sandbox['#finished']`. Batch does not make an operation safe automatically and is not a full background job independent of the browser: for that Queue is better.

### 12.3. Cron API

Periodic tasks: cleaning temporary data, processing queues, module maintenance, synchronization. Launch: Automated Cron, Drush, a system scheduler.

```php
function drupaljira_timelog_cron(): void { /* small tasks or putting items into a queue */ }
```

```bash
drush cron
*/15 * * * * cd /var/www/html && vendor/bin/drush cron
```

In `hook_cron()` do not do heavy processing of thousands of records without limits. The system cron merely launches Drupal Cron; on production an external scheduler is better. The time of the last run of your own tasks is stored in `State`.

### 12.4. Queue API

Concepts: queue, item, worker, claim, delete, backend.

```php
$this->queueFactory->get('drupaljira_notifications')->createItem(['task_id' => 123, 'recipient_uid' => 45]);
```

Worker:

```php
#[QueueWorker(id: 'drupaljira_notifications', title: new TranslatableMarkup('Task notifications'), cron: ['time' => 30])]
final class TaskNotificationWorker extends QueueWorkerBase {
  public function processItem($data): void { /* check the data, find the task and recipient, send */ }
}
```

`cron time` is a processing time budget in cron, not a guarantee. The task or user may have been deleted. Exceptions: `RequeueException` (retry later), `SuspendQueueException` (stop the queue, for example when an external system is unavailable). Manual run: `drush queue:run <name>`, list — `drush queue:list`. A queue can be processed not only by cron, but also by a separate CLI worker.

### 12.5. Queue reliability

- Reprocessing is possible: the worker is **idempotent** (a repeat does not produce duplicates: repeated charges, repeated emails).
- Job data is validated; put the minimum into the queue (IDs and parameters), load the current state in the worker.
- Temporary failures do not delete the item; errors are logged.
- Concurrent workers take races into account.
- Sensitive data and large objects are not put into the queue.

### 12.6. Comparison and choice

|API|Purpose|Scenario|
|---|---|---|
|Batch|a long operation in steps with progress|an admin updates thousands of tasks|
|Cron|periodic launch|cleaning old data|
|Queue|deferred processing of individual jobs|notifications after a task changes|

Combination: Batch imports a set of tasks → a notification job is created for each → cron runs the workers. For a short operation a queue is not needed.

> [!example] Practice 
> Batch — when the user launches a long action and waits for progress; Cron — regular maintenance; Queue — deferred independent jobs. In an update hook large volumes go in portions via `$sandbox`. In DrupalJira update hook `10001` processes existing tasks in portions, and it is logical to send notifications through a queue.

> [!warning] Common mistakes
> 
> - Heavy processing in `hook_cron()`.
> - Batch without a portion limit and without `$context['finished']`.
> - A non-idempotent worker.
> - Deleting a queue item on a temporary error.
> - Not setting up the system cron.
> - Sensitive data and huge objects in the queue.
> - A queue for short instant operations.

> [!question] Interview questions 
> **When Batch, Queue, Cron?** Batch — launched by a user with progress, Queue — deferred separate jobs, Cron — a schedule.
> 
> **What is worker idempotency?** Reprocessing an item does not produce duplicates; important because items can be executed repeatedly.
> 
> **A large volume in `hook_update_N()`?** `&$sandbox`, a portion per call, `$sandbox['#finished']` from 0 to 1.
> 
> **How to run a queue manually?** `drush queue:run <queue name>`, list — `drush queue:list`.
> 
> **`RequeueException` vs. `SuspendQueueException`?** The first returns the item to the queue for a retry, the second stops processing of the whole queue.

---

## Block 13. Migration API

> [!abstract] Summary 
> Data is transferred from a source into Drupal following the Source → Process → Destination scheme. Source reads rows, process plugins transform values, destination creates entities. The mapping of old and new records is stored in the ID map, so migrations can be rerun, updated, and rolled back. Relations between migrations are set by dependencies and `migration_lookup`.

### 13.1. Purpose

Transfer from an old Drupal version, CSV, XML/JSON, another CMS, an external DB, an outdated field structure. Used: Migrate (core), **Migrate Plus**, and **Migrate Tools**. Example: the old system stores the status `working`, the new one `in_progress`: Source provides `working`, Process converts it, Destination saves it into the field.

### 13.2. Components

- **Source** — where from: SQL, CSV, XML, JSON API, Drupal; rows with unique `ids`.
- **Process** — how to transform: rename, date format, status, find an entity, reference, merge.
- **Destination** — where to: nodes (`entity:node`), users, terms.
- **ID map** — a table mapping source identifiers to target identifiers.

### 13.3. Example

```yaml
id: legacy_tasks
label: 'Import legacy tasks'
migration_group: legacy
source:
  plugin: embedded_data
  data_rows:
    - { id: 1, title: 'Fix login', old_status: working }
  ids: { id: { type: integer } }
process:
  type: { plugin: default_value, default_value: task }
  title: title
  field_status:
    plugin: static_map
    source: old_status
    map: { todo: backlog, working: in_progress, qa: review, completed: done }
destination:
  plugin: 'entity:node'
```

`embedded_data` is a demonstration source. Migration files live in the module's `migrations/` directory (or as a config entity with Migrate Plus). If a value is not in `static_map`, the behavior is determined by settings (`default_value`, `bypass`): unknown values must be handled explicitly.

### 13.4. Process plugins

|Plugin|Purpose|
|---|---|
|`get`|get a value|
|`default_value`|default value|
|`static_map`|conversion by a map|
|`concat`|concatenation|
|`skip_on_empty`|skip when empty|
|`migration_lookup`|the result of another migration|
|`entity_lookup`|entity lookup (Migrate Plus)|
|`callback`, `format_date`, `explode`, `sub_process`|auxiliary|

A chain of plugins (pipeline) runs sequentially. A custom transformation is a custom process plugin (`transform()`).

### 13.5. Dependencies and lookup

First projects, then tasks:

```yaml
migration_dependencies:
  required: [legacy_projects]
process:
  field_project:
    plugin: migration_lookup
    migration: legacy_projects
    source: legacy_project_id
```

If a project is not found, decide: skip the task, import it without the relation, or fail with an error.

### 13.6. ID map

Source task `42` → node `137`. Needed for re-running, updating, relations between migrations, tracking processed records. The map does not replace a backup; a change in the source data will not necessarily update the target entities automatically.

### 13.7. Running

```bash
drush migrate:status
drush migrate:import legacy_tasks
drush migrate:import legacy_tasks --update     # update imported ones
drush migrate:import legacy_tasks --limit=10   # trial run
drush migrate:messages legacy_tasks
drush migrate:rollback legacy_tasks            # may delete created entities
```

Incremental imports — `track_changes` and `high_water_property`. Changing the YAML does not always update what has already been imported: determine which records to reprocess, whether `--update` or a rollback is needed, whether the updated definition needs to be imported (if it is a config entity). Test on a test copy first.

### 13.8. Migration vs. update hook

A migration transfers data **from an external source**; an update hook transforms an **existing** installation during a code update (for example, initializing `field_project_type` for existing tasks).

> [!example] Practice 
> Migrations are described in YAML, run via Migrate Tools and Drush, dependencies are set in `migration_dependencies`, relations via `migration_lookup`, values via `static_map`. Before a real run — a data copy, `--limit`, `migrate:messages`. In DrupalJira, when migrating statuses, the moderation state must also be aligned.

> [!warning] Common mistakes
> 
> - Not specifying `migration_dependencies`, running tasks before projects.
> - Changing YAML and expecting the imported data to update automatically.
> - `rollback` without understanding the consequences.
> - Not handling values outside `static_map`.
> - Migrating only `field_status`, forgetting the moderation state and publication.
> - Not testing on a test copy.
> - Using Migrate Plus plugins without installing the module.

> [!question] Interview questions 
> **What does a migration consist of?** Source (reading), process (transformation), destination (saving).
> 
> **What is the ID map?** A table mapping source records to created entities; allows re-running, updating, and rolling back.
> 
> **How to link two migrations?** `migration_dependencies` and the `migration_lookup` plugin.
> 
> **Which Drush commands?** `migrate:status`, `migrate:import` (`--update`, `--limit`), `migrate:messages`, `migrate:rollback`.

---

## Block 14. Field API

> [!abstract] Summary 
> The Field API adds structured data to entities. Field Type (the data type), Field Storage (how and where it is stored, cardinality), and Field Configuration (usage on a specific bundle) are distinguished. Input is handled by a widget, output by a formatter. Fields are base (in the entity code) or configurable (UI/configuration). There are also computed fields.

### 14.1. Field Type, Storage, Configuration

|Type|Purpose|
|---|---|
|`string`, `text_long`, `text`|short text, long text, formatted text|
|`integer`, `decimal`, `float`, `boolean`|numbers, logic|
|`entity_reference`|a reference to an entity|
|`datetime`|date and time|
|`list_string`|a list of allowed values (statuses)|
|`image`, `file`|files|

- **Field Storage** (`field.storage.node.field_x`, `FieldStorageConfig`) — type, cardinality, storage settings; bound to the entity type and the field name. Once data exists, some settings cannot be changed (for example, the target type of a reference).
- **Field Configuration** (`field.field.node.task.field_x`, `FieldConfig`) — usage on a bundle: label, description, required flag, default value. One storage can be used in several bundles, each has its own FieldConfig.

`field_estimate` — decimal, a single value, added to `task`, but absent from `project`.

### 14.2. Base fields and configurable fields

- **Base fields** — in code (`baseFieldDefinitions()`, `BaseFieldDefinition`), present on all bundles (`title`, `uid`, `status` of node); changed by code and update hooks.
- **Configurable fields** — via UI/configuration for specific bundles (`field_status` on `task`); changed via UI and Configuration Management.

For a custom entity such as `time_log` the fields are usually base. For base fields `setDisplayConfigurable('form'|'view', TRUE)` or `setDisplayOptions()` is needed, otherwise the field will not appear in the display settings.

### 14.3. Creating a field

Via UI: content type → manage fields → add → type → cardinality, required → form and display → **export the configuration**. Via YAML: write it based on an export of a working configuration rather than "at random"; the binding to the bundle is stored separately.

### 14.4. Reading and writing values

```php
$estimate = $node->get('field_estimate')->value;      // decimal — usually a string
$node->hasField('field_estimate');
$assignee = $node->get('field_assignee')->entity;     // may be NULL
foreach ($node->get('field_attachments') as $item) { $id = $item->target_id; }
$node->get('field_x')->isEmpty();
$node->get('field_x')->getValue();                    // array of values
$node->get('field_project')->referencedEntities();
```

`->value` reads only the first item. **FieldItemList** — the list of a field's values on an entity (`first()`, `get($delta)`, `getValue()`, `setValue()`, `isEmpty()`), **FieldItemBase** — a single value with properties (`value`, `target_id`, `format`). Different types have different properties: you cannot assume that `value` is always there.

```php
[['target_id' => 5], ['target_id' => 7]]   // structure of a multi-value reference
```

### 14.5. Cardinality and attributes

- Cardinality: `1`, a number N, `-1` (`CARDINALITY_UNLIMITED`).
- `required`, `translatable`, `revisionable`.

```php
BaseFieldDefinition::create('string')->setLabel(t('Note'))
  ->setCardinality(3)->setRequired(FALSE)->setTranslatable(TRUE)->setRevisionable(TRUE);
```

### 14.6. Entity Reference and taxonomy

Settings: `target_type` (`taxonomy_term`), `handler` (`default:taxonomy_term`), `handler_settings` with `target_bundles` and `auto_create`. A term is a Content Entity, a vocabulary is a Config Entity. Account for the existence and accessibility of the target, allowed types, behavior on deletion.

### 14.7. Widget and Formatter

A **Widget** is how a value is entered (a text field, a list, autocomplete; configured in _Manage form display_). A **Formatter** is how it is displayed (text, link, date, the name of a related entity; _Manage display_). One field type can have several widgets and formatters. These are plugins (Block 11): a custom field type is three plugins: FieldType, FieldWidget, FieldFormatter. A widget does not replace server-side validation.

### 14.8. Form Display and View Display

- Form Display (`core.entity_form_display.{type}.{bundle}.{mode}`) — widgets and order in the form.
- View Display (`core.entity_view_display.{type}.{bundle}.{mode}`) — formatters on output (full, teaser, etc.).

This is configuration, so it is exported to YAML.

### 14.9. Computed fields

The value is not stored in the DB, it is calculated on access. Two elements: a field definition with `setComputed(TRUE)` (no tables and no saving needed) and a list class with `computeValue()`. To add one to someone else's entity use `hook_entity_base_field_info()` or `hook_entity_bundle_field_info()`.

```php
$fields['logged_hours'] = BaseFieldDefinition::create('float')
  ->setLabel(t('Hours logged'))->setComputed(TRUE)->setReadOnly(TRUE)
  ->setClass(LoggedHoursItemList::class)->setDisplayConfigurable('view', TRUE);

class LoggedHoursItemList extends FieldItemList {
  use ComputedItemListTrait;
  protected function computeValue() {
    $value = 0;   // calculation, for example via a service
    $this->list[0] = $this->createItem(0, $value);
  }
}
```

|Stored field|Computed field|
|---|---|
|the value is in the DB, can be searched and sorted (Entity Query, Views)|there is no value in the DB, always up to date|
|read without calculations|cannot simply be searched or sorted, creates no tables|

When to use what: a computed field — the value is derived but is needed as an entity field (output, API, template); a service — complex calculations, sums over many entities, reuse (in DrupalJira — `TaskStatService`); an entity method — a simple calculation from the entity's own data; a regular field — the value must be stored, searched, sorted, entered.

Performance: in a list of many entities, the calculation and queries for each one is N+1. Mitigation: one query for all IDs in a service, static cache. If filters and sorting are needed — a stored field. The output must carry cache tags (on the `time_log` entities the value depends on), contexts, and max-age.

### 14.10. Fields, Views, Configuration Management

Views outputs fields without manual queries. Field definitions are configuration: when transferring, account for the module of the field type, the entity type and bundle, reference targets, widgets, formatters, display configurations. If something is missing, the import may fail. Exporting the structure does not transfer the field values of existing nodes.

### 14.11. Schema changes

A new schema is not applied to the existing DB automatically: update hooks, a configuration update, or a data migration are needed. Be especially careful when changing a field type and storage structure if data exists.

> [!example] Practice 
> Create a field via UI or configuration, check the widget and formatter, export the configuration, read values via the Entity API. For existing data, any storage change is an update hook or a migration. For a custom entity the fields are base, for nodes — configurable. In DrupalJira a task has a list of statuses, a decimal estimate, and a reference to a project.

> [!warning] Common mistakes
> 
> - Confusing field storage and field configuration.
> - Reading only `->value` of a multi-value field or a reference.
> - Not checking for `NULL` in an Entity Reference.
> - Changing the storage of a field with data without an update hook or migration.
> - Writing field YAML manually without an export; forgetting the export after UI work.
> - Changing `field_status` while forgetting the moderation state.
> - Treating a widget as a security mechanism.
> - Expecting that a computed field can be filtered on.

> [!question] Interview questions 
> **Field storage vs. field configuration?** Storage describes the type, cardinality, and storage for an entity type; configuration (`field.field.*`) is the usage on a specific bundle.
> 
> **Widget vs. formatter?** A widget is responsible for input, a formatter for display.
> 
> **How to read a multi-value field?** By iterating over the items or `getValue()`; `->value` gives only the first item.
> 
> **What to do when changing the storage of a field with data?** An update hook or a migration: the schema is not applied automatically.
> 
> **What is a computed field?** A field without storage, the value is calculated on access (`setComputed(TRUE)` and `computeValue()`); it cannot be searched or sorted.

---

## Block 15. Authentication API

> [!abstract] Summary 
> Authentication answers "who is making the request", authorization — "what are they allowed to do". Authentication is handled by authentication providers: cookie session, HTTP Basic, tokens and OAuth (contrib). After the user is determined, Drupal still checks access to the route, the entity, and business rules.

### 15.1. Session and providers

After login Drupal uses a session in a cookie. For external clients — a token or another method. An **Authentication Provider** is a component that determines whether a mechanism can authenticate the request (`applies()`, `authenticate()`). They are selected by request type, modules, route configuration (`options: _auth: [...]`) and must be enabled and configured.

|Mechanism|Principle|
|---|---|
|Session/Cookie|a cookie with the session id, a regular site|
|HTTP Basic (`basic_auth`)|login and password in `Authorization`, integrations|
|Bearer/OAuth/JWT|a token; contrib (`simple_oauth`, key-auth, etc.)|
|external identification|SSO|

Not all mechanisms are in core; the presence of a Bearer header does not enable support by itself.

### 15.2. REST and authentication

For REST, configure the resource, methods, formats, authentication (`authentication: [cookie, basic_auth]` in `rest.resource.*`), permissions, and access to entities. Even with a confirmed identity, the right to the operation is checked.

### 15.3. Basic Authentication

```http
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

Base64 is encoding, not encryption, so use it only over HTTPS. Suitable for individual integrations; account for the storage of secrets in the client and access restrictions. If a mechanism with limited tokens is possible, prefer it.

### 15.4. Cookie and CSRF

The browser sends cookies automatically, which creates a CSRF risk for state-changing operations. Check the method, the CSRF token (for REST/JSON:API with cookies — `X-CSRF-Token` from `/session/token`), permissions, access to the entity, validity of the operation. With a Bearer token in a header the threat model is different, but the API is not protected automatically: account for token storage, XSS, CORS, HTTPS, lifetime, revocation, permissions.

### 15.5. Tokens

Expiration, revocation, secure storage, scopes, HTTPS, protection against leaks through logs and URLs; secret tokens are not passed in the query string. For integrations — a separate account with minimal permissions.

### 15.6. Sequence and comparison

Request → authentication → route access → entity access → business rules → change → response.

|Authentication|Authorization|
|---|---|
|determines identity|determines available actions|
|credentials or a token|permissions and access rules|
|"Who are you?"|"What are you allowed to do?"|

### 15.7. Custom provider

Needed when standard mechanisms are insufficient. A service with the `authentication_provider` tag and `AuthenticationProviderInterface`:

```yaml
services:
  authentication.example_token:
    class: Drupal\example\Authentication\Provider\TokenAuth
    arguments: ['@entity_type.manager']
    tags:
      - { name: authentication_provider, provider_id: 'example_token', priority: 100 }
```

```php
final class TokenAuth implements AuthenticationProviderInterface {
  public function applies(Request $request): bool { return $request->headers->has('X-Api-Token'); }
  public function authenticate(Request $request) { /* check the token; return an account or NULL */ }
}
```

Consider the order of several providers, handling of invalid data, token security, tests of positive and negative scenarios. A custom mechanism is not created for a single route; a "user id" from the request is not accepted without verifying authenticity.

### 15.8. API security

HTTPS, minimal permissions, access to each entity, input validation, method restrictions, secrets not in logs, correct errors, tests without authentication and with insufficient permissions, flood control. A user must not get someone else's task by simply changing the ID in the URL.

> [!example] Practice 
> For an external integration choose a provider: cookie for the browser, Basic over HTTPS for simple cases, OAuth/tokens from contrib for APIs. Specify it in `rest.resource.*` or the route's `_auth`. A system that reads DrupalJira tasks gets a separate user with only the read permission.

> [!warning] Common mistakes
> 
> - Confusing authentication and authorization.
> - Assuming that a token or session grants the right to everything.
> - Basic Auth without HTTPS; tokens in URLs or logs.
> - Not enabling a provider for a route.
> - Ignoring CSRF for cookie requests.
> - A custom provider where a ready-made one is enough.
> - Administrator permissions for a technical account.

> [!question] Interview questions 
> **Authentication vs. authorization?** The first determines who is making the request, the second what they are allowed to do.
> 
> **What is an Authentication Provider?** A service with the `authentication_provider` tag and `AuthenticationProviderInterface`: `applies()` — whether it fits the request, `authenticate()` — returns the user.
> 
> **How to enable Basic Auth for a REST resource?** Enable `basic_auth` and add `basic_auth` to `authentication` in the `rest.resource.*` configuration.
> 
> **Why must Basic Auth not be used without HTTPS?** Base64 does not encrypt: the login and password can be read if intercepted.

---

## Block 16. Entity Handlers

> [!abstract] Summary 
> A handler is a class that performs one role for an entity type: list, form, permission check, routes. The entity type merely lists in the `handlers` parameter which class is responsible for what. Any role can be replaced separately. If a handler is not specified, the entity does not have this capability (without `route_provider` no pages will appear).

### 16.1. Main handlers

|Handler|Role|
|---|---|
|`storage`|loading and saving (default `SqlContentEntityStorage`)|
|`list_builder` (`EntityListBuilder`)|the list page: `buildHeader()`, `buildRow($entity)`, `getDefaultOperations()`|
|`form`|forms `default`, `add`, `edit`, `delete`; base classes `ContentEntityForm`, `EntityForm`, `ContentEntityDeleteForm`|
|`access` (`EntityAccessControlHandler`)|`checkAccess($entity, $operation, $account)`, `checkCreateAccess()` → `AccessResult`|
|`views_data` (`EntityViewsData`)|entity fields for Views|
|`route_provider`|automatically creates routes from `links`: `DefaultHtmlRouteProvider`, `AdminHtmlRouteProvider` (canonical, add_form, edit_form, delete_form, collection)|
|`view_builder` (`EntityViewBuilder`)|entity display|

Getting a handler: `getListBuilder()`, `getAccessControlHandler()`, `getFormObject($type, 'edit')`, `getStorage()`, `getViewBuilder()` on `EntityTypeManager`.

### 16.2. Connecting in the attribute

```php
#[ContentEntityType(
  id: 'time_log',
  label: new TranslatableMarkup('Time log'),
  handlers: [
    'list_builder' => TimeLogListBuilder::class,
    'form' => ['default' => TimeLogForm::class, 'delete' => ContentEntityDeleteForm::class],
    'access' => TimeLogAccessControlHandler::class,
    'views_data' => EntityViewsData::class,
    'route_provider' => ['html' => AdminHtmlRouteProvider::class],
  ],
  base_table: 'time_log',
  admin_permission: 'administer time_log',
  entity_keys: ['id' => 'id', 'uuid' => 'uuid'],
  links: [
    'collection' => '/admin/content/time-log',
    'add-form' => '/admin/content/time-log/add',
    'edit-form' => '/admin/content/time-log/{time_log}/edit',
    'delete-form' => '/admin/content/time-log/{time_log}/delete',
  ],
)]
```

### 16.3. A custom handler

Almost always you extend a standard class and override the needed methods.

```php
class TimeLogListBuilder extends EntityListBuilder {
  public function buildHeader() { $h['id'] = $this->t('ID'); return $h + parent::buildHeader(); }
  public function buildRow(EntityInterface $entity) { $r['id'] = $entity->id(); return $r + parent::buildRow($entity); }
}
```

If you need services — `createInstance(ContainerInterface $container, EntityTypeInterface $entity_type)` (already present in standard classes).

### 16.4. Handler, service, plugin, controller

- A service — one object per site, declared in `services.yml`, not bound to an entity type.
- A handler — the role of a specific entity type, specified in the entity description, created by the entity manager.
- A plugin — one of the variants of a kind, found by Drupal via an attribute.
- A controller — a route class, returns the page content.

> [!warning] Common mistakes
> 
> - No `route_provider` or `links`: no pages.
> - Writing a handler from scratch instead of extending.
> - Forgetting `drush cr` after changing the definition.

> [!question] Interview questions 
> **How does Drupal know which class is responsible for the list, form, permissions?** It reads `handlers` in the type description; `EntityTypeManager` finds and creates the class by the type id and the role.
> 
> **Handler vs. service?** A handler is the role of a specific entity type, a service is one shared object with a permanent role.

---

## Block 17. Form API

> [!abstract] Summary 
> Forms are built on `FormBase` (a regular form), `ConfigFormBase` (module settings), and `EntityForm`/`ContentEntityForm` (creating and editing an entity). Four methods: `getFormId()`, `buildForm()`, `validateForm()`, `submitForm()`. The Form API automatically protects forms with a CSRF token and supports AJAX.

### 17.1. Connecting

A regular form — a route with `_form: '\Drupal\mymodule\Form\MyForm'`. An entity form — via `handlers` (Block 16).

### 17.2. Methods

- `getFormId()` — a unique name (`hook_form_FORM_ID_alter()` works through it).
- `buildForm()` — an array describing the form.
- `validateForm()` — validation; error: `$form_state->setErrorByName('hours', $this->t('...'))`. If there are errors, `submitForm()` is not called.
- `submitForm()` — saving; redirect `$form_state->setRedirect('route.name')`.

```php
class HoursForm extends FormBase {
  public function getFormId() { return 'mymodule_hours_form'; }
  public function buildForm(array $form, FormStateInterface $form_state) {
    $form['hours'] = ['#type' => 'number', '#title' => $this->t('Hours'), '#required' => TRUE];
    $form['actions']['submit'] = ['#type' => 'submit', '#value' => $this->t('Save')];
    return $form;
  }
  public function validateForm(array &$form, FormStateInterface $form_state) {
    if ($form_state->getValue('hours') <= 0) {
      $form_state->setErrorByName('hours', $this->t('Hours must be greater than zero.'));
    }
  }
  public function submitForm(array &$form, FormStateInterface $form_state) { /* save */ }
}
```

### 17.3. Element properties

`#type` (`textfield`, `textarea`, `number`, `select`, `checkbox`, `checkboxes`, `radios`, `entity_autocomplete`, `submit`, `details`, `container`), `#title`, `#default_value`, `#required`, `#options` (for select/radios/checkboxes), `#states` (show/hide/disable based on another field without JS).

```php
$form['reason'] = ['#type' => 'textfield', '#title' => t('Reason'),
  '#states' => ['visible' => [':input[name="status"]' => ['value' => 'blocked']]]];
```

### 17.4. Values and storage

`$form_state->getValue('name')`, `getValues()`; with `'#tree' => TRUE` — `getValue(['group', 'field'])`. Data between steps — `$form_state->set()`/`get()`. Saving: into configuration `->set('key', $v)->save()`, into an entity `$entity->set('field', $v)->save()`. Additional handlers — `$form['#validate'][]`, `$form['#submit'][]` (or on a specific button).

### 17.5. DI in forms

```php
class MyForm extends FormBase {
  public function __construct(protected EntityTypeManagerInterface $entityTypeManager) {}
  public static function create(ContainerInterface $container) {
    return new static($container->get('entity_type.manager'));
  }
}
```

### 17.6. Settings form

```php
class SettingsForm extends ConfigFormBase {
  protected function getEditableConfigNames() { return ['mymodule.settings']; }
  public function getFormId() { return 'mymodule_settings'; }
  public function buildForm(array $form, FormStateInterface $form_state) {
    $form['limit'] = ['#type' => 'number', '#title' => $this->t('Hours limit'),
      '#default_value' => $this->config('mymodule.settings')->get('limit')];
    return parent::buildForm($form, $form_state);
  }
  public function submitForm(array &$form, FormStateInterface $form_state) {
    $this->config('mymodule.settings')->set('limit', $form_state->getValue('limit'))->save();
    parent::submitForm($form, $form_state);
  }
}
```

For module settings a config schema is described.

### 17.7. Entity form

`ContentEntityForm`, connected via `handlers`, `$this->entity` is available, fields are built from Form Display, saving is `save()`.

### 17.8. AJAX

Three steps: the element has `#ajax` (`callback`, `wrapper`, `event`); the block being updated is wrapped in a container with that id; the callback returns a part of the form or an `AjaxResponse` with commands (`ReplaceCommand`, `HtmlCommand`, `AppendCommand`, `InvokeCommand`, `CssCommand`, `MessageCommand`, `RedirectCommand`).

```php
$form['project'] = ['#type' => 'select', '#options' => $projects,
  '#ajax' => ['callback' => '::updateTasks', 'wrapper' => 'tasks-wrapper', 'event' => 'change']];
$form['tasks'] = ['#type' => 'select', '#options' => $tasks,
  '#prefix' => '<div id="tasks-wrapper">', '#suffix' => '</div>'];

public function updateTasks(array &$form, FormStateInterface $form_state) { return $form['tasks']; }
```

### 17.9. Altering other people's forms

`hook_form_FORM_ID_alter()` lets you add a field, change a label, hide an element (`#access => FALSE`), add a check and a submit. An added field will not be saved anywhere by itself: you need your own submit handler.

```php
function mymodule_form_node_task_edit_form_alter(&$form, FormStateInterface $form_state) {
  $form['my_note'] = ['#type' => 'textfield', '#title' => t('Note')];
  $form['#validate'][] = 'mymodule_task_validate';
  $form['actions']['submit']['#submit'][] = 'mymodule_task_submit';
}
```

> [!warning] Common mistakes
> 
> - Saving values in `validateForm()`.
> - Not calling `parent::buildForm()`/`parent::submitForm()` in `ConfigFormBase`.
> - Forgetting `getEditableConfigNames()`.
> - No container with an id for AJAX.
> - Adding a field to someone else's form without a submit handler.
> - Building an HTML form manually instead of the Form API (CSRF protection is lost).

> [!question] Interview questions 
> **The main methods of a form?** `getFormId()`, `buildForm()`, `validateForm()`, `submitForm()`.
> 
> **How does `ConfigFormBase` differ from `FormBase`?** It is intended for settings: values are saved into configuration, `getEditableConfigNames()` is needed.
> 
> **How to get services in a form?** `create()` takes them from the container and passes them to the constructor.
> 
> **How does AJAX work in a form?** `#ajax` with a callback and wrapper; the callback returns a part of the form or an `AjaxResponse` with commands.

---

## Block 18. Changing Someone Else's Code and Entities Without Editing

> [!abstract] Summary 
> Core and other people's modules are not edited: the changes disappear on update. Instead, extension points are used: alter hooks, event subscribers, service replacement, settings, theming. A value that lies in config is changed via config; a value written in a PHP attribute is changed via an alter hook.

### 18.1. Labels and entity definitions

An entity type is described in PHP (`#[ContentEntityType]`, `#[ConfigEntityType]`) — its "passport". It is not changed via the UI; from your own module — via hooks. After changing definitions, `drush cr`.

```php
function mymodule_entity_type_alter(array &$entity_types) {
  $entity_types['node']->setLabel(t('Content item'));        // also setClass(), etc.
}
function mymodule_entity_bundle_info_alter(array &$bundles) {
  $bundles['node']['article']['label'] = t('News');
}
function mymodule_entity_base_field_info_alter(array &$fields, EntityTypeInterface $entity_type) {
  if ($entity_type->id() === 'node') { $fields['title']->setLabel(t('Task name')); }
}
```

`hook_entity_bundle_field_info_alter()` changes the fields attached to a bundle. Field labels from Field UI are stored in configuration: they are changed there, not by code.

### 18.2. Config or entity definition

- **Config** — settings in the DB and YAML (`node.type.article`, `field.field.node.article.field_x`); changed via the admin UI, `drush config:set`, import; transferred between environments.
- **Entity definition** — in a PHP attribute; changed only by code (an alter hook), not exported to YAML.

### 18.3. Forms, actions, links, routes

|What to change|With what|
|---|---|
|form elements|`hook_form_alter()` and variants|
|operation links (Edit, Delete)|`hook_entity_operation_alter()`|
|tabs and "Add" buttons|`hook_menu_local_tasks_alter()`, `hook_menu_local_actions_alter()`|
|menu items|`hook_menu_links_discovered_alter()`|
|individual links|`hook_link_alter()`|
|route addresses, titles, permissions|`RouteSubscriber`, the `alterRoutes()` method|
|simple text replacement|string translation (locale) or `$settings['locale_custom_strings_en']`|

### 18.4. Three variants of form_alter

- `hook_form_alter()` — for all forms, check `$form_id` inside.
- `hook_form_BASE_FORM_ID_alter()` — for a family of forms (`hook_form_node_form_alter()` — all node forms).
- `hook_form_FORM_ID_alter()` — for one form; the most precise, usually the best.

General hooks are called before specific ones.

### 18.5. HTML

- `hook_theme()` — a module registers its own template (new output).
- `hook_preprocess_HOOK()` — changes or adds variables before the template.
- Overriding Twig in a theme — changes the markup; enable Twig debug to see template names.
- `hook_theme_suggestions_HOOK_alter()` — your own template names, `hook_theme_registry_alter()` — changing the registration of someone else's theme hook.

```php
function mymodule_preprocess_node(array &$variables) { $variables['is_overdue'] = FALSE; }
// template in the theme: templates/node--task--full.html.twig
```

### 18.6. Order of preference of methods

1. Settings (config).
2. Alter hooks.
3. An event subscriber.
4. Replacing or decorating a service.
5. Replacing a handler or plugin class via alter.
6. The theme (preprocess, Twig).
7. A patch via composer-patches — a last resort, record the reason.

> [!question] Interview questions 
> **How to change a field label?** A field from the admin UI — via config; a base field — `hook_entity_base_field_info_alter()`.
> 
> **How to change the name of an entity type or bundle?** `hook_entity_type_alter()`, `hook_entity_bundle_info_alter()`, or config (for content types).
> 
> **Which form_alter to choose?** The narrowest: `hook_form_FORM_ID_alter()`.
> 
> **How to change someone else's functionality without editing the code?** Config → alter hooks → events → service replacement → theme → patch.

---

## Block 19. settings.php and the DB Connection

> [!abstract] Summary 
> DB connection parameters live in the `settings.php` of the chosen site (and environment files), not in configuration: configuration is stored in the DB, and to read it you must first connect. On each environment they are different, secrets come from the environment, not from Git.

### 19.1. settings.php and settings.local.php

- `sites/default/settings.php` — the main PHP settings file of the site: DB connection, `hash_salt`, `config_sync_directory`, `trusted_host_patterns`, the private files path, overrides. Read at the very start of bootstrapping.
- `settings.local.php` — settings for this environment only, included by a block at the end of `settings.php` if the file exists; not put into Git (`.gitignore`). Also `settings.ddev.php`, `settings.lando.php` are created by the environment.

### 19.2. $databases

The structure is `$databases[key][target]`: `default` — the main DB, `replica` — a read copy.

```php
$databases['default']['default'] = [
  'driver' => 'mysql', 'database' => 'drupaljira',
  'username' => 'db_user', 'password' => getenv('DB_PASSWORD'),
  'host' => 'db', 'port' => '3306', 'prefix' => '',
  'namespace' => 'Drupal\\mysql\\Driver\\Database\\mysql',
  'autoload' => 'core/modules/mysql/src/Driver/Database/mysql/',
];
```

Parameters: `driver`, `database`, `username`, `password`, `host`/`port` (or `unix_socket`), `prefix`, `namespace`, `autoload`.

### 19.3. Connection settings and configuration

|settings.php|YAML configuration|
|---|---|
|PHP arrays `$databases`, `$settings`, `$config`|`config/sync`, `config/install`|
|environment, secrets, paths|site and module settings|
|does not end up in config sync|transferred via Git|

Rule: a value that differs per environment or is secret — in `settings.php`; common to the site — in configuration.

### 19.4. How Drupal chooses settings

By domain, port, and path a folder in `sites/` is looked up (the mappings are in `sites/sites.php`); if none — `sites/default`. Then the `settings.php` of that folder is read. In multisite each site has its own DB, files, and config sync (Block 8).

### 19.5. Secrets

Passwords and keys are taken from the environment: `getenv('DB_PASSWORD')`, `$settings['hash_salt'] = getenv('DRUPAL_HASH_SALT');`. Sources: server or container variables, `.env` (not in Git), CI secrets, Docker, Kubernetes. On production `settings.php` is made read-only (`chmod 444`). This applies to `hash_salt`, API keys, SMTP passwords as well.

Why not in Git: Git stores history, a deleted password remains in old commits; a password leak means access to the DB. `settings.local.php` and `.env` go in `.gitignore`, the shared `settings.php` has no secrets; if a password got into Git — change it immediately. `config/sync` is also in Git, secrets are not stored in it: use overrides or the Key and Encrypt modules.

### 19.6. Overrides

- `$config['settings.name']['key'] = value;` — substitution at runtime, not written to the DB (visible via `get()`, not visible via `getEditable()`).
- `$settings['config_sync_directory'] = '../config/sync';`
- `$settings['hash_salt']`, `trusted_host_patterns`, `file_private_path`, `file_temp_path`.
- `$settings['config_exclude_modules']` — exclude modules from synchronization.

### 19.7. Diagnosing the connection

`drush status` (the Database row), `drush sql:cli`, `drush sql:connect`, `drush sqlq "SELECT 1"`, the page `/admin/reports/status`. An empty `settings.php` or no connection — the installer (`install.php`) opens.

Typical MySQL errors:

- 1045 Access denied — wrong login/password or no rights to the DB;
- 1049 Unknown database — the DB does not exist or the name is wrong;
- 2002 Connection refused — wrong host/port, the DB server is not running; in Docker and DDEV the host is the container name (`db`), not `localhost`;
- Driver not found — the `pdo_mysql` extension is missing or `namespace`/`autoload` are wrong.

Check: connect with the `mysql` client using the same data, the values of environment variables, which settings file was actually included (was `settings.local.php` picked up), the DB user's rights.

> [!warning] Common mistakes
> 
> - The DB password in Git.
> - Confusing `$databases` in `settings.php` with YAML configuration.
> - Specifying `localhost` instead of the container name in Docker.
> - Forgetting that `settings.local.php` does not reach other environments.

> [!question] Interview questions 
> **Where does Drupal get the DB connection parameters?** From `$databases` in the `settings.php` of the chosen site and the included environment files.
> 
> **Why can't they be stored in configuration?** Configuration lies in the DB: to read it you must connect first. Besides, each environment has its own database and password, while configuration lies in Git.
> 
> **How not to commit secrets?** `getenv()`, `.env` and `settings.local.php` in `.gitignore`, CI/container secrets.
> 
> **What to do if a password got into Git?** Change it immediately.

---

## Block 20. Practical Flows: Creating via the Admin UI and in Code, Overriding, Nuances

> [!abstract] Summary 
> Any Drupal object can be created in three ways: the **admin UI** (the result lives in the active configuration/DB), **configuration in code** (YAML in a module's `config/install`), and **PHP code** (classes, plugins, services). The difference determines whether the object is transferred between environments and whether it can be changed after installation. A common technique: do it in the admin UI → `drush cex` → move the needed YAML files into the module.

### 20.0. Admin UI: addresses you need to know

|What|Path|
|---|---|
|Modules (enable/disable)|`/admin/modules`|
|Content types|`/admin/structure/types`|
|Fields of the `task` type|`/admin/structure/types/manage/task/fields`|
|Edit form (widgets)|`/admin/structure/types/manage/task/form-display`|
|Display (formatters)|`/admin/structure/types/manage/task/display`|
|Roles and permissions|`/admin/people/roles`, `/admin/people/permissions`|
|Views|`/admin/structure/views`|
|Block placement|`/admin/structure/block`|
|Menus|`/admin/structure/menu`|
|Taxonomy|`/admin/structure/taxonomy`|
|Text formats|`/admin/config/content/formats`|
|Languages / content translation|`/admin/config/regional/language`, `/admin/config/regional/content-language`|
|Configuration synchronization|`/admin/config/development/configuration`|
|Logs / status|`/admin/reports/dblog`, `/admin/reports/status`|

### 20.1. Creating a module (the basis of everything)

Structure:

```text
web/modules/custom/drupaljira_timelog/
  drupaljira_timelog.info.yml
  drupaljira_timelog.module            # hooks in procedural style
  drupaljira_timelog.install           # hook_install, hook_schema, hook_update_N
  drupaljira_timelog.services.yml
  drupaljira_timelog.routing.yml
  drupaljira_timelog.permissions.yml
  drupaljira_timelog.links.menu.yml / .links.task.yml / .links.action.yml
  config/install/  config/optional/  config/schema/
  src/ (Controller, Form, Entity, Plugin/..., EventSubscriber, Service)
  templates/
```

```yaml
# drupaljira_timelog.info.yml
name: 'DrupalJira TimeLog'
type: module
description: 'Time tracking for tasks.'
package: DrupalJira
core_version_requirement: ^10.3 || ^11
dependencies:
  - drupal:node
  - drupal:views
```

Flow: create the folder and `info.yml` → in the admin UI at `/admin/modules` or `drush en drupaljira_timelog` → `drush cr`. After that the code is picked up: PSR-4 autoload for `Drupal\drupaljira_timelog\` → `src/`.

Nuances:

- the namespace must match the path (`src/Plugin/Block/X.php` ↔ `Drupal\module\Plugin\Block`), otherwise the plugin/class will not be found;
- `dependencies` in `info.yml` and in the YAML configuration must be consistent, otherwise the module will not install;
- the module name = the prefix of all hooks (`drupaljira_timelog_cron`) and services (`drupaljira_timelog.task_stat`);
- scaffolding generation: `drush generate module`, `drush generate controller|form|plugin:block|service`.

### 20.2. Content type

**Via the admin UI.** Structure → Content types → Add → name (machine name `task`), settings: "Create new revision", "Display author information", preview → save. Then fields, form, display (20.3). Permissions: `/admin/people/permissions` → "Task: Create new content" and so on.

**In code (config).** File `config/install/node.type.task.yml`:

```yaml
langcode: en
status: true
dependencies: {}
name: Task
type: task
description: 'Work item for a project.'
help: ''
new_revision: true
preview_mode: 1
display_submitted: false
```

Installing the module will create the type automatically.

**How it is done in practice:** create it in the admin UI → `drush cex` → copy the needed files `node.type.task.yml`, `field.storage.*`, `field.field.*`, `core.entity_form_display.*`, `core.entity_view_display.*` into the module's `config/install` → **remove the `uuid` and `_core` keys from them** (otherwise a UUID conflict arises on installation on another site).

Nuances:

- roles, Views, vocabularies, text formats are created on the same principle: "made it in the UI → exported → put into the module";
- files in `config/install` are applied **only when the module is installed**; on an already installed site an update hook or a regular `config:import` is needed (Block 3);
- if the type has fields, their storage and instance must be installed earlier/together: dependencies (`dependencies.config`) set the order;
- permissions for the content type (`create task content`, `edit own task content`, `edit any task content`, `delete ...`) are created automatically when the type appears.

### 20.3. Field

**Via the admin UI.** Content type → Manage fields → Add field → type (number decimal, list list_string, reference entity reference) → machine name `field_estimate` → storage settings (cardinality, precision/scale, allowed values of the list) → field settings (label, required, default, allowed bundles for a reference) → Manage form display (widget, order) → Manage display (formatter, label above/hidden, order). Do not forget `drush cex`.

**In code — via configuration.** Four files:

```yaml
# field.storage.node.field_estimate.yml
langcode: en
status: true
dependencies:
  module: [node]
id: node.field_estimate
field_name: field_estimate
entity_type: node
type: decimal
settings: { precision: 10, scale: 2 }
module: core
locked: false
cardinality: 1
translatable: true
indexes: {}
persist_with_no_fallback: false
custom_storage: false
```

```yaml
# field.field.node.task.field_estimate.yml
langcode: en
status: true
dependencies:
  config: [field.storage.node.field_estimate, node.type.task]
id: node.task.field_estimate
field_name: field_estimate
entity_type: node
bundle: task
label: 'Estimate (hours)'
description: ''
required: false
translatable: false
default_value: {  }
default_value_callback: ''
settings: { min: 0, max: null, prefix: '', suffix: '' }
field_type: decimal
```

Plus `core.entity_form_display.node.task.default.yml` (the `number` widget) and `core.entity_view_display.node.task.default.yml` (the `number_decimal` formatter). Without the display files the field exists but is **not visible** either in the form or on output.

**In code — PHP.** For an existing site (update hook):

```php
FieldStorageConfig::create([
  'field_name' => 'field_estimate', 'entity_type' => 'node', 'type' => 'decimal',
  'settings' => ['precision' => 10, 'scale' => 2],
])->save();
FieldConfig::create([
  'field_name' => 'field_estimate', 'entity_type' => 'node', 'bundle' => 'task', 'label' => 'Estimate',
])->save();
\Drupal::service('entity_display.repository')
  ->getFormDisplay('node', 'task')->setComponent('field_estimate', ['type' => 'number'])->save();
\Drupal::service('entity_display.repository')
  ->getViewDisplay('node', 'task')->setComponent('field_estimate', ['type' => 'number_decimal'])->save();
```

A **base field** — only by code (`baseFieldDefinitions()` of your own entity; for someone else's — `hook_entity_base_field_info()`; after adding to an existing entity an update hook with `EntityDefinitionUpdateManager::installFieldStorageDefinition()` is needed).

Nuances:

- configurable field: once data appears, the type and some storage settings cannot be changed;
- `list_string`: the allowed values are stored in the storage; **you cannot change the keys (machine values) without migrating the data**, change the labels;
- for displaying a base field `setDisplayConfigurable()`/`setDisplayOptions()` is needed;
- translation: the field has a `translatable` flag in the storage and a separate checkbox for the bundle in `/admin/config/regional/content-language`.

### 20.4. Role and permissions

**Admin UI.** People → Roles → Add role → `/admin/people/permissions` (per role or the general matrix) → tick the permissions.

**Code.**

1. Declare permissions: `drupaljira_timelog.permissions.yml` (Block 7). Dynamic permissions — `permission_callbacks: ['\Drupal\mod\Class::method']`.
2. Grant to a role: in `config/install/user.role.project_manager.yml` with a `permissions` block, or programmatically:

```php
user_role_grant_permissions('authenticated', ['access content']);   // in hook_install / an update hook
Role::load('project_manager')->grantPermission('manage project tasks')->save();
```

3. Assign a role to a user: `$user->addRole('project_manager'); $user->save();`.

Nuances:

- declaring ≠ granting: without granting it, nobody has it except the administrator (the administrator role gets all permissions automatically, `is_admin`);
- `user.role.*` in `config/install` must have `dependencies` on the modules that declare the permissions, otherwise there is an error on installation;
- permissions and roles are configuration, transferred via `cex`/`cim`; assigning a role to a user is content, not transferred.

### 20.5. Route, page, menu

**Admin UI.** Routes are created only by code (or Views — a page/block). A menu item: Structure → Menus → add a link (stored as `menu_link_content` content, not as configuration).

**Code.**

```yaml
# drupaljira_timelog.routing.yml
drupaljira_timelog.task_list:
  path: '/tasks'
  defaults:
    _controller: '\Drupal\drupaljira_timelog\Controller\TaskController::list'
    _title: 'Tasks'
  requirements:
    _permission: 'access content'
```

```yaml
# drupaljira_timelog.links.menu.yml — a menu item
drupaljira_timelog.tasks:
  title: 'Tasks'
  route_name: drupaljira_timelog.task_list
  parent: system.admin_content
  weight: 10
```

```yaml
# drupaljira_timelog.links.task.yml — a tab
drupaljira_timelog.tab_list:
  title: 'List'
  route_name: drupaljira_timelog.task_list
  base_route: drupaljira_timelog.task_list
```

```yaml
# drupaljira_timelog.links.action.yml — an "Add" button
drupaljira_timelog.add:
  title: 'Add task'
  route_name: drupaljira_timelog.task_add
  appears_on: [drupaljira_timelog.task_list]
```

Nuances:

- after adding a route/link — `drush cr` (the route and menu cache);
- menu links from `*.links.menu.yml` are plugins (changed by alter hooks), links from the admin UI are content;
- the `{node}` parameter + `options.parameters.node.type: entity:node` gives upcasting, but **does not check the bundle** — check it manually;
- `_title_callback` is needed for a dynamic title.

### 20.6. Block

**Admin UI.** Structure → Block layout → "Place block" in the needed theme region → choose a block (system or from a module) → title, visibility (pages, content types, roles), region → save. For editable content: Content → Blocks → "Add block" (a custom block, which is a content entity), then place it.

**Code.** The plugin `src/Plugin/Block/TaskSummaryBlock.php` (Block 11) → after `drush cr` it appears in the "Place block" list. The **placement** itself is configuration `block.block.<id>.yml` (theme, region, plugin, visibility); in code:

```php
Block::create(['id' => 'task_summary', 'theme' => 'olivero', 'region' => 'sidebar',
  'plugin' => 'task_summary', 'settings' => ['label' => 'Summary']])->save();
```

Nuances:

- the block plugin and the block placement are **two different things**: there is one plugin, there can be many placements, each with its own settings and visibility;
- a block is placed **in a theme** (the placement is bound to the theme); when the theme changes, blocks must be placed again;
- block cache: `getCacheContexts()`, `getCacheTags()`, `getCacheMaxAge()` or `#cache` in `build()`;
- block access permissions: `blockAccess()`.

### 20.7. View

**Admin UI.** Structure → Views → Add view → show "Content" of type `task`, sorting → format (table/list/grid), fields, filters, sorts, relationships, contextual filters, pager → additionally: a page (path, menu), a block, or REST export → save → `drush cex`.

**Code.** File `config/install/views.view.task_list.yml` (exported). Extending: Views plugins (field/filter/sort/argument handlers), `hook_views_data()` (connect your own table/entity), `hook_views_data_alter()` (add a field to someone else's data), `hook_views_query_alter()`, `hook_views_pre_render()`.

Nuances:

- Views has its own cache (by tags/time), configured in "Advanced → Caching";
- the format chosen in Views "Fields" vs. "Content" (display via a view mode) affects performance;
- a view with a contextual filter requires a correct default argument value.

### 20.8. A custom entity: the full flow

1. Interface and class `src/Entity/TimeLog.php` (`#[ContentEntityType]`, Blocks 10/16).
2. `baseFieldDefinitions()` (fields) + entity keys.
3. Handlers: list_builder, form, access, views_data, route_provider (Block 16).
4. `links` (addresses) + `admin_permission` / `permissions.yml`.
5. Menu/button: `links.menu.yml`, `links.action.yml`, `links.task.yml`.
6. Enable the module: tables are created automatically (`drush en`); `drush cr`.
7. Fields via Field UI — only if `field_ui_base_route` is set (and the entity is "fieldable").
8. Check: `/admin/content/time-log`, permissions, the add form, deletion.
9. Tests: Kernel (CRUD, access) and Functional (pages).

**Changing an already installed entity:** if a base field was added/changed → on `/admin/reports/status` "Mismatched entity and/or field definitions" appears → an update hook is needed:

```php
function drupaljira_timelog_update_10002() {
  $definition = BaseFieldDefinition::create('string')->setLabel(t('Comment'));
  \Drupal::entityDefinitionUpdateManager()
    ->installFieldStorageDefinition('comment', 'time_log', 'drupaljira_timelog', $definition);
}
```

If data already exists and the type is changing — a data migration in a batch (`$sandbox`).

### 20.9. Service, form, event, plugin: brief "create" flows

|What|Files|Next|
|---|---|---|
|Service|a class in `src/Service`, an entry in `services.yml`|inject via `arguments`/`create()` → `drush cr`|
|Form|a class in `src/Form`, a route with `_form`|`drush cr`|
|Subscriber|a class in `src/EventSubscriber` + a service with the `event_subscriber` tag|`drush cr`|
|Plugin (block, worker, formatter)|a class in `src/Plugin/<Type>` with an attribute|`drush cr`|
|Permissions|`permissions.yml`|grant to roles|
|Update hook|`.install`, `hook_update_N()`|`drush updb`|
|Hook|a function in `.module` (or an OOP `#[Hook]`)|`drush cr`|

### 20.10. How to override (a cheat sheet by object)

|What to override|How|
|---|---|
|**Service** (replace the class)|`ServiceProvider`: class `Drupal\mod\ModServiceProvider extends ServiceProviderBase`, `alter(ContainerBuilder $c) { $c->getDefinition('x')->setClass(Y::class); }`. Or a decorator `decorates:` in `services.yml`|
|**Route** (path, permissions, controller)|`RouteSubscriber extends RouteSubscriberBase`, `alterRoutes(RouteCollection $c)`: `$c->get('entity.node.canonical')->setRequirement('_permission', '...')`, `->setDefault('_controller', ...)`; a service with the `event_subscriber` tag|
|**Controller**|replace `_controller` on the route (see above)|
|**Entity class**|`hook_entity_type_alter()`: `$entity_types['node']->setClass(MyNode::class)`; for a bundle — bundle classes (`hook_entity_bundle_info_alter`, the `class` key)|
|**Entity handler** (list, form, access)|`hook_entity_type_alter()`: `->setHandlerClass('list_builder', X::class)`, `setFormClass('default', X::class)`, `setAccessClass(X::class)`|
|**Plugin** (block, formatter, widget…)|`hook_block_alter()`, `hook_field_formatter_info_alter()`, `hook_field_widget_info_alter()`, in general `hook_<type>_info_alter()` — change `class` in the definition|
|**Form**|`hook_form_FORM_ID_alter()`; `hook_form_BASE_FORM_ID_alter()`; or replacing the form class via `hook_entity_type_alter()`|
|**Field widget/formatter**|`hook_field_widget_single_element_form_alter()`, `hook_field_formatter_third_party_settings_form`; or your own plugin and selecting it in Manage display|
|**Entity output**|`hook_entity_view_alter()`, `hook_ENTITY_TYPE_view_alter()`, `hook_entity_display_build_alter()`, preprocess|
|**Template**|a copy of `*.html.twig` in the theme; `hook_theme_suggestions_HOOK_alter()` for new names; preprocess for variables|
|**CSS/JS library**|in the theme `libraries-override` / `libraries-extend` (`theme.info.yml`), in a module `hook_library_info_alter()`; the whole page: `hook_page_attachments_alter()`|
|**Entity access permissions**|`hook_entity_access()`, `hook_ENTITY_TYPE_access()`, `hook_node_access()`, `hook_entity_field_access()`, `hook_entity_create_access()`; `AccessResult::forbidden()` overrides allowed|
|**Route permissions**|`RouteSubscriber` + `setRequirement()`; your own `access_check` service|
|**DB/entity query**|`hook_query_alter()`, `hook_query_TAG_alter()` (tags like `node_access`), `hook_views_query_alter()`|
|**Configuration**|UI/`config:set`/import (permanent); `$config[...]` in `settings.php` or `ConfigFactoryOverrideInterface` (substitution on the fly, not saved)|
|**Interface text**|string translation (`/admin/config/regional/translate`), `locale_custom_strings_<lang>`|
|**Entity/field/bundle definitions**|`hook_entity_type_alter()`, `hook_entity_bundle_info_alter()`, `hook_entity_base_field_info_alter()`, `hook_entity_bundle_field_info_alter()`|
|**Menu/tabs/actions**|`hook_menu_links_discovered_alter()`, `hook_menu_local_tasks_alter()`, `hook_menu_local_actions_alter()`, `hook_link_alter()`, `hook_entity_operation_alter()`|
|**Permissions (declarations)**|permissions from someone else's `permissions.yml` are not changed; they are granted/revoked on roles|
|**Emails**|`hook_mail_alter()`|
|**Response/headers**|a subscriber to `KernelEvents::RESPONSE`|

Order of choice: config → alter hook → event → decorator/ServiceProvider → theme → patch.

### 20.11. Nuances and pitfalls (summary)

**Cache and updating definitions**

- after `services.yml`, routing, plugin, hook, entity definition — `drush cr`; Twig changes in dev require `twig.config` (debug, auto_reload) or a cache clear;
- definitions of plugins/entities/fields are cached separately, `cr` clears them.

**Configuration**

- `config/install` works only on installation; `config/optional` — if dependencies exist;
- when moving files from `cex` into a module, remove `uuid` and `_core`;
- synchronization changes only configuration; content (nodes, terms, menu links `menu_link_content`, custom blocks) — does not.

**Entities and fields**

- `status` ≠ `field_status` ≠ moderation state;
- changing the schema after installation — an update hook; otherwise "Mismatched definitions";
- for a configurable field, once data exists you cannot change the type and some storage settings;
- deleting a field starts background data cleanup (by cron), not an instant one.

**Access**

- permission ≠ entity access; route access ≠ entity access;
- `accessCheck(FALSE)` in a query bypasses permissions — only deliberately;
- `AccessResult::forbidden()` is stronger than `allowed()`; `neutral` does not allow.

**Blocks, forms, themes**

- a block plugin and its placement are different objects; the placement is bound to a theme;
- in a form alter, an added field must be saved by yourself in submit;
- a Twig template in a theme overrides a module template; first enable Twig debug and find the file name.

**Environments**

- `settings.php` — environment and secrets, config — shared; overrides do not end up in the export;
- `drush cim` without `cst` can delete objects; the site UUID must match.

**Security and reliability**

- do not store secrets in YAML and code; `getenv()`;
- CSRF for state-changing routes; `|raw` only for trusted content;
- queue and cron logic is idempotent; batch and update hooks are in portions.

> [!question] Interview questions 
> **How to create a content type and a field so that they transfer between environments?** Create them in the admin UI, export with `drush cex`, put the YAML into the module's `config/install` (removing `uuid` and `_core`) or transfer them via `config/sync`; for existing sites — `cim` or an update hook.
> 
> **How is a configurable field created in code?** With the files `field.storage.*`, `field.field.*`, plus `core.entity_form_display.*` and `core.entity_view_display.*`; without the display files the field is not visible.
> 
> **How to override a service?** A decorator (`decorates`) or a `ServiceProvider` with `alter(ContainerBuilder)`.
> 
> **How to change the permissions or controller of someone else's route?** `RouteSubscriber::alterRoutes()`.
> 
> **How to replace the class of someone else's entity or its handler?** `hook_entity_type_alter()` (`setClass`, `setHandlerClass`, `setFormClass`).
> 
> **How to replace the implementation of someone else's plugin?** `hook_<type>_info_alter()` (for example, `hook_block_alter()`): change `class` in the definition.
> 
> **How does a block plugin differ from a placed block?** The plugin is the block's code, the placement is configuration (`block.block.*`) with the theme, region, and visibility; one plugin can have many placements.
> 
> **Why does "nothing happen" after changing routing/services/a plugin?** Definitions are cached: `drush cr` is needed.

---

## Appendix. Basic Topics the Blocks Rely On

### A. Routing and Controllers

```yaml
drupaljira_timelog.task_view:
  path: '/task/{node}'
  defaults:
    _controller: '\Drupal\drupaljira_timelog\Controller\TaskController::view'
    _title: 'Task'
  requirements:
    _permission: 'access content'
    node: \d+
  options:
    parameters:
      node: { type: entity:node }
```

- `{node}` + `type: entity:node` — a parameter converter (upcasting): the controller receives a `NodeInterface`, not an ID;
- `requirements` — `_permission`, `_entity_access`, `_custom_access`, `_csrf_token`, `_role`;
- `methods: [POST]` — restricting methods;
- a controller returns a render array or a `Response`, extends `ControllerBase`, dependencies — `create()`.

### B. Render API, Twig, and the Theme layer

Output is built from render arrays (`#theme`, `#markup`, `#type`, `#cache`, `#attached`); Drupal collects cache metadata (bubbling) and libraries. Templates are Twig (`*.html.twig`), variables are prepared in preprocess, theme hooks are registered by `hook_theme()`. CSS/JS libraries — `*.libraries.yml` and `#attached['library']`. Twig's auto-escaping is the basis of XSS protection.

### C. Hooks: procedural and OOP

Hooks are named extension points (`hook_entity_presave()`, `hook_form_alter()`, `hook_cron()`, `hook_theme()`). Traditionally functions in `.module`; in Drupal 11.1+ OOP hooks are methods with the `#[Hook('entity_presave')]` attribute (with DI via the constructor). In `.install`: `hook_install()`, `hook_uninstall()`, `hook_schema()`, `hook_update_N()` (the number, for example `10001`, determines the order), `hook_post_update_NAME()` (when all services and entities are available). Run: `drush updb`.

### D. Views

A configurable query builder: fields, filters, sorts, relationships (JOIN), contextual filters, modes, formats (table, list, JSON via REST export). Views are config entities (YAML), extended by plugins and `hook_views_data()` (for custom entities such as `time_log`). Cached by tags and time.

### E. Content Moderation and Workflows

The `workflows` and `content_moderation` modules add states (draft, published) and transitions, take roles and revisions into account. The **moderation state** is stored separately from the publication field `status` and the custom `field_status`: in migrations and update hooks (`10001`) all three are aligned.

### F. Tools

- **Composer**: `composer require drupal/module`, `composer update`, `composer audit`.
- **Drush**: `cr`, `updb`, `cex`/`cim`, `cron`, `queue:run`, `migrate:*`, `watchdog:show`, `sql:cli`, `php:eval`, `uli`.
- **Logs**: `logger.factory` (`$this->getLogger('drupaljira_timelog')->error(...)`), modules `dblog`/`syslog`.
- **Environment**: DDEV/Lando, `settings.local.php`, `development.services.yml` (disabling the cache for development).

---

## Final Cheat Sheet: What Is Responsible for What

|You need…|Use|
|---|---|
|to store settings and transfer them|Configuration Management|
|reusable logic with dependencies|Service + DI|
|a reaction to an action without tight coupling|Event + Subscriber (or hook)|
|to create, find, change data|Entity API (storage, Entity Query)|
|your own list, form, permissions, addresses of an entity|Entity Handlers|
|the structure of entity data|Field API|
|a value derived from other data|Computed field or service|
|a replaceable component (block, widget, worker)|Plugin API|
|a form or a settings form|Form API (`FormBase`, `ConfigFormBase`)|
|a long operation with progress|Batch API|
|regular maintenance|Cron|
|deferred independent jobs|Queue|
|transferring data from an external source|Migration API|
|transforming an existing site on update|update hook|
|an API for the frontend|JSON:API / REST Resource / a custom route|
|who is making the request|Authentication API|
|what they are allowed to do|Permissions + Access API|
|not to serve someone else's or stale data from the cache|cache contexts, tags, max-age|
|several languages|Language, Content/Interface/Config Translation|
|several sites on one code|Multisite|
|verifying correctness|Unit → Kernel → Functional|
|changing the label, form, HTML of someone else's module|alter hooks, preprocess, Twig|
|a secret or a value of one environment|`settings.php` and environment variables|