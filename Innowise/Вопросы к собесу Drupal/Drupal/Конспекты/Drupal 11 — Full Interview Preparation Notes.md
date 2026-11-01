---
custom-width: 80
---
---

> Complete theory, practical implementation, PHP examples, YAML configuration, Drush commands, and interview-ready answers.

---

# 1. Multilingual

## 1.1. Theory

Drupal provides a multilingual system that allows websites to support multiple languages for content, interface elements, and configuration.

Drupal has five main multilingual modules:

|Module|Responsibility|
|---|---|
|Language|Add and manage languages|
|Interface Translation|Translate interface strings|
|Content Translation|Translate content entities and fields|
|Configuration Translation|Translate configuration values|
|Language|Provides language-related functionality|

The Language module is responsible for managing available languages and language negotiation.

### Language negotiation

Language negotiation determines which language Drupal should use for the current request.

Common methods:

- URL prefix: `/en/about`, `/de/about`.
    
- Domain: `example.com`, `example.de`.
    
- Browser language preferences.
    
- User account language preference.
    
- Session or selected language.
    

Language negotiation determines the language; it does not automatically translate content.

### Content translation

Drupal supports translations of entities such as:

- Nodes.
    
- Taxonomy terms.
    
- Users.
    
- Custom content entities.
    

To translate an entity, its entity type and bundle must support translations, and Content Translation must be enabled and configured.

Fields can be configured as translatable or non-translatable.

For example:

- Title: translatable.
    
- Body: translatable.
    
- Internal reference: potentially non-translatable.
    

### Interface translation

The Interface Translation module translates interface strings.

Example:

```php
$this->t('Create a new task');
```

The `t()` function is used for translatable interface text, not for translating entity field values.

### Configuration translation

Configuration Translation allows supported configuration values to have translations.

Examples:

- Views labels.
    
- Menu labels.
    
- Block labels.
    
- Site configuration text.
    

### Entity translation API

Drupal provides methods for working with entity translations:

```php
$node->hasTranslation('de');

$translated_node = $node->getTranslation('de');

$node->addTranslation('de', [
  'title' => 'Neuer Titel',
]);

$original = $node->getUntranslated();
```

Important distinctions:

- `hasTranslation()` checks whether a translation exists.
    
- `getTranslation()` retrieves an existing translation.
    
- `addTranslation()` creates a translation.
    
- `getUntranslated()` retrieves the untranslated entity.
    

`addTranslation()` does not automatically translate field values. You must provide the translated values.

### Field translatability

For base fields in code:

```php
$fields['title']->setTranslatable(TRUE);
```

For configurable fields, translatability is configured through field storage and field instance configuration, subject to Drupal's field and entity capabilities.

### Language cache contexts

Drupal must distinguish cached content according to language.

Common cache contexts:

```php
languages:language_interface
languages:language_content
```

These ensure that cached output can vary depending on the interface or content language.

## 1.2. Practical workflow

To enable multilingual content:

1. Enable Language, Interface Translation, Content Translation, and Configuration Translation.
    
2. Add the required languages.
    
3. Configure language negotiation.
    
4. Enable content translation for the required entity bundle.
    
5. Mark the required fields as translatable.
    
6. Create the original content.
    
7. Add translations through the administration interface or Entity API.
    
8. Verify translations and cache behavior.
    

## 1.3. Common problems

If a translation is missing, check:

- Is Content Translation enabled?
    
- Does the entity type support translations?
    
- Is translation enabled for the bundle?
    
- Is the field translatable?
    
- Does the entity actually have the requested translation?
    
- Is the correct content language selected?
    
- Are cache contexts configured correctly?
    

## 1.4. Interview answer

Drupal provides several modules for multilingual functionality.

The Language module manages available languages and language negotiation. Interface Translation is responsible for translating interface strings, Content Translation allows us to translate content entities and fields, and Configuration Translation handles translatable configuration values.

Language negotiation determines which language Drupal should use for a request. It can use URL prefixes, domains, browser preferences, or user preferences.

To make content translatable, I need to enable Content Translation for the appropriate entity bundle and configure the required fields as translatable.

Drupal also provides Entity API methods such as `hasTranslation()`, `getTranslation()`, and `addTranslation()` to work with translations programmatically.

I also need to consider cache contexts to make sure Drupal serves the correct language-specific content.

---

# 2. REST API and JSON

## 2.1. Theory

An API is an interface that allows different software components to communicate.

In Drupal, APIs are commonly used to connect:

- React or Vue applications.
    
- Mobile applications.
    
- External services.
    
- Third-party systems.
    

Drupal provides REST functionality and JSON as different approaches to exposing data.

### REST API

REST is an architectural style for designing network APIs.

Common HTTP methods:

|Method|Purpose|
|---|---|
|GET|Retrieve data|
|POST|Create a resource or execute an operation|
|PATCH|Partially update a resource|
|PUT|Replace a resource, depending on the API contract|
|DELETE|Delete a resource|

Drupal's core REST module provides configurable REST resources.

The Serialization module handles serialization and deserialization of supported data formats.

REST configuration includes:

- Enabled resources.
    
- HTTP methods.
    
- Request and response formats.
    
- Authentication providers.
    
- Permissions.
    

Enabling the REST module does not automatically expose every entity or operation.

### JSON

JSON is a standardized specification for building JSON-based APIs.

Drupal provides a JSON module in core.

It exposes supported entity types through standardized endpoints.

Example:

```json
GET /jsonapi/node/article
```

Retrieve an individual article:

```json
GET /jsonapi/node/article/{uuid}
```

A JSON response commonly contains:

```json
{
  "data": [
    {
      "type": "node--article",
      "id": "article-uuid",
      "attributes": {
        "title": "Drupal Development",
        "status": true
      },
      "relationships": {}
    }
  ]
}
```

Main response elements:

- `data`: returned resources.
    
- `type`: entity resource type.
    
- `id`: resource UUID.
    
- `attributes`: entity attributes and fields.
    
- `relationships`: references to related resources.
    
- `included`: optionally included related resources.
    

### JSON filtering

Filtering allows the frontend to retrieve only matching resources.

Example:

```json
GET /jsonapi/node/article?filter[status]=1
```

More complex filters can use conditions:

```json
GET /jsonapi/node/article?filter[title-filter][condition][path]=title
```

Sparse fieldsets allow selecting specific fields:

```json
GET /jsonapi/node/article?fields[node--article]=title,body
```

Pagination can limit the number of returned resources.

Example:

```json
GET /jsonapi/node/article?page[limit]=10
```

These mechanisms reduce unnecessary data transfer.

### REST vs. JSON

|REST|JSON|
|---|---|
|Configurable resources|Standardized entity resources|
|Suitable for custom operations|Suitable for entity CRUD|
|Custom response structures possible|Standardized response structure|
|Requires resource configuration|Standard entity endpoints|
|Useful for custom business workflows|Useful for decoupled frontend applications|

### When to use each

Use JSON when:

- A React frontend needs to retrieve articles.
    
- The frontend needs to filter entities.
    
- Standard entity CRUD operations are sufficient.
    
- You want a standardized API structure.
    

Use a custom REST resource when:

- You need custom business logic.
    
- One request must execute several operations.
    
- You need an application-specific response.
    
- The operation does not correspond to ordinary entity CRUD.
    

A custom REST endpoint should delegate business logic to a Drupal service rather than putting all logic inside the endpoint class.

## 2.2. Practical implementation

Enable JSON:API:

```php
drush en jsonapi -y
drush cr
```

Retrieve articles:

```php
curl -i https://example.com/jsonapi/node/article
```

Retrieve published articles:

```json
GET /jsonapi/node/article?filter[status]=1
```

A custom REST resource generally requires:

1. A PHP resource plugin.
    
2. Route and plugin configuration through the resource plugin definition.
    
3. Authentication configuration.
    
4. Appropriate permissions and access checks.
    
5. Request validation.
    
6. A JSON response.
    

## 2.3. Authentication and authorization

Authentication identifies the user.

Authorization determines what the user can do.

Drupal supports different authentication mechanisms, including:

- Session-based authentication.
    
- Basic authentication, when configured.
    
- Other contributed or custom authentication providers.
    

For example:

- Anonymous users can read published articles.
    
- Editors can modify articles.
    
- Administrators can manage all content.
    

Drupal must enforce access on the server.

Hiding an edit button in React is not a security mechanism.

A user can still send an HTTP request manually, so Drupal must validate access when processing the request.

### HTTP status codes

|Status|Meaning|
|---|---|
|200|Successful request|
|201|Resource created|
|204|Successful request without response body|
|400|Invalid request|
|401|Authentication required or failed|
|403|Access denied|
|404|Resource not found|
|500|Internal server error|

## 2.4. Debugging a 403 response

Check:

1. Is the endpoint correct?
    
2. Is the relevant API module enabled?
    
3. Is the current user authenticated when necessary?
    
4. Does the user have the required permissions?
    
5. Does entity access allow the operation?
    
6. Is the entity published?
    
7. Are custom access rules blocking the request?
    
8. What do the response body and Drupal logs report?
    

A 403 usually indicates insufficient access, while a 401 often indicates an authentication issue. Exact responses can depend on the endpoint and authentication provider.

## 2.5. Interview answer

Drupal provides REST functionality and JSON for exposing application data through HTTP endpoints.

JSON is a standardized solution for working with Drupal entities. It provides endpoints for retrieving, filtering, creating, updating, and deleting supported entities.

I would prefer JSON when a decoupled frontend needs standard entity operations because it reduces custom development and provides a consistent response structure.

For more complex business operations, I would implement a custom REST resource. For example, an endpoint could update an article status, create a log entry, and trigger a notification.

I would keep the business logic inside a dedicated Drupal service and call that service from the REST resource.

I would also configure authentication and authorization properly. Drupal must validate permissions and entity access on the server for every protected operation.

---

# 3. Configuration Management

## 3.1. Theory

Drupal Configuration Management allows developers to export, import, and version-control configuration.

Configuration describes how the Drupal application is structured and behaves.

Examples:

- Content types.
    
- Field definitions.
    
- Views.
    
- Roles and permissions.
    
- Text formats.
    
- Image styles.
    
- Entity form displays.
    
- Entity view displays.
    
- Module configuration.
    

Configuration is generally stored in YAML files.

Example:

```php
name: Article
type: node
description: 'An article content type.'
```

The actual configuration files generated by Drupal contain additional identifiers, dependencies, and settings.

### Configuration vs. Content

This distinction is very important.

|Configuration|Content|
|---|---|
|Content type definitions|Article nodes|
|Field definitions|Field values|
|Views|Content records|
|Roles|User accounts|
|Image styles|Uploaded images|
|Display configuration|Content data|

Configuration defines the structure; content represents the actual information stored in the system.

### Configuration synchronization

Drupal uses a synchronization directory to store exported configuration.

The directory is defined in `settings.php`.

Example:

```php
$settings['config_sync_directory'] = '../config/sync';
```

The exact path depends on the project.

### Export configuration

```php
drush cex
```

Full command:

```php
drush config:export
```

Exported YAML files should be reviewed and committed to Git.

### Import configuration

```php
drush cim
```

Full command:

```php
drush config:import
```

This imports configuration from the synchronization directory into the active configuration storage.

### Configuration workflow

```
Local development
       |
       v
Make configuration changes
       |
       v
drush cex
       |
       v
YAML files
       |
       v
Git commit and push
       |
       v
Pull changes on staging
       |
       v
drush cim
       |
       v
Verify configuration
```

### Useful Drush commands

```php
drush cex
drush cim
drush config:status
drush config:get system.site
drush config:export
drush config:import
drush cr
```

`config:status` checks differences between active configuration and the synchronization directory.

`config:get` displays a configuration object.

### Configuration dependencies

Configuration objects may depend on:

- Modules.
    
- Other configuration objects.
    
- Content entities.
    
- Plugins.
    
- Field storage definitions.
    

Drupal validates these dependencies during configuration import.

### Configuration UUID

Each Drupal site has a UUID.

Configuration synchronization normally requires the source and destination sites to have matching site UUIDs.

This prevents accidental synchronization between unrelated Drupal sites.

### Configuration schema

Configuration schema describes the expected structure and data types of configuration values.

Invalid configuration can cause validation problems.

### Configuration splits and exclusions

The Configuration Split contributed module allows environments to use different configuration sets.

For example:

- Development-specific configuration.
    
- Staging-specific configuration.
    
- Production-specific configuration.
    

Configuration exclusions can also prevent selected configuration from being synchronized.

## 3.2. Practical workflow

Local:

```php
drush cex -y

git status

git add config/sync/

git commit -m "Update Drupal configuration"

git push
```

Staging:

```php
git pull

drush cim -y

drush cr
```

Before importing into production, review the configuration changes and understand their effects.

### Debugging missing configuration

Useful commands:

```php
drush config:status
drush config:get field.field.node.article.field_subtitle
drush cr
```

Check:

- Was the YAML file committed?
    
- Was the correct synchronization directory used?
    
- Did the import finish successfully?
    
- Are module dependencies enabled?
    
- Are configuration dependencies valid?
    
- Is the field attached to the correct bundle?
    
- Is the field enabled in the form display?
    
- Are Git merge conflicts resolved?
    
- Are database updates required?
    

Remember that a field can exist in configuration but remain hidden in the editing form.

Check:

```php
core.entity_form_display.node.article.default
```

This configuration controls the Article default form display.

## 3.3. Interview answer

Drupal Configuration Management allows developers to manage application configuration through YAML files and synchronize it between environments.

After making configuration changes locally, I use `drush cex` to export configuration into the synchronization directory.

Next, I review the generated YAML files, commit them to Git, and push the changes to the repository.

On staging, I pull the changes and run `drush cim` to import the configuration.

This approach makes configuration deployments reproducible and allows configuration changes to be tracked and reviewed through version control.

I distinguish configuration from content because configuration defines the application's structure, while content represents actual data stored in the database.

---

# 4. Services and Dependency Injection

## 4.1. Theory

Drupal uses a service container to manage reusable application services.

A service is an object responsible for specific functionality.

Examples:

- Entity Type Manager.
    
- Database Connection.
    
- Current User.
    
- Logger.
    
- Cache Backend.
    
- Custom business services.
    

Services help organize code and separate responsibilities.

### Service container

The service container stores service definitions and creates service objects when required.

Services are usually defined in a module's `.services.yml` file.

Example:

```php
services:
  my_module.task_manager:
    class: Drupal\my_module\Service\TaskManager
```

The service can then be retrieved from the container.

### Dependency Injection

Dependency Injection is a design pattern in which an object receives its dependencies from an external source instead of creating them directly.

Example:

```php
public function __construct(
  EntityTypeManagerInterface $entity_type_manager,
) {
  $this->entityTypeManager = $entity_type_manager;
}
```

The class declares what it needs, and Drupal provides the dependency.

### Why use Dependency Injection?

Advantages:

- Better testability.
    
- Lower coupling.
    
- Explicit dependencies.
    
- Easier mocking.
    
- Better maintainability.
    
- More reusable code.
    

### Static service access

Drupal also provides global service access:

```php
$entity_type_manager = \Drupal::entityTypeManager();

$logger = \Drupal::service('logger.factory');
```

This approach is convenient but creates stronger coupling to the global container.

Use constructor injection whenever the class supports it.

Static access may be acceptable in procedural code, hooks, or situations where injection is unavailable or impractical.

## 4.2. Register a custom service

File:

`my_module.services.yml`

```php
services:
  my_module.task_manager:
    class: Drupal\my_module\Service\TaskManager
    arguments:
      - '@entity_type.manager'
```

The `arguments` section defines dependencies.

`@entity_type.manager` references another registered service.

## 4.3. Implement the service

File:

`src/Service/TaskManager.php`

```php
<?php

namespace Drupal\my_module\Service;

use Drupal\Core\Entity\EntityTypeManagerInterface;

final class TaskManager {

  public function __construct(
    private readonly EntityTypeManagerInterface $entityTypeManager,
  ) {}

  public function loadTask(int $id): ?NodeInterface {
    return $this->entityTypeManager
      ->getStorage('node')
      ->load($id);
  }

}
```

For this example, the class must also import `Drupal\node\NodeInterface`.

## 4.4. Inject a service into a controller

Controllers commonly use the `ContainerInjectionInterface` pattern through `ControllerBase`.

```php
<?php

namespace Drupal\my_module\Controller;

use Drupal\Core\Controller\ControllerBase;
use Drupal\Core\Entity\EntityTypeManagerInterface;
use Drupal\my_module\Service\TaskManager;
use Symfony\Component\DependencyInjection\ContainerInterface;

final class TaskController extends ControllerBase {

  public function __construct(
    private readonly EntityTypeManagerInterface $entityTypeManager,
    private readonly TaskManager $taskManager,
  ) {}

  public static function create(ContainerInterface $container): static {
    return new static(
      $container->get('entity_type.manager'),
      $container->get('my_module.task_manager'),
    );
  }

}
```

The `create()` method retrieves dependencies from the service container and passes them into the constructor.

## 4.5. Dependency Injection vs. static service access

|Dependency Injection|Static service access|
|---|---|
|Constructor receives dependencies|Class accesses global container|
|Explicit dependencies|Hidden dependencies|
|Easier unit testing|Harder unit testing|
|Lower coupling|Higher coupling|
|Preferred in classes|Useful in procedural contexts|

## 4.6. Service replacement and decoration

Drupal allows developers to modify existing services.

### Service replacement

You can provide a different implementation under an existing service ID.

The replacement must preserve the expected contract and compatibility with existing consumers.

### Service decoration

Decoration wraps an existing service with another service.

It is useful when you want to add behavior without completely replacing the original implementation.

Example:

```php
services:
  my_module.task_manager_decorator:
    class: Drupal\my_module\Decorator\TaskManagerDecorator
    decorates: my_module.task_manager
    arguments:
      - '@my_module.task_manager_decorator.inner'
```

The decorator receives the original service through the `.inner` reference.

### Interview answer

Drupal uses a service container to manage reusable application services.

Dependency Injection is a design pattern where a class receives its dependencies instead of creating them directly.

In Drupal, I can register custom services in a `.services.yml` file and inject them through the constructor.

For controllers, I can implement the `create()` method to retrieve services from the container and pass them into the constructor.

I prefer Dependency Injection because it makes dependencies explicit, improves testability, and reduces coupling to the global container.

Drupal also supports service decoration, which allows developers to wrap existing services and extend their behavior without completely replacing the original implementation.

---

# 5. Performance and Scalability

## 5.1. Theory

Drupal performance depends on several areas:

- Database queries.
    
- PHP execution.
    
- Caching.
    
- Rendering.
    
- HTTP requests.
    
- External services.
    
- Infrastructure.
    
- Frontend assets.
    

The goal is to reduce unnecessary computation, database operations, and network traffic.

## 5.2. Drupal caching

Drupal provides several caching mechanisms.

### Render Cache

Stores rendered output to avoid rebuilding the same render arrays repeatedly.

### Dynamic Page Cache

Caches dynamic pages for users with different permissions and personalized content, while respecting cache metadata.

### Internal Page Cache

Caches complete pages for anonymous users.

### Cache Backend

Drupal can use different cache backends, such as database storage or Redis through a contributed integration.

## 5.3. Cache metadata

Drupal uses three important cache metadata concepts.

### Cache Tags

Cache tags identify what data a cached item depends on.

Example:

```php
$build['#cache']['tags'] = [
  'node:12',
];
```

When the corresponding node changes, Drupal can invalidate cache entries with that tag.

### Cache Contexts

Cache contexts define variations in cached output.

Examples:

```php
user
user.roles
languages:language_interface
url.query_args
```

For example, if output depends on the current user's role, the cache must vary by `user.roles`.

### Cache Max-Age

Defines how long an item can remain cached.

```php
$build['#cache']['max-age'] = 3600;
```

A value of `0` means the output is not cacheable.

### Cache bubbling

Cache metadata can bubble from child render arrays to parent render arrays.

This allows Drupal to collect cache contexts, tags, and max-age values from nested components.

## 5.4. Database performance

Avoid unnecessary queries.

A common problem is the N+1 query problem.

Example:

```php
1 query to load 100 tasks
100 additional queries to load their projects
```

This creates 101 queries instead of a smaller number of optimized queries.

Possible solutions:

- Entity reference loading strategies.
    
- Entity query.
    
- Batch loading.
    
- Appropriate database queries.
    
- Caching.
    
- Efficient Views configuration.
    

## 5.5. Entity Query

Example:

```php
$query = $entity_type_manager
  ->getStorage('node')
  ->getQuery();

$ids = $query
  ->condition('type', 'task')
  ->condition('status', 1)
  ->accessCheck(TRUE)
  ->execute();
```

Entity Query provides a Drupal abstraction for querying entities.

Use access checks where appropriate. If you intentionally need an administrative or system-level query, understand the security implications before disabling access checking.

## 5.6. BigPipe

BigPipe improves perceived page loading performance.

It allows Drupal to send the initial page structure while replacing placeholders with dynamic content as it becomes available.

This can improve the user experience when some components are expensive or personalized.

## 5.7. Frontend optimization

Common techniques:

- Aggregate CSS and JavaScript.
    
- Avoid unnecessary libraries.
    
- Optimize images.
    
- Use responsive image styles.
    
- Lazy-load images where appropriate.
    
- Reduce unnecessary HTTP requests.
    

## 5.8. Interview answer

Drupal provides several caching layers, including Internal Page Cache, Dynamic Page Cache, and Render Cache.

To configure caching correctly, I need to understand cache tags, contexts, and max-age.

Cache tags define which data a cache entry depends on. Cache contexts define how cached output varies, for example by user role or language. Max-age defines how long the cached output remains valid.

I would also optimize database queries and avoid the N+1 query problem by using efficient entity loading, queries, and caching.

For frontend performance, I would optimize images, aggregate assets, and avoid unnecessary JavaScript and CSS.

I would use profiling and logs to identify bottlenecks before applying optimizations.

---

# 6. Testing

## 6.1. Theory

Testing verifies that application functionality works correctly and prevents regressions.

Drupal supports several testing approaches.

### Unit Tests

Unit tests verify individual classes or methods in isolation.

They commonly use mocks for dependencies.

Example:

```php
public function testCalculateTotal(): void {
  $result = $this->calculator->calculate(10, 20);

  $this->assertEquals(30, $result);
}
```

### Kernel Tests

Kernel tests verify Drupal functionality with a minimal bootstrapped Drupal environment.

They can test:

- Entity storage.
    
- Database operations.
    
- Services.
    
- Plugins.
    
- Configuration.
    
- Entity APIs.
    

### Functional Tests

Functional tests verify application behavior through HTTP requests and browser-like interactions.

They can test:

- Forms.
    
- Routes.
    
- Permissions.
    
- Page rendering.
    
- User workflows.
    

### Functional JavaScript Tests

These tests execute browser JavaScript and are useful for AJAX and interactive interfaces.

### Browser testing

Tools such as Playwright can automate browser workflows.

Example:

```php
test('user can open task board', async ({ page }) => {
  await page.goto('/board');

  await expect(page).toHaveTitle(/Drupal/);
});
```

## 6.2. Test pyramid

A common testing strategy:

```
       E2E Tests
      Functional Tests
       Kernel Tests
        Unit Tests
```

Unit tests are generally faster and more isolated.

End-to-end tests verify complete user workflows but are usually slower and more expensive to maintain.

## 6.3. Mocking

Mocking allows us to replace dependencies with controlled test objects.

Dependency Injection makes mocking easier because dependencies are passed into constructors.

Example:

```php
$entity_manager = $this->createMock(
  EntityTypeManagerInterface::class
);
```

## 6.4. Drupal testing tools

Common tools:

- PHPUnit.
    
- Drupal Test Traits.
    
- BrowserTestBase.
    
- KernelTestBase.
    
- UnitTestCase.
    
- Playwright.
    
- PHPStan.
    
- PHPCS.
    

## 6.5. Practical testing workflow

```
Implement feature
      |
      v
Write unit tests
      |
      v
Run static analysis
      |
      v
Run kernel / functional tests
      |
      v
Run browser tests
      |
      v
Review failures
      |
      v
Fix and retest
```

## 6.6. Interview answer

Drupal supports several testing levels.

Unit tests verify individual classes and methods in isolation. Kernel tests verify Drupal functionality with a bootstrapped environment, including services, entities, and database operations.

Functional tests verify application behavior through HTTP requests, forms, routes, and permissions. Functional JavaScript tests can also verify interactive frontend behavior.

I prefer writing unit tests for business logic and using kernel or functional tests when I need to verify integration with Drupal.

Dependency Injection makes testing easier because I can replace services with mocks.

I also use static analysis tools such as PHPStan and coding standards tools such as PHPCS to detect problems before runtime.

---

# 7. Security

## 7.1. Theory

Drupal provides a security framework that protects content, users, forms, routes, and API operations.

Security must be enforced on the server, not only in the frontend.

Main security areas:

- Authentication.
    
- Authorization.
    
- Access control.
    
- Input validation.
    
- Output escaping.
    
- CSRF protection.
    
- SQL injection prevention.
    
- XSS prevention.
    
- Secure configuration.
    

## 7.2. Permissions and roles

Roles group permissions.

Examples:

- Anonymous user.
    
- Authenticated user.
    
- Editor.
    
- Administrator.
    

Permissions define what operations a role can perform.

Examples:

- Access content.
    
- Create article content.
    
- Edit own article content.
    
- Edit any article content.
    
- Delete article content.
    

Drupal checks permissions and entity access before allowing protected operations.

## 7.3. Route access

A route can define access requirements.

Example:

```php
my_module.admin:
  path: '/admin/custom'
  defaults:
    _controller: '\Drupal\my_module\Controller\AdminController::index'
  requirements:
    _permission: 'administer site configuration'
```

A route can also use entity access requirements or custom access check services.

## 7.4. Entity access

Entity access checks determine whether a user can perform an operation on a specific entity.

Example:

```php
$access = $node->access('update', $account);
```

The operation can be:

- `view`
    
- `update`
    
- `delete`
    

Entity access can depend on permissions, ownership, publication state, and custom access logic.

## 7.5. CSRF protection

CSRF attacks attempt to make a user perform an unwanted operation through an authenticated session.

Drupal provides CSRF protection mechanisms for appropriate routes and requests.

For custom state-changing routes, configure the correct route requirements and authentication approach.

Do not assume that every POST request is automatically safe.

## 7.6. XSS prevention

Cross-Site Scripting occurs when untrusted content is executed as JavaScript in a browser.

Drupal uses Twig autoescaping.

Prefer:

```php
{{ variable }}
```

Avoid bypassing escaping without a strong reason.

For markup that must be rendered as safe HTML, use Drupal's render and filtering APIs appropriately.

## 7.7. SQL injection

Never concatenate untrusted user input directly into SQL queries.

Use parameterized queries:

```php
$result = $database->select('users_field_data', 'u')
  ->fields('u', ['uid', 'name'])
  ->condition('uid', $uid)
  ->execute();
```

For entity data, prefer Entity API where appropriate.

## 7.8. Input validation

Validate all incoming data.

For example:

- Required values.
    
- Data types.
    
- Allowed values.
    
- Length limits.
    
- Entity references.
    
- User permissions.
    

Do not trust frontend validation alone.

## 7.9. Interview answer

Drupal provides a security framework based on authentication, permissions, roles, route access, and entity access.

Authentication identifies the user, while authorization determines which operations the user can perform.

I would configure permissions for each role and use entity access checks to control access to individual content entities.

For custom routes, I would configure appropriate access requirements and validate all incoming data.

I would also protect against common vulnerabilities such as XSS, SQL injection, and CSRF by using Drupal's APIs, parameterized database queries, Twig autoescaping, and appropriate CSRF protection.

Security must always be enforced on the server, even when the frontend already hides restricted operations.

---

# 8. Multisite

## 8.1. Theory

Drupal Multisite allows multiple Drupal websites to run from one codebase.

Each site can have its own:

- Database.
    
- Configuration.
    
- Files directory.
    
- Domain.
    
- Content.
    
- Users.
    

The sites share the Drupal core and contributed/custom codebase, but their application data and configuration are separated.

## 8.2. Directory structure

A simplified structure:

```
web/
  core/
  modules/
  themes/
  sites/
    default/
    example.com/
      settings.php
      files/
    second.example.com/
      settings.php
      files/
```

Drupal determines which site to load based on the requested hostname and multisite configuration.

## 8.3. Shared code

All multisite instances use the same codebase.

This means:

- Updating Drupal core affects all sites.
    
- Updating shared modules affects all sites.
    
- Shared themes can be used by multiple sites.
    

Each site can enable different modules and have different configuration.

## 8.4. Advantages

- Shared codebase.
    
- Centralized updates.
    
- Reduced code duplication.
    
- Easier management of related websites.
    

## 8.5. Disadvantages

- Shared code changes can affect multiple sites.
    
- Deployments require careful coordination.
    
- Different sites may need different module versions or functionality.
    
- A shared infrastructure failure can affect multiple websites.
    

## 8.6. Multisite vs. separate installations

|Multisite|Separate installations|
|---|---|
|Shared codebase|Independent codebases|
|Separate databases/configuration|Separate databases/configuration|
|Centralized code updates|Independent updates|
|Lower code duplication|Greater isolation|
|Shared dependency versions|Independent dependency versions|

## 8.7. Interview answer

Drupal Multisite allows multiple websites to run from a single Drupal codebase.

Each website can have its own database, configuration, files directory, and domain, while sharing Drupal core, modules, and themes.

The main advantage is centralized code management and reduced duplication.

However, shared code changes can affect all sites, so deployments and updates require careful testing.

I would use Multisite when websites have similar technical requirements and can share the same codebase and dependency versions.

If websites require completely independent development cycles or different dependencies, separate Drupal installations may be a better solution.

---

# 9. Events API

## 9.1. Theory

Drupal uses the Symfony Event Dispatcher component.

The Events API allows developers to react to events that occur during application execution.

Examples:

- Entity insertion.
    
- Entity update.
    
- Entity deletion.
    
- Request processing.
    
- Response processing.
    
- Kernel events.
    

An event represents something that happened.

The Event Dispatcher dispatches events to registered listeners.

An Event Subscriber defines which events a class listens to.

## 9.2. Event Subscriber

A subscriber must implement:

```php
EventSubscriberInterface
```

The interface requires:

```php
public static function getSubscribedEvents(): array;
```

This method defines the subscribed events and corresponding callback methods.

## 9.3. Practical implementation

Example subscriber:

```php
<?php

namespace Drupal\my_module\EventSubscriber;

use Drupal\Core\Entity\EntityEvents;
use Drupal\Core\Entity\EntityInsertEvent;
use Symfony\Component\EventDispatcher\EventSubscriberInterface;

final class NodeInsertSubscriber implements EventSubscriberInterface {

  public static function getSubscribedEvents(): array {
    return [
      EntityEvents::INSERT => 'onEntityInsert',
    ];
  }

  public function onEntityInsert(EntityInsertEvent $event): void {
    $entity = $event->getEntity();

    if ($entity->getEntityTypeId() === 'node') {
      // Execute custom logic.
    }
  }

}
```

Register it:

```php
services:
  my_module.node_insert_subscriber:
    class: Drupal\my_module\EventSubscriber\NodeInsertSubscriber
    tags:
      - { name: event_subscriber }
```

Rebuild cache:

```php
drush cr
```

Drupal discovers the subscriber through the service tag.

## 9.4. Custom events

Developers can also create and dispatch custom events.

A custom event class can carry data required by subscribers.

The dispatcher can be injected as a service:

```php
public function __construct(
  private readonly EventDispatcherInterface $dispatcher,
) {}
```

Dispatching a custom event:

```php
$this->dispatcher->dispatch(
  $event,
  'my_module.task_completed'
);
```

Subscribers can listen to that event by its name.

## 9.5. Hooks vs. Events API

|Hooks|Events|
|---|---|
|Drupal-specific|Symfony Event Dispatcher|
|Functions|Classes|
|Hook discovery|Service registration|
|Drupal extension points|Dispatched events|
|Often used for altering Drupal behavior|Often used for reacting to events|

The choice does not depend on the number of subscribers.

Hooks and events can sometimes react to similar application operations.

## 9.6. Interview answer

The Drupal Events API allows developers to implement event-driven functionality.

An event represents something that happens in the application. The Event Dispatcher dispatches the event to registered listeners.

I would implement an Event Subscriber by creating a class that implements `EventSubscriberInterface`.

In the `getSubscribedEvents()` method, I would define the events I want to listen to and the corresponding callback methods.

Then I would register the subscriber in the module's `.services.yml` file using the `event_subscriber` tag.

For example, I could subscribe to an entity insertion event and execute custom logic when a new node is created.

This approach separates event-related logic from the main application code and supports dependency injection.

---

# 10. Entity API

## 10.1. Theory

The Entity API is one of the most important parts of Drupal.

Entities represent structured data in Drupal.

Examples:

- Nodes.
    
- Users.
    
- Taxonomy terms.
    
- Files.
    
- Media.
    
- Custom entities.
    

Drupal provides a common API for creating, loading, updating, deleting, and querying entities.

## 10.2. Content Entities

Content entities represent actual application data.

Examples:

- Articles.
    
- Users.
    
- Products.
    
- Tasks.
    
- Time logs.
    

They are generally stored in database tables and can contain fields.

Content entities usually support:

- Revisions.
    
- Translations.
    
- Field values.
    
- Entity access.
    
- Entity forms and displays.
    

## 10.3. Configuration Entities

Configuration entities represent configuration objects managed by Drupal.

Examples:

- Content type definitions.
    
- Views.
    
- Image styles.
    
- Vocabulary definitions.
    
- User roles.
    

They are managed through Configuration Management and can be exported to YAML.

### Content Entities vs. Configuration Entities

|Content Entity|Configuration Entity|
|---|---|
|Stores application data|Stores application configuration|
|Usually database-backed content|Exportable configuration|
|Often has fields and revisions|Often defines application behavior|
|Examples: Node, User|Examples: View, Content Type|

## 10.4. Entity Type and Bundle

An entity type defines the general structure and behavior of an entity.

A bundle represents a subtype or variation of an entity type.

Example:

```php
Entity Type: Node

Bundles:
- Article
- Page
- Task
- Project
```

All are node entities, but they can have different fields and configuration.

## 10.5. Loading entities

Use the Entity Type Manager:

```php
$node = $entity_type_manager
  ->getStorage('node')
  ->load(12);
```

Load multiple entities:

```php
$nodes = $entity_type_manager
  ->getStorage('node')
  ->loadMultiple([12, 13, 14]);
```

## 10.6. Creating entities

```php
$node = $entity_type_manager
  ->getStorage('node')
  ->create([
    'type' => 'article',
    'title' => 'Drupal Entity API',
    'status' => 1,
  ]);

$node->save();
```

## 10.7. Updating entities

```php
$node = $entity_type_manager
  ->getStorage('node')
  ->load(12);

if ($node) {
  $node->setTitle('Updated title');
  $node->save();
}
```

## 10.8. Deleting entities

```php
$node = $entity_type_manager
  ->getStorage('node')
  ->load(12);

if ($node) {
  $node->delete();
}
```

## 10.9. Entity Query

Entity Query provides an abstraction for querying entities.

```php
$query = $entity_type_manager
  ->getStorage('node')
  ->getQuery();

$ids = $query
  ->condition('type', 'article')
  ->condition('status', 1)
  ->accessCheck(TRUE)
  ->execute();
```

Entity Query can filter entities by field values and other properties.

## 10.10. Entity Access

Entity access checks whether the current user can perform an operation.

```php
$node->access('view');
$node->access('update');
$node->access('delete');
```

Access checks are important for security.

## 10.11. Creating a custom Content Entity

A custom content entity commonly uses:

- An entity class.
    
- An entity type definition.
    
- Entity keys.
    
- Base fields.
    
- Storage configuration.
    
- Access handlers.
    
- List builders.
    
- Forms.
    
- Routing and permissions where required.
    

Drupal 11 supports attribute-based entity definitions.

Simplified example:

```php
<?php

namespace Drupal\my_module\Entity;

use Drupal\Core\Entity\ContentEntityBase;
use Drupal\Core\Entity\EntityTypeInterface;
use Drupal\Core\Entity\Attribute\ContentEntityType;
use Drupal\Core\Field\BaseFieldDefinition;

#[ContentEntityType(
  id: 'time_log',
  label: new \Drupal\Core\StringTranslation\TranslatableMarkup('Time log'),
  base_table: 'time_log',
  entity_keys: [
    'id' => 'id',
    'uuid' => 'uuid',
  ],
)]
class TimeLog extends ContentEntityBase {

  public static function baseFieldDefinitions(
    EntityTypeInterface $entity_type
  ): array {
    $fields = parent::baseFieldDefinitions($entity_type);

    $fields['id'] = BaseFieldDefinition::create('integer')
      ->setLabel(t('ID'))
      ->setReadOnly(TRUE);

    $fields['uuid'] = BaseFieldDefinition::create('uuid')
      ->setLabel(t('UUID'))
      ->setReadOnly(TRUE);

    return $fields;
  }

}
```

This is an illustrative starting point, not a complete production entity definition. A production entity also needs the appropriate handlers, additional base fields, access configuration, and potentially revision and ownership support.

### Entity Handlers

Entity handlers provide specialized behavior for entity operations.

Common handlers:

- Storage handler.
    
- Access handler.
    
- View builder.
    
- List builder.
    
- Form handlers.
    

## 10.12. Interview answer

The Entity API provides a common way to work with structured data in Drupal.

Entities can represent content, such as nodes and users, or configuration, such as views and content type definitions.

Content Entities store application data, while Configuration Entities describe how the application is configured.

To work with entities programmatically, I use the Entity Type Manager and storage handlers to load, create, update, and delete entities.

For example, I can load a node using `getStorage('node')->load($id)`.

If I need a custom entity type, I define an entity class, specify its entity type definition and entity keys, and implement its base fields.

I also need to consider storage, access handlers, forms, and other entity handlers depending on the requirements.

---

# 11. Plugin API

## 11.1. Theory

The Plugin API allows Drupal to provide extensible functionality through reusable components.

A plugin is a self-contained class that implements a specific behavior defined by a plugin type.

Examples:

- Blocks.
    
- Field formatters.
    
- Field widgets.
    
- Views plugins.
    
- Queue workers.
    
- Image effects.
    
- Actions.
    

A plugin type defines the contract that individual plugins must follow.

## 11.2. Plugin Manager

A Plugin Manager discovers, creates, and manages plugins of a particular type.

It is responsible for:

- Plugin discovery.
    
- Plugin definitions.
    
- Plugin instantiation.
    
- Plugin factories.
    

## 11.3. Plugin Discovery

Drupal supports different plugin discovery mechanisms.

Common mechanisms:

- PHP attributes.
    
- Annotations in older Drupal implementations.
    
- YAML definitions.
    
- Derivatives.
    

For modern Drupal 11 custom plugins, attributes are generally preferred where the plugin type supports them.

## 11.4. Example: Custom Block Plugin

```php
<?php

namespace Drupal\my_module\Plugin\Block;

use Drupal\Core\Block\BlockBase;
use Drupal\Core\Block\Attribute\Block;
use Drupal\Core\StringTranslation\TranslatableMarkup;

#[Block(
  id: 'my_custom_block',
  admin_label: new TranslatableMarkup('My custom block')
)]
class MyCustomBlock extends BlockBase {

  public function build(): array {
    return [
      '#markup' => $this->t('Hello from my custom block!'),
    ];
  }

}
```

The plugin manager discovers the block definition.

The block can then be placed through the Drupal administration interface.

## 11.5. Plugin vs. Service

|Plugin|Service|
|---|---|
|Implements a specific plugin type|Provides reusable application functionality|
|Discovered by plugin manager|Registered in service container|
|Has plugin definition|Has service definition|
|Examples: Block, Formatter|Examples: Logger, Task Manager|

A plugin can use Dependency Injection when its plugin type supports a container factory.

For example, a custom block can implement `ContainerFactoryPluginInterface` to receive services through its constructor and `create()` method.

## 11.6. Interview answer

The Plugin API allows Drupal to provide extensible and reusable functionality.

A plugin is a class that implements a specific behavior defined by a plugin type.

Examples include blocks, field formatters, field widgets, and queue workers.

Drupal uses Plugin Managers to discover and instantiate plugins. Plugin definitions can be provided through attributes, YAML, and other supported discovery mechanisms.

I would create a custom plugin when I need to implement functionality that should integrate with an existing Drupal extension system.

A service is different because it provides reusable application functionality through the service container, while a plugin implements a specific plugin type.

---

# 12. Batch API, Cron, and Queue API

## 12.1. Batch API

Batch API allows Drupal to process large operations in smaller steps.

It is useful when an operation takes a long time or involves many records.

Examples:

- Updating thousands of nodes.
    
- Importing large datasets.
    
- Processing many files.
    
- Performing bulk operations.
    

Instead of processing everything in one request, Drupal divides the work into operations.

### Batch process structure

A batch definition commonly contains:

- Operations.
    
- Processing callbacks.
    
- Finished callback.
    
- Progress information.
    

Simplified example:

```php
$batch = [
  'title' => t('Processing tasks'),
  'operations' => [
    [
      '\Drupal\my_module\Batch\TaskBatch::process',
      [],
    ],
  ],
  'finished' => '\Drupal\my_module\Batch\TaskBatch::finished',
];

batch_set($batch);
```

The callback processes a portion of the work and updates the context.

## 12.2. Cron API

Cron is Drupal's scheduled task system.

It allows Drupal and modules to perform periodic operations.

Examples:

- Cleaning temporary data.
    
- Checking scheduled content.
    
- Processing periodic maintenance.
    
- Executing module-specific scheduled tasks.
    

Drupal can run cron through the administration interface or Drush.

```php
drush cron
```

Cron execution can be triggered externally through a server scheduler.

## 12.3. Queue API

Queue API allows tasks to be stored and processed asynchronously.

It is useful when an operation does not need to finish during the original user request.

Examples:

- Sending emails.
    
- Processing large imports.
    
- Generating reports.
    
- Calling external APIs.
    
- Processing background jobs.
    

A queue stores items that workers process later.

### Queue Worker

A Queue Worker is a plugin responsible for processing queue items.

Simplified example:

```php
<?php

namespace Drupal\my_module\Plugin\QueueWorker;

use Drupal\Core\Queue\QueueWorkerBase;
use Drupal\Core\Queue\Attribute\QueueWorker;
use Drupal\Core\StringTranslation\TranslatableMarkup;

#[QueueWorker(
  id: 'my_module_task_worker',
  title: new TranslatableMarkup('Task worker'),
)]
class TaskWorker extends QueueWorkerBase {

  public function processItem($data): void {
    // Process the queue item.
  }

}
```

Queue workers can be processed through cron or manually using appropriate queue processing mechanisms.

## 12.4. Batch vs. Cron vs. Queue

|API|Main purpose|
|---|---|
|Batch API|Process long operations in manageable steps|
|Cron|Execute scheduled periodic tasks|
|Queue API|Store and process background tasks|

### When to use each

Use Batch API when the user starts a large operation and needs visible progress.

Use Cron when tasks must run periodically.

Use Queue API when work should be processed asynchronously or independently of the original request.

## 12.5. Interview answer

Drupal provides Batch API, Cron API, and Queue API for different types of background and long-running operations.

Batch API divides a large operation into smaller steps and provides progress information. I would use it for operations such as updating thousands of entities through an administrative interface.

Cron is used for scheduled tasks, such as maintenance and periodic processing.

Queue API allows developers to store tasks and process them asynchronously using queue workers.

I would use Queue API for operations such as sending emails, processing imports, or communicating with external services when the operation should not block the original request.

The main difference is that Batch API focuses on long-running operations with progress, Cron handles scheduled execution, and Queue API handles asynchronous task processing.

---

# 13. Migration API

## 13.1. Theory

Migration API allows developers to transfer data from external sources or older systems into Drupal.

Common migration sources:

- CSV files.
    
- SQL databases.
    
- Older Drupal installations.
    
- JSON APIs.
    
- XML files.
    

Migration usually involves three main stages:

1. Source.
    
2. Process.
    
3. Destination.
    

### Source

Defines where the original data comes from.

Examples:

- CSV file.
    
- Database table.
    
- Drupal database.
    
- External API.
    

### Process

Transforms source data into the structure required by the destination.

Examples:

- Mapping fields.
    
- Converting values.
    
- Looking up related entities.
    
- Changing text formats.
    
- Transforming identifiers.
    

### Destination

Defines where migrated data is stored.

Examples:

- Node entities.
    
- Users.
    
- Taxonomy terms.
    
- Custom entities.
    

## 13.2. Migration configuration

Migration definitions are commonly stored in YAML files.

Example:

```php
id: article_migration

label: 'Article migration'

source:
  plugin: csv
  path: modules/custom/my_module/data/articles.csv
  header_row_count: 1
  ids:
    - id

process:
  title: title
  body/value: body
  status:
    plugin: default_value
    default_value: 1

destination:
  plugin: 'entity:node'
  default_bundle: article
```

This example requires the relevant migration source plugin and destination configuration to be available.

## 13.3. Process plugins

Process plugins transform source values.

Common examples:

- `default_value`
    
- `concat`
    
- `skip_on_empty`
    
- `migration_lookup`
    
- `entity_generate`
    
- `format_date`
    

Example:

```php
process:
  status:
    plugin: default_value
    default_value: 1
```

### Migration Lookup

`migration_lookup` resolves references to entities created by another migration.

For example, an article migration can use a previously migrated author or taxonomy term.

## 13.4. Migration dependencies

Migrations can depend on other migrations.

Example:

```php
migration_dependencies:
  required:
    - user_migration
    - taxonomy_migration
```

This ensures required entities are migrated before dependent entities.

## 13.5. Drush commands

Common migration commands:

```php
drush migrate:status

drush migrate:import article_migration

drush migrate:messages article_migration

drush migrate:rollback article_migration

drush migrate:reset-status article_migration
```

The Migrate and Migrate Drupal modules are part of Drupal core, while Migrate Plus and Migrate Tools are commonly used contributed modules for additional functionality and management commands.

## 13.6. Migration workflow

```php
Identify source data
       |
       v
Define source plugin
       |
       v
Map fields
       |
       v
Configure process plugins
       |
       v
Define destination
       |
       v
Run migration
       |
       v
Review messages
       |
       v
Validate migrated entities
```

## 13.7. Interview answer

Drupal Migration API allows developers to transfer data from external systems or older Drupal installations.

A migration consists of three main parts: Source, Process, and Destination.

The Source defines where the data comes from. The Process stage transforms and maps the source values into the required Drupal structure. The Destination defines where the migrated data will be stored.

Migration definitions are commonly configured through YAML files.

I can use process plugins to transform values, set defaults, skip invalid records, and resolve references between entities.

I would use Drush commands to inspect migration status, run migrations, review errors, and roll back imported data when necessary.

For complex migrations, I would also define migration dependencies to ensure that referenced entities are migrated first.

---

# 14. Field API

## 14.1. Theory

Field API allows Drupal entities to store structured data through fields.

Examples:

- Text.
    
- Integer.
    
- Decimal.
    
- Boolean.
    
- Date.
    
- Entity Reference.
    
- File.
    
- Image.
    

Fields can be attached to entity bundles.

For example:

```php
Node
  |
  +-- Article
  |     +-- field_subtitle
  |     +-- field_author
  |
  +-- Task
        +-- field_status
        +-- field_estimate
```

## 14.2. Field Storage and Field Instance

Drupal distinguishes field storage definitions from field instances.

### Field Storage

Defines the underlying field structure.

Examples:

- Field type.
    
- Cardinality.
    
- Storage settings.
    
- Database storage.
    

### Field Instance

Defines how a field is attached to a particular entity bundle.

Examples:

- Label.
    
- Required setting.
    
- Default value.
    
- Bundle-specific settings.
    

A field storage definition can be shared by multiple bundles where appropriate.

## 14.3. Base Fields and Configurable Fields

### Base Fields

Defined in code by the entity type.

Example:

```php
$fields['status'] = BaseFieldDefinition::create('boolean')
  ->setLabel(t('Status'))
  ->setDefaultValue(TRUE);
```

### Configurable Fields

Created through Drupal configuration or the administration interface.

They can be exported through Configuration Management.

Examples:

- `field_project`
    
- `field_assignee`
    
- `field_estimate`
    

## 14.4. Entity Reference Fields

Entity Reference fields connect one entity to another.

Example:

```php
Task
  |
  +-- field_project
          |
          v
        Project
```

A configurable entity reference field can be configured to reference nodes of a specific bundle.

This is useful for relationships between content entities.

## 14.5. Cardinality

Cardinality defines how many values a field can store.

Examples:

- `1`: one value.
    
- `3`: maximum three values.
    
- `-1`: unlimited values.
    

## 14.6. Field Widgets

Widgets define how users enter field values in forms.

Examples:

- Text field.
    
- Select list.
    
- Autocomplete.
    
- Checkbox.
    
- Date picker.
    

## 14.7. Field Formatters

Formatters define how field values are displayed.

Examples:

- Plain text.
    
- Linked entity.
    
- Image.
    
- Date formatting.
    
- Custom formatter.
    

## 14.8. Computed Fields

Computed fields derive their values from other data instead of storing an independent value in the normal way.

Example use cases:

- Calculated totals.
    
- Derived labels.
    
- Formatted values.
    
- Aggregated information.
    

Computed fields should be designed carefully because their values can affect caching and performance.

## 14.9. Interview answer

Field API allows Drupal entities to store structured data through fields.

A field can be attached to an entity bundle and can contain values such as text, numbers, dates, images, or references to other entities.

Drupal distinguishes between field storage and field instances. Field storage defines the underlying field structure, while the field instance attaches that field to a particular bundle and defines settings such as labels and required status.

Drupal also supports base fields, which are defined in code, and configurable fields, which are generally managed through configuration.

Widgets control how users enter field values, while formatters control how those values are displayed.

I can also use Entity Reference fields to connect entities, for example connecting a Task to a Project.

---

# 15. Authentication API

## 15.1. Theory

Authentication verifies the identity of a user or client.

Drupal supports multiple authentication mechanisms.

Examples:

- Session authentication.
    
- Basic authentication.
    
- Cookie-based authentication.
    
- HTTP authentication.
    
- Custom authentication providers.
    
- OAuth-based authentication through contributed integrations.
    

Authentication is different from authorization.

Authentication answers:

"Who are you?"

Authorization answers:

"What are you allowed to do?"

## 15.2. Session Authentication

Drupal commonly uses session-based authentication for browser users.

After successful login, Drupal maintains an authenticated session.

The browser sends the session cookie with subsequent requests.

## 15.3. Basic Authentication

Basic authentication sends a username and password through an HTTP Authorization header.

It must only be used over HTTPS.

It requires the appropriate Drupal authentication provider and configuration.

It is not automatically enabled for every API endpoint.

## 15.4. API Authentication

For API integrations, the authentication method depends on the architecture.

Possible approaches:

- Session-based authentication.
    
- OAuth 2.0 through a suitable integration.
    
- HTTP Basic Authentication where appropriate.
    
- Custom authentication provider.
    

For production systems, prefer secure token-based authentication where it fits the integration requirements.

Never store credentials in frontend source code.

## 15.5. Authorization

After authentication, Drupal checks permissions and access rules.

Examples:

- View published content.
    
- Create articles.
    
- Edit own content.
    
- Edit any content.
    
- Administer configuration.
    

Entity access can further restrict operations on individual entities.

## 15.6. Interview answer

Drupal Authentication API provides mechanisms for identifying users and validating requests.

Authentication determines who is making the request, while authorization determines what operations that user can perform.

Drupal supports session-based authentication and can use other authentication providers, such as Basic Authentication or OAuth-based integrations.

For API integrations, I would choose an authentication mechanism based on the frontend architecture and security requirements.

After authentication, Drupal must check the user's permissions and entity access before allowing protected operations.

For example, anonymous users can be allowed to read published articles, while editors must authenticate and have the appropriate permissions to modify content.

---

# 16. Entity Handlers

## 16.1. Theory

Entity Handlers provide specialized functionality for working with entity types.

They allow Drupal to separate entity responsibilities into different classes.

Common handlers:

- Storage Handler.
    
- Access Handler.
    
- View Builder.
    
- List Builder.
    
- Form Handler.
    

## 16.2. Storage Handler

Responsible for entity persistence and loading.

Example:

```php
$storage = $entity_type_manager->getStorage('node');

$node = $storage->load(12);
```

Common methods:

```php
load()
loadMultiple()
create()
save()
delete()
```

## 16.3. Access Handler

Responsible for entity access checks.

Example:

```php
$access = $node->access('update');
```

Access handlers determine whether operations are allowed.

## 16.4. View Builder

Responsible for rendering entities.

Example:

```php
$view_builder = $entity_type_manager->getViewBuilder('node');

$build = $view_builder->view($node);
```

## 16.5. Form Handler

Responsible for building entity forms.

Examples:

- Add forms.
    
- Edit forms.
    
- Delete forms.
    

## 16.6. List Builder

Responsible for building administrative entity listing pages.

For example, custom entities can use a list builder to display records in an administration table.

## 16.7. Entity Type Manager

Entity Type Manager is the main service for accessing entity definitions and handlers.

Example:

```php
$entity_type_manager = \Drupal::entityTypeManager();

$storage = $entity_type_manager->getStorage('node');

$view_builder = $entity_type_manager->getViewBuilder('node');

$access_handler = $entity_type_manager->getAccessControlHandler('node');
```

In classes, prefer injecting `EntityTypeManagerInterface`.

## 16.8. Interview answer

Entity Handlers provide specialized functionality for working with Drupal entity types.

The Storage Handler is responsible for loading, creating, saving, and deleting entities.

The Access Handler manages access checks and determines whether users can view, update, or delete entities.

The View Builder is responsible for rendering entities, while Form Handlers manage entity forms such as add, edit, and delete forms.

The List Builder is used to create administrative listings of entities.

I usually access these handlers through Entity Type Manager, which provides methods such as `getStorage()`, `getViewBuilder()`, and `getAccessControlHandler()`.

---

# 17. Form API

## 17.1. Theory

Drupal Form API provides a standardized way to build, validate, and process forms.

It handles:

- Form rendering.
    
- Input processing.
    
- Validation.
    
- Submission.
    
- CSRF protection.
    
- Form state.
    
- AJAX interactions.
    

Forms can be created for:

- Content editing.
    
- Configuration.
    
- Custom operations.
    
- Administrative functionality.
    

## 17.2. FormBase

A common base class for custom forms is:

```php
FormBase
```

It provides a basic structure for implementing custom forms.

## 17.3. Form implementation

A custom form usually implements:

- `getFormId()`
    
- `buildForm()`
    
- `validateForm()`
    
- `submitForm()`
    

Example:

```php
<?php

namespace Drupal\my_module\Form;

use Drupal\Core\Form\FormBase;
use Drupal\Core\Form\FormStateInterface;

class TaskForm extends FormBase {

  public function getFormId(): string {
    return 'my_module_task_form';
  }

  public function buildForm(
    array $form,
    FormStateInterface $form_state
  ): array {
    $form['title'] = [
      '#type' => 'textfield',
      '#title' => $this->t('Task title'),
      '#required' => TRUE,
    ];

    $form['submit'] = [
      '#type' => 'submit',
      '#value' => $this->t('Save'),
    ];

    return $form;
  }

  public function validateForm(
    array &$form,
    FormStateInterface $form_state
  ): void {
    if (strlen($form_state->getValue('title')) < 3) {
      $form_state->setErrorByName(
        'title',
        $this->t('The title must contain at least 3 characters.')
      );
    }
  }

  public function submitForm(
    array &$form,
    FormStateInterface $form_state
  ): void {
    $title = $form_state->getValue('title');

    $this->messenger()->addStatus(
      $this->t('Task @title was submitted.', [
        '@title' => $title,
      ])
    );
  }

}
```

## 17.4. Form validation

Validation checks user input before submission.

Examples:

- Required fields.
    
- Minimum length.
    
- Allowed values.
    
- Numeric ranges.
    
- Entity existence.
    
- Business rules.
    

Validation can be implemented in `validateForm()` or through element validation callbacks.

## 17.5. Form submission

The submit handler processes validated values.

It can:

- Create entities.
    
- Update configuration.
    
- Call services.
    
- Redirect users.
    
- Trigger business operations.
    

Business logic should generally be placed in a service rather than directly inside the form class.

## 17.6. Form State

FormStateInterface provides access to submitted values and form processing state.

Example:

```php
$value = $form_state->getValue('title');
```

Set an error:

```php
$form_state->setErrorByName('title', $message);
```

Set a redirect:

```php
$form_state->setRedirect('entity.node.canonical', [
  'node' => $nid,
]);
```

## 17.7. AJAX Forms

Drupal Form API supports AJAX interactions.

Example:

```php
$form['details'] = [
  '#type' => 'container',
  '#attributes' => [
    'id' => 'task-details',
  ],
];
```

An element can define an `#ajax` callback that returns updated render arrays.

## 17.8. Dependency Injection in Forms

Forms can receive dependencies through constructor injection and the `create()` method.

For example, inject Entity Type Manager or a custom service to handle entity operations.

## 17.9. Interview answer

Drupal Form API provides a standardized way to create, validate, and process forms.

A custom form usually extends `FormBase` and implements methods such as `getFormId()`, `buildForm()`, `validateForm()`, and `submitForm()`.

The `buildForm()` method defines form elements and their properties.

The `validateForm()` method checks user input before submission.

The `submitForm()` method processes the validated values.

I can use FormStateInterface to retrieve values, set validation errors, and control form submission.

Drupal Form API also supports AJAX interactions and dependency injection.

For maintainability, I prefer moving complex business logic into services and keeping the form class responsible for form-related behavior.

---

# 18. Working with Someone Else's Code

## 18.1. Theory

When joining an existing Drupal project, the first step is understanding the existing architecture before making changes.

Avoid immediately rewriting code or introducing new patterns without understanding the project.

The goal is to make changes safely while preserving existing functionality.

## 18.2. Initial investigation

I would inspect:

- Drupal version.
    
- PHP version.
    
- Composer dependencies.
    
- Custom modules.
    
- Custom themes.
    
- Configuration.
    
- Routes.
    
- Services.
    
- Plugins.
    
- Entity types.
    
- Existing tests.
    
- Git history.
    

Useful commands:

```php
drush status

composer show drupal/core

drush pml --status=enabled

drush router:debug

git status

git log --oneline -10
```

## 18.3. Understand the architecture

I would identify:

- Where business logic is implemented.
    
- Which services are responsible for specific operations.
    
- How controllers and forms are structured.
    
- How entities are loaded and saved.
    
- How permissions are implemented.
    
- Which APIs and integrations are used.
    

## 18.4. Find existing implementations

Before creating a new solution, search the project for similar functionality.

For example:

```php
grep -R "getStorage('node')" web/modules/custom/
```

This can reveal existing patterns and reusable services.

## 18.5. Testing before changes

Before modifying code:

1. Set up the project locally.
    
2. Install dependencies.
    
3. Import configuration.
    
4. Run database updates if required.
    
5. Rebuild caches.
    
6. Run existing tests.
    
7. Reproduce the problem.
    

This establishes a baseline.

## 18.6. Make controlled changes

Recommended approach:

- Create a separate Git branch.
    
- Understand the affected components.
    
- Implement the smallest appropriate change.
    
- Follow existing coding standards.
    
- Add tests.
    
- Run static analysis.
    
- Review the diff.
    
- Create a pull request.
    

## 18.7. Git workflow

```php
git checkout develop

git pull

git checkout -b fix/task-access

# Implement changes

git status

git diff

# Run tests

git add .

git commit -m "Fix task access control"

git push -u origin fix/task-access
```

Then create a Pull Request for review.

## 18.8. Code quality tools

Common tools:

- PHPCS.
    
- PHPStan.
    
- PHPUnit.
    
- Drush.
    
- Composer.
    
- Git.
    
- Playwright.
    

## 18.9. Interview answer

When working with an existing Drupal project, I first try to understand its architecture and existing implementation patterns.

I inspect the Drupal and PHP versions, Composer dependencies, custom modules, services, routes, entities, configuration, and tests.

Before implementing changes, I try to reproduce the problem and identify the exact component responsible for it.

I also search for existing implementations that can be reused instead of introducing unnecessary duplicate code.

After implementing the change, I run relevant tests, static analysis, and coding standards checks.

Finally, I review the Git diff and create a separate branch and Pull Request so that the changes can be reviewed before merging.

---

# 19. settings.php and Database

## 19.1. Theory

The `settings.php` file contains environment-specific Drupal configuration.

It is located in:

```php
web/sites/default/settings.php
```

For multisite installations, individual sites can have their own settings files.

`settings.php` commonly defines:

- Database connection.
    
- Configuration synchronization directory.
    
- Trusted host patterns.
    
- Hash salt.
    
- File paths.
    
- Environment-specific settings.
    
- Reverse proxy configuration.
    
- Development settings.
    

## 19.2. Database configuration

Drupal uses database connection settings to connect to the database.

Example:

```php
$databases['default']['default'] = [
  'database' => 'drupal',
  'username' => 'drupal',
  'password' => 'password',
  'prefix' => '',
  'host' => 'database',
  'port' => '3306',
  'namespace' => 'Drupal\Core\Database\Driver\mysql',
  'driver' => 'mysql',
];
```

The actual host, database, and credentials depend on the environment.

Do not commit production credentials into Git.

## 19.3. Environment-specific settings

Different environments may require different settings.

Examples:

- Local development.
    
- Staging.
    
- Production.
    

A common approach is to load environment-specific settings from separate files or environment variables.

Example:

```php
$settings['hash_salt'] = getenv('DRUPAL_HASH_SALT');
```

The value should be provided securely by the environment.

## 19.4. Trusted Host Patterns

Trusted host patterns protect against Host header attacks.

Example:

```php
$settings['trusted_host_patterns'] = [
  '^example\.com$',
  '^www\.example\.com$',
];
```

Patterns should match the actual domains used by the installation.

## 19.5. Hash Salt

Drupal uses a hash salt for security-related operations.

Example:

```php
$settings['hash_salt'] = 'generated-random-secret';
```

It should be unique and kept secret.

## 19.6. Database API

Drupal provides a Database API for database operations.

Example:

```php
$database = \Drupal::database();

$query = $database->select('users_field_data', 'u')
  ->fields('u', ['uid', 'name'])
  ->condition('uid', 1);

$result = $query->execute();
```

In object-oriented classes, inject the database connection:

```php
Connection $database
```

through Dependency Injection.

## 19.7. Entity API vs. Database API

Prefer Entity API when working with Drupal entities.

Entity API handles entity-level functionality such as:

- Entity storage.
    
- Fields.
    
- Revisions.
    
- Translations.
    
- Access.
    
- Entity lifecycle.
    

Use Database API when you need lower-level database operations that are not appropriately handled through Entity API.

Avoid direct SQL queries that bypass important entity behavior.

## 19.8. Database backups

A database backup is important before risky deployments, updates, and migrations.

With Drush:

```php
drush sql:dump --result-file=backup.sql
```

Import:

```php
drush sql:cli < backup.sql
```

The exact backup and restore workflow depends on the environment.

Files should also be backed up separately because the database backup does not necessarily include uploaded files.

## 19.9. Interview answer

The `settings.php` file contains environment-specific Drupal configuration.

It commonly defines database connections, configuration synchronization directories, trusted host patterns, hash salt, and other settings.

Database connection settings include the database name, username, password, host, port, and database driver.

I would keep sensitive credentials outside version-controlled configuration, using environment variables or secure deployment configuration.

For database operations, Drupal provides Database API and Entity API.

I prefer Entity API when working with Drupal entities because it respects entity storage, fields, revisions, and access-related functionality.

I use Database API when lower-level database operations are required.

I would also create database backups before risky deployments or migrations and remember that uploaded files may need separate backups.

---

# Final Interview Revision

## Important Drupal concepts

|Concept|Main responsibility|
|---|---|
|Entity API|Work with structured data|
|Field API|Add structured fields to entities|
|Plugin API|Provide extensible components|
|Services|Reusable application functionality|
|Dependency Injection|Pass dependencies into classes|
|Events API|React to dispatched events|
|Hooks|Extend Drupal behavior|
|Form API|Build and process forms|
|REST|Expose custom API resources|
|JSON|Standardized entity API|
|Configuration Management|Synchronize configuration|
|Migration API|Transfer data|
|Batch API|Process long-running operations|
|Cron|Run scheduled tasks|
|Queue API|Process asynchronous tasks|
|Authentication|Identify the user|
|Authorization|Determine allowed operations|
|Cache Tags|Identify cache dependencies|
|Cache Contexts|Define cache variations|
|Cache Max-Age|Define cache lifetime|
|Entity Handlers|Handle specialized entity operations|
|Multisite|Run multiple sites from one codebase|

## Important Drush Commands

```php
# Status
drush status

# Enable modules
drush en module_name -y

# Disable modules
drush pm:uninstall module_name -y

# Clear cache
drush cr

# Export configuration
drush cex

# Import configuration
drush cim

# Configuration status
drush config:status

# Database updates
drush updb

# Run cron
drush cron

# Database backup
drush sql:dump --result-file=backup.sql

# Show enabled modules
drush pml --status=enabled

# Show routes
drush router:debug
```

## Important PHP and Drupal Patterns

### Entity loading

```php
$node = $entity_type_manager
  ->getStorage('node')
  ->load($id);
```

### Entity creation

```php
$node = $entity_type_manager
  ->getStorage('node')
  ->create([
    'type' => 'article',
    'title' => 'Example',
    'status' => 1,
  ]);

$node->save();
```

### Dependency Injection

```php
public function __construct(
  EntityTypeManagerInterface $entity_type_manager,
) {
  $this->entityTypeManager = $entity_type_manager;
}
```

### Entity access

```php
$node->access('update');
```

### Entity Query

```php
$query = $entity_type_manager
  ->getStorage('node')
  ->getQuery();

$ids = $query
  ->condition('type', 'article')
  ->accessCheck(TRUE)
  ->execute();
```

### Translation

```php
$node->hasTranslation('de');

$translated = $node->getTranslation('de');
```

### Logging

```php
$logger->notice('Task @id was updated.', [
  '@id' => $task_id,
]);
```

### Service registration

```php
services:
  my_module.task_manager:
    class: Drupal\my_module\Service\TaskManager
```

---

# Interview Preparation Checklist

- Explain Drupal multilingual architecture.
    
- Explain Language Negotiation.
    
- Understand entity translations.
    
- Compare REST and JSON.
    
- Explain API authentication and authorization.
    
- Debug 401 and 403 responses.
    
- Explain Configuration Management.
    
- Understand `drush cex` and `drush cim`.
    
- Debug configuration import problems.
    
- Explain Services and Dependency Injection.
    
- Implement a custom Drupal service.
    
- Inject dependencies into controllers.
    
- Compare DI with `\Drupal::service()`.
    
- Explain service replacement and decoration.
    
- Explain Events API and Event Subscribers.
    
- Compare Hooks and Event Subscribers.
    
- Explain Content and Configuration Entities.
    
- Create a custom Content Entity.
    
- Explain Entity Handlers.
    
- Explain Plugin API.
    
- Create a custom Block plugin.
    
- Compare Batch, Cron, and Queue.
    
- Explain Migration API.
    
- Understand Field Storage and Field Instances.
    
- Explain authentication mechanisms.
    
- Explain Drupal security mechanisms.
    
- Understand caching and cache metadata.
    
- Explain Drupal testing levels.
    
- Understand Multisite architecture.
    
- Explain Form API.
    
- Understand `settings.php` and database configuration.
    
- Explain how to work safely with an existing codebase.
    

**Preparation strategy:** For every topic, be ready to explain the theory, describe a real use case, write a small code or YAML example, and explain how you would debug a problem. Understanding the implementation is more valuable than memorizing definitions.