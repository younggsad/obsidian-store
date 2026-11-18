---
custom-width: 80
---
## **1. PHP Fundamentals**

### **What is PHP?**

PHP is a server-side scripting language mainly used for web development. It can generate dynamic pages, communicate with databases, process requests, and build APIs.

### **Is PHP interpreted or compiled?**

PHP is compiled into bytecode (opcodes) and executed by the Zend Engine. OPcache can store compiled bytecode to avoid recompilation on every request.

### **What are the main PHP data types?**

- **Scalar:** `int`, `float`, `string`, `bool`.
    
- **Compound:** `array`, `object`, `callable`, `iterable`.
    
- **Special:** `null`, `resource`.
    

### **What is the difference between** `==` **and** `===`**?**

- `==` compares values after type conversion.
    
- `===` compares both value and type.
    

```
5 == "5";   // true
5 === "5";  // false
```

I generally prefer strict comparison because it makes type conversion explicit and prevents unexpected behavior.

### **What is the difference between** `isset()` **and** `empty()`**?**

- `isset()` checks whether a variable exists and is not `null`.
    
- `empty()` checks whether a variable does not exist or contains a false-like value.
    

```
$value = 0;

isset($value); // true
empty($value); // true
```

Values such as `0`, `'0'`, `false`, `null`, and `''` are considered empty by `empty()`.

### **What is the difference between** `null`**,** `false`**, and** `0`**?**

- `null` means that a variable has no value.
    
- `false` is a boolean false value.
    
- `0` is an integer.
    

They are different values, even though all of them evaluate to `false` in a boolean context.

### **What is type juggling?**

Type juggling is automatic type conversion performed by PHP.

```
$result = "10" + 5; // 15
```

PHP converts the numeric string into an integer before performing the operation.

### **What is strict typing?**

Strict typing enables stricter scalar type checking for function calls from the declaring file.

```
<?php

declare(strict_types=1);

function calculate(int $number): int
{
    return $number * 2;
}
```

It helps prevent unexpected type conversions and makes function contracts more predictable.

### **What is the difference between** `include` **and** `require`**?**

- `include` produces a warning if the file cannot be loaded, and execution may continue.
    
- `require` produces an error and stops normal execution.
    

`include_once` and `require_once` prevent the same file from being included multiple times.

### **What is the difference between** `echo` **and** `print`**?**

- `echo` can output multiple arguments and has no return value.
    
- `print` outputs one argument and returns `1`.
    

In practice, `echo` is more commonly used.

### **What is the difference between** `isset()` **and** `array_key_exists()`**?**

- `isset()` checks whether a variable or array key exists and its value is not `null`.
    
- `array_key_exists()` checks whether a key exists, even if its value is `null`.
    

```
$user = [
    'name' => null,
];

isset($user['name']); // false

array_key_exists('name', $user); // true
```

I use `array_key_exists()` when the difference between a missing key and a key containing `null` is important.

### **What is the difference between** `is_null()` **and** `isset()`**?**

- `is_null()` checks whether a value is `null`.
    
- `isset()` checks whether a variable exists and is not `null`.
    

```
$value = null;

is_null($value); // true
isset($value);   // false
```

### **What is the difference between** `var_dump()` **and** `print_r()`**?**

- `var_dump()` displays the value and its data type.
    
- `print_r()` displays human-readable information about a variable.
    

I generally use `var_dump()` when I need to inspect types and `print_r()` when I want to inspect an array or object structure.

## **2. Variables, Constants and Operators**

### **What is the difference between** `const` **and** `define()`**?**

Both create constants.

- `const` is language-level syntax and can be used for class constants.
    
- `define()` creates constants at runtime.
    

Constants cannot be reassigned after they have been defined.

### **What is the difference between** `??` **and** `?:`**?**

- `??` checks whether a variable exists and is not `null`.
    
- `?:` checks whether a value evaluates to `true`.
    

```
$name = $input ?? 'Guest';
$name = $input ?: 'Guest';
```

For example, if `$input` is `0`, the null coalescing operator returns `0`, while the ternary shorthand returns `'Guest'`.

### **What is the null coalescing assignment operator?**

The `??=` operator assigns a value only if the variable does not exist or is `null`.

```
$name = null;

$name ??= 'Guest';

echo $name; // Guest
```

It is a shorter alternative to checking for `null` before assigning a default value.

### **What is the nullsafe operator?**

The nullsafe operator `?->` allows method or property access without checking for `null` manually.

```
$name = $user?->getProfile()?->getName();
```

If any step returns `null`, the whole expression returns `null`.

### **What is the difference between** `match` **and** `switch`**?**

- `switch` uses loose comparison.
    
- `match` uses strict comparison.
    
- `match` returns a value.
    
- `match` does not fall through between branches.
    
- `match` must be exhaustive or have a `default` branch.
    

```
$result = match ($status) {
    'active' => 'Enabled',
    'inactive' => 'Disabled',
    default => 'Unknown',
};
```

I use `match` when I need to return a value based on several possible conditions.

### **What is the ternary operator?**

The ternary operator is a shorthand for a simple `if/else` expression.

```
$status = $isActive ? 'active' : 'inactive';
```

It evaluates the condition and returns one of two values.

### **What are logical operators in PHP?**

PHP provides several logical operators:

- `&&` — logical AND.
    
- `||` — logical OR.
    
- `!` — logical NOT.
    
- `and` — logical AND with lower precedence.
    
- `or` — logical OR with lower precedence.
    
- `xor` — true when exactly one operand is true.
    

I generally prefer `&&` and `||` because their precedence is more predictable in expressions.

### **What is the difference between** `&&` **and** `and`**?**

Both represent logical AND, but they have different operator precedence.

```
$result = true && false; // false

$result = true and false;
// Assignment happens before "and".
```

I prefer `&&` to avoid precedence-related bugs.

### **What is the spaceship operator?**

The spaceship operator `<=>` compares two values.

It returns:

- `-1` if the left value is smaller.
    
- `0` if both values are equal.
    
- `1` if the left value is greater.
    

```
1 <=> 2; // -1
2 <=> 2; // 0
3 <=> 2; // 1
```

It is commonly used in custom sorting callbacks.

### **What is the difference between pre-increment and post-increment?**

- `++$value` increments the value and then returns it.
    
- `$value++` returns the current value and increments it afterward.
    

```
$value = 5;

echo ++$value; // 6

$value = 5;

echo $value++; // 5
```

## **3. Arrays and Iteration**

### **What types of arrays exist in PHP?**

PHP has one flexible array type that can work as:

### **1. Indexed Array**

An **indexed array** is an array where elements are stored using numeric keys, starting from `0` by default. Each element is accessed by its numeric index (position).

```
$numbers = [10, 20, 30];

echo $numbers[0]; // 10
echo $numbers[1]; // 20
echo $numbers[2]; // 30
```

**Key points:**

- Numeric keys are assigned automatically if they are not specified.
    
- The first element has index `0`.
    
- You access elements using their numeric index.
    

### **2. Associative Array**

An **associative array** is an array where elements are stored as `key => value` pairs. You define the keys yourself, usually using strings that describe the values.

```
$user = [
    'name' => 'Ivan',
    'age' => 25,
    'email' => 'ivan@example.com',
];

echo $user['name']; // Ivan
echo $user['age'];  // 25
```

**Key points:**

- Keys are usually strings, but they can also be integers.
    
- Each value is associated with a specific key.
    
- You access elements using their keys instead of their positions.
    

### **3. Multidimensional Array**

A **multidimensional array** is an array that contains other arrays as its elements. It is commonly used to represent structured data, such as lists of users, products, or orders.

```
$users = [
    ['id' => 1, 'name' => 'Ivan'],
    ['id' => 2, 'name' => 'Anna'],
];

echo $users[0]['id'];   // 1
echo $users[1]['name']; // Anna
```

**Key points:**

- An array can contain other arrays at any nesting level.
    
- You access nested values using a chain of keys or indexes.
    
- It is useful for storing collections of structured records.

### **How are PHP arrays implemented?**

PHP arrays are ordered maps. They can use both integer and string keys and preserve insertion order.

Unlike arrays in some other languages, PHP arrays are not simple continuous numeric lists. They can contain different value types and non-sequential keys.

### **What happens when you use a duplicate array key?**

If an array contains the same key more than once, the latest value overwrites the previous value.

```
$user = [
    'name' => 'John',
    'name' => 'Anna',
];

echo $user['name']; // Anna
```

### **What is the difference between** `array_map()`**,** `array_filter()`**, and** `array_reduce()`**?**

- `array_map()` transforms every element and returns a new array.
    
- `array_filter()` selects elements based on a condition.
    
- `array_reduce()` combines elements into one result.
    

```
$numbers = [1, 2, 3, 4];

$mapped = array_map(
    fn($number) => $number * 2,
    $numbers
);

$filtered = array_filter(
    $numbers,
    fn($number) => $number > 2
);

$total = array_reduce(
    $numbers,
    fn($carry, $number) => $carry + $number,
    0
);
```

### **Does** `array_filter()` **preserve array keys?**

Yes. `array_filter()` preserves the original keys of the elements that pass the condition.

```
$numbers = [
    0 => 10,
    1 => 20,
    2 => 30,
];

$result = array_filter(
    $numbers,
    fn($number) => $number > 10
);
```

The result is:

```
[
    1 => 20,
    2 => 30,
]
```

If I need sequential numeric keys, I can use `array_values()`.

```
$result = array_values($result);
```

### **Does** `array_map()` **modify the original array?**

No. `array_map()` returns a new array and does not modify the original array by itself.

```
$numbers = [1, 2, 3];

$result = array_map(
    fn($number) => $number * 2,
    $numbers
);

// $numbers: [1, 2, 3]
// $result:  [2, 4, 6]
```

However, objects inside an array are still object handles. Modifying an object inside a callback can affect that object.

### **What is the difference between** `array_filter()` **and a foreach condition?**

Both can filter array elements.

```
$activeUsers = array_filter(
    $users,
    fn($user) => $user['active'] === true
);
```

The same logic can be implemented using `foreach`:

```
$activeUsers = [];

foreach ($users as $user) {
    if ($user['active'] === true) {
        $activeUsers[] = $user;
    }
}
```

`array_filter()` is concise for simple filtering. `foreach` is more flexible when several operations are required.

### **What is the difference between** `array_column()` **and** `array_map()`**?**

- `array_column()` extracts values from a specific column of a multidimensional array.
    
- `array_map()` applies a callback to each element and can perform more complex transformations.
    

```
$users = [
    ['id' => 1, 'name' => 'John'],
    ['id' => 2, 'name' => 'Anna'],
];

$names = array_column($users, 'name');
```

The result is:

```
['John', 'Anna']
```

I use `array_column()` when I only need values from one field.

### **What is the difference between** `in_array()` **and** `array_key_exists()`**?**

- `in_array()` searches for a value in an array.
    
- `array_key_exists()` checks whether a specific key exists.
    

```
$roles = [
    'admin' => 1,
    'editor' => 2,
];

in_array(1, $roles, true); // true

array_key_exists('admin', $roles); // true
```

I use strict comparison with `in_array()` by passing `true` as the third argument.

### **Why should I use strict comparison with** `in_array()`**?**

Without the third argument, `in_array()` uses loose comparison. This can produce unexpected results when values have different types.

```
$values = ['10', '20'];

in_array(10, $values);       // true
in_array(10, $values, true); // false
```

Strict comparison checks both the value and its type.

### **What is the difference between** `array_key_exists()` **and** `isset()` **for arrays?**

- `array_key_exists()` returns `true` if the key exists, even when its value is `null`.
    
- `isset()` returns `false` if the key does not exist or its value is `null`.
    

```
$data = [
    'name' => null,
];

array_key_exists('name', $data); // true
isset($data['name']);             // false
```

### **What is the difference between** `unset()` **and** `array_values()`**?**

- `unset()` removes a variable or an array element.
    
- `array_values()` returns an array containing the values with new sequential numeric keys.
    

```
$numbers = [10, 20, 30];

unset($numbers[1]);

// Keys: 0 and 2.

$numbers = array_values($numbers);

// Keys: 0 and 1.
```

`unset()` does not automatically reindex numeric keys.

### **How do you remove an element from an array?**

I can use `unset()` to remove an element by its key.

```
$users = [
    'john',
    'anna',
    'peter',
];

unset($users[1]);
```

If I need sequential numeric keys afterward, I can call `array_values()`.

### **What is the difference between** `array_merge()` **and the** `+` **operator?**

- `array_merge()` combines arrays. Numeric keys are reindexed, and later string keys overwrite earlier ones.
    
- `+` is the array union operator. Existing keys from the left array take priority.
    

```
$a = [
    'name' => 'Alex',
    0 => 'PHP',
];

$b = [
    'name' => 'John',
    0 => 'Java',
];

$result = array_merge($a, $b);
```

The result is:

```
[
    'name' => 'John',
    0 => 'PHP',
    1 => 'Java',
]
```

Using the union operator:

```
$result = $a + $b;
```

The result is:

```
[
    'name' => 'Alex',
    0 => 'PHP',
]
```

The left array has priority when keys are duplicated.

### **How does** `array_merge()` **handle numeric keys?**

`array_merge()` reindexes numeric keys starting from zero.

```
$a = [
    0 => 'PHP',
    1 => 'Laravel',
];

$b = [
    0 => 'Java',
    2 => 'Drupal',
];

$result = array_merge($a, $b);
```

The result is:

```
[
    0 => 'PHP',
    1 => 'Laravel',
    2 => 'Java',
    3 => 'Drupal',
]
```

### **How does the array union operator handle numeric keys?**

The `+` operator preserves keys and gives priority to the left array.

```
$a = [
    0 => 'PHP',
    1 => 'Laravel',
];

$b = [
    0 => 'Java',
    2 => 'Drupal',
];

$result = $a + $b;
```

The result is:

```
[
    0 => 'PHP',
    1 => 'Laravel',
    2 => 'Drupal',
]
```

The value at key `0` from `$b` is ignored because that key already exists in `$a`.

### **What is the difference between** `sort()`**,** `asort()`**, and** `ksort()`**?**

- `sort()` sorts values and reindexes numeric keys.
    
- `asort()` sorts values while preserving keys.
    
- `ksort()` sorts an array by keys.
    

```
$prices = [
    'apple' => 30,
    'banana' => 10,
    'orange' => 20,
];

asort($prices);
```

The keys remain associated with their original values.

### **What is** `usort()`**?**

`usort()` sorts an array using a custom comparison callback. It reindexes numeric keys.

```
$users = [
    ['name' => 'John', 'age' => 30],
    ['name' => 'Anna', 'age' => 20],
];

usort(
    $users,
    fn($a, $b) => $a['age'] <=> $b['age']
);
```

The users are sorted by age in ascending order.

### **What is the difference between** `foreach` **and** `for`**?**

- `for` is useful when working with numeric indexes or a known number of iterations.
    
- `foreach` is designed for iterating over arrays and iterables.
    

I generally prefer `foreach` when working with arrays because it is more readable and does not require manual index management.

### **What happens when you use** `&` **in foreach?**

It creates a reference to the original array element. Changes affect the original array.

```
$numbers = [1, 2, 3];

foreach ($numbers as &$number) {
    $number *= 2;
}

unset($number);
```

`unset()` is important because the reference remains attached to the last element after the loop.

### **Why can foreach by reference cause bugs?**

The loop variable remains a reference to the last array element after the loop finishes.

If the same variable is reused in another loop, it can unexpectedly modify the original array.

```
$numbers = [1, 2, 3];

foreach ($numbers as &$number) {
    $number *= 2;
}

unset($number);

foreach ($numbers as $number) {
    echo $number;
}
```

I use `unset()` immediately after a by-reference loop to prevent accidental modification.

### **What is the difference between foreach by value and foreach by reference?**

- By value, the loop variable receives the current value. Reassigning it does not modify the original array element.
    
- By reference, the loop variable refers to the original array element. Reassigning it changes the original array.
    

```
$numbers = [1, 2, 3];

foreach ($numbers as $number) {
    $number *= 2;
}

// Original array is unchanged.
```

```
foreach ($numbers as &$number) {
    $number *= 2;
}

unset($number);

// Original array is modified.
```

### **What is destructuring?**

Destructuring allows you to extract array values into separate variables.

```
[$first, $second] = [10, 20];

['name' => $name, 'age' => $age] = $user;
```

It is useful when working with structured arrays.

### **What is the spread operator?**

The spread operator `...` expands an array into individual values.

```
$first = [1, 2];
$second = [3, 4];

$result = [...$first, ...$second];
```

It can also be used to unpack arguments when calling a function.

### **What is the difference between** `array_merge()` **and the spread operator?**

Both can combine arrays, but they have different syntax and behavior depending on PHP version and key types.

```
$first = [1, 2];
$second = [3, 4];

$result = [...$first, ...$second];
```

For ordinary numeric-key arrays, the spread operator creates a combined array with sequential numeric keys. For string keys, PHP 8.1 and later support string-key unpacking, with later values overwriting earlier ones.

### **What is a Generator?**

A Generator allows you to iterate over values without storing the full result in memory.

```
function getNumbers(): Generator
{
    for ($i = 1; $i <= 1000; $i++) {
        yield $i;
    }
}
```

Generators are useful when processing large datasets because values are produced one at a time.

### **What is the difference between** `return` **and** `yield`**?**

- `return` ends a function and returns one final value.
    
- `yield` produces a value from a Generator and stop execution until the next value is requested.
    

A Generator can produce many values without creating a complete array in memory.

### **What is the difference between an array and a Generator?**

- An array stores all its elements in memory.
    
- A Generator produces values one at a time during iteration.
    

I use arrays when I need to access values repeatedly or randomly. I use Generators when I need to process a large sequence without loading everything into memory.

### **What is the difference between** `count()` **and** `sizeof()`**?**

Both return the number of elements in an array or countable object. `sizeof()` is an alias of `count()`.

I generally use `count()` because its purpose is clearer.

## **4. Functions and Closures**

### **What is a function?**

A function is a reusable block of code that performs a specific operation.

Functions help organize code, reduce duplication, and separate responsibilities.

### **What is the difference between passing by value and passing by reference?**

- By value: the function receives a value. Changes to the parameter do not directly modify the caller's variable.
    
- By reference: the function can modify the original variable.
    

```
function update(int &$number): void
{
    $number = 100;
}

$value = 10;

update($value);

// $value is now 100.
```

### **What is a return type?**

A return type declares the type of value that a function must return.

```
function getUserName(): string
{
    return 'John';
}
```

Return types make function contracts clearer and help static analysis tools detect errors.

### **What is a nullable return type?**

A nullable return type allows a function to return either a specific type or `null`.

```
function findUser(int $id): ?User
{
    return null;
}
```

It is useful when a function may not find a requested resource.

### **What is a Closure?**

A Closure is an anonymous function that can be assigned to a variable, passed as an argument, or returned from another function.

```
$multiply = function (int $number): int {
    return $number * 2;
};
```

Closures are commonly used as callbacks.

### **What is the difference between Closure and arrow function?**

- A Closure can capture variables using `use`.
    
- An arrow function automatically captures variables from the parent scope by value.
    
- Arrow functions contain one expression and return its result automatically.
    

```
$factor = 2;

$multiply = fn(int $number): int => $number * $factor;
```

A traditional Closure can capture a variable by reference:

```
$factor = 2;

$multiply = function (int $number) use (&$factor): int {
    return $number * $factor;
};
```

### **What is variable capture in a Closure?**

Variable capture allows a Closure to access variables from the outer scope using the `use` keyword.

```
$factor = 3;

$multiply = function (int $number) use ($factor): int {
    return $number * $factor;
};
```

By default, the variable is captured by value. Using `&` captures it by reference.

### **What is a callback?**

A callback is a function passed to another function as an argument.

```
function process(callable $callback): void
{
    $callback();
}

process(fn() => print('Hello'));
```

Callbacks allow code to be passed as behavior and make functions more flexible.

### **What is recursion?**

Recursion is when a function calls itself.

It must have a termination condition to prevent infinite recursion.

```
function factorial(int $number): int
{
    if ($number <= 1) {
        return 1;
    }

    return $number * factorial($number - 1);
}
```

### **What is a variadic function?**

A variadic function accepts an unlimited number of arguments using `...`.

```
function sum(int ...$numbers): int
{
    return array_sum($numbers);
}

echo sum(1, 2, 3, 4);
```

The variadic parameter is collected into an array.

### **What is a higher-order function?**

A higher-order function accepts another function as an argument or returns a function.

Examples include `array_map()`, `array_filter()`, and `array_reduce()`.

### **What is the difference between a parameter and an argument?**

- A parameter is a variable declared in a function definition.
    
- An argument is the actual value passed when the function is called.
    

```
function greet(string $name): void
{
    echo "Hello, $name";
}

greet('John');
```

Here, `$name` is a parameter and `'John'` is an argument.

### **What are default parameter values?**

Default parameter values allow a function to use a predefined value when an argument is not provided.

```
function greet(string $name = 'Guest'): string
{
    return "Hello, $name";
}
```

### **What are named arguments?**

Named arguments allow callers to pass values by parameter name rather than by position.

```
function createUser(
    string $name,
    int $age,
    bool $active = true
): void {
    // ...
}

createUser(
    name: 'John',
    age: 30,
    active: false,
);
```

They improve readability when a function has several parameters.

### **What is the difference between** `callable` **and** `Closure`**?**

- `callable` is a type that accepts any valid PHP callback.
    
- `Closure` is a specific class representing an anonymous function.
    

A function name, static method, or object method can be callable without being a Closure object.

### **What is a pure function?**

A pure function always returns the same output for the same input and does not produce side effects.

```
function multiply(int $a, int $b): int
{
    return $a * $b;
}
```

Pure functions are easier to test because their behavior does not depend on external state.

### **What is a side effect?**

A side effect is an operation that changes state outside the function or interacts with external systems.

Examples include:

- Updating a database.
    
- Writing to a file.
    
- Sending an HTTP request.
    
- Modifying a global variable.
    

Separating pure calculations from side effects makes code easier to test and maintain.

## **5. Object-Oriented Programming (OOP)**

### **What is OOP?**

Object-Oriented Programming is a programming paradigm based on objects that combine data and behavior.

The four main principles are:

1. Encapsulation.
    
2. Inheritance.
    
3. Polymorphism.
    
4. Abstraction.
    

### **What is a class?**

A class is a blueprint that defines the properties and methods of objects.

```
class User
{
    public string $name;

    public function greet(): string
    {
        return "Hello, {$this->name}";
    }
}
```

### **What is an object?**

An object is an instance of a class. It contains its own state and can use the behavior defined by the class.

```
$user = new User();

$user->name = 'John';

echo $user->greet();
```

### **What is encapsulation?**

Encapsulation means hiding internal implementation details and controlling access to an object's data.

PHP provides:

- `public` — accessible from anywhere.
    
- `protected` — accessible within the class and its child classes.
    
- `private` — accessible only within the class that declares it.
    

### **What is inheritance?**

Inheritance allows a class to reuse and extend functionality from another class using `extends`.

```
class Animal
{
    public function eat(): void {}
}

class Dog extends Animal
{
    public function bark(): void {}
}
```

The child class inherits accessible methods and properties from the parent class.

### **What is polymorphism?**

Polymorphism allows different classes to implement the same interface or inherit the same contract while providing different behavior.

```
interface Payment
{
    public function pay(float $amount): void;
}

class CardPayment implements Payment
{
    public function pay(float $amount): void {}
}

class PayPalPayment implements Payment
{
    public function pay(float $amount): void {}
}
```

Both classes can be used wherever the `Payment` interface is expected.

### **What is abstraction?**

Abstraction hides implementation details and exposes only essential functionality.

It can be implemented using interfaces and abstract classes.

### **What is the difference between an interface and an abstract class?**

**Interface:**

- Defines a contract.
    
- A class can implement multiple interfaces.
    
- Does not provide instance properties or constructor implementations as an abstract class does.
    
- Can contain constants and method declarations.
    

**Abstract class:**

- Can contain implemented and abstract methods.
    
- Can have properties and a constructor.
    
- A class can extend only one class.
    
- Can provide shared implementation for child classes.
    

I use an interface when I need to define a contract for different implementations. I use an abstract class when related classes need to share common state or behavior.

### **What is the difference between** `self`**,** `static`**, and** `parent`**?**

- `self` refers to the class where the method is defined.
    
- `static` uses late static binding and refers to the called class.
    
- `parent` refers to the parent class.
    

### **What is late static binding?**

Late static binding allows PHP to resolve `static` based on the class that was called at runtime rather than the class where the method was originally defined.

```
class ParentClass
{
    public static function getClass(): string
    {
        return static::class;
    }
}

class ChildClass extends ParentClass {}

echo ChildClass::getClass(); // ChildClass
```

### **What is method overriding?**

Method overriding happens when a child class provides its own implementation of a method inherited from a parent class.

The child method must remain compatible with the parent method's signature and visibility requirements.

### **What is method overloading in PHP?**

PHP does not support traditional method overloading by different method signatures like Java or C++.

It provides magic methods such as `__get()`, `__set()`, and `__call()` for dynamic behavior.

### **What is a constructor?**

A constructor is a special method `__construct()` called when an object is created. It initializes the object.

```
class User
{
    public function __construct(
        private string $name,
    ) {}
}
```

### **What is a destructor?**

`__destruct()` is called when an object is destroyed or during shutdown. It can be used for cleanup, but should not be relied on for critical business operations.

### **What are static methods and properties?**

Static members belong to the class rather than a particular object.

They can be accessed without creating an instance.

```
class MathHelper
{
    public static function sum(int $a, int $b): int
    {
        return $a + $b;
    }
}

echo MathHelper::sum(2, 3);
```

Static methods should be used when behavior does not depend on an object's instance state.

### **What is a Trait?**

A Trait allows code reuse across multiple classes without inheritance.

```
trait LoggerTrait
{
    public function log(string $message): void {}
}

class UserService
{
    use LoggerTrait;
}
```

Traits help reuse methods, but they should not replace proper object composition or interfaces.

### **What is an Enum?**

An Enum defines a fixed set of possible values.

```
enum Status: string
{
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
}
```

Enums improve type safety and make domain values explicit.

### **What is a readonly property?**

A readonly property can be initialized once and cannot be reassigned afterward.

```
class User
{
    public function __construct(
        public readonly int $id,
    ) {}
}
```

PHP 8.2 introduced readonly classes. Readonly properties prevent reassignment, but they do not automatically make every nested object immutable.

### **What is a final class or method?**

- `final class` cannot be extended.
    
- `final method` cannot be overridden by child classes.
    

### **What is dependency injection?**

Dependency Injection means providing an object with its dependencies from outside instead of creating them internally.

```
class UserService
{
    public function __construct(
        private UserRepository $repository,
    ) {}
}
```

It improves testability, flexibility, and separation of concerns.

### **What is the difference between dependency injection and creating dependencies directly?**

With direct instantiation, a class creates its own dependencies and becomes tightly coupled to their concrete implementations.

With dependency injection, dependencies are provided from outside, making it easier to replace implementations and test the class.

```
class UserService
{
    public function __construct(
        private UserRepository $repository,
    ) {}
}
```

### **What is composition?**

Composition means building a class by combining other objects instead of inheriting their behavior.

```
class OrderService
{
    public function __construct(
        private PaymentService $paymentService,
    ) {}
}
```

Composition usually creates more flexible and loosely coupled code than deep inheritance hierarchies.

### **What is the difference between composition and inheritance?**

- Inheritance represents an IS-A relationship and allows a class to extend another class.
    
- Composition represents a HAS-A relationship and allows a class to use other objects.
    

I generally prefer composition because it reduces coupling and makes behavior easier to change.

## **6. Advanced OOP and Type System**

### **What is an anonymous class?**

An anonymous class is a class without a declared name. It is useful for small, one-time implementations.

```
$logger = new class {
    public function log(string $message): void
    {
        echo $message;
    }
};

$logger->log('Hello');
```

### **What are magic methods?**

Magic methods are special methods called automatically by PHP in specific situations.

Examples:

- `__construct()` — object creation.
    
- `__destruct()` — object destruction.
    
- `__get()` — inaccessible property read.
    
- `__set()` — inaccessible property write.
    
- `__call()` — inaccessible method call.
    
- `__toString()` — object converted to string.
    
- `__invoke()` — object called as a function.
    
- `__clone()` — object cloning.
    
- `__isset()` — called by `isset()` or `empty()` on inaccessible properties.
    
- `__unset()` — called by `unset()` on inaccessible properties.
    
- `__serialize()` and `__unserialize()` — customize serialization.
    

### **What is object cloning?**

Cloning creates a shallow copy of an object.

`__clone()` can customize the cloning process.

```
$copy = clone $original;
```

### **What is the difference between shallow copy and deep copy?**

- Shallow copy copies the object, but nested object references may still point to the same objects.
    
- Deep copy duplicates the nested object structure as well.
    

In PHP, the `clone` keyword performs a shallow copy by default.

### **What is covariance and contravariance?**

- Covariance allows a child implementation to return a more specific type.
    
- Contravariance allows a child implementation to accept a less specific parameter type.
    

These concepts make method overriding more flexible while preserving type compatibility.

### **What are union and intersection types?**

- Union types allow one of several types: `int|string`.
    
- Intersection types require a value to satisfy multiple types: `Countable&Iterator`.
    

PHP also supports DNF types, combining unions and intersections.

```
function process(int|string $value): void
{
    // ...
}
```

### **What is a nullable type?**

A nullable type allows a value or `null`.

```
function findUser(): ?User
{
    return null;
}
```

The `?User` syntax is equivalent to `User|null`.

### **What is a backed Enum?**

A backed Enum has scalar values such as strings or integers.

```
enum Role: string
{
    case ADMIN = 'admin';
    case USER = 'user';
}
```

A backed Enum can be converted to its scalar value using the `value` property.

```
echo Role::ADMIN->value; // admin
```

### **What is an Iterator?**

An Iterator allows an object to be traversed using `foreach`. It implements methods such as `current()`, `key()`, `next()`, `rewind()`, and `valid()`.

### **What is the difference between Iterator and Generator?**

An Iterator is an object implementing an iteration interface. A Generator is a simpler way to create an iterator using `yield`.

### **What is the difference between** `iterable` **and** `Iterator`**?**

- `iterable` is a built-in type that accepts arrays and Traversable objects.
    
- `Iterator` is an interface that defines methods for iterating over an object.
    

```
function processItems(iterable $items): void
{
    foreach ($items as $item) {
        // ...
    }
}
```

I use `iterable` when a function should accept both arrays and iterable objects.

### **What is the** `object` **type?**

The `object` type accepts any object instance.

```
function process(object $value): void
{
    // ...
}
```

When possible, I prefer a specific class or interface type because it provides a more precise contract.

### **What is the** `mixed` **type?**

The `mixed` type allows any PHP value.

```
function process(mixed $value): mixed
{
    return $value;
}
```

I use `mixed` only when a value can genuinely have different types. More specific types are preferable because they improve static analysis and readability.

### **What is the** `never` **return type?**

The `never` return type indicates that a function never returns normally. It must either throw an exception, terminate execution, or enter an infinite loop.

```
function abortRequest(): never
{
    throw new RuntimeException('Request failed');
}
```

### **What is the** `void` **return type?**

The `void` return type indicates that a function does not return a value.

```
function logMessage(string $message): void
{
    echo $message;
}
```

A `void` function can use `return;` without a value.

### **What is the difference between** `void` **and** `never`**?**

- `void` means a function completes without returning a value.
    
- `never` means a function does not complete normally.
    

For example, a logging function can return `void`, while a function that always throws an exception can return `never`.

### **What is the difference between** `clone` **and assignment?**

Assigning an object variable copies the object handle, so both variables refer to the same object.

Using `clone` creates a separate object instance.

```
$first = new stdClass();
$first->name = 'John';

$second = $first;
$third = clone $first;

$second->name = 'Anna';

echo $first->name; // Anna
echo $third->name; // John
```

## **7. Error Handling and Exceptions**

### **What is the difference between an Exception and an Error?**

Both implement `Throwable`.

- `Exception` represents conditions that application code may handle.
    
- `Error` represents engine-level or programming errors, such as type errors.
    

### **What is Throwable?**

`Throwable` is the base interface for both `Exception` and `Error`.

### **What is the difference between** `try`**,** `catch`**, and** `finally`**?**

- `try` contains code that may throw an exception.
    
- `catch` handles an exception.
    
- `finally` executes after the try/catch process, including when an exception is thrown.
    

```
try {
    processPayment();
} catch (PaymentException $exception) {
    logError($exception);
} finally {
    cleanup();
}
```

### **What is a custom exception?**

A custom exception is an application-specific exception class that extends `Exception` or another suitable exception class.

```
class UserNotFoundException extends RuntimeException {}
```

Custom exceptions make error handling more meaningful and allow callers to handle specific failure types.

### **When should you throw an exception?**

I throw an exception when an operation cannot continue normally and the caller needs to handle the failure.

For example, a service can throw `UserNotFoundException` when a requested user does not exist.

### **Should exceptions be used for normal application flow?**

No. Exceptions should represent exceptional situations, not ordinary business decisions.

For example, returning `false` or a result object can be more appropriate when a failed lookup is an expected outcome.

### **What is exception propagation?**

Exception propagation means an exception moves up the call stack until a matching `catch` block handles it.

If no matching handler is found, PHP terminates the request with an uncaught exception.

### **Can you catch both Exceptions and Errors?**

Yes. Both implement `Throwable`, so they can be caught using `catch (Throwable $exception)`.

```
try {
    performOperation();
} catch (Throwable $exception) {
    // Handle or log the failure.
}
```

I generally catch specific exception types when possible. Catching `Throwable` is useful at application boundaries, but it should not hide programming errors.

### **What is the difference between** `throw` **and** `return`**?**

- `return` ends a function and sends a value back to the caller.
    
- `throw` interrupts normal execution and transfers control to an exception handler.
    

### **What is exception chaining?**

Exception chaining means throwing a new exception while keeping the original exception as its previous cause.

```
try {
    $repository->save($user);
} catch (PDOException $exception) {
    throw new RuntimeException(
        'Could not save user',
        previous: $exception,
    );
}
```

It preserves the original error context while providing a higher-level application exception.

### **What is the difference between** `Exception` **and** `RuntimeException`**?**

`Exception` is a general base class for application exceptions. `RuntimeException` represents errors that occur during program execution and cannot necessarily be detected before runtime.

I use specific exception classes to communicate what went wrong.

## **8. Memory Management and Performance**

### **How does PHP manage memory?**

PHP automatically manages memory using reference counting and garbage collection.

Developers usually do not need to allocate or free memory manually.

### **What is reference counting?**

PHP tracks how many references point to a value. When the reference count reaches zero, the value can be released.

### **What is garbage collection?**

Garbage collection identifies and removes circular references that cannot be released by normal reference counting.

### **What is a circular reference?**

A circular reference occurs when two or more objects reference each other, creating a cycle.

```
$a = new stdClass();
$b = new stdClass();

$a->other = $b;
$b->other = $a;
```

Even if external references are removed, the objects can still reference each other. PHP's garbage collector can detect and collect such cycles.

### **What is OPcache?**

OPcache stores compiled PHP bytecode in shared memory. It reduces repeated compilation and improves performance.

### **What is JIT?**

JIT (Just-In-Time compilation) is a feature that compiles certain bytecode paths into machine code at runtime.

It can help some CPU-intensive workloads, but it does not automatically make typical web applications faster.

### **How can you optimize PHP performance?**

- Use OPcache.
    
- Avoid unnecessary database queries.
    
- Add database indexes.
    
- Cache frequently used data.
    
- Avoid loading large datasets into memory.
    
- Use generators for streaming large data.
    
- Profile before optimizing.
    
- Reduce unnecessary object creation.
    
- Avoid repeated expensive calculations.
    
- Use efficient algorithms and data structures.
    

### **What is the difference between time complexity and space complexity?**

- Time complexity describes how execution time grows with input size.
    
- Space complexity describes how memory usage grows with input size.
    

### **What is Big O notation?**

Big O describes the growth rate of an algorithm.

Examples:

- O(1) — constant.
    
- O(log n) — logarithmic.
    
- O(n) — linear.
    
- O(n log n) — linearithmic.
    
- O(n²) — quadratic.
    

### **What is the difference between O(1) and O(n)?**

- O(1) means the operation takes approximately the same amount of time regardless of input size.
    
- O(n) means execution time grows linearly with the number of elements.
    

Accessing an array element by its key is generally O(1), while iterating over all elements is O(n).

### **Why should you avoid nested loops?**

Nested loops can lead to quadratic time complexity, often O(n²), when both loops iterate over the same input size.

For large datasets, I look for ways to use associative arrays, lookup maps, or more efficient algorithms.

### **What is caching?**

Caching stores frequently accessed data so it does not need to be recalculated or retrieved from the original source every time.

It can reduce database load and improve response times.

### **What is the difference between in-memory caching and persistent caching?**

- In-memory caching stores data in memory and can be very fast.
    
- Persistent caching stores data in a system that can survive application requests or process restarts.
    

Examples include Redis and Memcached.

### **What is lazy loading?**

Lazy loading means loading data or creating an object only when it is actually needed.

It can reduce unnecessary work, especially when dealing with relationships or large datasets.

### **What is eager loading?**

Eager loading retrieves related data in advance to avoid repeated queries.

It is commonly used to prevent the N+1 query problem.

## **9. Composer and Autoloading**

### **What is Composer?**

Composer is PHP's dependency manager. It installs packages, manages versions, and provides autoloading.

### **What is** `composer.json`**?**

It defines project metadata, dependencies, autoloading rules, and scripts.

### **What is** `composer.lock`**?**

It stores exact installed dependency versions to ensure consistent installations across environments.

### **What is the difference between** `composer install` **and** `composer update`**?**

- `composer install` installs versions from `composer.lock`.
    
- `composer update` resolves newer compatible versions and updates the lock file.
    

In production and CI environments, I generally use `composer install` to ensure reproducible dependency versions.

### **What is PSR-4?**

PSR-4 is an autoloading standard that maps namespaces to directory structures.

```
{
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```

For example, `App\Service\UserService` can be mapped to `src/Service/UserService.php`.

### **What is autoloading?**

Autoloading automatically loads class files when classes are needed, without manually including each file.

Composer generates an autoloader that can be included with:

```
require __DIR__ . '/vendor/autoload.php';
```

### **What is semantic versioning?**

Semantic Versioning uses three numbers:

`MAJOR.MINOR.PATCH`

- MAJOR — breaking changes.
    
- MINOR — backward-compatible features.
    
- PATCH — backward-compatible bug fixes.
    

### **What is the difference between** `require` **and** `require-dev` **in Composer?**

- `require` contains dependencies needed in production.
    
- `require-dev` contains development dependencies such as testing and code-quality tools.
    

### **What is the** `vendor` **directory?**

The `vendor` directory contains installed Composer dependencies and generated autoloading files.

It is normally not committed to Git because it can be recreated using Composer.

### **What is PSR?**

PSR stands for PHP Standards Recommendation. It is a set of standards defined by the PHP-FIG community to improve interoperability between PHP projects.

Examples include:

- PSR-4 — autoloading.
    
- PSR-3 — logger interface.
    
- PSR-7 — HTTP message interfaces.
    
- PSR-11 — container interface.
    
- PSR-12 — coding style.
    

### **What is the difference between** `composer dump-autoload` **and** `composer update`**?**

- `composer dump-autoload` regenerates Composer's autoload files.
    
- `composer update` resolves and updates dependencies and modifies `composer.lock`.
    

I use `dump-autoload` when I change autoloading configuration or need to regenerate the autoloader without updating packages.

### **What are Composer scripts?**

Composer scripts allow commands to be defined and executed as part of Composer workflows.

```
{
    "scripts": {
        "test": "phpunit",
        "analyse": "phpstan analyse"
    }
}
```

They help standardize common development tasks.

## **10. Database and SQL**

### **What is PDO?**

PDO (PHP Data Objects) provides a consistent interface for accessing different database systems.

It supports prepared statements and multiple database drivers.

### **What are prepared statements?**

Prepared statements separate SQL code from user input. They help prevent SQL injection.

```
$stmt = $pdo->prepare(
    'SELECT * FROM users WHERE id = :id'
);

$stmt->execute(['id' => $id]);

$user = $stmt->fetch();
```

### **What is SQL injection?**

SQL injection is an attack where malicious SQL is inserted into a query through untrusted input.

Prevention:

- Prepared statements.
    
- Parameter binding.
    
- Input validation.
    
- Least-privilege database accounts.
    

### **What is the difference between validation and sanitization?**

- Validation checks whether input meets expected rules.
    
- Sanitization modifies input to remove or transform unwanted content.
    

Validation should be used to enforce business requirements. Output escaping should be used to prevent injection into the relevant output context.

### **What is a database transaction?**

A transaction groups multiple database operations into one logical unit.

If an operation fails, the transaction can be rolled back so that partial changes are not committed.

```
$pdo->beginTransaction();

try {
    // Execute related database operations.

    $pdo->commit();
} catch (Throwable $exception) {
    $pdo->rollBack();

    throw $exception;
}
```

### **What are ACID properties?**

- Atomicity — all operations succeed or none do.
    
- Consistency — data remains valid.
    
- Isolation — concurrent transactions do not interfere incorrectly.
    
- Durability — committed changes persist.
    

### **What is an index?**

An index improves data retrieval speed but requires additional storage and can slow down writes.

Indexes are especially useful for columns frequently used in `WHERE`, `JOIN`, and `ORDER BY` conditions.

### **What is a composite index?**

A composite index is an index built from multiple columns.

```
CREATE INDEX idx_users_status_created
ON users (status, created_at);
```

The order of columns matters because database engines can use the leftmost prefix of a composite index.

### **What is the N+1 query problem?**

It occurs when one query retrieves a collection and then an additional query is executed for each item.

For example, retrieving 100 orders and then executing a separate query for each order's customer can result in 101 queries.

Solution: use JOINs, eager loading, or batch queries.

### **What is normalization?**

Normalization organizes database tables to reduce data duplication and improve consistency.

### **What is denormalization?**

Denormalization intentionally introduces duplicated or precomputed data to improve read performance.

It can reduce expensive joins, but it increases the complexity of keeping data consistent.

### **What is the difference between INNER JOIN and LEFT JOIN?**

- INNER JOIN returns matching records from both tables.
    
- LEFT JOIN returns all records from the left table and matching records from the right table. If there is no match, the right-side columns contain `NULL`.
    

### **What is the difference between WHERE and HAVING?**

- `WHERE` filters rows before grouping.
    
- `HAVING` filters groups after aggregation.
    

```
SELECT status, COUNT(*) AS total
FROM orders
WHERE created_at >= '2026-01-01'
GROUP BY status
HAVING COUNT(*) > 10;
```

### **What is GROUP BY?**

`GROUP BY` groups rows that have the same values in specified columns. It is commonly used with aggregate functions such as `COUNT()`, `SUM()`, `AVG()`, `MIN()`, and `MAX()`.

### **What is the difference between DELETE, TRUNCATE, and DROP?**

- `DELETE` removes selected rows and can use a `WHERE` condition.
    
- `TRUNCATE` removes all rows from a table.
    
- `DROP` removes the table structure itself.
    

Their transaction and identity-reset behavior can differ between database systems.

### **What is a primary key?**

A primary key uniquely identifies each row in a table. It cannot contain `NULL` values and must be unique.

### **What is a foreign key?**

A foreign key is a column or set of columns that references a key in another table. It helps maintain referential integrity between related records.

### **What is optimistic locking?**

Optimistic locking assumes conflicts are uncommon and checks whether a record changed before updating it, often using a version column.

### **What is pessimistic locking?**

Pessimistic locking locks data while a transaction is working with it, preventing conflicting operations from modifying the same data concurrently.

### **What is a database deadlock?**

A deadlock happens when transactions wait for resources locked by each other.

Solutions include consistent lock ordering, short transactions, proper indexes, and retry logic.

### **What is the difference between offset and cursor pagination?**

- Offset pagination uses `LIMIT` and `OFFSET` to skip a number of rows.
    
- Cursor pagination uses a reference to the last retrieved record to fetch the next page.
    

Cursor pagination is generally more stable and efficient for large or frequently changing datasets.

## **11. HTTP, REST and APIs**

### **What is HTTP?**

HTTP is a protocol used for communication between clients and servers.

It follows a request-response model.

### **What are common HTTP methods?**

- GET — retrieve data.
    
- POST — create or process data.
    
- PUT — replace a resource.
    
- PATCH — partially update a resource.
    
- DELETE — delete a resource.
    

### **What is the difference between PUT and PATCH?**

- PUT replaces the resource representation.
    
- PATCH applies partial modifications.
    

### **What are common HTTP status codes?**

- 200 — OK.
    
- 201 — Created.
    
- 204 — No Content.
    
- 301 — Moved Permanently.
    
- 400 — Bad Request.
    
- 401 — Unauthorized.
    
- 403 — Forbidden.
    
- 404 — Not Found.
    
- 409 — Conflict.
    
- 422 — Unprocessable Content.
    
- 500 — Internal Server Error.
    
- 503 — Service Unavailable.
    

### **What is REST?**

REST is an architectural style for designing APIs around resources and standard HTTP operations.

A REST API commonly uses resource URLs, HTTP methods, status codes, and representations such as JSON.

### **What is JSON?**

JSON is a lightweight data-interchange format commonly used in APIs.

PHP provides `json_encode()` and `json_decode()`.

```
$json = json_encode($user);

$user = json_decode($json, true);
```

The second argument of `json_decode()` makes JSON objects become associative arrays.

### **What is the difference between authentication and authorization?**

- Authentication verifies who the user is.
    
- Authorization determines what the user is allowed to do.
    

### **What is JWT?**

JWT (JSON Web Token) is a token format used to transmit claims between parties.

A JWT contains a header, payload, and signature.

Important: JWT payloads are normally encoded, not encrypted. Sensitive information should not be stored in a JWT payload unless encryption is explicitly used.

### **What is idempotency?**

An operation is idempotent if repeating it produces the same intended result as executing it once.

GET, PUT, and DELETE are defined as idempotent HTTP methods. POST is not idempotent by default.

### **What is pagination?**

Pagination splits large result sets into smaller responses.

Common approaches:

- Offset pagination.
    
- Cursor pagination.
    

Cursor pagination is generally more stable for large or frequently changing datasets.

### **What is the difference between synchronous and asynchronous processing?**

- Synchronous processing waits for an operation to finish before continuing.
    
- Asynchronous processing allows work to continue independently, often using queues or background workers.
    

For example, an API can accept an order immediately and process email notifications asynchronously.

### **What is CORS?**

CORS (Cross-Origin Resource Sharing) is a browser security mechanism that controls whether a web page can access resources from a different origin.

The server uses HTTP headers to specify which origins, methods, and headers are allowed.

### **What is the difference between cookies and sessions?**

- A cookie is data stored in the client's browser and sent with matching HTTP requests.
    
- A session stores server-side data associated with a session identifier, which is commonly stored in a cookie.
    

Sessions are often used to maintain authentication state.

### **What is an API versioning strategy?**

API versioning allows an API to evolve while maintaining compatibility with existing clients.

Common approaches include:

- URL versioning, such as `/api/v1/users`.
    
- Header-based versioning.
    
- Query parameter versioning.
    

I prefer explicit versioning when changes can break existing clients.

## **12. Security**

### **How do you prevent SQL injection?**

Use prepared statements and parameterized queries. Never concatenate untrusted input directly into SQL statements.

### **How do you prevent XSS?**

Escape output according to its context and avoid inserting untrusted content into executable HTML or JavaScript.

For HTML output, use appropriate escaping such as `htmlspecialchars()`.

### **What is CSRF?**

Cross-Site Request Forgery tricks an authenticated user into performing an unwanted action.

Prevention:

- CSRF tokens.
    
- SameSite cookies.
    
- Appropriate origin checks.
    

### **How should passwords be stored?**

Use `password_hash()` and verify with `password_verify()`.

Never store plain-text passwords or use MD5/SHA1 for password storage.

```
$hash = password_hash($password, PASSWORD_DEFAULT);

if (password_verify($password, $hash)) {
    // Password is valid.
}
```

### **What is the difference between hashing and encryption?**

- Hashing is one-way.
    
- Encryption is reversible using a key.
    

Password storage should use a dedicated password hashing algorithm rather than reversible encryption.

### **What is session fixation?**

Session fixation is an attack where an attacker makes a victim use a known session ID.

Prevention: regenerate the session ID after authentication.

### **What is the principle of least privilege?**

Users and services should receive only the permissions necessary to perform their tasks.

### **How should secrets be managed?**

Use environment variables or a dedicated secrets manager. Never commit passwords, API keys, or private credentials to Git.

### **What is password salting?**

A salt is a unique random value used by password hashing algorithms to protect against precomputed attacks and identical password hashes.

Modern password hashing functions such as `password_hash()` handle salt generation automatically.

### **What is brute-force protection?**

Brute-force protection limits repeated authentication attempts to make password guessing more difficult.

Common measures include:

- Rate limiting.
    
- Temporary account lockouts.
    
- Increasing delays between failed attempts.
    
- Multi-factor authentication.
    
- Monitoring suspicious activity.
    

### **What is the difference between authentication and session management?**

Authentication verifies a user's identity. Session management maintains that authenticated state across multiple HTTP requests.

A secure application should use secure cookies, regenerate session identifiers after login, and expire sessions appropriately.

### **What is output escaping?**

Output escaping converts special characters into a safe representation for a specific output context.

For example, HTML escaping prevents user-provided text from being interpreted as HTML markup.

### **What is the difference between validation and output escaping?**

- Validation checks whether input is acceptable according to business rules.
    
- Output escaping prevents data from being interpreted as executable content in a specific context.
    

Validation does not replace output escaping.

### **What is the principle of defense in depth?**

Defense in depth means using multiple independent security controls rather than relying on one mechanism.

For example, an application can combine authentication, authorization, input validation, prepared statements, output escaping, and rate limiting.

# **PHP Backend Interview Notes — Part 3**

## **13. SOLID Principles**

### **What is SOLID?**

SOLID is a set of five principles for writing maintainable and flexible object-oriented code.

### **S — Single Responsibility Principle**

A class should have one responsibility and one main reason to change.

For example, a `UserRepository` should handle data persistence, while a `UserService` should contain application logic.

### **O — Open/Closed Principle**

Software entities should be open for extension but closed for modification.

Example: introduce a new payment implementation instead of modifying an existing payment service.

```
interface PaymentMethod
{
    public function pay(float $amount): void;
}

class CardPayment implements PaymentMethod
{
    public function pay(float $amount): void {}
}

class PayPalPayment implements PaymentMethod
{
    public function pay(float $amount): void {}
}
```

A new payment method can be added without changing the existing implementations.

### **L — Liskov Substitution Principle**

A child class should be usable wherever its parent type is expected without breaking behavior.

A child class must respect the contract established by its parent class or interface.

### **I — Interface Segregation Principle**

Prefer small, specific interfaces instead of large interfaces that force classes to implement unnecessary methods.

For example, a class that only reads data should not be forced to implement methods for updating and deleting data.

### **D — Dependency Inversion Principle**

High-level modules should depend on abstractions rather than concrete implementations.

```
interface UserRepositoryInterface
{
    public function find(int $id): ?User;
}

class UserService
{
    public function __construct(
        private UserRepositoryInterface $repository,
    ) {}
}
```

The service depends on an interface rather than a specific database implementation.

### **What is the difference between Dependency Injection and Dependency Inversion?**

- Dependency Injection is a technique for providing dependencies.
    
- Dependency Inversion is a design principle about depending on abstractions.
    

Dependency Injection can be used to implement Dependency Inversion.

### **Why is SOLID important?**

SOLID helps reduce coupling, improve testability, and make code easier to extend and maintain.

I use these principles as guidelines rather than strict rules because applying them unnecessarily can make simple code more complicated.

## **14. Design Patterns**

### **What is a design pattern?**

A design pattern is a reusable solution to a common software design problem.

Patterns provide established ways to structure code and solve recurring architectural problems.

### **What is Singleton?**

Ensures that a class has only one instance and provides a global access point.

Use carefully because it introduces global state and makes testing harder.

### **What is Factory?**

Creates objects without exposing the exact creation logic to the caller.

```
interface PaymentMethod
{
    public function pay(float $amount): void;
}

class PaymentFactory
{
    public function create(string $type): PaymentMethod
    {
        return match ($type) {
            'card' => new CardPayment(),
            'paypal' => new PayPalPayment(),
            default => throw new InvalidArgumentException(),
        };
    }
}
```

### **What is Strategy?**

Defines interchangeable algorithms behind a common interface.

Example: different payment methods.

```
interface PaymentStrategy
{
    public function pay(float $amount): void;
}

class CardPayment implements PaymentStrategy
{
    public function pay(float $amount): void {}
}

class PayPalPayment implements PaymentStrategy
{
    public function pay(float $amount): void {}
}

class CheckoutService
{
    public function __construct(
        private PaymentStrategy $payment,
    ) {}

    public function checkout(float $amount): void
    {
        $this->payment->pay($amount);
    }
}
```

The payment implementation can be changed without modifying the checkout service.

### **What is Repository?**

Encapsulates data access logic and separates persistence from business logic.

A repository provides methods such as `find()`, `save()`, and `delete()`.

### **What is Adapter?**

Allows incompatible interfaces to work together.

For example, an adapter can make an external payment provider compatible with an application's internal payment interface.

### **What is Observer?**

Allows multiple objects to react when an event occurs.

Example: after an order is created, separate listeners can send an email, update statistics, and publish a notification.

### **What is Decorator?**

Adds behavior to an object without modifying its original class.

For example, a decorator can add logging or caching around an existing service.

### **What is Dependency Injection Container?**

A container manages object creation and resolves dependencies automatically.

Instead of manually creating every dependency, the application asks the container for an object, and the container builds the required dependency graph.

### **What is the difference between Factory and Repository?**

- Factory creates objects.
    
- Repository retrieves and persists data.
    

### **What is the difference between Strategy and Factory?**

- Strategy defines interchangeable behavior or algorithms.
    
- Factory is responsible for creating objects.
    

A factory can create a strategy implementation, but they solve different problems.

### **What is the difference between Decorator and Adapter?**

- Decorator adds behavior while preserving a compatible interface.
    
- Adapter changes an interface so that incompatible components can work together.
    

## **15. Testing**

### **What is unit testing?**

Unit testing checks a small, isolated piece of application logic.

A unit test should be fast and should avoid unnecessary external dependencies.

### **What is integration testing?**

Integration testing checks whether multiple components work correctly together.

For example, it can test whether a service communicates correctly with a real database.

### **What is functional testing?**

Functional testing verifies application behavior from the perspective of a feature or user workflow.

It can test a complete request-response process.

### **What is PHPUnit?**

PHPUnit is a testing framework for PHP applications.

It provides assertions, test cases, data providers, mocks, and test lifecycle methods.

### **What is a mock?**

A mock object simulates a dependency and allows you to control its behavior and verify interactions.

### **What is a stub?**

A stub provides predefined responses to method calls.

### **What is the difference between a mock and a stub?**

A stub provides data. A mock is also used to verify expected interactions.

### **What is test coverage?**

Test coverage measures which parts of the code are executed during tests. High coverage does not automatically mean high test quality.

### **What is TDD?**

Test-Driven Development follows:

1. Write a failing test.
    
2. Implement the minimum code to pass.
    
3. Refactor the implementation.
    

This is commonly called the Red-Green-Refactor cycle.

### **What is static analysis?**

Static analysis examines code without executing it.

Examples: PHPStan, Psalm.

It can detect type errors, invalid method calls, and other potential problems.

### **What is code style checking?**

It checks whether code follows agreed coding standards.

Example: PHP_CodeSniffer (PHPCS).

### **What is the difference between unit and integration tests?**

- Unit tests verify isolated logic.
    
- Integration tests verify interactions between multiple components.
    

Unit tests are generally faster, while integration tests provide confidence that components work together correctly.

### **What is a data provider in PHPUnit?**

A data provider supplies multiple sets of input values to the same test method.

```
#[DataProvider('additionProvider')]
public function testAddition(
    int $a,
    int $b,
    int $expected,
): void {
    self::assertSame($expected, $a + $b);
}

public static function additionProvider(): array
{
    return [
        [1, 2, 3],
        [2, 3, 5],
        [5, 5, 10],
    ];
}
```

It helps test different scenarios without duplicating test methods.

### **What is a regression test?**

A regression test verifies that previously working functionality still works after code changes.

I add regression tests when fixing bugs to prevent the same issue from appearing again.

### **What is mocking a dependency?**

Mocking a dependency means replacing a real dependency with a controlled test object.

It allows me to test a service without requiring a real database, API, or external system.

### **What should you test?**

I focus on:

- Expected behavior.
    
- Edge cases.
    
- Invalid input.
    
- Error handling.
    
- Authorization and access rules.
    
- Important integration points.
    
- Regression scenarios.
    

## **16. Architecture and Senior-Level Concepts**

### **What is separation of concerns?**

It means keeping different responsibilities in separate components.

For example, controllers handle HTTP requests, services contain business logic, and repositories handle persistence.

### **What is a Service Layer?**

A service layer contains application or business operations and coordinates different components.

It keeps business logic out of controllers and provides reusable application functionality.

### **What is a DTO?**

A Data Transfer Object is an object used to transfer structured data between application layers without exposing internal domain entities.

A DTO usually contains data and does not implement complex business behavior.

### **What is an Entity?**

An Entity is an object defined by its identity, which remains important even if its attributes change.

For example, two users with the same name are still different entities if they have different IDs.

### **What is a Value Object?**

A Value Object is defined by its values rather than identity. It is usually immutable.

Example: Money, EmailAddress, DateRange.

Two Value Objects with the same values are generally considered equal.

### **What is Domain-Driven Design?**

DDD is an approach to designing software around business domains and their concepts.

Important concepts:

- Entity.
    
- Value Object.
    
- Aggregate.
    
- Repository.
    
- Domain Service.
    
- Bounded Context.
    
- Ubiquitous Language.
    

### **What is an Aggregate?**

An Aggregate is a group of domain objects treated as one consistency boundary.

The Aggregate Root controls access to the objects inside the aggregate.

### **What is CQRS?**

Command Query Responsibility Segregation separates operations that change state (commands) from operations that read data (queries).

This separation can make complex systems easier to scale and maintain, but it also introduces additional architectural complexity.

### **What is Event-Driven Architecture?**

Components communicate through events instead of directly depending on each other.

Example: after an order is created, separate listeners can send an email, update statistics, and publish a notification.

### **What is the difference between an Event and a Command?**

- An Event describes something that has already happened.
    
- A Command requests that something should happen.
    

For example, `OrderCreated` is an event, while `CreateOrder` is a command.

### **What is a Message Queue?**

A message queue allows tasks to be processed asynchronously.

Examples: RabbitMQ, Redis-based queues.

Benefits:

- Reduces request processing time.
    
- Supports background jobs.
    
- Improves scalability.
    
- Helps handle temporary failures through retries.
    

### **What is eventual consistency?**

Eventual consistency means that distributed systems may temporarily contain different data, but they are expected to become consistent over time.

### **What is caching?**

Caching stores frequently accessed data to reduce computation and database load.

Examples: Redis, Memcached, OPcache.

### **What is horizontal scaling?**

Horizontal scaling means adding more application servers.

### **What is vertical scaling?**

Vertical scaling means increasing the resources of an existing server, such as CPU and RAM.

### **What is a race condition?**

A race condition occurs when the result depends on the timing or order of concurrent operations.

Prevention includes transactions, locking, atomic operations, and proper concurrency control.

### **What is technical debt?**

Technical debt is the future cost caused by choosing a simpler or faster implementation instead of a more maintainable solution.

### **What is the difference between monolithic and microservices architecture?**

- A monolithic application is built and deployed as one main application.
    
- A microservices architecture divides the system into independently deployable services.
    

Microservices can improve independent scaling and team autonomy, but they introduce network communication, distributed data management, and operational complexity.

### **What is an API Gateway?**

An API Gateway is an entry point that routes client requests to appropriate backend services.

It can handle authentication, rate limiting, request routing, and other cross-cutting concerns.

### **What is a reverse proxy?**

A reverse proxy receives requests from clients and forwards them to backend servers.

It can provide load balancing, TLS termination, caching, and request filtering.

### **What is load balancing?**

Load balancing distributes incoming requests across multiple servers to improve availability and resource utilization.

### **What is a circuit breaker?**

A circuit breaker prevents repeated calls to a failing external service.

It temporarily blocks requests after failures and allows them again after a recovery period.

This helps prevent cascading failures.

### **What is retry with exponential backoff?**

It is a retry strategy where the delay between failed attempts increases over time.

For example, a client may wait 1 second, then 2 seconds, then 4 seconds before retrying.

It helps avoid overwhelming an already failing service.

### **How do you approach an unfamiliar codebase?**

1. Understand the project structure.
    
2. Read documentation and configuration.
    
3. Identify the main entry points.
    
4. Trace the request lifecycle.
    
5. Understand dependencies and data flow.
    
6. Run existing tests.
    
7. Make a small change and verify its impact.
    

### **How do you approach a production bug?**

1. Reproduce the issue.
    
2. Check logs and monitoring.
    
3. Identify the root cause.
    
4. Assess the impact.
    
5. Implement the smallest safe fix.
    
6. Add regression tests.
    
7. Deploy and monitor the result.
    

### **How do you perform a code review?**

I check correctness, readability, security, performance, test coverage, maintainability, and consistency with project architecture.

### **How do you decide whether to refactor code?**

I refactor when the current implementation makes changes risky, duplicates logic, or makes testing difficult.

I prefer small, incremental refactoring with tests to reduce the risk of introducing regressions.

## **17. PHP 8.x Features**

### **PHP 8.0**

- Named arguments.
    
- Attributes.
    
- Constructor property promotion.
    
- Union types.
    
- Match expressions.
    
- Nullsafe operator.
    
- JIT compilation.
    
- `mixed` type.
    
- `static` return type.
    
- String functions such as `str_contains()`, `str_starts_with()`, and `str_ends_with()`.
    

### **PHP 8.1**

- Enums.
    
- Fibers.
    
- Readonly properties.
    
- Intersection types.
    
- First-class callable syntax.
    
- `never` return type.
    
- `array_is_list()`.
    
- `readonly` properties.
    
- `new` in initializers for some contexts.
    

### **PHP 8.2**

- Readonly classes.
    
- Disjunctive Normal Form (DNF) types.
    
- Standalone `null`, `false`, and `true` types.
    
- Deprecation of dynamic properties.
    
- `true` type.
    
- `AllowDynamicProperties` attribute.
    

### **PHP 8.3**

- Typed class constants.
    
- Dynamic class constant fetch.
    
- `#[\Override]` attribute.
    
- `json_validate()`.
    
- New `Randomizer` functionality.
    

### **PHP 8.4**

- Property hooks.
    
- Asymmetric visibility.
    
- Property `new` initializers.
    
- Lazy objects.
    
- `#[\Deprecated]` attribute.
    
- New array functions such as `array_find()` and `array_any()`.
    

### **What are property hooks?**

Property hooks allow developers to define custom behavior when reading or writing properties.

They can be used for validation or computed values without manually writing getter and setter methods.

### **What is asymmetric visibility?**

Asymmetric visibility allows different visibility levels for reading and writing a property.

For example, a property can be public for reading but only protected for writing.

### **What are Fibers?**

Fibers provide a way to suspend and resume function execution.

They are a low-level mechanism used by asynchronous frameworks and libraries to implement concurrency-like workflows.

They do not automatically make PHP code asynchronous.

## **18. Common Practical Interview Questions**

### **How do you handle large files?**

I process files in chunks or use streams and generators instead of loading everything into memory.

### **How do you handle large database results?**

I use pagination, cursor-based iteration, or database-level chunking.

### **How do you avoid duplicate code?**

I extract reusable logic into services, functions, traits, or shared abstractions where appropriate.

### **How do you make code testable?**

I use dependency injection, small responsibilities, interfaces where useful, and avoid global state.

### **How do you improve a slow application?**

I profile the application, identify bottlenecks, optimize database queries, introduce caching, and measure the results.

### **How do you handle external API failures?**

I use timeouts, retries with backoff, error handling, logging, and circuit breakers when appropriate.

### **How do you prevent duplicate payment processing?**

I use idempotency keys, database constraints, transactions, and proper state transitions.

### **How do you manage database schema changes?**

I use version-controlled migrations, review their impact, and plan backward-compatible deployments.

### **How do you ensure backward compatibility?**

I avoid breaking public interfaces, introduce changes gradually, use deprecation periods, and maintain tests for existing behavior.

### **How do you choose between inheritance and composition?**

I generally prefer composition because it creates more flexible and loosely coupled code. I use inheritance when there is a clear and stable IS-A relationship.

### **When would you use an interface?**

When different implementations should follow the same contract.

### **When would you use an abstract class?**

When related classes share common state or implementation.

### **When would you use a Generator?**

When I need to process a large sequence of values without keeping the entire collection in memory.

### **How do you handle invalid user input?**

I validate input at the application boundary, return meaningful validation errors, and never trust client-side validation alone.

### **How do you handle database errors?**

I use transactions where appropriate, catch expected database exceptions, log useful diagnostic information, and return safe error responses without exposing internal details.

### **How do you structure a typical backend application?**

I separate the application into layers:

- Controllers handle HTTP requests and responses.
    
- Services coordinate business operations.
    
- Repositories handle data access.
    
- Entities represent domain objects.
    
- DTOs transfer data between layers.
    
- Validators check input and business constraints.
    

The exact structure depends on the size and complexity of the application.

### **What do you do when an API endpoint becomes slow?**

I measure the request first, inspect logs and database queries, identify expensive operations, and optimize the actual bottleneck.

Possible improvements include indexes, caching, pagination, eager loading, and background processing.

### **How do you safely deploy database changes?**

I prefer backward-compatible migrations. I first deploy changes that support both old and new application behavior, then deploy the application changes, and remove obsolete schema elements in a later deployment.

### **How do you work with legacy code?**

I first understand the current behavior and identify the most important risks. I add tests around existing functionality before making changes, then refactor incrementally.

## **19. Essential PHP Functions to Remember**

|Function|Purpose|
|---|---|
|`isset()`|Check if variable exists and is not null|
|`empty()`|Check if value is empty|
|`is_null()`|Check whether value is null|
|`unset()`|Remove variable or array element|
|`count()`|Count elements|
|`array_map()`|Transform elements|
|`array_filter()`|Filter elements|
|`array_reduce()`|Reduce array to one value|
|`array_merge()`|Merge arrays|
|`array_keys()`|Get array keys|
|`array_values()`|Get array values|
|`array_column()`|Extract a column from a multidimensional array|
|`array_key_exists()`|Check if a key exists|
|`in_array()`|Search for a value|
|`array_search()`|Find a value's key|
|`array_unique()`|Remove duplicate values|
|`array_reverse()`|Reverse array order|
|`array_slice()`|Extract a portion of an array|
|`array_splice()`|Remove or replace a portion of an array|
|`array_sum()`|Calculate the sum of values|
|`array_chunk()`|Split an array into chunks|
|`array_combine()`|Create an array from keys and values|
|`array_is_list()`|Check whether array has sequential integer keys|
|`array_find()`|Find first matching element (PHP 8.4)|
|`array_any()`|Check whether any element matches (PHP 8.4)|
|`sort()`|Sort values and reindex keys|
|`asort()`|Sort values while preserving keys|
|`ksort()`|Sort by keys|
|`usort()`|Sort using a custom comparison|
|`explode()`|Convert string to array|
|`implode()`|Convert array to string|
|`str_contains()`|Check substring|
|`str_starts_with()`|Check string prefix|
|`str_ends_with()`|Check string suffix|
|`trim()`|Remove whitespace|
|`substr()`|Extract part of string|
|`str_replace()`|Replace occurrences in a string|
|`sprintf()`|Format a string|
|`password_hash()`|Hash password|
|`password_verify()`|Verify password|
|`json_encode()`|Convert PHP value to JSON|
|`json_decode()`|Convert JSON to PHP value|
|`json_validate()`|Validate JSON syntax (PHP 8.3+)|
|`serialize()`|Serialize PHP value|
|`unserialize()`|Restore serialized value|
|`file_get_contents()`|Read file|
|`file_put_contents()`|Write file|
|`filter_var()`|Validate or sanitize a value|
|`class_exists()`|Check whether class exists|
|`method_exists()`|Check whether method exists|
|`property_exists()`|Check whether property exists|
|`is_array()`|Check whether value is an array|
|`is_object()`|Check whether value is an object|
|`is_string()`|Check whether value is a string|
|`is_int()`|Check whether value is an integer|
|`is_numeric()`|Check whether value is numeric|
|`gettype()`|Get the type of a variable|
|`get_class()`|Get an object's class name|

## **20. Key Differences — Quick Revision**

|Concept A|Concept B|
|---|---|
|`==` — loose comparison|`===` — strict comparison|
|`include` — warning|`require` — fatal error|
|`isset()` — exists and is not null|`array_key_exists()` — key exists even if null|
|`empty()` — checks false-like values|`isset()` — checks existence and non-null|
|Interface — contract|Abstract class — shared behavior|
|Trait — code reuse|Inheritance — parent-child relationship|
|`self` — defining class|`static` — called class|
|`array_map()` — transform|`array_filter()` — select|
|`array_reduce()` — aggregate|`foreach` — iterate|
|`array_column()` — extract field values|`array_map()` — transform elements|
|`in_array()` — search by value|`array_key_exists()` — search by key|
|`array_merge()` — merge and reindex numeric keys|`+` — union, left keys take priority|
|`unset()` — remove an element|`array_values()` — reindex values|
|`foreach` by value — no direct element modification|`foreach` by reference — modifies original elements|
|`return` — return a final value|`yield` — produce Generator values|
|Exception — application failure|Error — engine/programming failure|
|Hashing — one-way|Encryption — reversible|
|Authentication — identity|Authorization — permissions|
|PUT — replace|PATCH — partial update|
|Unit test — isolated unit|Integration test — component interaction|
|Stub — predefined response|Mock — verify interactions|
|Factory — create objects|Repository — manage data access|
|Strategy — interchangeable behavior|Factory — object creation|
|DI — provide dependencies|DIP — depend on abstractions|
|Vertical scaling — stronger server|Horizontal scaling — more servers|
|Synchronous — wait for result|Asynchronous — process independently|
|Entity — identity matters|Value Object — values matter|
|Command — request an action|Event — something that happened|
|`void` — no return value|`never` — never returns normally|
|Shallow copy — nested references may be shared|Deep copy — nested objects are duplicated|

## **Additional Interview Answer Patterns**

These are useful phrases for explaining your decisions during a technical interview.

### **How do you explain a technical decision?**

I chose this approach because it keeps the code simple, maintainable, and easy to test. It also avoids unnecessary coupling between components.

### **How do you explain a trade-off?**

Every solution has trade-offs. I consider performance, complexity, maintainability, and project requirements before choosing an implementation.

### **How do you explain that you do not know something?**

I have not worked with this feature in depth yet, but I understand its purpose. I would check the official documentation and test the behavior before implementing it in production.

### **How do you explain your debugging approach?**

I first reproduce the problem, inspect the logs, and identify where the actual behavior differs from the expected behavior. Then I isolate the root cause, implement a fix, and add a regression test.

## **Final Revision Checklist**

Use this checklist to assess your interview readiness.

- PHP types, strict typing, type juggling, comparison operators.
    
- Arrays, callbacks, closures, generators, references.
    
- OOP principles, interfaces, abstract classes, traits, enums.
    
- Dependency Injection, composition, inheritance, SOLID.
    
- Exceptions, Errors, Throwable, custom exceptions.
    
- Reference counting, garbage collection, OPcache.
    
- Composer, PSR-4, autoloading, semantic versioning.
    
- SQL, joins, indexes, transactions, ACID, N+1.
    
- HTTP methods, status codes, REST, JSON, JWT.
    
- SQL injection, XSS, CSRF, password hashing.
    
- Design patterns: Factory, Strategy, Repository, Adapter.
    
- PHPUnit, mocks, stubs, integration tests, TDD.
    
- Architecture: DTO, Entity, Value Object, CQRS, queues.
    
- PHP 8.0–8.4 features.
    
- Practical debugging, code review, production troubleshooting.
    

**Recommendation:** use this document as a theory reference, but practise explaining each topic aloud in English. For a Backend interview, being able to explain _why_ you chose a solution and demonstrate it in code is more valuable than memorizing definitions.