# Mulem Reference

*Version 0.1 (Draft)*

__The Mulem programming language__ is a general-purpose, expression-oriented language designed to balance conciseness and eloquence. It delivers highly readable syntax, robust safety mechanisms, granular execution control, and expressive data pipelining. Supporting both interpretation and compilation, Mulem is ideally suited for systems programming, AI, and game development.

Sample:

```mulem
where x: int

f(x) = x * x

f(2)   -- Value: 4
```

---

## Table of Contents

- __[Basics](#basics)__
- __[Assignment](#assignment)__
- __[Functions](#functions)__
- __[Constants](#constants)__
- __[Control Flow](#control-flow)__
- __[Types](#types)__
- __[Generics](#generics)__
- __[Operators](#operators)__
- __[Advanced](#advanced)__
- __[Code Sample](#putting-it-all-together)__
- __[Reserved Keywords](#reserved-keywords)__

---

## Basics

Comments are made with `--` or `{- -}`.

```mulem
-- Line Comment
{- Block Comment -}
{- {- Nested Comment -} -}
```

A program in Mulem is divided into expressions and sequences. Sequences can either be delimited with whitespace and semi-colons (`;`) or brackets and commas (`,`). The default is a whitespace sequence, where each line is an expression. Multiple expressions can be on one line separated by semi-colons (`;`).

```mulem
expr
expr

expr; expr
```

### Blocks

Certain keywords can start *blocks* if a line break follows them. This starts a whitespace sequence with a child scope. Indentation determines when a block ends. The last expression evaluated in a block is its value. Use `pass` to leave a block empty.

```mulem
do
    body

do
    pass
```

Keywords that can start blocks:

- `do`
- `then`
- `else`
- `try`
- `opt`
- `=` (assignment, functions, and patterns)

Some keywords at the end of lines start a **pattern sequence**. Each line start with `|` is a part of that sequence.

```mulem
match expr is
| Pattern(x) =
    body
```

Keywords that can start pattern sequences:

- `is`
- `catch`

### Dual Whitespace-Bracket System

Brackets sequences and whitespace sequences can be mixed. When a whitespace sequence is inside a bracket sequence, it and all child whitespace sequences will have a bracket parent. Find a comma `,` or closing bracket will end all whitespace sequences with a matching bracket parent.

```mulem
apiCall(\(result) =
    if result > 0 then
        print("Success! {result}")
    else
        print("Failure! {result}")
)
```

### Declarations

Variables are declared with just the equals sign (`=`). A type may be optionally added with a colon (`:`) after the variable name or inferred without it. This type of variable is **immutable.** Additional *assignments* to a variable **shadow** that variable. 

```mulem
a = 0
b: int = 1
a = 2      -- New `a`
b = 3      -- New `b`
```

Functions are declared with parentheses (`()`) before the equals sign (`=`). The return type can be inferred based on the return value, and the types of parameters may be inferred based on usage.

```mulem
f(x: int): int = x*x
g(x) = x*x*x
f(2)   -- Result: 4
g(2)   -- Result: 8
```

Assignment or functions can have multiple lines by adding a line-break after the equals sign (`=`).

```mulem
lunch =
    if getDayOfWeek() == "Tuesday" then
        "tacos"
    else
        "sandwich"
```

```mulem
isThirteen(x) =
    if x == 13 then
        return True
    False
```

[TOC](#table-of-contents)

---

## Assignment

Variables can be declared with the equals sign `=`. Type notation uses `: T =` but can be inferred.

```mulem
a = 0
b: int = 2
```

These are immutable variables. When the type is inferred, immutable variables can be shadowed with any type. Use `where` if you want to constrain a variable name to a particular type. This is not the same as mutating the variable. You create a new variable with the same name with each `=`, which is called **shadowing.**

```mulem
a = 1
a = 2
a = 'a'

where b: int    -- `b` can only be an `int` now.
b = 'b'         -- Error: `b` is constrained to type `int`, `char` found.
```

Adding a new line and indentation after the `=` starts a block. The last expression evaluated in the block is the value of that variable.

```mulem
lunch =
    if getDayOfWeek() == "Tuesday" then
        "tacos"
    else
        "sandwich"
```

Mutable variables are declared with the symbol `~` before the name. They must be set with the `:=` operator or any compound assignment operators such as `+=` or `-=`. `:=`​ and `=`​ are separated so that you don't accidentally mistype a variable name and create a new variable in scope. 

```mulem
~i: int = 0
i := 1
i += 1
i -= 1
```

The type of a mutable variable can be inferred. A reference can be declared with `ref` before the name. This is treated as an alias to the same spot in memory. Its mutability is carried over.

```mulem
~x = 0
ref xRef = x
xRef := 1
x        -- Value: 1
```

*For more info on `ref`, see [References](#references) down below.*

A variable can have its type constrained by declaring it with `where`. This will lock any variable with the same name to that type in any scope or child scope.

```mulem
where x: int

x = 0     -- OK!
x = 1.0   -- Error: `x` constrained to type `int`, found `float`
```

### Destructuring

Tuples can be destructured like this:

```mulem
(a, b) = (1, 2)        -- Positional tuple.
{x, y} = (x: 3, y: 4)  -- Named tuple.
```

When tuples are mixed, you can either use `(…) * {…}` or `{0 as x, …}`.

```mulem
tuple = (1, 2, x: 3, y: 4)
(a, b) * {x, y} = tuple
{0 as a, 1 as b, x, y} = tuple
```

Tuples can be spread into a function using the `..` prefix operator. Functions can have named parameters in the same manner as destructuring.

```mulem
add(a: int, b: int) * {x: int, y: int}: int =
    a + b + x + y

add(..tuple)              -- Result: 10
add(5, 6, x: 7, y: 8)    -- Result: 26
```

[TOC](#table-of-contents)

---

## Functions

Functions are declared by adding parameters before the equals sign `() = `. Functions declared this way are visible to other functions in the scope no matter what order. They can be overloaded with new declarations of the same name. Overloaded functions are dispatched to the scope that they're defined in.

```mulem
divide(a: int, b: int): int = a // b
divide(a: float, b: float): float = a / b

divide(5, 2)       -- Result: 2
divide(5.0, 2.0)   -- Result: 2.5
```

The parameters and return types can be inferred. Functions can have multiple lines by adding a line break after the `=` sign. The last expression evaluated is the implied return.

```mulem
isThirteen(x) =
    if x == 13 then
        return True
    False
```

```mulem
fib(n) = 
    if n < 1 then
        0
    else if n < 2 then
        1
    else
        fib(n - 1) + fib(n - 2)
```

Function overloading can lead to ambiguities, so you need to use `of` in order to resolve which definition you mean when you pass a function by value. *See [Untagged Unions](#untagged-unions) for how to use the keyword `of`.* The type notation of a function uses `->` between the parameters and the return type.

```mulem
add(a: int, b: int): int =
    a + b
add(a: float, b: float): float =
    a + b

withOneAndTwo(f: (int, int) -> int): int =
    f(1, 2)
withOneAndTwo(f: (float, float) -> float): float =
    f(1.0, 2.0)

withOneAndTwo(add)      -- Is this for ints or floats?
withOneAndTwo(add of (int, int) -> int)   -- Resolved.
```

### Capturing

Functions capture immutable variables automatically. Mutable variables must be captured with `@` in the function signature. If the function mutates the variable, then the variable should have `~` before the variable name.

```mulem
amount = 1   -- Automatically captured.
~count = 0   -- Must be explicitly captured.

increment() @ (~count): void =
    count += amount

getCount() @ (count): int =
    count

increment()      -- Result: 1
increment()      -- Result: 2
increment()      -- Result: 3
getCount()       -- Result:
3
```

### Function Pointers

Declaring a function and setting it to a function creates a **function pointer**. This holds a single function and is treated like a value. It can't be overloaded, but it doesn't require `of` to get a particular dispatch since it can only hold one function. Notice that there's a colon `:` before the parameter and the colon before the return type is replaced with `->`. This distinguishes declarations and definitions. We're saying this function pointer takes these arguments and types — parameter names omitted.

```mulem
addInt: (int, int) -> int = add
-- Or…
addInt = add of (int, int) -> int
```

These types of functions are analogous to regular assignment. They can either be immutable or mutable. Immutable function pointers can be shadowed, but they cannot be overloaded. You can pass them into other functions as values.

```mulem
addOne(x) = x + 1
addTwo(x) = x + 2

action = addOne
action = addTwo   -- Previous action is now shadowed.

array = map([1, 2, 3, 4], action)   -- Pass action as a value
```

Function pointers can also be mutable. You can set it to point to different functions or assigned a lambda function. Remember the colon before the parameter `: (T) -> T` signifies that we are not defining a function, only declaring a function pointer.

```mulem
~cb: (int) -> int
f1(x) = x + 1
f2(x) = x - 1
cb := f1
cb(1)     -- Result: 2
cb := f2
cb(1)     -- Result: 0
cb := \(x) = x * 2
cb(2)     -- Result: 4
```

### Lambda Functions (`\`)

The symbol `\` starts a **lambda function.** This creates a function pointer inside an expression. A name can optionally be given to create a self-reference inside the lambda function.

```mulem
apiCall(\(result) =
    if result > 0 then
        print("Success! {result}")
    else
        print("Failure! {result}")
)
```

```mulem
startCountdown(\count(n) =
    if n > 0 then
        print("{n}!")
        count(n - 1)
    else
        print("Go!")
, 10)
```

When you define a lambda function in an expression by itself, the function will be assigned to a variable with its name in that scope. In other words, saying `\name(x) = x` is the same as `name = \name(x) = x`.

```mulem
callback = \callback(result) =
    if result > 0 then
        print("Success! {result}")
    else
        print("Failure! {result}")

-- Or just:

\callback(result) =
    if result > 0 then
        print("Success! {result}")
    else
        print("Failure! {result}")

-- `callback` is a function pointer in this scope.

apiCall(callback)
```

[TOC](#table-of-contents)

---

## Constants

Constants are declared with `const`. This marks compile-time data, different from an immutable variable. It treats the expression as if it where a literal. The type can be inferred.

```mulem
const PI: float = 3.14159265
const NAMESPACE = "development"
```

Constants can have arguments like functions to make compile-time functions. 

```mulem
const MAX(a: int, b: int) =
    if a > b then a else b

MAX(5, 10)    -- Result: 10
MAX(7, 3)     -- Result: 7
```

[TOC](#table-of-contents)

---

## Control Flow

- __[`do`](#do)__ – Basic block
- __[`if` / `else`](#if--else)__ – Boolean branching
- __[`match` / `is`](#match--is)__ – Pattern matching
- __[`loop`](#loop)__ – Iteration
- __[`opt` / `else`](#opt--else)__ – Coalescing
- __[`try` / `catch`](#try--catch)__ – Error handling
- __[`return` + `raise`](#try--catch)__ – Exiting a function
- __[`defer`](#defer)__ – Clean-up

### `do`

```mulem
do expr; expr

do
    body

do.label
    body
    break.label
```

Creates a new scope. Its value is the last expression evaluated.

```mulem
x =
    do
        y = 1
        y + 1       -- block's value is 2
```

Any starting block can be given a label with `.label` after its starting keyword. Use this to call `break.` on a specific label. 

```mulem
do.label
    break.label
```

Inline `do` will start a sequence of expressions separated by semicolons `;` that ends at a new line or (when inside a bracket) at a comma or closing bracket. The last expression in that sequence is the value of that slot.

```mulem
(a, do b(); c, d)    -- Value: (a, c, d) with side effect b()
```

Adding `do` makes it's clear that `;` is connected to `do` and not the parentheses.

### `if` / `else`

```mulem
if cond then expr else expr

if cond then
    body
else
    expr
```

Basic Boolean branching.

```mulem
x = if x > 0 then x else -x

if x > 0 then
    print("positive")
else
    print("non-positive")
```

Use `and`/`or` to compare multiple booleans at once.

```mulem
a = True
b = False

if a and b then            -- True and False == False
    print("This will not print")
else if a or b then        -- True or False == True
    print("This will print")
```

### `match` / `is`

```mulem
match expr is (| ptrn(_) = expr | ptrn = expr | (_) = expr)

match expr is
| ptrn(_) =
    body
| ptrn =
    body
| (_) =
    body
```

Enum/error branching. Exhaustive by default. `| (_) =` for the default case.

```mulem
match expr is
| ptrn =
    body
| ptrn =
    body
| (_) =
    body

match expr is (| ptrn = expr | ptrn = expr | (_) = expr)
```

When inline, the patterns after `is` need to be in parentheses.

```mulem
-- Simple value mapping
color = match status is (
    | Ok  = "green"
    | Err = "red"
    | (_) =  "gray"
)

-- Inside another expression
result = process(data) |> match $ is (
    | Success(v) = v * 2
    | Failure(e) = do print("Failure: {e}"); 0
)
```

The patterns map to the type passed in after `match`, so you only need to reference the members of that type in each pattern.

```mulem
match choice is
| First =                  -- Each pattern can case starts its on block.
    print("First")         -- Ident for the new block.
| Second(x) =              -- Continue this for each case.
    print("Second({x})")   -- ……
| Third{val} =             -- ……
    print("Third \{ val={val} }")
                           -- All choices were exhausted, so no `| (_) =` is necessary.
```

```mulem
result = match x is (| Ptrn1 = 5 | Ptrn2 = 6 | (_) = 7)
--
message = match e is (| OpenError{filename} = "Open error: {filename}" | (_) = "Unknown error")
```

### `loop`

The universal loop keyword. All loop forms share the same `loop` keyword.

```mulem
-- Unconditional (break manually):
loop
    body
    break

loop.label
    body
    break.label

-- While condition is true:
loop cond then
    body

-- For-each:
loop x in expr then
    body

[..loop x in expr then x]

-- Do-until (runs at least once):
loop
    body
until cond
```

`else` after `loop cond` runs if the loop body never executed.

```mulem
loop False then
    pass
else
    print("Never ran")
```

Use `..`/`..=` with `loop x in` to loop over a range of integers.

```mulem
-- Count to 10
loop i in 1..=10 then
    print("{i}")
```

Steps may optionally be added to the loop's subject line. This is done by adding a semi-colon `;` between the subject and `then`. An expression after `;` will run at the end of each iteration of the loop. 

`loop [under this condition] ; [doing this every iteration] then`

```mulem
-- The Dangerous Way
~i = 1
loop i <= 100 then
    if i % 10 == 0 then
        print("{i}!!!")
        continue
        -- OOPS! We forgot to do `i += 1` before continuing. 
        -- Infinite loop on i = 10!
    print("{i}")
    i += 1

-- The Safe Way
~i = 1
loop i <= 100; i += 1 then
    if i % 10 == 0 then
        print("{i}!!!")
        continue    -- Automatically triggers `i += 1` before checking condition again
    print("{i}")
```

```mulem
-- Track index of `loop / in`
~idx = 0
loop item in inventory; idx += 1 then
    print("Slot {idx}: {item}")
```

Because `do` blocks isolate scopes and inline expressions sequence seamlessly, you can combine `do` and `loop` to create a traditional, strictly scoped counter loop without requiring a distinct `for` keyword:

```mulem
-- C-style for loop
do ~i = 1; loop i <= 100; i += 1 then
    if i % 10 == 0 then
        print("{i}!!!")
        continue
    print("{i}")

-- 'i' is automatically out of scope and cleaned up here
```

`;`​ means different things in different contexts. In a block, it separates expressions; after `do`​, it separates expression in an inline block; and after `loop`​, it separates the subject from steps. 

Note that even though it's possible to do a C-style for-loop, it's hard to read and preferable to do a `loop x in` with a range instead.

```mulem
-- Same thing but easier to read
loop i in 1..=100 then
    if i % 10 == 0 then
        print("{i}!!!")
        continue
    print("{i}")
```

Inlined `loop x in` returns a lazy iterator collected with `..`.

```mulem
doubled = [..loop x in list then x * 2]
```

Destructuring works in loop variables.

```mulem
loop (x, y, z) in listOfTuples then
    print("{x}, {y}, {z}")
```

Pattern matching works. All patterns must have fallbacks. *(See [Pattern Fallback](#pattern-fallbacks).)* This is because if you have an array/iterator of enums, it would be hard to determine if they're all a particular pattern. This ensures any mismatches are handled inside the loop. 

```mulem
loop Pattern(x?) in listOfPatterns then
    if x is Some(x) then
        print("Found match: {x}")
```

This can be combined with `opt` to automatically skip when there's a mismatch.

```mulem
loop Pattern(x?) in listOfPatterns then
    opt
        print("Found match: {x?}")
    else
        continue
```

#### `break` / `continue`

Both accept an optional label to target an outer loop.

```mulem
loop.outer x in 0..100 then
    loop y in 0..100 then
        if x * y >= 100 then
            continue.outer
        if x * y == 77 then
            break.outer
```

Pass a value to `break`, that becomes the value of the block, analogous to `return` for function.

```mulem
x = do.block
    if cond then
        break.block 5
    4

print("{x}")     -- Prints either "4" or "5"
```

### `opt` / `else`

```mulem
opt expr? else expr

opt
    body?
else
    expr
```

None-coalescing. Unwrap question types with `?` inside an `opt` block. If any `?` returns `None`, the block short-circuits.

```mulem
opt
    a = getA()?
    b = getB()?
    c = opt getC()? else 0     -- Fallback
    print("{a + b + c}")
else
    print("Didn't work")
```

Inline form.

```mulem
a = Some(10)
x = opt f(a?) else "fallback"
```

Using `?` inside a function automatically infers a question return type `T?`.

```mulem
addStuff(a: int, b: int): int? =
    x = getA()?
    y = getB()?
    Some(x + y)
```

Nested questions unwrap with multiple `??`:

```mulem
unnest(x: int??): int? = Some(x??)
```

Chain multiple `opt` / `else` together untill you get a fallback:

```mulem
getFirst(a: int, b: int, c: int): int =
    ( opt getA(a)? else
      opt getB(b)? else
      opt getC(c)? else
      0 )
```

Or use the `None`-coalescing operator (`?:`).

```mulem
getFirst(a: int, b: int, c: int): int =
    getA(a) ?: getB(b) ?: getC(c) ?: 0
```

Another example of a use for `opt`:

```mulem
crunchData(): int?!Err!CustomError =                                -- Multiple error types
    value: int? = someFunc()!
    -- Question to Exclamation
    data: int = opt value? else raise CustomError("Not found")      -- Exist function on fallback
    data
```

### `try` / `catch`

```mulem
try expr! catch expr

try expr! catch (| ptrn(_) = expr | ptrn = expr | (_) = expr)

try
    body!
catch
| ptrn(_) =
    body
| ptrn =
    body
| (_) =
    body
```

Error handling. Unwrap exclamation types with `!` inside a `try` block. Unhandled errors propagate upward. Like with `match … is`, inline `try … catch` needs parentheses around the pattern matching section after `catch`.

```mulem
try
    a = doSomething1(x)!
    b = doSomething2(a)!
    b
catch
| Err(e) =
    print("Error: {e}")
    0
```

```mulem
try
    data = riskyOperation()!
    data2 = anotherRisky()!
    final = process(data, data2)
    Ok(final)
catch
| IOError(e) =
    print("IO failed: {e}")
    Ok(defaultValue)
| ValidationError(e) =
    Err(e)
```

```mulem
result: int = try
    data = riskyOperation()!
    data2 = anotherRisky()!
    process(data, data2)
catch
| IOError(e) =
    print("IO failed: {e}")
    0              -- fallback value
| ValidationError(e) =
    return Err(e)   -- Escape function with error
```

Inline form. If you only have a wildcard case `(| (_) = x)`, you just write the value `x`.

```mulem
result = try divide(1, 0)! catch 0.0
```

If you plan to only fallback on any error, you can use the `!:` operator.

```mulem
result = divide(1, 0) !: 0.0
```

Using `!` inside a function automatically infers a exclamation return type `T!`.

```mulem
riskyFn(a: int): int! =
    b = step1(a)!
    c = step2(b)!
    Ok(c)
```

### `return`

__`return`__ – Exits out of a function. If a value is after it, that value is the return value, otherwise it's `void`. This must match the return type of the function. Last-line evaluation is still enabled by default.

```mulem
isThirteen(x) =
    if x == 13 then
        return True  -- Exits the function and returns true.
    False            -- Returns false.
```

### `defer`

Runs after a function done. For iterator functions, this is when the iterator was broken or exhausted. For asynchronous functions, this is when the asynchronous type is resolved or rejected. Each `defer` statement go in reverse order: *first-in last-out*. It can be one line `defer …` or a block `defer do`. Generally though it's just one line like `defer cleanUp()`. 

```mulem
deferPrint() =
    defer print("Last")
    print("First")
    defer print("Second to last")
    print("Second")
    defer print("Middle")
    print("Done")

deferPrint()
```

Prints:

```mulem
First
Second
Done
Middle
Second to last
Last
```

[TOC](#table-of-contents)

---

## Types

- __[Built-in Types](#built-in-types)__
- __[Primitives](#primitives):__
  - `: byte = 0y`
  - `: bool = False or True`
  - `: char = '\0'`
  - `: int = 0i`
  - `: uint = 0u`
  - `: float = 0.0`
- __[Strings](#strings):__
  - `: str` – immutable `char` array
  - `"Single Line String"`
  - `"""Multi-Line String"""`
  - `''Raw String''`
- __[Arrays](#arrays):__
  - `: [T]` – dynamic array
  - `: [N;T]` – fixed array
  - `: [[T]]` – 2D dynamic array
  - `: [N;[M;T]]` – 2D fixed array
- __[Dictionaries](#dictionaries):__
  - `: [K:T]` – `K` is key type, `T` is value type
- __[Tuples](#tuples):__
  - `: (T, U)` – Position tuple
  - `: {x: V}` – Named tuple
  - `: (T, U) * {x: V}` or `: {0: T, 1: U, x: V}` – Mixed tuple
- __[Pointers](#pointers-t):__
  - `: ptr` – Raw pointer
  - `: T^` – Typed pointer
- __[Questions and Exclamations](#questions-t-and-exclamations-te):__
  - `: T?` – `Some(T)` or `None`
  - `: T!E` – `Ok(T)` or `Err(E)`
  - `: T!` – Error type inferred
- __[Custom Types](#custom-types):__
  - `:: T` — Alias to another type
  - `:: *` — Structural data
  - `:: |` — Enumerable data / tagged unions
  - `:: +` – Untagged unions
  - `:: !` — Custom error types
  - `:: .` — Virtual interfaces

### Built-in Types

Some built-in types include `byte`, `int`, `uint`, `float`, `bool`, `char`, `str`, and `ptr`. Note that although built-in types use lowercase names, they are not *keywords*. This is just a naming convention. *It's recommended that users create custom types with capitalized names to differentiate from built-in types.*

```mulem
myByte: byte = 255y
myInt: int = -1234
myInt: uint = 5678
myFloat: float = 12.34
myBool: bool = True
myChar: char = 'a'
myStr: str = "Hello"
myPtr: ptr = ExternalLib.getSomething()
```

You can get the type of any variable with the compile-time function `typeof`. This fetches the type of that symbol at that point during compile time. *(See [Generics](#generics).)*

```mulem
x = 0
y: typeof[x] = 1    -- Ensures that x and y have the same type.
```

You can also get the default value of any type with the compile-time function `default`. The type needs to have a default value defined which is yet to be determined how, but they're already defined for basic types.

```mulem
x = default[byte]   -- Value: 0y
x = default[int]    -- Value: 0
x = default[float]  -- Value: 0.0
x = default[bool]   -- Value: False
x = default[char]   -- Value: '\0'
x = default[str]    -- Value: ""
x = default[ptr]    -- Value: Null
```

`default` can also infer the type. This can be useful in certain situations, like if you want to leave a function that returns something empty so that you can implement it later.

```mulem
implementLater(): int = default
```

You can also get the size of any type with the compile-time function `sizeof`. It returns a constant `uint` (unsigned integer) with the number of bytes of memory that type requires. The exact sizes of some types like `int` or `float` might vary, but you can rely on `byte` and `bool` being 1 byte each. The size of `ptr` depends on the pointer size of the system. 

```mulem
sizeOfByte = sizeof[byte]   -- Value: 1
sizeOfBool = sizeof[bool]   -- Value: 1
sizeOfPtr  = sizeof[ptr]    -- Value: 4 or 8
```

### Primitives

The following are types used for basic arithmetics such numbers are characters.

#### Booleans

`bool` is a built-in enum type with its only members being `False` and `True`. This means you can also pattern match with a bool, although it's recommended to use `if`/`else` instead. Enum-members are capitalized by convention, so that's why `True` and `False` are capitalized instead of being `true` and `false`. 

```mulem
match value is
| True =
    print("It's true!")
| False =
    print("It's false!")
```

These two are the same thing.

```mulem
if value then
    print("It's true!")
else
    print("It's false!")
```

#### Numbers

There are several number types. More may be added in the future, but for now we'll focus on the 3 main types. For any of these types, you can add a suffix to explicitly declare a number literal with a particular type. This one works for regular base 10 notation, not scientific notation or alternative base notation.

| Type    | Meaning               | Suffix | Examples                       |
|:--------|:-----------------------|:----|:---------------------------------|
| `int`   | signed integer         | `i` | `1`, `2i`, `10i`, `0xabcdef`     |
| `uint`  | unsigned integer       | `u` | `1`, `0u`, `5u`, `0x10ff`        |
| `float` | floating point number  | `f` | `1.0`, `1.5f`, `100f`, `2.0e100` |

A number literal starts with a digit `0123456789` followed by zero or more other digits, letters, or underscores `_`. All number literals are case-insensitive, so `1.0f == 1.0F`.

You can place underscores `_` anywhere in a number to break it up into segments. This doesn't change the value.

- Examples: `1_234`, `1_000_000`, `0b1111_0000`, `0xab_cd_ef`

The sign is considered an operator and not a part of the constant itself. This gets automatically calculated in constant expressions at compile-time so that it seems like it's a part of the constant.

- `- 1` → `minus` + `1` → `signflip(1)` → `-1`

The exception to this rule is in scientific notation: the sign after `e` is a part of the number itself. There must not be a space between `e` and the sign `-`/`+`. The sign is optional for positive exponent values.

- Examples: `2e-100`, `5.0e+76`, `6.8E-128`, `10e3`

A period in a place where a decimal is expected, then it's part of the number literal and not `.` for member/component access. This is space sensitive.

Leading zeros are allowed also and don't affect the value. Unless there's a base letter, it's still in base 10, unlike in most C languages where a leading 0 switches to base 8.

- Examples: `001`, `009`, `000100`, `000u`, `010.0f`

Changing the base involves adding a `0` + either the letter `b`, `o`, or `x` to the start of the number.

| Prefix | Base | Valid Digts                           |
|:------:|:----:|:-------------------------------------:|
| `0b`   | 2    | `01`                                  |
| `0o`   | 8    | `01234567`                            |
| `0x`   | 16   | `0123456789abcdef` (case insensitive) |

Specific sized number types can be specified with `[N]` after the each type.

- `int[8]` – 8-bit signed integer
- `int[16]` – 16-bit signed integer
- `int[32]` – 32-bit signed integer
- `int[64]` – 64-bit signed integer
- `uint[8]` – 8-bit unsigned integer
- `uint[16]` – 16-bit unsigned integer
- `uint[32]` – 32-bit unsigned integer
- `uint[64]` – 64-bit unsigned integer
- `float[32]` – 32-bit floating-point number
- `float[64]` – 64-bit floating-point number

#### Characters

Characters or `char` are written with apostrophes (`'…'`) *(also called single quotes).* You can do arithmetic on them like with numbers.

```mulem
a = 'a'
b = a + 1
print("{b}")   -- Print "b", the letter after 'a'
c = b + 1
print("{c}")   -- Print "c", the letter after 'b'
```

Characters within apostrophes or quotation marks can be escaped with a backslash `\`. Some letters have special values like `\t` for tabs, `\n` for new lines, etc. All the standard stuff you would expect from a modern language. 

```mulem
apostrophe = '\''
tab = '\t'
newLine = '\n'
nullChar = '\0'
unicode = '\uFFFF'
```

#### Bytes

Bytes `byte` are 8-bit unsigned integers. You can write a byte literal by putting `y` at the end of an integer. It must be in the range of 0 to 255 (inclusive).

```mulem
min = 0y
max = 255y
a = byte('a')
```

### Strings

Strings (`str`) are immutable 1D arrays of characters `char`. 

|    Form     | Purpose                                                                    |
|:-----------:|:---------------------------------------------------------------------------|
|    `"…"`    | Regular string with `{expr}` interpolation                                 |
|  `"""…"""`  | Multiline with interpolation; whitespace trimmed at closing `"""` position |
|   `''…''`   | Inline raw string, no interpolation or escaping                            |
|   `\\…`     | Multiline raw string, no interpolation or escaping                         |

Strings are marked with quotation marks (`"…"`) *(also called double quotes)* and can be formatted with curly braces (`{expr}`) in the string. Use a backslash to write a literal opening curly brace (`\{`). Note that string insertion and named tuples both use curly braces. This shouldn't be an issue though since one is inside strings and the other isn't. Inserted expressions are implicitly converted to strings, so using `str()` isn't necessary. This string is only allowed on a single-line, but literal-line characters can be inserted with `\n`.

```mulem
name = "world"  
hello = "Hello, {name}!"
helloEscaped = "Hello, \{name}!"
lines = "This \n string \n has \n linebreaks."
```

Subsequent string literals will automatically concatenate, and the `<>` operator can be used to concatenate non-literal strings.

```mulem
str1 = "This" " string"
str2 = " is broken"
str3 = str1 <> str2 <> " into multiple parts."
print(str3)
-- Prints "This string is broken into multiple parts."
```

Write multi-line strings with `"""` (3 quotation marks). A common issue in programming languages is how to fix the issue of leading whitespace in a multi-line string. Mulem uses significant whitespace, so unindenting the string wouldn't work. We don't want all the leading whitespace to be in the string, but how do we solve this? To fix this issue, whitespace gets trimmed at compile-time based on the positions of the last `"""`. Any spaces before it is automatically trimmed. Much like blocks, having too little indentation is a syntax error inside multi-line strings. This helps keep things readable and consistent and solves the whitespace issue inside strings.

```mulem
do
    myStr =
        """
        Hello.
        This string has multiple lines.
          This line will start with 2 spaces in the string.
        
        The lines above and below this are empty.
    
        One quotation mark is fine (").
        Two quotation marks are fine too ("").
        But three quotation marks like \""" need to be escaped.
        This is the last line because of the closing quotation marks below it.
        """
    
    print(myStr)
```

What it will print:

```mulem
Hello.
This string has multiple lines.
  This line will start with 2 spaces in the string.

The lines above and below this are empty.

One quotation mark is fine (").
Two quotation marks are fine too ("").
But three quotation marks like """ need to be escaped.
This is the last line because of the closing quotation marks below it.
```

Write a basic raw string with `''…''` (two apostrophes). Although apostrophes `'` are used for chars, an empty char isn't possible since the default char is written `'\0'` (null character). A common practice in programming languages uses the double quote `"` for formattable strings and the single quote mark `'` for raw strings, so it should be easy for any programmer to see the parallel. When you write `''`, every character after it **except for new-lines** is in the string until the closing `''`. Escaping with backslashes `\` and insertion with curly braces `{}` are disabled. This is for single-line raw strings only. If there's a line break before the closing `''`, then it's a syntax error.

```mulem
rawString = ''It's okay to put an apostrophe (') in the string.''
filePath = ''C:\files\on\windows.txt''
template = ''Insert here → {{variable}}''
```

Multi-line raw strings can be written with `\\` (2 backslashes). This works like line comments, escaping everything up to the line break into the string. Each consecutive line starting with `\\` is part of the same string and joined with line-breaks (`\n`).

```mulem
const multiline = 
   \\This is line 1.
   \\This is line 2.
   \\No need to escape \n or \"

print(multiline)
```

What it will print:

```
This is line 1.
This is line 2.
No need to escape \n or \"
```

### Arrays

Array types are declared with square brackets around their type (`[T]`). A number before a colon `:` makes it a fixed length array `[N;T]`. Arrays are statically sized when written `[N;T]`, while `[T]` is the dynamic form. Items are separated with commas (`,`). Index is done with the `^[]` and `.[]` operators. 

```mulem
list: [4;int] = [1, 2, 3, 4]
print("length of list: {len(list)}")
compressedList = [list^[0] + list^[1], list^[2] + list^[3]]
doubleArray: [3;[2;int]] = [[1, 2], [3, 4], [5, 6]]
item = doubleArray^[1]^[0]    -- The 2nd row, 1st column
item = doubleArray^[1,0]      -- Or separated with commas
print("{item}")               -- Prints "3"
```

`list^[i]` is the same as `(list+i)^` in Mulem like how `list[i]` means `*(list+i)` in C/C++.

If access to an index in an array cannot be guaranteed, you can use the safe access operator `.[]`. For an array of `[T]`, `.[]` will return type `T?` and `^[]` will return a type `T`.

```mulem
i = randInt()    -- Undeterministic number.
list.[i] ?: -1   -- Fallback to -1 if out of bounds.
```

In general, you'll mostly be using arrays by iterating or piping them. 

```mulem
loop x in list then
    print("{x}")
```

Use the spread operator `..` to spread an array into another array. This must be the first prefix operator in an expression, and it must be in a compatiable array or tuple literal. It always goes last in the slot's expression, so extra parentheses aren't necessary: `..a <> b` == `*(a <> b)`.

```mulem
a = [1, 2, 3]
b = [0, ..a, 4]               -- Value: [0, 1, 2, 3, 4]
c = a <> b                    -- Value: [1, 2, 3, 0, 1, 2, 3, 4]
d = [0, ..a <> b, 5, ..c, 6]  -- Value: [0, 1, 2, 3, 0, 1, 2, 3, 4, 5, 1, 2, 3, 0, 1, 2, 3, 4, 6]
```

The `<>` is used for concatenating two arrays, and its complement operators are `*>` *prepend* and `<*` *append.* For an array of type `[T]`, these take a type `T` on the side opposite of where they point. `*>` is right associative, allow you to chain multiple prepends onto one array. 

```mulem
1 *> 2 *> 3 *> [4]  -- Value: [1, 2, 3, 4]
[1] <* 2 <* 3 <* 4  -- Value: [1, 2, 3, 4]
1 *> [2] <* 3 <* 4  -- Value: [1, 2, 3, 4]
```

Splitting arrays is accomplished with destructuring, just like you can do with tuples.

```mulem
[head, ..tail] = [1, 2, 3, 4]
head    -- Value: 1
tail    -- Value: [2, 3, 4]
```

If you spread an array into a tuple, the type must be known and the tuple must have compatible components. Positional components will map to array indexes or iterator yields based on where the spread is placed inside the tuple. If the tuple runs out of space, the spread will be truncated. This works similar to variadic parameters in functions.

```mulem
ThreeInts :: (int, int, int)

list = [1, 2, 3]
a: ThreeInts = (..list)      -- Value: (1, 2, 3)
b: ThreeInts = (0, ..list)   -- Value: (0, 1, 2), truncated at the end
```

Tuples may also collect any remaining positional components into an array, just like variadic parameters in functions.

```mulem
TwoOrMoreInts :: (int, int, ..int)

list = [1, 2]
a: TwoOrMoreInts = (..list)              -- Value: (1, 2, [])
b: TwoOrMoreInts = (0, ..list)           -- Value: (0, 1, [2])
c: TwoOrMoreInts = (-1, 0, ..list)       -- Value: (-1, 0, [1, 2])
d: TwoOrMoreInts = (-2, -1, 0, ..list)   -- Value: (-2, -1, [0, 1, 2])
```

### Dictionaries

Dictionaries are a subtype of arrays. Instead of numbers, each item is given a **key.** A dictionary's type is the type of the value `V` and the type of the key `K` join with a colon `:` in between: `[K:V]`. This makes it semantically clear that they are a subtype of arrays. Dictionaries also use the same operator to access items. The type passed to the `^[]` or `.[]` operators must match the key type. Each key is marked with `[]:` in the array.

```mulem
dict: [float: float] = [
    [1.0f]: 10.0f,
    [1.5f]: 15.0f,
    [2.0f]: 20.0f,
]
dict^[1.5f]   -- Value is 15.0
```

If the key type is `str` and a key is a valid variable name, then the square brackets before `:` can be omitted. You can access it like a member with `.`, but this will return a question type `T?` like `.[]`.

```mulem
dict: [str:int] = [
    a: 1,
    b: 2,
    c: 3,
    ["invalid name"]: 127,
]
dict^["b"]    -- Value is 2
dict.["b"]    -- Value is Some(2)
dict.["b"]?   -- Value is 2
dict.b        -- Value is Some(2)
dict.b?       -- Value is 2
```

You can iterate through a dictioary like with arrays. This is the recommend way of using dictionaries. 

```mulem
loop (key, val) in dict then
    print("{key} = {val}")
```

### Tuples

Tuples use commas (`,`) to separate components for both **positional** (`()`) and **named** (`{}`) tuples. 

Product unions with the `*` operator can be used for both types and values. When combining two or more positional tuples, the positions of subsequent tuples get bumped up by the number of positions in the previous tuples, i.e. `(a, b) * (c, d)` becomes `(a, b, c, d)`. When you combine two or more named tuples, conflicting named parameters override each other with the last tuple taking priority—much like how shadowing works. So if you have `{x: 1} * {x: 2}`, the result is just `{x: 2}` since it overrides the `x` of the previous tuple. Positional tuples and named tuples can be combined together for example `(0, 1) * {x: 2}`. The shorthand for this is to write named parameters in a positional tuple like `(0, 1, x: 2)`. 

You can think of it like every tuple always having both dimensions, just with most slots empty:

| Type                     | Value          | Positional | Named  |
|:-------------------------|:---------------|:---------|:---------|
| `(int, int)`             | `(0, 1)`       | `(0, 1)` | `{}`     |
| `{x: int}`               | `(x: 2)`       | `()`     | `{x: 2}` |
| `(int, int) * {x: int}`  | `(0, 1, x: 2)` | `(0, 1)` | `{x: 2}` |
| `{x: int} * (int, int)`  | `(x: 2, 0, 1)` | `(0, 1)` | `{x: 2}` |

So `*` has different commutativity rules depending on what's being combined:

|      Combination  | Commutative? |        Rule                 |
|:-----------------------:|:---:|:-------------------------------|
| Positional * Positional | No  | Positions concatenate in order |
|      Named * Named      | No  | Conflicts resolve last-wins    |
| Positional * Named      | Yes | Orthogonal, no interaction     |

This makes the algebra quite principled. The only cases where order matters are also the cases where a conflict is actually possible — two positional slots or two named slots with the same key. When there's no possible conflict, order is irrelevant.

It also means the shorthand `(0, 1, x: 2)` isn't really special syntax. It's the natural representation of a tuple that has both dimensions populated.

Every opaque type by itself is its own tuple, so for example `char` and `(char)` are the same. This means that in the example, `int * float * char` is the equal to `(int, float, char)`.  Creating a product type of opaque types like primitives and enums coerce into a positional tuple, e.g. `int * float * char` becomes `(int, float, char)`. 

Combining empty tuples produces an empty tuple `() * () == ()`. The same is true for empty named tuples `{} * {} == {}`. This also means that empty positional tuples and empty named tuples are equivalent `() == {}`. Both tuples have zero dimensions in both positional and named components; therefore they are equivalent. Saying `() * {} * ()` or `{} * ()` and any combination of empty tuples all are the same type.

### Pointers `T^`

Raw pointers are type `ptr`. These cannot be dereferenced. They are ideally used for FFI to pass to functions of external libraries. You can also check if it's `Null`.

```mulem
o: ptr = externalLibrary.getObject()
if o =/= Null then
    externalLibrary.useObject(o)
else
    print("Initialization failed")
```

Typed pointers are type `T^`. They're are like references but dereferenced with the `^` postfix operator.

```mulem
~x = 0
xPtr: int^~ = @x
xPtr^ := 1
x             -- Value: 1
```

Safe pointers are made by wrapping `T^` or `T^~` in `Some`. If you pass in `Some(Null)`, it will be converted to `None`.

To allocate memory on the heap, use the `alloc[]` and `free[]` functions.

```mulem
Student ::
    * name: str
    * grade: char

student: Student^~ = alloc[ Student(name: "John", grade: 'A') ]?
defer free[student]

student^.name := "John Smith"
```

`alloc` will check the size of the type passed to it and allocate that much space, returning a `T^~?` (safe pointer). If successful, it will run the expression inside the square brackets and return `Some(T^~)`. If not, it will return `None`. Hence `?` after it to unwrap the return value. `T^~` can also be downgraded to `T^` if you don't plan to change the data.

`free` will free the memory to a pointer and convert it to `Null` even if it was declared immutable to prevent dangling pointers.

`defer` will run an expression at the end of a function.

This won't prevent all memory safety issues. Mulem isn't aiming to be the next Rust. Think of this memory model as more of an upgraded C — less need to do arithmetics like `malloc(N * sizeof(T))` or remembering to call `free(p)` before the end of a function.

By default, they're isn't an ownership model in Mulem. If you don't plan on using a pointer after getting one from a function, you can put `defer free[p]` on the next line.

```mulem
student = getStudent()
defer free[student]
-- Use student freely below.
```

**NOTE:** There is no `unsafe` block like in some languages. Programmers are responsible for assuring the safety of their own code.

### Questions `T?` and Exclamations `T!E`

These are Mulem's 2 built-in monadic types. 

- `T?` is a *question type:* it may contain a value or be `None`.
- `T!E` is a *exclamation type:* it may contain a value or an error of type `E`.

| Mulem's Type | Other Languages                  | In Plain English                                                             | Layers | Resolve Order             |
|:----------|:---------------------------------|:-----------------------------------------------------------------------------|-------:|:----------------------------|
| `T?`      | `Option<T>`                       | An option                                                                      |      1 | `?`                         |
| `T!E`     | `Result<T, E>`                   | A result with 1 possible error                                               |      1 | `!`                         |
| `T?!E`    | `Result<Option<T>, E>`            | A result with 1 possible error for an option                                    |      2 | `!` *then* `?`            |
| `T??`     | `Option<Option<T>>`                | An option of an option                                                           |      2 | `?` *then* `?`            |
| `T?!E!F`  | `Result<Option<T>, E\|F>`         | A result with 2 possible errors for an option                                  |      2 | `!` *then* `?`            |
| `T!E?`    | `Option<Result<T, E>>`            | An option of a result with 1 possible error                                   |      2 | `?` *then* `!`            |
| `T?!E?`   | `Option<Result<Option<T>, E>`      | An option of a result with 1 possible error for an option                       |      3 | `?` *then* `!` *then* `?` |
| `T!E?!F`  | `Result<Option<Result<T, E>>, F>` | A result with 1 possible error for an option of result with 1 possible error |      3 | `!` *then* `?` *then* `!` |

Think of each `?` or `!` on a *type* as a **layer.** When you call the same `?` or `!` in an *expression,* you **unwrap** that layer.

```mulem
x: int? = getSomeInt()    -- Get wrapped value.
y: int = x?               -- Unwrap the question.
                          -- Which is equivalent to…
y =
    match x is
    | Some(val) =
        val               -- Get the Some value.
    | None =
        return None       -- Exit block, return None if a function
```

```mulem
x: int!Err = getRiskyInt()   -- Get wrapped value.
y: int = x!                    -- Unwrap the exclamation
                               -- Which is equivalent to…
y =
    match x is
    | Ok(val) =
        val                    -- Get the Ok value.
    | Err(e) =
        return Err(e)          -- Exit block, return error if a function
```

For basic errors, use the built-in `Err` type. It optionally takes either a `string` (message) or an `int` named parameter `code:`, or both. If a message is missing, it will construct one based on the code, and if a code is missing, it will default to `-1`. 

```
Err()
Err("message")
Err(code: -1)
Err("message", code: -1)
```

## Custom Types

Custom types are made with `::`.

```mulem
MyStruct ::
    * name: str
    * value: int

MyEnum ::
    | First
    | Second(int)
    | Third{val: int}

MyUnion ::
    + int
    + float
    + char

MyInterface ::
    (self^).speak(): void

MyStruct <= MyInterface ::
    (self^).speak(): void = 
        print("I am {self^.name} and have {self^.value} dollars.")

object.:MyInterface.speak()
```

### Aliases

Assigning a type after `::` creates an alias. 

```mulem
numberType :: int
```

This alias is unique to the scope. Modifying it only affects the alias and not the original type. This prevents accidental conflictions between modules. *(See [Implementation](#implementing-impl).)*

You can also create aliases for basic product types or sum types.

```mulem
tuple        :: (int , float , char)                   -- Also called a "positional tuple".
namedTuple   :: {count: int, scale: float, code: char} -- Position not guaranteed.
mixedTuple   :: (int, float) * {code: char}            -- Has both positional and named components.
productUnion ::  int * float * char                    -- Is the size of all types combined.
sumUnion     ::  int + float + char                    -- Is the size of the largest type.
```

### Structural Types

Structs are product types—or in other words—plain data containers. They cannot extend other structs but can *embed* members of other structs. *(See [Embedding and Visibility](#embedding-and-visibility).)*

Being product types, structs are defined with `*` for each member. This is the same notation as a product type like `int * float * char`. Adding a name before a type like `* name: str` puts that item in a named member, otherwise it gets put into the next positional member like a tuple starting at `.0`. 

```mulem
MyStruct ::
    * name: str
    * value: int

MyStruct :: name: str * value: int
```

Instantiate a struct by calling it like a function. Each member is treated like a named argument.

```mulem
myObject = MyStruct(name: "Foobar", value: 1)
```

Structs are transparent. They can be destructured like named tuples. 

```mulem
TransparentThing ::
    * a: int
    * b: int

{a, b} = TransparentThing(a: 1, b: 2)
print("a: {a}, b: {b}")
```

Dereferencing is done with the `^` operator, so `*` can't be confused for a pointer. Lining up each member resembles a dotted list like in Markdown or YAML.

If a member doesn't have a name, then it gets put into the next positional member.

```
Mixed ::
    * int
    * name: str
    * float
```

`int` and `float` would be in positional members in this example, so the members are `.0: int`, `.name: str`, and `.1: float`.

### Enumerable Types

Enums are sum types. They define a closed set of variants. Variants may carry data turning them into a tagged union. 

```mulem
MyEnum ::
    | First
    | Second(int)
    | Third{val: int}

MyEnum :: First | Second(int) | Third{val: int}
```

Like structs, instantiate by calling the member like a function unless it doesn't carry any data.

```mulem
a = MyEnum.First
b = MyEnum.Second(2)
c = MyEnum.Third(val: 3)
```

When pattern match, the fill path to the type doesn't need to named on each case, only the name of each member. Use `_` while destructuring to discard the members data.

```mulem
match a is
| First =
    print("first!")
| Second(_) =
    print("second!")
| Third{_} =
    print("third!")
```

### Untagged Union Types

Untagged unions – also called *sum types* – can be defined with the `+` between two or more types. When picking members of a union, use `of` with the type. 

```mulem
sumUnion :: int + float + char

u: sumUnion = 1

u of int     -- Value is 1
u of float   -- Read binary representation of int 1 as if it were a float
u of char    -- Read binary representation of int 1 as if it were a, value is '\1'
```

This is similar to C unions where it doesn't do any conversion; it only reads whatever data is there with a different type. Unions will set overflow data to 0 so that if you set a small member and then read from a big member, you won't get undefined behavior. The zero-padding interacts with endianness in a way that's deterministic but platform-dependent. The behavior is always defined, just not always portable. 

```mulem
u: sumUnion = '\1'   -- char (1 byte), remaining bytes zeroed

-- Little-endian: memory is [0x01, 0x00, 0x00, 0x00]
u of int             -- Value: 1

-- Big-endian: memory is [0x01, 0x00, 0x00, 0x00]  (same bytes)
u of int             -- Value: 0x01000000 = 16777216
```

Overloaded functions are also a kind of untagged union, one that holds different function pointers instead of types. When called, they are automatically determined by the compiler based on its arguments, but there are times when this cannot be determined. Use `of` to pick out a certain definition of an overloaded function when this happens. *(See [Functions](#functions).)*

```mulem
withOneAndTwo(add of (int, int) -> int)
withOneAndTwo(add of (float, float) -> float)
```

`of` has a single consistent meaning of *give this specific type's interpretation of this thing that could be multiple types.* A coder who understands `of` on value unions will immediately understand what it means on overloaded functions, and vice versa.

- `u of int` — select the int interpretation from a value union
- `add of (int, int) -> int` — select the (int, int) -> int interpretation from a function union

This explains *why* overload resolution sometimes needs manual help — for the same reason you sometimes need to tell the compiler which union member you mean. 

### Error Types

Errors are bit like both structs and enums. Each error type represents a member of a potential **error tagged union** that's summed up per function with a exclamation type `T!E` return type. Every `try` / `catch` block matches patterns to the summed error tagged union in its block based on each `!` point. Exclamation types flatten, so `Exclamation[Exclamation[T, E], F]` would become `Exclamation[T, E|F]` where `E|F` is a tagged union of each possible error in that exclamation. Instantiation works the same as structs.

```mulem
OutOfBounds :: !void            -- No data.
ErrorMessage :: !(str)          -- Attach a position tuple.
DivideByZero :: !{value: int}   -- Attach named member
```

```mulem
divide(num: float, dem: float): float!DivideByZero =
    if dem == 0.0 then
        raise DivideByZero(val: num)     -- Causes branch in `try` block when called with `!`.
    num / dem

try
    divide(1, 0)!
catch
| DivideByZero{val} =
    print("Can't divide {val} by zero!")
```

Uncaught errors in a `try` / `catch` block are implicitly reraised. Each `catch` pattern removes a possible error from the exclamation type of that block. When all possible errors have a `catch` arm, the value of that block is automatically unwrapped so that `Exclamation[T,]` becomes just `T`. 

### Virtual Interfaces

Methods can be defined on any type using this notation.

```mulem
Type(self).method(args) =
    body
```

Before the method name is the self parameter. `self` can be any name. You can add `^`/`^~` to use a pointer to the self parameter instead of copying its value.

An interface defines a set of methods that other types can implement with `<=`.

```mulem
Speakable ::
    (self^).speak(): void

MyStruct <= Speakable ::
    (self^).speak() =
        print("My name is {self.name} and I have {self.value} dollars.")
```

If a type implements an inferface, it can either called with `.` or with `.:Interface.`.

```mulem
object.speak()               -- If object implements Speakable.
object.:Speakable.speak()    -- Full name of method.
```

[TOC](#table-of-contents)

---

## Generics

Adding a parameter before the double colon (`::`) turns it into a generic. Parameters are put in square brackets `[]` to distinguish them from regular functions which use parentheses `()`. The result is treated like a constant for run-time code. The parameters in `[]` can be inferred if it returns a function or type. In other words, `meta(_)` is equal to `meta[_](_)`. 

The syntax `[]` was chosen so that generic type inference will take precedence. `meta(a, b)` means to *call the instantiated function that `meta` returns with inferred types* whereas `meta[a, b]` means to *call the abstract function `meta` with these exact values.* This also makes it easy to distinguish actual function calls from macros/inlining. This removes the need for the more conventional arrow bracket `<>` syntax, which can get confusing. For example, in `f( g < a, b > ( c ) )`, is `g` a generic function or is that comparing two values and passing the results to `f`? The square bracket syntax removes this ambiguity, `f( g [ a, b ] ( c ) )`. This makes it semantically clear that you're doing a compile-time function call followed by a run-time function call. 

```mulem
-- Note that this is not the actual definition for a question type `T?`. This is just a user-defined enum that uses the same pattern.
Option[T] ::
    | Some(T)
    | None

Some[T] :: Option[T].Some
None[T] :: Option[T].None

optionInt = Some(1)
```

Type parameters can be omitted at the call site if they can be fully inferred from the value arguments, in which case the call uses only parentheses `()`. It can also be called explicitly by making an alias for it or calling with both brackets at the same time `[]()`

```mulem
SomeInt :: Some[int]
optionInt = SomeInt(1)

optionInt = Some[int](1)
```

Constraints can be made on parameters in the same manner as type notation for function parameters using `:`.

```mulem
List[T: type, N: const uint] ::
    * data [N;T]
```

An alternative method is to declare the parameters with `where` about the generics definition. In this example, `T` is set to a `type` for both `Option` and `List`:

```mulem
where T: type
where N: const uint

Option[T] ::
    | Some(T)
    | None

List[T, N] ::
    * data [N;T]
```

`where` is used for both type constraints and generic parameters constraints. They're treated the same. A constraint for a generic parameter is its type. A variable with type `type` is expecting type notation, and a variable with type `const T` is expecting a constant or literal of type `T`.

[TOC](#table-of-contents)

---

## Operators

| Level | Category                   | Operators                                       |
|:------|:---------------------------|:------------------------------------------------|
| 11    | Member access/Function     | `.` `^.` `.[]` `^[]` `?.` `?.[]`                |
| 10    | Postfix/Prefix             | `?` `!` `^` `@` `.:` `()`                       |
| 9     | Unary                      | `+` `-` `not`                                   |
| 8     | Exponent                   | `**` (right-associative)                        |
| 7     | Multiplicative / Shift     | `*` `/` `//` `%` `%%` `<*` `*>` `<<` `>>` `>>>` |
| 6     | Additive / Concat          | `+` `-` `<>` `&` `\|` `>\|`                     |
| 5     | Range                      | `..` `..=`                                      |
| 4     | Comparison                 | `==` `=/=` `<` `>` `<=` `>=`                    |
| 3     | Logical AND                | `and`                                           |
| 2     | Logical OR / None-Coalesce | `or` `?:` `!:`                                  |
| 1     | Pipeline                   | `\|>`                                           |
| 0     | Assignment / Spread        | `=` `:=` `+=` `-=` `..`                         |

| Operator       | Meaning                                              |
|:--------------:|:-----------------------------------------------------|
|   `lhs . rhs`  | Member access / safe pointer member access           |
|   `lhs .[rhs]` | Safe array/dictionary index                          |
|   `lhs ^[rhs]` | Raw array/dictionary index                           |
|   `lhs ?`      | Unwrap question, propagate `None` to nearest `opt` |
|  `lhs ?. rhs`  | Member access on a question type                     |
|  `lhs ?.[rhs]` | Safe array/dictionary index on a question type       |
|   `lhs !`      | Unwrap exclamation, propagate error to nearest `try` |
|   `lhs ^`      | Dereference typed pointer                            |
|   `lhs ^. rhs` | Raw pointer member access                            |
|       `@ rhs`  | Get reference                                        |
|      `.. rhs`  | Spread operator                                      |
|  `lhs .. rhs`  | Exclusive range                                       |
| `lhs ..= rhs`  | Inclusive range                                      |
| `lhs \|> rhs`  | Pipeline                                             |
|   `lhs + rhs`  | Addition                                             |
|   `lhs - rhs`  | Subtraction                                          |
|       `+ rhs`  | Unary positive                                       |
|       `- rhs`  | Sign flip                                            |
|   `lhs * rhs`  | Multiplication                                       |
|   `lhs / rhs`  | Exact division (float)                               |
|  `lhs // rhs`  | Floor division (int)                                 |
|   `lhs % rhs`  | Remainder, C-Style Modulo                            |
|  `lhs %% rhs`  | True Modulo                                          |
|  `lhs ** rhs`  | Exponentiation (right-associative)                   |
|  `lhs == rhs`  | Equality                                             |
| `lhs =/= rhs`  | Inequality                                           |
|   `lhs > rhs`  | Greater than                                         |
|   `lhs < rhs`  | Less than                                            |
|  `lhs >= rhs`  | Greater than or equal                                |
|  `lhs <= rhs`  | Less than or equal                                   |
|  `lhs <* rhs`  | Append to array                                      |
|  `lhs *> rhs`  | Prepend to array (right-associative)                 |
|  `lhs <> rhs`  | Concatenation                                        |
| `lhs and rhs`  | Logical AND                                          |
|  `lhs or rhs`  | Logical OR                                           |
|     `not rhs`  | Logical NOT / Bitwise NOT                            |
|  `lhs ?: rhs`  | `None`- coalescing                                   |
|  `lhs !: rhs`  | Error-coalescing                                     |
|   `lhs & rhs`  | Bitwise AND                                          |
|  `lhs \| rhs`  | Bitwise OR                                           |
| `lhs >\| rhs`  | Bitwise XOR                                          |
|  `lhs << rhs`  | Bitshift Left                                        |
|  `lhs >> rhs`  | Bitshift Right                                       |
| `lhs >>> rhs`  | Unsigned Bitshift Right                              |

__Compound Assignment Operators:__

| Operator        | Meaning              |
|:---------------:|:---------------------|
|  `lhs := rhs`   | *Assignment*         |
|  `lhs += rhs`   | `lhs := lhs + rhs`   |
|  `lhs -= rhs`   | `lhs := lhs - rhs`   |
|  `lhs *= rhs`   | `lhs := lhs * rhs`   |
|  `lhs /= rhs`   | `lhs := lhs / rhs`   |
| `lhs //= rhs`   | `lhs := lhs // rhs`  |
|  `lhs %= rhs`   | `lhs := lhs % rhs`   |
| `lhs %%= rhs`   | `lhs := lhs %% rhs`  |
| `lhs **= rhs`   | `lhs := lhs ** rhs`  |
| `lhs <*= rhs`   | `lhs := lhs <* rhs`  |
| `lhs *>= rhs`   | `lhs := rhs *> lhs`  |
| `lhs <>= rhs`   | `lhs := lhs <> rhs`  |
|  `lhs &= rhs`   | `lhs := lhs & rhs`   |
| `lhs \|= rhs`   | `lhs := lhs \| rhs`  |
| `lhs >\|= rhs`  | `lhs := lhs >\| rhs` |
| `lhs <<= rhs`   | `lhs := lhs << rhs`  |
| `lhs >>= rhs`   | `lhs := lhs >> rhs`  |
| `lhs >>>= rhs`  | `lhs := lhs >>> rhs` |

`=/=`​ was picked over `!=` because `!`​ is excusely used for error operators, and `/=`​ is the compound division operator, so that leaves `=/=​` as the most obvious symbol left for the not-equals operator. The symbol also benifits by making it easy to switch between `==` and `=/=` with adding or removing the slash `/`.

Chaining comparitive operators `>`/`<` works like in mathematics, a shorthand for `and` between operations. Each consecutive comparitive operator should point the same way, i.e. only `<` and `<=` or `>` and `>=`.

```mulem
(a < b <= c < d) == (a < b and b <= c and c < d)
(a > b >= c > d) == (a > b and b >= c and c > d)
```

### Key Type Modifiers & Postfix Operators

| Syntax         | Meaning            | Note                                                      |
|:---------------|:-------------------|:----------------------------------------------------------|
|  `T?`, `x?`    | Question           | Unwraps a question; propagates `None` to nearest `opt`. |
|  `T!`, `x!`    | Exclamation        | Unwraps a exclamation; propagates error to nearest `try`. |
|  `T^`, `x^`    | Pointer            | Dereference a pointer.                                    |
|  `@T`, `@x`    | Reference          | Get a reference to a place in memory.                     |

[TOC](#table-of-contents)

---

## Advanced

[TOC](#table-of-contents)

- __[Pipelining](#pipelining)__
- __[Optional Para\meters](#optional-parameters)__
- __[Lambda Functions](#lambda-functions)__
- __[Pattern Matching](#pattern-matching)__
- __[References](#references)__
- __[Iterator / Async Functions](#iterator-async-functions)__
- __[Embedding and Visibility](#embedding-and-visibility)__
- __[Manual Implementation](#manual-implementation)__
- __[Importing and Modules](#importing-and-modules)__

- __[Code Sample](#putting-it-all-together)__
- __[Reserved Keywords](#reserved-keywords)__

---

## Pipelining

Functions can get messy.

```mulem
print("{
    fetchC(
        fetchB(
            fetchA()
        )
    )
}")
```

To fix this, let's introduce a new system known as **pipelining.**

```mulem
fetchA()
|> fetchB(..$)
|> fetchC(..$)
|> print("{$}")
```

When placed at the start of a line, the pipelining operator `|>` takes on a special meaning: it takes the value of the previous line and makes it available in the next line as `$`. This symbol is known as the **pipeline context.** This makes it easy to chain a sequence of calls and read them in order. It reads like a plain-English list:

- `fetchA`
- *then* `fetchB`
- *then* `fetchC`
- *then* `print`

The first line of a block may also start with `|>`, in which case its pipeline context is an empty tuple `()`. A line starting with `|>` is not required to use the pipeline context. 

When mixed with `do`, multiple expressions separated by semicolons `;` on one line will share the same pipeline context. The last expression on the line is passed as the pipeline context to the next pipe.

```mulem
|> fetchA()                      -- Run fetchA,
|> do print("{$}"); fetchB(..$)   -- Print result, then fetchB
|> do print("{$}"); fetchC(..$)   -- Print result, then fetchC
|> print("{$}")                  -- Print result.
```

A **pipeline block** is started with `|> do` and a new line, either at the end of a line or on its own line. Within the block, `$` holds the piped-in value within the scope of that block.

```mulem
|> fetchA()         -- Set up things.
|> fetchB(..$)      -- …
|> fetchC(..$)      -- …
|> do               -- Pipeline context is now ready.
    print("{$}")    -- Use it here.

fetchA() |> fetchB(..$) |> fetchC(..$) |> do   -- Or in one line.
    print("{$}")                               -- Then use the result.

-- Freely mix the two formats:
fetchA() |> do         -- Start with this pipeline context.
    print("{$}")       -- Use the same `$` for these two lines.
    fetchB(..$)         -- Same pipeline context `$`.
    |> do print("{$}"); fetchC(..$) |> do  -- Start a new pipeline inline.
        print("{$}")                      -- Print the final result.
```

To get a value within a pipeline, use `as x` after any step to store it into a local variable. The assignment is written in reverse order — the variable name goes on the right.

```mulem
|> fetchA()
|> fetchB(..$)
|> fetchC(..$) as x    -- Put the result into `x`.

print("{x}")          -- Print the result.
```

This lets you extract the result of any step in a pipeline simply by appending `as name` to that line.

```mulem
-- Put all results of each step into variables.
|> fetchA() as a
|> fetchB(..$) as b
|> fetchC(..$) as c

print("a = {a}, b = {b}, c = {c}")
```

The variable type is always inferred, to avoid ambiguity with `:`. Mutability can be specified with `~`. *(See [Mutability](#mutability).)*

```mulem
fetchA() as ~x |> fetchB(x) |> do   -- Create a mutable variable `x`.
    x += 1                         -- Mutate it.
    print("{x}")                    -- Print it.
```

To put the final result of any pipeline into a variable, use `|> $ as x` at the end, where `x` is any variable name. This makes it easy to mix and match the pipes in between with `|> $ as x` at the end to collect it all into a variable. 

```mulem
|> fetchA()      -- Start.
|> fetchB(..$)    -- Pass pipeline context.
|> fetchC(..$)    -- Pass pipeline context.
|> $ as x        -- Put the pipeline context into `x`.

print("{x}")
```

One use case for a `|> do` block is to configure an object before assigning it to an immutable variable.

```mulem
user = User.create() |> do
    $.name = "John Smith"  -- Set properties on the pipeline context.
    $.dob = "1970-01-01"
    $                      -- Return the pipeline context.

print("User: {user.name}, born: {user.dob}")  -- Prints "User: John Smith, born: 1970-01-01"
```

This gives you a great deal of flexibility in how you choose to express your code.

[Advanced](#advanced) / [TOC](#table-of-contents)

---

## Optional Parameters

Parameters can have default values. Use `=` to to give an optional parameter a default value.

```mulem
addOptional(a = 0, b = 0): int = a + b     -- `a` and `b` are always `int`s

print("{addOptional()}")      -- Prints "0"
print("{addOptional(1)}")     -- Prints "1"
print("{addOptional(1, 1)}")  -- Prints "2"
```

Another option instead of giving a default value, you can it to a question type `T?` and default it to `None`.

```mulem
addOptional(a: int? = None, b: int? = None): int =
    aVal = a ?: 0    -- Coalesce optional arguments with default value 0.
    bVal = b ?: 0    -- This unwraps their value if they exist or set them to 0.
    aVal + bVal      -- Add the unwrapped values.

print("{addOptional()}")                  -- Prints "0"
print("{addOptional(Some(1))}")           -- Prints "1"
print("{addOptional(Some(1), Some(1))}")  -- Prints "2"
```

Use `..` to collect all arguments into a single variable. The variable should be type `[T]` (an array).

```mulem
addAll(..nums: [int]): int =
    ~sum: int = 0
    loop n in nums then
        sum += n
    sum

print("{addAll()}")         -- Prints "0"
print("{addAll(1)}")        -- Prints "1"
print("{addAll(1, 2)}")     -- Prints "3"
print("{addAll(1, 2, 3)}")  -- Prints "6"
```

```mulem
-- With pattern matching:
addAll(..nums: [int]): int =
    match nums is
    | []            = 0
    | [x]           = x
    | [x, ..rest]  = x + addAll(*rest)
```

A name is optional after `..`. You can use the symbol by itself to pass it to another function or itself in a functional loop. 

```mulem
addAll(x: int, ..): int =
    x + addAll(..)
addAll(x: int): int =
    x
addAll(): int =
    0

logAndAdd(msg: str, ..) =
    print("{msg} {addAll(..)}")

logAndAdd("Sum =")           -- Prints "Sum = 0"
logAndAdd("Sum =", 1)        -- Prints "Sum = 1"
logAndAdd("Sum =", 1, 2)     -- Prints "Sum = 3"
logAndAdd("Sum =", 1, 2, 3)  -- Prints "Sum = 6"
```

[Advanced](#advanced) / [TOC](#table-of-contents)

---

## Lambda Functions

Define a function within an expression with `\` + any name. For demonstration purposes, we'll be using anonymous functions in the pattern `\(x) = x`. This is useful for passing functions to other functions.

```mulem
(\(arg) = expr)

(\(arg) =
    body
)
```

```mulem
map(array, action) = [..loop x in array then action(x)]
array0 = [1, 2, 3, 4]
array1 = map(array0, \(x) = x + 1)   -- Inline
array2 = map(array0, \(x) =          -- Multi-line
    if x < 2 then
        x - 1
    else
        x + 2
)
```

A name is optional. Adding a name creates an immutable reference of the function itself.

```mulem
otherAction(\callback(val) =
    if val > 0 then
        print("{val}")
        callback(val - 1)
    else
        print("done")
)
```

Capturing also works inside lambda functions just like with named functions.

```mulem
~count = 0
forEach([1, 2, 3, 4], \(x) @ (~count) =
    count += x
)
```

### Curried functions

When a function returns another function, list each function parameters as the return type. Optionally, you can just let the return type be inferred.

```mulem
\curriedFn(a: int): (int) -> (int) -> int = _                 -- Immutable declaration
~curriedFnPtr: (int) -> (int) -> (int) -> int = curriedFn      -- Mutable declaration. 
```

The return type can be implied. Each function defined at the bottom is the implied return.

```mulem
curryFn(a: char) =
    print("In function 1: {a}")
    \(b: char) =
        print("In function 2: {b}")
        \(c: char) =
            print("In function 3: {c}")

curryFn('a')('b')('c')
{- Prints:
"In function 1: a"
"In function 2: b"
"In function 3: c"
-}
```

When capturing variables, each returned function needs to capture them separately.

```mulem
= 0
curryAddCount(a: int) @ (~count): (int) -> (int) -> int =
    count += a                                 -- (1) Evaluated immediately
    \(b: int) @ (~count): (int) -> int =       -- (2) Suspends and captures `count`
        count += b                             -- (3) Evaluated when second lambda is called
        \(c: int) @ (~count): int =
            count += c
            count

print("{ curryAddCount(1)(2)(3) } == { count }")   -- Prints "6 == 6"
```

[Advanced](#advanced) / [TOC](#table-of-contents)

---

## Pattern Matching

```mulem
match expr is
| Pattern1(x) = expr
| (_) = expr
```

The next control flow methods are based on pattern match. Generally, you see the word `is`, you next thing to expect after it is a pattern: `value is Pattern(x)`.

### Pattern Fallback

If a pattern can't be **guaranteed** for any reason, then you must have a **fallback.** There are two options available:

- __Optional binding:__ `Pattern(x?)` — wraps `x` in type `T?`, `Some(x)` if it matched, `None` if it didn't
- __Default value:__ `Pattern(x ?: default)` — `x` is type `T`, if it didn't match `x` is set to `default`

You can have multiple patterns match to one case. If any of the patterns destructure with a variable, the same variable name and type must be in all patterns. If not, use a wildcard `(_)` in each pattern or omit the data part entirely to disable destructuring. Otherwise, use a fallback in the pattern.

```mulem
match choice is
| First =
    print("First")
| Second(val) | Third{val} =             -- `val` must be in all patterns
    print("Second or Third, val={val}")
```

```mulem
-- `First` doesn't have any values, so destructuring must be disabled.
match choice is
| First | Second | Third =
    print("First, Second, or Third")
```

```mulem
-- Fallback, `val` is converted to question type `T?`:
match choice is
| First | Second(val?) | Third{val?} =
    print("First, Second, or Third: {opt val? else "None"}")
```

### Pattern Guards

Add `if` inside a pattern to conditionally match.

```mulem
match choice is
| Second(x if x > 0) =
    print("Positive: {x}")
| Second(x if x < 0) =
    print("Negative: {x}")
| Second(x) =
    print("Zero")
| (_) =
    print("No match")
```

### `fallthrough`

Proceeds to the next case, which must not destructure new values, unless fallbacks are used.

```mulem
match choice is
| First =
    print("First")
    fallthrough
| Second(x?) =              -- `?` in pattern wraps the variable in a question type
    if x is Some(x) then
        print("Definitely Second: {x}")
    -- Implicit break.
| (_) =
    print("No match")
```

`fallthrough`​ only goes to the next case. If the next case doesn't have a `fallthrough​` in it too, then it will break. It's a syntax error if you call `fallthrough` on the last case in the `match` block.

### `is` / `then`

```mulem
expr is ptrn then expr
```

Extract a single binding inline. Requires a guaranteed match or optional bindings.

```mulem
-- Guaranteed match (exhaustive type):
result = (value is Pattern(x) then x)
```

```mulem
-- With fallback (non-exhaustive):
result = (value is Pattern(x?) then x)                            -- Get wrapped Some(x) or None
result = (value is Pattern(x?) then opt x? else "fallback")       -- Unwrap with fallback
result = (value is Pattern(x ?: "fallback") then x)               -- Automatic fallback
```

```mulem
-- Multiple bindings:
result = (value is Pattern(x ?: 0, y ?: 0) then (x, y))
```

```mulem
-- Arbitrary expression over bindings:
result = (value is Pattern(x ?: 0, y ?: 0) then x + y)
```

Pairs naturally with pipelining.

```mulem
getValue()
|> ($ is Pattern(x ?: "fallback") then x)
|> doSomethingWith($)
```

### `if` + `is`

```mulem
if expr is ptrn then expr else expr
if expr is ptrn then
    body
else
    body
```

Combines the conditional branching of `if` with pattern matching of `is`. Useful if you want to destructure a single case of a sum type. This must be a pattern that matches the type of the value before `is`.

```mulem
if value is Pattern(x) then
    print("value is {x}")
else
    print("value doesn't match")

something = if value is Pattern(x) then x else "fallback"
```

Like with other patterns, multiple patterns can be checked for at once with `|`. All patterns must go on the right of `is`. Any destructured variables must match in name and type.

```mulem
if value is Pattern1(x: int) | Pattern2{data as x: int} then
    print("{x}")

if value is
| Pattern1(x: int)
| Pattern2{data as x: int} then
    print("{x}")
```

`x if` is also available like before and follows the same rules. You can iteratively match nested enums. 

```mulem
if nestedPattern is Pattern(Pattern(Pattern(Pattern(x if x >= 0)))) then
    print("Phew! That was a lot of unwrapping for {x}!")
else
    print("Either none of those nested patterns matched or x is negative.")
```

### `loop` + `is`

Loop while a pattern matches.

```mulem
loop nextValue() is Some(x) then
    print("{x}")
```

### `until` + `is`

Loop until a pattern matches. Bindings are in scope below the loop.

```mulem
~i = 0
loop
    print("Attempts: {i}")
    i += 1
until getValue() is Pattern(x)

print("{x}")    -- x is guaranteed set here.
```

If `break` is reachable inside the loop, optional bindings are required.

```mulem
loop
    if earlyCondition then
        break
until getValue() is Pattern(x?)

if x is Some(x) then
    print("{x}")
```

[Advanced](#advanced) / [TOC](#table-of-contents)

---

## References

`ref` gives you a reference whose mutability is determined by what you're binding to, not by how you use it. This is like a pointer but it auto derefs, no `^` operator needed.

```
~x = 5
ref y = x     -- y: int^~, x is mutable so ref inherits it

x2 = 5
ref y2 = x2   -- y2: int^, x2 is immutable
```

If the enum is immutable or mutable, then adding `@` on a pattern's data should match its mutability like when using `@` on another variable.

```mulem
~s = SomeStruct(value: 42)
match s is
| SomeStruct{ref value} =   -- value: int^~, s is mutable
    value := 100            -- modifies s.value in-place, no copy

s2 = SomeStruct(value: 42)
match s2 is
| SomeStruct{ref value} =   -- value: int^~, s2 is immutable
    print("{value}")        -- read-only, no copy
```

Function parameters/return types must use explicit pointers `T^`/`T^~` instead because `ref` does not work in contexts where the reference's life-time cannot be known. 

```mulem
getField(s: SomeStruct^): int^ = @(s^.value)

x = getField(@myStruct)   -- x: int^, immutable
```

```mulem
increment(x: int^~): void =
    x^ += 1

~x = 0
increment(@x)
x            -- Value is 1
```

```mulem
loop nextValue() is Some(x) then
    x^ := transform(x^)   -- mutates in place if iterator yields mutable refs `iter[T^~?]`
```

[Advanced](#advanced) / [TOC](#table-of-contents)

---

## Iterator / Async Functions

### `yield`

Exits out of a function with an `iter[_]` type. The return value of the function must be of type `iter[T]` where T is the yield type. When you have `yield` in your function, the actual return value in the function body is discarded, and using `return …` in it is a compile-time error. Use of `yield` will infer the return type to be `iter[_]`. 

```mulem
countUpTo(n: int): iter[int] =
    loop i in 0..n then
        yield i
```

If you use `yield`, you can only use a void `return` to exit the function. 

```mulem
countUntil(~i: int, max: int): iter[int] =
     loop
        if i >= max then
            return      -- Break out of the loop and the function.
        yield i
        i += 1
```

### `await`

Exits out of a function with an `async[_]` type. The return type of the function must be of type `async[T]` where T is the type that the asynchronous value will resolve in the end. The return value of the asynchronous instance is determined the same way that a non-asynchronous function does it. Use of `await` will infer the return type to be `async[_]`. 

```mulem
asyncFn(a, b): async[int] =
    a = await fetch(a)
    b = await fetch(b)
    a + b
```

Both `yield` and `await` can be used together in an `iter[async[_]]` type. The type suggests it—each yield is of type `async[_]`. Use `loop await` to wait for each async value to resolve in sequential order.

```mulem
asyncIterFn(n): iter[async[int]] =
    loop i in 0..n then
        val = await fetch(i)
        yield val

asyncCollect(n): async[[int]] =
    ~ret: [int] = []
    loop (await x) in asyncIterFn(n) then
        ret <>= x
    ret
```

Unlike in other languages where *promises* or *futures* can either resolve or reject, async types in Mulem **only resolve.** Instead you can use an exclamation type `T!E` inside an `async[T!E]` function. Unwrap it like you would an exclamation type. 

```mulem
(await x)!      -- Unwrap an `async[T!E]`
(await x)?      -- Unwrap an `async[T?]`
(await x)!?     -- Unwrap an `async[T?!E]`
```

[Advanced](#advanced) / [TOC](#table-of-contents)

---

## Embedding and Visibility

Even though structs cannot be extended the usual way, they can **embed** from other structs using destructuring. This works similarly to **importing.** It marks members that map to members of another struct, making conversion possible. It follows the same convention for pattern matching like destructuring. *(See [Destructuring](#destructuring).)* 

```mulem
Vector2 ::
    * x: float
    * y: float

Vector3 ::
    * {x, y}: Vector2        -- Grab members of Vector2
    * z: float

v3 = Vector3(x: 1.0, y: 2.0, z: 3.0)

radius2d(v: Vector2) = sqrt(v.x*v.x+v.y*v.y)
print("{radius2d(v3)}") -- This works because Vector3 embeds from Vector2.
```

When you embed, you don't just pick out some members. The entire parent struct exists in the child struct in memory, but only some members are visible. 

All members of a type are public by default. When making a subtype, embedded members become private to the subtype unless explicitly redeclared. This encourages separating public and private data into distinct types rather than using access modifiers.

```mulem
PrivateFields ::
    * val: int
    * secret: int

PublicFields ::
    * {val}: PrivateFields     -- Redeclared, `val` is public / `secret` is private
    * other: int
```

A subtype cannot accidentally expose or clash with a private embedded member because types only see members that have been explicitly declared within them. This mirrors the convention used for imports.

[Advanced](#advanced) / [TOC](#table-of-contents)

---

## Manual Implementation

Generics will automatically generate code based on their parameters, but you can also implement them by hand using pattern matching. If you only want to use the manual implementations for a generic function, you can set its body to `abstract[]`. This creates a virtual function that can be overloaded later. If you use a function that is defined with `abstract[]`, it will throw a compile-time error.

```mulem
-- Forces every type to have its own implementation
increment[T] :: (c: T^~): void = abstract[]

Counter :: *value: int

-- Specialized for Counter
increment[Counter] :: (c: Counter^~): void =
    c^.value += 1

-- Specialized for float
increment[float] :: (c: float^~): void =
    c^ += 1.0

~c = Counter(value: 2)
~f = 3.0
~b = True

increment(@c)   -- T is inferred Counter
increment(@f)   -- T is inferred float
{-
increment(@b)   -- T is inferred bool which has no implementation, compile-time error
-}
```

[Advanced](#advanced) / [TOC](#table-of-contents)

---

## Importing and Modules

* **Modules:** Declare with `mod ModuleName`. Only one per file.
* **Imports:** Must be explicit. `import a.b{c, d}`. Add `in "path"` for direct file imports.

Use `import` to import something, optionally giving the import an alias with `as`. You can either import a single export like `import a.b{c}` or multiple at once using destructuring rules `import a.b{c, d}`. *(See [Destructuring](#destructuring).)* This follows the same convention as destructuring with tuples. All imports must be **explicitly** declared—no `import a.b.*`. This helps prevent naming conflicts and track where things have been defined.

The most common import will likely be the `print` function, which will be defined somewhere in a standard library.

```mulem
import std{print}    -- This is just an example and not final.

print("Hello, world!")
```

Modules are named with the keyword `mod` near the top before anything is defined. This is the name you'll use when importing your module. **There can only be one `mod` declaration per file.** Multiple `mod` declarations is a syntax error. Imports are based on the include path when compiling or running a script. To import from a file by direct filepath, use `in "path"`.

```mulem
import someModule{thing} in "../../somewhere.mu"

mod myModule

addThing(x) = x + thing
```

In this example, you would import `addThing` like this (assuming the file is included):

```mulem
import myModule{addThing}
```

This connects the same explicit-list convention as embeding and capturing — *no hidden dependencies, everything that can affect behavior is named.* The three together form a consistent rule across the language.

### Memory Models

Mulem is multi-paradigm: different functions, structs, or modules can use different memory strategies in the same program. The model is controlled per-module. Boundary crossing between models follows FFI-like rules — automatic marshalling where possible, explicit escapes otherwise.

Modules define how memory is handled with the `memory` module setting. By default, modules use a garbage collector. Some options include `Collect(GC)` (default),  `Count(ARC)` (reference counting), and `Manual`. `GC` and `ARC` represent the standard garbage collector and reference counter respectively, but others can be defined and used instead.

```mulem
import std.mem{Count, ARC}

mod moduleThatUsesReferenceCounting {
    memory: Count[ARC],
}
```

[Advanced](#advanced) / [TOC](#table-of-contents)

---

## Putting It All Together…

Here is a quick synthesized example showing how Mulem's structs, implementations, pipelining, and error handling might look in a real script:

```mulem
import std{print}

mod exampleApp

{-
 - Define a struct and implement a prototype.
 -}
User ::
    * name: str
    * age: int

Speaker ::
    (self).speak(): str

User <= Speaker ::
    (self).speak() =
        "Hi, I'm {self.name}."

FetchError :: !(str)

{-
 - A function returning an exclamation type
 - (User or an Error).
 - It captures no outside state.
 -}
fetchUser(id: int): User!FetchError =
    if id > 0 then
        User(name: "Alice", age: 30)
    else
        FetchError("Invalid ID")

{-
 - Main logic demonstrating error unwrapping (!) 
 - and the pipeline operator (|>).
 -}
do
    try
        -- Fetch the user, unwrap the exclamation, and pipe it forward
        fetchUser(1)!
        |> do
            print($.speak())
            print("User is {$.age} years old.")
    catch
    | FetchError(e) =
        print("Failed to fetch user: {e}")
```

Here are some more examples:

```mulem
swap(x: int^~, y: int^~): void =
    if x =/= y then
        x^ := x^ >| y^
        y^ := x^ >| y^
        x^ := x^ >| y^
```

```mulem
rsqrt(number: float[32]): float[32] =
    ~i: int[32]
    ~x2: float[32]
    ~y: float[32] + int[32]
    threehalfs = 1.5f
    x2 := number * 0.5f
    y := number
    i := y of int
    i := 0x5f3759df - ( i >> 1 )
    y := i
    y := y of float * ( threehalfs - ( x2 * y of float * y of float ) )    -- 1st iteration
    -- y := y of float * ( threehalfs - ( x2 * y of float * y of float ) ) -- 2nd iteration, this can be removed
    y
```

[TOC](#table-of-contents)

---

## Reserved Keywords

Mulem has 28 reserved keywords. Note that built-in types (`int`, `str`), boolean values (`True`, `False`), standard question `T?` members (`Some`, `None`), are built-in symbols but *not* strict keywords.

**The Keyword List:**
`as`, `await`, `break`, `catch`, `const`, `continue`, `defer`, `do`, `else`, `fallthrough`, `if`, `import`, `in`, `is`, `loop`, `match`, `mod`, `of`, `opt`, `pass`, `raise`, `return`, `then`, `try`, `until`, `where`, `yield`.

[TOC](#table-of-contents)

---

*This document captures the current state of the Mulem design. The language is still evolving.*
