# Mulem Reference

*Version 0.1 (Draft)*

__The Mulem programming language__ is a general-purpose, expression-oriented language designed to balance conciseness and eloquence. It delivers highly readable syntax, robust safety mechanisms, granular execution control, and expressive data pipelining. Supporting both interpretation and compilation, Mulem is ideally suited for systems programming, AI, and game development.

---

## Table of Contents

- __[Basics](#basics)__
- __[Assignment](#assignment)__
- __[Functions](#functions)__
- __[Meta Assignment](#meta-assignment)__
- __[Control Flow](#control-flow)__
- __[Types](#types)__
- __[Meta Functions](#meta-functions)__
- __[Operators](#operators)__
- __[Advanced](#advanced)__
- __[Code Sample](#putting-it-all-together)__
- __[Reserved Keywords](#reserved-keywords)__

---

## Basics

Comments are made with two minus signs `--`.

```mulem
-- Single Line Comment
(-- Mulit-Line Comment --)
(-- (-- Nested Comment --) --)
```

### Dual Whitespace-Bracket System

Mulem is whitespace significant. Indentation determines when blocks start and end. Expressions are split by line-breaks and semi-colons (`;`). The value of each block is the last expression evaluated in it. 

```mulem
block
    expr

expr
expr

expr; expr
```

Certain keywords and symbols can start a block. Indentation can only increase if the previous line ends in a block-starting token. These tokens are:

- `=`/`:` (any assignment) 
- `do`
- `then`
- `else`
- `try`
- `maybe`

Some keywords start a pattern-matching sequence. When these words are at the end of a line, each line below it starting with a `|` at the same indentation will belong to the same sequence. These are the words:

- `is`
- `catch`

Whitespace becomes insignificant when inside brackets `()`/`[]`/`{}`. Slots in a bracket are delimited with commas `,`.

```mulem
(
    expr1_0 +
    expr1_1 +
    expr1_2,
    expr2_0
)
```

The two systems can mix: when a block starter is in a bracket, it can start a whitespace significant zone within the brackets. All sequences end when a closing bracket or comma is found.

```mulem
(do
    expr1
    expr2
    expr3, expr4)
```

In this example, the result is a tuple of `(expr3, expr4)`.

The benefit of this is that you can mix the two when you need them, such as passing a lambda function into a function.

```mulem
apiCall(_(result) =
    if result > 0 then
        print("Success! {result}")
    else
        print("Failure! {result}")
)
```

Whitespace is strict when it's significant. Expressions always end when there's a line break.

```mulem
x = 0
+ 1
+ 2
--  `x` is 0
```

To prevent this, you have some options. The easiest is to surround an expression in brackets.

```mulem
x = (0
+ 1
+ 2)
-- `x` is 3
```

The other option is to use backslashes (`\`), either at the end or beginning of a line. 

```mulem
x = 0 \
+ 1 \
+ 2

x = 0
\ + 1
\ + 2
```

This also applies to method chaining.

```mulem
(object.method1()
    .method2()
    .method3())
object.method1()
    \.method2()
    \.method3()
```

[TOC](#table-of-contents)

---

## Assignment

Variables can be declared with the equals sign `=`. Type notation uses `: T =` but can be inferred.

```mulem
a = 0
b: int = 2
```

These are immutable variables. When the type is inferred, immutable variables can be shadowed with any type.

```mulem
a = 1
a = 2
a = 'a'
```

Declare a variable with a type and it will be locked to that type until declared again with `: T`. Use `: *` to remove the locked type.

```mulem
c: char
c = 'c'
c = 0        -- Error!
c: int = 0   -- OK!
c: *
c = 'c'
c = 0        -- OK!
```

Adding a new line and indentation after the `=` starts a block. The last expression evaluated in the block is the value of that variable.

```mulem
lunch =
    if getDayOfWeek() == "Tuesday" then
        "tacos"
    else
        "sandwich"
```

Mutable variables are declared with the keyword `mu` before it. They must be set with the `:=` operator or any compound assignment operators such as `+:=` or `-:=`. Shadowing a mutable variable with `=` will throw an error unless redeclared with `: T` / `: *`. 

```mulem
mu i = 0
i := 1
i +:= 1
i -:= 1
i = 1   -- Error!
i: int  -- Shadow i
i = 1   -- OK!
```

`:=`​ and `=`​ are separated so that you don't accidentally mistype a variable name and create a new variable in scope. It also makes it easier to create compound assignment of custom operators such as `rem:=`​. 

### Destructuring

Tuples can be destructured like this:

```mulem
(a, b) = (1, 2)        -- Positional tuple.
{x, y} = (x: 3, y: 4)  -- Named tuple.
```

When tuples are mixed, you can either use `(…) & {…}` or `{0 as x, …}`.

```mulem
tuple = (1, 2, x: 3, y: 4)
(a, b) & {x, y} = tuple
{0 as a, 1 as b, x, y} = tuple
```

Tuples can be spread into a function using the `&` prefix operator. Functions can have named parameters in the same manner as desctructuring.

```mulem
add(a: int, b: int) & {x: int, y: int}: int =
    a + b + x + y

add(&tuple)              -- Result: 10
add(5, 6, x: 7, y: 8)    -- Result: 26
```

### References

A reference can be declared with `ref`. It points to the same memory location as another variable. Its mutability is carried over.

```mulem
mu x = 0
ref xRef = x
xRef := 1
x        -- Value: 1
```

Explicit reference types are `T%` (immutable) and `T%mu` (mutable). Use `%` and `%mu` prefix operators to get a memory address.

```mulem
mu x = 0
xRef: T% = %x
xRef := 1  -- Error!
xRef: T%mu = %mu x
xRef := 1  -- OK!
```

A `T%` can reference any variable of type `T`, but a `T%mu` can only reference mutable variables.

```mulem
x = 0
xRef: T% = %x         -- OK!
xRef: T%mu = %mu x    -- Error!
```

[TOC](#table-of-contents)

---

## Functions

Functions are declared using the keyword `fn`. Analogous to `mu`, this changes the behavior of subsequent declarations. Functions declared with `fn` are visible to other functions in the scope. They can be overloaded with new `fn` declarations of the same name.

```mulem
fn divide(a: int, b: int): int = a // b
fn divide(a: float, b: float: float = a / b

divide(5, 2)       -- Result: 2
divide(5.0, 2.0)   -- Reuslt: 2.5
```

The parameters and return types can be inferred. Functions can have multiple lines by adding a line break after the `=` sign. The last expression evalued is the implied return.

```mulem
fn isThirteen(x) =
    if x == 13 then
        return True
    False
```

```mulem
fn fib(n) = 
    if n < 1 then
        0
    else if n < 2 then
        1
    else
        fib(n - 1) + fib(n - 2)
```

Unlike normal assignment with just `=`, `fn` assignment returns the function that it creates. **Lambda functions** are created by defining a function inside an expression with `fn`. A name can optionally be given to create a self-reference inside the lambda function.

```mulem
apiCall(fn(result) =
    if result > 0 then
        print("Success! {result}")
    else
        print("Failure! {result}")
)
```

```
startCountdown(fn count(n) =
    if n > 0 then
        print("{n}!")
        count(n - 1)
    else
        print("Go!")
, 10)
```

### Function Pointers

Immutable function pointers are declared with parentheses before the equals sign. Parameter types and return type can be inferred.

```mulem
f(x: int): int = x*x
g(x) = x*x*x
g(2)    -- Result: 8
```

These types of functions are analogous to regular assignment. They are known as **function pointers** which can either be immutable or mutable. Immutable function pointers can be shadowed, but they cannot be overloaded. You can pass them into other functions as values.

```mulem
action(x) = x + 1
action(x) = x + 2   -- Previous action is now shadowed.

array = map([1, 2, 3, 4], action)   -- Pass action as a value
```

Function pointers have the same type strictness as immutable variables, so trying to overload a function pointer will result in an error.

```mulem
x: int = 5
x = "hello"  -- Error! Type mismatch

add(x: int) = x + 1       -- function pointer of type (int): int
add(x: float) = x + 1.0   -- Error! Same as assigning wrong type to a variable
```

Functions can also be mutable. You can set it to point to different functions.

```mulem
mu cb(int): int
f1(x) = x + 1
f2(x) = x - 1
cb := f1
cb(1)     -- Result: 2
cb := f2
cb(1)     -- Result: 0
```

Function pointers can be assigned a lambda function too.

```mulem
cb := fn(x) = x * 2
cb(2)     -- Result: 4
```

There are reasons to use both `fn` and just `=`. Sometimes overloading creates ambiguities. Function pointers are always exact.

```
fn add(a: int, b: int): int = a + b
fn add(a: float, b: float): float = a + b

fn withOneAndTwo(f(int, int): int): int = f(1, 2)
fn withOneAndTwo(f(float, float): float): float = f(1.0, 2.0)

withOneAndTwo(add)      -- Is this for ints or floats?

addInt(a: int, b: int): int = add(a, b)

withOneAndTwo(addInt)   -- Resolved.
```

`of` is another option. *See [Untagged Unions](#untagged-uni
ons).*

### Capturing

Functions capture immutable variables automatically. Mutable variables must be captured with `%` in the function signature. If the function mutates the variable, then it should have `mu` before the variable name.

```mulem
amount = 1     -- Automatically captured.
mu count = 0   -- Must be explicitly captured.

fn increment() % (mu count): void =
    count +:= amount

fn getCount() % (count): int =
    count

increment()      -- Result: 1
increment()      -- Result: 2
increment()      -- Result: 3
getCount()       -- Result: 3
```

[TOC](#table-of-contents)

---

## Meta Assignment

Meta assignments are made with `::`. These mark special data that tells the compiler the meaning of words and symbols. This includes things like constants, aliases, types, compile-time functions, and generics. This unifies many different concepts in programming without needing a lot of prefixed keywords, helping keep the name of assignments to the left. More on each of those later in this document.

To make a constant, you can replace `: T =` with `:: const T =`. This is different from an immutable variable. It treats the expression as if it where a literal. To infer the type, drop the `const T` so you just have `:: =`.

```
PI :: const float = 3.14159265
NAMESPACE :: = "development"
```

Constants can have arguments like functions to make inline functions. These use the square brackets `[]` instead of parentheses `()`. *See [Meta Functions](#meta-functions) for more details.*

```
MAX[a: int, b: int] :: const int =
    if a > b then a else b

MAX[5, 10]    -- Result: 10
MAX[7, 3]     -- Result: 7
```

[TOC](#table-of-contents)

---

## Control Flow

- __[`do`](#do)__ – Basic block
- __[`if` / `else`](#if--else)__ – Boolean branching
- __[`match` / `is`](#match--is)__ – Pattern matching
- __[`loop`](#loop)__ – Iteration
- __[`maybe` / `else`](#maybe--else)__ – Coalescing
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
(a, do b(); c, d)    -- == (a, c, d) with side effect b()
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

Use `or`/`and` to compare multiple booleans at once.

```mulem
a = True
b = False

if a and b then           -- True and False == False
    print("This will not print")
else if a or b then       -- True or False == True
    print("This will print")
```

### `match` / `is`

```mulem
match expr is (| ptrn(*) = expr | ptrn = expr | (*) = expr)

match expr is
| ptrn(*) =
    body
| ptrn =
    body
| (*) =
    body
```

Enum/error branching. Exhaustive by default. `| (*) =` for the default case.

```mulem
match expr is
| ptrn =
    body
| ptrn =
    body
| (*) =
    body

match expr is (| ptrn = expr | ptrn = expr | (*) = expr)
```

When inline, the patterns after `is` need to be in parentheses.

```mulem
-- Simple value mapping
color = match status is (
    | Ok  = "green"
    | Err = "red"
    | (*) =  "gray"
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
                           -- All choices were exhausted, so no `| (*) =` is necessary.
```

```mulem
result = match x is (| Ptrn1 = 5 | Ptrn2 = 6 | (*) = 7)
--
message = match e is (| OpenError{filename} = "Open error: {filename}" | (*) = "Unknown error")
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

[*loop x in expr then x]

-- Do-until (runs at least once):
loop
    body
until cond
```

`else` after `loop cond` runs if the loop body never executed.

```mulem
loop False then
    void
else
    print("Never ran")
```

When the parser encounters `loop`, it enters a *loop subject state.* Semicolons encountered in this state do not terminate the statement; they serve as delimiters for the step expressions. This state remains active until the parser matches the loop's body terminator (`then` or `until`).

When you have one or more semicolons `;` in the subject of a `loop` before `then`, the first in the sequence will be the loop's subject field (such as a condition or `in` expression) and the other expressions will run at the end of each iteration of the loop. 

```mulem
-- The Dangerous Way
mu i = 1
loop i <= 100 then
    if i rem 10 == 0 then
        print("{i}!!!")
        continue
        -- OOPS! We forgot to do `i +:= 1` before continuing. 
        -- Infinite loop on i = 10!
    print("{i}")
    i +:= 1

-- The Safe Way
mu i = 1
loop i <= 100; i +:= 1 then
    if i rem 10 == 0 then
        print("{i}!!!")
        continue    -- Automatically triggers `i +:= 1` before checking condition again
    print("{i}")
```

```mulem
-- Track index of `loop / in`
mu idx = 0
loop item in inventory; idx +:= 1 then
    print("Slot {idx}: {item}")
```

Because `do` blocks isolate scopes and inline expressions sequence seamlessly, you can combine `do` and `loop` to create a traditional, strictly scoped counter loop without requiring a distinct `for` keyword:

```mulem
-- C-style for loop
do mu i = 1; loop i <= 100; i +:= 1 then
    if i rem 10 == 0 then
        print("{i}!!!")
        continue
    print("{i}")

-- 'i' is automatically out of scope and cleaned up here
```

Inlined `loop x in` returns a lazy iterator collected with `*`.

```mulem
doubled = [*loop x in list then x * 2]
```

Destructuring works in loop variables.

```mulem
loop (x, y, z) in listOfTuples then
    print("{x}, {y}, {z}")
```

Pattern matching works. All patterns must have fallbacks. *(See [Pattern Fallback](#pattern-fallbacks).)* This is because if you have an array/iterator of enums, it would be hard to determine if they're all a particular pattern. This ensures any mismatches are handled inside the loop. 

```mulem
loop Pattern(opt x) in listOfPatterns then
    if x is Some(x) then
        print("Found match: {x}")
```

This can be combined with `maybe` to automatically skip when there's a mismatch.

```mulem
loop Pattern(opt x) in listOfPatterns then
    maybe print("Found match: {x?}")
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

### `maybe` / `else`

```mulem
maybe expr? else expr

maybe
    body?
else
    expr
```

None-coalescing. Unwrap question types with `?` inside an `maybe` block. If any `?` returns `None`, the block short-circuits.

```mulem
maybe
    a = getA()?
    b = getB()?
    c = maybe getC()? else 0     -- Fallback
    print("{a + b + c}")
else
    print("Didn't work")
```

Inline form.

```mulem
a = Some(10)
x = maybe f(a?) else "fallback"
```

Using `?` inside a function automatically infers a question return type `T?`.

```mulem
fn addStuff(a: int, b: int): int? =
    x = getA()?
    y = getB()?
    x + y
```

Nested questions unwrap with multiple `??`:

```mulem
fn unnest(x: int??): int? = x??
```

Chain multiple `maybe` / `else` together untill you get a fallback:

```mulem
fn getFirst(a: int, b: int, c: int): int =
    \ maybe getA(a)? else
    \ maybe getB(b)? else
    \ maybe getC(c)? else
    \ 0
```

Or use the `None`-coalescing operator (`?:`).

```mulem
fn getFirst(a: int, b: int, c: int): int =
    getA(a) ?: getB(b) ?: getC(c) ?: 0
```

Another example of a use for `maybe`:

```mulem
fn crunchData(): int?!Error!CustomError =                                -- Multiple error types
    value: int? = someFunc()!
    -- Question to Exclamation
    data: int = maybe value? else raise CustomError("Not found")      -- Exist function on fallback
    data
```

### `try` / `catch`

```mulem
try expr! catch expr

try expr! catch (| ptrn(*) = expr | ptrn = expr | (*) = expr)

try
    body!
catch
| ptrn(*) =
    body
| ptrn =
    body
| (*) =
    body
```

Error handling. Unwrap exclamation types with `!` inside a `try` block. Unhandled errors propagate upward. Like with `match … is`, inline `try … catch` needs parentheses around the pattern matching section after `catch`.

```mulem
try
    a = doSomething1(x)!
    b = doSomething2(a)!
    b
catch
| Error(e) =
    print("Error: {e}")
    0
```

```mulem
try
    data = riskyOperation()!
    data2 = anotherRisky()!
    final = process(data, data2)
    final
catch
| IOError(e) =
    print("IO failed: {e}")
    defaultValue
| ValidationError(e) =
    raise Err(e)
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
    raise Err(e)   -- Escape function with error
```

Inline form. If you only have a whildcard case `(| (*) = x)`, you just write the value `x`.

```mulem
result = try divide(1, 0)! catch 0.0
```

Using `!` inside a function automatically infers a exclamation return type `T!`.

```mulem
fn riskyFn(a: int): int! =
    b = step1(a)!
    c = step2(b)!
    c
```

### `return` + `raise`

__`return`__ – Exits out of a function. If a value is after it, that value is the return value, otherwise it's `void`. This must match the return type of the function. Last-line evaluation is still enabled by default.

```mulem
fn isThirteen(x) =
    if x == 13 then
        return True  -- Exits the function and returns true.
    False            -- Returns false.
```

__`raise`__ – Return out of the function with an error value. The function must return a exclamation type `T!`. If declared `T!E` where `E` is an `error` type, then the type passed to `raise` must match.

```mulem
-- T!E is inferred:
fn alwaysFail() =
    raise MyError("error message")

try
    alwaysFail()!
catch
| (e) =                -- Catch all errors.
    print("Error: {e}")
```

### `defer`

Runs after a function done. For iterator functions, this is when the iterator was broken or exhausted. For asynchronous functions, this is when the asynchronous type is resolved or rejected. Each `defer` statement go in reverse order: *first-in last-out*. It can be one line `defer …` or a block `defer do`. Generally though it's just one line like `defer cleanUp()`. 

```mulem
fn deferPrint() =
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
  - `: [*T]` – dyanmic array
  - `: [N*T]` – fixed array
  - `: [**T]` – 2D dyanmic array
  - `: [N*M*T]` – 2D fixed array
- __[Dictionaries](#dictionaries):__
  - `: [K:T]` – `K` is key type, `T` is value type
- __[Tuples](#tuples):__
  - `: (T, U)` – Position tuple
  - `: {x: V}` – Named tuple
  - `: (T, U) & {x: V}` or `: {0: T, 1: U, x: V}` – Mixed tuple
- __[Pointers](#pointers-t):__
  - `: ptr` – Raw pointer
  - `: T^` – Typed pointer
- __[Questions and Exclamations](#questions-t-and-exclamations-te):__
  - `: T?` – `Some(T)` or `None`
  - `: T!E` – `Ok(T)` or `Err(E)`
  - `: T!` – Error type inferred
- __[Custom Types](#custom-types):__
  - `:: _` — Alias to another type
  - `:: struct` — Structural data
  - `:: enum` — Sum types / tagged unions
  - `:: error` — Custom error types
  - `:: proto` — Virtual interfaces

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

You can get the type of any variable with the compile-time function `typeof`. This fetches the type of that symbol at that point during compile time. *(See [Meta Functions](#meta-functions).)*

```mulem
x = 0
y: typeof[x] = 1    -- Ensures that x and y have the same type.
```

You can also get the default value of any type with the compile-time function `default`. The type needs to have a default value defined which is yet to be determined how, but they're already defined for basic types.

```mulem
x = default[byte]   -- == 0y
x = default[int]    -- == 0
x = default[float]  -- == 0.0
x = default[bool]   -- == False
x = default[char]   -- == '\0'
x = default[str]    -- == ""
x = default[ptr]    -- == Null
```

`default[]` can also infer the type. This can be useful in certain situations, like if you want to leave a function that returns something empty so that you can implement it later.

```mulem
implementLater(): int = default[]
```

You can also get the size of any type with the compile-time function `sizeof`. It returns a constant `uint` (unsigned integer) with the number of bytes of memory that type requires. The exact sizes of some types like `int` or `float` might vary, but you can rely on `byte` and `bool` being 1 byte each. There's also the `void` type which represents no data. `ptr` depends on the pointer size of the system. 

```mulem
sizeOfByte = sizeof[byte]   -- == 1
sizeOfBool = sizeof[bool]   -- == 1
sizeOfVoid = sizeof[void]   -- == 0
sizeOfPtr  = sizeof[ptr]    -- == 4 or 8
```

### Primitives

The following are types used for basic arithmetics such numbers are characters.

#### Booleans

`bool` is a built-in enum type with its only members being `False` and `True`. This means you can also pattern match with a bool, although it's recommended to use `if`/`else` instead. Enum-members are capitalized by convention. 

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
| `float` | floating ploint number | `f` | `1.0`, `1.5f`, `100f`, `2.0e100` |

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

#### Characters

Characters or `char` are written with apostrophes (`'…'`) *(also called single quotes).* They store 1 byte of data. You can also do arithmetic on them like with numbers.

```mulem
a = 'a'
b = a + 1
print("{b}")   -- Print "b", the letter after 'a'
c = b + 1
print("{c}")   -- Print "c", the letter after 'b'
```

Characters can be escaped with a backslash `\` between the quotes. Some letters have special values like `\t` for tabs, `\n` for new lines, etc. All the standard stuff you would expect from a modern language. 

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

Write a basic raw string with `''…''` (two apostrophes). Although apostrophes `'` are used for chars, an empty char isn't possible since the default char is written `'\0'` (null character). A common practice in programming languages uses the double quote `"` for formattable strings and the single quote mark `'` for raw strings, so it should be easy for any programmer to see the parallel. When you write `''`, every character after it **except for new-lines** is in the string until the closing `''`. Escaping with backslashes `\` and insertion with curly braces `{}` are disabled. This is for single-line raw strings only. If there's a line break before the closing `''`, then it's a syntax error. *(See below for multi-line raw strings.)*

```mulem
rawString = ''It's okay to put an apostrophe (') in the string.''
filePath = ''C:\files\on\windows.txt''
template = ''Insert here → {{variable}}''
```

### Arrays

Array types are declared with square brackets around their type (`[*T]`). A number before the multiplication mark `*` makes it a fixed length array `[N*T]`. Arrays are statically sized when written `[N*T]`, while `[*T]` is the dynamic form. Items are separated with commas (`,`). Index is done with the `^[]` and `.[]` operators. 

```mulem
list: [4*int] = [1, 2, 3, 4]
print("length of list: {len(list)}")
compressedList = [list^[0] + list^[1], list^[2] + list^[3]]
doubleArray: [3*2*int] = [[1, 2], [3, 4], [5, 6]]
item = doubleArray^[1]^[0]    -- The 2nd row, 1st column
item = doubleArray^[1,0]      -- Or seperated with commas
print("{item}")               -- Prints "3"
```

If access to an index in an array cannot be guaranteed, you can use the safe access operator `.[]`. For an array of `[*T]`, `.[]` will return type `T?` and `^[]` will return a type `T`.

```mulem
i = randInt()    -- Undeterministic number.
list.[i] ?: -1   -- Fallback to -1 if out of bounds.
```

In general, you'll mostly be using arrays by iterating or piping them. 

```mulem
loop x in list then
    print("{x}")     -- No need to use `#`
```

Use the spread operator `*` to spread an array into another array. This must be the first prefix operator in an expression, and it must be in a compatiable array or tuple literal. It always goes last in the slot's expression, so extra parentheses aren't necessary: `*a <> b` == `*(a <> b)`.

```mulem
a = [1, 2, 3]
b = [0, *a, 4]              -- == [0, 1, 2, 3, 4]
c = a <> b                  -- == [1, 2, 3, 0, 1, 2, 3, 4]
d = [0, *a <> b, 5, *c, 6]  -- == [0, 1, 2, 3, 0, 1, 2, 3, 4, 5, 1, 2, 3, 0, 1, 2, 3, 4, 6]
```

If you spread an array into a tuple, the type must be known and the tuple must have compatible components. Positional components will map to array indexes or iterator yields based on where the spread is placed inside the tuple. If the tuple runs out of space, the spread will be truncated. This works similar to variadic parameters in functions.

```mulem
ThreeInts :: (int, int, int)

list = [1, 2, 3]
a: ThreeInts = (*list)      -- == (1, 2, 3)
b: ThreeInts = (0, *list)   -- == (0, 1, 2), truncated at the end
```

Tuples may also collect any remaining positional components into an array, just like variadic parameters in functions.

```mulem
TwoOrMoreInts :: (int, int, *int)

list = [1, 2]
a: TwoOrMoreInts = (*list)              -- == (1, 2, [])
b: TwoOrMoreInts = (0, *list)           -- == (0, 1, [2])
c: TwoOrMoreInts = (-1, 0, *list)       -- == (-1, 0, [1, 2])
d: TwoOrMoreInts = (-2, -1, 0, *list)   -- == (-2, -1, [0, 1, 2])
```

*Mulem separates tuple and array spread to preserve =type safety and avoid runtime surprises. `&` always preserves tuple structure; `*` always preserves array structure.*

### Dictionaries

Dictionaries are a subtype of arrays. Instead of numbers, each item is given a **key.** A dictionary's type is the type of the value `V` and the type of the key `K` join with a colon `:` in between: `[K: V]`. This makes it semantically clear that they are a subtype of arrays. Dictionaries also use the same operator to access items. The type passed to the `^[]` or `.[]` operators must match the key type. Each key is marked with `[]:` in the array.

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

Product unions with the `&` operator can be used for both types and values. When combining two or more positional tuples, the positions of subsequent tuples get bumped up by the number of positions in the previous tuples, i.e. `(a, b) & (c, d)` becomes `(a, b, c, d)`. When you combine two or more named tuples, conflicting named parameters override each other with the last tuple taking priority—much like how shadowing works. So if you have `{x: 1} & {x: 2}`, the result is just `{x: 2}` since it overrides the `x` of the previous tuple. Positional tuples and named tuples can be combined together for example `(0, 1) & {x: 2}`. The shorthand for this is to write named parameters in a positional tuple like `(0, 1, x: 2)`. 

You can think of it like every tuple always having both dimensions, just with most slots empty:

| Type                     | Value          | Positional | Named  |
|:-------------------------|:---------------|:---------|:---------|
| `(int, int)`             | `(0, 1)`       | `(0, 1)` | `{}`     |
| `{x: int}`               | `(x: 2)`       | `()`     | `{x: 2}` |
| `(int, int) & {x: int}`  | `(0, 1, x: 2)` | `(0, 1)` | `{x: 2}` |
| `{x: int} & (int, int)`  | `(x: 2, 0, 1)` | `(0, 1)` | `{x: 2}` |

So `&` has different commutativity rules depending on what's being combined:

|      Combination  | Commutative? |        Rule                 |
|:-----------------------:|:---:|:-------------------------------|
| Positional & Positional | No  | Positions concatenate in order |
|      Named & Named      | No  | Conflicts resolve last-wins    |
| Positional & Named      | Yes | Orthogonal, no interaction     |

This makes the algebra quite principled. The only cases where order matters are also the cases where a conflict is actually possible — two positional slots or two named slots with the same key. When there's no possible conflict, order is irrelevant.

It also means the shorthand `(0, 1, x: 2)` isn't really special syntax. It's the natural representation of a tuple that has both dimensions populated.

Every opaque type by itself is its own tuple, so for example `char` and `(char)` are the same. This means that in the example, `int & float & char` is the equal to `(int, float, char)`.  Creating a product type of opaque types like primitives and enums coerce into a positional tuple, e.g. `int & float & char` becomes `(int, float, char)`. 

Combining empty tuples produces an empty tuple `() & () == ()`. The same is true for empty named tuples `{} & {} == {}`. This also means that empty positional tuples and empty named tuples are equivalent `() == {}`. Both tuples have zero dimensions in both positional and named components; therefore they are equivalent. Saying `() & {} & ()` or `{} & ()` and any combination of empty tuples all are the same type.

### Pointers `T^`

Raw pointers are type `ptr`. These cannot be dereferenced. They are ideally used for FFI to pass to functions of external libraries. You can also check if it's `Null`.

```mulem
o: ptr = externalLibrary.getObject()
if o /= Null then
    externalLibrary.useObject(o)
else
    print("Initialization failed")
```

Typed pointers are type `T^`. They're are like references but dereferenced with the `^` postfix operator.

```mulem
mu x = 0
xPtr: int^mu = %mu x
xPtr^ := 1
x             -- Value: 1
```

Safe pointers are made by wrapping `T^` or `T^mu` in `Some`. If you pass in `Some(Null)`, it will be converted to `None`.

To allocate memory on the heap, use the `alloc[]` and `free[]` functions.

```mulem
Student :: struct =
    name: str
    grade: char

student: Student^mu = alloc[ Student(name: "John", grade: 'A') ]?
defer free[student]

student^.name := "John Smith"
```

`alloc` will check the size of the type passed to it and allocate that much space, returning a `T^mu?` (safe pointer). If successful, it will run the expression inside the square brackets and return `Some(T^mu)`. If not, it will return `None`. Hence `?` after it to unwrap the return value. `T^mu` can also be downgraded to `T^` if you don't plan to change the data.

`free` will free the memory to a pointer and convert it to `Null` even if it was declared immutable to prevent dangling pointers.

`defer` will run an expression at the end of a function.

This won't prevent all memory safety issues. Mulem isn't aiming to be the next Rust. Think of this memory model as more of an upgraded C — less need to do arithmetics like `malloc(N * sizeof(T))` or remembering to call `free(p)` before the end of a function.

By default, they're isn't an ownership model in Mulem. If you don't plan on using a pointer after getting one from a function, you can put `defer free[p]` on the next line.

```mulem
student = getStudent()
defer free[student]
-- Use student freely below.
```

When you are sure that a pointer exists, you can use `^.` for raw access, but if you aren't sure you can use just `.` to do a safe pointer check. This will return a `T%mu?` of that member.

```
student^.name := "John Smith"    -- Raw access
student.name? := "John Smith"    -- Safe access
```

`student.name` is a safe pointer operation like `.[]` for arrays, so you would write `student.name?`; because `student` isn't a question type `T?` but `student._` returns a question type of `T%mu?` of that member.

This makes it easier to chain when you have a safe pointer `T^?`/`T^mu?`

```
pointer?.field1?.field2?.field3?    -- Unwrap each field
```

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
x: int!Error = getRiskyInt()   -- Get wrapped value.
y: int = x!                    -- Unwrap the exclamation
                               -- Which is equivalent to…
y =
    match x is
    | Success(val) =
        val                    -- Get the Ok value.
    | Error(e) =
        return Error(e)        -- Exit block, return error if a function
```

For basic errors, use the built-in `Error` type. It optionally takes either a `string` (message) or an `int` named parameter `code:`, or both. If a message is missing, it will construct one based on the code, and if a code is missing, it will default to `-1`. 

```
raise Error()
raise Error("message")
raise Error(code: -1)
raise Error("message", code: -1)
```

## Custom Types

Custom types are made with `::` (meta assignment).

### Aliases

Assigning a type after `::` creates an alias. 

```mulem
numberType :: int
```

This alias is unique to the scope. Modifying it only affects the alias and not the original type. This prevents accidental conflictions between modules. *(See [Implementation](#implementing-impl).)*

You can also create aliases for basic product types or sum types.

```mulem
tuple        ::  int , float , char                    -- Also called a "positional tuple".
alsoTuple    :: (int , float , char)                   -- Optional parentheses.
namedTuple   :: {count: int, scale: float, code: char} -- Position not guaranteed.
mixedTuple   :: (int, float) & {code: char}            -- Has both positional and named components.
productUnion ::  int & float & char                    -- Is the size of all types combined.
sumUnion     ::  int | float | char                    -- Is the size of the largest type
.
```

### Structural Types (`struct`)

Structs are product types—or in other words—plain data containers. They cannot extend other structs, but can inherit members of other structs. *(See [Inheritance and Visibility](#inheritance-and-visibility).)*

Put an equals sign after the `struct`. This makes it easier to tell type definitions from aliases and lets the parser know that it's starting a block since a block always starts after a `:` or `=`. `=` was chosen over `:` to show that the block is some sort of data rather than control flow. This makes it clear that when you see `=` at the end of a line, something is being defined. It also resembles the familiar `var: type = value` but the `::` makes it clear that this isn't a run-time value. 

```mulem
MyStruct :: struct =
    name: str
    value: int
```

You can write it with one line, separating each member with a comma `,`.

```mulem
MyStruct :: struct = name: str, value: int
```

Instantiate a struct by calling it like a function. Each member is treated like a named argument.

```mulem
myObject = MyStruct(name: "Foobar", value: 1)
```

Structs are transparent. They can be destructured like named tuples. 

```mulem
TransparentThing :: struct =
    a: int
    b: int

{a, b} = TransparentThing(a: 1, b: 2)
print("a: {a}, b: {b}")
```

### Enumerable Types (`enum`)

Enums are sum types. They define a closed set of variants. Variants may carry data turning them into a tagged union. Like with `struct`, place an `=` after `enum` before starting the block. 

```mulem
MyEnum :: enum =
    First
    Second(int)
    Third{val: int}
```

The inline version works the same.

```mulem
MyEnum :: enum = First, Second(int), Third{val: int}
```

Like structs, instantiate by calling the member like a function unless it doesn't carry any data.

```mulem
a = MyEnum.First
b = MyEnum.Second(2)
c = MyEnum.Third(val: 3)
```

When pattern match, the fill path to the type doesn't need to named on each case, only the name of each member. Use `*` while destructuring to discard the members data.

```mulem
match a is
| First =
    print("first!")
| Second(*) =
    print("second!")
| Third{*} =
    print("third!")
```

#### Untagged Unions

`enum` can define a tagged union, but untagged unions can be defined with the `|` inside a type notation. When picking members of a union, use `of` with the type. 

```mulem
sumUnion :: int | float | char

u: sumUnion = 1

u of int     -- Value is 1
u of float   -- Read binary representation of int 1 as if it were a float
u of char    -- Read binary representation of int 1 as if it were a, value is '\1'
```

This is similar to C unions where it doesn't do any conversion; it only reads whatever data is there with a different type.

Unions will set overflow data to 0 so that if you set a small member and then read from a big member, you won't get undefined behavior. 

The zero-padding interacts with endianness in a way that's deterministic but platform-dependent. The behavior is always defined, just not always portable. 

```mulem
u: sumUnion = '\1'   -- char (1 byte), remaining bytes zeroed

-- Little-endian: memory is [0x01, 0x00, 0x00, 0x00]
u of int             -- 1

-- Big-endian: memory is [0x01, 0x00, 0x00, 0x00]  (same bytes)
u of int             -- 0x01000000 = 16777216
```

Overloaded functions are also a kind of untagged union, one that holds different function pointers instead of types. When called, they are automatically determined by the compiler based on its arguments, but there are times when this cannot be determined. Use `of` to pick out a certain definition of an overloaded function when this happens. *(See [Function Declarations](#function-declarations-fn).)*

```mulem
withOneAndTwo(add of (int, int): int)
withOneAndTwo(add of (float, float): float)
```

### Error Types (`error`)

Errors are bit like both `struct` and `enum`. Each `error` type represents a member of a potential **error tagged union** that's summed up per function with a exclamation type `T!E` return type. Every `try` / `catch` block matches patterns to the summed error tagged union in its block based on each `!` point. Exclamation types flatten, so `Exclamation[Exclamation[T, E], F]` would become `Exclamation[T, E|F]` where `E|F` is a tagged union of each possible error in that exclamation. Instantiation works the same as structs.

```mulem
OutOfBounds :: error                 -- No data.
ErrorMessage :: error = (str)        -- Attach a position tuple.
DivideByZero :: error = value: int   -- Attach named member
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

Uncaught errors in a `try` / `catch` block are implicitly reraised. Each `catch` pattern removes a possible error from the exclamation type of that block. When all possible errors have a `catch` arm, the value of that block is automatically unwrapped so that `Exclamation[T, ()]` becomes just `T`. 

### Prototypes (`proto`)

A `proto` is an abstract interface — a named contract with no data. It is equivalent to a `virtual class` / `trait` / `interface` in other languages. Each member is a function, also called a **method**. Methods that have a parameter named `self` at the beginning will be called like methods on the instance of that type, i.e. `self.method(*)`. This is equivalent to saying `typeof[self].method(self, *)`.

```mulem
MyPrototype :: proto =
    speak(self): str
```

### Implementing (`impl`)

Methods and trait implementations are added separately with `impl`. Much like `proto`, `self` in the first parameter of a method refers to the current instance. You can also add static values that are attached to the type itself. Use `.` to access static values and methods like with structs.

```mulem
MyStruct :: impl =
    staticValue = 1234
    init(name: str, value: int): MyStruct =
        MyStruct(name: name, value: value)

print("{ MyStruct.staticValue }")
```

A method with `self` as the first parameter are callable as a method on the instance. `self` refers to the instance, analogous to `self` / `this` in other languages. You can optionally give it another name with `as`. 

```mulem
MyType :: struct = foo: str

MyType :: impl =
    method1(self) = self.foo              -- `self` is a `MyType`.
    method2(self as this) = this.foo      -- Can be aliased with `as`.
    method3(self as myType) = myType.foo  -- Alias name can be any valid variable name.

x = MyType(foo: "bar")
print("{ x.method1() }")  -- Prints "foo"
print("{ x.method2() }")  -- Prints "foo"
print("{ x.method3() }")  -- Prints "foo"
```

To implement from a prototype, add the proto's name in the square brackets. Each implementation gets their own `impl[]` block. 

```mulem
MyStruct :: impl[MyPrototype] =
    speak(self) = "I am a {self.name} and I have ${self.value}."

MyEnum :: impl[MyPrototype] =
    speak(self) =
        match self is
        | First =
            "I am a MyEnum of First"
        | Second(x) =
            "I am a MyEnum of Second({x})"
        | Third{val} =
            "I am a MyEnum of Third \{ val={val} }"
```

When importing a prototype, you need to call `impl` on it to activate it in the scope of the module.

```
{MyStruct, MyPrototype} :: import    -- Import types.

impl MyPrototype                     -- Activate this prototype in scope.

me = MyStruct.init("Bob", 100)
me.speak()                           -- Prints "I am Bob, and I have $100."
```

[TOC](#table-of-contents)

---

## Meta Functions

Adding a parameter before the double colon (`::`) turns it into a **meta function.** *A meta‑function is a compile‑time function that returns code, types, or values.* Parameters are put in square brackets `[]` to distinguish them from regular functions which use parentheses `()`. The result is treated like a constant for run-time code. 

You can define a meta function by adding a parameter before the double colons and writing an expression after it. Like constants, they don't output a value in memory when compiled, useful for collecting repeated code. Unlike regular functions, meta function cannot be passed to another function. They only exist at compile-time. Each parameter is a variable within the expression, so you don't need to wrap them in parentheses `()` like with C macros. 

```mulem
max[a, b] :: = if a > b then a else b
min[a, b] :: = if a < b then a else b
```

You can also have multi-line meta function like regular functions. Each meta function creates a new scope. Defining variables that could bleed into the surrounding scope is not allowed. The last expression is the return value. Call it like a function using `[]`. 

```mulem
doSomethingComplicated[x] :: =
    x = x + 1
    x = x / 2
    x * x

value = doSomethingComplicated[3]
```

These two are the same thing:

```mulem
value =
    x = (3) + 1
    x = x / 2
    x * x
```

Some more examples using the `[]` notation:

```mulem
max[a, b] :: = if a > b then a else b
min[a, b] :: = if a < b then a else b
print("{ max[0, 1] }")           -- Prints "1"
print("{ min[0, 1] }")           -- Prints "0"
print("{ max[1+2, 3+4] }")       -- Prints "7"

f(x) = x * x
g(x) = x + 2
maxAdd[a, b] :: = if a > b then fn(c) = a + c else fn(c) = b + c
print("{ maxAdd[ f(0), g(0) ] (1) }")
```

The compiler will read the body of the macro and understand where to insert its parameters, so if a parameter gets shadowed, then it will no longer insert it for the rest of that scope. 

You can also define a type with a meta function. The meta functions parameters in `[]` will be inferred if it returns a function or type and you call it with parentheses `()`. In other words, `meta(*)` is equal to `meta[_](*)`. 

```mulem
-- Note that this is not the actual definition for a question type `T?`. This is just a user-defined enum that uses the same pattern.
Option[T] :: enum =
    Some(T)
    None

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

The syntax `[]` was chosen so that generic type inference will take precedence. `meta(a, b)` means to *call the instantiated function that `meta` returns with inferred types* whereas `meta[a, b]` means to *call the abstract function `meta` with these exact values.* This also makes it easy to distinguish actual function calls from macros/inlining. This removes the need for the more conventional arrow bracket `<>` syntax, which can get confusing. For example, in `f( g < a, b > ( c ) )`, is `g` a generic function or is that comparing two values and passing the results to `f`? The square bracket syntax removes this ambiguity, `f( g [ a, b ] ( c ) )`. This makes it semantically clear that you're doing a compile-time function call followed by a run-time function call. 

[TOC](#table-of-contents)

---

## Operators

| Level | Category                   | Operators                        |
|:------|:---------------------------|:---------------------------------|
| 11    | Member access/Function     | `.` `.[]` `^[]` `?.` `?.[]` `()` |
| 10    | Postfix/Prefix             | `?` `!` `^` `%` `%mu`            |
| 9     | Unary                      | `+` `-` `not`                    |
| 8     | Exponent                   | `**` (right-associative)         |
| 7     | Multiplicative / Shift     | `*` `/` `//` `<<` `>>`           |
| 6     | Additive / Concat          | `+` `-` `<>`                     |
| 5     | Range                      | `..` `..=`                       |
| 4     | Comparison                 | `==` `/=` `<` `>` `<=` `>=`      |
| 3     | Logical AND                | `and`                            |
| 2     | Logical OR / None-Coalesce | `or` `?:`                        |
| 1     | Pipeline                   | `\|>`                            |
| 0     | Assignment / Spread        | `=` `:=` `+:=` `-:=` `&` `*`     |

| Operator       | Meaning                                              |
|:--------------:|:-----------------------------------------------------|
|   `lhs . rhs`  | Member access                                        |
|   `lhs .[rhs]` | Safe array/dictionary index                          |
|   `lhs ^[rhs]` | Raw array/dictionary index                           |
|   `lhs ?`      | Unwrap question, propagate `None` to nearest `maybe` |
|  `lhs ?. rhs`  | Member access on a question type                     |
|  `lhs ?.[rhs]` | Safe array/dictionary index on a question type       |
|   `lhs !`      | Unwrap exclamation, propagate error to nearest `try` |
|   `lhs ^`      | Dereference typed pointer                            |
|       `% rhs`  | Get immutable reference                              |
|     `%mu rhs`  | Get mutable reference                                |
|       `* rhs`  | Spread array into array                              |
|       `& rhs`  | Spread tuple into tuple (same type)                  |
|  `lhs .. rhs`  | Exclusive range                                      |
| `lhs ..= rhs`  | Inclusive range                                      |
| `lhs \|> rhs`  | Pipeline                                             |
|   `lhs = rhs`  | Declaration                                          |
| `lhs: T = rhs` | Explicit typed declaration                           |
|   `lhs + rhs`  | Addition                                             |
|   `lhs - rhs`  | Subtraction                                          |
|       `+ rhs`  | Unary positive                                       |
|       `- rhs`  | Sign flip                                            |
|   `lhs * rhs`  | Multiplication                                       |
|   `lhs / rhs`  | Exact division (float)                               |
|  `lhs // rhs`  | Floor division (int)                                 |
|  `lhs ** rhs`  | Exponentiation (right-associative)                   |
|  `lhs == rhs`  | Equality                                             |
|  `lhs /= rhs`  | Inequality                                           |
|   `lhs > rhs`  | Greater than                                         |
|   `lhs < rhs`  | Less than                                            |
|  `lhs >= rhs`  | Greater than or equal                                |
|  `lhs <= rhs`  | Less than or equal                                   |
|  `lhs << rhs`  | Append to array                                      |
|  `lhs >> rhs`  | Prepend to array (right-associative)                 |
|  `lhs <> rhs`  | Concatenation                                        |
| `lhs and rhs`  | Logical AND                                          |
|  `lhs or rhs`  | Logical OR                                           |
|     `not rhs`  | Logical NOT                                          |

Some math operators will be put into a standard library. These will be inlined to ensure performance.

```
std.math{Arithmetic, Bitwise} :: import

lhs = 1
rhs = 2

impl Arithmetic

lhs rem rhs    -- Remainder (C-Style modulo) `%`
lhs mod rhs    -- True Modulo

impl Bitwise

lhs band rhs   -- Bitwise AND `&`
lhs bor rhs    -- Bitwise OR `|`
lhs xor rhs    -- Bitwise XOR `^`
bnot rhs       -- Bitwise NOT `~`
lhs shl rhs    -- Bitshift Left `<<`
lhs shr rhs    -- Bitshift Right `>>`
lhs shru rhs   -- Unsigned Bitshift Right `>>>`
```

__Compound Assignment Operators:__

| Operator         | Meaning                         |
|:----------------:|:--------------------------------|
|  `lhs := rhs`    | Assignment                      |
|  `lhs +:= rhs`   | `lhs := lhs + rhs`              |
|  `lhs -:= rhs`   | `lhs := lhs - rhs`              |
|  `lhs *:= rhs`   | `lhs := lhs * rhs`              |
|  `lhs /:= rhs`   | `lhs := lhs / rhs`              |
| `lhs <<:= rhs`   | `lhs := lhs << rhs`             |
| `lhs <>:= rhs`   | `lhs := lhs <> rhs`             |
| `lhs rem:= rhs`  | `lhs := lhs rem rhs`            |
| `lhs mod:= rhs`  | `lhs := lhs mod rhs`            |
| `lhs band:= rhs` | `lhs := lhs band rhs`           |
| `lhs bor:= rhs`  | `lhs := lhs bor rhs`            |
| `lhs xor:= rhs`  | `lhs := lhs xor rhs`            |
| `lhs shl:= rhs`  | `lhs := lhs shl rhs`            |
| `lhs shr:= rhs`  | `lhs := lhs shr rhs`            |
| `lhs shru:= rhs` | `lhs := lhs shru rhs`           |

### Key Type Modifiers & Postfix Operators

| Syntax         | Meaning            | Note                                                      |
|:---------------|:-------------------|:----------------------------------------------------------|
|  `T?`, `x?`    | Question           | Unwraps a question; propagates `None` to nearest `maybe`. |
|  `T!`, `x!`    | Exclamation        | Unwraps a exclamation; propagates error to nearest `try`. |
|  `T^`, `x^`    | Pointer            | Dereference a pointer.                                    |
|  `T%`, `%x`    | Reference          | Get a reference to a place in memory.                     |

[TOC](#table-of-contents)

---

## Advanced

[TOC](#table-of-contents)

- __[Operator Overloading](#operator-overloading)__
- __[Pipelining](#pipelining)__
- __[Parameter Modifiers](#parameter-modifiers)__
- __[Lambda Functions](#lambda-functions)__
- __[Pattern Matching](#pattern-matching)__
- __[Iterator / Async Functions](#iterator-async-functions)__
- __[Inheritance and Visibility](#inheritance-and-visibility)__
- __[Manual Implementation](#manual-implementation)__
- __[Importing and Modules](#importing-and-modules)__

- __[Code Sample](#putting-it-all-together)__
- __[Reserved Keywords](#reserved-keywords)__

---

### Operator Overloading 

Each operator is a prototype that can be implemented on a type.

```
Vector2D :: struct =
    x: float
    y: float

Vector2D :: impl[Add] =
    op[+](self as lhs, rhs: Vector2D): Vector2D =
        Vector2D(
            x: lhs.x + rhs.x,
            y: lhs.y + rhs.y,
        )

impl Add      -- Use this prototype in scope.

v1 = Vector2D(x: 1, y: 2)
v2 = Vector2D(x: 3, y: 4)

v1 + v2   -- Result: Vector2D(x: 4, y: 6)
```

Custom operators can be defined with `op`. These must be defined above where they will be used so that the lexer can correctly identify user-defined operators. The symbol can be either entirely ASCII symbol characters or a valid variable name. It must not conflict with any other operator symbols in scope. If its a valid variable name, then that name can no longer be used as a variable name when the operator is in scope. 

```
Remainder :: op[order: 5.0, symbol: "rem", side: op.Infix, rightAssoc: False]

int :: impl[Remainder] =
    @inlined
    op[rem](self as lhs, rhs: int): int =
        lhs - rhs * (lhs // rhs)

impl Remainder      -- Use this prototype in scope.

15 rem 12    -- Result: 3
```

It has to be defined above where it's being used otherwise if the lexer finds `a b c d​`, it won't know an operator from a variable or in what order they need to go in. 

You don't import the operator itself like `rem`​; you import the operator's prototype `Remainder` in this case​, and if that's in scope, then the operator is in use. Then, anytime the lexer sees `a rem b​` it knows that's an operation with a set order. It won't know what it does; that's for the parser who will look up the implementation of `Remainder​` on `a​`. If you used a custom operator on a type that doesn't implement it, then it would be the same kind of error you get if you used a built-in operator on the wrong types like `string + string`​ instead of `string <> string`​. 

[Advanced](#advanced) / [TOC](#table-of-contents)

---

### Pipelining `|>`

When placed at the start of a line in a block, the pipelining operator `|>` takes on a special meaning: it takes the value of the previous line and makes it available in the next line as `$`. This symbol is known as the **pipeline context.** This makes it easy to chain a sequence of calls and read them in order.

```mulem
print("{
    fetchC(
        fetchB(
            fetchA()
        )
    )
}")
```

*Becomes…*

```mulem
fetchA()
|> fetchB(&$)
|> fetchC(&$)
|> print("{$}")
```

This makes the order of operations easy to follow at a glance. It reads like a plain-English list:

- `fetchA`
- *then* `fetchB`
- *then* `fetchC`
- *then* `print`

The first line of a block may also start with `|>`, in which case its pipeline context is an empty tuple `()`. A line starting with `|>` is not required to use the pipeline context. 

When mixed with `do`, multiple expressions separated by semicolons `;` on one line will share the same pipeline context. The last expression on the line is passed as the pipeline context to the next pipe.

```mulem
|> fetchA()                      -- Run fetchA,
|> do print("{$}"); fetchB(&$)   -- Print result, then fetchB
|> do print("{$}"); fetchC(&$)   -- Print result, then fetchC
|> print("{$}")                  -- Print result.
```

Note that `|>` at the beginning of a line is different from `\ |>` which is a split expression on the next line. The pipe will continue after it.

```mulem
|> fetchA()
\ |> fetchB(&$)
|> fetchC(&$)
|> print("{$}")
```

*Is the same as…*

```mulem
|> fetchA() |> fetchB(&$)    -- Goes back to this line.
|> fetchC(&$)                -- Pipe from the previous statement.
|> print("{$}")
```

A **pipeline block** is started with `|> do` and a new line, either at the end of a line or on its own line. Within the block, `$` holds the piped-in value within the scope of that block.

```mulem
|> fetchA()         -- Set up things.
|> fetchB(&$)       -- …
|> fetchC(&$)       -- …
|> do               -- Pipeline context is now ready.
    print("{$}")    -- Use it here.

fetchA() |> fetchB(&$) |> fetchC(&$) |> do   -- Or in one line.
    print("{$}")                             -- Then use the result.

-- Freely mix the two formats:
fetchA() |> do         -- Start with this pipeline context.
    print("{$}")       -- Use the same `$` for these two lines.
    fetchB(&$)         -- Same pipeline context `$`.
    |> print("{$}"); fetchC(&$) |> do  -- Start a new pipeline inline.
        print("{$}")                   -- Print the final result.
```

To get a value within a pipeline, use `as x` after any step to store it into a local variable. The assignment is written in reverse order — the variable name goes on the right.

```mulem
|> fetchA()
|> fetchB(&$)
|> fetchC(&$) as x    -- Put the result into `x`.

print("{x}")          -- Print the result.
```

This lets you extract the result of any step in a pipeline simply by appending `as name` to that line.

```mulem
-- Put all results of each step into variables.
|> fetchA() as a
|> fetchB(&$) as b
|> fetchC(&$) as c

print("a = {a}, b = {b}, c = {c}")
```

The variable type is always inferred, to avoid ambiguity with `:`. Mutability can be specified with `mu`. *(See [Mutability](#mutability).)*

```mulem
fetchA() as mu x |> fetchB(x) |> do   -- Create a mutable variable `x`.
    x +:= 1                         -- Mutate it.
    print("{x}")                    -- Print it.
```

To put the final result of any pipeline into a variable, use `|> $ as x` at the end, where `x` is any variable name. This makes it easy to mix and match the pipes in between with `|> $ as x` at the end to collect it all into a variable. 

```mulem
|> fetchA()      -- Start.
|> fetchB(&$)    -- Pass pipeline context.
|> fetchC(&$)    -- Pass pipeline context.
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

### Parameter Modifiers

| Modifier         | Behavior          | Mutable inside function?           |
|:-----------------|:------------------|:-----------------------------------|
| *(none)*         | Pass by copy      | No                                 |
| `mu`             | Pass by copy      | Yes                                |
| `ref`            | Pass by reference | Yes / No                           |
| `out`            | Unset reference   | Yes (Must be assigned)             |
| `opt`            | Optional argument | Wraps in T?`                       |

Function parameters can be declared like variables. Likewise, you can modify their mutability and reference-ness the same way.

`ref` is used to infer a `T%` (immutable reference) or `T%mu` (mutable reference). Wether it's immutable or mutable depends on the usage in the function.

```mulem
fn increment(ref x) =
    x +:= 1

mu y = 0
increment(y)
```

`out` parameters are guaranteed‑set references. They behave like `ref`, except they begin the function in an *unset* state. Use an `out` parameter when a function’s job is to produce a value rather than consume one. An `out` parameter must not remain unset in any branch of the function. It must either be:

- Assigned directly, *or*
- Passed to another function that also takes an out parameter.

This guarantees that the variable is initialized after the call completes.

```mulem
fn setInt(out i): void =
    i = 3

mu x: int
setInt(x)
print("{x}")    -- Prints "3"
```

This is fine, but what if you just wanted to grab the `out` variable in one go? Normally you could just write `n = setInt()` for return values — but `out` parameters aren't retaurn values. We can inline a variable from the `out` parameter with the keyword `as` — just like how we use `as` for aliasing. This makes it clear that we're expecting a new variable at the call site. That way it's *explicit* that we're intentionally declaring a new variable instead of passing in an existing one.

```mulem
setInt(as n)    -- Declare a new variable as `n` that gets set by `setInt`.
print("{n}")    -- Prints "3"
```

`setInt(as n)` → "Set int as n"—it says exactly what it does. This makes it clear that you're intentionally making a new variable. Using `as` makes the intent unambiguous: you are creating a variable, not passing an existing one. `as` already means "bind this to a name" throughout the language:

```mulem
{x as y} = {x: 2}           -- destructuring
choice is Second(x as val)  -- pattern matching
method(self as this)        -- self aliasing
setInt(as n)                -- out parameter
```

Languages that use return values for this kind of thing (`n = setInt()`) imply the value comes out of the function through the normal return channel, which is misleading when the mechanism is actually a reference parameter. `setInt(as n)` makes the call-site declaration explicit without requiring you to pre-declare a `mu` variable just to hand it in.

#### Optional Parameters

Parameters can be made optional with the `opt` modifier. This wraps the variable in type `T?`. If the parameter is missing, it will be set to `None`.

```mulem
fn addOptional(opt a: int, opt b: int): int =
    aVal = a ?: 0    -- Coalesce optional arguments with default value 0.
    bVal = b ?: 0    -- This unwraps their value if they exist or set them to 0.
    aVal + bVal      -- Add the unwrapped values.

print("{addOptional()}")      -- Prints "0"
print("{addOptional(1)}")     -- Prints "1"
print("{addOptional(1, 1)}")  -- Prints "2"
```

When calling a function with an explicit `T?`, you give it a value of `T?`. When calling a function with `opt`, you give it a value of `T`. 

```mulem
fn optionalParam(opt val: int) =
    if val is Some(x) then
        print("Some({x})")
    else
        print("None")

fn requiredParam(val: int?) =
    if val is Some(x) then
        print("{x}")
    else
        print("None")

x: int = 5
optionalParam(x)         -- Prints "Some(5)"
optionalParam()          -- Prints "None"
requiredParam(Some(x))   -- Prints "Some(5)"
requiredParam(None)      -- Prints "None"
-- requiredParam(x)      -- Error: `x` is not `int?`
-- requiredParam()       -- Error: missing parameter `val: int?`
```

`opt` parameters can also have default values. Use `=` to to give an optional parameter a default value. This will make it a type `T`, so unwrapping is unnecessary.

```mulem
fn addOptional(opt a = 0, opt b = 0): int = a + b     -- `a` and `b` are always `int`s

print("{addOptional()}")      -- Prints "0"
print("{addOptional(1)}")     -- Prints "1"
print("{addOptional(1, 1)}")  -- Prints "2"
```

Use `*` to collect all arguments into a single variable. The variable should be type `[*T]` (an array).

```mulem
fn addAll(*nums: [*int]): int =
    mu sum: int = 0
    loop n in nums:
        sum +:= n
    sum

print("{addAll()}")         -- Prints "0"
print("{addAll(1)}")        -- Prints "1"
print("{addAll(1, 2)}")     -- Prints "3"
print("{addAll(1, 2, 3)}")  -- Prints "6"
```

```mulem
-- With pattern matching:
fn addAll(*nums: [*int]): int =
    match nums is
    | []            = 0
    | [x]           = x
    | [x, *rest]  = x + addAll(*rest)
```

A name is optional after `*`. You can use the symbol by itself to pass it to another function or itself in a functional loop. 

```mulem
fn addAll(x: int, *): int =
    x + addAll(*)
fn addAll(x: int): int =
    x
fn addAll(): int =
    0

fn logAndAdd(msg: str, *) =
    print("{msg} {addAll(*)}")

logAndAdd("Sum =")           -- Prints "Sum = 0"
logAndAdd("Sum =", 1)        -- Prints "Sum = 1"
logAndAdd("Sum =", 1, 2)     -- Prints "Sum = 3"
logAndAdd("Sum =", 1, 2, 3)  -- Prints "Sum = 6"
```

[Advanced](#advanced) / [TOC](#table-of-contents)

---

### Lambda Functions

Define a function within an expression with `fn` + any name. For demonstration purposes, we'll be using anonymous functions in the pattern `fn(x) = x`. This is useful for passing functions to other functions.

```mulem
(fn(arg) = expr)

(fn(arg) =
    body
)
```

```mulem
map(array, action) = [*loop x in array then action(x)]
array0 = [1, 2, 3, 4]
array1 = map(array0, fn(x) = x + 1)   -- Inline
array2 = map(array0, fn(x) =          -- Multi-line
    if x < 2 then
        x - 1
    else
        x + 2
)
```

A name is optional. Adding a name creates an immutable reference of the function itself.

```mulem
otherAction(fn callback(val) =
    if val > 0 then
        print("{val}")
        callback(val - 1)
    else
        print("done")
)
```

Capturing also works inside lambda functions just like with named functions.

```mulem
mu count = 0
forEach([1, 2, 3, 4], fn(x) / (count) =
    count +:= x
)
```

#### Curried functions

When a function returns another function, list each function parameters as the return type. Optionally, you can just let the return type be inferred.

```mulem
curriedFn(a: int): (int): (int): int = _                 -- Immutable declaration
mu curriedFnPtr(int): (int): (int): int = curriedFn      -- Mutable declaration. 
```

The return type can be implied. Each function defined at the bottom is the implied return.

```mulem
curryFn(a: char) =
    print("In function 1: {a}")
    fn(b: char) =
        print("In function 2: {b}")
        fn(c: char) =
            print("In function 3: {c}")

curryFn('a')('b')('c')
(-- Prints:
"In function 1: a"
"In function 2: b"
"In function 3: c"
--)
```

When capturing variables, each returned function needs to capture them separately.

```mulem
mu count = 0
curryAddCount(a: int) % (mu count): (int): (int): int =
    count +:= a                                   -- (1) Evaluated immediately
    fn(b: int) % (mu count): (int): int =         -- (2) Suspends and captures `count`
        count +:= b                               -- (3) Evaluated when second `fn` is called
        fn(c: int) % (mu count): int =
            count +:= c
            count

print("{ curryAddCount(1)(2)(3) } == { count }")   -- Prints "6 == 6"
```

[Advanced](#advanced) / [TOC](#table-of-contents)

---

### Pattern Matching

```mulem
match expr is
| Pattern1(x) = expr
| (*) = expr
```

The next control flow methods are based on pattern match. Generally, you see the word `is`, you next thing to expect after it is a pattern: `value is Pattern(x)`.

#### `fallthrough`

Proceeds to the next case, which must not destructure new values, unless fallbacks are used.

```mulem
match choice is
| First =
    print("First")
    fallthrough
| Second(opt x) =           -- `opt x` in pattern wraps the variable in a question type
    if x is Some(x) then
        print("Definitely Second: {x}")
    -- Implicit break.
| (*) =
    print("No match")
```

#### Pattern Fallback

If a pattern can't be **guaranteed** for any reason, then you must have a **fallback.** There are two options available:

- __Optional binding:__ `Pattern(opt x)` — wraps `x` in type `T?`, `Some(x)` if it matched, `None` if it didn't
- __Default value:__ `Pattern(opt x = default)` — `x` is type `T`, if it didn't match `x` is set to `default`

You can have multiple patterns match to one case. If any of the patterns destructure with a variable, the same variable name and type must be in all patterns. If not, use a wildcard `(*)` in each pattern or omit the data part entirely to disable destructuring. Otherwise, use a fallback in the pattern.

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
| First | Second(opt val) | Third{opt val} =
    print("First, Second, or Third: {maybe val? else "None"}")
```

#### Pattern Guards

Add `if` inside a pattern to conditionally match.

```mulem
match choice is
| Second(x if x > 0) =
    print("Positive: {x}")
| Second(x if x < 0) =
    print("Negative: {x}")
| Second(x) =
    print("Zero")
| (*) =
    print("No match")
```

#### `is` / `then`

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
result = (value is Pattern(opt x) then x)                              -- Get wrapped Some(x) or None
result = (value is Pattern(opt x) then maybe x? else "fallback")       -- Unwrap with fallback
result = (value is Pattern(opt x = "fallback") then x)                 -- Automatic fallback
```

```mulem
-- Multiple bindings:
result = (value is Pattern(opt x = 0, opt y = 0) then (x, y))
```

```mulem
-- Arbitrary expression over bindings:
result = (value is Pattern(opt x = 0, opt y = 0) then x + y)
```

Pairs naturally with pipelining.

```mulem
getValue()
|> ($ is Pattern(opt x = "fallback") then x)
|> doSomethingWith($)
```

#### `if` + `is`

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

#### `loop` + `is`

Loop while a pattern matches.

```mulem
loop nextValue() is Some(x) then
    print("{x}")
```

#### `until` + `is`

Loop until a pattern matches. Bindings are in scope below the loop.

```mulem
mu i = 0
loop
    print("Attempts: {i}")
    i +:= 1
until getValue() is Pattern(x)

print("{x}")    -- x is guaranteed set here.
```

If `break` is reachable inside the loop, optional bindings are required.

```mulem
loop
    if earlyCondition then
        break
until getValue() is Pattern(opt x)

if x is Some(x) then
    print("{x}")
```

[Advanced](#advanced) / [TOC](#table-of-contents)

---

### Iterator / Async Functions

#### `yield`

Exits out of a function with an `iter[_]` type. The return value of the function must be of type `iter[T]` where T is the yield type. When you have `yield` in your function, the actual return value in the function body is discarded, and using `return …` in it is a compile-time error. Use of `yield` will infer the return type to be `iter[_]`. 

```mulem
fn countUpTo(n: int): iter[int] =
    loop i in 0..n then
        yield i
```

If you use `yield`, you can only use a void `return` to exit the function. 

```mulem
fn countUntil(mu i: int, max: int): iter[int] =
     loop
        if i >= max then
            return      -- Break out of the loop and the function.
        yield i
        i +:= 1
```

#### `await`

Exits out of a function with an `async[_]` type. The return type of the function must be of type `async[T]` where T is the type that the asynchronous value will resolve in the end. The return value of the asynchronous instance is determined the same way that a non-asynchronous function does it. Use of `await` will infer the return type to be `async[_]`. 

```mulem
fn asyncFn(a, b): async[int] =
    a = await fetch(a)
    b = await fetch(b)
    a + b
```

Both `yield` and `await` can be used together in an `iter[async[_]]` type. The type suggests it—each yield is of type `async[_]`. Use `loop await` to wait for each async value to resolve in sequential order.

```mulem
fn asyncIterFn(n): iter[async[int]] =
    loop i in 0..n then
        val = await fetch(i)
        yield val

fn asyncCollect(n): async[[*int]] =
    mu ret: [*int] = []
    loop (await x) in asyncIterFn(n) then
        ret <>= x
    ret
```

Unlike in other languages where *promises* or *futures* can either resolve or reject, async types in Mulem **only resolve.** Instead you can use an exclamation type `T!E` inside an `async[T!E]` function. Unwrap it like you would an exclamation type. Because this is common, `await` has special rules in regards to the `!` and `?` opeerators when placed after it.

```mulem
(await!  x) == (await x)!      -- Unwrap an `async[T!E]`
(await?  x) == (await x)?      -- Unwrap an `async[T?]`
(await!? x) == (await x)!?     -- Unwrap an `async[T?!E]`
```

[Advanced](#advanced) / [TOC](#table-of-contents)

---

### Inheritance and Visibility

Even though structs cannot be extended the usual way, they can **inherit** from other structs using destructuring. This works similarly to **importing.** It marks members that map to members of another struct, making conversion possible. It follows the same convention for pattern matching like destructuring. *(See [Destructuring](#destructuring).)* 

```mulem
Vector2 :: struct =
    x: float
    y: float

Vector3 :: struct =
    {x, y}: Vector2        -- Grab members of Vector2
    z: float

v3 = Vector3(x: 1.0, y: 2.0, z: 3.0)

radius2d(v: Vector2) = sqrt(v.x*v.x+v.y*v.y)
print("{radius2d(v3)}") -- This works because Vector3 inherits from Vector2.
```

When you inherit, you don't just pick out some members. The entire parent struct exists in the child struct in memory, but only some members are visible. 

All members of a type are public by default. When making a subtype, inherited members become private to the subtype unless explicitly redeclared. This encourages separating public and private data into distinct types rather than using access modifiers.

```mulem
PrivateFields :: struct =
    val: int
    secret: int

PublicFields :: struct =
    {val}: PrivateFields     -- Redeclared, `val` is public / `secret` is private
    other: int
```

A subtype cannot accidentally expose or clash with a private inherited member because types only see members that have been explicitly declared within them. This mirrors the convention used for imports.

[Advanced](#advanced) / [TOC](#table-of-contents)

---

### Manual Implementation

Generics will automatically generate code based on their parameters, but you can also implement them by hand using pattern matching. If you only want to use the manual implementations for a generic function, you can set its body to `unimplemented[]`. This creates a virtual function that can be overloaded later. If you use a function that is defined with `unimplemented[]`, it will throw a compile-time error.

```mulem
-- Forces every type to have its own implementation
increment[T] :: (ref c: T): void = unimplemented[]

Counter :: struct = value: int

-- Specialized for Counter
increment[Counter] :: (ref c: Counter): void =
    c.value +:= 1

-- Specialized for float
increment[float] :: (ref c: float): void =
    c +:= 1.0

c = Counter(value: 2)
f = 3.0
b = True

increment(c)   -- T is inferred Counter
increment(f)   -- T is inferred float
(--
increment(b)   -- T is inferred bool which has no implementation, compile-time error
--)
```

[Advanced](#advanced) / [TOC](#table-of-contents)

---

### Importing and Modules

* **Modules:** Declare with `moduleName :: mod`. Only one per file.
* **Imports:** Must be explicit. `a.b{c, d} :: import`. Use `import "path"` for direct file imports.

Use `import` to import something, optionally giving the import an alias with `as`. You can either import a single export like `a.b{c} :: import` or multiple at once using destructuring rules `a.b{c, d} :: import`. *(See [Destructuring](#destructuring).)* This follows the same convention as destructuring with tuples. All imports must be **explicitly** declared—no `import a.b.*`. This helps prevent naming conflicts and track where things have been defined.

The most common import will likely be the `print` function, which will be defined somewhere in a standard library.

```mulem
std{print} :: import    -- This is just an example and not final.

print("Hello, world!")
```

Modules are named with the keyword `mod` near the top before anything is defined. This is the name you'll use when importing your module. **There can only be one `mod` declaration per file.** Multiple `mod` declarations is a syntax error. Imports are based on the include path when compiling or running a script. To import from a file by direct filepath, use `import "path"`.

```mulem
someModule{thing} :: import "../../somewhere.mu"

myModule :: mod

fn addThing(x) = x + thing
```

In this example, you would import `addThing` like this (assuming the file is included):

```mulem
myModule{addThing} :: import
```

This connects the same explicit-list convention as inheriting and capturing — *no hidden dependencies, everything that can affect behavior is named.* The three together form a consistent rule across the language.

#### Memory Models

Mulem is multi-paradigm: different functions, structs, or modules can use different memory strategies in the same program. The model is controlled per-module. Boundary crossing between models follows FFI-like rules — automatic marshalling where possible, explicit escapes otherwise.

Modules define how memory is handled with the `memory` module setting. By default, modules use a garbage collector. Some options include `Collect(GC)` (default),  `Count(ARC)` (reference counting), and `Manual`. `GC` and `ARC` represent the standard garbage collector and reference counter respectively, but others can be defined and used instead.

```mulem
std.mem{Count, ARC} :: import

moduleThatUsesReferenceCounting :: mod =
    memory: Count[ARC]
```

[Advanced](#advanced) / [TOC](#table-of-contents)

---

## Putting It All Together…

Here is a quick synthesized example showing how Mulem's structs, implementations, pipelining, and error handling might look in a real script:

```mulem
std{print} :: import

exampleApp :: mod

(-- 
    Define a struct and implement a prototype.
--)
User :: struct = 
    name: str
    age: int

Speaker :: proto = 
    speak(self): str

User :: impl[Speaker] =
    speak(self) = "Hi, I'm {self.name}."

FetchError :: error = (str)

(-- 
    A function returning an exclamation type (User or an Error).
    It captures no outside state.
--)
fn fetchUser(id: int): User! =
    if id > 0 then
        User(name: "Alice", age: 30)
    else
        raise FetchError("Invalid ID")

(-- 
    Main logic demonstrating error unwrapping (!) 
    and the pipeline operator (|>).
--)
do
    try
        -- Fetch the user, unwrap the exclamation, and pipe it forward
        fetchUser(1)!
        |> do print($.speak()); $
        |> print("User is {$.age} years old."); $
    catch
    | FetchError(e) =
        print("Failed to fetch user: {e}")
```

[TOC](#table-of-contents)

---

## Reserved Keywords

Mulem has 37 reserved keywords. Note that built-in types (`int`, `str`), boolean values (`True`, `False`), standard question `T?` members (`Some`, `None`), are built-in symbols but *not* strict keywords.

**The Keyword List:**
`and`, `as`, `await`, `break`, `catch`, `continue`, `defer`, `do`, `else`, `enum`, `error`, `fallthrough`, `fn`, `if`, `impl`, `import`, `in`, `is`, `loop`, `match`, `maybe`, `mod`, `not`, `opt`, `or`, `out`, `proto`, `raise`, `ref`, `return`, `self`, `struct`, `then`, `try`, `until`, `void`, `where`, `yield`.

[TOC](#table-of-contents)

---

*This document captures the current state of the Mulem design. The language is still evolving.*


