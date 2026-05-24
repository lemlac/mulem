# Mu Language Reference

*Version 0.1 (Draft)*

Mu (also called *Mulang*) is a general-purpose, multi-paradigm language with significant whitespace. It is expression-oriented and gives programmers explicit control over evaluation strategy, memory model, and error handling. It funds the right balance between *conciseness* and *eloquence.* Mu is planned to be both compiled and interpreted within the same language, making it suitable for systems programming, AI, and game development.

Mu targets developers who want Python-like readability, Rust-like control and safety features, and F#-style expressive pipelines — especially for domains like robotics, systems programming, AI, and games.

Key design goals:
- Strong expression-orientation
- Hybrid significant/insignificant whitespace model
- Modern error and option handling
- Support for both interpretation and compilation (including direct shared library output)

---

## Core Design Philosophy

- **Expression-oriented**: Almost everything is an expression and returns a value.
- **Significant whitespace** with smart inline support.
- **Modern error & option handling**: `?` and `!` propagation, `||` coalescing.
- **Flexible**: Multi-paradigm (functional, procedural, low-level).

---

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

Whitespace is significant. Indentation marks where blocks begin and end. Four (4) spaces per level is recommended. Tabs and spaces may be mixed, but each expression within a block must have exactly the same indentation sequence — the same number of tabs and spaces in the same order.

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
| Words                | `x`, `PI`, `1`, `3.14`, `0xABCDEF`, `$`, `$x`, `$0` |
| String/Char literals | `'a'`, `"foo"`, `"""big string"""`, `''raw''`       |
| Delimiters           | `,` (tuples/arrays), `;` (expressions)              |
| Symbols              | `~!@#%^&*-+=\|:<.>/?` (excluding `--`)              |
| Brackets             | `()`, `[]`, `{}`                                    |
| Whitespace           | spaces, tabs, newlines                              |

---

## Expressions and Blocks

A program is a sequence of expressions. The golden rule in Mulang is that *any line ending in `:` or `=` starts a block.* Blocks are created with `:` after a construct, followed by indented content. The last expression in a block is its value.

```
if condition:
    action()
    result
```

Each block or bracket statement (`()`/`[]`/`{}`) starts a new scope. For example:

```
(x = 0; x + 1)
```

`x` is isolated to inside the parentheses, and the result of the expression is `x + 1`. 

Commas have lower presedence than semicolons. So in an expression like this…

```
(a, b; c, d; e)
```

…It would be evaluated as `(a, (b; c), (d; e))` and **not** `((a, b); (c, d); e)`. The result is a tuple: `(a, c, e)`.

### Expression Splitting

Semicolons and newlines are ignored for expressions when they are inside a multi-line string `"""…"""` or comment `(-- --)`. 

```
x = 1 + (-- New lines and semicolons ignored here;
   --) 2
```

```
str = """
      Big
      string;
      """
```

Otherwise, a semicolon always ends an expression. New lines have special rules that any line starting with a `.` continues from the previous line. This is known as **expression splitting.** 

```
-- Method chaining across lines:
object.method1()
      .method2()
      .method3()

-- object.method1();  ← semicolon disables splitting
--       .method2()   ← SyntaxError
```

It works similarly to a backslash `\` in other languages but offers more advantages. With backslashes, they all go at the end of the a line and are unaligned in mosts cases without formatting.

```
x = 1 \
    + 2 + 3 \
    + 4 \
    + 5 + 6
```

With Mulang's expression splitting, this becomes much more readable. 

```
x = 1
. + 2 + 3
. + 4
. + 5 + 6
```

The periods (`.`) become like dots in a bullet-point list, and it also takes on the appearance of an elipses (…) making it apparent that you have one long expression. There are special rules in regard to periods before operators. *For more information, see [Operators](#Operators).*

### Blocks

A block wraps multiple expressions into one. A `:` or `=` followed by a newline and indentation starts a block. The last expression evaluated in a block is its value. Use `_` (wildcard) to leave a block empty.


```
do:
    expr
    expr    -- This is the block's value.

do:
    _       -- Empty block.
```

### Inline Blocks `( …: … )`

To switch from block mode to inline mode (and vice versa):

```
(do:
    _
)
```

Mulang takes special care when switching between block mode and inline mode, allowing coders to format their code naturally and elegantly. No special keywords or extra brackets are needs, and once you see it, you'll realize it's quite intuitive. 

There are 3 rules that Mulang follows:

1. Any line ending in `:` or `=` starts a block (significant whitespace).
2. When inside a bracket `(`/`[`/`{`, the matching bracket `)`/`]`/`}` or comma `,` ends the block and switches back to inline mode (insignificant whitespace)
3. If all brackets are closed, return to block mode at the end of a line.

```
apiFetch(fn(result) =    -- Start a block for the function.
    if result > 0:       -- Whitespace is significant here.
        print("Success! {result}")
    else:
        print("Failure! {result}")
)                        -- Bracket match, switch back to the inline mode.
                         -- All brackets closed, next line.
```

- *Line ends with ":" or "=" → Enter block mode*
- *Dangling "(" "[" "{" on same line → Inline mode until closed*
- *All brackets closed → Return to block mode*

If a bracket has a line ending in `:` or `=`, it will start an **inline-block.** Nesting works freely. You only need to worry about closing the bracket when you're done with the block. 

```
(if x:
    do:
        _
else:
    do:
        _
)
```

### Inlining with `then`

Any block that has a *subject* can be inlined using `then` instead of `:`.

```
if x then "True" else "False"   -- Inline form.

if x:                           -- Block form.
    "True"
else:
    "False"
```

---

## Operators

Mu has a clean, consistent operator set with both symbolic and word forms.

Symbols are grouped by meaning: `*`/`/` for math, `?` for options, `!` for results, `#` for arrays, `&` for tuples, `^` for pointers. Repeating an operator gives a more technical variant: `+` addition vs `++` concatenation, `*` multiplication vs `**` exponentiation, `/` division vs `//` floor division. Bitwise operators use `band`, `bor`, and `xor` rather than `&`, `|`, and `^` because those symbols have other meanings in Mu.

### Precedence (Highest to Lowest)

| Level | Category                  | Operators                |
|:------|:--------------------------|:-------------------------|
| 11    | Member access/Function    | `.` `[]` `()`            |
| 10    | Postfix                   | `? ! ^`                  |
| 9     | Unary                     | `+ - not ~`              |
| 8     | Exponent                  | `**` (right-associative) |
| 7     | Multiplicative / Shift    | `* / // % %% << >> >>>`  |
| 6     | Additive / Concat         | `+ - ++`                 |
| 5     | Bitwise                   | `band bor xor`           |
| 4     | Range                     | `.. ..=`                 |
| 3     | Comparison                | `== != < > <= >=`        |
| 2     | Logical AND               | `and`                    |
| 1     | Logical OR / Pipeline     | `or \|\| \|>`            |
| 0     | Assignment / Spread       | `= := += => & ...`       |

| Operator       | Meaning                                             | Precedence |
|:---------------|:----------------------------------------------------|:----------:|
| `lhs . rhs`    | Member access                                       |     11     |
| `lhs # rhs`    | Array/dictionary index                              |     11     |
| `lhs ?`        | Unwrap option, propagate `None` to nearest `opt`    |     10     |
| `lhs !`        | Unwrap result, propagate exception to nearest `try` |     10     |
| `lhs ^`        | Dereference typed pointer                           |     10     |
| `~ rhs`        | Inferred type conversion                            |     10     |
| `... rhs`      | Spread array into array                             |      0     |
| `& rhs`        | Spread tuple into tuple (same type)                 |      0     |
| `lhs .. rhs`   | Exclusive range                                     |      4     |
| `lhs ..= rhs`  | Inclusive range                                     |      4     |
| `lhs \|> rhs`  | Pipeline                                            |      1     |
| `lhs => rhs`   | Pipeline assignment                                 |      1     |
| `lhs = rhs`    | Assignment or declaration                           |      0     |
| `lhs := rhs`   | Explicit inferred-type declaration                  |      0     |
| `lhs: T = rhs` | Explicit typed declaration                          |      0     |
| `lhs + rhs`    | Addition                                            |      6     |
| `lhs - rhs`    | Subtraction                                         |      6     |
| `+ rhs`        | Unary positive                                      |      9     |
| `- rhs`        | Sign flip                                           |      9     |
| `lhs * rhs`    | Multiplication                                      |      7     |
| `lhs / rhs`    | Exact division (float)                              |      7     |
| `lhs // rhs`   | Floor division (int)                                |      7     |
| `lhs % rhs`    | Modulo (sign matches `lhs`)                         |      7     |
| `lhs %% rhs`   | Floor modulo (sign matches `rhs`)                   |      7     |
| `lhs ** rhs`   | Exponentiation (right-associative)                  |      8     |
| `lhs == rhs`   | Equality                                            |      3     |
| `lhs != rhs`   | Inequality                                          |      3     |
| `lhs > rhs`    | Greater than                                        |      3     |
| `lhs < rhs`    | Less than                                           |      3     |
| `lhs >= rhs`   | Greater than or equal                               |      3     |
| `lhs <= rhs`   | Less than or equal                                  |      3     |
| `lhs band rhs`   | Bitwise AND                                       |      5     |
| `lhs bor rhs`   | Bitwise OR                                         |      5     |
| `lhs xor rhs`   | Bitwise XOR                                        |      5     |
| `lhs << rhs`   | Shift left                                          |      7     |
| `lhs >> rhs`   | Shift right                                         |      7     |
| `lhs >>> rhs`  | Unsigned shift right                                |      7     |
| `lhs ++ rhs`   | Concatenation                                       |      6     |
| `lhs \|\| rhs` | None-coalescing                                     |      1     |
| `lhs and rhs`  | Logical AND                                         |      2     |
| `lhs or rhs`   | Logical OR                                          |      1     |
| `not rhs`      | Logical NOT (or bitwise NOT for numbers)            |      9     |

Mulang doesn't have a `--` decrement operator. It uses `-=`. 

Different uses of `|`:

- `|` — pattern match cases
- `|>` — pipeline operator
- `||` — None-coalescing

Symbol operators gain assignment forms with `=` and pipeline-assignment forms with `=>`.

```
x += 1          -- x = x + 1
expr +=> x      -- pipeline: x = x + expr
```

All infix operators (symbolic or keyword) may have a period in front of them. This doesn't affect order of operations.

```
a   + b   * c   - d   / e   % f   band g
a . + b . * c . - d . / e . % f . band g    -- These statements are equivalent.
```

This allows you to use *expression splitting* to split an expression into multiple lines. 

```
a           -- Add newlines before each period.
. + b
. * c
. - d
. / e
. % f
. band g
```

Indentation is more leanient with expression splitting. As long as the line starts with a period (`.`), then it belongs to the same expression.

```
do:
    a
        . + b
      . * c
        . - d
    . / e
  . % f                -- Indenting less than the start works too.
      . band g
```

This rule also applies to the range operator `..`, making `..` and `...` equivalent. Only the `=` affects if it's exclusive or inclusive.

```
0..1  ==  0...1    -- Exclusive range
0..=1 ==  0...=1   -- Inclusive range
```

Note that the period inside of number literals such as `3.14` is treated differently. *For more information, see [Numbers](#Numbers).*

### Function Calls

Function can be called in 3 ways:

```
function(a, b, c)   -- positional
function{a, b, c}   -- named, turns into {a: a, b: b, c: c}
function x          -- spread tuple into function
```

`function x` is short for `function(&x)`, where `&` is the tuple spread operator.

### Pipelining `|>`

When placed at the start of a line in a block, the pipelining operator `|>` takes on a special meaning: it takes the value of the previous line and makes it available in the next line as `$`. This symbol is known as the **pipeline context.** It has several uses in Mulang, but in the context of pipelines it simply represents the result of the previous line. This makes it easy to chain a sequence of calls and read them in order.

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
|> fetchB $
|> fetchC $
|> print("{$}")
```

This makes the order of operations easy to follow at a glance. It reads like a plain-English list:

- `fetchA`
- *then* `fetchB`
- *then* `fetchC`
- *then* `print`

The first line of a block may also start with `|>`, in which case its pipeline context `$` is an empty tuple `()`. A line starting with `|>` is not required to use `$`. Some functions also require certain variables to be defined in the pipeline context. *(See [Function Declarations](#Function-Declarations).)*

Multiple expressions separated by semicolons `;` on one line share the same context `$`. The last expression on the line is passed as the context to the next pipe.

```
|> fetchA()                   -- Run fetchA,
|> print("{$}"); fetchB $     -- Print result, then fetchB
|> print("{$}"); fetchC $     -- Print result, then fetchC
|> print("{$}")               -- Print result.
```

Note that `|>` at the beginning of a line is different from `. |>` which is a split expression on the next line. The pipe will continue after it.

```
|> fetchA()
. |> fetchB $
|> fetchC $
|> print("{$}")
```

*Is the same as...*

```
|> fetchA() . |> fetchB $    -- The period doesn't effect this statement.
|> fetchC $                  -- Pipe from the previous statement.
|> print("{$}")
```

A **pipeline block** is started with `|>:`, either at the end of a line or on its own line. Within the block, `$` holds the piped-in value within the scope of that block.

```
|> fetchA()         -- Set up things.
|> fetchB $         -- …
|> fetchC $         -- …
|>:                 -- Context is now ready.
    print("{$}")    -- Use it here.

fetchA() |> fetchB $ |> fetchC $ |>:  -- Or in one line.
    print("{$}")                        -- Then use the result.

-- Freely mix the two formats:
fetchA() |>:           -- Start with this context.
    print("{$}")       -- Use the same `$` for these two lines.
    fetchB $           -- Same context `$`.
    |> print("{$}"); fetchC $ |>:   -- Start a new context inline.
        print("{$}")                 -- Print the final result.
```

To get a value within a pipeline, use `=> x` after any step to store it into a local variable. The assignment is written in reverse order — the variable name goes on the right.

```
|> fetchA()
|> fetchB $
|> fetchC $ => x    -- Put the result into `x`.

print("{x}")        -- Print the result.
```

This lets you extract the result of any step in a pipeline simply by appending `=> name` to that line.

```
-- Put all results of each step into variables.
|> fetchA() => a
|> fetchB $ => b
|> fetchC $ => c

print("a = {a}, b = {b}, c = {c}")
```

The variable type is always inferred, to avoid ambiguity with `:`. Mutability can be specified with `mu`. *(See [Mutability](#Mutability).)*

```
fetchA() => mu x |> fetchB(x) |>:  -- Create a mutable variable `x`.
    x += 1                          -- Mutate it.
    print("{x}")                    -- Print it.
```

To put the final result of any pipeline into a variable, use `|> $ => x` at the end, where `x` is any variable name. This makes it easy to mix and match the pipes in between with `|> $ => x` at the end to collect it all into a variable. 

```
|> fetchA()      -- Start.
|> fetchB $      -- Pass context.
|> fetchC $      -- Pass context.
|> $ => x        -- Put the final context into `result`.

print("{x}")
```

A use case for `|>:` is to configure an object before assigning it to an immutable variable.

```
user = User.create() |>:
    $.name = "John Smith"  -- Set properties on the context.
    $.dob = "1970-01-01"
    $                      -- Return the context.

print("User: {user.name}, born: {user.dob}")  -- Prints "User: John Smith, born: 1970-01-01"
```

This gives you a great deal of flexibility in how you choose to express your code.

### Pipeline Context (`$`)

Any variable starting with `$` is a **context variable.** These are scoped variables that functions can see when they're called. They must be declared in the function's signature with captured variables to see them. *(See [Function Declarations](#function-declarations).)*

Variable names in Mulang don't allow `$` inside them, but this symbol is treated like a name. You can put symbols next to it, but other words needs to be seperated with a space next to it.

```
-$    -- This is okay, 1 symbol + 1 word: `-` + `$`.
not$  -- This is one word, error since `not$` doesn't exist.
not $ -- This is okay, 2 words: `not` + `$`.
```

Variables declared as `$x` and `$0` and such are context variables, and one context variable gets set during pipeline: `$` by itself also called the **pipeline context.** It holds the value of the previous pipeline expression.

```
(0, x: 1)               -- Create a tuple with position member and named member `x`.
|>:                     -- Pass to a pipeline block, sets $ = (0, x: 1) 
    print("{$.0 + $.x}")  -- Prints "1".

(0, x: 1) |> print("{$.0 + $.x}")  -- Or in-lined.
```

Going back to the example in the previous section, we can make it even more concise like this:

```
user = User.create() |> {
    &$,                   -- Append current pipeline context to this object.
    name: "John Smith",   -- Modify select properties below it.
    dob: "1970-01-01",
}
```

`&$` means to spread the pipeline result in this tuple. This makes an object with new properties set below it. Because this is such a common action, we can abriviate with `&{}`. 

```
user = User.create() |> &{  -- Means to append to the pipes context.
    name: "John Smith",
    dob: "1970-01-01",
}
```

A top level `&` will spread based on the current pipeline context `$`. This makes it easier to pipe data and only change what you need. 

## Basic Bindings

There are two types of bindings: basic `=` and meta `::`. See [Meta Bindings](#Meta-Bindings-) below for details about `::`.

### Variable Declarations

```
x = 42                    -- inferred
y: int = 42               -- explicit type
z := 42                   -- forced inference
mu counter = 0            -- mutable
```

Variables are declared with just the equals sign (`=`). Type is inferred, but can be declared with a colon (`:`). You can also use the `:=` operator instead to declare and infer the type at the same time. This is useful for shadowing mutable variables. *(See [Mutability](#Mutability).)* For now, just know that anytime you see `:` before `=`, *it always declares a new variable,* and if you see just `=`, *it's either declaring or mutating a variable.*

```
a = 0               -- Implicit declaration, type inferred.
b: int = 1          -- Explicit type.
c := 2              -- Explicit declaration, inferred type.
```

Adding a new line and indentation after the `=` starts a block. The last expression evaluated in the block is the value of that variable.

```
lunch =
    if getDayOfWeek() == "Tuesday":
        "tacos"
    else:
        "sandwich"
```

Variables are immutable, but declaring it again shadows it. Any subsequent `=` of an immutable variable is an implicit declaration. Redeclaring a variable with the same name is called **shadowing.** This makes Mulang flexible while still having the advantages of being statically typed.

```
a = 1
a = 2           -- New variable, shadows previous `a`.
a = "hello"     -- Type can change when shadowing.
```

You can also shadow a variable using its previous value. This works for both regular assignment `=` and pipeline assignment `=>`.

```
i = 0
i = i + 1     -- Sets new `i` based on old `i`.
i += 1        --\ 
i + 1 => i    ---\
1 +=> i       ---- All do the same thing.
```

`=` is a void statement and may not be used inside expressions that expect a non-void value. Use `==` for comparison and `=>` for inline assignment.

```
-- Error:
if x = 0: _

-- Correct:
if x == 0: _
```

Reversing the normal order of assignment for `=>` operators helps with programmers who might confuse `=>` for `>=` (greater than or equals).

```
-- Error: did you mean `x >= 0`?
if x => 0: _
```

Contextual variables are declared with `$` at the start of their name. This is used for functions with contextual parameters, letting you to share variables between functions without passing them directly. *(See [Contextual Parameters](#contextual-parameters).)*

```
printX() / $x: int =   -- Function that requires `$x` to be defined.
    print("{x}")

$x = 0
printX()        -- Prints "0"
```

| Channel    | Meaning                       | Lifetime          |
|:-----------|:------------------------------|:------------------|
| `$`        | pipeline value                | per pipeline step |
| `$name`    | shared contextual variable    | lexical scope     |
| `$name: T` | required contextual parameter | function call     |

*See [Contextual Parameters](#comtextual-parameters) for more details.*

### Mutability (`mu`)

Mutable variables are declared with `mu`. Setting them later mutates the value rather than shadowing it.

```
mu count = 0
count += 1          -- Mutates count.
count := 0          -- Shadows count with a new immutable variable.
```

A mutable variable may be declared without an initial value, but cannot be used until it is set.

```
x: mu int
x = 1
doSomething(x)      -- OK now.
```

Functions do not automatically capture mutable variables. Any assignment inside a function to an outer mutable variable creates a new local variable unless explicitly captured. *(See [Capturing](#capturing).)*

```
mu count = 0

addCount() / count =
    count += 1

addCount()
print("{count}")    -- "1"
```

### References (`ref` / `ref mu`)

A reference points to the same memory location as another variable.

```
mu x = 0
ref mu xRef = x
xRef = 1
print("{x}")        -- "1"
```

| Syntax            | Meaning                            |
|:------------------|:-----------------------------------|
| `ref x = y`       | Immutable reference, inferred type |
| `x: ref T = y`    | Immutable reference, explicit type |
| `ref mu x = y`    | Mutable reference, inferred type   |
| `x: ref mu T = y` | Mutable reference, explicit type   |

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
    if n < 1:
        0
    else if n < 2:
        1
    else:
        fib(n - 1) + fib(n - 2)
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

Functions can also be declared with type `fn` / `mu fn` to be set later. This type is called a **function pointer.** It lets you treat functions that same way you do with variables.

```
action: mu fn(int, int): int
add(a, b) = a + b
sub(a, b) = a - b
action = add
print("1 + 1 = {action(1, 1)}")  -- Prints "2".
action = sub
print("1 - 1 = {action(1, 1)}")  -- Prints "0".
```

#### Parameter Modifiers

Function parameters can be declared like variables. Likewise, you can modify their mutability and reference-ness the same way.

```
increment(x: ref mu int) =
    x += 1

mu y = 0
increment(y)
```

What each modifier means changes the functionality:

| Modifier | Copies/References | Mutable in function |
|:---------|:------------------|:--------------------|
| *(none)* | Copy              | **No**              |
| `mu`     | Copy              | Yes                 |
| `ref`    | Reference         | **No**              |
| `ref mu` | Reference         | Yes                 |
| `out`    | Reference         | *Yes, must be set*  |

Another type of parameter is `out`. `out` parameters are guaranteed‑set references. They behave like `ref mu`, except they begin the function in an *unset* state. Use an `out` parameter when a function’s job is to produce a value rather than consume one.

An out parameter must not remain unset in any branch of the function. It must either be:

- Assigned directly, *or*
- Passed to another function that also takes an out parameter.

This guarantees that the variable is initialized after the call completes.

```
setInt(out i): void =
    i = 3

x: mu int
setInt(x)
print("{x}")    -- Prints "3"
```

This is fine, but what if you just wanted to grab the `out` variable in one go? Normally you could just write `n = setInt()` — but `out` parameters don’t return values. We can inline a variable from the `out` parameter with the keyword `as`. Just like how we use `as` for aliasing, this makes it clear that we're expecting a new variable at the call site. That way it's *explicit* that we're intentionally declaring a new variable instead of passing in an existing one.

```
setInt(as n)    -- Declare a new variable as `n` that gets set by `setInt`.
print("{n}")    -- Prints "3"
```

`setInt(as n)` → "Set int as n"—This makes it clear what you're trying to do.

Using `as` makes the intent unambiguous: you are creating a variable, not passing an existing one. `as` already means "bind this to a name" throughout the language:

```
{x as y} = {x: 2}                 -- destructuring
match choice is Second(x as val)  -- pattern matching
method(self as this)              -- self aliasing
setInt(as n)                      -- out parameter
```

Languages that use return values for this kind of thing (`n = setInt()`) imply the value comes out of the function through the normal return channel, which is misleading when the mechanism is actually a reference parameter. `setInt(as n)` makes the call-site declaration explicit without requiring you to pre-declare a `mu` variable just to hand it in.

Parameters can be made optional with the `opt` modifier. This distinguishes them from `T?` which means a required parameter that's an option type. The parameter must be unwrapped before it can be used.

When calling a function with `T?`, you give it a value of `T?`. When calling a function with `opt T`, you give it a value of `T`. 

`||` is the *`None`-coalescing operation.* It unwraps a `T?` into a `T`. In other languages, this means `or`. However, None/null coalescing isn't much different from a logical OR. They both have the same order of operations, and most languages who don't have it just use `||` instead. Mulang makes `or` and `||` distinct so that the intention is clear. Other languages use `??` or `?:`, but these would conflict with `?` so `||` was chosen instead.

```
addOptional(a: opt int, b: opt int): int =
    aVal = a || 0             -- Coalesce optional arguments with default value 0.
    bVal = b || 0             -- This unwraps their value if they exist or set them to 0.
    aVal + bVal               -- Add the unwrapped values.

print("{addOptional()}")      -- Prints "0"
print("{addOptional(1)}")     -- Prints "1"
print("{addOptional(1, 1)}")  -- Prints "2"
```

- `T?` describes the **type of the value** — the caller is passing in an option and is responsible for handling `None`.
- `opt T` describes the **presence of the argument** — the caller can omit it entirely, and the function handles the absence.

When calling a function with `T?`, you give it a value of `T?`. When calling a function with `opt T`, you give it a value of `T`. 

```
optionalParam(val: opt int) =
    if val is Some(x):
        print("Some({x})")
    else:
        print("None")

requiredParam(val: int?) =
    if val is Some(x):
        print("{x}")
    else:
        print("None")

x: int = 5
optionalParam(x)         -- Prints "Some(5)"
optionalParam()          -- Prints "None"
requiredParam(Some(x))   -- Prints "Some(5)"
requiredParam(None)      -- Prints "None"
-- requiredParam(x)      -- Error: `x` is not `int?`
-- requiredParam()       -- Error: missing parameter `val: int?`
```

`opt` parameters can aslo have default values. Use `=` to to give an optional parameter a default value. This will make it a type `T` if it's used. 

```
addOptional(opt a = 0, opt b = 0): int = a + b     -- `a` and `b` are always `int`s

print("{addOptional()}")      -- Prints "0"
print("{addOptional(1)}")     -- Prints "1"
print("{addOptional(1, 1)}")  -- Prints "2"
```

- `x: opt T` — `T?`
- `opt x` — `T? *inferred*
- `x: opt T = default` — `T`
- `opt x = default` — `T` *inferred*

Use `...` to collect all variables into a single variable. The variable should be type `T#` (an array).

```
addAll(...nums: int#): int =
    sum: mu int = 0
    loop n in nums:
        sum += n
    sum

print("{addAll()}")         -- Prints "0"
print("{addAll(1)}")        -- Prints "1"
print("{addAll(1, 2)}")     -- Prints "3"
print("{addAll(1, 2, 3)}")  -- Prints "6"
```

A name is optional after `...`. You can use the symbol by itself to pass it to another function or itself in a functional loop. Use `| () = ` immediately after a function defintion. The first parameter to match will go.

```
addAll(x: int, ...): int =
    x + addAll(...)
| (x: int): int =
    x
| (): int =
    0

print("{addAll()}")         -- Prints "0"
print("{addAll(1)}")        -- Prints "1"
print("{addAll(1, 2)}")     -- Prints "3"
print("{addAll(1, 2, 3)}")  -- Prints "6"
```

#### Capturing

Immutable variables can be captured without any issue. They can't change, so they can't affect the output of the function. And if you try to set an immutable variable within a function, it will only get shadowed within the scope of the function. This includes capturing other functions which are also immutable by default.

**By default, functions cannot capture mutable variables.**

This is an intentional decision for better safety in Mulang. Functions don't automatically capture mutable variables. Instead, any variable set inside a function is treated like a new variable. This helps prevent accidentally mutating a variable that you didn't mean to and encourage good functional programming practices.

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

To capture a mutable variable, write `/` at the end of the function signature. This goes after the return type `: T` *(or after the parameter if inferred)* and before the equals sign `=`. List each *mutable* (`mu`) variables that the function uses. This helps make it easy to see which functions depend on mutable variables and which ones don't since `/` doesn't appear in type notation or anywhere else to the left of the equals sign `=` in function signatures except for captures. You can think of it like division where `1/0` is an error. In this case it means that this function depends on these variables or else there will be an error, so the program will take special care to make sure they remain in scope for the functon.

This follows the same practice that `import` and `inherit` where all words in a given context are listed out clearly so that there are no accident name collisions or hidden gotchas.

```
amount = 1                 -- Immutable variable, doesn't need to be captured.
mu count = 0               -- Mutable variables, must be captured with `/`.
mu squared = 1
mu cubed = 1
   
addCount(): int / count / squared / cube =     -- Capture 3 variables at once.
    count += amount                            -- Mutate captured variables inside the function.
    squared = count * count
    cubed = squared * count

addCount()
addCount()
addCount()
print("{count}, {squared}, {cubed}") -- Prints "3, 9, 27"
```

Error messages will highlight cases where someone would be confused about `/` in a function signature:

**Forgot to capture a mutable variable:**
```
mu count = 0
addCount() =
    count += 1  -- Error here
```
> `count` is mutable but not captured. Did you mean `addCount() / count =`?

**Accidentally tried to capture an immutable:**
```
x = 1
f() / x =
    x + 1
```
> `x` is immutable and is captured automatically — remove `/ x` from the signature.

**Captured a variable that doesn't exist in scope:**
```
f() / ghost =
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
> `count` is mutable but not captured by this lambda. Did you mean `fn(x) / count =`?

**The error should appear at the mutation site, not the signature**, and always suggest the fix explicitly. 

#### Lambda Functions

Define a function within an expression with the keyword `fn` in the pattern `fn(x) = x`. This is useful for passing functions to other functions.

```
fn(arg) = _

fn(arg) =
    _

(fn(arg) =
    _
)
```

```
map(array, func) = [...loop x in array then func(x)]
array0 = [1, 2, 3, 4]
array1 = map(array0, fn(x) = x + 1)   -- Inline
array2 = map(array0, fn(x) =          -- Multi-line
    if x < 2:
        x - 1
    else:
        x + 2
)
```

A name is optional. Adding a name creates an immutable reference of the function itself.

```
doThing(fn callback(val) =
    if val > 0:
        callback(val - 1)
    else:
        print("done")
)
```

The same keyword for creating functions is also used for function type notation. If this seems confusing, just remember where the context is: if it's being used like a type, it means a *function pointer type*; if it's being used like a value, it's a *lambda function*.

```
action: mu fn(int, int): int    -- Declaring a variable with a function type.
action = fn(a, b) = a + b       -- Passing to the variable with a function value.
```

Note that if you try to declare a function with the name `fn`, it will throw an error. This prevents potential gotchas and silent errors. In order to return a function, you need to write out `return fn` at the start of the line so that it's clear you aren't trying to declare a function.

```
(-- This is an error because you are trying to declare a function with the name `fn`:
fn(a, b) =
    a + b
fn(1, 2)
--)

-- This is not an error; it's a function that returns another function:
curryAdd(a: int): fn(int): fn(int): int =
    return fn(b) =
        return fn(c) =
            a + b + c

curryAdd(1)(2)(3)
```

Because this is a common practice, you can leave the `=` so that there isn't so much indententing. The rest of the scope at that point becomes the next function body.

```
curryAdd(a: int) =          -- Inferred return type.
    return fn(b: int)       -- No equals sign or identing.
    return fn(c: int)       -- Inside the second function.
    a + b + c               -- Inside the last function, final return value.

curryAdd(1)(2)(3)
```

Capturing also works inside lambda functions just like with named functions.

```
mu count = 0
forEach([1, 2, 3, 4], fn(x) / count =
    count += x
)
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

#### Contextual Parameters

Functions may require context variables by declaring them with `$name: type`. These are resolved from the calling scope rather than passed as arguments. Whenever you see `$` inside a function, think *"this comes from the surrounding context rather than a direct argument."* They are declared like captured variables using `/`. You can think of it like a variadic capture that's based on the *context* at function call.

```
addContext() / $a: int / $b: int =
    $a + $b

do:                         -- Issolate parameters to this scope
    $a = 1
    $b = 2
    result = addContext()   -- Uses $a and $b from context.
    print("{result}")       -- Prints "3"
    $a = 3                  -- Change a parameter.
    result = addContext()   -- Uses new $a
    print("{result}")       -- Prints "5"

-- Exit scope.

-- result = addContext() -- Error: $a and $b aren't defined.
```

The type can be inferred like other parameters. 

```
addContext() / $a / $b =
    $a + $b
```

Sometimes it's hard to tell where contextual variables are coming from. In that case, you could use a decorator like `@defined` to assert that variables are defined in a the scope.

```
@defined($a, $b)    -- Will throw at compile-time if $a or $b are undefined.
addContext()
```

The contextual variable prefix makes it clear that anything with `$` is a contextual variables that's shared with functions in the *same scope.* `$` by itself is just one specific variable which gets set by the return value of a *pipeline* expression.

Optional contextual parameters use `opt T`. This will wrap it in an option type `T?` if it doesn't exit or match the type. 

```
isThereX() / $x: opt int =
    match $x is
    | Some(_): print("$x exists")
    | None:    print("No $x")
```

*Note the difference…"

- `$x: opt T` – Optional type `T` parameter.
- `$x: T?` - **Required** type `T?` parameter.

Declaring the pipeline context `$` in the function will require that function to be pipelined with a certain type. Recall that `$` on its own is a variable. This makes it easy to see pipeline functions. When calling them, pass the `$` argument to `|>`

* `x |> f()` → `$` is `x`

```
AddArgs :: { a: 1, b: 2 }

pipeAdd() / $: AddArgs =    -- Needs an AddArgs object piped to it.
    $.a + $.b

result = AddArgs(a: 1, b: 2) |> pipeAdd()   -- This sets $ to an AddArgs
print("{result}")      -- Prints "3"
```

You can also destructure from the pipeline context to turn members into local variables.

```
pipeAdd() / $: AddArgs =
    {a, b} = $        -- Get a and b from the pipeline context.
    a + b


pipeAdd() / $ as {a, b}: AddArgs =   -- Get a and b from the pipeline context.
    a + b

pipeAdd() / $ as ctx: AddArgs =   -- Or create alias for context
    ctx.a + ctx.b
```

If a function doesn't use the pipeline context, you can pass it in manually with `f($)` / `f($, x)` / `f(x, $)` or any other argument position, or you can spread it into all arguments with `f(&$)` or `f $`. 

```
add() & AddArgs{a, b} =   -- Regular function with named parameterrs
    a + b

result = AddArgs(a: 1, b: 2) |> add $     -- Pass to add directly.
print("{result}")      -- Prints "3"
```

---

## Types

Notation:

- **Basic**: `name: type`
- **Functions**: `fn(type, type): type`
- **Options**: `type?`
- **Results**: `type!` or `type!E` where `E` is an exception type
- **Arrays**: `type#` or `type#N` where `N` is the length
- **Multi-dimensional Arrays**: `type##`, an extra `#` for each dimension, each dimension can be fixed or dynamic: `type#N#`, `type##N`, `type#N#N`, `type#N##`, etc.
- **Dictionaries**: `type#type`
- **Inferred**: omit the annotation entirely

### Built-in Types

Some built-in types include `int`, `uint`, `float`, `bool`, `char`, `str`, and `ptr`. Note that although built-in types use lowercase names, they are not *keywords*. This is just a naming convention. *It's recommended that users create custom types with capitalized names to differentiate from built-in types.*

```
myInt: int = -1234
myInt: uint = 5678
myFloat: float = 12.34
myBool: bool = True
myChar: char = 'a'
myStr: str = "Hello"
myPtr: ptr = ExternalLib.getSomething()
```

You can get the type of any variable with the compile-time function `typeof`. This fetches the type of that symbol at that point during compile time. *(See [Meta Functions](#Meta-Functions).)*

```
x = 0
y: typeof[x] = 1    -- Ensures that x and y have the same type.
```

You can also get the default value of any type with the compile-time function `default`. The type needs to have a default value defined which is yet to be determined how, but they're already defined for basic types.

```
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

There will be an API for defining the default value of custom types, but that is outside the scope of this document. Here is an example of what that might look like:

```
MyType :: struct =
    value: int

MyType :: impl[Default] =            -- Implement the Default proto
    getDefault() = MyType(value: 0)  -- Default value for this type
```

*This API is subject to change.*

You can also get the size of any type with the compile-time function `sizeof`. It returns a constant `uint` (unsigned integer) with the number of bytes of memory that type requires. The exact sizes of some types like `int` or `float` might vary, but you can rely on `char` and `bool` being 1 byte each. There's also the `void` type which represents no data. `ptr` depends on the pointer size of the system. 

```
sizeOfBool = sizeof[bool]   -- == 1
sizeOfChar = sizeof[char]   -- == 1
sizeOfVoid = sizeof[void]   -- == 0
sizeOfPtr  = sizeof[ptr]    -- == 4 or 8
```

Use `~` to convert one type into another. This only works if the result type of the expression is known. The symbol `~` plays on its general meaning of "about" or "roughly", like *"I left at about ~3:30."* 

```
x: float = 1.5  -- `x` is a float.
y: int = ~x     -- `y` is expecting an int, so call `int(x)`
print("{y}")    -- Prints "1"
```

If the result type cannot be inferred, you can call a type like a function around it. This only works if both the input type and output type are compatible.

```
x = 1
y = float(x)
print("{y}")     -- Prints "1.0"
```

#### Booleans

`bool` is a built-in enum type with its only members being `False` and `True`. This means you can also pattern match with a bool, although it's recommended to use `if`/`else` instead. Enum-members are generally capitalized, and this matches Python's `True` and `False` convention. 

```
match value is True:
    print("It's true!")
| False:
    print("It's false!")
```

These two are the same thing.

```
if value:
    print("It's true!")
else:
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

It's essential to put spaces between operators at times so that they don't get confused for something else.

```
1++1  -- Creates an array       → [1, 1]
1+ +1 -- Add 1+1                → 2
1--1  -- Just 1 with a comment  → 1
1- -1 -- Subtract 1-(-1) → 1+1 → 2
```

It's not likely anyone would mark a positive number and add it on the right hand side, and neither is it likely anyone would flip the sign of a number and subtract it on the right-hand side. For most people, both `1+(+1)` and `1-(-1)` would just be written `1+1`. 

#### Characters

Characters or `char` are written with apostrophes (`'_'`) *(also called single quotes).* They store 1 byte of data. You can also do arithmetic on them like with numbers.

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

#### Strings

|    Form     | Purpose                                                                    |
|:-----------:|:---------------------------------------------------------------------------|
|    `"…"`    | Regular string with `{expr}` interpolation                                 |
|  `"""…"""`  | Multiline with interpolation; whitespace trimmed at closing `"""` position |
|   `''…''`   | Inline raw string, no interpolation or escaping                            |
| `@"""…"""@` | Multiline raw string; `@` count must match to close                        |

Strings are marked with quotation marks (`"…"`) *(also called double quotes)* and can be formatted with curly braces (`{expr}`) in the string. Use a backslash to write a literal opening curly brace (`\{`). Note that string insertion and named tuples both use curly braces. This shouldn't be an issue though since they're used in different contexts. Inserted expressions are implicitly converted to strings, so using `str()` or `~` isn't necessary. This string is only allowed on a single-line, but literal-line characters can be inserted with `\n`.

```
name = "world"  
hello = "Hello, {name}!"
helloEscaped = "Hello, \{name}!"
lines = "This \n string \n has \n linebreaks."
```

Subsequent string literals will automatically concatenate, and the `++` operator can be used to concatenate non-literal strings.

```
str1 = "This" " string"
str2 = " is broken"
str3 = str1 ++ str2 ++ " into multiple parts."
print(str3)
-- Prints "This string is broken into multiple parts."
```

You can also write multi-line strings with `"""` (3 quotation marks). A common issue in programming languages is how to fix the issue of leading whitespace in a multi-line string. Mulang uses significant whitespace, so unindenting the string wouldn't work. We don't want all the leading whitespace to be in the string, but how do we solve this? To fix this issue, whitespace gets trimmed at compile-time based on the positions of the last `"""`. Any spaces before it is automatically trimmed. Much like blocks, having too little indentation is a syntax error inside multi-line strings. This helps keep things readable and consistent and solves the whitespace issue inside strings.

```
do:
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

You can write a basic raw string with `''…''` (two apostrophes). Although apostrophes `'` are used for chars, an empty char isn't possible since the default char is written `'\0'` (null character). A common practice in programming languages uses the double quote `"` for formattable strings and the single quote mark `'` for raw strings, so it should be easy for any programmer to see the parallel. When you write `''`, every character after it **except for new-lines** is in the string until the closing `''`. Escaping with backslashes `\` and insertion with curly braces `{}` are disabled. This is for single-line raw strings only. If there's a line break before the closing `''`, then it's a syntax error. *(See below for multi-line raw strings.)*

```
rawString = ''It's okay to put an apostrophe (') in the string.''
filePath = ''C:\files\on\windows.txt''
template = ''Insert here → {{variable}}''
```

To make a multi-line raw string, add an at sign `@` before and after triple quotation marks `"""`. The number of `@`s must match to close the raw string. This symbol is also used for decorators such as `@unsafe`, so the two connect giving `@` the general syntactical meaning of a compile-time signal.

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

Sometimes, we just want to copy and paste string data without formatting it. To help with this, Mulang allows you to put a raw string enclosed in `@` signs at the start of a new line without breaking a block. Significant whitespace is temporarially disabled when the line starts with `@`-style raw string, and the parser ignores indentation significance while inside the raw string. All whitespace and characters are put into the string without formatting until it gets to the matching `''@` marker. Then, re-ident in the next line to resume the block. This is useful for debugging and embedding data in code.

```
do:
    do:
        do:
            do:
                nestedRawString =  -- Put raw string on the next line without indenting.
@"""                       
                       
    Indentation  
  doesn't matter     
 here.              

"""@  -- Right here: the string ends and the block resumes.
                -- Re-indent to return to the block.
                print("{nestedRawString}")
```

What it prints:

```
                       
                       
    Indentation  
  doesn't matter     
 here.              
                  
   
```

While this is possible, this is not recommended for the sake of legibility. For anything more complicated, it's recommended to save the string to a document and open the file inside your program, which will be handled by a separate library and lies outside the scope of this document.

`@"""…"""@` inherits the `"""` closing-position trim anchor, so if you *do* want controlled indentation trimming, you still get it from where you place the closing `"""@`. That makes the two systems consistent with each other and interchangeable.

#### Arrays

Array types are declared with the hash symbol (`#`). This was chosen because the `#` is commonly used for numbers. For example, `#1` is read `number 1`. A number after the `#` makes it a fixed length array `type#N`. Arrays are statically sized when written `type#N`; `type#` is the dynamic form. Items are separated with commas (`,`). Index is done with the `#[]` operator. When the index is a number literal, you can omit the `[]`, similar to `.` (member access) for tuples.

```
list: int#4 = [1, 2, 3, 4]
print("length of list: {len(list)}")
compressedList = [list#0 + list#1, list#2 + list#3]
doubleArray: int#2#2 = [[1, 2], [3, 4]]
```

```
i = 2
print("{list#[i]}")   -- Prints "3" because list#2 is 3
```

This builds on the visible symmetry between type notation and their value expressions:

| Type | Notation | Expression |
|:---------|:------:|:--------:|
| **Results**  | `T!`  | `x!`  |
| **Options**  | `T?`  | `x?`  |
| **Pointers** | `T^`  | `x^`  |
| **Arrays**   | `T#N` | `x#n` |

```
item = doubleArray#1#0    -- The 2nd row, 1st column
print("{item}")           -- Prints "3"
```

Raw indexing is generally frowned upon. Mu prevents you from accessing an array unless can be guaranteed to be within range. If you wish to access an index without safety, mark your code with `@unsafe`. 

```
@unsafe                     -- Allow raw indexing.
do:
    list: int# = getData()
    print("{list#100}")     -- Might fail.
```

In general, you'll mostly be using arrays by iterating or piping them. 

```
loop x in list:
    print("{x}")     -- No need to use `#`
```

If the left hand side of `++` isn't an array, it will automatically create one. A non-array on the right-hand side will push it to the end of the resulting array. In this way, you can abandon square brackets notation for array literals entirely and just use `++`.

```
a = 1 ++ 2 ++ 3 ++ 4  -- == [1, 2, 3, 4]
b = 0 ++ a ++ 5       -- == [0, 1, 2, 3, 4, 5]
c = b ++ 6 ++ 7 ++ 8  -- == [0, 1, 2, 3, 4, 5, 6, 7, 8]
```

Use the spread operator `...` to spread an array into another array. This must be the first prefix operator in an expression, and it must be in a compatiable array or tuple literal. It always goes last in the slot's expression, so extra parentheses aren't necessary: `...a ++ b` == `...(a ++ b)`.

```
a = [1, 2, 3]
b = [0, ...a, 4]                -- == [0, 1, 2, 3, 4]
c = a ++ b                      -- == [1, 2, 3, 0, 1, 2, 3, 4]
d = [0, ...a ++ b, 5, ...c, 6]  -- == [0, 1, 2, 3, 0, 1, 2, 3, 4, 5, 1, 2, 3, 0, 1, 2, 3, 4, 6]
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

*Mulang separates tuple and array spread to preserve =type safety and avoid Python‑style runtime surprises. `&` always preserves tuple structure; `...` always preserves array structure.*

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
print("{ dict#["b"] }")   -- Prints "2"
print("{ dict#b }")       -- Prints "2"
```

Like with arrays, you can't access dictionaries directly unless it's guarenteed to succeed. If not, you should mark it in a block with `@unsafe`. 

```
@unsafe                      -- Allow raw dictionary accessing
do:
    dict: int#str = getData()
    print("{dict#data}")     -- Might fail.
```

You can iterate through a dictioary like with arrays.

```
loop (val, key) in dict:
    print("{key} = {val}")
```

#### Pointers

Although most things can be achieved without manual manipulation of pointers, some low level code requires it. Opaque pointers use the type `ptr`. This represents a pointer where the type that it represents is unknown. It's ideal for FFI where you need to pass a pointer a around and let an external library handle it. 

```
result: ptr = ExternalLib.getSomething()
ExternalLib.doSomethingWith(result)
```

You can manually check if the pointer is `Null` and give a helpful error message in scripts. `Null` is a `ptr` type that points to the NULL pointer. It's distinct from `None` which is an varied option type `T?`. 

```
result: ptr = ExternalLib.getSomething()
if result == Null:
    print("No result found")
    raise NotFound
```

You can use `||` to fallback to another pointer if you get a `Null`.

```
result: ptr = ExternalLib.getSomething() || fallback
```

A standard library will be made to safely handle pointer dereferencing and do pointer arithmetic, but that is outside the scope of this document. Here is an example of how it might work:

```
mu x = 0             -- Create a local mutable variable.
xPtr = getMuPtr(x)?  -- Map `Null` to a option type, branch if it's `None`, return `Some(ptr)` if it's not and unwrap it with `?`.
xPtr.set(1)!         -- Safely set the pointer and branch if there's an error.
print("{x}")         -- "1", the pointer successfully mutated `x`.
```

Sometimes, it's necessary to dig deep into the unsafe territory. Mulang normally prevents you from doing this unless you mark the code with `@unsafe`. The `^` is the symbol associated with pointers, analogues to `?` for options, `!` for results, and `#` for arrays. It can be used in type notation, but it's also the operator to dereference a pointer. Thy type must be known at compile-time. Dereferencing an opaque pointer `ptr` is a compile-time error. In the type notation, `T^` prevents the pointer from mutating its memory or `T^mu` allows mutation with `^ =` (dereference + assignment). 

```
@unsafe                      -- Allow pointer manipulation in this block.
do:
    mu x = 0                 -- `ptr` type takes a reference and creates a generic pointer.
    xPtr: int^mu = ~ptr(x)   -- Convert `ptr` to `int^mu`, type is known.
    xPtr^ = 1                -- Mutate the memory.
    print("{xPtr^}")         -- Prints "1".
    print("{x}")             -- Prints "1".
```

Pointer types have 2 kinds of mutability: one for the reference, and one for the pointer variable itself. Here is a table of each kind and what it means.

|    Type   | What It Means                         | Can reassign pointer | Can mutate memory | *Think…* |
|:---------:|:--------------------------------------|:--------------------:|:-----------------:|:---------|
|    `T^`   | immutable pointer to immutable memory |       **No**         |      **No**       | *This will never change.* |
|    `T^mu` | immutable pointer to mutable memory   |       **No**         |        Yes        | *Like a more low-level `ref mu`.* |
| `mu T^`   | mutable pointer to immutable memory   |         Yes          |      **No**       | *I need to switch what I'm looking at.* |
| `mu T^mu` | mutable pointer to mutable memory     |         Yes          |        Yes        | *I need full control.* |

---

## Control Flow

All branching constructs share the same block / inline pattern:

```
-- Block form
keyword subject:
    body
keyword:
    body

-- Inline expression form
keyword subject then expr
keyword expr
```

The presence of `:` signifies if a keyword is in block mode or inline mode. `then` is used to separate a subject and expression when a keyword block is in-lined. 

**Any variables created in the subject field shadow any variables in the parent scope.** This prevents accidental mutations and unintended side-effects. 

#### `do`

```
do:
    _

do.label:
    _
```

Creates a new scope. Its value is the last expression evaluated.

```
x =
    do:
        y = 1
        y + 1       -- block's value is 2
```

Any starting block can be given a label. Use this to call `break` on a specific block. This uses the same convention as membership access `.`. The distinction is that this only comes after a keyword such as `do.`, `loop.`, or `break.`. 

```
do.block:
    break.block
```

#### `if` / `else`

```
if cond then expr else expr
if cond:
    _
else:
    _
```

Basic Boolean branching.

```
x = if x > 0 then x else -x

if x > 0:
    print("positive")
else:
    print("non-positive")
```

Use `or`/`and` to compare multiple booleans at once.

```
a = True
b = False

if a and b:         -- True and False == False
    print("This will not print")
else if a or b:     -- True or False == True
    print("This will print")
```

#### `loop`

The universal loop keyword. All loop forms share the same `loop` keyword.

```
-- Unconditional (break manually):
loop:
    _
    break

-- While condition is true:
loop cond:
    _

-- For-each:
loop x in expr:
    _

-- Do-until (runs at least once):
loop:
    body
until cond
```

`else` after `loop cond` runs if the loop body never executed.

```
loop False:
    pass
else:
    print("Never ran")
```

Inlined `loop x in` returns a lazy iterator collected with `...`.

```
doubled = [...loop x in list then x * 2]
```

Destructuring works in loop variables.

```
loop (x, y, z) in listOfTuples:
    print("{x}, {y}, {z}")
```

Pattern matching works. All patterns must have fallbacks. *(See [Pattern Fallback](#pattern-fallbacks).)* This is because if you have an array/iterator of enums, it would be hard to determine if they're all a particular pattern. This ensures any mismatches are handled inside the loop. 

```
loop Pattern(opt x) in listOfPatterns:
    if x is Some(x):
        print("Found match: {x}")
```

This can be combined with `then opt` to automatically skip when there's a mismatch.

```
loop Pattern(opt x) in listOfPatterns then opt:    -- Add `then opt` to the end of the loop's subject.
    print("Found match: {x?}")                     -- If `x` is None, the loop skips to the next iteration
```

#### `break` / `continue`

Both accept an optional label to target an outer loop.

```
loop.outer x in 0..100:
    loop.inner y in 0..100:
        if x * y >= 100:
            break.inner
        if x * y == 77:
            break.outer
```

Pass a value to `break`, that becomes the value of the block, analogous to `return` for function.

```
x = do.block
    if cond:
        break.block 5
    4

print("{x}")     -- Prints either "4" or "5"
```

### Pattern Matching

```
match expr is
| Pattern1(x): expr
| Pattern2(y || default): expr
| else: expr
```

The next control flow methods are based on pattern match. Generally, you see the word `is`, you next thing to expect after it is a pattern: `value is Pattern(x)`.

#### `match` / `is`

Enum/exception branching. Exhaustive by default. `| else:` for the default case.

```
match expr is ptrn:
    _
| ptrn:
    _
| else:
    _
```

Each pattern starts with `|`. This was chosen because pattern matching is a core feature in Mulang and a core identity of functional programming. This keeps it much briefer than the usual `switch`/`case` statement, closer to the pattern matching found in functional programming languages. 

Its syntax is a bit different than most blocks. You start with `match expr is` with no colon. Each case starts with `|`. This is a special case since each pattern starts a block. The colons are put at the end of the pattern on each case, with each case being its own block. 

The patterns map to the type passed in after `match`, so you only need to reference the members of that type in each pattern.

```
match choice is First:     -- Each pattern case starts its on block.
    print("First")         -- Ident for the new block.
| Second(x):               -- Continue this for each case.
    print("Second({x})")   -- ……
| Third{val}:              -- ……
    print("Third \{ val={val} }")
                           -- All choices were exhausted, so no `| else:` is necessary.
```

The first case can also be put on the next like this, making it easy to line up all the patterns:

```
match choice is          -- Put case on next line
| First:                 -- Start it with `|`
    print("First")
| Second(x):
    print("Second({x})")
| Third{val}:
    print("Third \{ val={val} }")
```

The inline form keeps `:` after patterns. This makes it easier to read and take up less space. Patterns are usually words, so you can visually sequence it into pattern/expression pairs: `| ptrn: expr | ptrn: expr | ptrn: expr` etc.. Other symbols like `=>` wouldn't work because it would clash with operators. `:` also follows the `key: value` pattern that tuples and dictionaries use. 

```
tuple = (a: 1, b: 2)
dict = [x: 3, y: 4]
restult = match x is Ptrn1: 5 | Ptrn2: 6 | else: 7
--
message = match e is OpenError{filename}: "Open error: {filename}" | else: "Unknown error"
```

You can have multiple patterns match to one case. If any of the patterns destructure with a variable, the same variable name and type must be in all patterns. If not, use a wildcard `_` in each pattern or omit the tuples part entirely to disable destructuring. Otherwise, use a fallback in the pattern.

```
match choice is
| First:
    print("First")
| Second(val) | Third{val}:                  -- `val` must be in all patterns
    print("Second or Third, val={val}")
```

```
-- `First` doesn't have any values, so destructuring must be disabled.
match choice is First | Second | Third:
    print("First, Second, or Third")
```

```
-- Fallback, `val` is converted to option type `T?`:
match choice is First | Second(opt val) | Third{opt val}:
    print("First, Second, or Third: {val || "None"}")
```

#### `fallthrough`

Proceeds to the next case, which must not destructure new values, unless fallbacks are used.

```
match choice is
| First:
    print("First")
    fallthrough
| Second(opt x):            -- `opt x` in pattern wraps the variable in an option
    if x is Some(x):
        print("Definitely Second: {x}")
```

#### Pattern Fallback

If a pattern can't be **guarenteed** for any reason, then you must have a **fallback.** There are two options available:

- __Optional binding:__ `Pattern(opt x)` — wraps `x` in type `T?`, `Some(x)` if it matched, `None` if it didn't
- __Default value:__ `Pattern(opt x = default)` — `x` is type `T`, if it didn't match `x` is set to `default`
- __Short hand:__ `Pattern(x || default)` — `x` is type `T`, if it didn't match `x` is set to `default`

#### Pattern Guards

Add `if` inside a pattern to conditionally match.

```
match choice is
| Second(x if x > 0):
    print("Positive: {x}")
| Second(x if x < 0):
    print("Negative: {x}")
| Second(x):
    print("Zero")
| else: _
```

#### `is` / `then`

```
expr is ptrn then expr
```

Extract a binding inline. Requires a guaranteed match or optional bindings.

```
-- Guaranteed match (exhaustive type):
result = value is Pattern(x) then x
```

```
-- With fallback (non-exhaustive):
result = value is Pattern(opt x) then x
result = value is Pattern(opt x) then x || "fallback"    -- Wrap in Some(x), then coalesce
result = value is Pattern(x || "fallback") then x        -- Automatic fallback
```

```
-- Multiple bindings:
result = value is Pattern(x || 0, y || 0) then (x, y)
```

```
-- Arbitrary expression over bindings:
result = value is Pattern(x || 0, y || 0) then x + y
```

Pairs naturally with pipelining.

```
getValue()
|> $ is Pattern(x || "fallback") then x
|> doSomethingWith($)
```

#### `=>` / `else`

```
expr => ptrn
expr => ptrn else expr
expr => ptrn else:
    _
```

The same thing as `is` but sets a variable in scope. This can be done with any pipeline assignment operator `=>`. 

```
getStuff() => Pattern(opt x)

print("{x?}")  -- unwrap x
```

```
getStuff() => Pattern(x) else raise Error("No match")  -- branch out when match fails

print("{x}")   -- x is guaranteed here
```

#### `if` + `is`

```
if expr is ptrn then expr else expr
if expr is ptrn:
    _
else:
    _
```

Combines the conditional branching of `if` with pattern matching of `is`. Useful if you want to destructure a single case of a sum type. This must be a pattern that matches the type of the value before `is`.

```
if value is Pattern(x):
    print("value is {x}")
else:
    print("value doesn't match")

something = if value is Pattern(x) then x else "fallback"
```

Like with other patterns, multiple patterns can be checked for at once with `|`. All patterns must go on the right of `is`. Any destructured variables must match in name and type.

```
if value is Pattern1(x: int) | Pattern2{data as x: int}:
    print("{x}")
```

`x if` is also available like before and follows the same rules. You can iteratively match nested enums. 

```
if nestedPattern is Pattern(Pattern(Pattern(Pattern(x if x >= 0)))):
    print("Phew! That was a lot of unwrapping for {x}!")
else:
    print("Either none of those nested options matched or x is negative.")
```

### `loop` + `is`

Loop while a pattern matches.

```
loop nextValue() is Some(x):
    print("{x}")
```

### `until` + `is`

Loop until a pattern matches. Bindings are in scope below the loop.

```
mu i = 0
loop:
    print("Attempts: {i}")
    i += 1
until getValue() is Pattern(x)

print("{x}")    -- x is guaranteed set here.
```

If `break` is reachable inside the loop, optional bindings are required.

```
loop:
    if earlyCondition:
        break
until getValue() is Pattern(opt x)

if x is Some(x):
    print("{x}")
```

### Error/Optional Control Flow

Error and null handling is done through the `try` and `opt` keywords. These blocks are for unwrapping the 2 monadic types in Mulang: *optionals* `T?` and *results* `T!`. When resolving a monadic, you go in *reverse order* that was notated by the type. Think of it like a box: you start from the outside and work your way in.

- `T?` is an *optional:* it may contain a value or be `None`.
- `T!E` is a *result:* it may contain a value or an error of type `E`.

| Mu's Type | Other Languages           | In Plain English                                              | Layers | Resolve Order             |
|:----------|:--------------------------|:--------------------------------------------------------------|-------:|:--------------------------|
| `T?`      | `Option<T>`               | An optional                                                   |      1 | `?`                       |
| `T!E`     | `Result<T, E>`            | A result with 1 possible error                                |      1 | `!`                       |
| `T?!E`    | `Result<Option<T>, E>`    | A result with 1 possible error of an optional                 |      2 | `!` *then* `?`            |
| `T??`     | `Option<Option<T>>`       | An optional of an optional                                    |      2 | `?` *then* `?`            |
| `T?!E!F`  | `Result<Option<T>, E\|F>` | A result with 2 possible errors of an optional                |      2 | `!` *then* `?`            |
| `T!E?`    | `Option<Result<T, E>>`    | An optional of a result with 1 possible error                 |      2 | `?` *then* `!`            |
| `T?!E?`   | `Option<Result<Option<T>, E>`    | An optional of a result with 1 possible error of an optional  |      3 | `?` *then* `!` *then* `?` |
| `T!E?!F`  | `Result<Option<Result<T, E>>, F>` | An reesult with 1 possible error of an optional of result with 1 possible error | 3 | `!` *then* `?` *then* `!` |

Think of each `?` or `!` on a *type* as a **layer.** When you call the same `?` or `!` in an *expression,* you **unwrap** that layer.

```
x: int!Error = getRiskyInt()             -- Get wrapped result value.
y: int = x!                              -- Unwrap the result.
                                         -- Which is equivalent to…
y = (match x is
    | Ok(val): val                       -- Get the Ok value.
    | Exception(e): return Exception(e)  -- Exit block, return Exception if a function
)
```

```
x: int? = getMaybeInt()   -- Get wrapped optional value.
y: int = x?               -- Unwrap the optional.
                          -- Which is equivalent to…
y = (match x is
    | Some(val): val      -- Get the Some value.
    | None: return None   -- Exit block, return None if a function
)
```

#### `try` / `except`

Error handling. Unwrap result types with `!` inside a `try` block. Unhandled exceptions propagate upward.

```
try:
    a = doSomething1(x)!
    b = doSomething2(a)!
    b
except Exception(e):
    print("Error: {e}")
    0
```

```
try:
    data = riskyOperation()!
    data2 = anotherRisky()!
    final = process(data, data2)
    final
except IOError(e):
    print("IO failed: {e}")
    defaultValue
except ValidationError(e):
    raise Err(e)
```

```
result: int = try:
    data = riskyOperation()!
    data2 = anotherRisky()!
    process(data, data2)
except IOError(e):
    print("IO failed: {e}")
    0              -- fallback value
except ValidationError(e):
    raise Err(e)   -- Escape function with error
```

Inline form.

```
result = try divide(1, 0)! except _ then 0.0
```

Using `!` inside a function automatically infers a result return type `T!`.

```
riskyFn(a: int): int! =
    b = step1(a)!
    c = step2(b)!
    c
```

#### `opt`

None-coalescing. Unwrap option types with `?` inside an `opt` block. If any `?` returns `None`, the block short-circuits.

```
opt:
    a = getA()?
    b = getB()?
    c = getC() || 0
    print("{a + b + c}")
else:
    print("Didn't work")
```

Inline form.

```
x = opt f(a?) || "fallback"
```

Using `?` inside a function automatically infers an option return type `T?`.

```
addStuff(a: int, b: int): int? =
    x = getA()?
    y = getB()?
    x + y
```

Nested options unwrap with multiple `??`.

```
unnest(x: int??): int? = x??
```

Chain `||` to try multiple options with a final fallback.

```
getFirst(a: int, b: int, c: int): int =
    getA(a) || getB(b) || getC(c) || 0
```

Other examples:

```
crunchData(): int?!Error!CustomError =    -- Multiple error types
    value: int? = someFunc()?
    -- Option to Result
    data: int = value || raise CustomError("Not found")      -- Exist function on fallback
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

Return out of the function with an exception value. The function must return a result type `type!`. If an except type is also declared `type!except`, then the type passed to `raise` must match.

```
-- `type!` is inferred:
alwaysFail() =
    raise MyError("error message")

try:
    alwaysFail()!
except e:
    print("{e}")
```

#### `yield`

Exits out of a function with an `iter[_]` type. The return value of the function must be of type `iter[T]` where T is the yield type. When you have `yield` in your function, the actual return value in the function body is discarded, and using `return _` in it is a compile-time error. Use of `yield` will infer the return type to be `iter[_]`. 

```
count(n: int): iter[int] =
    loop i in 0..n:
        yield i
```

If you use `yield`, you can only use a void `return` to exit the function. 

```
countUntil(i: mu int, max: int): iter[int] =
     loop:
        if i >= max:
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
    loop i in 0..n:
        val = await fetch(i)
        yield val

asyncCollect(n): async[int#] =
    ret: mu int# = []
    loop (await x) in asyncIterFn(n):
        ret ++= x
    ret
```

Unlike in other languages where *promises* or *futures* can either resolve **or reject,** async types in Mulang **only resolve.** Instead you can use a result type `T!E` inside an `async[T!E]` function. Unwrap it like you would a result type. Because this is common, `await` has special rules in regards to the `!` and `?` opeerators when placed after it.

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

Variable and functions primarily use the equals sign (`=`) and are for storing actual data within a program, but there's another type of binding used for abstract values for the compiler to know about like constants, types, inline-functions, and generics. This type of declaration is **constant**; in other words, they cannot be **mutated** or **shadowed**. Depending on what it is, subsequent `::` of the same name will modify its definition. The most common is `:: impl` with adds methods and static variables to a meta binding. 

Meta bindings are meant to resemble definitions. *"This is that."* Only when things get complicated should you have to expand on that. It should be easy and simple to define something. They are like the language file of your API, telling the compiler how to speak your own custom language—not just the language that you *speak* but the language that you *think* in.

### Aliases

Assigning a type after `::` creates an alias. 

```
numberType :: int
```

This alias is unique to the scope. Modifying it only affects the alias and not the original type. This prevents accidental conflictions between modules. *(See [Implementation](#Implementing-impl).)*

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

Tuples use commas (`,`) to separate components for both positional (`()`) and named (`{}`) tuples. This follows the same rules that function parameters does. *(See [Function Declarations](#Function-Declarations).)*

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

Opaque types like primitives and enums coerce into a tuple of one, so creating a product type of them creates a positional tuple, e.g. `int & float & char` becomes `(int, float, char)`. Structs convert to named tuples unless declared with an `@opaque` decorator, in which case they behave like opaque types. The `void` type coerces to an empty tuple `()`. *(See [Decorators](#Decorators)* for more informations on available decorators.)

Combining empty tuples produces an empty tuple `() & () == ()`. The same is true for empty named tuples `{} & {} == {}`. This also means that empty positional tuples and empty named tuples are equivalent `() == {}`. Both tuples have zero dimensions in both positional and named components; therefore they are equivalent. Saying `() & {} & ()` or `{} & ()` and any combination of empty tuples all produce an empty tuple. If you think about it, this makes sense. They all represent nothing, like a cup that can holds 0mL of water. If you stacked a bunch of 0mL cups, you would still not be able to hold any water in it. Is it even a cup then? That's why empty tuples are treated like `void`, which verbally represents nothing, because they're all nothing. Therefore `void == () == {}`. The 3 types are all the same. 

This also means that a void function and a function that returns an empty tuple are the same. *Empty positional tuples, empty named tuples, and void are fully interchangeable at the type level.*

```
voidFn(): void = ()
voidFn(): () = ()
voidFn(): {} = ()
```

We say that a void function returns nothing. Well, that's what an empty tuple is: *nothing*. So there isn't any issue here. This is one of Mulang's strengths. It's not afraid to make bold claims and go against the grain. This has a lot of potential in math and logic. 

### Constants

Constants are declared with type `const T`. The `const` makes it clear that we're defining a constant value and not an alias or another type of meta binding. After the type is an equals sign `=` and it's value. The type can be inferred with `:: =`. 

```
PI :: const float = 3.14159
E  :: = 2.71828                -- Inferred
```

You can also bind a function to a constant with `const fn`. Define it like you would with basic functions. The `const fn` can is opitional for shorthand form, just `:: (param) =` is needed. This can be useful if you need to pass a function multiple times but don't want it to be outputted when compiled.

```
IDENTITY :: const fn(x) = x     -- Full definition.
addOne :: (x) = x + 1           -- Shorthand.
value = addOne(2)               -- Means (fn(x) = x + 1)(2), result is 3.
array = map([1, 2, 3, 4], addOne)
```

### Structural Types (`struct`)

Structs are product types—or in other words—plain data containers. They cannot extend other structs, but can inherit members of other structs. *(See [Inheritance and Visibility](#Inheritance-and-Visibility).)*

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

Structs are transparent. They can be destructured like named arrays. Use `@opaque` if you need to disable this. That will treat the struct type to be an opaque type and prevent it from being inherited with `inherit`. *(See [Inheritance and Visibility](Inheritance-and-Visibility).)*

```
TransparentThing :: struct =
    a: int
    b: int

{a, b} = TransparentThing(a: 1, b: 2)
print("a: {a}, b: {b}")

@opaque
OpaqueThing :: struct =
    a: int
    b: int

o = OpaqueThing(a: 1, b: 2)
print("a: {o.a}, b: {o.b}")
```

*(See [Decorators](#Decorators) for more informations on available decorators.)*

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
| First:
    print("first!")
| Second(_):
    print("second!")
| Third{_}:
    print("third!")
```

### Exception Types (`except`)

Exceptions are like enums but used for error handling. See "Error Handling" for more details. Instantiation works the same as enums.

```
MyException :: except =
    OutOfBounds
    DivideByZero(int)
```

Because exceptions group together to form custom exception types per `try` block, each member must match with their full name. This helps prevent potential name-clashes.

```
try
    risky()!
except MyException.OutOfBounds:
    print("Out of bounds!")
except MyException.DivideByZero(x):
    print("Can't divide {x} by zero!")
```

Alternatively, you can give each exception an alias.

```
OOB :: MyException.OutOfBounds
DBZ :: MyException.DivideByZero

try
    risky()!
except OOB:
    print("Out of bounds!")
except DBZ(x):
    print("Can't divide {x} by zero!")
```

### Prototypes (`proto`)

A `proto` is an abstract interface — a named contract with no data. It is equivalent to a `virtual class` (C++), `trait` (Rust), or `interface` (Java, TypeScript, etc.) in other languages. Each member is a function, also called a **method**. Methods that have a parameter named `self` at the beginning will be called like methods on the instance of that type, i.e. `self.method(…)`. This is equivalent to saying `typeof[self].method(self, …)`.

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

print("{MyStruct.staticValue}")
```

A method with `self` in the first parameter are callable as a method on the instance. `self` refers to the instance, analogous to `self` or `this` in other languages. You can optionally give it another name with `as`. 

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
    speak(self) =
        "I am a MyStruct \{ name={self.name}, value={self.value} }"

MyEnum :: impl[MyPrototype] =
    speak(self) =
        match self is
        | First:
            "I am a MyEnum of First"
        | Second(x):
            "I am a MyEnum of Second({x})"
        | Third{val}:
            "I am a MyEnum of Third \{ val={val} }"
```

### Inheritance and Visibility

Even though structs cannot be extended the usual way, they can **inherit** from other structs using the `inherit` keyword. This works similarly to **importing.** It marks members that map to members of another struct, making conversion possible. It follows the same convention for pattern matching like destructuring. *(See [Destructuring](#Destructuring).)* The struct being inherited must not be `@opaque` or else it's a compile-time error. *(See [Decorators](#Decorators) for more information on available decorators.)*

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
-- Note that this is not the actual definition for an option type `type?`. This is just a user-defined enum that uses the same pattern.
Maybe[T] :: enum =
    Some(T)
    None

Some[T] :: (x: T) = Maybe[T].Some(x)

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
Pari[A, B] :: where =
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

The `else` on a constraint provides the error message when the constraint fails. This is a developer experience feature more than a type theory feature, but it pays enormous dividends in practice. 

```
groupBy[T, K] :: where =
    T: type
    K: impl[Hashable] & impl[Comparable]
    K: typeof[fn(T): K]
        else "groupBy key function must return a Hashable, Comparable type"

groupBy[T, K] :: (arr: T#, keyFn: fn(T): K): T##K =
    result: mu T##K = [:]
    loop item in arr:
        key = keyFn(item)
        result#key = (result#key || []) ++ item
    result
```

### Manual Implementation

Generics will automatically generate code based on their parameters, but you can also implement them by hand using pattern matching. If you only want to use the manual implementations for a generic function, you can set its body to `never`. This creates a virtual function that can be overloaded later. If you use a function that is defined with `never`, it will throw a compile-time error.

```
-- Forces every type to have its own implementation
increment[T] :: (c: ref mu T): void = never

Counter :: struct = value: int

-- Specialized for Counter
increment[Counter] :: (c: ref mu Counter): void =
    c.value += 1

-- Specialized for float
increment[float] :: (c: ref mu float): void =
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

Use `import` to import something, optionally giving the import an alias with `as`. You can either import a single export like `import a.b.c` or multiple at once using destructuring rules `import a.b{c, d}`. *(See [Destructuring](#Destructuring).)* Note that there is no `.` before the `{`. This follows the same convention that destructuring with tuples does. All imports must be **explicitly** declared—no `import a.b._`. This helps prevent naming conflicts and track where things have been defined.

The most common import will likely be the `print` function, which will be defined somewhere in a standard library.

```
import std.print     -- This is just an example and not final.

print("Hello, world!")
```

Modules are named with the keyword `mod` near the top before anything is defined. This is the name you'll use when importing your module. **There can only be one `mod` declaration per file.** Multiple `mod` declarations is a syntax error. Imports are based on the include path when compiling or running a script. To import from a file by direct filepath, use `@from("path")` before `import`.

```
@from("../../somewhere.mu") import someModule{thing}

mod myModule

addThing(x) = x + thing
```

In this example, you would import `addThing` like this (assuming the file is included):

```
import myModule.addThing
```

This connects the same explicit-list convention as `inherit` and capturing — *no hidden dependencies, everything that can affect behavior is named.* The three together form a consistent rule across the language:

| Keyword   | "I am explicitly pulling in…"       |
|:----------|:------------------------------------|
| `import`  | Names from another module.          |
| `inherit` | Members from another struct.        |
| `() / =`  | Mutable variables from outer scope. |

### Base Context / Command Line Arguments

Some context variables are defined when a module starts that hold the command-line arguments passed to your program when it's called. These are all type optional string `str?`. The first argument `$0` is either the filepath to the main script when in interpreated mode or `None` when running as a compiled programm. You can use this to check if your running as a script or not.

```
if $0 is Some(x):
    print("Script name: {x}")
else:
    print("Compiled mode")

print("Hello, {$1?}!")             -- Prints first argument.
```

If your file is `hello.mu` and you run it as `mu hello.mu world`, this is what would be printed:

```
Script name: hello.mu
Hello, world!
```

### Memory Models

Mulang is multi-paradigm: different functions, structs, or modules can use different memory strategies in the same program. The model is controlled per-module via decorators. Boundary crossing between models follows FFI-like rules — automatic marshalling where possible, explicit escapes otherwise.

Modules define how memory is handled with the `@memory` decorator. *(See [Decorators](#Decorators) for more information on available decorators.)* By default, modules use a garbage collector. Some options include `Collect(GC)` (default),  `Count(ARC)` (reference counting), and `Manual`. `GC` and `ARC` represent the standard garbage collector and reference counter respectively, but others can be defined and used instead.

```
import std.mem{memory, Count, ARC}

@memory(Count(ARC))
mod moduleThatUsesReferenceCounting
```

How this is implemented is outside of the scope of this document. That will be saved for when it's time to make a standard library for Mulang. For now, Mulang will focus on only implementing the garbage collector which will work in both interpreted and compiled mode.

---

## Decorators

There have been a few examples of decorators in this document like `@opaque` and `@memory`. These are compile-time functions that communicate to the compiler directly and can alter the behavior of things. The API for defining your own decorators is not set in stone yet. More information on them will be available in the future. The syntax for adding decorators goes like this:

```
@dec
@dec(arg)
expr
```

```
@dec expr
@dec(arg) expr
```

Decorators can be stacked and will run in reverse order. *Closest decorator to the expression runs, then the next one above that, then the next one, etc.*

Built-in decorators demonstrated so far include `@unsafe`, `@inlined`, `@opaque`, `@from`, and `@memory`. More planned for the future.

```
@memory(Manual) -- Call it like a function to pass a variable.
mod myModule

@opaque         -- No function needed if there are no arguments.
Thing :: struct =
    value: int

@inlined
setBitAnd(ref mu a, ref b) =
    a = a band b
```

Some other ideas for built-in decorators include:

- `@private` — locks a symbol to only be used within its module.
- `@static` — make a variable global but only available within the scope that it was defined in.
- `@inline` — marks that a regular function should inline itself like a meta function.
- `@comptime` — run a function at compile-time, return it's value as a constant.
- `@pure` — enforces pure function programming practices: *no capturing, no context, no `ref mu`, no `out`, etc.*
- `@safe` — enforces borrow-checking at compile time for this module or function.
- `@override` — marks that a previously implemented method will be overridden.

This is a work in progress though. How these decorators are implemented and their API are subject to change.

---

## Design Philosophy

- **Readability first** — significant whitespace and opinionated formatting.
- **Patterns scale with complexity** — simple things like declaring a mutable variable (`mu`), making a function (`fn`), or wrapping a block (`(…: …)`) use short patterns, more complex things use bigger patterns.
- **Performance on demand** — start with GC; change to a lower-level memory model where necessary.
- **Explicit but ergonomic** — `!` for errors, `?` for options, same keywords used between inline and block expressions.
- **Trace and auditability** — `import`, `inherit`, and capture require variables to be listed out to know where they're coming from; no glob-like imports.
- **Unified concepts** — `/` for scope capture; `inherit` for member visibility; `::` for all top-level definitions.

Some choices in Mulang are unconventional, but ask yourself…

- *Why does array indexes need to be `name[i]`?*
- *Why do bitwise operators need to be `&`, `|`, `^`?*
- *Why does bitwise not need to be separated from logical not?*
- *Why is `--` for comments and not the decrement operator?*

Just because it's different doesn't mean it's bad. Mulang as a new language is free to explore new conventions for the sake of experimentation and progress. 

---

## Keywords

1. `and`
2. `as`
3. `await`
4. `band`
5. `bor`
6. `break`
7. `continue`
8. `defer`
9. `do`
10. `else`
11. `enum`
12. `except`
13. `fallthrough`
14. `fn`
15. `if`
16. `impl`
17. `import`
18. `in`
19. `inherit`
20. `is`
21. `loop`
22. `match`
23. `mod`
24. `mu`
25. `never`
26. `not`
27. `opt`
28. `or`
29. `out`
30. `proto`
31. `raise`
32. `ref`
33. `return`
34. `self`
35. `struct`
36. `then`
37. `try`
38. `until`
39. `where`
40. `xor`
41. `yield`

*NOTE: Built-in types, values, functions, and decorators like `int`, `True`, `Some`, `default`, `print`, `@memory` etc. are not considered keywords.*

---

*This document captures the current state of the Mulang design. The language is still evolving.*
