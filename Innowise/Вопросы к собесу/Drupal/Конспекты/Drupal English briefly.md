---
custom-width: 80
---
## 1. Drupal Fundamentals

### What is Drupal?

Drupal is an open-source Content Management System (CMS) written in PHP. It is also a flexible framework for building complex websites, APIs, and web applications.

It provides:

- Content management.
    
- User and permission management.
    
- Flexible content modeling.
    
- Configuration management.
    
- REST and JSON.
    
- Extensible architecture through modules, plugins, and services.
    

### What is Drupal Core?

Drupal Core is the main codebase that provides essential functionality, such as:

- Entity API.
    
- Form API.
    
- Plugin API.
    
- Routing system.
    
- Database API.
    
- Configuration API.
    
- Cache API.
    
- User and access systems.
    

### What is a Drupal module?

A module is a package that adds or extends Drupal functionality.

There are three main types:

- **Core modules** — maintained as part of Drupal Core.
    
- **Contributed modules** — developed by the Drupal community.
    
- **Custom modules** — developed specifically for a project.
    

### What is the difference between a module and a theme?

- A module provides functionality and business logic.
    
- A theme controls presentation and the visual appearance of a website.
    

### What is Drupal's architecture?

Drupal follows a modular architecture based on Symfony components and its own APIs.

Important parts:

- Modules.
    
- Themes.
    
- Services and dependency injection.
    
- Plugins.
    
- Entities.
    
- Hooks and events.
    
- Configuration.
    
- Routing and controllers.
    

### What is the difference between Drupal and a traditional PHP application?

Drupal provides a ready-made framework with content management, user permissions, configuration management, caching, and extensibility.

In a traditional PHP application, developers usually need to implement these features themselves or integrate external libraries.

### What is Drush?

Drush is a command-line tool for managing Drupal applications.

Common commands:

```
drush cr                 # Clear caches
drush cim -y             # Import configuration
drush cex -y             # Export configuration
drush updb -y            # Run database updates
drush status             # Show Drupal status
drush uli                # Generate login link
drush sql:dump           # Export database
drush watchdog:show      # Show recent log messages
```

### What is Drupal's project structure?

Important directories:

```
web/
  core/
  modules/
    contrib/
    custom/
  themes/
    contrib/
    custom/
  profiles/
  sites/
    default/
      settings.php
      services.yml
```

- `core/` — Drupal Core.
    
- `modules/custom/` — custom modules.
    
- `modules/contrib/` — contributed modules.
    
- `themes/custom/` — custom themes.
    
- `sites/default/settings.php` — site-specific settings.
    

### What is the difference between Drupal 10 and Drupal 11?

Drupal 11 continues the modernization of Drupal's architecture and dependencies.

Important points:

- Drupal 11 requires PHP 8.3 or newer.
    
- It uses modern Symfony components.
    
- It continues to improve APIs and developer experience.
    
- Deprecated APIs from earlier versions may be removed.
    
- Drupal 11 encourages modern PHP practices and up-to-date dependencies.
    

---

## 2. Content Architecture and Entities

### What is an Entity?

An Entity is a structured data object managed by Drupal's Entity API.

Examples:

- Node.
    
- User.
    
- Taxonomy Term.
    
- Media.
    
- Custom entities.
    

Entities provide a consistent way to store, load, update, delete, and access data.

### What is a Node?

A Node is a content entity used to store website content.

Examples:

- Article.
    
- Basic Page.
    
- Blog Post.
    
- Product.
    

### What is the difference between Node and Entity?

An Entity is a general concept for structured data in Drupal.

A Node is a specific content entity type.

All Nodes are entities, but not all entities are Nodes.

### What is an Entity Type?

An Entity Type defines a type of structured data and its behavior.

Examples:

- `node`
    
- `user`
    
- `taxonomy_term`
    
- `media`
    

### What is the difference between Content Entity and Configuration Entity?

**Content Entity:**

- Stores user-generated or business content.
    
- Usually stored in database tables.
    
- Can have fields and revisions.
    
- Examples: Node, User, Media.
    

**Configuration Entity:**

- Stores configuration that defines how the application works.
    
- Usually exportable through configuration management.
    
- Examples: Content Type, View, Role, Image Style.
    

### What is a Bundle?

A Bundle is a subtype of an entity type.

For example:

- Entity type: `node`.
    
- Bundles: `article`, `page`.
    

Different bundles can have different fields and configurations.

### What is a Field?

A Field stores a specific piece of data attached to an entity.

Examples:

- Text.
    
- Image.
    
- Entity Reference.
    
- Date.
    
- Number.
    
- Boolean.
    

### What is Field Storage?

Field Storage defines the underlying field definition and storage configuration, including its data type and cardinality.

### What is Field Instance?

Historically, Drupal 7 used the concept of field instances. In modern Drupal, fields are attached to bundles through field configuration entities, mainly `FieldConfig`.

### What is an Entity Reference?

An Entity Reference field creates a relationship between entities.

Example: a Task references a Project.

### What is a Revision?

A Revision represents a previous version of an entity.

Revisions allow Drupal to maintain content history and support moderation workflows.

### What is Entity Translation?

Entity Translation allows one entity to have translated field values in different languages.

### How do you create a Node programmatically?

```
use Drupal\node\Entity\Node;

$node = Node::create([
    'type' => 'article',
    'title' => 'My Article',
    'status' => 1,
]);

$node->save();
```

### How do you load an Entity?

```
$node = Node::load(10);
```

For multiple entities:

```
$nodes = Node::loadMultiple([10, 11, 12]);
```

### How do you update an Entity?

```
$node = Node::load(10);

if ($node) {
    $node->setTitle('Updated title');
    $node->save();
}
```

### How do you delete an Entity?

```
$node = Node::load(10);

if ($node) {
    $node->delete();
}
```

### What is Entity API?

Entity API provides a unified way to work with structured Drupal data.

It supports:

- Loading and saving entities.
    
- Entity queries.
    
- Fields.
    
- Access control.
    
- Revisions.
    
- Translation.
    
- Entity storage handlers.
    

---

## 3. Hooks and Events API

### What is a Hook?

A Hook is a mechanism that allows modules to interact with or alter Drupal functionality at specific points.

Hooks are commonly used to extend existing behavior without modifying Drupal Core.

### Where are hooks implemented?

Usually in a module's `.module` file.

Example:

```
function mymodule_preprocess_node(array &$variables): void {
    $variables['custom_message'] = 'Hello Drupal';
}
```

### What is hook_form_alter()?

It allows developers to modify an existing Drupal form.

```
function mymodule_form_alter(
    array &$form,
    \Drupal\Core\Form\FormStateInterface $form_state,
    string $form_id
): void {
    if ($form_id === 'user_login_form') {
        $form['name']['#description'] = 'Enter your username';
    }
}
```

### What is hook_entity_insert()?

It is invoked when a new entity is inserted into storage.

Similar hooks:

- `hook_entity_presave()`
    
- `hook_entity_insert()`
    
- `hook_entity_update()`
    
- `hook_entity_delete()`
    

### What is hook_theme()?

It registers custom theme hooks that allow modules to define their own rendering templates.

### What is the difference between Hooks and Events?

**Hooks:**

- Drupal-specific extension mechanism.
    
- Usually procedural functions.
    
- Implemented by modules.
    
- Used to alter or extend specific Drupal operations.
    

**Events:**

- Based on Symfony's Event Dispatcher.
    
- Use event objects and event subscribers.
    
- Allow multiple subscribers to react to the same event.
    
- Useful for decoupled event-driven logic.
    

### What is an Event Subscriber?

An Event Subscriber is a class that listens to specific events and executes logic when they occur.

Example:

```
namespace Drupal\mymodule\EventSubscriber;

use Symfony\Component\EventDispatcher\EventSubscriberInterface;
use Symfony\Component\HttpKernel\KernelEvents;

class RequestSubscriber implements EventSubscriberInterface {

    public static function getSubscribedEvents(): array {
        return [
            KernelEvents::REQUEST => ['onRequest', 100],
        ];
    }

    public function onRequest(): void {
        // Custom logic.
    }

}
```

It must also be registered as a service.

### Can multiple subscribers listen to the same event?

Yes. Multiple subscribers can listen to the same event.

They can have different priorities, which determine their execution order.

### What is Event Propagation?

Event propagation determines whether an event continues to other listeners.

A listener can stop propagation when necessary.

### When should you use Hooks and when Events?

I use hooks when Drupal provides a specific extension point for modifying existing behavior.

I use events when I need decoupled components to react to an event, especially when multiple subscribers are involved.

---

## 4. Services and Dependency Injection

### What is a Service?

A Service is an object that provides reusable functionality.

Examples:

- Database operations.
    
- Email sending.
    
- Business logic.
    
- External API communication.
    
- Custom calculations.
    

### What is a Service Container?

The Service Container manages service definitions and object dependencies.

It creates services and injects their dependencies when needed.

### What is Dependency Injection?

Dependency Injection means providing an object with its dependencies from outside instead of creating them internally.

### How do you define a custom service?

Create a service definition in `mymodule.services.yml`.

```
services:
  mymodule.user_manager:
    class: Drupal\mymodule\Service\UserManager
    arguments: ['@current_user']
```

Service class:

```
namespace Drupal\mymodule\Service;

use Drupal\Core\Session\AccountProxyInterface;

class UserManager {

    public function __construct(
        private AccountProxyInterface $currentUser,
    ) {}

    public function getCurrentUserId(): int {
        return (int) $this->currentUser->id();
    }

}
```

### How do you use a service in a Controller?

Use constructor injection.

```
class UserController {

    public function __construct(
        private UserManager $userManager,
    ) {}

    public function content(): array {
        $uid = $this->userManager->getCurrentUserId();

        return [
            '#markup' => (string) $uid,
        ];
    }

}
```

For Drupal controllers, use `create()` to retrieve dependencies from the container when the class is instantiated by the controller resolver.

### What is the difference between Dependency Injection and Drupal::service()?

**Dependency Injection:**

- Dependencies are explicitly declared.
    
- Easier to test.
    
- Reduces coupling.
    
- Recommended for services, controllers, forms, and plugins where supported.
    

**Drupal::service():**

- Provides direct access to the global service container.
    
- Creates hidden dependencies.
    
- Makes unit testing harder.
    
- Useful in some procedural contexts where injection is unavailable.
    

### What is Constructor Injection?

Constructor Injection passes dependencies through the class constructor.

It is generally the preferred dependency injection approach in Drupal.

### What is Setter Injection?

Setter Injection provides dependencies through setter methods.

It is useful when dependencies are optional or need to be changed after construction.

### What is a Service Decoration?

Service Decoration allows you to extend or modify an existing service without replacing its original implementation completely.

It is useful when changing behavior provided by Core or a contributed module.

### What is a Compiler Pass?

A Compiler Pass allows developers to modify service definitions during container compilation.

It is an advanced Symfony service container feature.

---

## 5. Plugin API

### What is a Plugin?

A Plugin is a reusable component that provides a specific functionality within a defined plugin type.

Drupal uses plugins to make systems extensible.

### What is a Plugin Type?

A Plugin Type defines a contract and discovery mechanism for a group of plugins.

Examples:

- Blocks.
    
- Field Types.
    
- Field Widgets.
    
- Field Formatters.
    
- Views plugins.
    
- Queue Workers.
    
- Migrate plugins.
    

### What is Plugin Discovery?

Plugin Discovery is the process of finding available plugin implementations.

Depending on the plugin type and API, Drupal can use PHP attributes or annotations and other discovery mechanisms.

### What is a Block Plugin?

A Block Plugin defines a reusable block that can be placed in a region of a theme.

```
namespace Drupal\mymodule\Plugin\Block;

use Drupal\Core\Block\BlockBase;

/**
 * Provides a custom block.
 *
 * @Block(
 *   id = "custom_block",
 *   admin_label = @Translation("Custom Block")
 * )
 */
class CustomBlock extends BlockBase {

    public function build(): array {
        return [
            '#markup' => $this->t('Hello Drupal'),
        ];
    }

}
```

### What is the difference between a Block and a Block Plugin?

A Block Plugin defines the functionality.

A Block Configuration Entity represents a placed and configured instance of a block.

### What is a Field Widget?

A Field Widget controls how users enter field values in entity forms.

Example: a text field widget.

### What is a Field Formatter?

A Field Formatter controls how field values are displayed.

Example: displaying an image as a thumbnail.

### What is a Queue Worker Plugin?

A Queue Worker Plugin defines how queued items are processed.

It is useful for background processing.

### What is a Plugin Manager?

A Plugin Manager discovers, instantiates, and manages plugins of a specific type.

### What is the difference between a Service and a Plugin?

- A Service provides reusable application functionality and is managed by the service container.
    
- A Plugin is a specific implementation of a plugin type and is managed by a plugin manager.
    

A plugin can also receive services through dependency injection.

---

## 6. Routing, Controllers and Access

### What is Routing in Drupal?

Routing defines URLs and connects them to controllers, forms, or other handlers.

Routes are usually declared in a module's `.routing.yml` file.

### How do you define a custom route?

```
mymodule.user:
  path: '/custom-user'
  defaults:
    _controller: '\Drupal\mymodule\Controller\UserController::content'
    _title: 'Custom User'
  requirements:
    _permission: 'access content'
```

### What is a Controller?

A Controller handles an HTTP request and returns a response or render array.

### What is a Render Array?

A Render Array is a structured PHP array that describes how Drupal should render content.

```
return [
    '#theme' => 'item_list',
    '#items' => [
        'First item',
        'Second item',
    ],
];
```

### What is the difference between a Render Array and HTML?

A Render Array describes the output using Drupal's rendering system.

Drupal can process cache metadata, attachments, themes, and access information before generating HTML.

### What is Access Control?

Access Control determines whether a user can access a route, entity, or operation.

### What is the difference between permissions and roles?

- A permission defines a specific action a user can perform.
    
- A role is a collection of permissions.
    
- A user can have multiple roles.
    

### How do you define a custom permission?

Create `mymodule.permissions.yml`.

```
access custom data:
  title: 'Access custom data'
  description: 'Allows access to custom data.'
```

### How do you check permissions in code?

```
$access = $account->hasPermission('access content');
```

### What is Entity Access?

Entity Access determines whether a user can perform operations such as view, update, or delete on a particular entity.

### What is AccessResult?

AccessResult represents an access decision and can carry cacheability metadata.

```
use Drupal\Core\Access\AccessResult;

return AccessResult::allowedIf(
    $account->hasPermission('administer nodes')
);
```

### What is the difference between 401 and 403?

- 401 means authentication is required or has failed.
    
- 403 means access is forbidden.
    

### What is CSRF protection in Drupal?

Drupal uses CSRF tokens to protect sensitive operations, especially forms and routes that modify data.

For custom routes, appropriate route requirements and access checks must be configured.

---

## 7. Form API

### What is Form API?

Form API is Drupal's system for creating, processing, validating, and submitting forms.

### What are the main Form API methods?

- `getFormId()` — returns a unique form ID.
    
- `buildForm()` — builds the form.
    
- `validateForm()` — validates submitted values.
    
- `submitForm()` — processes valid submissions.
    

### How do you create a custom form?

```
namespace Drupal\mymodule\Form;

use Drupal\Core\Form\FormBase;
use Drupal\Core\Form\FormStateInterface;

class CustomForm extends FormBase {

    public function getFormId(): string {
        return 'custom_form';
    }

    public function buildForm(
        array $form,
        FormStateInterface $form_state,
    ): array {
        $form['name'] = [
            '#type' => 'textfield',
            '#title' => $this->t('Name'),
            '#required' => TRUE,
        ];

        $form['submit'] = [
            '#type' => 'submit',
            '#value' => $this->t('Submit'),
        ];

        return $form;
    }

    public function submitForm(
        array &$form,
        FormStateInterface $form_state,
    ): void {
        $name = $form_state->getValue('name');

        $this->messenger()->addStatus(
            $this->t('Hello @name', ['@name' => $name])
        );
    }

}
```

### What is Form Validation?

Validation checks submitted data before the form is processed.

It can be implemented in `validateForm()`.

### What is Form State?

FormStateInterface stores submitted values, validation errors, and other information during form processing.

### What is Form Alter?

Form Alter allows developers to modify an existing form without rewriting it.

### What is the difference between FormBase and ConfigFormBase?

- `FormBase` is used for general custom forms.
    
- `ConfigFormBase` is designed for forms that manage Drupal configuration.
    

### What is a Multistep Form?

A Multistep Form divides a large form into multiple steps.

Form state can store temporary data between steps.

---

## 8. Configuration Management

### What is Configuration Management?

Configuration Management allows developers to export, import, and synchronize Drupal configuration between environments.

Examples:

- Content Types.
    
- Views.
    
- Roles.
    
- Image Styles.
    
- Field Definitions.
    
- Site Settings.
    

### What is the difference between Configuration and Content?

**Configuration:**  
Defines how the application works.

Examples: Views, content types, roles.

**Content:**  
Represents actual website data.

Examples: articles, users, products.

### What is Configuration Synchronization?

It is the process of transferring configuration between environments using exported YAML files.

### What is `config/sync`?

It is the directory where Drupal stores exported configuration files.

### What is `drush cex`?

It exports active configuration to YAML files.

### What is `drush cim`?

It imports configuration from YAML files into the active configuration storage.

### What is Active Configuration?

Active Configuration is the configuration currently used by the running Drupal application.

### What is the difference between Default and Active Configuration?

- Default configuration is provided by modules or installation profiles.
    
- Active configuration is stored in the site's configuration storage and represents the currently applied configuration.
    

### What is Configuration Schema?

Configuration Schema describes the structure and data types of configuration values.

It helps Drupal validate and understand configuration.

### How do you create default configuration?

Create a YAML file inside the module's `config/install` directory.

Example: `mymodule.settings.yml`.

```
api_endpoint: 'https://example.com'
enabled: true
```

### How do you read configuration?

```
$config = $this->configFactory
    ->get('mymodule.settings');

$endpoint = $config->get('api_endpoint');
```

### How do you update configuration?

```
$this->configFactory
    ->getEditable('mymodule.settings')
    ->set('enabled', TRUE)
    ->save();
```

### What is the difference between `get()` and `getEditable()`?

- `get()` returns immutable configuration.
    
- `getEditable()` returns editable configuration that can be changed and saved.
    

### What is the difference between Configuration API and State API?

- Configuration API stores deployable settings.
    
- State API stores environment-specific or runtime information that should not normally be exported.
    

Example:

- API endpoint setting — Configuration API.
    
- Last cron execution timestamp — State API.
    

### What is a Configuration Entity?

A Configuration Entity is a configuration object managed through Drupal's Entity API.

Examples: Views, roles, content types.

### What is a Config Split?

Config Split is a contributed module that allows configuration to be separated by environment.

For example, development configuration can differ from production configuration.

---

## 9. Database API and Entity Query

### What is Drupal Database API?

Drupal Database API provides an abstraction layer for database operations.

It allows developers to write database queries without depending directly on a specific database engine.

### How do you execute a database query?

```
$query = $this->database->select('users_field_data', 'u');

$query->fields('u', ['uid', 'name']);

$results = $query->execute()->fetchAll();
```

### What is Entity Query?

Entity Query is an API for querying Drupal entities.

```
$ids = $this->entityTypeManager
    ->getStorage('node')
    ->getQuery()
    ->accessCheck(TRUE)
    ->condition('type', 'article')
    ->condition('status', 1)
    ->execute();
```

### What is the difference between Entity Query and Database API?

- Entity Query works with entities and their fields.
    
- Database API works directly with database tables and records.
    

I prefer Entity Query when working with Drupal entities because it respects the Entity API abstraction.

### What is `entityTypeManager()`?

EntityTypeManager is a service responsible for managing entity types and their storage handlers, definitions, and access to entity storage.

### How do you load entities using EntityTypeManager?

```
$storage = $this->entityTypeManager
    ->getStorage('node');

$node = $storage->load(10);
```

### What is an Entity Storage Handler?

It manages persistence operations for an entity type.

Examples:

- Load.
    
- Save.
    
- Delete.
    
- Query.
    

### What is the difference between Entity Storage and Entity Repository?

- Entity Storage handles persistence and loading.
    
- Entity Repository provides higher-level entity retrieval functionality, including canonical and revision-related lookup methods.
    

### What is a Database Transaction?

A transaction groups database operations so they can be committed together or rolled back.

```
$transaction = $connection->startTransaction();

try {
    // Database operations.
} catch (\Throwable $exception) {
    $transaction->rollBack();
    throw $exception;
}
```

---

## 10. Cache API and Performance

### What is Drupal Cache API?

Cache API stores computed data to avoid repeating expensive operations.

It improves performance by reducing database queries and processing time.

### What are the main cache concepts?

- Cache Tags.
    
- Cache Contexts.
    
- Cache Max-Age.
    

### What are Cache Tags?

Cache Tags identify the data that a cached item depends on.

They allow Drupal to invalidate cached data when related content changes.

Example:

```
'#cache' => [
    'tags' => ['node:10'],
],
```

When Node 10 changes, cache entries with this tag can be invalidated.

### What are Cache Contexts?

Cache Contexts define variations of cached output.

Examples:

- `user`
    
- `user.roles`
    
- `languages:language_interface`
    
- `url.path`
    
- `url.query_args`
    

```
'#cache' => [
    'contexts' => ['user.roles'],
],
```

### What is Cache Max-Age?

Cache Max-Age defines how long cached data can be considered valid.

- `0` — not cacheable.
    
- Positive value — cache lifetime in seconds.
    
- `Cache::PERMANENT` — no time-based expiration.
    

### What is Cache Metadata?

Cache Metadata combines cache tags, contexts, and max-age.

Drupal uses it to determine whether cached output can be reused and when it must be invalidated.

### What is Cache Invalidation?

Cache Invalidation marks cached data as outdated so Drupal knows it should be rebuilt.

### What is Render Cache?

Render Cache stores rendered output of render arrays.

### What is Dynamic Page Cache?

Dynamic Page Cache caches pages even when some parts depend on dynamic contexts.

### What is Internal Page Cache?

Internal Page Cache caches complete pages for anonymous users.

### What is the difference between Render Cache and Dynamic Page Cache?

- Render Cache stores individual rendered components.
    
- Dynamic Page Cache stores rendered page responses and supports varying by cache contexts.
    

### What is Cacheability Metadata?

Cacheability Metadata describes how a render element or response can be cached.

It includes:

- Cache tags.
    
- Cache contexts.
    
- Cache max-age.
    

### How do you improve Drupal performance?

- Enable OPcache.
    
- Use Drupal caching correctly.
    
- Optimize database queries.
    
- Use appropriate indexes.
    
- Avoid unnecessary entity loading.
    
- Use Views efficiently.
    
- Use BigPipe when appropriate.
    
- Process expensive tasks asynchronously.
    
- Profile performance before optimizing.
    

### What is BigPipe?

BigPipe is a rendering technique that sends the initial page response quickly and replaces placeholders with dynamic content as it becomes available.

It improves perceived page performance.

### What is Cache Context Optimization?

It means using the narrowest correct cache contexts to avoid unnecessary cache variations.

For example, use `user.roles` instead of `user` when output depends only on roles.

---

## 11. Theming and Twig

### What is a Drupal Theme?

A theme controls the presentation layer of a Drupal website.

It defines templates, stylesheets, JavaScript, and visual components.

### What is Twig?

Twig is a template engine used by Drupal for rendering HTML.

It separates presentation from PHP business logic.

### What is a Template Suggestion?

Template Suggestions allow Drupal to choose different Twig templates depending on the context.

Examples:

- `node.html.twig`
    
- `node--article.html.twig`
    
- `node--article--full.html.twig`
    

### What is hook_theme()?

It registers custom theme hooks and defines the variables available to Twig templates.

### What is hook_preprocess_HOOK()?

It allows developers to prepare variables before a template is rendered.

Example:

```
function mytheme_preprocess_node(
    array &$variables
): void {
    $variables['custom_value'] = 'Hello';
}
```

### What is a Render Array in theming?

A Render Array is a structured description of output that Drupal processes before rendering it through Twig.

### What is a Drupal Library?

A Library defines CSS and JavaScript assets.

It is declared in a `.libraries.yml` file.

```
global:
  css:
    theme:
      css/style.css: {}
  js:
    js/script.js: {}
  dependencies:
    - core/drupal
    - core/once
```

### How do you attach a library?

```
return [
    '#theme' => 'item_list',
    '#items' => ['One', 'Two'],
    '#attached' => [
        'library' => [
            'mytheme/global',
        ],
    ],
];
```

### What is the difference between a Theme and a Module?

A module provides functionality, while a theme controls presentation.

### What is Twig Autoescaping?

Twig automatically escapes output to help prevent XSS attacks.

Developers should avoid disabling escaping for untrusted data.

---

## 12. REST, JSON and Web Services

### What is REST?

REST is an architectural style for building APIs around resources and standard HTTP methods.

### What is RESTful Web Services in Drupal?

Drupal provides REST functionality through Core modules.

It allows developers to expose resources through HTTP endpoints.

### What is JSON?

JSON is a standardized API specification for exposing and manipulating resources using JSON.

Drupal provides a JSON implementation in Core.

### What is the difference between REST and JSON?

- REST is an architectural style.
    
- JSON is a specific API specification built around resource-oriented HTTP communication.
    

Drupal's REST module provides configurable REST resources and formats.

JSON provides a standardized approach to working with entity resources.

### How do you enable REST in Drupal?

Enable the REST module and configure resource settings, supported formats, authentication providers, and HTTP methods.

REST configuration is usually stored in YAML configuration entities.

### What is Serialization?

Serialization converts PHP objects or data structures into a format such as JSON or XML.

### What is the Serializer component?

It converts data between PHP representations and external formats using normalizers and encoders.

### What is Authentication in Drupal APIs?

Drupal supports multiple authentication approaches depending on the API and configuration.

Examples:

- Cookie-based authentication.
    
- HTTP Basic Authentication.
    
- OAuth 2.0 through contributed modules.
    
- Other configured authentication providers.
    

### What is CORS?

Cross-Origin Resource Sharing controls whether browsers allow requests from different origins.

### What is an Entity Normalizer?

An Entity Normalizer converts Drupal entities into a serialized representation, such as JSON or another supported format.

---

## 13. Security

### How does Drupal handle security?

Drupal provides security mechanisms such as:

- User roles and permissions.
    
- Entity access control.
    
- Route access requirements.
    
- CSRF protection.
    
- Input validation.
    
- Output escaping.
    
- Secure password handling.
    
- Database abstraction.
    

### What is XSS?

Cross-Site Scripting is an attack where malicious scripts are injected into a webpage.

Prevention:

- Twig autoescaping.
    
- Proper output escaping.
    
- Input validation.
    
- Avoid rendering untrusted HTML.
    

### What is SQL Injection?

SQL Injection occurs when malicious SQL is inserted into database queries.

Prevention:

- Use Database API.
    
- Use parameterized queries.
    
- Never concatenate untrusted input into SQL.
    

### What is CSRF?

Cross-Site Request Forgery tricks an authenticated user into performing an unwanted action.

Drupal uses CSRF tokens and route-level protections to prevent these attacks.

### What is Access Control?

Access Control determines whether a user can perform an operation.

It can be implemented through:

- Permissions.
    
- Route requirements.
    
- Entity access handlers.
    
- Custom access checkers.
    

### What is the difference between `#access` and route access?

- `#access` controls whether a render element is visible.
    
- Route access controls whether the user can access a URL.
    

Hiding a button does not replace backend authorization.

### What is Input Validation?

Input Validation checks whether submitted data meets expected rules before processing it.

### What is Output Escaping?

Output Escaping converts potentially dangerous characters into safe representations depending on the output context.

### What is `Xss::filter()`?

It is a Drupal API method that filters HTML to remove or neutralize potentially dangerous markup.

It should not be treated as a replacement for proper context-aware output escaping.

### What is Trusted HTML?

Trusted HTML is content explicitly marked as safe for HTML rendering.

Developers must only mark content as trusted when its safety has been established.

---

## 14. Drupal Migration API

### What is Migration API?

Migration API is used to transfer data from external systems or older Drupal versions into Drupal.

Examples:

- Drupal 7 to Drupal 11.
    
- CSV to Drupal.
    
- Legacy databases to Drupal.
    
- External CMS to Drupal.
    

### What are the main migration components?

- Source Plugin.
    
- Process Plugins.
    
- Destination Plugin.
    
- Migration Definition.
    
- Migration ID Map.
    

### What is a Source Plugin?

A Source Plugin reads data from the original source.

Examples:

- CSV.
    
- SQL database.
    
- Drupal entities.
    
- External systems.
    

### What is a Process Plugin?

A Process Plugin transforms source data into the destination structure.

Examples:

- `get`
    
- `default_value`
    
- `migration_lookup`
    
- `concat`
    
- `skip_on_empty`
    

### What is a Destination Plugin?

A Destination Plugin defines where migrated data is stored.

Examples:

- Nodes.
    
- Users.
    
- Taxonomy Terms.
    
- Custom entities.
    

### What is Migration ID Map?

ID Map stores the relationship between source IDs and destination IDs.

It allows Drupal to track migrated records and update them later.

### What is a Migration Definition?

A Migration Definition is a YAML configuration describing how data should be transferred.

Example:

```
id: custom_users
label: 'Import users'

source:
  plugin: csv
  path: modules/custom/mymodule/data/users.csv
  header_row_count: 1
  ids:
    - id

process:
  name: username
  mail: email

destination:
  plugin: 'entity:user'
```

### What is Migration Lookup?

Migration Lookup finds the destination entity ID associated with a source ID from another migration.

It is useful for migrating entity references.

### What is the difference between Migration API and Update Hooks?

- Migration API transfers data between systems or data structures.
    
- Update Hooks apply database or configuration updates when deploying new module versions.
    

### What are Migrate Plus and Migrate Tools?

They are contributed modules that extend migration functionality.

Migrate Plus provides additional migration capabilities, while Migrate Tools provides useful Drush commands for managing migrations.

### How do you execute a migration?

```
drush migrate:status
drush migrate:import custom_users
drush migrate:rollback custom_users
```

---

## 15. Cron, Queue API and Batch API

### What is Cron?

Cron is Drupal's mechanism for executing scheduled tasks.

Examples:

- Cleaning expired data.
    
- Sending scheduled notifications.
    
- Updating external information.
    

### How do you implement hook_cron()?

```
function mymodule_cron(): void {
    // Scheduled operations.
}
```

### What is Queue API?

Queue API allows developers to store tasks for later processing.

It is useful for background operations.

### What is a Queue Worker?

A Queue Worker processes items stored in a queue.

It is usually implemented as a plugin.

### What is the difference between Cron and Queue API?

- Cron triggers scheduled operations.
    
- Queue API stores and processes individual tasks.
    

Cron can be used to trigger queue processing.

### What is Batch API?

Batch API allows long-running operations to be split into smaller steps.

It prevents timeouts and provides progress information.

### When would you use Queue API instead of Batch API?

I use Queue API for asynchronous background processing, especially when tasks can be executed independently.

I use Batch API when a user starts a long-running operation and needs to see its progress.

---

## 16. Update Hooks and Installation Hooks

### What is an Update Hook?

An Update Hook is used to perform database or configuration updates when a module is updated.

It is implemented in the `.install` file.

```
function mymodule_update_10001(): void {
    // Update logic.
}
```

### What is hook_install()?

It runs when a module is installed.

It is commonly used for initial setup.

### What is hook_uninstall()?

It runs when a module is uninstalled.

It can remove module-specific configuration or other resources when appropriate.

### What is the difference between hook_install() and Update Hooks?

- `hook_install()` runs during initial module installation.
    
- Update Hooks run when an existing module is updated to a newer version.
    

### What is hook_schema()?

It defines custom database tables owned by a module.

### What is the purpose of Update Hooks?

They allow developers to safely apply changes to existing installations without reinstalling the module.

### How do you execute Update Hooks?

```
drush updb
```

### Should you modify an existing Update Hook?

No. Once an update hook has been deployed and executed, it should not be modified.

Create a new update hook instead.

---

## 17. Multilingual Drupal

### What is multilingual support in Drupal?

Drupal provides multilingual functionality through Core language modules.

It supports translated content, configuration, interface strings, and language negotiation.

### What are the main multilingual modules?

- Language.
    
- Interface Translation.
    
- Content Translation.
    
- Configuration Translation.
    

### What is the difference between Content Translation and Configuration Translation?

- Content Translation translates content entities and their fields.
    
- Configuration Translation translates configuration values, such as labels and descriptions.
    

### What is Interface Translation?

It translates interface strings used by Drupal and modules.

### What is Language Negotiation?

Language Negotiation determines which language should be used for the current request.

Possible methods:

- URL prefix.
    
- Domain.
    
- Browser language.
    
- Session.
    
- User preferences.
    

### What is Translatable Field?

A field whose values can differ between language versions of an entity.

### What is a Translation Entity?

Drupal generally stores translations as language-specific values of the same content entity rather than creating completely separate entities.

---

## 18. Testing in Drupal

### What testing types does Drupal support?

Drupal supports:

- Unit Tests.
    
- Kernel Tests.
    
- Functional Tests.
    
- Functional JavaScript Tests.
    

### What is a Unit Test?

A Unit Test checks a small piece of isolated functionality without bootstrapping the full Drupal application.

### What is a Kernel Test?

A Kernel Test bootstraps a minimal Drupal environment.

It can test services, entities, database operations, and other integration points.

### What is a Functional Test?

A Functional Test checks application behavior through HTTP requests using a simulated browser.

### What is a Functional JavaScript Test?

It tests browser behavior that requires JavaScript execution.

### What is PHPUnit?

PHPUnit is the main testing framework used by Drupal.

### What is the difference between Unit and Kernel Tests?

- Unit Tests are isolated and usually faster.
    
- Kernel Tests initialize Drupal's kernel and test integration between components.
    

### How do you mock services?

You can use PHPUnit mocks to replace dependencies with controlled test doubles.

### What is BrowserTestBase?

BrowserTestBase is a Drupal testing base class for functional browser tests.

### What is KernelTestBase?

KernelTestBase is used for tests that need Drupal's kernel and selected services but not a full browser.

### What is Test Coverage?

Test Coverage measures which parts of the code are executed by tests.

High coverage does not automatically guarantee good test quality.

### What tools do you use for code quality?

- PHPUnit — automated tests.
    
- PHPStan — static analysis.
    
- PHP_CodeSniffer — coding standards.
    
- PHP CS Fixer — code formatting.
    
- GrumPHP — automated quality checks in Git workflows.
    

---

## 19. Drupal Architecture — Senior Level

### What is a Drupal Service Layer?

A Service Layer contains reusable application logic and coordinates different components.

It helps keep controllers, forms, and plugins thin.

### What is Separation of Concerns?

Separation of Concerns means dividing application responsibilities into separate components.

For example:

- Controller — handles HTTP requests.
    
- Service — contains business logic.
    
- Repository or Storage — handles data access.
    
- Twig — handles presentation.
    

### What is the difference between a Controller and a Service?

- A Controller handles the request and prepares the response.
    
- A Service implements reusable business or application logic.
    

### What is the difference between a Service and a Plugin?

A Service is managed by the service container.

A Plugin is managed by a plugin manager and follows a specific plugin contract.

### What is the difference between an Entity and a Configuration Object?

An Entity is a structured data object managed by the Entity API.

A Configuration Object stores application settings through the Configuration API.

### What is a Custom Entity?

A Custom Entity is an entity type created for project-specific data that does not fit existing entity types.

It can be a Content Entity or Configuration Entity.

### When would you create a Custom Entity?

When I need a reusable structured data model with its own lifecycle, storage, access rules, and potentially revisions or fields.

### What is a Repository Pattern?

Repository Pattern separates data access from business logic.

In Drupal, Entity Storage already provides a persistence abstraction, so a custom repository should be introduced when it adds meaningful domain-specific query or data-access behavior.

### What is a DTO?

A Data Transfer Object is an object used to transfer structured data between layers without exposing internal entities or implementation details.

### What is a Value Object?

A Value Object represents a concept defined by its values rather than identity.

Examples:

- Money.
    
- Email Address.
    
- Date Range.
    

It is usually immutable.

### What is Domain-Driven Design?

DDD is an approach to software design based on business domains and their concepts.

Important concepts:

- Entities.
    
- Value Objects.
    
- Aggregates.
    
- Repositories.
    
- Domain Services.
    
- Bounded Contexts.
    

### What is Event-Driven Architecture?

Event-Driven Architecture allows components to react to events without requiring direct dependencies between them.

Example: when a task is completed, different subscribers can send notifications, update statistics, and write logs.

### What is CQRS?

CQRS separates operations that modify data (commands) from operations that read data (queries).

It can help when read and write requirements are significantly different.

### What is a Bounded Context?

A Bounded Context is a clearly defined area of a business domain with its own model and terminology.

### What is Technical Debt?

Technical Debt is the future cost of choosing a quick or less maintainable implementation.

### How do you approach an unfamiliar Drupal codebase?

1. Understand the project structure.
    
2. Review enabled modules and dependencies.
    
3. Read custom module service and routing definitions.
    
4. Identify entities and bundles.
    
5. Understand the request lifecycle.
    
6. Trace hooks, events, and plugins.
    
7. Review configuration.
    
8. Run existing tests.
    

### How do you debug a Drupal issue?

1. Reproduce the problem.
    
2. Check Drupal logs.
    
3. Inspect the relevant service or controller.
    
4. Review cache metadata.
    
5. Check permissions and access results.
    
6. Inspect database queries if necessary.
    
7. Add tests for the issue.
    
8. Verify the fix.
    

### How do you review Drupal code?

I check:

- Drupal coding standards.
    
- Dependency injection.
    
- API usage.
    
- Access control.
    
- Cache metadata.
    
- Database query performance.
    
- Configuration management.
    
- Test coverage.
    
- Maintainability.
    

---

## 20. Drupal Settings and Deployment

### What is settings.php?

`settings.php` contains site-specific configuration, such as database connection details, trusted host settings, and environment-specific overrides.

### What is services.yml?

It allows developers to override or configure services and parameters in the service container.

### What is settings.local.php?

It is commonly used for local development settings, such as debugging, local database configuration, and development-specific behavior.

### What is a Drupal Environment?

An environment is a separate deployment instance.

Common environments:

- Local.
    
- Development.
    
- Staging.
    
- Production.
    

### What is Configuration Management in deployment?

Configuration Management ensures that configuration changes are version-controlled and consistently applied across environments.

### What is a typical Drupal deployment workflow?

1. Update code through Git.
    
2. Install Composer dependencies.
    
3. Run database updates.
    
4. Import configuration.
    
5. Rebuild caches.
    
6. Verify the application.
    

Example:

```
composer install --no-dev
drush updb -y
drush cim -y
drush cr
```

The exact order can depend on the deployment architecture and whether configuration changes depend on database updates.

### What is a Drupal Multisite?

Drupal Multisite allows multiple websites to share one Drupal codebase while maintaining separate site-specific configurations and databases.

### What are the advantages of Multisite?

- Shared codebase.
    
- Centralized updates.
    
- Reduced code duplication.
    

### What are the disadvantages of Multisite?

- Shared deployment risks.
    
- Different sites may need different module versions.
    
- More complicated maintenance and configuration management.
    

### What is Composer in Drupal?

Composer manages Drupal Core, contributed modules, themes, and third-party PHP dependencies.

### What is `composer.lock`?

It stores exact dependency versions to ensure consistent installations across environments.

---

## 21. Important Drupal APIs and Services

|API / Service|Purpose|
|---|---|
|Entity API|Manage structured data|
|Plugin API|Create extensible components|
|Form API|Build and process forms|
|Routing API|Define URLs and controllers|
|Render API|Build structured output|
|Cache API|Cache data and rendered output|
|Configuration API|Manage deployable settings|
|State API|Store runtime state|
|Database API|Execute database queries|
|Queue API|Process background tasks|
|Batch API|Handle long-running operations|
|Migration API|Import and transform data|
|Translation API|Support multilingual content|
|Access API|Control permissions and access|
|Logger API|Write application logs|
|Messenger API|Display user messages|
|EntityTypeManager|Access entity definitions and storage|
|EntityTypeManager::getStorage()|Retrieve entity storage|
|ConfigFactory|Read and update configuration|
|CurrentUser|Access current user information|
|Database|Access database connections|
|CacheBackendInterface|Work with cache bins|
|DateFormatter|Format dates|
|Renderer|Render render arrays|
|RequestStack|Access the current HTTP request|
|PathMatcher|Match the current route or path|
|ModuleHandler|Interact with enabled modules and hooks|
|PluginManager|Discover and manage plugins|
|EventDispatcher|Dispatch events|

---

## 22. Common Drupal Interview Questions

### What is the difference between Hooks and Events?

Hooks are Drupal-specific extension points, while Events use Symfony's Event Dispatcher and event subscribers.

### What is the difference between a Service and a Plugin?

Services provide reusable functionality and are managed by the service container. Plugins implement specific plugin types and are managed by plugin managers.

### What is the difference between a Content Entity and Configuration Entity?

Content Entities store business content, while Configuration Entities store application configuration.

### What is the difference between `drush cim` and `drush cex`?

`cim` imports configuration, while `cex` exports active configuration to YAML files.

### What is the difference between Config API and State API?

Config API stores deployable configuration, while State API stores runtime or environment-specific information.

### What are Cache Tags, Contexts, and Max-Age?

Tags define what invalidates cache, contexts define variations, and max-age defines cache lifetime.

### What is the difference between Entity Query and Database API?

Entity Query works with entities, while Database API works directly with database tables.

### What is the difference between a Node and a Taxonomy Term?

A Node represents content, while a Taxonomy Term represents a classification value.

### What is the difference between a Block Plugin and a Block Entity?

A Block Plugin defines block functionality, while a Block Configuration Entity represents a configured placement of that block.

### What is the difference between FormBase and ConfigFormBase?

FormBase is for general forms, while ConfigFormBase is specifically designed for configuration forms.

### What is the difference between `hook_entity_presave()` and `hook_entity_insert()`?

`hook_entity_presave()` runs before an entity is saved to storage. `hook_entity_insert()` runs after a new entity has been inserted.

### What is the difference between Cron and Queue API?

Cron schedules tasks, while Queue API stores individual tasks for later processing.

### What is the difference between Queue API and Batch API?

Queue API is useful for background processing. Batch API is useful for long-running operations with progress feedback.

### What is the difference between JSON and REST?

JSON follows a standardized resource-based specification. Drupal REST provides configurable REST resources and serialization.

### What is the difference between Content Translation and Configuration Translation?

Content Translation translates website content, while Configuration Translation translates configuration values.

### What is the difference between a Custom Module and a Contributed Module?

A Custom Module is developed for a specific project. A Contributed Module is maintained by the Drupal community.

### Why should you use Dependency Injection?

It reduces coupling, makes dependencies explicit, and improves testability.

### Why should you avoid `\Drupal::service()`?

It creates hidden dependencies and makes code harder to test. Dependency Injection is preferred when possible.

### Why should you not put business logic in Controllers?

Controllers should handle HTTP requests and responses. Business logic belongs in services to improve reuse and maintainability.

### Why is cache metadata important?

Without correct cache metadata, Drupal can serve stale content or display user-specific data to the wrong users.

---

## 23. Drupal Practical Questions

### How would you create a custom module?

1. Create a module directory.
    
2. Add an `.info.yml` file.
    
3. Define services or routes if needed.
    
4. Implement controllers, plugins, forms, or hooks.
    
5. Enable the module.
    
6. Rebuild caches.
    

### How would you create a custom service?

Create a PHP class, define it in `mymodule.services.yml`, inject dependencies through the constructor, and use the service where needed.

### How would you create a custom route?

Define the route in `mymodule.routing.yml` and connect it to a controller or form.

### How would you create a custom Block?

Create a class extending `BlockBase` and define the Block plugin metadata.

### How would you create a custom Entity?

Define an entity class with the appropriate entity type annotation or attribute, storage information, handlers, and field definitions.

### How would you create a custom permission?

Define it in `mymodule.permissions.yml` and use it in route requirements or access checks.

### How would you modify an existing form?

Implement `hook_form_alter()` or a more specific form alter hook.

### How would you add a custom field to an entity?

Define the field storage and field configuration, usually through Drupal configuration management or the Field API.

### How would you add custom CSS and JavaScript?

Define a library in `.libraries.yml` and attach it to the relevant render array or theme.

### How would you expose custom data through an API?

Depending on requirements, I would use JSON for entity resources or create a custom REST resource for specific business operations.

### How would you migrate CSV data?

Use Migration API with a CSV source plugin, process plugins, and an entity destination plugin.

### How would you improve a slow Drupal page?

I would profile the page, inspect database queries, review cache metadata, optimize Views and entity loading, and introduce caching where appropriate.

### How would you secure a custom route?

I would define appropriate permissions or custom access checks, validate input, use CSRF protection for state-changing operations, and return proper access results.

### How would you implement background processing?

I would create a Queue Worker plugin, add items to the queue, and configure queue processing through cron or another worker mechanism.

### How would you handle a database schema change?

I would implement an update hook or an appropriate entity definition update, depending on the change, and test it against existing data.

---

## 24. Essential Drupal File Extensions

|File|Purpose|
|---|---|
|`.info.yml`|Module or theme definition|
|`.module`|Procedural hooks|
|`.install`|Installation and update hooks|
|`.services.yml`|Service definitions|
|`.routing.yml`|Route definitions|
|`.permissions.yml`|Custom permissions|
|`.links.menu.yml`|Menu links|
|`.links.task.yml`|Local tasks and tabs|
|`.libraries.yml`|CSS and JS libraries|
|`.links.action.yml`|Action links|
|`.links.contextual.yml`|Contextual links|
|`.theme`|Theme hooks and preprocess functions|
|`.schema.yml`|Configuration schema|
|`.settings.yml`|Default configuration|
|`.config.yml`|Configuration entity definitions|
|`.twig`|Twig templates|
|`.php`|PHP classes|
|`.yml`|YAML configuration|

---

## 25. Key Differences — Quick Revision

|Concept A|Concept B|
|---|---|
|Module — functionality|Theme — presentation|
|Node — content entity|View — configuration entity|
|Content Entity — business data|Config Entity — application settings|
|Hook — Drupal extension point|Event — event dispatcher mechanism|
|Service — reusable functionality|Plugin — extensible implementation|
|Service Container — manages services|Plugin Manager — manages plugins|
|Entity Query — entity-based query|Database API — direct database query|
|Config API — deployable settings|State API — runtime state|
|`drush cex` — export|`drush cim` — import|
|`hook_install()` — initial setup|Update Hook — update existing installation|
|`hook_entity_presave()` — before save|`hook_entity_insert()` — after insert|
|Cache Tags — invalidation|Cache Contexts — variations|
|Cache Max-Age — lifetime|Cache Tags — dependencies|
|FormBase — general forms|ConfigFormBase — configuration forms|
|JSON — standard specification|REST — configurable REST resources|
|Queue API — background tasks|Batch API — progressive operations|
|Content Translation — content|Configuration Translation — settings|
|Controller — HTTP handling|Service — business logic|
|Twig — presentation|PHP — application logic|
|Unit Test — isolated code|Kernel Test — Drupal integration|
|Kernel Test — minimal Drupal kernel|Functional Test — HTTP/browser behavior|

---

## 26. Questions You Should Be Able to Answer Without Thinking

1. What is Drupal?
    
2. What is Drupal Core?
    
3. What is a custom module?
    
4. What is the difference between a module and a theme?
    
5. What is an Entity?
    
6. What is the difference between Content Entity and Configuration Entity?
    
7. What is a Node?
    
8. What is a Bundle?
    
9. What is an Entity Reference?
    
10. What is the Entity API?
    
11. What are Hooks?
    
12. What is the difference between Hooks and Events?
    
13. What is an Event Subscriber?
    
14. What is a Service?
    
15. What is Dependency Injection?
    
16. What is a Service Container?
    
17. What is the difference between a Service and a Plugin?
    
18. What is a Plugin Manager?
    
19. What is a Block Plugin?
    
20. What is Routing?
    
21. What is a Controller?
    
22. What is a Render Array?
    
23. What is AccessResult?
    
24. What is Form API?
    
25. What is the difference between FormBase and ConfigFormBase?
    
26. What is Configuration Management?
    
27. What is the difference between Config API and State API?
    
28. What are Cache Tags, Contexts, and Max-Age?
    
29. What is Entity Query?
    
30. What is the difference between Entity Query and Database API?
    
31. What is JSON?
    
32. What is the difference between REST and JSON?
    
33. What is Migration API?
    
34. What are Source, Process, and Destination Plugins?
    
35. What is Queue API?
    
36. What is Batch API?
    
37. What is the difference between Cron and Queue API?
    
38. What is the difference between Content Translation and Configuration Translation?
    
39. What are Unit, Kernel, and Functional Tests?
    
40. How do you optimize Drupal performance?
    
41. How do you secure a custom route?
    
42. How do you create a Custom Entity?
    
43. How do you debug a Drupal application?
    
44. How do you deploy a Drupal application?
    
45. How do you approach an unfamiliar Drupal codebase?
    

---

## Final Interview Advice

For technical interview answers, use this structure:

1. **Definition** — explain what the concept is.
    
2. **Purpose** — explain why Drupal needs it.
    
3. **Practical example** — explain where you would use it.
    

Keep your answers clear and concise. If the interviewer wants more details, explain the implementation.

### Example answers

**Question: What is Dependency Injection in Drupal?**

"Dependency Injection is a technique where a class receives its dependencies from outside instead of creating them internally. Drupal uses a service container to manage services and their dependencies. I usually use constructor injection because it improves testability and reduces coupling."

**Question: What is the difference between Hooks and Events?**

"Hooks are Drupal-specific extension points that allow modules to alter or extend existing functionality. Events are based on Symfony's Event Dispatcher and allow multiple subscribers to react to the same event. I use hooks for standard Drupal extension points and events for decoupled event-driven logic."

**Question: What are Cache Tags, Contexts, and Max-Age?**

"Cache Tags define what data invalidates a cache entry. Cache Contexts define how cached output varies depending on the current context, such as user roles or language. Max-Age defines how long the cache remains valid."

**Question: What is the difference between a Service and a Plugin?**

"A Service provides reusable functionality and is managed by Drupal's service container. A Plugin is a reusable implementation of a specific plugin type and is managed by a plugin manager. For example, a custom business service can be injected into a Block Plugin."

**Question: What is Configuration Management?**

"Configuration Management allows us to export and import Drupal configuration between environments using YAML files. We normally use Drush commands such as config export and config import to synchronize configuration."

**Question: How do you improve Drupal performance?**

"First, I identify bottlenecks using profiling tools and logs. Then I optimize database queries, review cache tags and contexts, reduce unnecessary entity loading, and use appropriate caching. I always measure performance before and after optimization."

**Question: What is the difference between Entity Query and Database API?**

"Entity Query is used to retrieve Drupal entities using entity properties and fields. Database API is used for direct database queries. I prefer Entity Query when working with entities because it respects Drupal's Entity API abstraction."

**Question: How do you approach a new Drupal project?**

"First, I inspect the project structure, dependencies, enabled modules, and configuration. Then I identify the main entities, services, routes, and plugins. I also review existing tests and coding standards before implementing new functionality."

**The main goal:** understand Drupal's architecture, know which API to use for a particular task, and explain why you chose that approach.