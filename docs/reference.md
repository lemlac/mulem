# Mu Language Syntax and Semantics Reference

*Version 0.1 (Draft)*

Mu is a general-purpose, multi-paradigm programming language with Python-like syntax and significant whitespace. It targets Python developers who need C-level performance all within the same language. It is expression-oriented where possible and provides explicit control over evaluation strategy, memory model, and error handling. Another goal is to make it minimalist and have opinionated formatting to make it easier for both humans to read and LLMs to produce without over-consuming tokens.

Mu will be both a compiled and an interpreted language. Unlike Python, you won't need to use another language like C to increase performance. You can instead compile some of it and then call it as a shared library all within the same language. Some use cases for this are AI, systems programming, and game development.

---

## Lexical Conventions

- **Indentation**: Significant whitespace (4 spaces recommended). 
- **Statements**: Each statement is divided by new lines. Semi-colons (;) can also be used, but there must be another statement after it—i.e. no trailing semi-colons. Double semi-colon (`;;`) has a special meaning to end a block.
- **Comments**: Double dash (`--`) to end-of-line. Block comments start with `--[[` and end with `--]]`.
- **String literals**: `"..."` with interpolation with `{expr}` inside. To write a literal curly brace (`{`), escape it with a backslash `\{`. 

---

## Expressions vs Statements

Almost everything is an expression. Some statements can be either inline or block depending on the presence of a new-line. When mixing the two (i.e. a block statement within an inline statement), the `;;` symbol is required to exit block mode and return to inline mode. The last statement evaluated in a block is its value. 

## Comments

Double dash (`--`) is a comment. Double dash and double square bracket (`--[[`+`--]]`) is a multi-line comment.

```mu
-- This is a comment.   
--[[
This is a comment.
--]]
```

## Variable Declarations

Variables are declared with just the equals sign (=). Type is inferred, but can be declared with a colon (:).

```mu
a = 1
b: int = 1
```

Variables are immutable, but setting it again shadows it.

```mu
a = 1
a = 2
a = 3
```

You can also shadow a variable using its previous value.

```mu
i = 0
i = i + 1
i += 1 -- Same as above
```

Mutable variables are declared with `mu type`. Setting it will change the value instead of shadowing it.

```mu
x: mu int = 0
x = 1          -- x is mutated
```

You can also infer the type with `mu _ = _`:

```mu
mu x = 0
x = 1
```

Or declare the type and set it later:

```mu
x: mu int
doSomething()
x = 1
```

## Functions

Functions are declared with parentheses before the equals sign. These are pure functions, so mutable variables cannot be captured. The type of the parameters can be either explicitly typed or inferred based on usage.

```mu
add(a, b) = a + b
result = add(1, 2)
```

Functions can have multiple lines. The last line evaluated is the return value.

```mu
fib(n) = 
    if n < 1 then
        0
    else if n < 2 then
        1
    else
        fib(n - 1) + fib(n - 2)
```

### Lambda Functions

You can define a function within an expression with the pattern `(_) => _`. This is useful for passing functions to other functions. If the lambda function has multiple lines, it must terminate with a double semi-colon (`;;`).

```mu
map(array, func) = [for x in array then func(x)]
array0 = [1 2 3 4]
array1 = map(array0, (x) => x + 1)
array2 = map(array0, (x) =>
    if x < 2 then
        x - 1
    else
        x + 2
;;)
```

## Inline binding (`let` and `as`)

You can also bind variables within an expression using `let ... then` and `as`. `let` is used for a single expression, while `as` binds for the rest of the scope.

```mu
squared = let x = getSomething() then x * x

while next() as val != None then
    print "{val}"
```

---

## Types

- **Basic**: `name: type`
- **Function**: `(type, type): type`
- **Result / error**: `type!` or `type!error`
- **Inferred**: omit the annotation entirely

### Built-in Types

Built-in types include `int`, `float`, `char`, and `str`. 

```mu
myInt: int = 1234
myFloat: float = 12.34
myChar: char = 'a'
myStr: str = "Hello"
```

Strings can be formatted with curly braces (`{expr}`) in the string. Use a backslash to write a literal opening curly brace (`\{`).

```mu
name = "world"  
hello = "Hello, {world}!"
helloEscaped = "Hello, \{world}!"
```

You can also write multi-line strings in the following way:

1. Omit the closing quotation mark.
2. Add a quotation mark on the next line.
3. A line with a closing quotation ends the string.

Indentation within the string will start after the starting quotation mark on each line. If the next line starts with anything other than a quotation mark before the string is closed, then it's a syntax error.

```mu
myStr =
    "Hello.
    "This string has multiple lines.
    "But this is the last line."
```

Arrays are declared with the hash symbol (`#`). A number after the hash makes it a fixed length array. Arrays are fixed length by default, but this mainly matters for mutable arrays since immutable arrays can be shadowed with different type. Items in an array are separated with spaces or new lines. This helps keep the number of characters low when initializing arrays. If one of the items in an inline array is an expression, you must surround the expression in parentheses. The hash symbol is also used for accessing an array.

```mu
list: float#4 = [1 2 3 4]
compressedList = [(list#0 + list#1) (list#3 + list#4)]
```

To make chaining accesses easier, there's a special rule for square brackets: any operator `o` can be expressed with `a[o b]` which is the same as `((a) o (b))`.   

```mu
matrix = [
    [1 2 3 4]
    [5 6 7 8]
    [9 10 11 12]
    [13 14 15 16]
]
oneItem = matrix[#2][#1]   -- 3rd row, 2nd column
```

---

## Special Bindings (`::`)

Variable and functions primarily use the equals sign (`=`), but there's another type of binding used for constants, types, and procedures (another function type). This type of declaration is constant; in other words, they cannot be mutated or shadowed. 

### Constants

Putting a constant value after `::` creates a constant. This holds an unchangeable value that must be known at compile time. Its type is inferred. Explicit typing isn't necessary since it cannot be changed.

```mu
PI :: 3.1415926535
```

Not that constants are lazily evaluated. You can call a function in it, and it won't be evaluated until it's used.

```mu
SQRT_TWO :: sqrt(2)
value = SQRT_TWO + 1  -- Here sqrt(2) is calculated.
```

You can also bind a function to a constant. When calling it, it would be the same as defining it inline and then calling. This can be useful if you need to pass a function multiple times but don't want it to be outputed when compiled.

```mu
IDENTITY :: (x) => x
addOne :: (x) => x + 1
value = addOne(2) -- Same as ((x) => x + 1)(2), result is 3.
array = map([1 2 3 4], addOne)
```

### Alias

Assigning a type after `::` creates an alias

```mu
numberType :: int
```

You can also create aliases for basic product types (tuples) or sum times (unions).

```mu
tuple :: int, float, char 
alsoTuple :: (int, float, char) -- Optional parentheses.
namedTuple :: {count: int; scale: float; code: char}
union :: int | float | char
```

### Procedures (`proc`)

Another type of double-colon declaration is a `proc`, short for procedure. Unlike normal functions, procs are impure but don't return anything. They also don't use any parentheses, and you can use `out` on parameters instead of a return value.  Triple dash (---) is used to divide the parameters from the function body. Captured mutable variables need to be redeclared with the `inherit` keyword. This helps make sure that the proc is actually capturing a variable and declaring a new variable.

```mu
count: mu int = 0
myProc :: proc
    a: int
    b: int
    result: out int
    ---
    inherit count
    count += 1
    result = a + b + count

sum: mu int
myProc 1, 2, sum
```

You can also use `out` when calling a proc to declare an out parameter and input it at the same time. This is useful if you want it to be immutable.

```mu
myProc 1, 2, out sum
doSomethingWith(sum)
```

#### Parameter Bindings
There are different kinds of bindings for a proc's parameters.

1. One-way input binding (default) — the value is copied.
2. One-way output binding (`out`) — the value will be discarded and set to a new value within the scope.
3. One-way input binding with `ref` — the value is passed by reference but not changed.
4. Two-way binding (`ref`+`mu`) — the value is passed by reference and may be altered.

```mu
normalize_in_place :: proc
    v: ref mu Vector2
    ---
    normalized = normalize(v)
    v.x = normalized.x
    v.y = normalized.y

mu mutable_v = Vector2{ x: 5.0; y: 12.0 }
normalize_in_place mutable_v
```

### Structures (`struct`)

Structs are product types—or in other words—plain data containers. They cannot extend other structs, but can inherit members of other structs (see [Inheritance and Visibility](#Inheritance-and-Visibility)).

```mu
MyStruct :: struct
    name: str
    value: int
```

### Enumerables (`enum`)

Enums are sum types. They define a closed set of variants. Variants may carry data turning them into a tagged union.

```mu
MyEnum :: enum
    First
    Second(int)
    Third{val: int}
```

### Exceptions (`except`)

Exceptions are similar to enums but used for error handling. See "Error Handling" down below for more details.

```mu
MyException :: except
    OutOfBounds
    DivideByZero(int)
```

### Virtual Types (`virt`)

A `virt` is an abstract interface — a named contract with no data. It is equivalent to a trait or interface in other languages.

```mu
MyVirtual :: virt
    speak(self): str
```

### Implementing (`impl`)

Methods and trait implementations are added separately with `impl`:

```mu
impl MyStruct
    init(x: int, y: int): Self =
        MyStruct{ x: x; y: y }
```

To implement a virt to another type, you write `impl Type(Virt)` — following "type extends type" order and mirroring Python's `class Name(Super)` pattern.

```mu
impl MyStruct(MyVirtual)
    speak(self) =
        "I am a MyStruct {{ x={self.x}, y={self.y} }}"

impl MyEnum(MyVirtual)
    speak(self) =
        match self
        case First then
            "I am a MyEnum of First"
        case Second(x) then
            "I am a MyEnum of Second({x})"
        case Third{val} then
            "I am a MyEnum of Third {{ val={val} }}"
```

### Inline Functions / Macros / Generics

Adding a parameter before the double colon (`::`) turns in into a macro. 

Analogous to constant values, you can define an inline function by adding a parameter before the double colons. Like constants, they don't output a value in memory when compiled, useful for collecting repeated code. Unlike constant functions, they cannot be passed to another function. They are only for inserting an expression.

```mu
max(a, b) :: if a > b then a else b
min(a, b) :: if a < b then a else b
```

You can also have multi-line macros similar to functions. The last expression is the return value.

```mu
doSomethingComplicated(x) ::
	x = x + 1
	x = x / 2
	x * x

value = doSomethingComplicated(3)
```

Note that this is basically the same as this:

```mu
value = (let x = 3 then
	x = x + 1
	x = x / 2
	x * x
;;)
```

You can also pass a type back to make generic types.

```mu
Option(T) :: enum
	Some(T)
	None

maybeInt = Option(int).Some(1)
```

## Inheritance and Visibility

Even though structs cannot be extended the usual way, they can inherit from other structs using the `inherit` keyword. This works similar to importing. It marks members that map to members of another struct, making conversion possible. 

```mu
Vector2 :: struct
    x: float
    y: float

Vector3 :: struct
    inherit x, y from Vector2
    z: float

v3 = Vector3{
    x: 1.0
    y: 2.0
    z: 3.0
}

radius2d(v: Vector2) = sqrt(v.x*v.x+v.y*v.y)
print "{radius2d(v3}" -- This works because Vector3 inherits from Vector2.
```

When you inherit, you don't just pick out some members. The entire super type is inherited, but only some members are visible. 

All members of a type are public by default. When making a subtype, inherited members become private to the subtype unless explicitly redeclared with `inherit`. This encourages separating public and private data into distinct types rather than using access modifiers.

```mu
PrivateType :: struct
    val: int
    secret: int

PublicType :: struct
    inherit val from PrivateType  -- redeclared — remains public in MyOtherClass
    other: int
```

A subtype cannot accidentally expose or clash with a private inherited member because types only see members that have been explicitly declared within them. This mirrors the convention used for imports.

---

## Control Flow

All branching constructs (`if`, `match`, `for`, `while`, `do`+`until`, `try`, and `with`) share the same block / inline pattern:

```mu
-- Block form
keyword subject then
    body
keyword
    body

-- Inline expression form
keyword subject then expr
keyword expr
```

### if / else

Basic boolean branching.

```mu
x = if x > 0 then x else -x

if x > 0 then
    print "positive"
else
    print "non-positive"
```

### match / case

Enum/exception branching. Exhaustive by default. `case _ then` for the default case. The indentation of each `case` must be the same as the starting `match`.

```mu
match self
case First then
    print "First"
case Second(x)
    print "Second({x})"
case Third{val}
    print "Third {{ val={val} }}"

-- Inline form (line breaks ignored inside parentheses)
message = (match e
    case file.OpenError { filename } then "Open error: {filename}"
    case _ then "Unknown error")
```

### for

Iterates through an array.

```mu
new_list = [for x in list then x * 2]

for x in list then
    print(x)
```

### while

Repeats a block of code until the condition is true.

```mu
while cond then
    body
```

### do / until

`do` marks a block of code with its own scope that runs only once.

```mu
x = 0
do
   x = 1
   print "{x}"  -- 1
print "{x}"     -- 0
```

`until` repeats the previous expression until the condition is true.

```mu
mu count = 0
count += 1
until count >= 10
-- `count` is now 10 or more.
```

The two can be combined to create another type of loop.

```mu
do
    body
until cond
```

### try / except

Exceptionable types are unwrapped with a question mark within a try block. Function that return an exceptionable type act as a try block. If a function doesn't have a return type and a `?` is used, then a exceptionable type is inferred. Use of `?` outside of a try-like block is a syntax. Note that inline try expression don't use `?` because it expects an exceptionable type to be passed directly.

```mu
safeResult = try divide(1, 0) except _ then 0.0

try
    risky()?
except e then
    print "{e}"
```

### with

Automically cleans up certain objects. Use `as` to use the object within a scope.

```mu
try
	with file.open("a.txt")? as f, file.open("b.txt")? as g then
	    g.write(f.read()?)?
except _ then
    pass       -- Ignore all errors
```

### Closing Multiple Blocks with `;;`

Multiple blocks can be closed at once with `;;`. This can help with readability.

```mu
if file1.endswith(".txt") and file2.endswith(".txt") then
    with file.open(file1)? as f, file.open(file2)? as g then
        g.write(f.read()?)?
;; -- closes both `if` and `with`
```

---

## Error Handling

- `expr?` — propagate an error upward (Rust/Swift style).
- Errors are typed sum types; the compiler unions all possible error types from every `?` site in a function.
- `try` / `except` can be an expression or a block and must unify return types (like `if` / `else`).
- Pattern matching on errors works with `match` or `if case Pattern(x) = value`.

---

## Pattern Matching and Destructuring

- Full `match` / `case` (exhaustive unless `case _` is present).
- `if case Pattern(x) = value then` — like Rust's `if let`.
- Destructuring supports structs, tuples, enum variants, and wildcards (`_`).

---

## Importing and Modules

Use `import _ from _` to import something. You can give the import an alias with `as`. All imports must be implicitly declared—no "import *". This helps prevent naming conflicts and track where things are defined.

Modules are named with the keyword `mod` near the top before anything is defined. This is the name you'll use after `from` when importing. 

```mu
import thing from somewhere

mod myModule

addThing(x) = x + thing
```

Modules define how memory is handled with the `@memory` decorator. By default, modules use a garbage collector. Some options include `Collect(GC)` (default),  `Count(ARC)` (reference counting), `Borrow` (borrow checking), and `Manual`. `GC` and `ARC` represent the standard garbage collector and reference counter respectively, but others can be defined and used instead.

```mu
import memory, Count, ARC from std.mem

@memory(Count(ARC))
mod moduleThatUsesReferenceCounting
```

## Memory Models

Mu is multi-paradigm: different functions, structs, or modules can use different memory strategies in the same program. The model is controlled per-module via decorators. Boundary crossing between models follows FFI-like rules — automatic marshalling where possible, explicit escapes otherwise.

---

## Design Philosophy

- **Readability first** — Python-like syntax with significant whitespace and opinionated formatting.
- **Performance on demand** — start with GC; change to a lower level memory model where necessary.
- **Explicit but ergonomic** — `?` for errors, attributes for memory models, same keywords used between inline and block expressions.
- **Unified concepts** — `inherit` for both scope capture and member visibility; `::` for all top-level definitions.
- **Python-developer friendly** — gradual typing, familiar control flow, no second language or FFI layer required.

---

*This document captures the current state of the Mu design. The language is still evolving; future revisions will cover modules, async/concurrency, operator overloading, and full Python interop.*
