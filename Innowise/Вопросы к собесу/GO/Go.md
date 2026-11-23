---
custom-width: 80
---
## **1. Go Fundamentals**

### **What is Go?**

Go (Golang) is a statically typed, compiled, garbage-collected language created at Google. It is designed for simple syntax, fast compilation, built-in concurrency, and easy deployment. It is widely used for backend services, microservices, CLI tools, DevOps tooling (Docker, Kubernetes, Terraform), and network software.

### **Is Go compiled or interpreted?**

Go is compiled ahead of time into a single native machine-code binary. The binary includes the Go runtime (scheduler, garbage collector), so it usually has no external dependencies and is easy to deploy.

```
go build -o app ./cmd/app
./app
```

### **What are the main Go data types?**

- **Boolean:** `bool`.
    
- **Numeric:** `int`, `int8`–`int64`, `uint`, `uint8`–`uint64`, `uintptr`, `float32`, `float64`, `complex64`, `complex128`.
    
- **Text:** `string`, `byte` (alias of `uint8`), `rune` (alias of `int32`).
    
- **Composite:** `array`, `slice`, `map`, `struct`, `pointer`, `function`, `interface`, `channel`.
    

### **What are the zero values in Go?**

Every variable declared without an explicit value gets a zero value. There is no "uninitialized" state.

|Type|Zero value|
|---|---|
|`int`, `float64`|`0`|
|`bool`|`false`|
|`string`|`""`|
|pointer, slice, map, channel, function, interface|`nil`|
|`struct`|each field set to its zero value|

I design types so that the zero value is useful (for example `sync.Mutex`, `bytes.Buffer`).

### **What is the difference between** `var` **and** `:=`**?**

- `var` declares a variable and can be used at package level, with or without an initial value.
    
- `:=` is a short declaration, allowed only inside functions, and the type is inferred.
    

```
var count int       // zero value
var name = "Go"     // type inferred

func main() {
    age := 30       // short declaration
}
```

### **What is the difference between** `=` **and** `:=`**?**

- `:=` declares and assigns at least one new variable.
    
- `=` assigns to an already declared variable.
    

A common pitfall is shadowing: `:=` in an inner scope creates a new variable instead of reusing the outer one.

```
err := doA()
if true {
    err := doB() // new variable, outer err is NOT changed
    _ = err
}
```

### **What is the blank identifier** `_`**?**

It discards a value. Go does not allow unused variables or imports, so `_` is used to ignore results.

```
value, _ := strconv.Atoi("42") // ignore the error (only if safe)
import _ "github.com/lib/pq"   // import only for side effects (init)
```

### **What is the** `init()` **function?**

`init()` runs automatically once per package, after package-level variables are initialized and before `main()`. A package may contain several `init()` functions. I use it sparingly, because hidden initialization makes code harder to test.

### **What is the structure of a Go program?**

```
package main

import "fmt"

func main() {
    fmt.Println("Hello, Go")
}
```

- `package main` with `func main()` defines an executable.
    
- Other packages are libraries.
    
- Exported identifiers start with an uppercase letter.
    

### **How does visibility (exported vs unexported) work?**

There are no `public`/`private` keywords. If a name starts with an uppercase letter, it is exported (visible outside the package). If it starts with lowercase, it is visible only inside the package.

### **What is the difference between** `int` **and** `int64`**?**

`int` is platform-dependent (32 or 64 bits), while `int64` is always 64 bits. `int` and `int64` are different types and require explicit conversion. I use `int` for indexes and sizes, and fixed-size types for binary formats, databases, and APIs.

### **What is** `byte` **and** `rune`**?**

- `byte` is an alias for `uint8`; it represents one byte.
    
- `rune` is an alias for `int32`; it represents one Unicode code point.
    

### **Does Go have implicit type conversion?**

No. Go requires explicit conversion between different types, even between `int` and `int64`.

```
var a int = 10
var b float64 = float64(a) // explicit
```

### **What are untyped constants?**

Constants such as `42` or `3.14` have no fixed type until they are used. This lets them adapt to the context.

```
const x = 10
var f float64 = x // OK
var i int32 = x   // OK
```

### **What is** `iota`**?**

`iota` is an auto-incrementing counter inside a `const` block. It is used to create enums.

```
type Status int

const (
    Pending Status = iota // 0
    Active                // 1
    Blocked               // 2
)
```

### **Does Go have enums?**

Not as a language feature. The idiom is a named type with constants and `iota`, often with a `String()` method.

```
func (s Status) String() string {
    switch s {
    case Pending:
        return "pending"
    case Active:
        return "active"
    }
    return "unknown"
}
```

### **What is a pointer in Go?**

A pointer stores the memory address of a value. `&` takes an address, `*` dereferences. Go has no pointer arithmetic (except via the `unsafe` package).

```
x := 10
p := &x
*p = 20
fmt.Println(x) // 20
```

### **What is** `new` **and what is** `make`**?**

- `new(T)` allocates zeroed memory for `T` and returns `*T`.
    
- `make` creates and initializes slices, maps, and channels and returns the value itself (not a pointer).
    

```
p := new(int)             // *int pointing to 0
s := make([]int, 0, 10)   // slice len 0, cap 10
m := make(map[string]int) // ready-to-use map
ch := make(chan int, 5)   // buffered channel
```

### **What is the difference between** `nil` **in different types?**

`nil` is the zero value for pointers, slices, maps, channels, functions, and interfaces. Behavior differs:

- A `nil` slice can be read with `len`, appended to, and ranged over.
    
- A `nil` map can be read but writing causes a panic.
    
- A `nil` channel blocks forever on send and receive.
    
- A `nil` pointer dereference causes a panic.
    

### **What are the Go tools I use every day?**

|Command|Purpose|
|---|---|
|`go run`|Compile and run|
|`go build`|Compile into a binary|
|`go test`|Run tests, benchmarks|
|`go fmt` / `gofmt`|Format code|
|`go vet`|Static checks for suspicious code|
|`go mod tidy`|Sync dependencies|
|`go get`|Add/upgrade a dependency|
|`go install`|Build and install a tool|
|`go doc`|Show documentation|
|`go generate`|Run code generators|

## **2. Variables, Constants and Operators**

### **What is the difference between** `const` **and** `var`**?**

- `const` is a compile-time value and cannot change; it can only be a number, string, or bool (no slices, maps, structs).
    
- `var` is a variable that can change at runtime.
    

### **What is variable scope in Go?**

Go uses block scope: package, function, and any `{}` block (including `if`, `for`, `switch`). Names declared in an inner block shadow outer ones.

### **What is the difference between** `if` **with a short statement and a normal** `if`**?**

`if` can contain an initialization statement. The variable is visible only inside the `if/else` chain.

```
if err := doWork(); err != nil {
    return err
}
// err is not visible here
```

### **How does** `switch` **work in Go?**

- No automatic fall-through; `break` is not needed.
    
- Cases can be expressions, not only constants.
    
- `fallthrough` forces the next case.
    
- A switch without a condition replaces long `if/else` chains.
    

```
switch {
case score >= 90:
    grade = "A"
case score >= 75:
    grade = "B"
default:
    grade = "C"
}
```

### **What is a type switch?**

It checks the dynamic type of an interface value.

```
switch v := value.(type) {
case int:
    fmt.Println("int", v)
case string:
    fmt.Println("string", v)
default:
    fmt.Println("unknown")
}
```

### **What loops exist in Go?**

Only `for`, in several forms:

```
for i := 0; i < 10; i++ {}        // classic
for condition {}                  // like while
for {}                            // infinite
for i, v := range items {}        // range
for i := range 10 {}              // range over int (Go 1.22+)
```

### **What does** `range` **return?**

|Type|Values|
|---|---|
|slice / array|index, value|
|string|byte index, rune|
|map|key, value (random order)|
|channel|value (until closed)|
|integer (1.22+)|index|
|function (1.23+)|values from an iterator|

### **What is the loop variable gotcha before Go 1.22?**

Before Go 1.22 all iterations shared one loop variable, so closures and goroutines captured the same variable. Since Go 1.22 each iteration has its own copy (controlled by the `go` version in `go.mod`).

```
for _, v := range items {
    go func() { fmt.Println(v) }() // safe in 1.22+, buggy before
}
```

### **What are** `break`**,** `continue`**, and labels?**

`break` leaves the innermost `for`/`switch`/`select`; `continue` goes to the next iteration. Labels allow controlling an outer loop.

```
outer:
for _, row := range matrix {
    for _, v := range row {
        if v == 0 {
            continue outer
        }
    }
}
```

### **What is** `goto`**?**

Go has `goto`, but it is almost never used. Labeled `break`/`continue` and functions are cleaner.

### **What are the operators in Go?**

- Arithmetic: `+ - * / % ++ --` (`++` and `--` are statements, not expressions).
    
- Comparison: `== != < > <= >=`.
    
- Logical: `&& || !`.
    
- Bitwise: `& | ^ << >> &^` (AND NOT).
    
- Assignment: `= += -= *= /= %= &= |= ^= <<= >>=`.
    
- Channel: `<-`.
    

### **Is there a ternary operator in Go?**

No. Go deliberately omits it; use a normal `if/else`. For simple defaults, since 1.21 there is `cmp.Or` and built-in `min`/`max`.

### **What happens on integer overflow?**

Integers wrap around silently for signed and unsigned types. Division by zero for integers causes a panic; for floats it gives `+Inf`, `-Inf`, or `NaN`.

### **What is the difference between integer and float division?**

```
fmt.Println(7 / 2)     // 3 (integer division)
fmt.Println(7 / 2.0)   // 3.5
fmt.Println(7 % 3)     // 1
```

### **How do you compare floats?**

I do not compare floats with `==` because of precision errors. I compare with an epsilon: `math.Abs(a-b) < 1e-9`. For money I use integers (cents) or a decimal library.

### **Can you compare values of any type with** `==`**?**

Only comparable types: numbers, strings, bools, pointers, channels, interfaces, arrays and structs of comparable types. Slices, maps, and functions are not comparable (only with `nil`). `reflect.DeepEqual` or `slices.Equal` / `maps.Equal` can be used instead.

### **What is the** `comparable` **constraint?**

A generic constraint for types that support `==` and `!=`. It is often used for map keys in generic code.

## **3. Strings, Arrays, Slices and Maps**

### **How are strings represented in Go?**

A string is an immutable sequence of bytes, usually UTF-8 text. Internally it is a pointer to the data and a length.

```
s := "héllo"
fmt.Println(len(s))          // 6 bytes
fmt.Println(len([]rune(s)))  // 5 characters
```

### **Why are strings immutable and what does it mean for concatenation?**

You cannot modify a string in place. Each `+` creates a new string, so concatenation in a loop is expensive. I use `strings.Builder`.

```
var sb strings.Builder
for _, w := range words {
    sb.WriteString(w)
}
result := sb.String()
```

### **How do you iterate over characters of a string?**

`for range` decodes UTF-8 runes. Indexing `s[i]` returns a byte.

```
for i, r := range "Привет" {
    fmt.Println(i, string(r))
}
```

### **What is the difference between an array and a slice?**

- **Array:** fixed length that is part of its type (`[3]int` ≠ `[4]int`); it is a value type, copied on assignment.
    
- **Slice:** a dynamic view over an array; it is a small header with a pointer, `len`, and `cap`.
    

### **How is a slice implemented?**

A slice header contains three fields: pointer to the underlying array, length, capacity. Assigning or passing a slice copies the header, not the data, so both slices may point to the same array.

### **What is the difference between length and capacity?**

- `len` — number of elements in the slice.
    
- `cap` — number of elements from the slice start to the end of the underlying array.
    

### **How does** `append` **work?**

`append` adds elements. If capacity is enough, it writes into the same array; otherwise it allocates a bigger array, copies the data, and returns a new slice. Always assign the result.

```
s := []int{1, 2, 3}
s = append(s, 4)
```

### **What is the slice aliasing gotcha?**

Two slices can share the same array, so changing one can change the other.

```
a := []int{1, 2, 3, 4}
b := a[:2]
b[0] = 99
fmt.Println(a) // [99 2 3 4]

b = append(b, 100) // overwrites a[2] because cap allows it
fmt.Println(a)     // [99 2 100 4]
```

To avoid it I use the full slice expression `a[:2:2]` (limits capacity) or `slices.Clone`.

### **How do you copy a slice?**

```
dst := make([]int, len(src))
copy(dst, src)

dst2 := slices.Clone(src) // Go 1.21+
```

### **How do you remove an element from a slice?**

```
s = append(s[:i], s[i+1:]...)  // keeps order
s = slices.Delete(s, i, i+1)   // Go 1.21+

s[i] = s[len(s)-1]             // does not keep order
s = s[:len(s)-1]
```

### **Why preallocate slices?**

If I know the approximate size, `make([]T, 0, n)` avoids repeated reallocations and copying.

### **What is a memory leak with slices?**

A small sub-slice keeps the whole underlying array alive. If I need only a few elements from a huge slice, I copy them into a new slice so the large array can be collected.

### **What are** `nil` **slice vs empty slice?**

```
var a []int        // nil
b := []int{}       // empty, not nil
fmt.Println(a == nil, b == nil) // true false
fmt.Println(len(a), len(b))     // 0 0
```

Both work with `len`, `append`, and `range`. The difference matters for JSON: `nil` becomes `null`, empty becomes `[]`.

### **What is a map in Go?**

A hash table with unique keys. The key type must be comparable.

```
ages := map[string]int{"Anna": 30}
ages["Bob"] = 25
v, ok := ages["Carl"] // ok == false, v == 0
delete(ages, "Anna")
```

### **How do you check if a key exists?**

Use the "comma ok" idiom. Reading a missing key returns the zero value, so `ok` is needed to distinguish it.

### **Is map iteration ordered?**

No. The order is intentionally randomized. To get a deterministic order I collect and sort the keys.

```
keys := slices.Sorted(maps.Keys(m)) // Go 1.23+
```

### **Are maps safe for concurrent use?**

No. Concurrent read and write (or two writes) cause a fatal error. Options: `sync.RWMutex` with a normal map, `sync.Map` for special cases (mostly reads, disjoint keys), or sharding.

### **What happens when you write to a nil map?**

It panics. Always initialize with `make` or a literal.

### **Can you take the address of a map element?**

No. `&m["key"]` is not allowed because elements can move when the map grows. For struct values I store pointers or reassign the whole value.

### **How do you implement a set in Go?**

Use `map[T]struct{}`; the empty struct takes zero bytes.

```
set := map[string]struct{}{}
set["a"] = struct{}{}
_, exists := set["a"]
```

### **What is the** `slices` **and** `maps` **packages?**

Standard library packages (Go 1.21+) with generic helpers: `slices.Sort`, `slices.Contains`, `slices.Index`, `slices.BinarySearch`, `slices.Reverse`, `maps.Keys`, `maps.Clone`, and others.

### **What are multi-dimensional slices?**

A slice of slices. Each row must be allocated separately.

```
grid := make([][]int, rows)
for i := range grid {
    grid[i] = make([]int, cols)
}
```

### **What is** `clear` **(Go 1.21)?**

A built-in that removes all entries from a map or zeroes all elements of a slice.

## **4. Functions, Closures and defer**

### **How do functions work in Go?**

Functions are first-class values: they can be assigned to variables, passed as arguments, and returned. Arguments are always passed by value.

```
func add(a, b int) int {
    return a + b
}
```

### **Can a function return multiple values?**

Yes. This is the basis of Go's error handling.

```
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

### **What are named return values?**

Return values can have names, which act as variables initialized to zero values. A bare `return` returns them. They are useful for documentation and for `defer` modifications, but I avoid bare returns in long functions.

```
func parse(s string) (n int, err error) {
    defer func() {
        if err != nil {
            err = fmt.Errorf("parse: %w", err)
        }
    }()
    n, err = strconv.Atoi(s)
    return
}
```

### **What are variadic functions?**

Functions that accept a variable number of arguments of one type; inside they are a slice.

```
func sum(nums ...int) int {
    total := 0
    for _, n := range nums {
        total += n
    }
    return total
}

sum(1, 2, 3)
sum(values...) // spread a slice
```

### **Is Go pass-by-value or pass-by-reference?**

Always pass-by-value. A copy is made on each call. But copying a slice, map, channel, or pointer copies only the small header/reference, so the callee can modify shared data.

```
func change(s []int) { s[0] = 100 }   // modifies caller's data
func grow(s []int)   { s = append(s, 1) } // caller does not see it
```

### **What is a closure?**

A function that captures variables from the surrounding scope. The captured variables live as long as the closure.

```
func counter() func() int {
    c := 0
    return func() int {
        c++
        return c
    }
}

next := counter()
next() // 1
next() // 2
```

### **What are anonymous functions?**

Functions without a name, defined inline and often called immediately or passed to `go`/`defer`.

```
go func(id int) {
    process(id)
}(42)
```

### **What is** `defer`**?**

`defer` schedules a function call to run when the surrounding function returns (normally or via panic). Deferred calls run in LIFO order. Arguments are evaluated at the moment of the `defer` statement.

```
func readFile(path string) error {
    f, err := os.Open(path)
    if err != nil {
        return err
    }
    defer f.Close()
    // work with f
    return nil
}
```

### **What is the defer order and argument evaluation?**

```
for i := 0; i < 3; i++ {
    defer fmt.Println(i)
}
// prints 2 1 0
```

### **What are common** `defer` **pitfalls?**

- `defer` in a loop postpones everything until the function ends; wrap the body in a function for per-iteration cleanup.
    
- Ignoring the error from `Close()` on writes can hide data loss.
    
- Deferred calls have a small cost (much smaller in modern Go).
    

### **What is** `panic` **and** `recover`**?**

- `panic` stops normal execution and unwinds the stack running deferred functions.
    
- `recover` (only inside a deferred function) stops the panic and returns its value.
    

```
func safe() (err error) {
    defer func() {
        if r := recover(); r != nil {
            err = fmt.Errorf("recovered: %v", r)
        }
    }()
    riskyOperation()
    return nil
}
```

I use panic only for programmer errors and impossible states. Ordinary failures return `error`.

### **What is a method value and a method expression?**

```
f := obj.Method        // method value, bound to obj
g := Type.Method       // method expression, receiver is the first argument
```

### **What are higher-order functions?**

Functions that take or return other functions. Examples: middleware, functional options, `sort.Slice`, `slices.SortFunc`.

### **What are functional options?**

A pattern for flexible constructors with optional settings.

```
type Server struct {
    addr    string
    timeout time.Duration
}

type Option func(*Server)

func WithTimeout(d time.Duration) Option {
    return func(s *Server) { s.timeout = d }
}

func NewServer(addr string, opts ...Option) *Server {
    s := &Server{addr: addr, timeout: 30 * time.Second}
    for _, opt := range opts {
        opt(s)
    }
    return s
}
```

### **What is recursion in Go and is there tail-call optimization?**

Go supports recursion, but there is no tail-call optimization. Goroutine stacks grow dynamically, so deep recursion is possible, but I prefer iteration for very deep cases.

### **How do you sort in Go?**

```
sort.Ints(nums)
sort.Slice(users, func(i, j int) bool { return users[i].Age < users[j].Age })

slices.Sort(nums)
slices.SortFunc(users, func(a, b User) int { return cmp.Compare(a.Age, b.Age) })
```

Use `sort.SliceStable` / `slices.SortStableFunc` when equal elements must keep their order.

### **What are generic functions? (short)**

Functions with type parameters (Go 1.18+). Details are in the Generics section.

```
func Map[T, U any](items []T, f func(T) U) []U {
    result := make([]U, 0, len(items))
    for _, item := range items {
        result = append(result, f(item))
    }
    return result
}
```

## **5. Structs, Methods and OOP in Go**

### **Does Go have classes?**

No. Go has `struct` types with methods, interfaces, and embedding. It supports encapsulation (via exported/unexported names) and polymorphism (via interfaces), but not class inheritance.

### **What is a struct?**

A composite type that groups named fields.

```
type User struct {
    ID    int
    Name  string
    Email string
}

u := User{ID: 1, Name: "Anna"}
```

### **How do you define methods?**

A method is a function with a receiver.

```
func (u User) FullInfo() string {
    return fmt.Sprintf("%d %s", u.ID, u.Name)
}

func (u *User) Rename(name string) {
    u.Name = name
}
```

### **What is the difference between a value receiver and a pointer receiver?**

- **Value receiver** `(u User)` — works on a copy; cannot modify the original.
    
- **Pointer receiver** `(u *User)` — works on the original; can modify it and avoids copying large structs.
    

### **When do you use a pointer receiver?**

- The method modifies the receiver.
    
- The struct is large.
    
- The struct contains a `sync.Mutex` or similar (must not be copied).
    
- For consistency: if one method needs a pointer receiver, I use pointer receivers for all.
    

### **What is the method set rule?**

- The method set of `T` contains methods with receiver `T`.
    
- The method set of `*T` contains methods with receiver `T` and `*T`.
    

So a value of type `T` does not satisfy an interface if the interface methods have pointer receivers.

### **What is embedding?**

A struct can include another type without a field name. Its fields and methods are promoted.

```
type Animal struct{ Name string }

func (a Animal) Speak() string { return a.Name + " makes a sound" }

type Dog struct {
    Animal
    Breed string
}

d := Dog{Animal{"Rex"}, "Husky"}
fmt.Println(d.Speak()) // promoted
```

### **Is embedding the same as inheritance?**

No. It is composition with syntax sugar. The embedded type does not know about the outer type, so there is no virtual dispatch; the outer type can override a method by defining its own, but the inner method still calls the inner implementation.

### **How do you create a constructor?**

Go has no constructors. The convention is a `NewXxx` function that returns a value or pointer and validates input.

```
func NewUser(name string) (*User, error) {
    if name == "" {
        return nil, errors.New("name is required")
    }
    return &User{Name: name}, nil
}
```

### **What are struct tags?**

String metadata attached to fields, read through reflection by packages such as `encoding/json`, `db`, `validate`.

```
type User struct {
    ID    int    `json:"id" db:"id"`
    Email string `json:"email,omitempty" validate:"required,email"`
    Pass  string `json:"-"`
}
```

### **How do you compare structs?**

Structs can be compared with `==` if all fields are comparable. Structs with slices or maps are not comparable.

### **How do you copy a struct?**

Assignment copies the struct (shallow copy). Fields that are slices, maps, or pointers still share underlying data. For a deep copy I write a clone method.

### **What is an anonymous struct?**

A struct type without a name, useful in tests and for one-off JSON.

```
point := struct{ X, Y int }{1, 2}
```

### **What is the empty struct** `struct{}`**?**

A type with size zero. Used for sets (`map[T]struct{}`) and signal channels (`chan struct{}`).

### **How is encapsulation done in Go?**

By package boundaries and naming. Unexported fields can be accessed only inside the package, so I expose behavior through methods and constructors.

### **Does Go support method overloading?**

No. Function and method names must be unique within their scope. Optional parameters are replaced by variadic arguments, functional options, or config structs.

### **How does polymorphism work in Go?**

Through interfaces. Any type that implements the interface methods can be used where the interface is expected.

### **What is the difference between a struct and a class (for a PHP/Java developer)?**

|Go struct|Class (PHP/Java)|
|---|---|
|Methods defined outside the type body|Methods inside the class|
|No inheritance, only embedding|Inheritance|
|Implicit interface implementation|Explicit `implements`|
|No constructors, `NewXxx` functions|Constructors|
|No `private`/`protected` keywords|Access modifiers|
|Value semantics by default|Reference semantics for objects|

## **6. Interfaces and the Type System**

### **What is an interface in Go?**

A set of method signatures. A type implements an interface automatically by having those methods; there is no `implements` keyword (structural typing).

```
type Notifier interface {
    Notify(msg string) error
}

type EmailNotifier struct{}

func (EmailNotifier) Notify(msg string) error { return nil }

var n Notifier = EmailNotifier{}
```

### **What is the proverb about interfaces?**

"Accept interfaces, return structs." Functions accept the smallest interface they need and return concrete types. And: "The bigger the interface, the weaker the abstraction."

### **Where should interfaces be defined?**

On the consumer side (where they are used), not next to the implementation. This keeps interfaces small and decoupled.

### **What are small interfaces and examples from the standard library?**

```
type Reader interface { Read(p []byte) (n int, err error) }
type Writer interface { Write(p []byte) (n int, err error) }
type Stringer interface { String() string }
type error interface { Error() string }
```

### **What is the empty interface** `any`**?**

`interface{}` (alias `any` since Go 1.18) accepts any value. It loses type safety, so I avoid it when generics or concrete types are possible.

### **What is a type assertion?**

It extracts the concrete value from an interface.

```
s, ok := value.(string) // safe form
s := value.(string)     // panics if not a string
```

### **How is an interface value represented internally?**

As two words: a pointer to type information and a pointer to the data (`itab` + data for non-empty interfaces). An interface is `nil` only if both the type and the value are nil.

### **What is the nil interface gotcha?**

An interface holding a typed nil pointer is not equal to `nil`.

```
func find() error {
    var e *MyError = nil
    return e // returns a non-nil error interface!
}

err := find()
fmt.Println(err == nil) // false
```

Fix: return an explicit `nil`.

### **How do you verify at compile time that a type implements an interface?**

```
var _ Notifier = (*EmailNotifier)(nil)
```

### **What is interface embedding?**

Interfaces can include other interfaces.

```
type ReadWriter interface {
    Reader
    Writer
}
```

### **What are generics in Go?**

Type parameters (Go 1.18+) let functions and types work with many types while staying type-safe.

```
type Number interface {
    ~int | ~int64 | ~float64
}

func Sum[T Number](items []T) T {
    var total T
    for _, v := range items {
        total += v
    }
    return total
}
```

### **What are type constraints?**

An interface that describes allowed types for a type parameter: `any`, `comparable`, a union such as `int | string`, or a method set. `~int` means "any type whose underlying type is `int`". The standard package `cmp` provides `cmp.Ordered`.

### **What is a generic type?**

```
type Stack[T any] struct {
    items []T
}

func (s *Stack[T]) Push(v T) { s.items = append(s.items, v) }

func (s *Stack[T]) Pop() (T, bool) {
    var zero T
    if len(s.items) == 0 {
        return zero, false
    }
    v := s.items[len(s.items)-1]
    s.items = s.items[:len(s.items)-1]
    return v, true
}
```

### **When should you use generics and when not?**

I use them for containers, utility functions over slices/maps, and when the same algorithm works for many types. I avoid them when an interface is enough or when they make code harder to read. Methods cannot have their own type parameters.

### **What is the difference between an interface and generics?**

- Interface: behavior is chosen at runtime (dynamic dispatch).
    
- Generics: types are chosen at compile time; the algorithm is the same for the given types.
    

### **What is a named type vs a type alias?**

```
type UserID int       // new distinct type
type Alias = int      // alias, identical to int
```

Named types let me attach methods and prevent mixing values (for example `UserID` vs `OrderID`).

### **What is reflection?**

The `reflect` package inspects types and values at runtime. It is powerful but slow and loses compile-time safety, so I use it mainly in libraries (JSON, ORM, validators).

### **What is the** `unsafe` **package?**

It bypasses the type system (pointer conversion, sizes). I avoid it except in performance-critical or low-level code.

### **What is** `embed` **?**

The `embed` package (Go 1.16+) embeds files into the binary at compile time.

```
//go:embed templates/*.html
var templates embed.FS
```

## **7. Error Handling**

### **How does error handling work in Go?**

Errors are ordinary values. A function returns `error` as the last result, and the caller checks it explicitly.

```
data, err := os.ReadFile(path)
if err != nil {
    return fmt.Errorf("read config: %w", err)
}
```

### **What is the** `error` **interface?**

```
type error interface {
    Error() string
}
```

Any type with this method is an error.

### **How do you create errors?**

```
errors.New("not found")
fmt.Errorf("user %d not found", id)
```

### **What is error wrapping?**

Adding context while preserving the original error using `%w`.

```
return fmt.Errorf("load user %d: %w", id, err)
```

### **What are** `errors.Is` **and** `errors.As`**?**

- `errors.Is(err, target)` checks if any error in the chain equals a specific value.
    
- `errors.As(err, &target)` finds the first error in the chain of a specific type.
    

```
if errors.Is(err, sql.ErrNoRows) { ... }

var pathErr *fs.PathError
if errors.As(err, &pathErr) {
    fmt.Println(pathErr.Path)
}
```

### **What are sentinel errors?**

Predefined package-level error values that callers compare with `errors.Is`.

```
var ErrNotFound = errors.New("not found")
```

### **What are custom error types?**

Types that implement `Error()` and carry extra data.

```
type ValidationError struct {
    Field string
    Msg   string
}

func (e *ValidationError) Error() string {
    return e.Field + ": " + e.Msg
}
```

### **Sentinel error or custom type?**

- Sentinel: simple conditions the caller only needs to detect (`ErrNotFound`).
    
- Custom type: when the caller needs details (field, code, retry info).
    

### **What is** `errors.Join`**?**

Go 1.20 allows combining several errors into one, for example to collect validation errors. `errors.Is`/`As` work on joined errors.

### **Why not ignore errors?**

An ignored error hides failures and corrupts state. If I ignore one intentionally, I use `_ =` and add a comment explaining why.

### **Should you log and return an error?**

No. I either handle the error (log it and continue) or return it with context. Logging and returning produces duplicate log lines. The logging happens once, at the top level (handler, `main`).

### **When do you use** `panic`**?**

- Unrecoverable programmer errors (invalid state, failed initialization at startup).
    
- `Must...` helpers (`template.Must`, `regexp.MustCompile`) for values known at compile time.
    

A library should not panic across its public API.

### **How do you handle panics in a server?**

A recovery middleware catches panics per request, logs the stack trace, and returns HTTP 500. Panics in a goroutine without `recover` crash the whole program.

### **What is the** `error` **vs** `panic` **rule of thumb?**

Expected failure (file missing, bad input, network error) → return an error. Bug or impossible condition → panic.

### **How do you design errors for an API?**

I define domain errors in the service layer, wrap infrastructure errors with context, and map them to HTTP status codes in the handler layer.

```
switch {
case errors.Is(err, ErrNotFound):
    http.Error(w, "not found", http.StatusNotFound)
case errors.As(err, &valErr):
    http.Error(w, valErr.Error(), http.StatusBadRequest)
default:
    http.Error(w, "internal error", http.StatusInternalServerError)
}
```

## **8. Concurrency**

### **What is a goroutine?**

A lightweight function execution managed by the Go runtime, not by the OS. It starts with a small stack (about 2 KB, grows dynamically), so a program can run hundreds of thousands of goroutines.

```
go doWork()
```

### **What is the difference between concurrency and parallelism?**

- **Concurrency:** structuring a program as independent tasks that may overlap in time.
    
- **Parallelism:** running tasks at the same moment on several CPU cores.
    

Go provides concurrency primitives; parallelism depends on `GOMAXPROCS` and available cores.

### **What is the Go scheduler (GMP model)?**

- **G** — goroutine.
    
- **M** — OS thread.
    
- **P** — logical processor holding a run queue of goroutines.
    

The scheduler multiplexes many goroutines onto few threads. It supports work stealing between `P`s and asynchronous preemption (Go 1.14+).

### **What is** `GOMAXPROCS`**?**

The number of `P`s, that is, how many OS threads can execute Go code at once. By default it equals the number of CPU cores. Be careful in containers with CPU limits (newer Go versions respect cgroup limits; older ones may need the `automaxprocs` library).

### **What is a channel?**

A typed conduit for communication between goroutines.

```
ch := make(chan int)
go func() { ch <- 42 }()
v := <-ch
```

### **Buffered vs unbuffered channels?**

- **Unbuffered:** send blocks until another goroutine receives (synchronization point).
    
- **Buffered:** send blocks only when the buffer is full; receive blocks when empty.
    

### **What happens when you close a channel?**

- Receives from a closed channel return remaining values, then zero values with `ok == false`.
    
- Sending to a closed channel panics.
    
- Closing a closed or nil channel panics.
    
- Only the sender should close a channel, and only when it is needed (for example to stop `range`).
    

```
v, ok := <-ch
for v := range ch { ... } // ends when ch is closed
```

### **What are directional channels?**

Types that restrict usage and document intent.

```
func producer(out chan<- int) {}   // send only
func consumer(in <-chan int) {}    // receive only
```

### **What does** `select` **do?**

It waits on several channel operations and executes the one that is ready (random choice if several are ready). A `default` case makes it non-blocking.

```
select {
case msg := <-ch:
    handle(msg)
case <-time.After(time.Second):
    fmt.Println("timeout")
case <-ctx.Done():
    return ctx.Err()
}
```

### **What is a deadlock?**

A state where goroutines wait for each other forever. If all goroutines are blocked, the runtime reports `fatal error: all goroutines are asleep - deadlock!`. Typical cause: sending on an unbuffered channel with no receiver.

### **What is a goroutine leak?**

A goroutine that never finishes (blocked on a channel, lock, or I/O with no way out). It holds memory and resources. I prevent it by always giving goroutines an exit path: `context`, closing channels, or timeouts.

### **What is a race condition and how do you detect it?**

A data race happens when two goroutines access the same memory at the same time and at least one writes, without synchronization. The race detector finds it:

```
go test -race ./...
go run -race main.go
```

### **What does** `sync.WaitGroup` **do?**

It waits for a group of goroutines to finish.

```
var wg sync.WaitGroup
for _, job := range jobs {
    wg.Add(1)
    go func() {
        defer wg.Done()
        process(job)
    }()
}
wg.Wait()
```

Go 1.25 adds `wg.Go(func() {...})`, which combines `Add` and `Done`.

### **What is** `sync.Mutex` **and** `sync.RWMutex`**?**

- `Mutex` — exclusive lock.
    
- `RWMutex` — many readers or one writer; useful when reads dominate.
    

```
type Counter struct {
    mu sync.Mutex
    n  int
}

func (c *Counter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.n++
}
```

A mutex must not be copied after first use.

### **Mutex or channel?**

"Don't communicate by sharing memory; share memory by communicating." In practice:

- Channels: pass ownership of data, pipelines, signaling, worker pools.
    
- Mutex: protect shared state (cache, counters, map).
    

### **What is** `sync.Once`**?**

Runs a function exactly once, safely across goroutines. Used for lazy initialization. `sync.OnceValue` and `sync.OnceFunc` (Go 1.21+) are convenient wrappers.

### **What is** `sync/atomic`**?**

Lock-free operations on integers and pointers (`atomic.AddInt64`, `atomic.Int64`, `atomic.Value`). Good for simple counters and flags.

### **What is** `sync.Pool`**?**

A pool of temporary objects to reduce allocations and GC pressure (buffers, for example). Items may be removed by the GC at any time, so it is a cache, not a storage.

### **What is** `sync.Cond`**?**

A condition variable for waiting and notifying goroutines. Rare; channels are usually simpler.

### **What is** `context.Context`**?**

A standard way to carry cancellation, deadlines, and request-scoped values across API boundaries and goroutines.

```
ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
defer cancel()

req, _ := http.NewRequestWithContext(ctx, http.MethodGet, url, nil)
```

### **What are context rules?**

- Pass `ctx` as the first parameter named `ctx`.
    
- Do not store a context inside a struct (except in rare cases).
    
- Always call `cancel()` (usually with `defer`).
    
- Use context values only for request-scoped data (trace ID), not for optional parameters.
    
- Use a custom unexported key type for values.
    

### **What are the context constructors?**

`WithCancel`, `WithTimeout`, `WithDeadline`, `WithValue`, `WithCancelCause`, `WithoutCancel`, `AfterFunc`.

### **What is the worker pool pattern?**

A fixed number of goroutines read jobs from a channel. It limits concurrency.

```
jobs := make(chan Job)
results := make(chan Result)

var wg sync.WaitGroup
for i := 0; i < workers; i++ {
    wg.Add(1)
    go func() {
        defer wg.Done()
        for job := range jobs {
            results <- handle(job)
        }
    }()
}

go func() {
    for _, j := range allJobs {
        jobs <- j
    }
    close(jobs)
}()

go func() {
    wg.Wait()
    close(results)
}()
```

### **What is fan-out / fan-in?**

- **Fan-out:** several goroutines read from the same channel to share work.
    
- **Fan-in:** results from several channels are merged into one.
    

### **What is a pipeline?**

Stages connected by channels: each stage receives values, processes them, and sends them onward. Each stage should respect `ctx.Done()`.

### **What is** `errgroup`**?**

`golang.org/x/sync/errgroup` runs goroutines, collects the first error, and cancels the shared context. `SetLimit` bounds concurrency.

```
g, ctx := errgroup.WithContext(ctx)
g.SetLimit(5)
for _, url := range urls {
    g.Go(func() error {
        return fetch(ctx, url)
    })
}
if err := g.Wait(); err != nil { ... }
```

### **How do you limit concurrency?**

- A worker pool.
    
- A buffered channel as a semaphore.
    
- `errgroup.SetLimit`.
    
- `golang.org/x/sync/semaphore`.
    

```
sem := make(chan struct{}, 10)
sem <- struct{}{}        // acquire
defer func() { <-sem }() // release
```

### **How do you implement a timeout?**

With `context.WithTimeout` and `select`, or `time.After` for simple cases. In loops I use `time.NewTimer` to avoid creating many timers.

### **What is a rate limiter?**

`time.Ticker` or `golang.org/x/time/rate` (token bucket) restrict the number of operations per time unit.

### **How do you stop a goroutine gracefully?**

I give it a `ctx` or a `done` channel and check it in a `select`. The owner waits for completion with `WaitGroup` before exiting.

### **What is graceful shutdown of an HTTP server?**

```
srv := &http.Server{Addr: ":8080", Handler: mux}

go srv.ListenAndServe()

ctx, stop := signal.NotifyContext(context.Background(), os.Interrupt, syscall.SIGTERM)
defer stop()
<-ctx.Done()

shutdownCtx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
defer cancel()
srv.Shutdown(shutdownCtx)
```

### **Is** `sync.Map` **better than a map with a mutex?**

Not by default. It is optimized for two cases: keys written once and read many times, or goroutines working with disjoint sets of keys. In other cases a normal map with `RWMutex` is simpler and often faster.

### **What are the common concurrency bugs?**

- Data races.
    
- Deadlocks.
    
- Goroutine leaks.
    
- Writing to a closed channel.
    
- Copying a mutex.
    
- Capturing a loop variable (before Go 1.22).
    
- Forgetting `wg.Add` before starting a goroutine.
    

### **What is the happens-before relationship?**

The Go memory model defines when a write in one goroutine is guaranteed to be visible in another. Synchronization operations (channel send/receive, mutex unlock/lock, `sync.Once`, atomics) create this ordering.

## **9. Memory Management and Performance**

### **How does garbage collection work in Go?**

Go uses a concurrent, non-generational, tri-color mark-and-sweep garbage collector. Most work happens concurrently with the program, so stop-the-world pauses are very short (typically sub-millisecond).

### **What are stack and heap in Go?**

- **Stack:** per goroutine, cheap, freed automatically when a function returns.
    
- **Heap:** shared, managed by the GC.
    

The compiler decides where a variable lives through escape analysis, not the programmer.

### **What is escape analysis?**

The compiler checks whether a value can be referenced after the function returns. If yes, it "escapes" to the heap. Returning a pointer to a local variable is safe in Go.

```
go build -gcflags="-m" ./...
```

### **What causes heap allocations?**

Returning pointers to locals, storing values in interfaces, closures capturing variables, slices that grow, large objects, and values passed to `fmt.Println` or other `any` parameters.

### **How can you reduce allocations?**

- Preallocate slices and maps with known capacity.
    
- Use `strings.Builder` and `bytes.Buffer`.
    
- Reuse buffers with `sync.Pool`.
    
- Avoid unnecessary conversions (`[]byte` ↔ `string`).
    
- Pass small structs by value, large ones by pointer.
    

### **What are** `GOGC` **and** `GOMEMLIMIT`**?**

- `GOGC` (default 100) controls how much the heap may grow before the next GC cycle.
    
- `GOMEMLIMIT` (Go 1.19+) sets a soft memory limit; useful in containers to prevent out-of-memory kills.
    

### **How do you profile a Go application?**

With `pprof` and `go tool trace`.

```
import _ "net/http/pprof"

// then:
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
go tool pprof http://localhost:6060/debug/pprof/heap
```

Profiles: CPU, heap, goroutine, block, mutex, allocs.

### **How do you write a benchmark?**

```
func BenchmarkConcat(b *testing.B) {
    for i := 0; i < b.N; i++ {
        concat()
    }
}
```

```
go test -bench=. -benchmem
```

`-benchmem` shows allocations per operation. Go 1.24 introduces `b.Loop()` as a safer loop form.

### **What are memory leaks in Go?**

Even with a GC, leaks are possible: goroutines that never exit, unbounded caches or maps, slices keeping huge arrays alive, forgotten `time.Ticker` (call `Stop()`), unclosed response bodies and files, and global references.

### **What is the cost of converting between string and []byte?**

Each conversion normally copies data. In hot paths I avoid repeated conversions or use `strings.Builder`/`bytes.Buffer`.

### **What is struct padding and alignment?**

The compiler adds padding between fields to align them. Ordering fields from large to small can reduce struct size.

### **What are the performance rules I follow?**

I measure first (benchmark, pprof), optimize the actual bottleneck, and keep the code readable. Premature optimization is avoided.

### **What does the** `runtime` **package offer?**

`runtime.NumGoroutine()`, `runtime.GC()`, `runtime.ReadMemStats()`, `runtime.Gosched()`, `runtime.NumCPU()`, `runtime.Caller()`.

## **10. Modules, Packages and Project Structure**

### **What is a Go module?**

A module is a collection of packages versioned together, described by `go.mod` at the root. It replaced `GOPATH`-based development.

```
module github.com/acme/orders

go 1.23

require github.com/google/uuid v1.6.0
```

### **What is the difference between a package and a module?**

- **Package:** a directory of `.go` files with the same `package` name; the unit of compilation and import.
    
- **Module:** a set of packages released and versioned together.
    

### **What are** `go.mod` **and** `go.sum`**?**

- `go.mod` — module path, Go version, direct and indirect dependencies.
    
- `go.sum` — checksums of dependency versions, used to verify integrity.
    

Both are committed to version control.

### **What do the main module commands do?**

|Command|Purpose|
|---|---|
|`go mod init`|Create a module|
|`go mod tidy`|Add missing and remove unused dependencies|
|`go get pkg@version`|Add or change a dependency version|
|`go mod vendor`|Copy dependencies into `vendor/`|
|`go mod why`|Explain why a package is needed|
|`go list -m all`|List all dependencies|
|`go mod download`|Download modules to the cache|

### **How does Go versioning work?**

Semantic versioning. For major version 2 and above, the import path must end with `/v2` (module path suffix). Go uses Minimal Version Selection (MVS): it picks the minimum version that satisfies all requirements, which makes builds reproducible.

### **What is a** `replace` **directive?**

It substitutes a module with another version or a local path, for example to test a local fork.

```
replace github.com/acme/lib => ../lib
```

### **What is** `internal/`**?**

Packages inside an `internal` directory can be imported only by code in the parent of that directory. It is the way to hide implementation details from other modules.

### **What is a Go workspace?**

`go.work` lets me work on several modules at once locally without `replace` directives (Go 1.18+).

### **How do you organize a typical Go project?**

```
project/
├── cmd/
│   └── api/main.go        # entry points
├── internal/
│   ├── handler/           # HTTP layer
│   ├── service/           # business logic
│   ├── repository/        # data access
│   └── domain/            # entities, errors
├── pkg/                   # public reusable code (optional)
├── migrations/
├── go.mod
└── go.sum
```

The layout is a convention, not a rule. For small projects, a flat structure is better.

### **What are good package naming rules?**

Short, lowercase, one word, no underscores, no `util`/`common`. The package name is part of the call: `http.Client`, not `http.HTTPClient`.

### **How do you handle import cycles?**

Go forbids them. I solve them by extracting shared types into a separate package, moving an interface to the consumer side, or rethinking the dependency direction.

### **What are build tags?**

Constraints for conditional compilation.

```
//go:build linux && amd64
```

Also file suffixes: `_linux.go`, `_test.go`, `_windows.go`.

### **How do you cross-compile?**

```
GOOS=linux GOARCH=amd64 go build -o app ./cmd/api
```

### **What are** `ldflags` **used for?**

Setting variables at build time (version, commit) and stripping symbols.

```
go build -ldflags "-s -w -X main.version=1.2.3" ./cmd/api
```

### **What is** `CGO`**?**

A mechanism to call C code from Go. It complicates cross-compilation and static binaries; `CGO_ENABLED=0` produces a fully static binary.

### **How do you write a minimal Dockerfile for Go?**

A multi-stage build: compile in a Go image, copy the binary into a small image.

```
FROM golang:1.23 AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /app ./cmd/api

FROM gcr.io/distroless/static
COPY --from=build /app /app
ENTRYPOINT ["/app"]
```

### **What are linters and static analysis tools?**

`go vet`, `staticcheck`, `golangci-lint` (aggregator), `gofumpt`, `goimports`, `govulncheck` (known vulnerabilities in dependencies).

## **11. Database and SQL**

### **How do you work with databases in Go?**

Through the standard `database/sql` package plus a driver (`pgx`, `lib/pq`, `go-sql-driver/mysql`). Higher-level options: `sqlx`, `sqlc` (code generation from SQL), `GORM`, `ent`.

```
db, err := sql.Open("pgx", dsn)
if err != nil {
    return err
}
defer db.Close()

if err := db.PingContext(ctx); err != nil {
    return err
}
```

### **What does** `sql.DB` **represent?**

A connection pool, not a single connection. It is safe for concurrent use and should be created once and shared.

### **How do you configure the connection pool?**

```
db.SetMaxOpenConns(25)
db.SetMaxIdleConns(25)
db.SetConnMaxLifetime(5 * time.Minute)
db.SetConnMaxIdleTime(1 * time.Minute)
```

Limits should match database capacity and the number of application instances.

### **How do you run a query?**

```
rows, err := db.QueryContext(ctx, "SELECT id, name FROM users WHERE age > $1", 18)
if err != nil {
    return err
}
defer rows.Close()

for rows.Next() {
    var u User
    if err := rows.Scan(&u.ID, &u.Name); err != nil {
        return err
    }
    users = append(users, u)
}
return rows.Err()
```

### **What is the difference between** `Query`**,** `QueryRow`**, and** `Exec`**?**

- `Query` — many rows.
    
- `QueryRow` — one row, error returned by `Scan`.
    
- `Exec` — statements without result rows (INSERT, UPDATE, DELETE).
    

### **What are common mistakes with** `rows`**?**

- Forgetting `rows.Close()`, which leaks connections.
    
- Not checking `rows.Err()` after the loop.
    
- Not using the `Context` variants.
    

### **How do you handle NULL values?**

Use `sql.NullString`, `sql.NullInt64`, `sql.Null[T]` (Go 1.22+), or pointer fields.

### **How do you handle "no rows"?**

`QueryRow().Scan()` returns `sql.ErrNoRows`. I convert it into a domain error such as `ErrNotFound`.

### **How do you prevent SQL injection?**

Always use parameterized queries (placeholders), never string concatenation.

```
db.QueryContext(ctx, "SELECT * FROM users WHERE email = $1", email)
```

Placeholders: `$1` in PostgreSQL, `?` in MySQL/SQLite. Identifiers (table/column names) cannot be parameters, so they must come from an allow-list.

### **How do transactions work?**

```
tx, err := db.BeginTx(ctx, nil)
if err != nil {
    return err
}
defer tx.Rollback() // safe no-op after Commit

if _, err := tx.ExecContext(ctx, "UPDATE accounts SET balance = balance - $1 WHERE id = $2", amount, from); err != nil {
    return err
}
if _, err := tx.ExecContext(ctx, "UPDATE accounts SET balance = balance + $1 WHERE id = $2", amount, to); err != nil {
    return err
}
return tx.Commit()
```

### **What is ACID?**

- **Atomicity** — all or nothing.
    
- **Consistency** — data stays valid according to constraints.
    
- **Isolation** — concurrent transactions do not break each other.
    
- **Durability** — committed data survives crashes.
    

### **What are isolation levels?**

Read Uncommitted, Read Committed, Repeatable Read, Serializable. Higher levels prevent more anomalies (dirty reads, non-repeatable reads, phantoms) but reduce concurrency. In Go: `db.BeginTx(ctx, &sql.TxOptions{Isolation: sql.LevelSerializable})`.

### **What is a prepared statement?**

A query parsed once and executed many times with different parameters.

```
stmt, err := db.PrepareContext(ctx, "INSERT INTO logs(msg) VALUES($1)")
defer stmt.Close()
stmt.ExecContext(ctx, "hello")
```

### **What is the N+1 problem?**

Running one query for a list and then one extra query per item. Solutions: `JOIN`, `WHERE id = ANY($1)` batch loading, or a dataloader.

### **What is an index and when do you use it?**

An index speeds up reads on columns used in `WHERE`, `JOIN`, `ORDER BY`. It slows writes and uses space. I check query plans with `EXPLAIN ANALYZE`.

### **What are SQL joins?**

- `INNER JOIN` — matching rows in both tables.
    
- `LEFT JOIN` — all rows from the left table plus matches.
    
- `RIGHT JOIN` — all rows from the right plus matches.
    
- `FULL JOIN` — all rows from both.
    

### **What are migrations and which tools do you use?**

Versioned scripts that change the database schema: `golang-migrate`, `goose`, `atlas`. They live in the repository and run in CI/CD.

### **What is the repository pattern in Go?**

An interface that hides data access from business logic.

```
type UserRepository interface {
    GetByID(ctx context.Context, id int64) (*User, error)
    Create(ctx context.Context, u *User) error
}
```

### **ORM or plain SQL?**

Many Go teams prefer SQL-first tools (`sqlc`, `sqlx`, `pgx`) because queries stay explicit and fast. GORM or `ent` speed up development for CRUD, but can hide queries and cause N+1. I choose by project size and team preference.

### **How do you implement pagination?**

- Offset pagination: `LIMIT n OFFSET m` — simple, slow on large offsets.
    
- Keyset (cursor) pagination: `WHERE id > $last ORDER BY id LIMIT n` — stable and fast.
    

### **What is Redis used for in Go services?**

Caching, rate limiting, sessions, distributed locks, queues, pub/sub. Popular client: `go-redis`.

### **How do you handle timeouts in database calls?**

I always pass a `context` with a deadline, so a slow query is cancelled and the connection is released.

### **What is connection pool exhaustion?**

All connections are in use (for example, unclosed `rows` or long transactions), and new requests wait. I monitor `db.Stats()`, close resources, and keep transactions short.

## **12. HTTP, REST and APIs**

### **How do you create an HTTP server in Go?**

```
mux := http.NewServeMux()
mux.HandleFunc("GET /users/{id}", func(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")
    fmt.Fprintf(w, "user %s", id)
})

log.Fatal(http.ListenAndServe(":8080", mux))
```

Since Go 1.22 `ServeMux` supports methods and path parameters in patterns.

### **What is** `http.Handler`**?**

An interface with one method:

```
type Handler interface {
    ServeHTTP(w http.ResponseWriter, r *http.Request)
}
```

`http.HandlerFunc` adapts a function to this interface.

### **What is middleware?**

A function that wraps a handler to add behavior (logging, auth, recovery, CORS, rate limit).

```
func Logging(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        start := time.Now()
        next.ServeHTTP(w, r)
        log.Printf("%s %s %v", r.Method, r.URL.Path, time.Since(start))
    })
}
```

### **Which routers and frameworks are popular?**

Standard `net/http` (often enough), `chi` (lightweight, compatible with `net/http`), `gin`, `echo`, `fiber`. I prefer `net/http` or `chi` for simplicity and compatibility.

### **How do you work with JSON?**

```
type CreateUserRequest struct {
    Name  string `json:"name"`
    Email string `json:"email"`
}

var req CreateUserRequest
if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
    http.Error(w, "bad request", http.StatusBadRequest)
    return
}

w.Header().Set("Content-Type", "application/json")
w.WriteHeader(http.StatusCreated)
json.NewEncoder(w).Encode(resp)
```

### **How can you make JSON decoding stricter?**

```
dec := json.NewDecoder(r.Body)
dec.DisallowUnknownFields()
r.Body = http.MaxBytesReader(w, r.Body, 1<<20) // limit body size
```

### **What are** `json.Marshal` **gotchas?**

- Only exported fields are encoded.
    
- `nil` slice becomes `null`; empty slice becomes `[]`.
    
- `omitempty` skips zero values (including `0` and `false`); use pointers to distinguish "missing" from "zero".
    
- Numbers decode into `float64` when the target is `any`.
    
- Custom behavior is possible via `MarshalJSON` / `UnmarshalJSON`.
    

### **What are the main HTTP methods?**

|Method|Purpose|Idempotent|
|---|---|---|
|GET|Read|Yes|
|POST|Create / action|No|
|PUT|Replace|Yes|
|PATCH|Partial update|Not necessarily|
|DELETE|Delete|Yes|

### **What are common HTTP status codes?**

- `200 OK`, `201 Created`, `204 No Content`.
    
- `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`, `429 Too Many Requests`.
    
- `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`, `504 Gateway Timeout`.
    

### **What is REST?**

An architectural style: resources identified by URLs, standard HTTP methods, stateless requests, representations (usually JSON), and meaningful status codes.

### **How do you design a REST API?**

- Nouns for resources: `/users`, `/users/{id}/orders`.
    
- Correct methods and status codes.
    
- Pagination, filtering, sorting for collections.
    
- Versioning (`/v1/...` or headers).
    
- Consistent error format.
    
- Idempotency for unsafe operations when needed.
    

### **How do you configure an HTTP client properly?**

The default client has no timeout, which is dangerous. I create my own client and reuse it.

```
client := &http.Client{Timeout: 10 * time.Second}

resp, err := client.Do(req)
if err != nil {
    return err
}
defer resp.Body.Close()
```

### **Why must you close** `resp.Body`**?**

Otherwise the connection cannot be reused and resources leak. I also read the body to the end (or `io.Copy(io.Discard, ...)`) if I want connection reuse.

### **What are the server timeouts?**

```
srv := &http.Server{
    Addr:              ":8080",
    Handler:           mux,
    ReadHeaderTimeout: 5 * time.Second,
    ReadTimeout:       10 * time.Second,
    WriteTimeout:      15 * time.Second,
    IdleTimeout:       60 * time.Second,
}
```

Without them the server is vulnerable to slow-client attacks.

### **What is CORS?**

A browser mechanism controlling cross-origin requests. The server sends headers such as `Access-Control-Allow-Origin`. It is implemented in middleware and does not protect the API from non-browser clients.

### **What is JWT?**

A signed token with claims (header.payload.signature). The server validates the signature and expiration without storing sessions. I keep tokens short-lived, use refresh tokens, never put secrets in the payload (it is only encoded, not encrypted), and validate `alg`, `exp`, `iss`, `aud`.

### **Sessions or JWT?**

- Sessions: easy revocation, server-side state.
    
- JWT: stateless, convenient for microservices, harder to revoke.
    

### **What is gRPC?**

A high-performance RPC framework using HTTP/2 and Protocol Buffers. It supports streaming, strong typing, and code generation. It is common for internal service-to-service communication in Go.

### **REST or gRPC?**

REST/JSON for public APIs and browser clients; gRPC for internal services needing speed, contracts, and streaming.

### **What is WebSocket and when do you use it?**

A persistent full-duplex connection for real-time features (chat, notifications). Libraries: `gorilla/websocket`, `nhooyr.io/websocket` (`coder/websocket`). Server-Sent Events are simpler for one-way streams.

### **What is idempotency and how do you implement it?**

An operation is idempotent if repeating it gives the same result. For POST requests I accept an `Idempotency-Key` header, store the result, and return it for repeated requests.

### **How do you version an API?**

URL (`/v1/users`), header, or media type. I prefer URL versioning for clarity and avoid breaking changes inside a version.

### **What is the request lifecycle in a Go service?**

Router → middleware (logging, recovery, auth) → handler (parse and validate) → service (business logic) → repository (database) → response (status code and JSON).

### **How do you structure logging?**

With `log/slog` (Go 1.21+): structured, leveled logs with key-value pairs and a request ID.

```
slog.Info("user created", "user_id", id, "ip", r.RemoteAddr)
```

### **What is observability?**

Logs, metrics (Prometheus), and traces (OpenTelemetry). In Go services I also expose `/healthz` and `/readyz` endpoints and `pprof` on an internal port.

## **13. Security**

### **How do you prevent SQL injection in Go?**

Parameterized queries (`$1`, `?`), no string concatenation, allow-lists for dynamic identifiers, least-privilege database users.

### **How do you prevent XSS?**

Use `html/template`, which escapes output automatically by context. I never use `text/template` for HTML and avoid `template.HTML` with untrusted data. I also use a Content-Security-Policy header.

### **What is CSRF and how do you protect against it?**

An attack that makes a browser send an authenticated request from another site. Protection: CSRF tokens, `SameSite` cookies, checking `Origin`/`Referer`. Pure token-based APIs (Authorization header) are not affected by cookie-based CSRF.

### **How do you store passwords?**

Hash them with a slow, salted algorithm: `bcrypt`, `argon2id`, or `scrypt`. Never store plain text or use fast hashes like MD5/SHA-256 alone.

```
hash, err := bcrypt.GenerateFromPassword([]byte(password), bcrypt.DefaultCost)
err = bcrypt.CompareHashAndPassword(hash, []byte(input))
```

### **What is the difference between hashing and encryption?**

Hashing is one-way; encryption is reversible with a key.

### **Which random generator do you use for security?**

`crypto/rand`, never `math/rand`, for tokens, keys, and session IDs.

```
b := make([]byte, 32)
_, err := rand.Read(b) // crypto/rand
```

### **How do you compare secrets safely?**

`subtle.ConstantTimeCompare` avoids timing attacks.

### **How do you use TLS in Go?**

`http.ListenAndServeTLS` or, more commonly, TLS is terminated by a reverse proxy/load balancer. For automatic certificates there is `golang.org/x/crypto/acme/autocert`. I never set `InsecureSkipVerify: true` in production.

### **How do you manage secrets?**

Environment variables or a secret manager (Vault, cloud secret managers). Secrets are not committed to Git and are never logged.

### **What is authentication vs authorization?**

- Authentication — who the user is.
    
- Authorization — what the user is allowed to do.
    

### **What is path traversal?**

A request such as `../../etc/passwd` reading files outside the intended directory. I clean and validate paths, use `filepath.Clean`, `http.FileServer` with `http.Dir`, or `os.Root` (Go 1.24+).

### **What is SSRF?**

A server-side request forgery: an attacker makes the server request internal resources. I validate and allow-list target URLs and block private IP ranges.

### **How do you limit abuse?**

Rate limiting, request size limits (`http.MaxBytesReader`), timeouts, input validation, and authentication.

### **How do you check dependencies for vulnerabilities?**

`govulncheck ./...`, Dependabot/Renovate, pinned versions in `go.sum`.

### **What is the OWASP Top 10?**

A list of the most critical web security risks: broken access control, cryptographic failures, injection, insecure design, misconfiguration, vulnerable components, authentication failures, data integrity failures, logging failures, SSRF.

### **How do you validate input?**

At the application boundary: decode into a struct, validate fields (`go-playground/validator` or manual checks), return `400/422` with clear messages, and never rely on client-side validation.

## **14. SOLID Principles in Go**

### **What is SOLID?**

Five object-oriented design principles that also apply to Go through interfaces and composition.

### **Single Responsibility Principle**

A type or package should have one reason to change. A handler parses HTTP, a service has business rules, a repository works with the database.

### **Open/Closed Principle**

Code is open for extension and closed for modification. In Go: add new behavior by implementing an interface instead of editing existing code (for example a new `PaymentProvider`).

### **Liskov Substitution Principle**

Any implementation of an interface must behave according to its contract. If an implementation returns unexpected errors or panics, it breaks callers.

### **Interface Segregation Principle**

Prefer small interfaces. Go's `io.Reader` and `io.Writer` are the standard example; a consumer should not depend on methods it does not use.

### **Dependency Inversion Principle**

High-level code depends on abstractions, not on concrete implementations. In Go: the service depends on a `UserRepository` interface defined by the service, and `main` wires the real implementation.

### **What is Dependency Injection in Go?**

Passing dependencies through constructors rather than creating them inside or using globals.

```
type UserService struct {
    repo   UserRepository
    mailer Mailer
}

func NewUserService(repo UserRepository, mailer Mailer) *UserService {
    return &UserService{repo: repo, mailer: mailer}
}
```

### **Which DI tools exist?**

Manual wiring in `main` (most common), `google/wire` (compile-time code generation), `uber-go/fx` and `dig` (runtime). I prefer manual wiring because it is explicit.

### **Is SOLID always applicable in Go?**

Applied pragmatically. Go values simplicity: I avoid excessive interfaces and layers, and introduce an abstraction when there is a real need (tests, several implementations).

## **15. Design Patterns**

### **Which patterns are common in Go?**

Functional options, Factory (`NewXxx`), Strategy (interfaces), Repository, Adapter, Decorator (middleware), Worker pool, Pipeline, Singleton (`sync.Once`), Observer (channels/callbacks).

### **Factory**

A function that creates objects and hides construction details.

```
func NewStorage(kind string) (Storage, error) {
    switch kind {
    case "memory":
        return NewMemoryStorage(), nil
    case "postgres":
        return NewPostgresStorage(), nil
    }
    return nil, fmt.Errorf("unknown storage: %s", kind)
}
```

### **Strategy**

Interchangeable algorithms behind one interface.

```
type PaymentMethod interface {
    Pay(amount int) error
}

type Card struct{}
type PayPal struct{}
```

### **Decorator / Middleware**

Wraps an object or handler to add behavior without changing it. HTTP middleware is the typical example.

### **Adapter**

Converts one interface into another expected by the client. For example, wrapping a third-party SDK behind my own interface.

### **Singleton**

One instance for the whole program, created lazily and safely with `sync.Once`.

```
var (
    instance *Config
    once     sync.Once
)

func GetConfig() *Config {
    once.Do(func() { instance = loadConfig() })
    return instance
}
```

I prefer dependency injection to global singletons because it is easier to test.

### **Observer**

Subscribers react to events. In Go it is often implemented with channels or callbacks.

### **Builder**

Step-by-step creation of complex objects. In Go, functional options or a config struct usually replace it.

### **Repository**

An abstraction over data storage, see the Database section.

### **Circuit Breaker**

Stops calling a failing dependency for a time to avoid cascading failures (`sony/gobreaker`).

### **Retry with backoff**

Repeat failed calls with growing delays and jitter, only for idempotent operations and transient errors.

### **Pub/Sub and Event-Driven design**

Services communicate through events (Kafka, RabbitMQ, NATS) instead of direct calls, which reduces coupling.

## **16. Testing**

### **How do you write tests in Go?**

Files end with `_test.go`; functions are `TestXxx(t *testing.T)`.

```
func TestAdd(t *testing.T) {
    got := Add(2, 3)
    if got != 5 {
        t.Errorf("Add(2, 3) = %d, want 5", got)
    }
}
```

```
go test ./...
go test -v -run TestAdd ./pkg
go test -cover ./...
```

### **What is a table-driven test?**

The standard Go style: a table of cases and a loop with subtests.

```
func TestDivide(t *testing.T) {
    tests := []struct {
        name    string
        a, b    float64
        want    float64
        wantErr bool
    }{
        {"ok", 10, 2, 5, false},
        {"zero", 1, 0, 0, true},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Divide(tt.a, tt.b)
            if (err != nil) != tt.wantErr {
                t.Fatalf("unexpected error: %v", err)
            }
            if got != tt.want {
                t.Errorf("got %v, want %v", got, tt.want)
            }
        })
    }
}
```

### **What is the difference between** `t.Error` **and** `t.Fatal`**?**

- `t.Error` reports a failure and continues.
    
- `t.Fatal` reports a failure and stops the test immediately.
    

### **What are** `t.Helper()`**,** `t.Cleanup()`**,** `t.TempDir()`**,** `t.Setenv()`**?**

- `Helper` — marks a function as a helper so errors point to the caller.
    
- `Cleanup` — registers cleanup code.
    
- `TempDir` — a temporary directory removed after the test.
    
- `Setenv` — sets an environment variable restored after the test.
    

### **What is** `t.Parallel()`**?**

Marks a test to run in parallel with other parallel tests, which speeds up the suite and helps find races.

### **How do you mock dependencies in Go?**

Because interfaces are implicit, I write small hand-made fakes/stubs, or generate mocks with `gomock` / `mockery` / `moq`.

```
type fakeRepo struct{ user *User }

func (f fakeRepo) GetByID(ctx context.Context, id int64) (*User, error) {
    return f.user, nil
}
```

### **What is the difference between a stub, a fake, and a mock?**

- **Stub:** returns predefined answers.
    
- **Fake:** a simplified working implementation (in-memory repository).
    
- **Mock:** also verifies interactions (which methods were called and how).
    

### **How do you test HTTP handlers?**

With `net/http/httptest`.

```
req := httptest.NewRequest(http.MethodGet, "/users/1", nil)
rec := httptest.NewRecorder()

handler.ServeHTTP(rec, req)

if rec.Code != http.StatusOK {
    t.Fatalf("status = %d", rec.Code)
}
```

`httptest.NewServer` starts a real test server, useful for client tests.

### **How do you test code that uses a database?**

- Unit tests with fake repositories.
    
- Integration tests with a real database in a container (`testcontainers-go`) or `sqlmock` for simple cases.
    

### **What are** `testify` **and similar libraries?**

`stretchr/testify` provides `assert`, `require`, and `mock`. The standard library is sufficient, but testify makes assertions shorter. `go-cmp` is good for comparing complex structures.

### **What are benchmarks and fuzzing?**

- Benchmarks: `BenchmarkXxx(b *testing.B)`, run with `-bench`.
    
- Fuzzing (Go 1.18+): `FuzzXxx(f *testing.F)` generates random inputs to find crashes.
    

### **What is** `TestMain`**?**

A function that controls setup and teardown for all tests in a package.

### **What are** `testdata` **and golden files?**

`testdata` is a directory ignored by the Go toolchain and used for fixtures. Golden files store expected output to compare against.

### **How do you check coverage?**

```
go test -coverprofile=cover.out ./...
go tool cover -html=cover.out
```

Coverage is a signal, not a goal. I focus on meaningful behavior.

### **What is the testing pyramid?**

Many fast unit tests, fewer integration tests, and a small number of end-to-end tests.

### **What is TDD?**

Test-Driven Development: write a failing test, make it pass with minimal code, refactor.

### **Why run tests with** `-race`**?**

To detect data races automatically. It should be part of CI.

### **What is** `testing/synctest`**?**

A package (Go 1.25) for testing concurrent code with a fake clock, which makes time-dependent tests fast and deterministic.

## **17. Architecture and Senior-Level Concepts**

### **What is Clean / Hexagonal architecture?**

The domain and business logic are at the center and do not depend on frameworks, databases, or transport. Adapters (HTTP, DB, queues) depend on the core through interfaces.

### **What are typical layers in a Go service?**

- Transport (HTTP/gRPC handlers).
    
- Service / use case (business logic).
    
- Repository (data access).
    
- Domain (entities, value objects, errors).
    

### **What is a DTO?**

A data transfer object: a struct for moving data across a boundary (request/response models), separate from domain and database models.

### **What is the difference between Entity and Value Object?**

- Entity: has identity (`User` with ID).
    
- Value Object: defined by its values and immutable (`Money`, `Email`).
    

### **What is CQRS?**

Separating commands (change state) and queries (read state), sometimes with different models or databases.

### **What is Event Sourcing?**

Storing a sequence of events as the source of truth instead of the current state.

### **What is a monolith vs microservices?**

- Monolith: one deployable unit, simpler to develop and debug.
    
- Microservices: independent services, scalable and separately deployable, but with network complexity, distributed transactions, and operational cost.
    

I start with a modular monolith and split services when there is a clear reason.

### **How do services communicate?**

Synchronously (HTTP, gRPC) or asynchronously (Kafka, RabbitMQ, NATS).

### **What is eventual consistency?**

Data in different services becomes consistent after some time rather than immediately. It is a trade-off for availability and scalability.

### **What is the Saga pattern?**

A way to manage a distributed transaction as a sequence of local transactions with compensating actions on failure.

### **What is the Outbox pattern?**

Write the business data and an outgoing event in the same database transaction, then a separate process publishes the event. It guarantees no lost events.

### **How do you ensure messages are processed reliably?**

At-least-once delivery plus idempotent consumers, retries with backoff, and a dead-letter queue.

### **What is horizontal vs vertical scaling?**

- Vertical: a stronger machine.
    
- Horizontal: more instances behind a load balancer. Go services are usually stateless, so they scale horizontally well.
    

### **How do you make a service stateless?**

Store state in a database, cache, or queue; do not keep user sessions in process memory.

### **What caching strategies do you know?**

Cache-aside, write-through, write-behind, TTL, and invalidation. Problems to remember: cache stampede (use `singleflight`), stale data, and invalidation complexity.

### **What is** `singleflight`**?**

`golang.org/x/sync/singleflight` merges simultaneous identical calls into one, which protects the backend from stampedes.

### **What is a message queue used for?**

Decoupling, buffering load spikes, background processing, retries, and event-driven communication.

### **What is Kubernetes readiness/liveness?**

- Liveness: is the process alive (restart if not).
    
- Readiness: is the service ready for traffic.
    

### **What are the 12-factor app principles relevant for Go?**

Config in environment, stateless processes, logs to stdout, dependencies declared in `go.mod`, disposable processes with graceful shutdown, dev/prod parity.

### **How do you manage configuration?**

Environment variables, config files (YAML), and libraries such as `envconfig`, `viper`, or `koanf`, validated on startup so that the service fails fast.

### **What is a monorepo vs polyrepo?**

Monorepo keeps many services in one repository (shared code, atomic changes); polyrepo splits them (independence, clearer ownership). Go workspaces and modules support both.

### **What is the Go proverb list worth remembering?**

- Clear is better than clever.
    
- A little copying is better than a little dependency.
    
- Don't just check errors, handle them gracefully.
    
- Make the zero value useful.
    
- The bigger the interface, the weaker the abstraction.
    
- Concurrency is not parallelism.
    

## **18. Go Version Features**

### **What was added in Go 1.18?**

Generics, fuzzing, workspaces (`go.work`), `any` alias.

### **What was added in Go 1.19–1.20?**

`GOMEMLIMIT`, atomic types (`atomic.Int64`), `errors.Join`, multiple `%w` in `fmt.Errorf`, `context.WithCancelCause`.

### **What was added in Go 1.21?**

Built-ins `min`, `max`, `clear`; packages `slices`, `maps`, `cmp`; `log/slog`; `sync.OnceFunc`/`OnceValue`; `context.WithoutCancel`, `context.AfterFunc`.

### **What was added in Go 1.22?**

Per-iteration loop variables, `range` over integers, enhanced `ServeMux` (methods and wildcards), `math/rand/v2`, `slices.Concat`, `sql.Null[T]`.

### **What was added in Go 1.23?**

Range-over-func iterators and the `iter` package, `slices.Collect`, `slices.Sorted`, `maps.Keys` returning iterators, `unique` package, timer/ticker improvements (unreferenced timers are garbage collected).

### **What was added in Go 1.24?**

Generic type aliases, `os.Root` (safe filesystem access), `testing.B.Loop`, `weak` package, Swiss-table-based map implementation (faster maps), `omitzero` JSON tag option.

### **What was added in Go 1.25?**

`sync.WaitGroup.Go`, `testing/synctest`, container-aware `GOMAXPROCS`, experimental "Green Tea" garbage collector. Newer releases come every six months, so I check the official release notes for the latest changes.

### **What is a range-over-func iterator?**

A function that yields values to the loop body.

```
func Count(n int) iter.Seq[int] {
    return func(yield func(int) bool) {
        for i := 0; i < n; i++ {
            if !yield(i) {
                return
            }
        }
    }
}

for v := range Count(3) {
    fmt.Println(v)
}
```

### **What is** `omitzero`**?**

A JSON struct tag option (Go 1.24) that omits fields with zero values, including `time.Time`, which `omitempty` does not handle correctly.

### **What is** `math/rand/v2`**?**

A modernized random package with a cleaner API (`rand.IntN`, `rand.N`) and better generators. For security-sensitive values I still use `crypto/rand`.

## **19. Common Practical Interview Questions**

### **How do you debug a goroutine leak in production?**

I check `runtime.NumGoroutine()` over time, take a goroutine profile (`/debug/pprof/goroutine?debug=2`), find where many goroutines are blocked, and add context cancellation or timeouts.

### **How do you find a memory problem?**

I take heap profiles at different times with `pprof`, compare them (`-base`), look for growing allocations, and check goroutines, caches, and slices keeping large arrays.

### **How do you improve a slow endpoint?**

I measure first (tracing, logs, pprof), then check database queries (indexes, N+1), add caching or pagination, parallelize independent calls with `errgroup`, and re-measure.

### **How do you handle external API failures?**

Timeouts, retries with exponential backoff and jitter for idempotent calls, a circuit breaker, fallback behavior, and good logging/metrics.

### **How do you implement graceful shutdown?**

Catch `SIGTERM` with `signal.NotifyContext`, stop accepting new requests (`srv.Shutdown`), wait for in-flight work and background workers, close DB and queue connections, and exit within a deadline.

### **How do you prevent duplicate payment processing?**

Idempotency keys, a unique database constraint, transactions, and explicit state transitions (`pending → paid`).

### **How do you handle a high number of concurrent requests?**

Stateless service, bounded worker pools, connection pool limits, timeouts, rate limiting, caching, and horizontal scaling.

### **How do you structure a REST service in Go?**

`cmd/` for entry points, `internal/` with handler, service, repository, domain; dependencies wired in `main`; config from environment; migrations in a separate folder.

### **How do you share data between goroutines safely?**

Channels for ownership transfer and communication, mutexes for shared state, atomics for simple counters, and `go test -race` to verify.

### **How do you read a large file without loading it into memory?**

Stream it with `bufio.Scanner` or `bufio.Reader`, or `io.Copy`.

```
f, _ := os.Open("big.log")
defer f.Close()

sc := bufio.NewScanner(f)
for sc.Scan() {
    process(sc.Text())
}
```

### **How do you design a rate limiter?**

Token bucket (`x/time/rate`) per client key, stored in memory for one instance or in Redis for several; return `429` with a `Retry-After` header.

### **How do you write a concurrent cache?**

A map protected by `sync.RWMutex`, with TTL expiration, size limits, and `singleflight` for loading. For production, libraries such as `ristretto` or `bigcache`.

### **How do you handle configuration for different environments?**

Environment variables with defaults, validated at startup, secrets from a secret manager.

### **How do you manage database schema changes safely?**

Backward-compatible migrations: add the new column first, deploy code that works with both versions, migrate data, then remove the old column later.

### **How do you work with legacy code?**

Understand current behavior, add tests around it, refactor in small steps, and keep changes reviewable.

### **When would you use a channel and when a mutex?**

Channel for passing data and coordinating goroutines; mutex for protecting a shared structure with short critical sections.

### **When would you use generics?**

For reusable data structures and algorithms over many types where an interface would lose type safety or add overhead.

### **How do you ensure code quality in a team?**

`gofmt`, `golangci-lint`, code review, tests with `-race` in CI, clear package boundaries, and documentation comments on exported names.

### **How do you handle invalid user input?**

Validate at the boundary, return `400/422` with meaningful messages, never trust the client, and log without exposing sensitive data.

### **How do you propagate request IDs and tracing?**

Generate or read the ID in middleware, store it in `context`, add it to every log line, and pass it to downstream calls via headers (`traceparent` with OpenTelemetry).

## **20. Essential Go Functions and Packages to Remember**

### **Built-in functions**

|Function|Purpose|
|---|---|
|`len()`|Length of string, slice, map, array, channel|
|`cap()`|Capacity of slice, array, channel|
|`append()`|Add elements to a slice|
|`copy()`|Copy elements between slices|
|`make()`|Create slice, map, channel|
|`new()`|Allocate zeroed memory, return pointer|
|`delete()`|Remove a map key|
|`clear()`|Empty a map or zero a slice (1.21+)|
|`min()` / `max()`|Smallest / largest value (1.21+)|
|`close()`|Close a channel|
|`panic()` / `recover()`|Raise / catch a panic|
|`print()` / `println()`|Low-level output (debug only)|

### **Package** `fmt`

|Function|Purpose|
|---|---|
|`Println()` / `Printf()`|Print to stdout|
|`Sprintf()`|Format into a string|
|`Fprintf()`|Format into a writer|
|`Errorf()`|Create (wrapped) error with `%w`|
|`Sscanf()` / `Sscan()`|Parse from string|

Useful verbs: `%v` value, `%+v` struct with field names, `%#v` Go syntax, `%T` type, `%q` quoted string, `%d` integer, `%s` string, `%f` float, `%p` pointer.

### **Package** `strings`

|Function|Purpose|
|---|---|
|`Contains()`|Check substring|
|`HasPrefix()` / `HasSuffix()`|Prefix / suffix check|
|`Split()` / `Join()`|String ↔ slice|
|`Fields()`|Split by whitespace|
|`TrimSpace()` / `Trim()`|Remove whitespace / characters|
|`ToUpper()` / `ToLower()`|Change case|
|`Replace()` / `ReplaceAll()`|Replace occurrences|
|`Index()`|Find substring position|
|`Builder`|Efficient string building|
|`EqualFold()`|Case-insensitive compare|
|`CutPrefix()` / `Cut()`|Split around a separator|

### **Package** `strconv`

|Function|Purpose|
|---|---|
|`Atoi()` / `Itoa()`|String ↔ int|
|`ParseInt()` / `ParseFloat()` / `ParseBool()`|Parse with bit size|
|`FormatInt()`|Int to string in a base|
|`Quote()`|Quote a string|

### **Package** `slices` **and** `maps`

|Function|Purpose|
|---|---|
|`slices.Sort()` / `SortFunc()`|Sort|
|`slices.Contains()` / `Index()`|Search|
|`slices.Reverse()`|Reverse in place|
|`slices.Equal()`|Compare slices|
|`slices.Clone()`|Copy|
|`slices.Delete()` / `Insert()`|Remove / insert|
|`slices.BinarySearch()`|Search in a sorted slice|
|`slices.Max()` / `Min()`|Extreme values|
|`maps.Keys()` / `Values()`|Iterators over keys / values|
|`maps.Clone()` / `Copy()`|Copy maps|
|`maps.DeleteFunc()`|Delete by condition|

### **Package** `errors`

|Function|Purpose|
|---|---|
|`errors.New()`|Create an error|
|`errors.Is()`|Match a value in the chain|
|`errors.As()`|Match a type in the chain|
|`errors.Unwrap()`|Get the wrapped error|
|`errors.Join()`|Combine errors|

### **Package** `time`

|Function|Purpose|
|---|---|
|`time.Now()`|Current time|
|`time.Since()` / `Until()`|Duration from / to a time|
|`time.Sleep()`|Pause|
|`time.After()` / `NewTimer()`|Timer channel|
|`time.NewTicker()`|Repeating ticker|
|`t.Format()` / `time.Parse()`|Formatting with the reference date `2006-01-02 15:04:05`|
|`t.Add()` / `t.Sub()`|Time arithmetic|
|`time.Duration`|Time span (`5 * time.Second`)|

### **Package** `encoding/json`

|Function|Purpose|
|---|---|
|`json.Marshal()`|Value to JSON bytes|
|`json.Unmarshal()`|JSON bytes to value|
|`json.NewEncoder()` / `NewDecoder()`|Streaming|
|`json.MarshalIndent()`|Pretty JSON|
|`json.RawMessage`|Delay decoding|

### **Package** `io` **and** `os` **and** `bufio`

|Function|Purpose|
|---|---|
|`io.Copy()`|Copy reader to writer|
|`io.ReadAll()`|Read everything|
|`io.Reader` / `io.Writer` / `io.Closer`|Core interfaces|
|`io.EOF`|End of stream marker|
|`io.TeeReader()` / `MultiWriter()`|Duplicate streams|
|`os.ReadFile()` / `WriteFile()`|Whole-file read / write|
|`os.Open()` / `Create()`|Open / create a file|
|`os.Getenv()` / `LookupEnv()`|Environment variables|
|`os.Args`|Command-line arguments|
|`bufio.Scanner`|Line-by-line reading|
|`bufio.NewReader()` / `NewWriter()`|Buffered I/O|

### **Package** `sort` **and** `cmp`

|Function|Purpose|
|---|---|
|`sort.Slice()`|Sort by a less function|
|`sort.Ints()` / `Strings()`|Sort basic slices|
|`sort.Search()`|Binary search|
|`cmp.Compare()`|Three-way comparison|
|`cmp.Or()`|First non-zero value|

### **Package** `context`

|Function|Purpose|
|---|---|
|`context.Background()` / `TODO()`|Root contexts|
|`WithCancel()`|Manual cancellation|
|`WithTimeout()` / `WithDeadline()`|Time limit|
|`WithValue()`|Request-scoped value|
|`ctx.Done()` / `ctx.Err()`|Cancellation signal / reason|

### **Package** `sync`

|Type|Purpose|
|---|---|
|`Mutex` / `RWMutex`|Locks|
|`WaitGroup`|Wait for goroutines|
|`Once`|Run once|
|`Pool`|Object reuse|
|`Map`|Concurrent map for special cases|
|`Cond`|Condition variable|
|`atomic.Int64`, `atomic.Value`|Lock-free operations|

### **Package** `net/http`

|Item|Purpose|
|---|---|
|`http.Handler` / `HandlerFunc`|Handler abstraction|
|`http.ServeMux`|Router|
|`http.Server`|Server with timeouts, `Shutdown`|
|`http.Client`|Client (always set `Timeout`)|
|`http.NewRequestWithContext()`|Create a request bound to a context|
|`http.Error()`|Write an error response|
|`http.MaxBytesReader()`|Limit request body size|
|`httptest`|Test helpers|

### **Other useful packages**

|Package|Purpose|
|---|---|
|`log/slog`|Structured logging|
|`regexp`|Regular expressions|
|`path/filepath`|File path handling|
|`flag`|Command-line flags|
|`text/template`, `html/template`|Templates (use `html/template` for HTML)|
|`crypto/rand`, `crypto/sha256`, `crypto/hmac`|Cryptography|
|`encoding/base64`, `encoding/hex`|Encoding|
|`reflect`|Runtime reflection|
|`embed`|Embed files in the binary|
|`runtime/pprof`, `net/http/pprof`|Profiling|
|`testing`, `testing/iotest`, `testing/fstest`|Testing|

### **Popular third-party libraries**

|Library|Purpose|
|---|---|
|`chi`, `gin`, `echo`|HTTP routers/frameworks|
|`pgx`, `sqlx`, `sqlc`, `gorm`, `ent`|Database access|
|`golang-migrate`, `goose`|Migrations|
|`go-redis`|Redis client|
|`testify`, `gomock`, `testcontainers-go`|Testing|
|`zap`, `zerolog`|Fast structured logging|
|`viper`, `envconfig`|Configuration|
|`golang.org/x/sync`|`errgroup`, `singleflight`, `semaphore`|
|`golang.org/x/time/rate`|Rate limiting|
|`cobra`|CLI applications|
|`validator`|Struct validation|
|`prometheus/client_golang`, `OpenTelemetry`|Metrics and tracing|
|`grpc-go`|gRPC|
|`golang-jwt/jwt`|JWT|

## **21. Key Differences — Quick Revision**

|Concept A|Concept B|
|---|---|
|`var` — declaration, works at package level|`:=` — short declaration, only inside functions|
|`new(T)` — pointer to zero value|`make()` — initialized slice, map, channel|
|Array — fixed size, value type|Slice — dynamic view, header with pointer, len, cap|
|`len` — number of elements|`cap` — space until end of underlying array|
|`nil` slice — `null` in JSON|Empty slice — `[]` in JSON|
|Map — unordered, not concurrency-safe|Slice — ordered, indexed|
|`byte` — one byte (`uint8`)|`rune` — Unicode code point (`int32`)|
|Value receiver — works on a copy|Pointer receiver — can modify the original|
|Embedding — composition with promotion|Inheritance — not supported in Go|
|Interface — behavior contract, implicit|Struct — data and methods|
|`any` — no type safety|Generics — type-safe reuse|
|Type assertion — extract concrete type|Type switch — branch by dynamic type|
|`error` — expected failure|`panic` — bug or impossible state|
|`errors.Is` — compare value in chain|`errors.As` — find type in chain|
|Sentinel error — simple detection|Custom error type — carries details|
|`%w` — wraps an error|`%v` — only formats it|
|Goroutine — lightweight task|OS thread — heavy, managed by OS|
|Concurrency — structure of independent tasks|Parallelism — simultaneous execution|
|Unbuffered channel — synchronous handoff|Buffered channel — queue with capacity|
|Channel — communicate and transfer ownership|Mutex — protect shared state|
|`Mutex` — exclusive access|`RWMutex` — many readers or one writer|
|`sync.Map` — special cases|`map` + `RWMutex` — general case|
|`atomic` — simple counters and flags|`Mutex` — complex critical sections|
|`WaitGroup` — wait for completion|`errgroup` — wait and collect first error|
|`context.WithCancel` — manual stop|`context.WithTimeout` — stop after duration|
|`defer` — runs at function end, LIFO|`go` — runs concurrently|
|Stack — per goroutine, automatic|Heap — managed by the GC|
|`GOGC` — GC frequency|`GOMEMLIMIT` — soft memory cap|
|`init()` — implicit package setup|`main()` — program entry point|
|`text/template` — no escaping|`html/template` — context-aware escaping|
|`math/rand` — not secure|`crypto/rand` — secure random|
|Hashing — one-way|Encryption — reversible|
|Authentication — who you are|Authorization — what you may do|
|PUT — replace|PATCH — partial update|
|REST — resources over HTTP, JSON|gRPC — HTTP/2, Protobuf, streaming|
|Session — server-side state|JWT — stateless token|
|Unit test — isolated|Integration test — real components|
|Stub — predefined answer|Mock — verifies interactions|
|Fake — simplified working implementation|Mock — expectation-based|
|`t.Error` — continue test|`t.Fatal` — stop test|
|Table-driven test — many cases in one loop|Benchmark — measures performance|
|Offset pagination — simple, slow for large pages|Keyset pagination — stable and fast|
|Monolith — one deployable|Microservices — many deployables|
|Entity — identity matters|Value Object — values matter|
|Command — request an action|Event — something that happened|
|Vertical scaling — stronger server|Horizontal scaling — more instances|
|Shallow copy — nested references shared|Deep copy — nested data duplicated|
|`go.mod` — dependencies and version|`go.sum` — checksums|
|`internal/` — private to the module|`pkg/` — public convention|
|Liveness — is the process alive|Readiness — can it receive traffic|

## **Go vs PHP — Short Comparison**

This section helps when moving from PHP to Go.

|Topic|PHP|Go|
|---|---|---|
|Execution|Interpreted/bytecode on a request basis (PHP-FPM)|Compiled binary, long-running process|
|Typing|Dynamic with optional types|Static, strict|
|OOP|Classes, inheritance, traits|Structs, interfaces, embedding|
|Interfaces|Explicit `implements`|Implicit (structural)|
|Errors|Exceptions (`try/catch`)|Returned `error` values|
|Concurrency|Mostly one request per process; async via extensions/Swoole|Goroutines and channels built in|
|Memory between requests|Nothing shared by default|Shared in process, so synchronization is needed|
|Dependencies|Composer, `composer.json`|Go modules, `go.mod`|
|Deployment|PHP runtime + web server|Single binary|
|Null handling|`null`, `?->`, `??`|Zero values, `nil`, explicit checks|
|Arrays|One flexible `array` type|Array, slice, map are separate|
|Generics|Docblock-based (no native)|Native since 1.18|
|Autoload|PSR-4|Packages and imports|

## **Additional Interview Answer Patterns**

These are useful phrases for explaining your decisions during a technical interview.

### **How do you explain a technical decision?**

I chose this approach because it keeps the code simple, readable, and easy to test. It also avoids unnecessary coupling between packages.

### **How do you explain a trade-off?**

Every solution has trade-offs. I consider performance, complexity, maintainability, and project requirements before choosing an implementation.

### **How do you explain that you do not know something?**

I have not used this feature in depth yet, but I understand its purpose. I would check the official documentation, write a small test, and verify the behavior before using it in production.

### **How do you explain your debugging approach?**

I first reproduce the problem, check logs and metrics, and find where actual behavior differs from expected. Then I isolate the root cause, fix it, and add a regression test.

### **How do you explain your experience moving from PHP to Go?**

I come from a PHP background, so I am used to web development, databases, and API design. Go's static typing, explicit error handling, and built-in concurrency are different, but the core backend ideas — HTTP, SQL, testing, clean architecture — are the same. I am actively learning the Go idioms and the standard library.

### **How do you explain why Go is a good choice for a backend service?**

Go compiles to a single fast binary, has a strong standard library for HTTP and concurrency, uses little memory, starts quickly, and is easy to deploy in containers. Its simplicity also makes large codebases easier to maintain.

### **When would you not choose Go?**

For heavy UI work, quick prototypes where a dynamic language is faster to iterate, data science and ML-heavy tasks, or when the team and ecosystem are already strongly invested in another stack.

## **Final Revision Checklist**

Use this checklist to assess your interview readiness.

- Go types, zero values, `var` vs `:=`, `new` vs `make`, `iota`.
    
- Arrays, slices (len/cap, append, aliasing), maps, strings, runes.
    
- Functions, closures, `defer`, `panic`/`recover`, variadic functions.
    
- Structs, methods, value vs pointer receivers, embedding.
    
- Interfaces, implicit implementation, type assertions, nil interface gotcha.
    
- Generics, constraints, `comparable`, when to use them.
    
- Error handling: `error`, wrapping with `%w`, `errors.Is/As`, custom errors.
    
- Goroutines, channels, `select`, scheduler (GMP), deadlocks, leaks.
    
- `sync`: Mutex, RWMutex, WaitGroup, Once, Pool, atomic; `-race`.
    
- `context`: cancellation, timeouts, propagation.
    
- Concurrency patterns: worker pool, fan-in/fan-out, pipeline, `errgroup`.
    
- Garbage collector, escape analysis, `GOGC`, `GOMEMLIMIT`, `pprof`, benchmarks.
    
- Modules, `go.mod`, `internal/`, project layout, build and Docker.
    
- `database/sql`, connection pool, transactions, SQL injection, N+1, migrations.
    
- `net/http`, handlers, middleware, JSON, timeouts, graceful shutdown.
    
- REST, status codes, JWT, gRPC, idempotency.
    
- Security: bcrypt/argon2, `crypto/rand`, XSS, CSRF, SSRF, `govulncheck`.
    
- SOLID, dependency injection, small interfaces, "accept interfaces, return structs".
    
- Design patterns: functional options, factory, strategy, decorator, repository.
    
- Testing: table-driven tests, `httptest`, mocks/fakes, `-race`, fuzzing, benchmarks.
    
- Architecture: layers, Clean/Hexagonal, CQRS, Saga, Outbox, caching, queues.
    
- Go 1.18–1.25 features: generics, `slices`/`maps`, `slog`, loop variable change, iterators.
    
- Practical debugging, production troubleshooting, code review.
    

**Recommendation:** use this document as a theory reference, but practise explaining each topic aloud in English. For a Backend interview, being able to explain _why_ you chose a solution and show it in code is more valuable than memorizing definitions.
