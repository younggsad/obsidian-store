---
custom-width: 80
---
## Block 1 — Drupal Basics

### 1. What is Drupal, and what is it used for?

**Answer:**

Drupal is a content management system (CMS) used to build and manage websites. It allows developers to create websites through a user interface and extend functionality with modules and custom code. Drupal provides many features that make it a powerful CMS.

**Explanation:**

A CMS helps users create, edit, organize, and display website content without building every feature from scratch.

In DrupalJira, Drupal stores projects and tasks and displays them on a task board.

**Key terms:**

- CMS — Content Management System.
    
- Content — information stored on a website.
    
- Module — a package that adds or changes functionality.
    

### 2. What is Drupal Core, and why should developers avoid modifying it directly?

**Answer:**

Drupal Core is the main codebase of Drupal and provides the basic functionality of the system. Developers should not modify Core directly because their changes can cause problems during updates. Custom functionality should usually be implemented in custom modules or themes.

**Explanation:**

Drupal Core provides essential functionality, including routing, entity APIs, forms, user management, and rendering.

If you modify Core directly, your changes may be overwritten during an update.

**Best practice:** Use custom modules to add functionality and custom themes to control presentation.

### 3. What is a Drupal module, and why do developers use modules?

**Answer:**

A module is a collection of files that adds or changes website functionality. Developers use modules to extend Drupal and implement custom features. A module can contain PHP, YAML, and other files.

**Explanation:**

Modules allow developers to add functionality without modifying Drupal Core.

Examples from DrupalJira:

- `drupaljira_board` — provides task board functionality.
    
- `drupaljira_timelog` — provides time-tracking functionality.
    

### 4. What is the difference between a Drupal module and a Drupal theme?

**Answer:**

A Drupal theme controls the appearance and presentation of a website, while a module adds or changes its functionality. Themes commonly use Twig templates, CSS, and JavaScript. Modules can contain business logic, routes, controllers, and other functionality.

**Explanation:**

A module is mainly responsible for what the website does. A theme is mainly responsible for how the website looks.

Examples:

- Module: checks whether a user can update a task.
    
- Theme: controls the task card's layout and colors.
    

**Key terms:**

- Business logic — rules that define how an application works.
    
- Twig — Drupal's template engine.
    
- CSS — controls visual presentation.
    

### 5. What is a Drupal node, and how is it related to a content type?

**Answer:**

A node is a content entity that represents a specific piece of content. A content type is a bundle of the node entity type that defines the structure of that content. For example, Task and Article can be different content types.

**Explanation:**

A content type defines the structure. A node is an individual content item created using that structure.

In DrupalJira:

- `Task` is a content type.
    
- `Project` is a content type.
    
- A task named `Fix login bug` is an individual node.
    

### 6. What is a field in Drupal, and why are fields important for content types?

**Answer:**

A field is a structured piece of data that stores specific information in an entity. Fields define what information a content item can contain. For example, a Task can have fields for status, assignee, estimate, and project.

**Explanation:**

Fields allow Drupal to store information in a structured way instead of keeping everything in one text field.

Examples from DrupalJira:

- `field_status` — task status.
    
- `field_assignee` — assigned user.
    
- `field_estimate` — estimated effort.
    
- `field_project` — related project.
    

### 7. What is an entity in Drupal, and what is the difference between a content entity and a configuration entity?

**Answer:**

An entity in Drupal is a structured object that represents something in the system. Content entities store content data, while configuration entities store configuration and settings. For example, a node is a content entity, while a View is a configuration entity.

**Explanation:**

Content entities represent content or data managed by the website.

Examples:

- Node.
    
- User.
    
- Media.
    
- A custom TimeLog entity.
    

Configuration entities represent configurable parts of Drupal.

Examples:

- Content type definitions.
    
- Views.
    
- Workflows.
    
- Field storage and field configuration.
    

### 8. What is a Drupal route, and what happens when a user visits a route?

**Answer:**

A Drupal route defines a URL path that a user can access. When a user visits the URL, Drupal finds the matching route and checks access requirements. The route can then call a controller or another handler to generate a response.

**Explanation:**

A route connects a URL to the code responsible for handling the request.

Example:

```php
drupaljira_board.project_board:
  path: '/project/{node}/board'
  defaults:
    _controller: '\Drupal\drupaljira_board\Controller\TaskBoardController::build'
    _title: 'Project Board'
  requirements:
    _permission: 'access content'
```

This example illustrates the basic structure. Real routes should use access requirements appropriate to the operation and entity.

### 9. What is a Drupal controller, and what is its responsibility?

**Answer:**

A controller is a PHP class that handles a request for a specific route. It can use services, process request data, and return a response, such as a render array or JSON response. Controllers should not contain all the application's business logic.

**Explanation:**

A controller coordinates request handling. It can retrieve route parameters, call services, and prepare the response.

Example:

```php
public function build(): array {
  return [
    '#markup' => 'Hello, Drupal!',
  ];
}
```

This controller method returns a render array.

### 10. What is a Drupal service, and why are services used?

**Answer:**

A service is an object that provides a specific piece of functionality. Drupal uses services to perform tasks such as managing entities, accessing the database, and sending emails. Services help developers reuse functionality across different parts of an application.

**Explanation:**

Services provide reusable functionality and are managed by Drupal's service container.

Examples:

- `entity_type.manager` — manages entity types and storage handlers.
    
- `current_user` — provides information about the current user.
    
- `logger.factory` — provides logging channels.
    
- `database` — provides database access.
    

## Block 2 — Hooks, Plugins, and Dependency Injection

### 11. Why is dependency injection preferred over creating services directly inside a Drupal class?

**Answer:**

Dependency injection allows us to pass dependencies into a class, usually through its constructor. It reduces coupling and makes classes easier to test and maintain. It also makes it easier to replace a dependency without changing the class's main logic.

**Explanation:**

A dependency is an object or service that a class needs to perform its work.

Example:

```php
use Drupal\Core\Entity\EntityTypeManagerInterface;

final class TaskManager {

  public function __construct(
    private EntityTypeManagerInterface $entityTypeManager,
  ) {}

}
```

The service is passed into the class instead of being retrieved from the global container inside each method.

### 12. What is a hook in Drupal, and why are hooks useful for module development?

**Answer:**

A hook allows a module to react to a specific event or alter Drupal's behavior. Drupal invokes hooks at specific points in its processes. For example, `hook_entity_presave()` allows a module to react before an entity is saved.

**Explanation:**

Hooks provide extension points that let modules interact with Drupal's existing behavior.

Example:

```php
function mymodule_entity_presave(
  \Drupal\Core\Entity\EntityInterface $entity,
): void {
  // React before the entity is saved.
}
```

In DrupalJira, a presave hook can synchronize a task's status field with its moderation state before the entity is saved.

### 13. What is the difference between a hook and a service in Drupal?

**Answer:**

A hook is an extension point that allows a module to react to a Drupal process. Drupal calls the hook at the appropriate time. A service is an object that provides reusable functionality and can be called by different parts of an application.

**Explanation:**

- A hook defines when a module can react.
    
- A service provides functionality that can be reused.
    

A hook can call a service to perform more complex operations.

### 14. What is a Drupal plugin, and how is it different from a service?

**Answer:**

A plugin is an object that provides a specific type of functionality within Drupal's plugin system. Drupal discovers and uses plugins in the appropriate context. A service provides reusable functionality, while a plugin implements a specific type of behavior, such as a block or field formatter.

**Explanation:**

Plugins are used when Drupal supports a particular type of extensible component.

Examples:

- Block plugins.
    
- Field widgets.
    
- Field formatters.
    
- Views plugins.
    

A service can be used by many different classes. A plugin usually follows a specific plugin type's API and lifecycle.

### 15. Can a Drupal plugin use a service? Explain why or why not.

**Answer:**

Yes, a Drupal plugin can use a service through dependency injection. For example, a block plugin can use the entity type manager service to load entities and display their data. This allows plugins to reuse Drupal's existing functionality.

**Explanation:**

Dependency injection makes services available to plugins without requiring the plugin to retrieve them directly from the global container.

Many plugins that need constructor injection implement `ContainerFactoryPluginInterface`.

### 16. What is a Drupal View, and why do developers use Views?

**Answer:**

Drupal Views is a tool for querying, filtering, sorting, and displaying data without writing custom SQL queries manually. Developers use Views to create lists, tables, pages, and blocks. For example, a View can display tasks filtered by project or status.

**Explanation:**

Views is useful for standard data listings and many common content-display requirements.

In DrupalJira, the `task_board` View provides a way to retrieve and display tasks.

### 17. What is the difference between a Drupal View and a custom controller?

**Answer:**

A View is a tool for getting, filtering, sorting, and displaying data through Drupal's configuration interface. A custom controller is a PHP class that handles a request for a specific route and returns a response. Views are useful for standard data displays, while controllers provide more control over custom request handling and logic.

**Explanation:**

Use Views when the requirements fit its query and display capabilities. Use a custom controller when you need custom request handling or response preparation.

They can also work together. A controller can use services or render elements that display data produced by Views.

### 18. What is the difference between configuration and content in Drupal?

**Answer:**

Configuration consists of settings and values that define how a Drupal website works. Content consists of information stored on the website, such as tasks, projects, and articles. For example, a Workflow is configuration, while a specific Task is content.

**Explanation:**

Drupal configuration can be exported and imported, commonly as YAML files.

Examples of configuration:

- Content type definitions.
    
- Field definitions.
    
- Views.
    
- Workflows.
    
- Display settings.
    

Examples of content:

- Individual task nodes.
    
- Project nodes.
    
- Uploaded media items.
    

### 19. What is the difference between a content type and a field in Drupal?

**Answer:**

A content type defines what data a content item can store, while a field stores a specific piece of information. For example, the Task content type can have a status field, an assignee field, and an estimate field. Each field has a data type and stores values of that type.

**Explanation:**

A content type groups together the fields that define the structure of a type of content.

For example, a Task content type can contain `field_status`, `field_assignee`, and `field_estimate`.

### 20. What is the difference between a node and a content type in Drupal?

**Answer:**

A node is a content entity that represents a specific piece of content. A content type defines the structure of that content, while a node stores the actual values. For example, Task is a content type, and `Fix login bug` can be a node created from it.

**Explanation:**

Think of a content type as a template for the structure of content and a node as one actual content item.

### 21. What is an entity reference field in Drupal, and how can it be used?

**Answer:**

An entity reference field in Drupal is a field that connects one Drupal entity to another entity. It allows content to reference existing entities instead of storing their information as plain text.

**Explanation:**

In DrupalJira, `field_project` on a Task references a Project entity.

This allows Drupal to maintain a relationship between the task and the project.

### 22. What is the difference between an entity reference field and a regular text field in Drupal?

**Answer:**

A regular text field stores text, while an entity reference field stores a reference to another entity. An entity reference field allows Drupal to maintain a relationship between entities.

**Explanation:**

A text field containing `Website Redesign` stores only text.

An entity reference field pointing to the Website Redesign project refers to an actual Project entity. Drupal can use that relationship for entity loading, access checks, and related queries.

### 23. What is the difference between a required field and an optional field in Drupal?

**Answer:**

A required field must contain a value, while an optional field does not have to. Required fields are used when information is necessary, whereas optional fields allow users to leave information empty.

**Explanation:**

Drupal forms normally validate required fields before saving content.

In DrupalJira, `field_project` can be required because every task must belong to a project.

When creating entities programmatically, developers should also ensure that entity validation requirements are satisfied where appropriate.

### 24. What is the difference between field configuration and field value in Drupal?

**Answer:**

Field configuration defines the settings of a field, while a field value is the actual data stored in that field. Configuration determines how the field works, and the value represents information for a specific entity.

**Explanation:**

For example, the configuration for `field_status` defines which status values are allowed.

A particular task might have the value `in_progress`.

The field configuration describes the field; the field value belongs to a specific entity.

### 25. What is the difference between a field type and a field widget in Drupal?

**Answer:**

A field type defines what kind of data a field stores and how that data is structured. A field widget defines how a user enters or selects a field value in an editing form.

**Explanation:**

Examples of field types:

- Text.
    
- Integer.
    
- Decimal.
    
- Entity Reference.
    

Examples of widgets:

- Text field.
    
- Select list.
    
- Autocomplete.
    

The field type defines the data structure, while the widget defines the form input.

### 26. What is a field formatter in Drupal, and how is it different from a field widget?

**Answer:**

A field formatter determines how a field value is displayed to the user. A field widget defines how the user enters or selects a field value in an editing form.

**Explanation:**

For `field_estimate`:

- The widget controls how a user enters an estimate.
    
- The formatter controls how the estimate appears when viewing a task.
    

Widgets are for input. Formatters are for display.

## Block 3 — Drupal Configuration and Dependency Injection

### 27. What is the difference between a content entity and a configuration entity in Drupal?

**Answer:**

A content entity stores content data, such as tasks, projects, and articles. A configuration entity represents configurable objects, such as content types, Views, and workflows.

**Explanation:**

Examples of content entities:

- An individual Task node.
    
- A Project node.
    
- A User entity.
    

Examples of configuration entities:

- The Task content type definition.
    
- The `task_board` View.
    
- The `task_workflow` workflow.
    

The key difference is whether the entity represents content data or a configurable object.

### 28. What is the difference between a module's `.info.yml` file and its `.routing.yml` file?

**Answer:**

A module's `.info.yml` file contains metadata about the module, such as its name, description, and dependencies. A module's `.routing.yml` file defines URL paths, access requirements, and the controllers or other handlers that respond to requests.

**Explanation:**

Example `drupaljira_board.info.yml`:

```php
name: DrupalJira Board
type: module
description: Provides task board functionality.
core_version_requirement: ^11
```

The `.info.yml` file describes the module to Drupal.

The `.routing.yml` file defines the routes provided by the module.

### 29. What is the purpose of a `.services.yml` file in a Drupal module?

**Answer:**

A `.services.yml` file defines services that can be used throughout a Drupal project. It tells Drupal which service classes to register and how to configure their dependencies.

**Explanation:**

Example `drupaljira_board.services.yml`:

```php
services:
  drupaljira_board.task_manager:
    class: Drupal\drupaljira_board\Service\TaskManager
```

This registers a service with the ID `drupaljira_board.task_manager`.

Other classes can request the service through dependency injection.

### 30. What is dependency injection in Drupal, and why is it useful?

**Answer:**

Dependency injection is a technique for passing dependencies into a class, usually through its constructor. It reduces coupling and makes code easier to test and maintain.

**Explanation:**

Instead of creating or retrieving dependencies inside a class, provide them from outside.

Example:

```php
use Drupal\Core\Entity\EntityTypeManagerInterface;

final class TaskManager {

  public function __construct(
    private EntityTypeManagerInterface $entityTypeManager,
  ) {}

}
```

The class receives an entity type manager and can use it to work with Drupal entities.

### 31. What is the difference between dependency injection and calling `\Drupal::service()` directly in a Drupal class?

**Answer:**

Calling `\Drupal::service()` directly accesses a service through Drupal's global service container. Dependency injection passes the required service into a class explicitly. Dependency injection is generally preferred in classes that support it because it reduces coupling and improves testability.

**Explanation:**

Direct service lookup:

```php
$entityTypeManager = \Drupal::service('entity_type.manager');
```

Dependency injection makes the dependency explicit in the class constructor.

Dependency injection is generally preferred in controllers, services, and other classes that support it. Direct service lookup can still be appropriate in certain procedural contexts.

## Block 4 — Access Control and HTTP Methods

### 32. What is the purpose of access control in Drupal, and why is it important?

**Answer:**

The main purpose of access control is to manage permissions and protect data from unauthorized access. It ensures that users can perform only the actions they are allowed to perform.

**Explanation:**

Access control determines whether a user can view, create, update, or delete content and perform other protected actions.

Hiding a button in the interface is not enough. Drupal must enforce access rules on the server side.

In DrupalJira, a user should not be able to update a task merely by manually sending a request to a route.

### 33. What is the difference between authentication and authorization in Drupal?

**Answer:**

Authentication verifies a user's identity, while authorization determines what the user is allowed to do.

**Explanation:**

Authentication answers: "Who are you?"

Authorization answers: "What are you allowed to do?"

A user can be successfully authenticated but still lack permission to edit a particular task.

### 34. What is the difference between a Drupal permission and entity access?

**Answer:**

A permission defines which actions a user is generally allowed to perform. Entity access determines whether the user can perform a specific action on a particular entity.

**Explanation:**

Check a general permission:

```php
if ($account->hasPermission('access content')) {
  // The account has this permission.
}
```

Check access to a specific task:

```php
$access = $task->access('update', $account);

if ($access->isAllowed()) {
  // The account can update this task.
}
```

The `access()` method returns an access result. Checking a general permission does not automatically guarantee access to every entity.

### 35. What is CSRF protection in Drupal, and why is it important for routes that modify data?

**Answer:**

CSRF protection helps protect a website from cross-site request forgery attacks. Drupal uses CSRF tokens to verify certain requests and help prevent unauthorized actions that modify data.

**Explanation:**

A CSRF attack attempts to make a user's browser submit an unwanted request while the user is authenticated.

For example, an attacker might try to make a logged-in user submit a request that changes a task's status.

CSRF protection is not a replacement for permissions or entity access checks. A route that changes data must still enforce appropriate authorization.

The exact token requirements depend on the route and request mechanism.

### 36. Why should a Drupal route that modifies data usually use the POST method instead of GET?

**Answer:**

GET is intended for retrieving data and should not modify the application's state. POST is commonly used to submit data or perform actions that change the state of a website. Using GET for data modification can cause unexpected side effects and security problems.

**Explanation:**

HTTP methods have different intended purposes:

- GET — retrieve data.
    
- POST — submit data or perform an operation.
    
- PUT — replace a resource.
    
- PATCH — partially update a resource.
    
- DELETE — delete a resource.
    

CRUD stands for Create, Read, Update, and Delete.

The HTTP method alone does not provide access control or CSRF protection. State-changing routes still need appropriate security checks.

## Quick Revision

|Concept|Meaning|
|---|---|
|Drupal Core|The main Drupal codebase|
|Module|Adds or changes functionality|
|Theme|Controls presentation|
|Node|An individual content item|
|Content type|Defines the structure of a type of node|
|Field|Stores a specific piece of data|
|Entity|A structured object in Drupal|
|Content entity|Represents content or application data|
|Configuration entity|Represents a configurable object|
|Route|Maps a URL to a request handler|
|Controller|Handles a request and returns a response|
|Service|Provides reusable functionality|
|Dependency injection|Passes dependencies into a class|
|Hook|Lets a module react to a Drupal process|
|Plugin|Implements a specific type of extensible behavior|
|View|Queries and displays data through configuration|
|Entity reference|Links one entity to another|
|Widget|Defines how a field is entered|
|Formatter|Defines how a field is displayed|
|Authentication|Verifies identity|
|Authorization|Determines allowed actions|
|Permission|Defines a general capability|
|Entity access|Checks access to a specific entity operation|
|CSRF protection|Helps prevent forged requests|
|GET|Retrieves data; should not change application state|
|POST|Commonly submits data or performs state-changing operations|
