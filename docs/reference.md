# The Mulem Programming Language Reference

*Version 0.1 (Draft)*

__Mulem__ is a general-purpose, expression-oriented language designed to balance conciseness and eloquence. It delivers highly readable syntax, robust safety mechanisms, granular execution control, and expressive data pipelining. Supporting both interpretation and compilation, Mulem is ideally suited for systems programming, AI, and game development.

This document is not focused on when you would apply any of the mechanics described. It's just describing the mechanics themselves within the Mulem programming language.

---

## Core Design Philosophy

- **Expression-oriented**: Almost everything is an expression and returns a value.
- **Significant whitespace** with smart inline support.
- **Modern error & option handling**: `?` and `!` propagation
- **Flexible**: Multi-paradigm (functional, procedural, low-level).

## Lexical Conventions

- **Indentation**: Significant (4 spaces recommended).
- **Line endings**: Newlines (`\n`, `\r`, or `\r\n`) or semicolons `;` separate expressions.
- **Comments**:
  - Single-line: `-- comment`
  - Block: `(-- comment --)` (nesting allowed)
- **Strings**:
  - `"double quotes"` with `{interpolation}`
  - `''raw strings''`
  - `"""multi-line strings"""`

### Whitespace and Indentation

Whitespace is significant. Indentation marks where blocks begin and end. Four (4) spaces per level is recommended. The use of tabs or spaces must be the same throughout a block. 

Statements are separated by newlines or semicolons (`;`). The two are interchangeable *in most cases.* The one thing to know is that if you see a semiclon (`;`) outside of a string or comment, then the expression **always** ends. Newlines may be `\n`, `\r`, or `\r\n` and have special rules that will be explained further.

### Comments

```
-- Single-line comment.

(--
    Multi-line comment.
--)

(--
    (-- Nesting is allowed. --)
--)
```

### Lexical Categories

| Category             | Examples                                            |
|:---------------------|:----------------------------------------------------|
| Words                | `x`, `PI`, `1`, `3.14`, `0xABCDEF`,                 |
| String/Char literals | `'a'`, `"foo"`, `"""big string"""`, `''raw''`       |
| Delimiters           | `,` (tuples/arrays), `;` (expressions)              |
| Symbols              | `~!@#%^&*-+=\|:<.>/?` (excluding `--`)              |
| Brackets             | `()`, `[]`, `{}`                                    |
| Whitespace           | spaces, tabs, newlines                              |

---

## Expressions and Blocks

A program is a sequence of expressions. Each expression is separated by lines or grouped together on one line and delimitted with semicolons (`;`). Semi-colons at the end of a line are ignored.

```
expr
expr

expr; expr

expr;
expr;
```

Whitespace is significant. When expressions are sperated by lines, the indentation can't be higher unless it's the start of a new block. 

```
expr
    expr   -- Error: unexpected indentation at line 2.
```

Certain keywords and symbols such as `do` or `then` start a block when they are at the end of a line. Increased indentation is expected on the next line, and decreasing indentation exits the block.

```
do         -- Line ends with `do`, expecting a new block.
expr       -- Error: expected indentation missing at line 2.

do
    expr   -- Start of the new block.
    expr   -- Both expressions are in the same block.

expr       -- Decreasing indentation exits the block.
```

Each sequence starts a new scope. For example:

```
(do x = 2; x + 1)
```

`x` is isolated to inside the `do` sequence, and the result of the expression is `2 + 1`. 

Bracket expressions `()`/`[]`/`{}` are white-space insensitive and only delimited with commas `,` until a keyword changes the sequence type. That's what `do` is doing here. It changes the sequence type from brackets to an inline `do` expression. When a non-bracket sequence sees `,` or a closing bracket, it automatically bubbles up to the nearest bracket sequence. 

Given that `;` and `,` can both exist inside an expression, you *have* to define their relative precedence. There's no neutral option. For example:

```
(getX(), print("fetching y"); getY(), getZ())
```

Is it `getX(), print("fetching y")` then `getY(), getZ()`, or is it a tuple of `getX()`, `print("fetching y"); getY()`, and `getZ()`? To maintain syntactic clarity and eliminate operator precedence ambiguity between commas (slot separators) and semicolons (expression sequencers), `,` and `;` **cannot be mixed at the same nesting level**. You must resolve this by explicitly isolating the sequenced expressions. This can be done by wrapping one sequence in an inline `do` expression:

```
(getX(), print("fetching y"); getY(), getZ())     -- Error: unexpected character: ";"
(getX(), do print("fetching y"); getY(), getZ())  -- OK: isolated via inline 'do'
```

The `do` here says to start a sequence that's seperated by semi-colons `;`. The comma `,` ends the sequence. It's now clear that `print(…)` is only there to run side effects after `getX()`, and the value of the expression is the tuple `(x, y, z)`.

That's why the rule *"`,` and `;` cannot be mixed at the same nesting level"* is in place. It prevents the disagreement of what `(a, b; c, d)` means. `(a, do b; c, d)` makes it clear that `do b;` is a side-effect and `c` is the value for the slot. Each `;` after `do` can be read as "then" like **do** *this* **then** *that.*

### Expression Splitting

Semicolons and newlines are ignored—as far as syntax is concerned—when they are inside a multi-line comment `(-- --)` or string `"""…"""`. 

```
x = 1 + (-- New lines and semicolons ignored here;
   --) 2
```

```
s = """
    Big
    string;
    """
```

Otherwise, a semicolon always ends an expression. 

It's not always clear if `- 1` at the start of a line means *subtract 1* or *negative 1.* Some languages use a backslash (`\`) for this, but that has its downsides. With backslashes, they all go at the end of the a line which hardly ever line up without manual formatting.

```
x = 1 \
  + 2 + 3 \
  + 4 \
  + 5 + 6
```

In Mulem, you use a tilde (`~`) for this. It will treate whitespace *before* and *after* it as a single space (` `). This allows you to easily format your code anyway you prefer.

```
x = 1 ~
  + 2 + 3 ~
  + 4 ~
  + 5 + 6 ~

x = 1
~ + 2 + 3
~ + 4
~ + 5 + 6
```

Indentation is lenient with expression splitting. As long as there's a tilde (`~`) in between two tokens in an expression, then it belongs to the same expression.

```
do
    a
        ~ + b ~    -- Double is fine.
      ~ * c
        ~ - d ~
     ~ e
    ~ rem f        -- Indentation must match the block.
      ~ band g
```

It's recommended to keep the indentation the same. This is especially useful for long `if` statement.

```
if a or b
~ and c or d        -- Each `~` line is in the `if` condition.
~ and e or f
~ then              -- Block starts here.
    print("True")
else
    print("False")
```

### Block Expressions

A block wraps multiple expressions into one. Each block is a new scope. Newline after certain keywords or symbols and indentation starts a block. The last expression evaluated in a block is its value. Use `void` to leave a block empty.


```
x =
    expr
    expr    -- Value that x gets set to.

do
    expr
    expr    -- This is the block's value.

do
    void    -- Empty block.
```

__Keywords that can start a block:__

- `do`
- `then`
- `else`
- `loop`
- `try`
- `maybe`

__Symbols that can start a block:__
- `=` *(for assignment or functions)*
- *Opening brackets:* `(`/`[`/`{`

### Inline Expressions

Any block keyword or symbol is inlined when a new line is absent after it.

```
if x then "True" else "False"    -- Inline form.

if x then                        -- Block form.
    "True"
else
    "False"
```

### Inline Mode / Block Mode

- **Block mode** — indentation is meaningful, newlines end expressions
- **Inline mode** — indentation is ignored, brackets determine structure

To switch from block mode to inline mode (and vice versa):

```
(if cond then    -- Start a block inside inline expression
    expr
else
    expr
) + expr         -- Close block and continue inline expression
```

Mulem takes special care when switching between block mode and inline mode, allowing coders to format their code naturally and elegantly. No special keywords like `do…end` or curly braces `{}` are needs, and once you see it, you'll realize it's quite intuitive. 

There are 3 rules that Mulem follows:

1. Any line ending in certain keywords/symbols starts a block (significant whitespace).
2. When inside a bracket `(`/`[`/`{`, the matching bracket `)`/`]`/`}` or comma `,` ends the block and switches back to inline mode (insignificant whitespace)
3. If all brackets are closed, return to block mode at the end of a line.

```
apiFetch((result) =      -- This starts a block for the function.
    if result > 0 then   -- Whitespace is significant here.
        print("Success! {result}")
    else
        print("Failure! {result}")
)                        -- Bracket match, switch back to the inline mode.
                         -- New line ends then inline expression, switch to block mode and continue to next line.
```

Nesting works freely. You only need to worry about closing the bracket when you're done with the block. 

```
-- Complicated logic…
renderScene(
    if settings.quality == Ultra then
        try
            loadHighResAssets()!
        catch
        | AssetError(msg) then
            print("Error: {msg}")
            loadFallbackAssets()
    else
        loadFallbackAssets()
    ,
    camera: activeCamera
)
-- But it all makes sense following the 3 rules.
```

*The core idea…* Mulem let's you write code that looks like the structure it describes. Blocks of logic get indentation. Inline expressions stay on one line. The switching rules exist so you never have to fight the formatter to achieve either one.

### Pattern Splitting

A **pattern** is special type of expression. It's a sequence of enumerable members separated by vertical bars (`|`). Some keywords can start a pattern which terminates at `then`. When a pattern keyword is at the end of a line, the lines following it that start with `|` point to the same pattern sequence. This is similar to expression splitting, so we call it **pattern splitting.**

```
match expr is
| Pattern(x) then
    print("{x}")
| then        -- Wildcard
    print("no match")
```

__Keywords that can start a pattern sequence:__

- `is`
- `catch`

---

## Operators

Mulem has mix of symbolic and word-form operators. For a complete list, see [Table of Operators](#table-of-operators) at the end of this document.

Increment and decrement are the same as assignement:

```
i += 1     -- i = i + 1
i -= 1     -- i = i - 1
i *= 2     -- i = i * 2
i /= 2     -- i = i / 2
i //= 2    -- i = i // 2
s <>= "a"  -- s = s <> "a"
```

### Function Calls

Function can be called in 3 ways:

```
function(a, b, c)   -- positional
function{a, b, c}   -- named, turns into {a: a, b: b, c: c}
function x          -- spread tuple into function
```

`function x` is short for `function(&x)`, where `&` is the tuple spread operator.

### Pipelining `|>`

When placed at the start of a line in a block, the pipelining operator `|>` takes on a special meaning: it takes the value of the previous line and makes it available in the next line as `$`. This symbol is known as the **pipeline context.** This makes it easy to chain a sequence of calls and read them in order.

```
print("{
    fetchC(
        fetchB(
            fetchA()
        )
    )
}")
```

*Becomes…*

```
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

```
|> fetchA()                      -- Run fetchA,
|> do print("{$}"); fetchB(&$)   -- Print result, then fetchB
|> do print("{$}"); fetchC(&$)   -- Print result, then fetchC
|> print("{$}")                  -- Print result.
```

Note that `|>` at the beginning of a line is different from `~ |>` which is a split expression on the next line. The pipe will continue after it.

```
|> fetchA()
~ |> fetchB(&$)
|> fetchC(&$)
|> print("{$}")
```

*Is the same as...*

```
|> fetchA() |> fetchB(&$)    -- Goes back to this line.
|> fetchC(&$)                -- Pipe from the previous statement.
|> print("{$}")
```

A **pipeline block** is started with `|> do` and a new line, either at the end of a line or on its own line. Within the block, `$` holds the piped-in value within the scope of that block.

```
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

```
|> fetchA()
|> fetchB(&$)
|> fetchC(&$) as x    -- Put the result into `x`.

print("{x}")          -- Print the result.
```

This lets you extract the result of any step in a pipeline simply by appending `as name` to that line.

```
-- Put all results of each step into variables.
|> fetchA() as a
|> fetchB $ as b
|> fetchC $ as c

print("a = {a}, b = {b}, c = {c}")
```

The variable type is always inferred, to avoid ambiguity with `:`. Mutability can be specified with `mu`. *(See [Mutability](#mutability).)*

```
fetchA() as mu x |> fetchB(x) |> do   -- Create a mutable variable `x`.
    x += 1                          -- Mutate it.
    print("{x}")                    -- Print it.
```

To put the final result of any pipeline into a variable, use `|> $ as x` at the end, where `x` is any variable name. This makes it easy to mix and match the pipes in between with `|> $ as x` at the end to collect it all into a variable. 

```
|> fetchA()      -- Start.
|> fetchB(&$)    -- Pass pipeline context.
|> fetchC(&$)    -- Pass pipeline context.
|> $ as x        -- Put the pipeline context into `x`.

print("{x}")
```

One use case for a `|> do` block is to configure an object before assigning it to an immutable variable.

```
user = User.create() |> do
    $.name = "John Smith"  -- Set properties on the pipeline context.
    $.dob = "1970-01-01"
    $                      -- Return the pipeline context.

print("User: {user.name}, born: {user.dob}")  -- Prints "User: John Smith, born: 1970-01-01"
```

This gives you a great deal of flexibility in how you choose to express your code.

## Basic Bindings

There are two types of bindings: basic `=` and meta `::`. See [Meta Bindings](#meta-bindings-) below for details about `::`.

### Variable Declarations

```
x = 42          -- inferred
y: int = 42     -- explicit type
z: _ = 42       -- forced inference
mu counter = 0  -- mutable
```

Variables are declared with just the equals sign (`=`). Type is inferred, but can be declared with a colon (`:`). You can also use `: _ =` instead to declare and infer the type at the same time. This is useful for shadowing mutable variables. *(See [Mutability](#mutability).)* For now, just know that anytime you see `:` before `=`, *it always declares a new variable,* and if you see just `=`, *it's either declaring or mutating a variable.*

```
a = 0               -- Implicit declaration, type inferred.
b: int = 1          -- Explicit type.
c: _ = 2            -- Explicit declaration, inferred type.
```

Adding a new line and indentation after the `=` starts a block. The last expression evaluated in the block is the value of that variable.

```
lunch =
    if getDayOfWeek() == "Tuesday" then
        "tacos"
    else
        "sandwich"
```

Variables are immutable, but declaring it again shadows it. Any subsequent `=` on an immutable variable is an implicit declaration. Redeclaring a variable with the same name is called **shadowing.** This makes Mulem flexible while still having the advantages of being statically typed.

```
a = 1
a = 2           -- New variable, shadows previous `a`.
a = "hello"     -- Type can change when shadowing.
```

You can also shadow a variable using its previous value. 

```
i = 0
i = i + 1     -- Sets new `i` based on old `i`.
i += 1        -- Does the same thing.
```

`=` is a void statement and may not be used inside expressions that expect a non-void value. Use `==` for comparison and `as` for inline assignment.

```
-- Error:
if x = 0 then
    void

-- Correct:
if x == 0 then
    void
```

### Mutability (`mu`)

Mutable variables are declared with `mu`. Setting them later mutates the value rather than shadowing it.

```
mu count = 0
count += 1          -- Mutates count.
count: _ = 0        -- Shadows count with a new immutable variable.
```

A mutable variable may be declared without an initial value, but cannot be used until it is set.

```
mu x: int
x = 1
doSomething(x)      -- OK now.
```

Functions do not automatically capture mutable variables. Any assignment inside a function to an outer mutable variable creates a new local variable unless explicitly captured. *(See [Capturing](#capturing).)*

```
mu count = 0

addCount() % (count) =
    count += 1

addCount()
print("{count}")    -- "1"
```

### References (`ref`)

A reference points to the same memory location as another variable. Its mutability is carried over.

```
mu x = 0
ref xRef = x
xRef = 1
print("{x}")        -- "1"
```

#### Destructuring

```
(a, b) = (0, 1)                             -- Positional.
{x} = {x: 2}                                -- Named.
(a, b) & {x} = (0, 1, x: 2)                 -- Mixed.
{x as y} = {x: 2}                           -- Alias.
{x as y: int} = {x: 2}                      -- Alias with type.
{0 as a, 1 as b} = (3, 4)                   -- Positional by index.
(_, b, _) & {x, _} = (0, 1, 2, x: 3, y: 4)  -- Skip with `_`.
```

When destructuring a named type, the type may be placed after the last tuple.

```
Thing :: {x: int, y: int}
{x, y}: Thing = thing
{x, y} = thing              -- Or infer.
```

### Function Declarations

* __Basic:__ `add(a, b) = a + b`
* __Parameter Modifiers:__ `mu` / `ref` / `in` / `out` / `opt`
* __Lambdas:__ `fn(x) = x + 1`

Functions are declared by adding parentheses `()` and the name and before the colon `:` or equals sign `=`. The return type and parameter types can be either explicitly declared or inferred based on usage.

```
add(a: int, b: int): int = a + b
-- Or inferred:
add(a, b) = a + b

result = add(1, 2)
```

This keeps function declarations short and sweet. Anyone familar with algebra will be able to recognize its format and understand what it means right away.

```
f(x) = x*x + 2*x + 1
```

Functions can be a block statement by placing a new line and indentation after the equals sign `=`. This allows you to put multiple lines in one function. The last line evaluated is the return value.

```
fib(n) = 
    if n < 1 then
        0
    else if n < 2 then
        1
    else
        fib(n - 1) + fib(n - 2)
```

`fn` is a reserve word for lambda functions. Declaring a function with this name will throw an error.

```
fn(x) 
```

The arguments of a function are a **tuple.** The share the same syntax. Arguments are separated by commas `,`. Trailing commas are ignored, but leading commas and double commas are considered a syntax error. 

```
add3(a, b, c) = a + b + c
-- Also okay:
add3(a, b, c,) = a + b + c

add3(1, 2, 3,)   -- This is okay.
add(
    1,
    2,
    3,           -- Useful if you list arguments in a block.
)

(-- Uncommenting this would get an error:
add3(,1,2, ,3,,) -- This is not okay.
--)
```

Functions can also be declared with `mu` to be set later. This type is called a **function pointer.** It lets you treat functions that same way you do with variables.

```
mu action(int, int): int
add(a, b) = a + b
sub(a, b) = a - b
action = add
print("1 + 1 = {action(1, 1)}")  -- Prints "2".
action = sub
print("1 - 1 = {action(1, 1)}")  -- Prints "0".
```

Standard function overloading isn't possible because that would shadow the previous definitions.

```
f(): int = 0
f(x: int): int = x   -- Shadows previous f
f()                  -- Error: f expects 1 argument.
```

Instead, one function can be defined with multiple definitions using `match ... is`. The next patterns will have function signatures and their bodies. The first function signature to match will go. This can also work for lambda functions. 

```
safeDivide(...) = match (...) is
| (x: float, y: float if y != 0.0): float then
    x / y
| (x: float, y: float): float then
    0.0
```

#### Parameter Modifiers

| Modifier         | Behavior          | Mutable inside function?           |
|:-----------------|:------------------|:-----------------------------------|
| *(none)*         | Pass by copy      | No                                 |
| `mu`             | Pass by copy      | Yes                                |
| `in`             | Pass by reference | No                                 |
| `ref`            | Pass by reference | Yes                                |
| `out`            | Unset reference   | Yes (Must be assigned)             |
| `opt`            | Optional argument | Wraps in T?`                       |

Function parameters can be declared like variables. Likewise, you can modify their mutability and reference-ness the same way.

```
increment(ref x: int) =
    x += 1

mu y = 0
increment(y)
```

`in` and `out` are complements of each other. One is **read-only** (`in`) and the other is **write-only** (`out`). `ref` combines these two to pass a variable that you can both **read and write** to.

`out` parameters are guaranteed‑set references. They behave like `ref`, except they begin the function in an *unset* state. Use an `out` parameter when a function’s job is to produce a value rather than consume one. An `out` parameter must not remain unset in any branch of the function. It must either be:

- Assigned directly, *or*
- Passed to another function that also takes an out parameter.

This guarantees that the variable is initialized after the call completes.

```
setInt(out i): void =
    i = 3

mu x: int
setInt(x)
print("{x}")    -- Prints "3"
```

This is fine, but what if you just wanted to grab the `out` variable in one go? Normally you could just write `n = setInt()` for return values — but `out` parameters aren't retaurn values. We can inline a variable from the `out` parameter with the keyword `as` — just like how we use `as` for aliasing. This makes it clear that we're expecting a new variable at the call site. That way it's *explicit* that we're intentionally declaring a new variable instead of passing in an existing one.

```
setInt(as n)    -- Declare a new variable as `n` that gets set by `setInt`.
print("{n}")    -- Prints "3"
```

`setInt(as n)` → "Set int as n"—it says exactly what it does. This makes it clear that you're intentionally making a new variable. Using `as` makes the intent unambiguous: you are creating a variable, not passing an existing one. `as` already means "bind this to a name" throughout the language:

```
{x as y} = {x: 2}                 -- destructuring
match choice is Second(x as val)  -- pattern matching
method(self as this)              -- self aliasing
setInt(as n)                      -- out parameter
```

Languages that use return values for this kind of thing (`n = setInt()`) imply the value comes out of the function through the normal return channel, which is misleading when the mechanism is actually a reference parameter. `setInt(as n)` makes the call-site declaration explicit without requiring you to pre-declare a `mu` variable just to hand it in.

### Optional Parameters

Parameters can be made optional with the `opt` modifier. This wraps the variable in type `T?`. If the parameter is missing, it will be set to `None`.

```
addOptional(opt a: int, opt b: int): int =
    aVal = a ?: 0    -- Coalesce optional arguments with default value 0.
    bVal = b ?: 0    -- This unwraps their value if they exist or set them to 0.
    aVal + bVal      -- Add the unwrapped values.

print("{addOptional()}")      -- Prints "0"
print("{addOptional(1)}")     -- Prints "1"
print("{addOptional(1, 1)}")  -- Prints "2"
```

When calling a function with an explicit `T?`, you give it a value of `T?`. When calling a function with `opt`, you give it a value of `T`. 

```
optionalParam(opt val: int) =
    if val is Some(x) then
        print("Some({x})")
    else
        print("None")

requiredParam(val: int?) =
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

```
addOptional(opt a = 0, opt b = 0): int = a + b     -- `a` and `b` are always `int`s

print("{addOptional()}")      -- Prints "0"
print("{addOptional(1)}")     -- Prints "1"
print("{addOptional(1, 1)}")  -- Prints "2"
```

Use `...` to collect all arguments into a single variable. The variable should be type `T#` (an array).

```
addAll(...nums: int#): int =
    mu sum: int = 0
    loop n in nums:
        sum += n
    sum

print("{addAll()}")         -- Prints "0"
print("{addAll(1)}")        -- Prints "1"
print("{addAll(1, 2)}")     -- Prints "3"
print("{addAll(1, 2, 3)}")  -- Prints "6"
```

```
-- With pattern matching:
addAll(...nums: int#): int =
    match nums is
    | []            then 0
    | [x]           then x
    | [x, ...rest]  then x + addAll(...rest)
```

A name is optional after `...`. You can use the symbol by itself to pass it to another function or itself in a functional loop. 

```
addAll(...) = match (...) is
| (x: int, ...): int then
    x + addAll(...)
| (x: int): int then
    x
| (): int then
    0

logAndAdd(msg: str, ...) =
    print("{msg} {addAll(...)}")

logAndAdd("Sum =")           -- Prints "Sum = 0"
logAndAdd("Sum =", 1)        -- Prints "Sum = 1"
logAndAdd("Sum =", 1, 2)     -- Prints "Sum = 3"
logAndAdd("Sum =", 1, 2, 3)  -- Prints "Sum = 6"
```

- `(match)` — dispatch on arguments, `|` arms follow
- `(...)` — accept and forward whatever arguments received to the next function

#### Capturing

Immutable variables can be captured without any issue. They can't change, so they can't affect the output of the function. And if you try to set an immutable variable within a function, it will only get shadowed within the scope of the function. This includes capturing other functions which are also immutable by default.

**By default, functions cannot capture mutable variables.**

This is an intentional decision for better safety in Mulem. Functions don't automatically capture mutable variables. Instead, any variable set inside a function is treated like a new variable. This helps prevent accidentally mutating a variable that you didn't mean to and encourage good functional programming practices.

```
x = 1

addFromX(y) = x + y
addFromX2(y, z) = addFromX(y) + z

cannotChangeX(newX) =
    x = newX
    print("{x}")

cannotChangeX(2) -- Prints "2"
print("{x}")     -- Prints "1"
```

To capture a mutable variable, write `~` at the end of the function signature. This goes after the parameters and before the return type `: T =`. List each *mutable* (`mu`) variables that the function uses. This helps make it easy to see which functions depend on mutable variables and which ones don't since `~` doesn't appear in type notation or anywhere else to the left of the equals sign `=` in function signatures except for captures. 

This follows the same practice that `import` and `inherit` where all words in a given scope are listed out clearly so that there are no accident name collisions or hidden gotchas.

```
amount = 1               -- Immutable variable, doesn't need to be captured.
mu count = 0             -- Mutable variables, must be captured with `~`.
mu squared = 1
mu cubed = 1
   
addCount() % (count, squared, cube): int =     -- Capture 3 variables at once.
    count += amount                            -- Mutate captured variables inside the function.
    squared = count * count
    cubed = squared * count

addCount()
addCount()
addCount()
print("{count}, {squared}, {cubed}")     -- Prints "3, 9, 27"
```

Error messages will highlight cases where someone would be confused about `%` in a function signature:

**Forgot to capture a mutable variable:**
```
mu count = 0
addCount() =
    count += 1    -- Error here
```
> `count` is mutable but not captured. Did you mean `addCount() % (count) =`?

**Accidentally tried to capture an immutable:**
```
x = 1
f() % (x) =
    x + 1
```
> `x` is immutable and is captured automatically — remove `% (x)` from the signature.

**Captured a variable that doesn't exist in scope:**
```
f() % (ghost) =
    ghost + 1
```
> `ghost` is not defined in the enclosing scope. Captures must refer to mutable variables in the outer scope.

**Mutated without capturing, inside a lambda:**
```
mu count = 0
forEach([1,2,3], fn(x) =
    count += x
)
```
> `count` is mutable but not captured by this lambda. Did you mean `fn(x) % (count) =`?

#### Lambda Functions

Define a function within an expression with the keyword `fn` in the pattern `fn(x) = x`. This is useful for passing functions to other functions.

```
(fn(arg) = expr)

(fn(arg) =
    body
)
```

```
map(array, action) = [...loop x in array then action(x)]
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

```
otherAction(fn callback(val) =
    if val > 0 then
        print("{val}")
        callback(val - 1)
    else
        print("done")
)
```

Capturing also works inside lambda functions just like with named functions.

```
mu count = 0
forEach([1, 2, 3, 4], fn(x) % (count) =
    count += x
)
```

#### Curried functions

When a function returns another function, list each function parameters as the return type. Optionally, you can just let the return type be inferred.

```
curriedFn(a: int): (int): (int): int = _                 -- Immutable declaration
mu curriedFnPtr(int): (int): (int): int = curriedFn      -- Mutable declaration. 
```

Normally, functions can have an implicit return, but this poses a problem for curried functions…

```
curriedFn(a: int): (int): (int): int = 
    fn(b: int) =          -- Is this a lambda function or a function declaration named `fn`?
        …
```

`fn(x) = ` is a lambda expression, but `name(x) = ` is a function declaration. Since `name(x) = ` is a function declaration, Mulem would interpret `fn(b: int) = ` as declaring a local function named `fn` — which then fails because `fn` is reserved. If you try to declare a function with the name `fn`, Mulem will throw an error to prevent it.

```
(-- 
fn(a, b) = a + b  -- Error: `fn` is a reserved keyword and cannot be used as a name
fn(1, 2)          -- Error: `fn` is a reserved keyword and cannot be used as a name
--)
```

If Mulem could implicitly return lambda functions, that wouldn't solve the issue, say you have a function like this:

```
curryFn(a: char) =
    print("In function 1: {a}")
    fn(b: char) =
        print("In function 2: {b}")
        fn(c: char) =
            print("In function 3: {c}")
```

The problem is `fn(b: char) =` looks like a function definition. If you typo'd `fn`, you would get silent errors:

```
curryFn(a: char) =
    print("In function 1: {a}")
    fun(b: char) =           -- Declare a function named `fun` and never use it. This function now returns `void`.
        print("In function 2: {b}")
        fn(c: char) =
            print("In function 3: {c}")
```

To solve this issue, you either must wrap the function in parentheses or explicitly say `return fn` in order to curry functions. This ensures that `fn(x) =` is only used in expressions and not accidentally declaring a new function. 

```
curryFn(a: char): (char): (char): void =
    print("In function 1: {a}")
    return fn(b: char): (char): void =          -- Note: types can be inferred, but it's written out for demonstration purposes.
        print("In function 2: {b}")
        return fn(c: char): void =
            print("In function 3: {c}")
-- or --
curryFn(a: char): (char): (char): void =
    print("In function 1: {a}")
    (fn(b: char): (char): void =
        print("In function 2: {b}")
        (fn(c: char): void =
            print("In function 3: {c}")
        )
    )

curryFn('a')('b')('c')
(-- Prints:
"In function 1: a"
"In function 2: b"
"In function 3: c"
--)
```

There can be a lot of indentation when you curry functions like this. Is there a way we could write this more elegantly? *Yes!*

```
curryFn(a: char): (char): (char): void =
    print("In function 1: {a}")
    (b: char) = fn: (char): void              -- Parameters go on the left, return goes on the right.
    print("In function 2: {b}")
    (c) = fn                                  -- Types can also be inferred.
    print("In function 3: {c}")

curryFn('a')('b')('c')             -- Works the same.
(-- Prints:
"In function 1: a"
"In function 2: b"
"In function 3: c"
--)
```

Now all the functions line up together, and the next parameters resemble normal assignment. Each `() = fn` is an explicit suspension point from the function similar to `yeild` or `await`. The remaining part of the function becomes the body of the next function. 

When capturing variables, each returned function needs to capture them seperately.

```
mu count = 0
curryAddCount(a: int) % (count): (int): (int): int =
    count += a                                   -- (1) Evaluated immediately
    (b: int) = fn % (count): (int): int   -- (2) Suspends and captures `count`
    count += b                                   -- (3) Evaluated when second `fn` is called
    (c: int) = fn % (count): int
    count += c
    count

print("{ curryAddCount(1)(2)(3) } == { count }")   -- Prints "6 == 6"
```

If a curried function takes no parameters, put an empty tuple `()` on the left. 

```
curryLoop() =
    loop
        print("I'm looping!")
        () = fn   -- Suspend function.

next = curryLoop()       -- Prints "I'm looping!"
next = next()            -- Prints "I'm looping!"
next = next()            -- Prints "I'm looping!"
next = next()            -- Prints "I'm looping!"
```

#### Named Parameters

```
add{ a: int, b: int }: int = a + b
add(b: 1, a: 2)             -- Order doesn't matter.
```

Mixed positional and named.

```
add(x: int) & {a: int, b: int}: int = x + a + b
add(1, a: 2, b: 3)
add(a: 2, 1, b: 3)          -- Named params can go anywhere.
```

Positional parameters may be set by number at the call site.

```
isGreaterThan(a, b) = a > b
isGreaterThan(0: 10, 1: 5)  -- a=10, b=5 → True
isGreaterThan(1: 10, 0: 5)  -- a=5,  b=10 → False
```

---

## Types

Notation:

- **Basic**: `x: T`
- **Functions**: `f(x: T): T`
- **Maybes**: `T?`
- **Results**: `T!` or T!E` where `E` is an `error` type
- **Arrays**: `T#` or `T#N` where `N` is the length
- **Multi-dimensional Arrays**: `T##`, an extra `#` for each dimension, each dimension can be fixed or dynamic: `T#N#`, `T##N`, `T#N#N`, `T#N##`, etc.
- **Dictionaries**: `T#U`
- **Inferred**: omit the annotation entirely

### Built-in Types

Some built-in types include `byte`, `int`, `uint`, `float`, `bool`, `char`, `str`, and `ptr`. Note that although built-in types use lowercase names, they are not *keywords*. This is just a naming convention. *It's recommended that users create custom types with capitalized names to differentiate from built-in types.*

```
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

```
x = 0
y: typeof[x] = 1    -- Ensures that x and y have the same type.
```

You can also get the default value of any type with the compile-time function `default`. The type needs to have a default value defined which is yet to be determined how, but they're already defined for basic types.

```
x = default[byte]   -- == 0y
x = default[int]    -- == 0
x = default[float]  -- == 0.0
x = default[bool]   -- == False
x = default[char]   -- == '\0'
x = default[str]    -- == ""
x = default[ptr]    -- == Null
```

`default[]` can also infer the type. This can be useful in certain situations, like if you want to leave a function that returns something empty so that you can implement it later.

```
implementLater(): int = default[]
```

You can also get the size of any type with the compile-time function `sizeof`. It returns a constant `uint` (unsigned integer) with the number of bytes of memory that type requires. The exact sizes of some types like `int` or `float` might vary, but you can rely on `byte` and `bool` being 1 byte each. There's also the `void` type which represents no data. `ptr` depends on the pointer size of the system. 

```
sizeOfChar = sizeof[byte]   -- == 1
sizeOfBool = sizeof[bool]   -- == 1
sizeOfVoid = sizeof[void]   -- == 0
sizeOfPtr  = sizeof[ptr]    -- == 4 or 8
```

#### Booleans

`bool` is a built-in enum type with its only members being `False` and `True`. This means you can also pattern match with a bool, although it's recommended to use `if`/`else` instead. Enum-members are capitalized by convention. 

```
match value is
| True then
    print("It's true!")
| False then
    print("It's false!")
```

These two are the same thing.

```
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

Another rule is that if there's a period in a place where a decimal is expected, then it's part of the number literal and not `.` for member/component access. This is space sensitive.

*What the tokenizer sees:*

- `1.0` is a float `1.0`.
- `1. 0` is a float `1.` followed by int `0`.
- `1 .0` is an int `1` followed by access `.` to its `0`th component.
- `1.0.0` is a float `1.0`  followed by access `.` to its `0`th component.

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

```
a = 'a'
b = a + 1
print("{b}")   -- Print "b", the letter after 'a'
c = b + 1
print("{c}")   -- Print "c", the letter after 'b'
```

Characters can be escaped with a backslash `\` between the quotes. Some letters have special values like `\t` for tabs, `\n` for new lines, etc. All the standard stuff you would expect from a modern language. 

```
apostrophe = '\''
tab = '\t'
newLine = '\n'
nullChar = '\0'
unicode = '\uFFFF'
```

#### Bytes

Bytes `byte` are 8-bit unsigned integers. You can write a byte literal by putting `y` at the end of an integer or by putting @ in front of a `char` literal. It must be in the range of 0 to 255 (inclusive).

```
min = 0y
max = 255y
a = @'a'
```

#### Strings

Strings (`str`) are immutable 1D arrays of characters `char`. 

|    Form     | Purpose                                                                    |
|:-----------:|:---------------------------------------------------------------------------|
|    `"…"`    | Regular string with `{expr}` interpolation                                 |
|  `"""…"""`  | Multiline with interpolation; whitespace trimmed at closing `"""` position |
|   `''…''`   | Inline raw string, no interpolation or escaping                            |
| `@"""…"""@` | Multiline raw string; `@` count must match to close                        |

Strings are marked with quotation marks (`"…"`) *(also called double quotes)* and can be formatted with curly braces (`{expr}`) in the string. Use a backslash to write a literal opening curly brace (`\{`). Note that string insertion and named tuples both use curly braces. This shouldn't be an issue though since one is inside strings and the other isn't. Inserted expressions are implicitly converted to strings, so using `str()` isn't necessary. This string is only allowed on a single-line, but literal-line characters can be inserted with `\n`.

```
name = "world"  
hello = "Hello, {name}!"
helloEscaped = "Hello, \{name}!"
lines = "This \n string \n has \n linebreaks."
```

Subsequent string literals will automatically concatenate, and the `<>` operator can be used to concatenate non-literal strings.

```
str1 = "This" " string"
str2 = " is broken"
str3 = str1 <> str2 <> " into multiple parts."
print(str3)
-- Prints "This string is broken into multiple parts."
```

Write multi-line strings with `"""` (3 quotation marks). A common issue in programming languages is how to fix the issue of leading whitespace in a multi-line string. Mulem uses significant whitespace, so unindenting the string wouldn't work. We don't want all the leading whitespace to be in the string, but how do we solve this? To fix this issue, whitespace gets trimmed at compile-time based on the positions of the last `"""`. Any spaces before it is automatically trimmed. Much like blocks, having too little indentation is a syntax error inside multi-line strings. This helps keep things readable and consistent and solves the whitespace issue inside strings.

```
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

```
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

```
rawString = ''It's okay to put an apostrophe (') in the string.''
filePath = ''C:\files\on\windows.txt''
template = ''Insert here → {{variable}}''
```

To make a multi-line raw string, add an at sign `@` before and after triple quotation marks `"""`. The number of `@`s must match to close the raw string. 

|  Opening | Closing  |
|---------:|:---------|
|   `@"""` | `"""@`   |
|  `@@"""` | `"""@@`  |
| `@@@"""` | `"""@@@` |

*etc…*

```
-- Add a `@` to escape the `"""` within the string.
bigDocument = @"""
    This  "
   is   """      """
  all       """
  in  "         "
   a     """" "
    string
"""@
-- Matching number of `@` closes the string.

-- `@@"""` to escape the `"""@@` within the string.
nestedDocument = @@"""
bigDocument = @"""
    This  "
   is   """      """
  all       """
  in  "         "
   a     """" "
    string
"""@      
"""@@
-- `"""@@` closes the matching `@@"""`.
```

Sometimes, we just want to copy and paste string data without formatting it. We can take advantage of the fact that brackets start as whitespace insensitive. All whitespace and characters are put into the string without formatting until it gets to the matching `"""@` marker and closing bracket. Then, re-ident in the next line to resume the block. This is useful for debugging and embedding data in code.

```
do
    do
        do
            do                       -- Deeply nest block.
                nestedRawString = (  -- Put raw string on the next line without indenting.
@"""                       
                       
    Indentation  
  doesn't matter     
 here.              

"""@
                )                    -- Re-indent to return to the block.
                print("{nestedRawString}")
```

What it prints:

```
                       
                       
    Indentation  
  doesn't matter     
 here.              
                  
   
```

`@"""…"""@` inherits the `"""` closing-position trim anchor, so if you *do* want controlled indentation trimming, you still get it from where you place the closing `"""@`. That makes the two systems consistent with each other and interchangeable.

```
do
    do
        do
            do                       -- Deeply nest block.
                nestedRawString =
                    @"""                       
                                           
                        Indentation  
                      doesn't matter     
                     here.              
                    
                    """@
                print("{nestedRawString}")
```

#### Arrays

Array types are declared with the hash symbol (`#`). This was chosen because the `#` is commonly used for numbers. For example, `#1` is read `number 1`. A number after the `#` makes it a fixed length array `type#N`. Arrays are statically sized when written `type#N`; `type#` is the dynamic form. Items are separated with commas (`,`). Index is done with the `#[]` operator. When the index is a number literal, you can omit the `[]`, similar to `.` (member access) for tuples.

```
list: int#4 = [1, 2, 3, 4]
print("length of list: {len(list)}")
compressedList = [list#0 + list#1, list#2 + list#3]
doubleArray: int#2#2 = [[1, 2], [3, 4]]
item = doubleArray#1#0    -- The 2nd row, 1st column
print("{item}")           -- Prints "3"
```

This builds on the visible symmetry between type notation and their value expressions:

| Type | Notation | Expression |
|:---------|:------:|:--------:|
| **Results**  | `T!`  | `x!`  |
| **Maybes**   | `T?`  | `x?`  |
| **Pointers** | `T^`  | `x^`  |
| **Arrays**   | `T#N` | `x#n` |

Accessing directly with a number literal `#N` gives you a type `T`. To access with a variable, you must use `#[n]` which returns a maybe type `T?`. This you must unwrap this with `maybe` and/or one of the maybe-type operators: `?`, `?:`, `?.`, `?#`.

```
i = 2
print("{ list#[i] ?: 0 }")   -- Prints "3" because list#2 is 3
```

In general, you'll mostly be using arrays by iterating or piping them. 

```
loop x in list then
    print("{x}")     -- No need to use `#`
```

At least one operand in a chain of `<>` operations must be an array `T#`. Other operands can be type `T#` or `T`. If all are type `T`, you can put an empty array `[]` in the chain start a new one. 

```
a = 1 <> 2 <> 3 <> 4 <> []  -- == [1, 2, 3, 4]
b = 0 <> a <> 5             -- == [0, 1, 2, 3, 4, 5]
c = b <> 6 <> 7 <> 8        -- == [0, 1, 2, 3, 4, 5, 6, 7, 8]
```

`<>` is either right or left associative depending on the type on the left-hand side.

- `T# <> T#` – left associative
- `T# <> T` – left associative
- `T <> T#` - right associative
- `T <> T` - right associative

Because any `lhs <> rhs` returns `T#`, the chain becomes left-associative at the first `T#` operand. The example gets parsed like this:

```
a = ( 1 <> ( 2 <> ( 3 <> ( 4 <> [] ) ) ) )     -- right associative for whole chain
b = ( 0 <> ( a <> 5 ) )                        -- left associative, then right associative
c = ( ( ( b <> 6 ) <> 7 ) <> 8 )               -- left associative for whole chain
```

Use the spread operator `...` to spread an array into another array. This must be the first prefix operator in an expression, and it must be in a compatiable array or tuple literal. It always goes last in the slot's expression, so extra parentheses aren't necessary: `...a <> b` == `...(a <> b)`.

```
a = [1, 2, 3]
b = [0, ...a, 4]                -- == [0, 1, 2, 3, 4]
c = a <> b                      -- == [1, 2, 3, 0, 1, 2, 3, 4]
d = [0, ...a <> b, 5, ...c, 6]  -- == [0, 1, 2, 3, 0, 1, 2, 3, 4, 5, 1, 2, 3, 0, 1, 2, 3, 4, 6]
```

If you spread an array into a tuple, the type must be known at compile-time and the tuple must have compatible components. Positional components will map to array indexes or iterator yields based on where the spread is placed inside the tuple. If the tuple runs out of space, the spread will be truncated. This works similar to variadic parameters in functions.

```
ThreeInts :: (int, int, int)

list = [1, 2, 3]
a: ThreeInts = (...list)      -- == (1, 2, 3)
b: ThreeInts = (0, ...list)   -- == (0, 1, 2), truncated at the end
```

Tuples may also collect any remaining positional components into an array, just like variadic parameters in functions.

```
TwoOrMoreInts :: (int, int, ...int#)

list = [1, 2]
a: TwoOrMoreInts = (...list)              -- == (1, 2, [])
b: TwoOrMoreInts = (0, ...list)           -- == (0, 1, [2])
c: TwoOrMoreInts = (-1, 0, ...list)       -- == (-1, 0, [1, 2])
d: TwoOrMoreInts = (-2, -1, 0, ...list)   -- == (-2, -1, [0, 1, 2])
```

*Mulem separates tuple and array spread to preserve =type safety and avoid runtime surprises. `&` always preserves tuple structure; `...` always preserves array structure.*

#### Dictionaries

Dictionaries are a subtype of arrays. Instead of numbers, each item is given a **key.** A dictionary's type is the type of the value `V` and the type of the key `K` join with a hash `#` in between: `V#K`. This makes it semantically clear that they are a subtype of arrays. Dictionaries also use the same operator to access items. The type passed to the `#` operator must match the key type. Each key is marked with `[]:` in the array.

```
dict: float#float = [
    [1.0f]: 10.0f,
    [1.5f]: 15.0f,
    [2.0f]: 20.0f,
]
print("{ dict#1.5f }")   -- Prints "15.0"
```

If the key type is `str` and a key is a valid variable name, then the square brackets before `:` can be omitted. You can access it like a member with `.` but with `#`. This makes it easier to distinguish compile-time access `.` and run-time access `#`. 

```
dict: int#str = [
    a: 1,
    b: 2,
    c: 3,
    ["invalid name"]: 127,
]
print("{ dict#["b"] ?: 0 }")   -- Prints "2"
print("{ dict#"b" }")          -- Prints "2",
```

You can iterate through a dictioary like with arrays. This is the recommend way of using dictionaries. 

```
loop (val, key) in dict then
    print("{key} = {val}")
```

#### Pointers

Although most things can be achieved without manual manipulation of pointers, some low level code requires it. Opaque pointers use the type `ptr`. This represents a pointer where the type that it represents is unknown. It's ideal for FFI where you need to pass a pointer a around and let an external library handle it. 

```
result: ptr = ExternalLib.getSomething()
ExternalLib.doSomethingWith(result)
```

You can manually check if the pointer is `Null` and give a helpful error message in scripts. `Null` is a `ptr` type that points to the NULL pointer. It's distinct from `None` which is an varied maybe type `T?`. 

```
result: ptr = ExternalLib.getSomething()
if result == Null then
    print("No result found")
    raise NotFound
```

Sometimes, it's necessary to dig deep into the unsafe territory. The `^` is the symbol associated with pointers, analogues to `?` for maybes, `!` for results, and `#` for arrays. It can be used in type notation, but it's also the operator to dereference a pointer. Thy type must be known at compile-time. Dereferencing an opaque pointer `ptr` is a compile-time error. In the type notation, `T^` prevents the pointer from mutating its memory or `T^mu` allows mutation with `^ =` (dereference + assignment). 

```
mu x = 0                 -- `ptr` type takes a reference and creates a generic pointer.
xPtr: int^mu = ptr(x)    -- Convert `ptr` to `int^mu`, type is known.
xPtr^ = 1                -- Mutate the memory.
print("{xPtr^}")         -- Prints "1".
print("{x}")             -- Prints "1".
```

Pointer types have 2 kinds of mutability: one for the reference, and one for the pointer variable itself. Here is a table of each kind and what it means.

|     Type     | What It Means                         | Can reassign pointer | Can mutate memory | *Think…* |
|:------------:|:--------------------------------------|:--------------------:|:-----------------:|:---------|
|    `x: T^`   | immutable pointer to immutable memory |       **No**         |      **No**       | *This will never change.* |
|    `x: T^mu` | immutable pointer to mutable memory   |       **No**         |        Yes        | *Like a more low-level `ref`.* |
| `mu x: T^`   | mutable pointer to immutable memory   |         Yes          |      **No**       | *I need to switch what I'm looking at.* |
| `mu x: T^mu` | mutable pointer to mutable memory     |         Yes          |        Yes        | *I need full control.* |

---

## Control Flow

* [__`do`__](#do): Creates a scoped block. Can be labeled (`do:label`).
* [__`if` / `else`__](#if--else): `if x > 0 then x else -x`
* [__`loop`__](#loop): Universal looping keyword.
  * _Unconditional:_ `loop _`
  * _While:_ `loop cond then _`
  * _For-each:_ `loop x in array then _`
  * _Do-while-not:_ `loop _ until cond`
* [__`match` / `is`__](#match--is): Pattern Matching
* [__`try` / `catch`__](#try--catch): Resolves Results (`T!`).
* [__`maybe` / `else`__](#maybe--else): Resolves maybes (`T?`). If any `?` inside fails, the block short-circuits.

All branching constructs share the same block / inline pattern:

```
-- Block form
keyword subject then
    body
keyword
    body

-- Inline expression form
keyword subject then expr
keyword expr
```

`then` is used to separate a subject and expression when a keyword block is in-lined. 

**Any variables created in the subject field shadow any variables in the parent scope.** This prevents accidental mutations and unintended side-effects. 

#### `do`

```
do expr; expr

do
    body

do:label
    body
```

Creates a new scope. Its value is the last expression evaluated.

```
x =
    do
        y = 1
        y + 1       -- block's value is 2
```

Any starting block can be given a label with `:label` after its starting keyword. Use this to call `break` on a specific label. 

```
do:label
    break:label
```

Inline `do` will start a sequence of expressions separated by semicolons `;` that ends at a new line or (when inside a bracket) at a comma or closing bracket. The last expression in that sequence is the value of that slot. For example, in the example shown earlier:

```
(getX(), (print("fetching y"); getY()))
```

We can also write it as this:

```
(getX(), do print("fetching y"); getY())
```

Adding `do` makes it's clear that `;` is connected to `do` and not the parentheses. You can read it as **do** `print("fetching y")` **then** `getY()`.

#### `if` / `else`

```
if cond then expr else expr
if cond then
    body
else
    body
```

Basic Boolean branching.

```
x = if x > 0 then x else -x

if x > 0 then
     print("positive")
else
    print("non-positive")
```

Use `or`/`and` to compare multiple booleans at once.

```
a = True
b = False

if a and b then           -- True and False == False
    print("This will not print")
else if a or b then       -- True or False == True
    print("This will print")
```

#### `loop`

The universal loop keyword. All loop forms share the same `loop` keyword.

```
-- Unconditional (break manually):
loop
    body
    break

-- While condition is true:
loop cond then
    body

-- For-each:
loop x in expr then
    body

-- Do-until (runs at least once):
loop
    body
until cond
```

`else` after `loop cond` runs if the loop body never executed.

```
loop False then
    void
else
    print("Never ran")
```

When the parser encounters `loop`, it enters a *loop subject state.* Semicolons encountered in this state do not terminate the statement; they serve as delimiters for the step expressions. This state remains active until the parser matches the loop's body terminator (`then` or `until`).

When you have one or more semicolons `;` in the subject of a `loop` before `then`, the first in the sequence will be the loop's subject field (such as a condition or `in` expression) and the other expressions will run at the end of each iteration of the loop. 

```
-- The Dangerous Way
mu i = 1
loop i <= 100 then
    if i rem 10 == 0 then
        print("{i}!!!")
        continue
        -- OOPS! We forgot to do `i += 1` before continuing. 
        -- Infinite loop on i = 10!
    print("{i}")
    i += 1

-- The Safe Way
mu i = 1
loop i <= 100; i += 1 then
    if i rem 10 == 0 then
        print("{i}!!!")
        continue    -- Automatically triggers `i += 1` before checking condition again
    print("{i}")
```

```
-- Track index of `loop / in`
mu idx = 0
loop item in inventory; idx += 1 then
    print("Slot {idx}: {item}")
```

Because `do` blocks isolate scopes and inline expressions sequence seamlessly, you can combine `do` and `loop` to create a traditional, strictly scoped counter loop without requiring a distinct `for` keyword:

```
-- C-style for loop
do mu i = 1; loop i <= 100; i += 1 then
    if i rem 10 == 0 then
        print("{i}!!!")
        continue
    print("{i}")

-- 'i' is automatically out of scope and cleaned up here
```

Inlined `loop x in` returns a lazy iterator collected with `...`.

```
doubled = [...loop x in list then x * 2]
```

Destructuring works in loop variables.

```
loop (x, y, z) in listOfTuples then
    print("{x}, {y}, {z}")
```

Pattern matching works. All patterns must have fallbacks. *(See [Pattern Fallback](#pattern-fallbacks).)* This is because if you have an array/iterator of enums, it would be hard to determine if they're all a particular pattern. This ensures any mismatches are handled inside the loop. 

```
loop Pattern(opt x) in listOfPatterns then
    if x is Some(x):
        print("Found match: {x}")
```

This can be combined with `maybe` to automatically skip when there's a mismatch.

```
loop Pattern(opt x) in listOfPatterns then
    maybe print("Found match: {x?}")
```        

#### `break` / `continue`

Both accept an optional label to target an outer loop.

```
loop:outer x in 0...100 then
    loop:inner y in 0...100 then
        if x * y >= 100 then
            break:inner
        if x * y == 77 then
            break:outer
```

Pass a value to `break`, that becomes the value of the block, analogous to `return` for function.

```
x = do:block
    if cond then
        break:block 5
    4

print("{x}")     -- Prints either "4" or "5"
```

### Pattern Matching

```
match expr is
| Pattern1(x) then expr
| then expr
```

The next control flow methods are based on pattern match. Generally, you see the word `is`, you next thing to expect after it is a pattern: `value is Pattern(x)`.

#### `match` / `is`

Enum/error branching. Exhaustive by default. `| then` for the default case.

```
match expr is
| ptrn then
    body
| ptrn then
    body
| then
    body

match expr is (ptrn: expr | ptrn: expr | expr)
```

When inline, the patterns after `is` need to be in parentheses and `then` is replaced with `:`.

```
-- Simple value mapping
color = match status is (
    | Ok  : "green"
    | Err : "red"
    |       "gray"
)

-- Inside another expression
result = process(data) |> match $ is (
    | Success(v) : v * 2
    | Failure(e) : do print("Failure: {e}"); 0
)
```

The patterns map to the type passed in after `match`, so you only need to reference the members of that type in each pattern.

```
match choice is
| First then               -- Each pattern can case starts its on block.
    print("First")         -- Ident for the new block.
| Second(x) then           -- Continue this for each case.
    print("Second({x})")   -- ……
| Third{val} then          -- ……
    print("Third \{ val={val} }")
                           -- All choices were exhausted, so no `| then` is necessary.
```

```
restult = match x is (Ptrn1: 5 | Ptrn2: 6 | 7)
--
message = match e is (OpenError{filename}: "Open error: {filename}" | "Unknown error")
```

You can have multiple patterns match to one case. If any of the patterns destructure with a variable, the same variable name and type must be in all patterns. If not, use a wildcard `(_)` in each pattern or omit the data part entirely to disable destructuring. Otherwise, use a fallback in the pattern.

```
match choice is
| First then
    print("First")
| Second(val) | Third{val} then          -- `val` must be in all patterns
    print("Second or Third, val={val}")
```

```
-- `First` doesn't have any values, so destructuring must be disabled.
match choice is
| First | Second | Third then
    print("First, Second, or Third")
```

```
-- Fallback, `val` is converted to maybe type `T?`:
match choice is
| First | Second(opt val) | Third{opt val} then
    print("First, Second, or Third: {maybe val? else "None"}")
```

#### `fallthrough`

Proceeds to the next case, which must not destructure new values, unless fallbacks are used.

```
match choice is
| First then
    print("First")
    fallthrough
| Second(opt x) then    -- `opt x` in pattern wraps the variable in a maybe
    if x is Some(x) then
        print("Definitely Second: {x}")
    -- Implicit break.
| then
    print("No match")
```

Can also be used on `if`/`else` blocks. Doing so will go to the next `else` or `else if` block.

```
if state == Initialize then
    setup_memory()
    fallthrough
else if state == Ready then
    start_processing()
```

```
render_flags = if quality == Ultra then
    enable_raytracing()
    fallthrough
else if quality == High then
    enable_dynamic_shadows()
    fallthrough
else
    enable_base_geometry() -- The final function evaluates to the return value
```

```
-- Assume 'level' is an enum: Debug, Info, Error
loop level in log_queue then
    if level == Debug then
        print("[DEBUG] Tracking internal memory pointers...")
        fallthrough
    else if level == Info then
        print("[INFO] System heartbeat normal.")
        fallthrough
    else
        print("[WARN/ERROR] Event logged to persistent storage.")
```

#### Pattern Fallback

If a pattern can't be **guaranteed** for any reason, then you must have a **fallback.** There are two options available:

- __Optional binding:__ `Pattern(opt x)` — wraps `x` in type `T?`, `Some(x)` if it matched, `None` if it didn't
- __Default value:__ `Pattern(opt x = default)` — `x` is type `T`, if it didn't match `x` is set to `default`

#### Pattern Guards

Add `if` inside a pattern to conditionally match.

```
match choice is
| Second(x if x > 0) then
    print("Positive: {x}")
| Second(x if x < 0) then
    print("Negative: {x}")
| Second(x) then
    print("Zero")
| then
    print("No match")
```

#### `is` / `then`

```
expr is ptrn then expr
```

Extract a single binding inline. Requires a guaranteed match or optional bindings.

```
-- Guaranteed match (exhaustive type):
result = (value is Pattern(x) then x)
```

```
-- With fallback (non-exhaustive):
result = (value is Pattern(opt x) then x)                              -- Get wrapped Some(x) or None
result = (value is Pattern(opt x) then maybe x? else "fallback")       -- Unwrap with fallback
result = (value is Pattern(opt x = "fallback") then x)                 -- Automatic fallback
```

```
-- Multiple bindings:
result = (value is Pattern(opt x = 0, opt y = 0) then (x, y))
```

```
-- Arbitrary expression over bindings:
result = (value is Pattern(opt x = 0, opt y = 0) then x + y)
```

Pairs naturally with pipelining.

```
getValue()
|> ($ is Pattern(opt x = "fallback") then x)
|> doSomethingWith($)
```

#### `if` + `is`

```
if expr is ptrn then expr else expr
if expr is ptrn then
    body
else
    body
```

Combines the conditional branching of `if` with pattern matching of `is`. Useful if you want to destructure a single case of a sum type. This must be a pattern that matches the type of the value before `is`.

```
if value is Pattern(x) then
    print("value is {x}")
else
    print("value doesn't match")

something = if value is Pattern(x) then x else "fallback"
```

Like with other patterns, multiple patterns can be checked for at once with `|`. All patterns must go on the right of `is`. Any destructured variables must match in name and type.

```
if value is Pattern1(x: int) | Pattern2{data as x: int} then
    print("{x}")

if value is
| Pattern1(x: int)
| Pattern2{data as x: int} then
    print("{x}")
```

`x if` is also available like before and follows the same rules. You can iteratively match nested enums. 

```
if nestedPattern is Pattern(Pattern(Pattern(Pattern(x if x >= 0)))) then
    print("Phew! That was a lot of unwrapping for {x}!")
else
    print("Either none of those nested patterns matched or x is negative.")
```

### `loop` + `is`

Loop while a pattern matches.

```
loop nextValue() is Some(x) then
    print("{x}")
```

### `until` + `is`

Loop until a pattern matches. Bindings are in scope below the loop.

```
mu i = 0
loop
    print("Attempts: {i}")
    i += 1
until getValue() is Pattern(x)

print("{x}")    -- x is guaranteed set here.
```

If `break` is reachable inside the loop, optional bindings are required.

```
loop
    if earlyCondition then
        break
until getValue() is Pattern(opt x)

if x is Some(x) then
    print("{x}")
```

### Error/Maybe Control Flow

Error and null handling is done through the `try` and `maybe` keywords. These blocks are for unwrapping the 2 monadic types in Mulem: *maybes* `T?` and *results* `T!`. When resolving a monadic, you go in *reverse order* that was notated by the type. Think of it like a box: you start from the outside and work your way in.

- `T?` is a *maybe:* it may contain a value or be `None`.
- `T!E` is a *result:* it may contain a value or an error of type `E`.

| Mulem's Type | Other Languages                  | In Plain English                                                             | Layers | Resolve Order             |
|:----------|:---------------------------------|:-----------------------------------------------------------------------------|-------:|:----------------------------|
| `T?`      | `Maybe<T>`                       | A maybe                                                                      |      1 | `?`                         |
| `T!E`     | `Result<T, E>`                   | A result with 1 possible error                                               |      1 | `!`                         |
| `T?!E`    | `Result<Maybe<T>, E>`            | A result with 1 possible error of a maybe                                    |      2 | `!` *then* `?`            |
| `T??`     | `Maybe<Maybe<T>>`                | A maybe of a maybe                                                           |      2 | `?` *then* `?`            |
| `T?!E!F`  | `Result<Maybe<T>, E\|F>`         | A result with 2 possible errors of a maybe                                  |      2 | `!` *then* `?`            |
| `T!E?`    | `Maybe<Result<T, E>>`            | A maybe of a result with 1 possible error                                   |      2 | `?` *then* `!`            |
| `T?!E?`   | `Maybe<Result<Maybe<T>, E>`      | A maybe of a result with 1 possible error of a maybe                       |      3 | `?` *then* `!` *then* `?` |
| `T!E?!F`  | `Result<Maybe<Result<T, E>>, F>` | An result with 1 possible error of a maybe of result with 1 possible error |      3 | `!` *then* `?` *then* `!` |

Think of each `?` or `!` on a *type* as a **layer.** When you call the same `?` or `!` in an *expression,* you **unwrap** that layer.

```
x: int!Error = getRiskyInt()   -- Get wrapped result value.
y: int = x!                    -- Unwrap the result
                               -- Which is equivalent to…
y =
    match x is
    | Ok(val) then
        val                    -- Get the Ok value.
    | Error(e) then
        return Error(e)        -- Exit block, return error if a function
```

```
x: int? = getMaybeInt()   -- Get wrapped maybe value.
y: int = x?               -- Unwrap the maybe.
                          -- Which is equivalent to…
y =
    match x is
    | Some(val) then
        val               -- Get the Some value.
    | None then
        return None       -- Exit block, return None if a function
```

#### `try` / `catch`

```
try
    risky()!
catch
| Error(e) then    -- Catch specific error type.
    body
| (e) then         -- Catch all remaining error type.
    body
| then             -- Catch all remaining error type, but disregard value.
    body

try risky()! catch (Error(e) then expr | then expr)
```

Error handling. Unwrap result types with `!` inside a `try` block. Unhandled errors propagate upward. Like with `match _ is`, inline `try _ catch` needs parentheses around the pattern matching section after `catch`.

```
try
    a = doSomething1(x)!
    b = doSomething2(a)!
    b
catch
| Error(e) then
    print("Error: {e}")
    0
```

```
try
    data = riskyOperation()!
    data2 = anotherRisky()!
    final = process(data, data2)
    final
catch
| IOError(e) then
    print("IO failed: {e}")
    defaultValue
| ValidationError(e) then
    raise Err(e)
```

```
result: int = try
    data = riskyOperation()!
    data2 = anotherRisky()!
    process(data, data2)
catch
| IOError(e) then
    print("IO failed: {e}")
    0              -- fallback value
| ValidationError(e) then
    raise Err(e)   -- Escape function with error
```

Inline form. If you only have a whildcard case `( | x)`, you just write the value `x`.

```
result = try divide(1, 0)! catch 0.0
```

Using `!` inside a function automatically infers a result return type `T!`.

```
riskyFn(a: int): int! =
    b = step1(a)!
    c = step2(b)!
    c
```

#### `maybe` / `else`

None-coalescing. Unwrap maybe types with `?` inside an `maybe` block. If any `?` returns `None`, the block short-circuits.

```
maybe
    a = getA()?
    b = getB()?
    c = maybe getC()? else 0     -- Fallback
    print("{a + b + c}")
else
    print("Didn't work")
```

Inline form.

```
a = Some(10)
x = maybe f(a?) else "fallback"
```

Using `?` inside a function automatically infers a maybe return type `T?`.

```
addStuff(a: int, b: int): int? =
    x = getA()?
    y = getB()?
    x + y
```

Nested maybes unwrap with multiple `??`:

```
unnest(x: int??): int? = x??
```

Chain multiple `maybe` / `else` together untill you get a fallback:

```
getFirst(a: int, b: int, c: int): int =
    ~ maybe getA(a)? else
    ~ maybe getB(b)? else
    ~ maybe getC(c)? else
    ~ 0
```

Or use the `None`-coalescing operator (`?:`).

```
getFirst(a: int, b: int, c: int): int =
    getA(a) ?: getB(b) ?: getC(c) ?: 0
```

Another example of a use for `maybe`:

```
crunchData(): int?!Error!CustomError =                                -- Multiple error types
    value: int? = someFunc()!
    -- Maybe to Result
    data: int = maybe value? else raise CustomError("Not found")      -- Exist function on fallback
    data
```

### Function Control Flow

These keywords change the control flow within a function.

#### `return`

Exits out of a function. If a value is after it, that value is the return value, otherwise it's `void`. This must match the return type of the function. Last-line evaluation is still enabled by default.

```
isThirteen(x) =
    if x == 13:
        return True  -- Exits the function and returns true.
    False            -- Returns false.
```

#### `raise`

Return out of the function with an error value. The function must return a result type `T!`. If declared `T!E` where `E` is an `error` type, then the type passed to `raise` must match.

```
-- T!E is inferred:
alwaysFail() =
    raise MyError("error message")

try
    alwaysFail()!
catch
| (e) then                -- Catch all errors.
    print("Error{e}")
```

#### `yield`

Exits out of a function with an `iter[_]` type. The return value of the function must be of type `iter[T]` where T is the yield type. When you have `yield` in your function, the actual return value in the function body is discarded, and using `return _` in it is a compile-time error. Use of `yield` will infer the return type to be `iter[_]`. 

```
countUpTo(n: int): iter[int] =
    loop i in 0...n then
        yield i
```

If you use `yield`, you can only use a void `return` to exit the function. 

```
countUntil(mu i: int, max: int): iter[int] =
     loop
        if i >= max then
            return      -- Break out of the loop and the function.
        yield i
        i += 1
```

#### `await`

Exits out of a function with an `async[_]` type. The return type of the function must be of type `async[T]` where T is the type that the asynchronous value will resolve in the end. The return value of the asynchronous instance is determined the same way that a non-asynchronous function does it. Use of `await` will infer the return type to be `async[_]`. 

```
asyncFn(a, b): async[int] =
    a = await fetch(a)
    b = await fetch(b)
    a + b
```

Both `yield` and `await` can be used together in an `iter[async[_]]` type. The type suggests it—each yield is of type `async[_]`. Use `loop await` to wait for each async value to resolve in sequential order.

```
asyncIterFn(n): iter[async[int]] =
    loop i in 0...n then
        val = await fetch(i)
        yield val

asyncCollect(n): async[int#] =
    mu ret: int# = []
    loop (await x) in asyncIterFn(n) then
        ret <>= x
    ret
```

Unlike in other languages where *promises* or *futures* can either resolve or reject, async types in Mulem **only resolve.** Instead you can use a result type `T!E` inside an `async[T!E]` function. Unwrap it like you would a result type. Because this is common, `await` has special rules in regards to the `!` and `?` opeerators when placed after it.

```
await! x == (await x)!      -- Unwrap an `async[T!E]`
await? x == (await x)?      -- Unwrap an `async[T?]`
await!? x == (await x)!?    -- Unwrap an `async[T?!E]`
```

#### `defer`

Runs after a function done. For iterator functions, this is when the iterator was broken or exhausted. For asynchronous functions, this is when the asynchronous type is resolved or rejected. Each `defer` statement go in reverse order: *first-in last-out*. It can be one line `defer _` or a block `defer:`. Generally though it's just one line like `defer cleanUp()`. 

```
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

```
First
Second
Done
Middle
Second to last
Last
```

---

## Meta Bindings (`::`)

Variable and functions primarily use the equals sign (`=`) and are for storing actual data within a program, but there's another type of binding used for abstract values for the compiler to know about like constants, types, inline-functions, and generics. This type of declaration is **constant**; in other words, they cannot be *mutated* or *shadowed.* Depending on what it is, subsequent `::` of the same name will *modify its definition.* The most common is `:: impl` with adds methods and static variables to a meta binding. 

* **Aliases:** `numberType :: int`
* **Constants:** `PI :: const float = 3.14`
* **Structs:** `MyStruct :: struct = _`

Other languages often use a set of prefixed keywords to define these things, but the `::` symbol keeps the name of the variable on the left.

- `const PI`, `struct MyStruct`, `enum MyEnum`
- `PI :: const`, `MyStruct :: struct, `MyEnum :: enum`

Meta bindings are meant to resemble definitions. *"This is that."* Only when things get complicated should you have to expand on that. It should be easy and simple to define something. They are like the language file of your API, telling the compiler how to speak your own custom language—not just the language that you *speak* but the language that you *think* in.

### Aliases

Assigning a type after `::` creates an alias. 

```
numberType :: int
```

This alias is unique to the scope. Modifying it only affects the alias and not the original type. This prevents accidental conflictions between modules. *(See [Implementation](#implementing-impl).)*

You can also create aliases for basic product types or sum types.

```
tuple        ::  int , float , char                    -- Also called a "positional tuple".
alsoTuple    :: (int , float , char)                   -- Optional parentheses.
namedTuple   :: {count: int, scale: float, code: char} -- Position not guaranteed.
mixedTuple   :: (int, float) & {code: char}            -- Has both positional and named components.
productUnion ::  int & float & char                    -- Is the size of all types combined.
sumUnion     ::  int | float | char                    -- Is the size of the largest type.
```

#### Tuples

Every opaque type by itself is its own tuple, so for example `char` and `(char)` are the same. This means that in the example, `int & float & char` is the equal to `(int, float, char)`.

Tuples use commas (`,`) to separate components for both positional (`()`) and named (`{}`) tuples. This follows the same rules that function parameters does. *(See [Function Declarations](#function-declarations).)*

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

Opaque types like primitives and enums coerce into a tuple of one, so creating a product type of them creates a positional tuple, e.g. `int & float & char` becomes `(int, float, char)`. 

Combining empty tuples produces an empty tuple `() & () == ()`. The same is true for empty named tuples `{} & {} == {}`. This also means that empty positional tuples and empty named tuples are equivalent `() == {}`. Both tuples have zero dimensions in both positional and named components; therefore they are equivalent. Saying `() & {} & ()` or `{} & ()` and any combination of empty tuples all are the same type.

### Constants

Constants are declared with type `const T`. The `const` makes it clear that we're defining a constant value and not an alias or another type of meta binding. After the type is an equals sign `=` and it's value. The type can be inferred with `:: =`. 

```
PI :: const float = 3.14159
E  :: = 2.71828                -- Inferred
```

You can also bind a function to a constant with `const fn`. Define it like you would with basic functions. The `const fn` is opitional for shorthand form, just `:: (param) =` is needed. This can be useful if you need to pass a function multiple times but don't want it to be outputted when compiled.

```
IDENTITY :: const fn(x) = x     -- Full definition.
addOne :: (x) = x + 1           -- Shorthand.
value = addOne(2)               -- Means (fn(x) = x + 1)(2), result is 3.
array = map([1, 2, 3, 4], addOne)
```

### Structural Types (`struct`)

Structs are product types—or in other words—plain data containers. They cannot extend other structs, but can inherit members of other structs. *(See [Inheritance and Visibility](#inheritance-and-visibility).)*

Put an equals sign after the `struct`. This makes it easier to tell type definitions from aliases and lets the parser know that it's starting a block since a block always starts after a `:` or `=`. `=` was chosen over `:` to show that the block is some sort of data rather than control flow. This makes it clear that when you see `=` at the end of a line, something is being defined. It also resembles the familiar `var: type = value` but the `::` makes it clear that this isn't a run-time value. 

```
MyStruct :: struct =
    name: str
    value: int
```

You can write it with one line, separating each member with a comma `,`.

```
MyStruct :: struct = name: str, value: int
```

Instantiate a struct by calling it like a function. Each member is treated like a named argument.

```
myObject = MyStruct(name: "Foobar", value: 1)
```

Structs are transparent. They can be destructured like named tuples. 

```
TransparentThing :: struct =
    a: int
    b: int

{a, b} = TransparentThing(a: 1, b: 2)
print("a: {a}, b: {b}")
```

### Enumerable Types (`enum`)

Enums are sum types. They define a closed set of variants. Variants may carry data turning them into a tagged union. Like with `struct`, place an `=` after `enum` before starting the block. 

```
MyEnum :: enum =
    First
    Second(int)
    Third{val: int}
```

The inline version works the same.

```
MyEnum :: enum = First, Second(int), Third{val: int}
```

Like structs, instantiate by calling the member like a function unless it doesn't carry any data.

```
a = MyEnum.First
b = MyEnum.Second(2)
c = MyEnum.Third(val: 3)
```

When pattern match, the fill path to the type doesn't need to named on each case, only the name of each member. Use `_` while destructuring to discard the members data.

```
match a is
| First then
    print("first!")
| Second(_) then
    print("second!")
| Third{_} then
    print("third!")
```

### Error Types (`error`)

Errors are bit like both `struct` and `enum`. Each `error` type represents a member of a potential **error tagged union** that's summed up per function with a result `T!E` return type. Every `try` / `catch` block matches patterns to the summed error tagged union in its block based on each `!` point. Result types flatten, so `Result[Result[T, E], F]` would become `Result[T, E|F]` where `E|F` is a tagged union of each possible error in that result. Instantiation works the same as structs.

```
OutOfBounds :: error                 -- No data.
ErrorMessage :: error = (str)        -- Attach a position tuple.
DivideByZero :: error = value: int   -- Attach named member
```

```
divide(num: float, dem: float): float!DivideByZero =
    if dem == 0.0 then
        raise DivideByZero(val: num)     -- Causes branch in `try` block when called with `!`.
    num / dem

try
    divide(1, 0)!
catch
| DivideByZero{val} then
    print("Can't divide {val} by zero!")
```

Uncaught errors in a `try` / `catch` block are implicitly reraised. Each `catch` pattern removes a possible error from the result type of that block. When all possible errors have a `catch` arm, the value of that block is automatically unwrapped so that `Result[T, ()]` becomes just `T`. 

### Prototypes (`proto`)

A `proto` is an abstract interface — a named contract with no data. It is equivalent to a `virtual class` / `trait` / `interface` in other languages. Each member is a function, also called a **method**. Methods that have a parameter named `self` at the beginning will be called like methods on the instance of that type, i.e. `self.method(_)`. This is equivalent to saying `typeof[self].method(self, _)`.

```
MyPrototype :: proto =
    speak(self): str
```

### Implementing (`impl`)

Methods and trait implementations are added separately with `impl`. Much like `proto`, `self` in the first parameter of a method refers to the current instance. You can also add static values that are attached to the type itself. Use `.` to access static values and methods like with structs.

```
MyStruct :: impl =
    staticValue = 1234
    init(name: str, value: int): MyStruct =
        MyStruct(name: name, value: value)

print("{ MyStruct.staticValue }")
```

A method with `self` as the first parameter are callable as a method on the instance. `self` refers to the instance, analogous to `self` / `this` in other languages. You can optionally give it another name with `as`. 

```
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

```
MyStruct :: impl[MyPrototype] =
    speak(self) = "I am a MyStruct \{ name={self.name}, value={self.value} }"

MyEnum :: impl[MyPrototype] =
    speak(self) =
        match self is
        | First then
            "I am a MyEnum of First"
        | Second(x) then
            "I am a MyEnum of Second({x})"
        | Third{val} then
            "I am a MyEnum of Third \{ val={val} }"
```

### Inheritance and Visibility

Even though structs cannot be extended the usual way, they can **inherit** from other structs using the `inherit` keyword. This works similarly to **importing.** It marks members that map to members of another struct, making conversion possible. It follows the same convention for pattern matching like destructuring. *(See [Destructuring](#destructuring).)* 

```
Vector2 :: struct =
    x: float
    y: float

Vector3 :: struct =
    inherit Vector2.x       -- Seperate each inherited member.
    inherit Vector2.y
    z: float

Vector3 :: struct =
    inherit Vector2{x, y}   -- Or in one line.
    z: float

v3 = Vector3(x: 1.0, y: 2.0, z: 3.0)

radius2d(v: Vector2) = sqrt(v.x*v.x+v.y*v.y)
print("{radius2d(v3)}") -- This works because Vector3 inherits from Vector2.
```

When you inherit, you don't just pick out some members. The entire parent struct exists in the child struct in memory, but only some members are visible. 

All members of a type are public by default. When making a subtype, inherited members become private to the subtype unless explicitly redeclared with `inherit`. This encourages separating public and private data into distinct types rather than using access modifiers.

```
PrivateFields :: struct =
    val: int
    secret: int

PublicFields :: struct =
    inherit PrivateFields.val     -- Redeclared, `val` is public / `secret` is private
    other: int
```

A subtype cannot accidentally expose or clash with a private inherited member because types only see members that have been explicitly declared within them. This mirrors the convention used for imports.

## Meta Functions

Adding a parameter before the double colon (`::`) turns it into a **meta function.** *A meta‑function is a compile‑time function that returns code, types, or values.* Parameters are put in square brackets `[]` to distinguish them from regular functions which use parentheses `()`. The result is treated like a constant for run-time code. 

You can define a meta function by adding a parameter before the double colons and writing an expression after it. Like constants, they don't output a value in memory when compiled, useful for collecting repeated code. Unlike regular functions, meta function cannot be passed to another function. They only exist at compile-time. Each parameter is a variable within the expression, so you don't need to wrap them in parentheses `()` like with C macros. 

```
max[a, b] :: = if a > b then a else b
min[a, b] :: = if a < b then a else b
```

You can also have multi-line meta function like regular functions. Each meta function creates a new scope. Defining variables that could bleed into the surrounding scope is not allowed. The last expression is the return value. Call it like a function using `[]`. 

```
doSomethingComplicated[x] :: =
    x = x + 1
    x = x / 2
    x * x

value = doSomethingComplicated[3]
```

These two are the same thing:

```
value =
    x = (3) + 1
    x = x / 2
    x * x
```

Some more examples using the `[]` notation:

```
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

You can also define a type with a meta function. The meta functions parameters in `[]` will be inferred if it returns a function or type and you call it with parentheses `()`. In other words, `meta(_)` is equal to `meta[_](_)`. 

```
-- Note that this is not the actual definition for a maybe type `T?`. This is just a user-defined enum that uses the same pattern.
MyMaybe[T] :: enum =
    Some(T)
    None

Some[T] :: MyMaybe[T].Some
None[T] :: MyMaybe[T].None

maybeInt = Some(1)
```

Type parameters can be omitted at the call site if they can be fully inferred from the value arguments, in which case the call uses only parentheses `()`. It can also be called explicitly by making an alias for it or calling with both brackets at the same time `[]()`

```
SomeInt :: Some[int]
maybeInt = SomeInt(1)

maybeInt = Some[int](1)
```

The syntax `[]` was chosen so that generic type inference will take precedence. `meta(a, b)` means to *call the instantiated function that `meta` returns with inferred types* whereas `meta[a, b]` means to *call the abstract function `meta` with these exact values.* This also makes it easy to distinguish actual function calls from macros/inlining. This removes the need for the more conventional arrow bracket `<>` syntax, which can get confusing. For example, in `f( g < a, b > ( c ) )`, is `g` a generic function or is that comparing two values and passing the results to `f`? The square bracket syntax removes this ambiguity, `f( g [ a, b ] ( c ) )`. This makes it semantically clear that you're doing a compile-time function call followed by a run-time function call. 

### Where Block

This is not required for all meta functions but is useful for defining what patterns each parameter is expected to be. It must be the first definition, and any subsequent definitions should have patterns that match the `where` clause. One common pattern is simply `type` which indicates that a parameter is any literal type. 

```
List[T, N] :: where =
    T: type           -- `type` refers to any literal type, i.e. not a value
    N: const int      -- A constant `int` that must be known at compile time

List[T, N] :: struct =
    data: T#N

List[T, N] :: impl =
    init() =
        data: T#N = [...loop _ in 0..N then default]
        List[T, N](data: data)
```

Types can require a certain `proto` to have been implemented:

```
sort[T] :: where =
    T: impl[Comparable]      -- This type T needs to have implemented `Comparable`.

sort[T] :: (arr: T#): T# = _
```

This is the bread and butter of generic programming. Without it, you can't write a generic `sort`, `min`, `max`, or any algorithm that requires behavior from its type parameter. 

*"Types can also have multiple `proto` requirments."*

```
serialize[T] :: where =
    T: impl[Serializable] & impl[Comparable]     -- This type T needs both `Serializable` and `Comparable`.
```

*"The key type of this dictionary must implement Hashable."*

```
lookup[V, K] :: where =
    V: impl[Hashable]    -- V is a type T#K
    K: keyof[V]

--       V  = T#K
-- keyof[V] = K
-- valof[V] = T
lookup[V, K] :: (dict: V, key: K): valof[V]? = _
```

*"The return type of this function must match the element type of the array."*

```
transform[T, U] :: where =
    U: typeof[fn(T): U]  -- U is whatever the mapping function returns

transform[T, U] :: (arr: T#, f: fn(T): U): U# = ...
```

```
Matrix[T, R, C] :: where =
    T: impl[Numeric]
    R: const int & (R > 0)    -- Must be positive
    C: const int & (C > 0)

-- Constraint across parameters:
Slice[T, Start, End] :: where =
    Start: const int
    End: const int
    End >= Start            -- Relationship between parameters
```

```
-- A generic pair type
Pair[A, B] :: where =
    A: impl[Comparable]
    B: impl[Comparable]

Pair[A, B] :: struct =
    first: A
    second: B

-- Pair is only Comparable if both A and B are
Pair[A, B] :: impl[Comparable] =
    compare(self, other: Pair[A, B]): int =
        r = self.first.compare(other.first)
        if r != 0 then r else self.second.compare(other.second)
```

*"These two type parameters must be the same type."*

```
zip[A, B, C] :: where =
    C == (A, B)     -- C must be exactly a tuple of A and B
```

*"This parameter must be a specific projection of another."*

```
flatten[T] :: where =
    T: T##          -- T must be a 2D array, inner type inferred
```

```
sort[T] :: where =
    T: impl[Comparable] else "sort requires T to implement Comparable — define `compare(self, other: T): int` on your type"
```

### Manual Implementation

Generics will automatically generate code based on their parameters, but you can also implement them by hand using pattern matching. If you only want to use the manual implementations for a generic function, you can set its body to `not return`. This creates a virtual function that can be overloaded later. If you use a function that is defined with `not return`, it will throw a compile-time error.

```
-- Forces every type to have its own implementation
increment[T] :: (ref c: T): void = not return

Counter :: struct = value: int

-- Specialized for Counter
increment[Counter] :: (ref c: Counter): void =
    c.value += 1

-- Specialized for float
increment[float] :: (ref c: float): void =
    c += 1.0

c = Counter(value: 2)
f = 3.0
b = True

increment(c)   -- T is inferred Counter
increment(f)   -- T is inferred float
(--
increment(b)   -- T is inferred bool which has no implementation, compile-time error
--)
```

---

## Importing and Modules

* **Modules:** Declare with `moduleName :: mod`. Only one per file.
* **Imports:** Must be explicit. `a.b{c, d} :: import`. Use `import "path"` for direct file imports.

Use `import` to import something, optionally giving the import an alias with `as`. You can either import a single export like `a.b{c} :: import` or multiple at once using destructuring rules `a.b{c, d} :: import`. *(See [Destructuring](#destructuring).)* This follows the same convention as destructuring with tuples. All imports must be **explicitly** declared—no `import a.b.*`. This helps prevent naming conflicts and track where things have been defined.

The most common import will likely be the `print` function, which will be defined somewhere in a standard library.

```
std{print} :: import    -- This is just an example and not final.

print("Hello, world!")
```

Modules are named with the keyword `mod` near the top before anything is defined. This is the name you'll use when importing your module. **There can only be one `mod` declaration per file.** Multiple `mod` declarations is a syntax error. Imports are based on the include path when compiling or running a script. To import from a file by direct filepath, use `import "path"`.

```
someModule{thing} :: import "../../somewhere.mu"

myModule :: mod

addThing(x) = x + thing
```

In this example, you would import `addThing` like this (assuming the file is included):

```
myModule{addThing} :: import
```

This connects the same explicit-list convention as `inherit` and capturing — *no hidden dependencies, everything that can affect behavior is named.* The three together form a consistent rule across the language:

| Keyword   | "I am explicitly pulling in…"       |
|:----------|:------------------------------------|
| `import`  | Names from another module.          |
| `inherit` | Members from another struct.        |
| `% (x)`   | Mutable variables from outer scope. |

### Memory Models

Mulem is multi-paradigm: different functions, structs, or modules can use different memory strategies in the same program. The model is controlled per-module. Boundary crossing between models follows FFI-like rules — automatic marshalling where possible, explicit escapes otherwise.

Modules define how memory is handled with the `memory` module setting. By default, modules use a garbage collector. Some options include `Collect(GC)` (default),  `Count(ARC)` (reference counting), and `Manual`. `GC` and `ARC` represent the standard garbage collector and reference counter respectively, but others can be defined and used instead.

```
std.mem{Count, ARC} :: import

moduleThatUsesReferenceCounting :: mod =
    memory: Count[ARC]
```

How this is implemented is outside of the scope of this document. That will be saved for when it's time to make a standard library for Mulem. For now, Mulem will focus on only implementing the garbage collector which will work in both interpreted and compiled mode.

---

## Design Philosophy

Mulem's unconventional choices are intentional, prioritizing readability, explicit data tracing, and scalable complexity:

* __Readability First:__ Significant whitespace avoids bracket clutter, while the strict block/inline switching rules (`:` vs `()`) prevent you from fighting the formatter.
* __Traceable Dependencies:__ The language forces you to explicitly name what you bring into scope. There are no glob imports or implicit mutable state captures. `import`, `inherit`, and `%` (capturing) all use explicit listing.
* __Unified Concepts:__ `::` is the universal operator for top-level, compile-time definitions (structs, aliases, constants).
* __Type and Expression Symmetry:__ `?` (Maybes) and `!` (Results) work identically whether used in type definitions or as unwrapping operators.
* __Scalable Patterns:__ Simple tasks use simple syntax (e.g., `fn(x) = x`), but the language easily scales up to explicit types, memory models, and generic constraints as complexity demands.

---

## Putting It All Together…

Here is a quick synthesized example showing how Mulem's structs, implementations, pipelining, and error handling might look in a real script:

```
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
    A function returning a Result type (User or an Error).
    It captures no outside state.
--)
fetchUser(id: int): User! =
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
        -- Fetch the user, unwrap the result, and pipe it forward
        fetchUser(1)!
        |> print($.speak()); $
        |> print("User is {$.age} years old."); $
    catch
    | FetchError(e) then
        print("Failed to fetch user: {e}")
```

---

## Reserved Keywords

Mulem has 42 reserved keywords. Note that built-in types (`int`, `str`), boolean values (`True`, `False`), standard maybes (`Some`, `None`), are built-in symbols but *not* strict keywords.

**The Keyword List:**
`and`, `as`, `await`, `band`, `bor`, `break`, `catch`, `continue`, `defer`, `do`, `else`, `enum`, `error`, `fallthrough`, `fn`, `if`, `impl`, `import`, `in`, `inherit`, `is`, `loop`, `match`, `maybe`, `mod`, `not`, `opt`, `or`, `out`, `proto`, `raise`, `ref`, `rem`, `return`, `self`, `struct`, `then`, `try`, `until`, `void`, `where`, `xor`, `yield`.

---

## Table of Operators

| Level | Category                   | Operators                                 |
|:------|:---------------------------|:------------------------------------------|
| 12    | Member access/Function     | `.` `?.` `#` `?#` `()`                    |
| 11    | Postfix                    | `?` `!` `^`                               |
| 10    | Unary                      | `+` `-` `not`                             |
| 9     | Exponent                   | `**` (right-associative)                  |
| 8     | Multiplicative / Shift     | `*` `/` `//` `rem` `mod` `<<` `>>` `>>>`  |
| 7     | Additive / Concat          | `+` `-` `<>`                              |
| 6     | Bitwise                    | `band` `bor` `xor`                        |
| 5     | Range                      | `...` `..=`                               |
| 4     | Comparison                 | `==` `!=` `<` `>` `<=` `>=`               |
| 3     | Logical AND                | `and`                                     |
| 2     | Logical OR / None-Coalesce | `or` `?:`                                 |
| 1     | Pipeline                   | `|>`                                      |
| 0     | Assignment / Spread        | `=` `+=` `-=` `&` `...`                   |

| Operator       | Meaning                                             |
|:--------------:|:----------------------------------------------------|
|   `lhs . rhs`  | Member access                                       |
|   `lhs # rhs`  | Direct array/dictionary index                       |
|   `lhs #[rhs]` | Safe array/dictionary index                         |
|   `lhs ?`      | Unwrap maybe, propagate `None` to nearest `maybe`   |
|  `lhs ?. rhs`  | None-coalessing member access                       |
|  `lhs ?# rhs`  | None-coalessing array/dictionary access             |
|   `lhs !`      | Unwrap result, propagate error to nearest `try`     |
|   `lhs ^`      | Dereference typed pointer                           |
|     `... rhs`  | Spread array into array                             |
|       `& rhs`  | Spread tuple into tuple (same type)                 |
|  `lhs ... rhs` | Exclusive range                                     |
| `lhs ..= rhs`  | Inclusive range                                     |
| `lhs \|> rhs`  | Pipeline                                            |
|   `lhs = rhs`  | Assignment or declaration                           |
|  `lhs += rhs`  | Increment                                           |
|  `lhs -= rhs`  | Decrement                                           |
| `lhs: T = rhs` | Explicit typed declaration                          |
|   `lhs + rhs`  | Addition                                            |
|   `lhs - rhs`  | Subtraction                                         |
|       `+ rhs`  | Unary positive                                      |
|       `- rhs`  | Sign flip                                           |
|   `lhs * rhs`  | Multiplication                                      |
|   `lhs / rhs`  | Exact division (float)                              |
|  `lhs // rhs`  | Floor division (int)                                |
|  `lhs rem rhs` | Modulo (sign matches `lhs`)                         |
|  `lhs mod rhs` | Floor modulo (sign matches `rhs`)                   |
|  `lhs ** rhs`  | Exponentiation (right-associative)                  |
|  `lhs == rhs`  | Equality                                            |
|  `lhs != rhs`  | Inequality                                          |
|   `lhs > rhs`  | Greater than                                        |
|   `lhs < rhs`  | Less than                                           |
|  `lhs >= rhs`  | Greater than or equal                               |
|  `lhs <= rhs`  | Less than or equal                                  |
| `lhs band rhs` | Bitwise AND                                         |
| `lhs bor rhs`  | Bitwise OR                                          |
| `lhs xor rhs`  | Bitwise XOR                                         |
|  `lhs << rhs`  | Shift left                                          |
|  `lhs >> rhs`  | Shift right                                         |
| `lhs >>> rhs`  | Unsigned shift right                                |
|  `lhs <> rhs`  | Concatenation                                       |
| `lhs and rhs`  | Logical AND                                         |
|  `lhs or rhs`  | Logical OR                                          |
|     `not rhs`  | Logical NOT (or bitwise NOT for numbers)            |

`not` behaves like the `!` prefix operator in some languages which use both logical NOT and bitwise NOT depending on the type.

```
not False == True
not 5 == -6
```

__Remainder vs. Modulo:__

- **`rem` is "remainder"** *(C-style modulo)* — what's left over after truncating toward zero. The result takes the sign of what you started with.

```
 7 rem  3 ==  1    -- positive because 7 is positive
-7 rem  3 == -1    -- negative because -7 is negative
 7 rem -3 ==  1    -- positive because 7 is positive
-7 rem -3 == -1    -- negative because -7 is negative
```

- **`mod` is "true modulo"** — the mathematical kind where the result always lives in the range `[0, divisor)`. The result takes the sign of what you're dividing *by*.

```
 7 mod  3 ==  1    -- same as above, no difference here
-7 mod  3 ==  2    -- "wraps around": -7 mod 3 = 2 in math
 7 mod -3 == -2    -- wraps the other way
-7 mod -3 == -1    -- same as % here
```

The practical case where it matters is things like clock arithmetic, array wrapping, or anything where you want `(-1) mod n` to give you `n - 1` instead of `-1`. With `rem` you'd have to write `((x rem n) + n) rem n` — the classic defensive idiom.

### Key Type Modifiers & Postfix Operators

| Syntax     | Meaning            | Note                                                     |
|:-----------|:-------------------|:---------------------------------------------------------|
| `T?`, `x?` | Maybe              | Unwraps a maybe; propagates `None` to nearest `maybe`.   |
| `T!`, `x!` | Result / Error     | Unwraps a result; propagates error to nearest `try`.     |
| `T#`, `x#` | Array / Dict Index | `T#N` is fixed-size. Access: `array#index`.              |
| `T^`, `x^` | Pointer            | Dereference a pointer.                                   |

---

*This document captures the current state of the Mulem design. The language is still evolving.*
