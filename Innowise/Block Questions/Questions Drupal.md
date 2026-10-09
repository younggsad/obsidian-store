---
custom-width: 80
---
## Topic 1. Drupal Basics

### 1. What is Drupal, and what is it used for?

Drupal is a content management system (CMS) used to build and manage websites. It allows developers to create websites through a user interface and extend functionality with modules and custom code. Drupal provides many features that make it a powerful CMS.

### 2. What is Drupal Core, and why should developers avoid modifying it directly?

Drupal Core is the main codebase of Drupal and provides the basic functionality of the system. Developers should not modify core directly because their changes can cause problems during updates. Custom functionality should usually be implemented in custom modules or themes.

### 3. What is a Drupal module, and why do developers use modules?

A module is a collection of files that adds or changes website functionality. Developers use modules to extend Drupal and implement custom features. A module can contain PHP, YAML, and other files.

### 4. What is the difference between a Drupal module and a Drupal theme?

A Drupal theme controls the appearance and presentation of a website, while a module adds or changes its functionality. Themes commonly use Twig templates, CSS, and JavaScript. Modules can contain business logic, routes, controllers, and other functionality.

## Topic 2. Entities and Content

### 5. What is a Drupal node, and how is it related to a content type?

A node is a content entity that represents a specific piece of content. A content type is a bundle of the node entity type that defines the structure of that content. For example, Task and Article can be different content types.

### 6. What is a field in Drupal, and why are fields important for content types?

A field is a structured piece of data that stores specific information in an entity. Fields define what information a content item can contain. For example, a Task can have fields for status, assignee, estimate, and project.

### 7. What is an entity in Drupal, and what is the difference between a content entity and a configuration entity?

An entity in Drupal is a structured object that represents something in the system. Content entities store content data, while configuration entities store configuration and settings. For example, a node is a content entity, while a View is a configuration entity.

## Topic 3. Routes and Controllers

### 8. What is a Drupal route, and what happens when a user visits a route?

A Drupal route defines a URL path that a user can access. When a user visits the URL, Drupal finds the matching route and checks access requirements. The route can then call a controller or another handler to generate a response.

### 9. What is a Drupal controller, and what is its responsibility?

A controller is a PHP class that handles a request for a specific route. It can use services, process request data, and return a response, such as a render array or JSON response. Controllers should not contain all the application's business logic.

## Topic 4. Services and Dependency Injection

### 10. What is a Drupal service, and why are services used?

A service is an object that provides a specific piece of functionality. Drupal uses services to perform tasks such as managing entities, accessing the database, and sending emails. Services help developers reuse functionality across different parts of an application.

### 11. Why is dependency injection preferred over creating services directly inside a Drupal class?

Dependency injection allows us to pass dependencies into a class, usually through its constructor. It reduces coupling and makes classes easier to test and maintain. It also makes it easier to replace a dependency without changing the class's main logic.

## Topic 5. Hooks and Plugins

### 12. What is a hook in Drupal, and why are hooks useful for module development?

A hook allows a module to react to a specific event or alter Drupal's behavior. Drupal invokes hooks at specific points in its processes. For example, `hook_entity_presave()` allows a module to react before an entity is saved.

### 13. What is the difference between a hook and a service in Drupal?

A hook is an extension point that allows a module to react to a Drupal process. Drupal calls the hook at the appropriate time. A service is an object that provides reusable functionality and can be called by different parts of the application.

### 14. What is a Drupal plugin, and how is it different from a service?

A plugin is an object that provides a specific type of functionality within Drupal's plugin system. Drupal discovers and uses plugins in the appropriate context. A service provides reusable functionality, while a plugin implements a specific type of behavior, such as a block or field formatter.

### 15. Can a Drupal plugin use a service? Explain why or why not.

Yes, a Drupal plugin can use a service through dependency injection. For example, a block plugin can use the entity type manager service to load entities and display their data. This allows plugins to reuse Drupal's existing functionality.

## Topic 6. Views

### 16. What is a Drupal View, and why do developers use Views?

Drupal Views is a tool for querying, filtering, sorting, and displaying data without writing custom SQL queries manually. Developers use Views to create lists, tables, pages, and blocks. For example, a View can display tasks filtered by project or status.

### 17. What is the difference between a Drupal View and a custom controller?

A View is a tool for getting, filtering, sorting, and displaying data through Drupal's configuration interface. A custom controller is a PHP class that handles a request for a specific route and returns a response. Views are useful for standard data displays, while controllers provide more control over custom request handling and logic.

## Topic 7. Configuration and Fields

### 18. What is the difference between configuration and content in Drupal?

Configuration consists of settings and values that define how a Drupal website works. Content consists of information stored on the website, such as tasks, projects, and articles. For example, a Workflow is configuration, while a specific Task is content.

### 19. What is the difference between a content type and a field in Drupal?

A content type defines what data a content item can store, while a field stores a specific piece of information. For example, the Task content type can have a status field, an assignee field, and an estimate field. Each field has a data type and stores values of that type.

### 20. What is the difference between a node and a content type in Drupal?

A node is a content entity that represents a specific piece of content. A content type defines the structure of that content, while a node stores the actual values. For example, Task is a content type, and "Fix login bug" can be a node created from it.