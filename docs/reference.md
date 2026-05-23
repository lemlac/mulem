# Mu Language Reference

*Version 0.1 (Draft)*

Mu (also called *Mulang*) is a general-purpose, multi-paradigm language with significant whitespace. It is expression-oriented and gives programmers explicit control over evaluation strategy, memory model, and error handling. Mu is planned to be both compiled and interpreted within the same language, making it suitable for systems programming, AI, and game development.

Mu targets developers who want Python-like readability, Rust-like control and safety features, and F#-style expressive pipelines — especially for domains like robotics, systems programming, AI, and games.

It aims to support both interpretation and compilation (including to shared libraries).

---

## Core Design Philosophy

- **Expression-oriented**: Almost everything is an expression and returns a value.
- **Significant whitespace** with smart inline support via `do`/`end`.
- **Modern error & option handling**: `?` and `!` propagation, `||` coalescing.
- **Flexible**: Multi-paradigm (functional, procedural, low-level).

---

## Lexical Conventions

- **Indentation**: Significant (4 spaces recommended).
- **Line endings**: Newlines (`\n`, `\r`, or `\r\n`) or semicolons `;` separate expressions.
- **Comments**:
  - Single-line: `-- comment`
  - Block: `(-- ... --)` (nesting allowed)
- **Strings**:
  - `"double quotes"` with `{interpolation}`
  - `''raw strings''`
  - `"""multi-line strings"""`

### Whitespace and Indentation

Whitespace is significant. Indentation marks where blocks begin and end. Four (4) spaces per level is recommended. Tabs and spaces may be mixed, but each expression within a block must have exactly the same indentation sequence — the same number of tabs and spaces in the same order.

### Statement Separators

Statements are separated by newlines or semicolons (`;`). The two are interchangeable. Newlines may be `\n`, `\r`, or `\r\n`.

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

A program is a sequence of expressions. Blocks are created with `:` after a construct, followed by indented content. The last expression in a block is its value.

```mu
if condition:
    do_something()
    result
```

### Expression Splitting

A single expression may span multiple lines under these conditions:
- Inside brackets: `()`, `[]`, `{}`
- Lines starting with `.` (method chaining)
- Lines starting with `|>` (pipelines)
- Lines starting with `|` followed by a pattern continue a `match` block
- Inside a multi-line string `"""..."""`
- `;` ends the expression early.

```
-- Method chaining across lines:
object.method1()
      .method2()
      .method3()

-- object.method1();  ← semicolon disables splitting
--       .method2()   ← SyntaxError
```

### Blocks

A block wraps multiple expressions into one. A `:` or `=` followed by a newline and indentation starts a block. The last expression evaluated in a block is its value.

```
block:
    expr
    expr    -- This is the block's value.
```

Use `pass` to leave a block empty.

```
block:
    pass
```

### Inline Blocks (`do` / `end`)

Significant whitespace makes it awkward to pass multi-line lambdas inline. `do`…`end` switches between block mode and inline mode. `end` always closes the nearest unclosed `do`.

```
apiFetch(do fn(result) =
    if result > 0:
        print("Success! {result}")
    else:
        print("Failure! {result}")
end)
```

`do` must be followed by a block keyword and a `:` or `=`. Nesting works freely.

```
(do if x:
    expr
else:
    expr
end)
```

### Inlining with `then`

Any block that has a *subject* can be inlined using `then` instead of `:`.

```
if x then "True" else "False"

if x:           -- Block form.
    "True"
else:
    "False"
```

---

## Operators

Mu has a clean, consistent operator set with both symbolic and word forms.

Symbols are grouped by meaning: `*`/`/` for math, `?` for options, `!` for results, `#` for arrays, `&` for tuples, `^` for pointers. Repeating an operator gives a more technical variant: `+` addition vs `++` concatenation, `*` multiplication vs `**` exponentiation, `/` division vs `//` floor division. Bitwise operators use `/\`, `\/`, and `><` rather than `&`, `|`, and `^` because those symbols have other meanings in Mu.

### Precedence (Highest to Lowest)

| Level | Category                  | Operators                |
|:------|:--------------------------|:-------------------------|
| 11    | Member access             | `.`                      |
| 10    | Postfix                   | `? ! ^ #`                |
| 9     | Unary                     | `+ - not ~`              |
| 8     | Exponent                  | `**` (right-associative) |
| 7     | Multiplicative / Shift    | `* / // % %% << >> >>>`  |
| 6     | Additive / Concat         | `+ - ++`                 |
| 5     | Bitwise                   | `/\ \/ ><`               |
| 4     | Range                     | `.. ..=`                 |
| 3     | Comparison                | `== != < > <= >=`        |
| 2     | Logical AND               | `and`                    |
| 1     | Logical OR / Pipeline     | `or \|\| \|>`            |
| 0     | Assignment / Spread       | `= := += => & ~&`        |

| Operator       | Meaning                                             | Precedence |
|:---------------|:----------------------------------------------------|:----------:|
| `lhs . rhs`    | Member access                                       |     11     |
| `lhs # rhs`    | Array index                                         |     10     |
| `lhs ?`        | Unwrap option, propagate `None` to nearest `opt`    |     10     |
| `lhs !`        | Unwrap result, propagate exception to nearest `try` |     10     |
| `lhs ^`        | Dereference typed pointer                           |     10     |
| `~ rhs`        | Inferred type conversion                            |     10     |
| `++ rhs`       | Spread array into array                             |      0     |
| `& rhs`        | Spread tuple into tuple (same type)                 |      0     |
| `~& rhs`       | Spread tuple into tuple (new product type)          |      0     |
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
| `lhs /\ rhs`   | Bitwise AND                                         |      5     |
| `lhs \/ rhs`   | Bitwise OR                                          |      5     |
| `lhs >< rhs`   | Bitwise XOR                                         |      5     |
| `lhs << rhs`   | Shift left                                          |      7     |
| `lhs >> rhs`   | Shift right                                         |      7     |
| `lhs >>> rhs`  | Unsigned shift right                                |      7     |
| `lhs ++ rhs`   | Concatenation                                       |      6     |
| `lhs \|\| rhs` | None-coalescing                                     |      1     |
| `lhs and rhs`  | Logical AND                                         |      2     |
| `lhs or rhs`   | Logical OR                                          |      1     |
| `not rhs`      | Logical NOT (or bitwise NOT for numbers)            |      9     |

Symbol operators gain assignment forms with `=` and pipeline-assignment forms with `=>`.

```
x += 1          -- x = x + 1
x /\= 0xFF      -- x = x /\ 0xFF
expr +=> x      -- pipeline: x = x + expr
```

### Prefix / Function Form

Operators may be called as functions. Prefix unary operators apply element-wise to a tuple. Binary operators chain across all arguments. Comparison operators return a tuple of bools one shorter than the input.

```
not(a, b, c)             -- (not a, not b, not c)
add(a, b, c)             -- a add b add c
(<)(a, b, c, d)           -- (a < b, b < c, c < d)
and(<)(a, b, c, d)       -- (a < b) and (b < c) and (c < d)
```

### Pipelining `|>`

When placed at the start of a line in a block, the pipelining operator `|>` takes on a special meaning. It takes the value of the previous line and inserts it into the next line as `$`. This symbol is known as the **contextual reference.** This symbol has several uses in Mulang, but for now just know that it represents the value of the previous line before `|>`. This makes it easy to chain functions one after another in sequence.

```
print("{
    fetchC(
        fetchB(
            fetchA()
        )
    )
}")
```

*Becomes...*

```
fetchA()
|> fetchB($)
|> fetchC($)
|> print("{$}")
```

This makes it easier to see what gets called in what order. It reads like a list written in plain English:

- `fetchA`
- *then* `fetchB`
- *then* `fetchC`
- *then* `print`

The first line of a block can also start with `|>`, it's context `$` is an empty tuple `()`. Even if a line starts with `|>`, it does not have to have `$` in it. Some functions also require a certain variables in the contextual reference to be defined. *(See [Function Declarations](#Function Declarations).)*

Multiple expressions seperated by semi-colons `;` on one line will also use the same context `$`. The last expression will be passed to the next pipe.

```
|> fetchA()                   -- Run fetchA,
|> print("{$}"); fetchB($)    -- Print result, then fetchB
|> print("{$}"); fetchC($)    -- Print result, then fetchC
|> print("{$}")               -- Print result,
```

A **pipeline block** is started by the symbol `|>:`. It can be at the end of a line or on its own line. `$` becomes the inputed value for the duration of the block.

```
|> fetchA()         -- Set up things.
|> fetchB($)        -- …
|> fetchC($)        -- …
|>:                 -- Set up done.
    print("{$}")    -- Now print the result.

fetchA() |> fetchB($) |> fetchC($) |>:  -- Or in one line.
    print("{$}")                        -- Then print the result.

               -- Freely mix the two formats:
fetchA() |>:       -- Start with this context.
    print("{$}")   -- Use the same `$` for these two lines.
    fetchB($)      -- Same context `$`
    |> print("{$}"); fetchC($) |>:     -- Start another context, inline it.
        print("{$}")   -- Print the final results.
```

To set a variable in the middle of a pipeline expression, use `=>` or its variants after an expression. The order of assignment is reversed — varible name goes on the right.

```
|> fetchA()
|> fetchB($)
|> fetchC($) => x   -- Do these things and put them into `x`

print("{x}")        -- Print the result.
``` 

This lets you freely extract the result of any expression in the middle of a pipeline by adding `=> name` at the end of a line.

```
-- Get all the results
|> fetchA()  => a
|> fetchB($) => b
|> fetchC($) => c

-- Print each result.
print("a = {a}, b = {b}, c = {c}")
```

The type must be inferred. This is to avoid ambiguities with `:`. You can change the mutability with the keyword `mu`. *(See [Mutability](#Mutability).)*

```
|> fetchA() => mu x |> fetchB(x) |>:  -- Create mutable variable `x`.
    x += 1                            -- Mutate `x`.
    print("{x}")                      -- Print `x`.
```

Get the final result of a pipeline expression put putting `|> $ =>` at the end. This gets the last context and puts it into a variable.

```
fetchA()         -- Start.
|> fetchB($)     -- Pass context.
|> fetchC($)     -- Pass context.
|> $ => result   -- Put context into result.

print("{result}")
```

The shorthand for `|> $ =>` is `| =>`, a pipe `|` followed by a pipeline assigment operator such as `=>`. The two symbols can be spaced apart or combined like `|=>`.

```
fetchA()         -- Start.
|> fetchB($)     -- Pass context.
|> fetchC($)     -- Pass context.
|=> result       -- Put context into result.

print("{result}")
```

This gives you a lot of flexability on how you want to express your code.

### Contextual Reference (`$`)

`$` has a special meaning in Mulang. It holds the values of the current context. Variable names in Mulang don't allow `$` inside them, but this symbol is treated like a name. You can put symbols next to it, but other words needs to be seperated with a space next to it.

```
-$    -- This is okay, 1 symbol + 1 word: `-` + `$`.
not$  -- This is one word, error since `not$` doesn't exist.
not $ -- This is okay, 2 words: `not` + `$`.
```

Variables prefixed with `$` come from the contextual reference. This lets you use the context to write expressive code. Named values use their names like `x` → `$x`, position values use numbers like `$0`, `$1`, `$2`, etc..

```
(0, x: 1)               -- Create a tuple with position member and named member `x`.
|>:                     -- Pass to a pipeline block.
    print("{$0 + $x}")  -- Prints "1".

(0, x: 1) |> print("{$0 + $x}")  -- Or in-lined.
```

This pairs with the next concept of Mulang: *operation chaining.*

### Operation chaining `[]`

**Infix operator** are operators who have both an `lhs` and `rhs`. For any non-void infix operator `op`, you can write it like `lhs op[rhs]` (an operator + an array literal). This is shorthand for writing `((lhs) op (rhs))` and has the same precedence like accessing with `.` or wrapping parentheses around the expression. This is primarily used for arrays, but has many potential use cases which can be explored.

Operation chaining has the same precedence has member accessing `.`. This allows you to treat any operator like a member access. The most useful use cases for this are for array indexing `#` and pipelining `|>`.

```
array#[0]#[1]                  -- → ( ( array # 0 ) # 1 )
fn1() |> [fn2($)] |> [fn3($)]  -- → ( ( fn1() |> fn2($) ) |> fn3($) ) → fn3( fn2( fn1() ) )
```

Like with method chaining, operation chaining also allows expression splitting. Lines that start with an operator + square brackets `[]` continue the expression from the previous line.

```
bigString = "This"
    ++[" string"]
    ++[" is"]
    ++[" split"]
    ++[" into"]
    ++[" seperate"]
    ++[" lines."]

print(bigString)   -- Prints "This string is split into seperate lines."
```

Pipelining can be particularly useful when combined with operation chaining. For example, you could reuse a repeated part of an expression or call another function in the middle of method-chaining.

```
-- Repeated value:
onePlusTwoCubed = (1+2) |> [ $ * $ * $ ]   -- Same as `( (1+2) * (1+2) * (1+2) )`
print("{onePlusTwoCubed}")                 -- Prints "27"

-- Method chaining:
object.method1() |> [ fn1($) ].method2() |> [ fn2($) ]   -- `|> []` has the same order of operations as `.`
-- Becomes…
fn2( fn1( object.method1() ).method2() )
```

Although arrays and tuples are treated differently, their syntaxes are analogous to each other—one uses square brackets `[array]` and the other uses parentheses `(tuple)` or curly braces `{tuple}`. We can use this to our advantage to give operation chaining another feature: **tuple expressions**. If the array literal after an operation has more than one indexes (e.g. `[expr, expr]`) or one or more named indexes (e.g. `[name: expr]`), then it will return a **tuple.** *For more information on tuples, see [Tuples](#Tuples).*

```
x = (13, 5) |>[ $0 // $1, $0 %% $1 ]  -- Get 12 divmod 5 → (2, 3)
print("12/5 = {x.0} and {x.1}/5")     -- Prints "12/5 = 2 and 3/5"

-- Apply `x ==` to each value, returns a tuple of bools.
-- Prefix `or` takes a tuple of bools and returns `True` if any member is `True`.
if or x ==[0, 2, 5]:
    print("x is 0 or 2 or 5")

-- That's the same as...
if x == 0 or x == 2 or x == 5:
    print("x is 0 or 2 or 5")
```

You can create an named tuple instead of a position tuple too. Members can have names by adding the name and a colon `:` in front of each index. It must be a valid variable name. If you define a member with `name:`, you can access it in the next expression with `$name`. 

Combined with `&`, you can use this to set the values of an object. `lhs &[ rhs ]` is equvialent to spreading each side into a new tuple `( &lhs, &rhs )`. The resulting type is `lhs`, and `rhs` must be compatible, i.e. member names found in `lhs`. 

```
user = User.create()
    &[name: "John Doe"]
    &[dob: "1970-01-01"]

-- Or all in one go...

user = User.create()&[
    name: "John Doe",
    dob: "1970-01-01",
]
```

If you want to create a new type, you can use `~&[]` instead. `lhs ~&[ rhs ]` is equivalent to saying `(&lhs, &rhs)` — in other words: *create a new tuple and spread the members of each side in order.* The type is `typeof[lhs] & typeof[rhs]`, the **product type** of both sides.

```
tuple = ()    -- Create empty tuple.
    ~&[a: 1]  -- Set ().a to 1.
    ~&[b: 2]  -- Set (a: 1).b to 2.
    ~&[c: 3]  -- Set (a: 1, b: 2).c to 3.
              -- `tuple` is (a: 1, b: 2, c: 3).
```

The assignment variant of `&[]` is `&=` and `&=>`. This takes the members of the inputed value and assigns the same members to the outputed variable. Use `{}` or `()` to create a tuple of named members.

```
user &= {
    name: "John Doe",
    dob: "1970-01-01",
}

-- Alternative format:
(name: "John Doe", dob: "1970-01-01") &=> user

-- Is the same as...
user.name = "John Doe"
dob.dob = "1970-01-01"

-- Append a new member to a tuple.
tuple ~&= (d: 4)

-- `tuple` is (a: 1, b: 2, c: 3, d: 4)
```

Another use for variables in operation chaining is to create temporary variables and then collect them into a result.

```
-- Split `1` into 4 variables and collect.
y = 1 |> [
    a: $,
    b: $ + 1,
    c: $ + 2,
    d: $ + 3,
] |> [ $a * $b * $c + $d ]
```

`y` simplifies like this:

1. `x` *pipe to* `[ a: $, b: $ + 1, c: $ + 2, d: $ + 3 ]` *pipe to* `[ $a * $b * $c + $d ]`
2. `( a: x, b: x + 1, c: x + 2, d: x + 3 )` *pipe to* `[ $a * $b * $c + $d ]`
3. `( x * (x + 1) * (x + 2) + (x + 3) )`
4. `x` *is* `1` *so…*
5. `( 1 * (1 + 1) * (1 + 2) + (1 + 3) )`
6. `( 1 * 2 * 3 + 4 )`
7. `( 10 )`

Tuples are transparent by default, and a tuple with only one position component and no named components is equal to just its positional component. *For an explanation why, see [Tuples](#Tuples).* Semantically, this makes sese. If you have a parenthetical expressions, you essentially have a tuple of one. Some languages make a distinction between tuples and parenthetical expressions with syntax like `(10,)`, but this is not necessary in Mulang. 

- `(10) == (10).0`
- `(10).0 == 10`
- _Therefore:_ `(10) == 10`

**Mu the programming language is a novel and experimental one.** It has no obligation to follow normal conventions. Most other languages use `[]` by itself for indexing an array. While this is still possible in Mulang, operation chaining gives the coder a lot more options and ability to write expressive code. This is a break from the traditional style of programming, but it opens the door for more pragmatic and ergonomic code. This makes it instantly clear that when you see `#`, it's an array, just like `?` for options and `!` for results, staying consistent with itself rather than with other languages.

The transition from array indexing with `[]` in other languages to array indexing with `#[]` in Mulang is simple: add `#` before each square bracket. In most cases, the square brackets aren't even needed, making the `#` pattern more compacted than the `[]` one.

- `array[0][1]` → `array#[0]#[1]` → `array#0#1`

This frees `name[]` to take on a new meaning in Mulang. *For more on that, see [Meta Functions](#Meta-Functions).*

Mulang has a fallback for programmers who are used to arrays being written like `array[0][1]`. *For more information on array indexes, see [Arrays](#Arrays).*

## Basic Bindings

There are two types of bindings: basic `=` and meta `::`. See [Meta Bindings](#Meta-Bindings-) below for details about `::`.

### Variable Declarations

Variables are declared with just the equals sign (`=`). Type is inferred, but can be declared with a colon (`:`). You can also use the `: =` or `:=` operator instead to declare and infer the type at the same time. This is useful for shadowing mutable variables. *(See [Mutability](#Mutability).)* For now, just know that anytime you see `:` before `=`, *it always declares a new variable,* and if you see just `=`, *it's either declaring or mutating a variable.*

```
a = 0       -- Implicit declaration
b: int = 1  -- Explicit declaration with type.
c := 2      -- Explicit declaration but infer its type.
d: = 4      -- Space between the `:` and `=` doesn't matter.
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
a = 1         --\
a = 2         --- Each `a` is a new variable on the stack.
a = 3         --/
a = "hello"   -- The type of a shadowed variable doesn't have to match.
```

You can also shadow a variable using its previous value. This works for both regular assignment `=` and pipeline assignment `=>`.

```
i = 0
i = i + 1     -- Sets new `i` based on old `i`.
i += 1        --\ 
i + 1 => i    ---\
1 +=> i       ---- Does the same thing.
```

Using the single equal-sign is a void statement. If you use it within an expression and not on its own, it's a syntax error. This helps prevent the common bug of using `=` when you meant `==`. For inline binding, use `=>` instead. *(See [Pipelining(#Pipelining-).)*

```
(-- Error:
if x = 0:
    print("x is 0")
--)
-- Do this instead:
if x == 0:
    print("x is 0")
```

Reversing the normal order of assignment for `=>` operators helps with programmers who might confuse `=>` for `>=` (greater than or equals).

```
if x => 0:      -- Error: did you mean `x >= 0`?
    print("x is greater than or equal to 0")
```

Contextual variables are declared with `$` at the start of their name. This is used for functions with contextual parameters, letting you to share variables between functions without passing them directly. *(See [Contextual Parameters](#contextual-parameters).)*

```
printX() =      -- Function that requires `$x` to be defined.
    $x: int
    print("{x}")

$x = 0
printX()        -- Prints "0"
```

#### Mutability (`mu`)

Mutable variables are marked with the keyword `mu`, the main star of the language. Declare a mutable variable with `mu type`. Setting it will change the value instead of shadowing it. The type of value when mutating it must match its original type.

```
mut: mu int = 0
mut = 1          -- `mut` is mutated
```

You can also infer the type with `mu _ = _`:

```
mu mut = 0   -- Shorthand for `mut: mu int = 0`.
mut = 1
```

Or you can declare the type and set it later:

```
mut: mu int
action()
mut = 1
```

Not setting a mutable variable implies `= unset[T]` after it which means it cannot be used until it's been set. The exception is passing unset variables to the `out` parameters of functions. *(See [Function Declarations](#Function-Declarations).)*

```
mut: mu int = unset[mu int]
-- `mut` cannot be used here.
(--
doSomething(mut)   -- This is an error.
--)

mut = 1
-- `mut` can be used now.
doSomething(mut)   -- This is okay.
```

`mu` variables cannot be shadowed by `=` or `=>`. They can only be mutated. Assigning to the variable for the rest of the scope and any sub-scopes will mutate the variable—except for functions which always set a new variable unless its been explicitly captured. *(See [Capturing](#Capturing).)*

```
mu mut = 0
block:
    mut = 1
    print("{mut}") -- Prints "1", same outer `mut`.

print("{mut}")     -- Prints "1", `mut` was mutated.

cantSetMut() =
    mut = 2        -- Declares a new variable `mut` in this function.

cantSetMut()
print("{mut}")     -- Prints "1" again, cantSetMut didn't change it.
```

If you wish to shadow a mutable variable, you can redeclare the variable with `: T =` or `:=`.

```
mu mut = 0
block:
    mut := 1        -- Declare new `mut` in this block.
    print("{mut}")  -- Prints "1", inner `mut` was shadowed.
                    -- Exit block
print("{mut}")      -- Prints "0", outer `mut` was not mutated.
```

__A brief word on the choice of the keyword `mu`…__

Just like Go has its `go` keyword, Mulang has `mu`. Go's `go` has a clear and obvious connection with *goroutines*, and Mu's `mu` has a clear and obvious connection with *mutable.* This was chosen since mutability is a common practice in programming much like functions are—which is why functions also get their own two letter keyword `fn`. *(See [Function Declarations](#Function-Declarations).)* The brevity of it is its strength. It beats other keywords such `mut` *(mutt? like a mixed dog breed? meaning too ambiguous),* `val` *(too common of a variable name),* `var` *(doesn't imply mutability).* The general rule of thumb in Mulang is that *patterns scale with complexity*—simple things like declaring a variable or function use short patterns, more complex things use bigger patterns. That's why the two letter keyword `mu` beats any three letter keyword. Making a *mutable* variable in *Mu* is easy like making a *goroutine* in *Go.* 

This is grounded in the language's own internal consistency. If `fn` for functions is acceptable at two characters, `mu` for mutability follows the same pattern naturally. The pairing is actually elegant:

- `fn` — the thing that does something
- `mu` — the thing that changes

You might think the choice of the keyword `mu`—which is the same name that the language itself is—will be a potential source of confusion. However, Go doesn't have this problem with its `go` keyword. When searching or discussing about *Mu*, it's sometimes preferable to write it *Mulang* for clarity, just like other languages can have "lang" at the end of their names like *Go* → *Golang* or *D* → *Dlang*.

#### References (`ref`/`ref mu`)

A reference type points to the same spot in memory that another variable has. It's like a lightweight version of a pointer. *(See [Pointers](#Pointers).)* The right hand side has be something stored in memory, i.e. not a constant. References are immutable by default unless declared with `ref mu`. The syntax follows the same pattern that `mu` does:

* `x: ref T = y` = Immutable reference with explicit type.
* `ref x = y` = Immutable reference with inferred type.
* `x: ref mu T = y` = Mutable reference with explicit type.
* `ref mu x = y` = Mutable reference with inferred type. 

```
mu x = 0
ref mu xRef = x
xRef = 1
print("x is {x}")   -- Prints "x is 1".
```

#### Destructuring

Tuples can be split into separate variables. Use parentheses (`()`) for positional tuples and curly braces (`{}`) for named tuples. If a tuple is mixed, split each on the left side with `&` like `() & {}` or `{} & ()`. This is to avoid confliction between the different uses of `:`, type notation on the left of the equal sign and key-value pairing on the right of the equal sign. Use `as` in a named tuple `{}` and the left hand side to create an alias for named component with type notation going after the alias name. See [Tuples](#Tuples) for more information.

```
(a, b) = (0, 1)               -- Basic position destructuring.
(a: int, b: int) = (0, 1)     -- Type notation.
{x} = {x: 2}                  -- Named destructuring.
{x: int} = {x: 2}             -- Named destructuring with type notation.
(a, b) & {x} = (0, 1, x: 2)   -- This tuple has both positional and named components.
(a: int, b: int) & {x: int} = (0, 1, x: 2)
{x as y} = {x: 2}             -- Set the `x` component as `y` in this scope.
{x as y: int} = {x: 2}        -- Type notation goes after `as`.
```

An alternative method for destructoring a mixed tuple is to use `{}` with integers for the positional components. They must be aliased with `as`.

```
{0 as a, 1 as b} = (3, 4) -- Use alias for positional components.
print("a={a} b={b}")      -- Prints "a=3 b=4".
```

All components of a tuple should be referenced on the left. Use `_` to explicitly skip components. Also put `_` at the end of a named tuple to skip other keys.

```
(_, b, _) & {x, _} = (0, 1, 2, x: 3, y: 4)
-- Or...
{1 as b, x, _} = (0, 1, 2, x: 3, y: 4)
```

When destructuring a type that isn't anonymous, the type can optionally be put after the parentheses/braces, otherwise it's automatically inferred. 

```
Thing :: {x: int, y: int}
thing = Thing(x: 1, y: 2)
{x, y}: Thing = thing         -- Split thing into its components.
{x, y} = thing                -- Or infer the type.

Args :: (int, int) & {a: int}    -- Type has positional and named components.
args = Args(0, 1, a: 3)          -- Same order as the type, but named component can go in any position.
(a, b) & {a as c}: Args = args   -- Type goes after the last tuple.
(a, b) & {a as c} = args         -- Or infer the type.
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

#### Mutable / Reference parameters

Function parameters can be declared like variables. Likewise, you can modify their mutability and reference-ness the same way.

```
increment(x: ref mu int) =
    x += 1

mu y = 0
increment(y)
```

What each modifier means changes the functionality:

| Modifiers | What It Does     | Mutability                    |
|:----------|:-----------------|:------------------------------|
| *nothing* | Copies value     | Immutable *(in the function)* |
| `mu`      | Copies value     | Mutable *(in the function)*   |
| `ref`     | Passes reference | Immutable                     |
| `ref mu`  | Passes reference | Mutable                       |

Another type of parameter is `out`. This is like `ref mu` but is treated like `unset[_]` at the start of the function. Use it to set a variable that hasn't been set yet. The parameter must not be `unset[_]` in any branch within the function. This means either setting it within the function or passing it to another function with an `out` parameter. This ensures that the variable is set after the function has been called. 

```
setInt(out i): void =
    i = 3

x: mu int
setInt(x)
print("{x}")    -- Prints "3"
```

This works for mutable variables, but what if you wanted to make an immutable variable using `out`? You can't pipe the return value with `=>` because it's in the paraemter, not the return of the function. `setInt() => n` would be an error. Not only use the `out` parameter not set, but it's not returning anything either. For this, we have a special rule: when `=>` is prefixed in a parameter, it has the same effect as using `=>` on the return value of a function. This way we can pipeline the `out` parameter of a function. *(See [Pipelining(#Pipelining-).)*

```
setInt(=> n)    -- Declare a new variable `n` that gets set by `setInt`.
print("{n}")    -- Prints "3"
```

#### Capturing

Immutable variables can be captured without an issue. If you try to set it within a function, it will get shadowed within the scope of the function. This also includes other functions which are also immutable by default.

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

To capture a mutable variable, you can write `@capture` at the top of the function body and capture multiple variables at once. This helps make it easy to see which functions can mutate other variables and which don't. This follows the same practice that `import` and `inherit` where all words in a given context are listed out clearly so that there are no accident name collisions or hidden gotchas.

```
mu count = 0
mu squared = 1
mu cubed = 1
   
addCount() =
    @capture(count, squared, cubed)    -- Capture 3 variables at once.
    count += 1
    squared = count * count
    cubed = squared * count

addCount()
addCount()
addCount()
print("{count}, {squared}, {cubed}") -- Prints "3, 9, 27"
```

This highlights the flexibility of the language. It doesn't need a dedicated keyword like `capture`. Compiler behavior can be dictated using decorators, changing the language itself. *(See [Decorators](#Decorators).)* 

#### Lambda Functions

You can define a function within an expression with the keyword `fn` in the pattern `fn(_) = _`. This is useful for passing functions to other functions. If the lambda function has multiple lines, it must be wrapped in `do`…`end`.

```
map(array, func) = [++for x in array then func(x)]
array0 = [1, 2, 3, 4]
array1 = map(array0, fn(x) = x + 1)
array2 = map(array0, do fn(x) =
    if x < 2:
        x - 1
    else:
        x + 2
end)
```

A name is optional. Adding a name creates an immutable reference of the function itself.

```
doThing(do fn callback(val) =
    if val > 0:
        callback(val - 1)
    else:
        print("done")
end)
```

The same keyword for creating functions is also used for function type notation. If this seems confusing, just remember where the context is: if it's being used like a type, it means a *function pointer type*; if it's being used like a value, it's a *lambda function*.

```
action: mu fn(int, int): int    -- Declaring a variable with a function type.
action = fn(a, b) = a + b       -- Passing to the variable with a function value.
```

Note that if you try to declare a function with the name `fn`, it will throw an error. This prevents potential gotchas and silent errors. The exception is if it's the last value in a block, then the return value of that block is a function.

```
(-- This is an error because you are trying to declare a function with the name `fn`:
fn(a, b) =
    a + b
fn(1, 2)
--)

-- This is not an error; it's a function that returns another function:
curryAdd(a: int): fn(int): fn(int): int =
    fn(b) =
        fn(c) =
            a + b + c

curryAdd(1)(2)(3)
```

Capturing also works inside lambda functions just like with named functions.

```
mu count = 0
forEach([1, 2, 3, 4], do fn(x) =
    @capture(count)
    count += x
end)
```

#### Named Parameters

Functions declared with a named tuple `{}` require each parameter to be named in order to call them. Named tuples can be destructured so that their members become variables in the scope. Call the function with parentheses like before but with the name of each parameter inside.

```
add{ a: int, b: int }: int =
    a + b

add(b: 1, a: 2)     -- Order doesn't matter.
```

You can also have both positional and named parameters. Split the positional parts into `()` and named parts into `{}` and place a `&` between them. Parentheses mark a **positional tuple**, and curly braces mark a **named tuple**. *(See [Tuples](#Tuples).)* This works the same way that **destructuring** does. *(See [Destructuring](#Destructuring).)* Named parameters don't take up any position, so they can be placed anywhere in the parameters.

```
add(x: int) & { a: int, b: int }: int =
    x + a + b

-- Order doesn't matter, `x` is the same for all of these:
add(1, a: 2, b: 3)
add(a: 2, 1, b: 3)
add(a: 2, b: 3, 1)
```

Alterantively, you can use `{}` with all the positional parameters aliased using `as` like with destructuring.

```
-- Give the first parameter the name `x`.
add{ 0 as x: int, a: int, b: int }: int =
    x + a + b

-- Order doesn't matter, `x` is the same for all of these:
add(1, a: 2, b: 3)
add(a: 2, 1, b: 3)
add(a: 2, b: 3, 1)
```

Positional parameters can be set like named paramaeters in a function by putting its number position before the colon `:` when calling it.

```
isGreaterThan(a, b) = a > b

print("{ isGreaterThan( 0: 10, 1: 5 ) }")  -- True:  a = 10, b = 5, = 10 > 5 
print("{ isGreaterThan( 1: 10, 0: 5 ) }")  -- False: a = 5, b = 10, = 5 > 10
```


Named parameters can be defined in their own object and then passed in with the `&` operator also. The named parameters in the function can also be collected into a single variable using `as`. An empty positional parameter `()` needs to be placed before the typed name parameters even if the function doesn't have any positional parameters. Objects are spread with `&object` in the argument of the function when called.

```
Settings :: { enabled: bool, key: str }
doThing() & Settings as settings =
    if settings.enabled:
        Some(callApi(key))
    else:
        None

settings = Settings(key: "foo", enabled: True)

doThing(&settings)                  -- Spreads `settings` into the arguments.
doThing(key: "foo", enabled: True)  -- Or passed like a named parameter.
```

See [Tuples](#Tuples) for more details on the `&` operator.

#### Contextual Parameters

Some functions require to be in a certain context. Contextual parameters are declared with `$x: type`. These can go anywhere inside scope of the function. It's recommended to put them on the top where it's easy to see. This lets you pass shared variables between functions.

```
addContext() =
    $a: int     --- Require $a and $b to be defined.
    $b: int     --/
    $a + $b           -- Add result.

$a = 1    --- Set $a and $b in this context.
$b = 2    --/
result = addContext()  -- Grabs $a and $b from the context.

print("{result}")   -- Prints "3"
```

This can be combined with pipelining to modify a context and pass it to the next function.

```
incrementX(n) =
    $x: mu float    -- Require mutable $x to be defined.
    $x += n         -- Increment $x.
    $               -- Pass the context to the next function.

multiplyX(n) =
    $x: mu float    -- Require mutable $x to be defined.
    $x *= n         -- Multiply $x.
    $               -- Pass the context to the next function.

mu $x = 0.0         -- Define $x as a variable.

incrementX(1.0)     -- +1 -> 1.0     -- Copies current context.
|> incrementX(1.0)  -- +1 -> 2.0     -- Returns copy to next pipe.
|> incrementX(1.0)  -- +1 -> 3.0     -- Returns copy to next pipe.
|> incrementX(2.0)  -- +2 -> 5.0     -- Returns copy to next pipe.
|> multiplyX(2.0)   -- *2 -> 10.0    -- Returns copy to next pipe.
|> incrementX(4.0)  -- +4 -> 14.0    -- Returns copy to next pipe.
|> multiplyX(0.5)   -- /2 -> 7.0     -- Returns copy to next pipe.
|>:                                  -- Get the resulting context in this block.
    print("{$x}")                    -- Prints "{7.0}"
```

You can set the current context with `&=> $`. This will spread the result into the current context.

```
mu $x = 0.0         -- Define $x as a variable.

incrementX(1.0)     -- +1 -> 1.0     -- Copies current context.
|> incrementX(1.0)  -- +1 -> 2.0     -- Returns copy to next pipe.
|> incrementX(1.0)  -- +1 -> 3.0     -- Returns copy to next pipe.
|> incrementX(2.0)  -- +2 -> 5.0     -- Returns copy to next pipe.
|> multiplyX(2.0)   -- *2 -> 10.0    -- Returns copy to next pipe.
|> incrementX(4.0)  -- +4 -> 14.0    -- Returns copy to next pipe.
|> multiplyX(0.5)   -- /2 -> 7.0     -- Returns copy to next pipe.
|> $ &=> $          -- Put the resulting context into this scope's context.

print("{$x}")                    -- Prints "{7.0}"
```

Optional contextual parameters can be declared with `T?`. This will wrap the parameter in an option type if it exist. 

```
isThereX() =
    $x: int?
    match $x is
    | Some(_): print("There is an $x!")
    | None:    print("No $x...")

isThereX()    -- Prints "No $x..."
$x = 0
isThereX()    -- Prints "There is an $x!"
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

If the result type cannot be inferred, you can call a type like a function around it. The `~` before the argument distinguishes it from instantiation which also uses a function call. This only works if both the input type and output type are compatible.

```
x = 1
y = float(~x)
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

The sign is considered an operator and not a part of the constant itself. This gets automatically calculated in constant expressions at compile-time so that it seems like it's a part of the constant.

- `- 1` → `minus` + `1` → `signflip(1)` → `-1`

The exception to this rule is in scientific notation: the sign after `e` is a part of the number itself. There must not be a space between `e` and the sign `-`/`+`. The sign is optional for positive exponent values.

- Examples: `2e-100`, `5.0e+76`, `6.8E-128`, `10e3`

You can place underscores `_` anywhere in a number to break it up into segments. This doesn't change the value.

- Examples: `1_234`, `1_000_000`, `0b1111_0000`, `0xab_cd_ef`

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

Strings are marked with quotation marks (`"…"`) *(also called double quotes)* and can be formatted with curly braces (`{expr}`) in the string. Use a backslash to write a literal opening curly brace (`\{`). Note that string insertion and named tuples both use curly braces. This shouldn't be an issue though since they're used in different contexts. Expressions are implicitly converted to strings, so using `str()` or `~` isn't necessary. This string is only allowed on a single-line, but literal-line characters can be inserted with `\n`.

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
block:
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

To make a multi-line raw string, add an at sign `@` before and after triple quotation marks `"""`. The number of `@`s must match to close the raw string. This symbol is also used for decorators such as `@capture`, so the two connect giving `@` the general syntactical meaning of a compile-time signal.

|  Opening | Closing  |
|---------:|:---------|
|   `@"""` | `"""@`   |
|  `@@"""` | `"""@@`  |
| `@@@"""` | `"""@@@` |

*etc...*

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
block:
    block:
        block:
            block:
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

__Quick Glance on Strings:__

| Form        | Purpose                                                       |
|:------------|:--------------------------------------------------------------|
| `"…"`       | Regular string with interpolation                             |
| `"""…"""`   | Multiline with interpolation, trim by closing position        |
| `''…''`     | Inline raw, no interpolation                                  |
| `@"""…"""@` | Multiline raw, indentation suspended, `@` escalates if needed |

#### Arrays

Array types are declared with the hash symbol (`#`). This was chosen because the `#` is commonly used for numbers in English. For example, `#1` is read `number 1`. A number after the `#` makes it a fixed length array `type#N`. Arrays are statically sized when written `type#N`; `type#` is the dynamic form. Items are separated with commas (`,`). The hash symbol is also used for accessing an array, giving a clear visible mirror between the two. 

```
list: int#4 = [1, 2, 3, 4]
print("length of list: {len(list)}")
compressedList = [list#0 + list#1, list#2 + list#3]
doubleArray: int#2#2 = [[1, 2], [3, 4]]
```

This builds on the visible symmetry between type notation and their value expressions:

| Type | Notation | Expression |
|:---------|:------:|:--------:|
| **Results**  | `T!`  | `x!`  |
| **Options**  | `T?`  | `x?`  |
| **Pointers** | `T^`  | `x^`  |
| **Arrays**   | `T#N` | `x#n` |

Other languages use `[]` for indexing, but that has another meaning in Mulang. Instead, you can use operation chaining to do the same thing like `[]` in other languages. *(See [Operation Chaining](#Operation-Chaining).)*

```
item = doubleArray#[1]#[0]  -- The 2nd row, 1st column
print("{item}")             -- Prints "3"
```

Habits can be hard to break. Many programmers have internalized the `array[i]` format to always mean "array index i". To help with that, Mu allows the familiar `array[i]` format to be the same as `array#[i]`, but special care is taken into consideration to make sure it doesn't clash with [meta functions](#Meta-Functions). This rules out any tokens that looks like `name[]` or `name[x, y]` since array indexes normally only take one argument in other languages.

- `name[x]` — could be array indexing, check the type to resolve
- `name[]` — always a meta function
- `name[x, y]` — always a meta function

The fallback exists precisely for the one case where a programmer migrating from another language would instinctively write `array[i]`. If Mulang thinks something might be an old-fashioned array index access like that, it will check the context to determine what the type of that variable is to resolve it. If it's an array in this context, it will automatically treat it the same as `array#[i]` without any fuss. Tooling can be made to warn users and make it easier to transisition from `array[i]` to `array#[i]` / `array#i`. This lets Mulang have the familiar syntax while also having the power and potential that comes with other features like *operation chaining* and *meta functions.*

You may want to do an operation with an array literal on the right hand side of an operation. In that case, you can either surround the array literal in parentheses `()` store it in a variable first.

```
print("{ [1, 2, 3, 4] ==  [1, 2, 3, 4]  }")  -- Prints "(False, False, False, False)" because of operation chaining.
print("{ [1, 2, 3, 4] == ([1, 2, 3, 4]) }")  -- Prints "True" because the two arrays are equal.
a = [1, 2, 3, 4]                             -- Store array into variable.
print("{ [1, 2, 3, 4] == a }")               -- Prints "True"
```

If the left hand side of `++` isn't an array, it will automatically create one. A non-array on the right-hand side will push it to the end of the resulting array. In this way, you can abandon square brackets notation for array literals entirely and just use `++`.

```
a = 1 ++ 2 ++ 3 ++ 4  -- == [1, 2, 3, 4]
b = 0 ++ a ++ 5       -- == [0, 1, 2, 3, 4, 5]
c = b ++ 6 ++ 7 ++ 8  -- == [0, 1, 2, 3, 4, 5, 6, 7, 8]
```

Another use for `++` is to spread an array into another array. This was chosen because it's also used for concatenation, giving the two operations an obvious syntactical connection. 

```
a = [1, 2, 3]
b = [0, ++a, 4]    -- == [0, 1, 2, 3, 4]
c = a ++ b         -- == [1, 2, 3, 0, 1, 2, 3, 4]
```

This means that you can either spread or concatenate just by putting the right-hand side inside or outside of the array literal.

```
a = [ 4, 5 ]
b = [ 1, 2, 3 ] ++a   -- == [1, 2, 3, 4, 5]
c = [ 1, 2, 3, ++a ]  -- == [1, 2, 3, 4, 5]
b == c                -- True.
```

Spread operators `++` and `&` have the lowest precedence in the order of operations. This is so that you can write any expression to the right of it without needing to surrounding it in brackets. Spread operators only work at the start of positional expressions: `[]` for `++` ; `()` or `{}` for `&`.

```
d = [++ 1 ++ 2 ++ 3, 4, 5] -- Becomes [++ [1, 2, 3], 4, 5] first, then, spread [1, 2, 3, 4, 5]
b == c == d                -- True.
```

Some languages use `...`, but this visually conflicts with the range `..` operator. `++` conveys the meaning better without any issues. *What if you wanted to spread a range into an array?* 

```
tenDigits = [...0..10 ]  -- Huh? . + range + 0 + range + 10? 
tenDigits = [ ++0..10 ]  -- Oh! Spread + 0 + range + 10!
```

The second makes it clear that it's two operations: spread (`++`) and range (`..`). 

#### Dictionaries

Dictionaries are a subtype of arrays. Instead of numbers, each item is given a **key.** A dictionary's type is the type of the value `V` and the type of the key `K` join with a hash `#` in between: `V#K`. This makes it semantically clear that they are a subtype of arrays. Dictionaries also use the same operator to access items. The type passed to the `#` operator must match the key type. Each key is marked with `[]:` in the array.

```
dict: float#float = [
    [1.0]: 10.0,
    [1.5]: 15.0,
    [2.0]: 20.0,
]
print("{ dict#[1.5] }")   -- Prints "15.0"
```

If the key type is `str` and a key is a valid variable name, then the square brackets before `:` can be omitted. You can access it like a member with `.`.

```
dict: int#str = [
    a: 1,
    b: 2,
    c: 3,
    ["invalid name"]: 127,
]
print("{ dict#["b"] }")   -- Prints "2"
print("{ dict.b }")       -- Prints "2"
```

How dictionaries are implemented is yet to be determined. This document only focuses on their syntax in the language. 

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

You can use `||` or `orelse` to fallback to another pointer if you get a `Null`.

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

Sometimes, it's necessary to dig deep into the unsafe territory. Mulang normally prevents you from doing this unless you put the code in an `unsafe:` block. The `^` is the symbol associated with pointers, analogues to `?` for options, `!` for results, and `#` for arrays. It can be used in type notation, but it's also the operator to dereference a pointer. Thy type must be known at compile-time. Dereferencing an opaque pointer `ptr` is a compile-time error. In the type notation, `T^` prevents the pointer from mutating its memory or `T^mu` allows mutation with `^ =` (dereference + assignment). 

```
unsafe:                      -- Allow pointer manipulation here.
    mu x = 0                 -- `ptr` type takes a reference and creates a generic pointer.
    xPtr: int^mu = ~ptr(x)   -- Convert `ptr` to `int^mu`, type is known.
    xPtr^ = 1                -- Mutate the memory.
    print("{xPtr^}")         -- Prints "1".
    print("{x}")             -- Prints "1".
```

Pointer types have 2 kinds of mutability: one for the reference, and one for the pointer variable itself. Here is a table of each kind and what it means.

| Type | Can be set with `=` | Can be set with `^=` |
|:----------|:-----|:-----|
|    `T^`   |  No. | No.  |
|    `T^mu` |  No. | Yes. |
| `mu T^`   | Yes. | No.  |
| `mu T^mu` | Yes. | Yes. |

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

#### `pass`

The keyword `pass` can be put into any block to leave it empty. Not indenting after a block is considered a syntax error, so this is required in certain circumstances. However, this might result in a compile-time error when the block is expected to return a value. A function with `pass` in its body infers a return type of `void`. 

```
block:
    pass
```

#### `do` / `end`

Wraps a block inside an expression. Switches from inline mode to block mode.

```
(do block:
    body
end)
```

The most common use case for this is for callback functions. *(See [Lambda Functions](#Lambda-Functions).)*

```
apiFetch(do fn(result) =
    print("{result}")
end)
```

#### `block`

Runs a block of code that runs once, creating a new scope. The last expression evaluated is its value.

```
x = 1             -- Declare outer x.
block:            -- Create a new scope.
    x = 2         -- Declare inner x.
    print("{x}")  -- Prints "2", inner x.
                  -- Exit the scope.
print("{x}")      -- Prints "1", outer x.
```

You can optionally give it a label. This will let you break of it early. The label must be a valid variable name. 

```
block label:
    -- code
    break label
    -- never runs
```

You can put `block label` before any other block type to give it a label.

```
block branchLabel if a:
    print("Condition pass.")
    if b:
        print("Break check.")
        break branchLabel
    print("a and not b but without `and`, `not`, or `else`")
```

#### `if` / `else`

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

Repeats a block of code unconditionally until `break` is called.

```
loop:
    print("I'm looping!")
    break

-- Infinite loop:
loop:
    print("I'm looping!")
    -- no break
```

Add a condition to break when the condition is `True`. This works like a `while` loop in other language.

```
loop cond:
    body
```

The `else` after a `loop cond` block will run if the loop never ran even once. This is analogous to `if` / `else`. 

```
x = False
loop x:
    pass
else:
    print("The loop failed")
```

Some programming languages let you do something like `while (value = getValue()) {body}`. Mulang doesn't allow this because `=` is a void statement. Instead, you can use `=>` to achieve the same thing but more explicitly.

```
loop (getValue() => value) != None:
    print("value = {value}")
```

`loop` has several varients to change what condition will break. This makes it clear that whenever you see `loop`, then you have a loop of some kind. A programmer can see the word `loop` and think *"Ah, there's a loop here."* 

#### `loop` / `in`

Iterates through an array or iterator.

```
loop var in expr:
    body
```

If in-lined, it returns an iterator `iter[T]` which is lazily executed. It won't activate until it's been collected into an array with `++` or passed into another `in` loop.

```
iterator = loop x in list then x * 2
list = [++iterator]
```

The items can also be destructured. *(See [Destructuring](#Destructuring).)*

```
listOfTuples = [(1, '2', "three"), (4, '5', "six")]

loop (x, y, z) in listOfTuples:
    print("{x}, {y}, {z}")
```

Destructuring also works.

```
loop {x, y}: Thing in getThings():
    print("Thing\{x={x}, y={y}}")
```

Any asynchornous component can be awaited for using `await`. It will automatically wait for the item before continuing. It must be in an asynchronous function. *(See [`await`](#await).)*

```
loop (x, await y) in asyncIter():
    print("{x} and {y}")
```

#### `loop` / `until`

Postpone the conditional check until the end of the loop. Add `until` after a `loop` block to create a condition that will break the loop. Unlike `loop cond`, this will break when the condition is `True`, analogous to `if cond then break` inside the body. This would be a `do { body } while (!cond);` loop in other languages like C. The reason it doesn't use `while` is because that would get confused for the start of a block, and `do` already has a different meaning in Mulang, so `loop` and `until` were chosen instead. This makes it clear that `loop` and `until` are semantically connected since `until` can only come after `loop`.

```
loop:
    body
    if cond:
        break

-- Transforms into:

loop:
    body
until cond
```

The loop will run at least once before checking the condition.

`loop`/`until` is arguably a better choice than `do`/`while` because it's more descriptive of what it's doing. *"Loop until if this is true."* You can visually see that the condition is true after the `loop`/`until` block is finished, whereas you would need to invert the condition in your head for a `do`/`while` loop. It acts like a blockade to wait until a certain condition is true.

```
mu i = 0
loop:
    i += 1
until i >= 10
-- i is >=10 at this point
```

#### `break` / `continue`

Controls the iteration of any loop type mentioned. `break` exits out of the loop, and `continue` skips to the next iteration. This is only allowed in block-level loops. Inlined `for` loops are not allowed to skip iterations. This keeps the mapping between iterators 1-to-1.

After `break` or `continue`, you can give it a label to break. It must be the name of a `block` in the same scope or higher. 

```
block label loop:
    loop:
        print("Double loop! How do I break?")
        break label

print("I'm free!")
```

```
block outer loop x in 0..100:       -- Label this block `outer`.
    block inner loop y in 0..100:   -- Label this block `inner`.
        prod = x * y
        print("{x} * {y} == {prod}")
        if prod >= 100:
            break inner             -- Break inner loop, continue outer loop.
        if prod == 77:
            break outer             -- Break outer loop, loop ends.
```

### Pattern Matching

The next control flow methods are based on pattern match. Generally, you see the word `is`, you next thing to expect after it is a pattern: `value is Pattern(x)`.

#### `match` / `is`

Enum/exception branching. Exhaustive by default. `| _:` for the default case.

```
match expr is ptrn:
    body
| ptrn:
    body
    –
| _:
    body
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
                           -- All choices were exhausted, so no `| _:` is necessary.
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
restult = match x is | Ptrn1: 5 | Ptrn2: 6 | _: 7
--
message = match e is | OpenError{filename}: "Open error: {filename}" | _: "Unknown error"
```

You can have multiple patterns match to one case. If any of the patterns destructure with a variable, the same variable name and type must be in all patterns. If not, use a wildcard `_` in each pattern or omit the tuples part entirely to disable destructuring.

```
match choice is
| First:
    print("First")
| Second(val) | Third{val}:                  -- `val` must be in all patterns
    print("Second or Third, val={val}")

match choice is First | Second | Third:      -- `First` doesn't have any values, so destructuring must be disabled.
    print("First, Second, or Third")
```

#### `fallthrough`

Unlike old-style `switch` blocks, each pattern block breaks automatically without the need for a `break`. Instead you can use the keyword `fallthrough` to go to the next case. The next case can't have any destructured values if `fallthrough` is used.

```
match choice is
| First:
    print("First!")
    fallthrough
| Second(_):              -- Destructuring is disabled.
    print("And second!")
    fallthrough
| Third:                  -- `(_)` is optional.
    print("And third!")
```

If you pipe into a match, you can abreviate `match $ is` to just `match`.

```
fetchA()
|> fetchB($)
|> fetchC($)
|> match           -- The same as `match $ is`.
| Pattern(x):
    print("{x}")
| _:
    print("No match")
```

#### Pattern + `if`

Inside any pattern, you can add `if` to conditionally check the variable and only match when the condition is met. Only the first condition to pass will go *(unless there's a `fallthrough`).*

```
match choice is
| Second(x if x > 0):
    print("Second is positive: {x}")
| Second(x if x < 0):
    print("Second is negative: {x}")
| Second(x):
    print("Second is zero: {x}")
| _:
    print("Choice is not Second.")
```

If you have `as` and `if` together, the `as` goes first and the `if` statement checks with the alias.

```
match choice is Third{val as x if x == 1}:
    print("Third.val is one: {x}")
| _:
    print("Either Third.val isn't one or choice isn't Third.")
```

#### `if` / `is`

Combines the conditional branching of `if` with pattern matching of `match`. Useful if you want to destructure a single case of a sum type. This must be a pattern that matches the type of the value before `is`.

```
if getStuff() is Pattern(x):
    print("value is {x}")
else:
    print("value doesn't match")

something = if getSomething() is Some(x) then x else "fallback"
```

This makes it clear that you are pattern matching instead of checking a `bool` value. We want to avoid using `=` in an `if` block because it could lead to potential ambiguity issues.

```
x = 0

if Some(x) = getStuff():  -- Did you mean `==`? Are you creating a new `x` or wrapping the existing `x`?
```

Multiple patterns can be checked for at once with `|` like with a `match` block. All patterns must go on the right of `is`. Any destructured variables must match in name and type.

```
if getResult() is ResultType1(val: int) | ResultType2{data as val: int}:
    print("val = {val}")
```

`x if` is also available like before and follows the same rules. You can iteratively match nested enums. This means there's no need for `x when` because it would be redundant.

```
if nestedOpt is Some(Some(Some(Some(x if x >= 0)))):
    print("Phew! That was a lot of unwrapping for {x}!")
else:
    print("Either none of those nested options matched or x is negative.")
```

#### `loop` / `is`

Are you sensing a pattern? We have `if`+`is`, so that means we also get… `loop`+`is`! This will loop until the pattern breaks. In some cases, this might be more desirable than the `loop` + `=>` format.

```
loop nextValue() is Some(x):
    print("value is {x}")
```

#### `until` / `is`

Like how we have `loop` and `loop`+`is`, we also have `until` and `until`+`is`. This creates a new variable below the loop when the condition is met. This is because it appears at the end of the block instead of the top, so it won't be set *until* it matches which itself is the condition for the loop to break. This can be useful if you want to repeatedly call a function *until* you get something. This means that the loop cannot conditionally break inside the body because then the variable would not be set when it breaks. This ensures that the destructured variables are defined after the loop finishes. The loop runs until the pattern matches, and when it does, the matched value is your result. The scoping rule follows directly from the semantics. The variable is visible *below* the `is`. You can't use the value inside the `loop` body since `until` is a termination condition, so there's only one place the binding could go — outside. *The scope of bindings mirrors the position of the pattern.*

```
value: mu int?

loop:
    value = getValue()             -- Try to get something.
until value is Some(x: int)        -- Loop again if it fails. Exit here if it works.
                                   -- Below the loop, `value` is guaranteed to be `Some(x)`, and `x` is guaranteed to be set.
print("value = {x}")               -- Print the unwrapped value.
```

The reason it can't break is because then the condition wouldn't be guaranteed, and any destructured variables would not be set.

```
loop:
    if someCondition:
        break                      -- Compile-time error: cannot break inside `loop`/`until when`
until getValue() is Some(x)
```

If you wish to use `break`, you should disable destructuring so that there's no need for a guarantee.

```
loop:
    if someCondition:
        break                     -- Okay!
until when Some(_) = getValue()   -- No new variables. This is fine now. 
```

#### `=>` / `else`

Pipe a pattern into a variable. If the patter breaks, it will go to `else` instead. This is like saying `if` / `is` but with less indenting. Use it to extracts an enum and binds its value to the scope. You must have `else` at the end, and if you extract any variables from the pattern, the block must break out of the scope like with `break`, `continue`, `return`, `raise`, etc.. This guarentees that the variable exists in scope like with `until`+`is`. This is analogous to the `let pattern(x) = value else {}` pattern found in other languages.

```
block label:
    getStuff() => Pattern(x) else:
        -- `x` is unset here because the pattern didn't match.
        break label
    -- `x` is set here because the pattern matched.
    print("{x}")

getStuffPlease() =
    getStuff() => Pattern(x) else:
        -- `x` is unset here because the pattern didn't match.
        raise Error("It doesn't match")
    -- `x` is set here because the pattern matched.
    print("{x}")

mightGetSomething() =
    case Some(x: int) = getSomething() else return None
    Some(x + 1)
```

This helps save you from having to indent for an `if`+`is` block.

```
getStuffPlease() =
    if getStuff() is Pattern(x):
        -- `x` is set here because the pattern matched.
        print("{x}")
    else:
        -- `x` is unset here because the pattern didn't match.
        raise Error("It doesn't match")
-- Or...
getStuffPlease() =
    getStuff() => Pattern(x) else:
        -- `x` is unset here because the pattern didn't match.
        raise Error("It doesn't match")
    -- `x` is set here because the pattern matched.
    print("{x}")
```

Like with `match`, you can also conditionally check with `if`. This works for all other pattern matching blocks too.

```
getStuffPlease() =
    getStuff() => Pattern(x if x >= 100) else:
        raise Error("It either doesn't match or x ({x}) is less than 100")
    print("{x} is definitely 100 or more.")
```

### Monadic Blocks

This blocks are for wrapping and unwrapping monadic types such as options `type?` and results `type!`. 

#### `try` / `except`

Result types are unwrapped with an exclamation mark (`!`) within a try block.

```
safeResult = try divide(1, 0)! except _ then 0.0

try:
    risky()!
except e:
    print("{e}")
```

Pattern matching works in the `except` clause like in `match` / `is`. If a pattern is missing, the exception is raised to the next block above it. The type of exception in the last `except e then` block is all the possible exceptions minus the caught exceptions. 

```
try:
    risky()!
except Exception(e):
    print("Exception {e}")
(-- implied:
except e:
    raise e
--)
```

If a function returns a result type `type!`, then it can also use the `!` like in a `try` block. Use of a `!` in a function automatically infers a result type to be the return. If the function doesn't have a return value, like with options, the type is `void!` which unwraps into an empty tuple `()`. *[See [Tuples](#Tuples).)*

```
riskyFunction(a: int): int! =
    b = doSomething1(a)!
    c = doSomething2(b)!
    c
```

Result types flatten similarly to option types. The rules go the following:

- For every `raise` or `!` in a `try` block, an exception is added to the exception sum type of the result.
- For every `except` after the `try` block, an exception is removed from the exception sum type of the result.

When all exceptions have been handled, the result is `type!void` which automatically converges to just `type`.

#### `opt`

Wraps a single expression in an option type. Inside the `opt` expression, you can use `?` to unwrap multiple option values. If one `?` returns `None`, then the whole `opt` expression stops and returns `None`. The `None`-coalescing operator `||` is optional. It unwraps the option, giving the right-hand side if the left-hand side is `None`. When you inline `opt`, it will stop the expression at the nearest `||`, so `opt x || y` becomes `(opt x) || y`. 

```
x = opt f(a?) || "fallback"         -- `x` is "fallback" if `a` is none.
x = opt f(a?) orelse "fallback"     -- Optionally use `orelse` instead.
y = opt f(a?)                       -- Or store the resulting option and unwrap it later.
z = y.unwrap()
```

Use it as a block to unwrap multiple values at once. The fallback goes to in an `else:` block. 

```
opt:
    a = getOpt(a)?
    b = getOpt(b)?
    c = getOpt(c) || 0      -- Fallback on a single option
    print("{a + b + c}")
else:                       -- Optional, runs when the `opt` block ends early.
    print("Didn't work")
```

Or inline `opt … else:` to behave like a conditional.

```
opt areYouThere()? else:
    print("Not there...")
```

If a function returns an option type, then use of the `?` is allowed without `opt`. Using an `?` inside a function will automatically infer the return type to be an option. The return will be automatically wrapped in `Some(_)`. If there isn't a return value in the function, then it should be a type `void?`. Unwrapping it gives an empty tuple `()`. *[See [Tuples](#Tuples).)*

```
addStuff(a: int, b: int, c: int): int? =
    a = getOpt(a)?
    b = getOpt(b)?
    c = getOpt(c) || 0
    a + b + c
```

If you have nested options like `type??`, you can add additional `?`s to continuously unwrap it until you get to the value. 

```
unnestOptions(x: int??): int? = x??
```

Chain `||` to unwrap multiple option types with a final fallback at the end. The type `T?` should match on all left-hand side arguments, and the final fallback needs to have a matching type `T`. The first `Some` option is returned. 

```
getFirstSome(a: int, b: int, c: int): int = getOpt(a) || getOpt(b) || getOpt(c) || 0
```

Note that although `||` normally means logical OR in other languages, Mulang uses `or` for that purpose. However, anyone who's done null coalescing in a scripting language is probably familiar with the `x or fallback` pattern. In that case, they will feel familiar with `||` since it's baked right into the language. 

Other options wouldn't work:

- `??` — is nested option unwrapping `x??`
- `?:` — colon is heavily used in the language
- `else` — would clash with `if`— `if a then o else fallback else b` *Which `else` is this?*
- `else?` — doesn't feel like an operator

So `||` is the best option remaining for the job of coalescing to a fallback.

```
opt a? || opt b? || opt c? || fallback
```

You can combine `?` and `!` together when the return type is `type?!` (an **option result type**). When unwrapping it, use `!?`. This means *"unwrap the result type"* **then** *"unwrap the option type."*

```
-- With type notation:
getSomething(x: str?): str?! =
    riskyBusiness(x?)!

-- Or inferred:
getSomething(x) =
    riskyBusiness(x?)!

-- Unwrap both in one go:
printTheThing(): void?! =
    x = getSomething("thing")!?
    print("{x}")
```

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

## Looping

- `loop` — marks any kind of loop.
- `loop cond` + `loop expr is ptrn` — loops *if* the condition is true.
- `loop` / `until` — loops *until* the condition is true.
- `loop x in expr` — loops through an iterator or array.

## Error Handling

- `expr!` — propagate an error upward (Rust/Swift style).
- Exceptions are sum types; the compiler unions all possible exception types from every `!` site in a `try` block.
- `try` / `except` can be an expression or a block and must unify return types (like `if` / `else`).
- Pattern matching on errors works with `match` or `ifcase Pattern(x) = value`.

## Pattern Matching and Destructuring

- Full `match` (exhaustive unless `| _:` is present).
- `if`+`is` — like Rust's `if let`.
- `if` on destructured values inside patterns.
- Destructuring supports structs, tuples, enum variants, and wildcards (`_`).

---

## Meta Bindings (`::`)

Variable and functions primarily use the equals sign (`=`) and are for storing actual data within a program, but there's another type of binding used for abstract values for the compiler to know about like constants, types, inline-functions, and generics. This type of declaration is **constant**; in other words, they cannot be **mutated** or **shadowed**. Depending on what it is, subsequent `::` of the same name will modify its definition. The most common is `:: impl[]` with adds methods and static variables to a meta binding. 

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

It also means the shorthand `(0, 1, x: 2)` isn't really special syntax. It's the natural representation of a tuple that has both dimensions populated, which any `&` expression across the two types would produce anyway.

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

Pair `::` and `=` together to create a constant. This can be combined together as `::=` to mean the same thing. This distinguishes them from aliases and makes it clear that there's a value on the right hand side. A constant holds an unchangeable value that must be known at compile time. Explicit typing isn't necessary since it cannot be changed.

```
PI :: = 3.14159
E ::= 2.71828   -- No space between `::` and `=` is needed.
```

You can also bind a function to a constant. To do so, but the parameter before the `=` like you would with basic functions. When calling it, it would be the same like defining it inline and then calling. This can be useful if you need to pass a function multiple times but don't want it to be outputted when compiled.

```
IDENTITY :: (x) = x
addOne :: (x) = x + 1
value = addOne(2)               -- Means (fn(x) = x + 1)(2), result is 3.
array = map([1, 2, 3, 4], addOne)
```

### Definition Blocks

Some keywords after `::` start a **definition block.** They can only be used in `::` definitions. These are special blocks used for abstract data like types and static members. 

#### Structural Types (`struct`)

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

#### Enumerable Types (`enum`)

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

#### Exception Types (`except`)

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

#### Prototypes (`proto`)

A `proto` is an abstract interface — a named contract with no data. It is equivalent to a `virtual class` (C++), `trait` (Rust), or `interface` (Java, TypeScript, etc.) in other languages. Each member is a function, also called a **method**. Methods that have a parameter named `self` at the beginning will be called like methods on the instance of that type, i.e. `self.method(…)`. This is equivalent to saying `typeof[self].method(self, …)`.

```
MyPrototype :: proto =
    speak(self): str
```

#### Implementing (`impl`)

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

### Operators (`op`)

__(WIP)__

Mu is a flexable language. The language itself can be modified to suit the needs of the programmer. Even new operators can be defined or redefined to suit anyone's needs. One of the issues with defining custom operators is that they need to be defined before they're used. Functions are always in the form `f(x)`—the parentheses make it clear: this is a function. But if you have a sequence of words like `not a and b or c`, you would need to know ahead of time to figure out what `not` means and what `and` is, are they variables or keywords? And what order does it? If you can't tell why parsing the script, it just becomes a random sequence of words. We need a away to instruct the compiler during compilation what these words mean.

The keyword `op` makes it easy to find any operator. Operators aren't like normal functions because they need to know where to go and in what order.

- __Prefix:__ `op rhs`
- __Infix:__ `lhs op rhs`
- __Postfix:__ `lhs op`

Prefix/postfix operators map easily to functions. It's just a single input and a single output. The 



```
add    / (+)  [T] :: op[  6 ](lhs:        T, rhs:    T): T    = ...
sub    / (-)  [T] :: op[  6 ](lhs:        T, rhs:    T): T    = ...
pos    / (+)  [T] :: op[  9 ](            _, rhs:    T): T    = ...
neg    / (-)  [T] :: op[  9 ](            _, rhs:    T): T    = ...
mul    / (*)  [T] :: op[  7 ](lhs:        T, rhs:    T): T    = ...
div    / (/)  [T] :: op[  7 ](lhs:        T, rhs:    T): T    = ...
fdiv   / (//) [T] :: op[  7 ](lhs:        T, rhs:    T): T    = ...
mod    / (%)  [T] :: op[  7 ](lhs:        T, rhs:    T): T    = ...
rem    / (%%) [T] :: op[  7 ](lhs:        T, rhs:    T): T    = ...
pow    / (**) [T] :: op[ -8 ](lhs:        T, rhs:    T): T    = ...
eq     / (==) [T] :: op[  3 ](lhs:        T, rhs:    T): bool = ...
neq    / (!=) [T] :: op[  3 ](lhs:        T, rhs:    T): bool = ...
gt     / (>)  [T] :: op[  3 ](lhs:        T, rhs:    T): bool = ...
lt     / (<)  [T] :: op[  3 ](lhs:        T, rhs:    T): bool = ...
gte    / (>=) [T] :: op[  3 ](lhs:        T, rhs:    T): bool = ...
lte    / (<=) [T] :: op[  3 ](lhs:        T, rhs:    T): bool = ...
and               :: op[  2 ](lhs:     bool, rhs: bool): bool = if lhs then (if rhs then True else False) else False
or                :: op[  1 ](lhs:     bool, rhs: bool): bool = if lhs then True else (if rhs then True else False)
not           [T] :: op[  9 ](            _, rhs:    T): T    = ...
not        [bool] :: op[  9 ](            _, rhs: bool): bool = if rhs then False else True
band          [T] :: op[  5 ](lhs:        T, rhs:    T): T    = ...
bor           [T] :: op[  5 ](lhs:        T, rhs:    T): T    = ...
xor           [T] :: op[  5 ](lhs:        T, rhs:    T): T    = ...
shl   / (<<)  [T] :: op[  7 ](lhs:        T, rhs:    T): T    = ...
shr   / (>>)  [T] :: op[  7 ](lhs:        T, rhs:    T): T    = ...
ushr  / (>>>) [T] :: op[  7 ](lhs:        T, rhs:    T): T    = ...
rem   / (%%)  [T] :: op[  7 ](lhs:        T, rhs:    T): T    = ...
concat / (++) [T] :: op[  6 ](lhs:        T, rhs:    T): T    = ...
spread / (++) [T] :: op[ 10 ](lhs: ref mu T, rhs:    T): void = ...
exrange / (..) [T] :: op[ 4 ](lhs:        T, rhs: T): iter[T] = ...
inrange / (..=) [T] :: op[ 4 ](lhs:       T, rhs: T): iter[T] = ...

index / (#) [T, N: uint] :: op [ 10 ](lhs: T#N, rhs: uint): T = ...
get   / (#) [T, U: type] :: op [ 10 ](lhs: T#U, rhs:    U): T = ...
coalsc / (||) [T, U] :: op[ 1 ](lhs:   T[U], rhs:    U): U    = ... 
(.) :: op ...
(&) :: op ...
(~&) :: op ...
(?) :: op ...
(!) :: op ...
(^) :: op ...
(|>) :: op ...
(=>) :: op ...
(~) :: op ...
```

- __OpType1:__ *Prefix* `lhs/rhs: T, ret: T`
    - `+ rhs`
    - `- rhs`
    - `not rhs`
- __OpType2:__ *Prefix* `lhs/rhs: T, ret: U`
    - `deref rhs`
- __OpType3:__ *Postfix* `lhs/rhs: T, ret: T`
- __OpType4:__ *Postfix* `lhs/rhs: T, ret: V`
- __OpType5:__ *Infix* `lhs: T`, `rhs: T`, `ret: T`
- __OpType6:__ *Infix* `lhs: T`, `rhs: T`, `ret: bool`
- __OpType7:__ *Infix* `lhs: T`, `rhs: W`, `ret: X`
- __OpType8:__ *Infix* `lhs: T`, `rhs: key`, `ret: Y`

__Order of Operations:__

0. Assignment / Spread (Lowest)
1. Logical OR / Pipelining
2. Logical AND
3. Comparisons/Membership
4. Range/Interval Operators
5. Bitwise Logic
6. Additive / Vector Operators
7. Multiplicative/Bit Shift Operators
8. Exponentiation *(right-associative: `2**3**2` is `2**(3**2)`)*
9. Unary Operators
10. Primary / Postfix Operators
11. Member Accessing/Inline-Binding (Highest)

add :: op[6](op

__(WIP)__

### Inheritance and Visibility

Even though structs cannot be extended the usual way, they can **inherit** from other structs using the `inherit` keyword. This works similar to **importing.** It marks members that map to members of another struct, making conversion possible. It follows the same convention for pattern matching like destructuring. *(See [Destructuring](#Destructuring).)* The struct being inherited must not be `@opaque` or else it's a compile-time error. *(See [Decorators](#Decorators) for more information on available decorators.)*

```
Vector2 :: struct =
    x: float
    y: float

Vector3 :: struct =
    inherit Vector2{x, y}
    z: float

v3 = Vector3(x: 1.0, y: 2.0, z: 3.0)

radius2d(v: Vector2) = sqrt(v.x*v.x+v.y*v.y)
print("{radius2d(v3)}") -- This works because Vector3 inherits from Vector2.
```

When you inherit, you don't just pick out some members. The entire parent struct exists in the child struct in memory, but only some members are visible. 

All members of a type are public by default. When making a subtype, inherited members become private to the subtype unless explicitly redeclared with `inherit`. This encourages separating public and private data into distinct types rather than using access modifiers. If you only inherit some fields but not all, you must put a `_` at the end of the tuple list to mark that not all members are inherited. 

```
PrivateFields :: struct =
    val: int
    secret: int

PublicFields :: struct =
    inherit PrivateFields{val, _}     -- Redeclared, `val` is public / `secret` is private
    other: int
```

A subtype cannot accidentally expose or clash with a private inherited member because types only see members that have been explicitly declared within them. This mirrors the convention used for imports.

## Meta Functions

Adding a parameter before the double colon (`::`) turns it into a **meta function** which combines the concepts of **inline functions**, **macros**, and **generics**. Parameters are put in square brackets `[]` to distinguish them from regular functions which use parentheses `()`. The result is treated like a constant for run-time code. 

You can define a meta function by adding a parameter before the double colons and writing an expression after it. Like constants, they don't output a value in memory when compiled, useful for collecting repeated code. Unlike regular functions, meta function cannot be passed to another function. They only exist at compile-time. Each parameter is a variable within the expression, so you don't need to wrap them in parentheses `()` like with C macros. 

```
max[a, b] ::= if a > b then a else b
min[a, b] ::= if a < b then a else b
```

You can also have multi-line meta function like regular functions. Each meta function creates a new scope. Defining variables that could bleed into the surrounding scope is not allowed. The last expression is the return value. Call it like a function using `[]`. 

```
doSomethingComplicated[x] ::=
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
max[a, b] ::= if a > b then a else b
min[a, b] ::= if a < b then a else b
print("{ max[0, 1] }")           -- Prints "1"
print("{ min[0, 1] }")           -- Prints "0"
print("{ max[1+2, 3+4] }")       -- Prints "7"

f(x) = x * x
g(x) = x + 2
maxAdd[a, b] ::= if a > b then fn(c) = a + c else fn(c) = b + c
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
    N: int            -- A constant `int` that must be known at compile time

List[T, N] :: struct =
    data: T#N

List[T, N] :: impl =
    init() =
        data: T#N = [++for _ in 0..N then default]
        List[T, N](data: data)
```

### Manual Implementation

Generics will automatically generate code based on their parameters, but you can also implement them by hand using pattern matching. If you only want to use the manual implementations for a generic function, you can set its body to `unset[]`. We've mentioned `unset[T]` before when declaring variables without set tinging them. `unset` itself is a meta function and not a keyword. It takes a type parameter `unset[T]` can be inferred with `_`. This creates a virtual function that can be overloaded later. If you use a function that is defined with `unset[]`, it will throw a compile-time error.

```
-- Forces every type to have its own implementation
increment[T] :: (c: ref mu T): void = unset[]

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

### Base Context / Command Line Arguments

The context reference `$` holds the command-line arguments passed to your program when it's called. This is a tuple of optional strings `str?`. The first argument `$0` is either the filepath to the script when in interpreated mode or `None` when running as a compile programm. You can use this to check if your running as a script or not.

```
if $0 is Some(x):
    print("Script name: {x}")
else:
    print("Compiled mode")

print("Hello, {$1?}!")             -- Prints first argument.
```

If your file is `hello.mu` and you run it as `mu hello.mu world`, this is what would be printed:

```
Script naem: hello.mu
Hello, world!
```

### Memory Models

Mulang is multi-paradigm: different functions, structs, or modules can use different memory strategies in the same program. The model is controlled per-module via decorators. Boundary crossing between models follows FFI-like rules — automatic marshalling where possible, explicit escapes otherwise.

Modules define how memory is handled with the `@memory` decorator. *(See [Decorators](#Decorators) for more information on available decorators.)* By default, modules use a garbage collector. Some options include `Collect(GC)` (default),  `Count(ARC)` (reference counting), and `Manual`. `GC` and `ARC` represent the standard garbage collector and reference counter respectively, but others can be defined and used instead.

```
import std.mem{memory, Count, ARC, _}

@memory(Count(ARC))
mod moduleThatUsesReferenceCounting
```

How this is implemented is outside of the scope of this document. That will be saved for when it's time to make a standard library for Mulang. For now, Mulang will focus on only implementing the garbage collector which will work in both interpreted and compiled mode.

---

## Decorators

There have been a few examples of decorators in this document like `@opaque` and `@memory`. These are compile-time functions that communicate to the compiler directly and can alter the behavior of things. The API for defining your own decorators is not set in stone yet. More information on them will be available in the future. The syntax for adding decorators goes like this:

```
@decorator1      -- No arguments.
@decorator2(arg) -- With arguments.
expr             -- Modifies whatever this expression is.
```

Decorators can be stacked and will run in reverse order. *Closest decorator to the expression runs, then the next one above that, then the next one, etc.*

Built-in decorators demonstrated so far include `@inlined`, `@capture`, `@opaque`, `@from`, and `@memory`. More planned for the future.

```
@memory(Manual) -- Call it like a function to pass a variable.
mod myModule

@opaque         -- No function needed if there are no arguments.
Thing :: struct =
    value: int

@inlined
setband(ref mu a, ref b) =
    a /\= b

mu count = 0
increment() =
    @capture(count)
    count += 1   -- In-lined decorator.
```

Some other ideas for built-in decorators include:

- `@nonlocal` - assign to a non-local variable like a one-off `@capture()`
- `@private` — locks a symbol to only be used within its module.
- `@static` — make a variable global but only available within the scope that it was defined in.
- `@inline` — marks that a regular function should inline itself like a meta function.
- `@comptime` — run a function at compile-time, return it's value as a constant.
- `@pure` — enforces pure function programming practices: *no `ref mu`, no `capture`, no `out`, etc.*
- `@safe` — enforces borrow-checking at compile time for this module or function.
- `@override` — marks that a previously implemented method will be overridden.

This is a work in progress though. How these decorators are implemented and their API are subject to change.

---

## Design Philosophy

- **Readability first** — significant whitespace and opinionated formatting.
- **Patterns scale with complexity** — simple things like declaring a mutable variable (`mu`), making a function (`fn`), or wrapping a block (`do`+`end`) use short patterns, more complex things use bigger patterns.
- **Performance on demand** — start with GC; change to a lower-level memory model where necessary.
- **Explicit but ergonomic** — `!` for errors, attributes for memory models, same keywords used between inline and block expressions.
- **Trace and auditability** — `import`, `inherit`, and `capture` require variables to be listed out to know where they're coming from; no glob-like imports.
- **Unified concepts** — `@capture` for scope capture; `inherit` for member visibility; `::` for all top-level definitions.

---

## Keywords

1. `add`
2. `and`
3. `as`
4. `await`
5. `band`
6. `bor`
7. `break`
8. `concat`
9. `continue`
10. `defer`
11. `div`
12. `do`
13. `else`
14. `end`
15. `enum`
16. `eq`
17. `except`
18. `fallthrough`
19. `fdiv`
20. `fn`
21. `gt`
22. `gte`
23. `if`
24. `impl`
25. `import`
26. `in`
27. `inherit`
28. `is`
29. `loop`
30. `lt`
31. `lte`
32. `match`
33. `mod`
34. `mod`
35. `mu`
36. `mul`
37. `neg`
38. `neq`
39. `not`
40. `opt`
41. `or`
42. `orelse`
43. `out`
44. `pass`
45. `pos`
46. `pow`
47. `proto`
48. `raise`
49. `ref`
50. `rem`
51. `return`
52. `self`
53. `shl`
54. `shr`
55. `struct`
56. `sub`
57. `then`
58. `try`
59. `until`
60. `ushr`
61. `where`
62. `xor`
63. `yield`

*NOTE: Built-in types, values, functions, and decorators like `int`, `True`, `Some`, `default`, `print`, `@memory` etc. are not considered keywords.*

---

*This document captures the current state of the Mulang design. The language is still evolving.*

---

---

---

---




































### Pipelining (`|>`)

`|>` at the start of a line passes the previous line's value as `$` into the next expression.

```
fetchA()
|> fetchB($)
|> fetchC($)
|> print("{$}")
```

`$` is the *contextual reference*. It may also appear multiple times on one line, and multiple semicolon-separated expressions share the same `$`.

```
|> fetchA()
|> print("{$}"); fetchB($)   -- print result, then fetchB
|> fetchC($)
```

A **pipeline block** begins with `|>:` and gives every expression inside it the same `$`.

```
fetchA() |> fetchB($) |>:
    print("{$}")
    fetchC($)
```

To store a pipeline result, use `=>` or its shorthand `|=>`.

```
fetchA()
|> fetchB($)
|> fetchC($) => result      -- store result mid-pipeline

fetchA()
|> fetchB($)
|> fetchC($)
|=> finalResult             -- shorthand for |> $ =>
```

### Operation Chaining (`op[]`)

Any infix operator `op` may be written as `lhs op[rhs]`, at the same precedence as member access. This is primarily useful for array indexing and inline pipelining.

```
array#[0]#[1]                        -- ( (array # 0) # 1 )
fn1() |>[fn2($)] |>[fn3($)]          -- fn3(fn2(fn1()))
```

Lines starting with `op[]` continue the previous expression, allowing operator chaining across lines.

```
bigString = "This"
    ++[" string"]
    ++[" is"]
    ++[" split"]
    ++[" across lines."]
```

If the brackets contain more than one value or named values, the result is a **tuple** rather than an operation.

```
x = (13, 5) |>[ $0 // $1, $0 %% $1 ]   -- (2, 3)
```

---

## Variables and Declarations

```mu
x = 42                  -- inferred
y: Int = 42             -- explicit type
z := 42                 -- always inferred declaration
```

**Mutability**: Use `mut` keyword (details in full spec).

---

## Control Flow

### Conditionals

```mu
if condition:
    ...
else if other:
    ...
else:
    ...
```

Inline form:
```mu
if condition then expr else expr
```

### Loops

- `while condition:`
- `until condition:` (loop while *not* condition)
- `for item in iterable:`
- `loop: ...` (infinite, with `break` / `continue`)

### Pattern Matching

```mu
match value is
| Pattern1(x): ...
| Pattern2(y || default): ...
| _: ...
```

**Proposed `is ... then` sugar** (from our discussion):
```mu
result = value is Ok(x || default) then x
```

---

## Functions

```mu
add(a, b) = a + b

-- With block
process(data):
    ...
    result
```

Lambdas:
```mu
fn(x) = x * 2
do fn(x): ... end
```

---

## Types

- **Primitives**: `Int`, `Float`, `Bool`, `Char`, `String`
- **Tuples**: `(a, b, c)`
- **Arrays / Lists**
- **Option**: `T?` or `opt T`
- **Result / Error handling**
- **Structs / Records**
- **Enums / Sum types** (via pattern matching)

**Tuple operations** (as discussed):
- Element-wise when shapes match: `(a, b) + (c, d) → (a+c, b+d)`
- Broadcasting for scalars

---

## Error and Option Handling

- `?` — propagate on None/Error
- `!` — propagate exceptions
- `||` — coalescing
- `opt` and `try` blocks

---

## Modules and Organization

(Planned / to be detailed)

---

**Here's a more formal, specification-style rewrite of the Mu (Mulang) Language Reference.**

---

# Mu Language Specification

**Version 0.1 (Draft)**  
**Language Name**: Mu (Mulang)  
**Status**: Experimental

## Introduction

Mu (also known as **Mulang**) is a general-purpose, multi-paradigm programming language designed for **readability**, **expressiveness**, and **low-level control**. It features significant whitespace while providing explicit mechanisms to escape into inline mode.

Key design goals:
- Strong expression-orientation
- Hybrid significant/insignificant whitespace model
- Modern error and option handling
- Support for both interpretation and compilation (including direct shared library output)
- Suitable for systems programming, robotics, AI, games, and high-performance applications

This document defines the formal syntax and semantics of the core language. Standard library features are out of scope.

---

## 2. Lexical Conventions

### 2.1 Character Set
Source code is UTF-8 encoded.

### 2.2 Whitespace and Indentation
- **Indentation is significant**.
- Recommended indentation: 4 spaces.
- All expressions in the same block must share identical indentation prefix (exact sequence of spaces and tabs).
- Mixing tabs and spaces is permitted only if the entire prefix is consistent within a block.

### 2.3 Comments
- Line comment: `--` until end of line
- Block comment: `(--` ... `--)`, nesting supported

### 2.4 Literals
- **Strings**:
  - `"..."` — interpolated with `{expression}`
  - `'''...'''` — raw string
  - `"""..."""` — multi-line string
- **Characters**: `'c'`
- **Numbers**: Decimal, hexadecimal (`0x`), binary (`0b`), with underscores for readability

### 2.5 Separators
- Newlines (`\n`, `\r`, `\r\n`) or `;` separate expressions
- `,` separates tuple and array elements

---

## 3. Syntax Fundamentals

### 3.1 Expressions
A Mu program is a sequence of expressions. Nearly all constructs are expressions and produce values.

### 3.2 Blocks
A block is introduced by `:` (or `=` for definitions) followed by a newline and increased indentation. The value of a block is the value of its last expression.

### 3.3 Inline-Block Mode (`do` / `end`)
To switch from block mode to inline mode (and vice versa):

```mu
do <construct>:
    ...
end
```

`do` begins an inline block; `end` terminates the nearest open `do`.

### 3.4 Expression Continuation
An expression may span multiple lines when:
- Inside brackets: `()`, `[]`, `{}`
- A line starts with `.` (method chaining)
- A line starts with `|>` (pipeline)
- Inside a multi-line string literal (`"""`)

---

## 4. Operators

### 4.1 Precedence (Highest to Lowest)

### 4.2 Operator Forms

Assignment variants (`+=`, `=>`, etc.) and prefix word forms are supported for applicable operators.

---

## 5. Variables and Bindings

```mu
x = 42                    -- inferred
y: Int = 42               -- explicit type
z := 42                   -- forced inference
mut counter = 0           -- mutable
```

---

## 6. Control Flow

### 6.1 Conditionals
```mu
if condition:
    ...
else if condition2:
    ...
else:
    ...
```

Inline form:
```mu
if condition then expr else expr
```

### 6.2 Loops
- `while condition:`
- `until condition:` (loop while condition is false)
- `for pattern in iterable:`
- `loop:` (infinite)

### 6.3 Pattern Matching
```mu
match expr is
| Pattern1(x): expr
| Pattern2(y || default): expr
| _: expr
```

**Proposed extraction sugar** (non-normative extension):
```mu
value is Some(x || default) then x
```

---

## 7. Functions and Procedures

```mu
add(a: Int, b: Int) = a + b

process(data):
    ...
    final_result
```

Lambda expressions:
```mu
fn(x) = x * 2
do fn(x): ... end
```

---

## 8. Type System (Overview)

- **Primitives**: `Int`, `Float`, `Bool`, `Char`, `String`
- **Product types**: Tuples `(T, U)`, Records
- **Sum types**: Enums (via pattern matching)
- **Wrapper types**: `opt T` (or `T?`), `Result<T, E>`
- **Pointer types**: Typed pointers with dereference `^`

**Tuple arithmetic** (defined behavior):
- Element-wise when shapes match
- Broadcasting of scalars

---

## 9. Error and Option Handling

- `expr?` — propagate `None` / error to nearest `opt` / `try`
- `expr!` — propagate exception to nearest `try`
- `expr || fallback` — None coalescing
- `opt` and `try` blocks for scoped handling

---

## 10. Memory Model (Planned)

Mu will expose explicit control over:
- Ownership and borrowing
- Stack vs heap allocation
- Raw pointers and manual memory management
- Shared library interop

---

---

---

---

---




































---

---

## 4. Bindings

There are two kinds of bindings: basic (`=`) and meta (`::`). Meta bindings are compile-time declarations such as types, protocol implementations, and custom operators.

### Variable Declarations

```
a = 0               -- Implicit declaration, type inferred.
b: int = 1          -- Explicit type.
c := 2              -- Explicit declaration, inferred type.
d: = 4              -- Same as above, space between `:` and `=` is fine.
```

A newline after `=` starts a block. The last expression is the variable's value.

```
lunch =
    if getDayOfWeek() == "Tuesday":
        "tacos"
    else:
        "sandwich"
```

Variables are immutable by default. Redeclaring with the same name **shadows** it; the type does not need to match.

```
a = 1
a = 2           -- New variable, shadows previous `a`.
a = "hello"     -- Type can change when shadowing.
```

`=` is a void statement and may not be used inside expressions. Use `==` for comparison and `=>` for inline assignment.

```
-- Error:
if x = 0: ...

-- Correct:
if x == 0: ...
```

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

Functions do not automatically capture mutable variables. Any assignment inside a function to an outer mutable variable creates a new local variable unless explicitly captured.

```
mu count = 0

addCount() =
    @capture(count)
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

| Syntax | Meaning |
|:-------|:--------|
| `ref x = y` | Immutable reference, inferred type |
| `x: ref T = y` | Immutable reference, explicit type |
| `ref mu x = y` | Mutable reference, inferred type |
| `x: ref mu T = y` | Mutable reference, explicit type |

### Destructuring

```
(a, b) = (0, 1)                         -- Positional.
{x} = {x: 2}                            -- Named.
(a, b) & {x} = (0, 1, x: 2)            -- Mixed.
{x as y} = {x: 2}                       -- Alias.
{x as y: int} = {x: 2}                  -- Alias with type.
{0 as a, 1 as b} = (3, 4)              -- Positional by index.
(_, b, _) & {x, _} = (0, 1, 2, x: 3, y: 4)  -- Skip with `_`.
```

When destructuring a named type, the type may be placed after the last tuple.

```
Thing :: {x: int, y: int}
{x, y}: Thing = thing
{x, y} = thing              -- Or infer.
```

### Function Declarations

```
add(a: int, b: int): int = a + b
add(a, b) = a + b           -- Inferred types.
```

Block body — last expression is the return value.

```
fib(n) =
    if n < 1:
        0
    else if n < 2:
        1
    else:
        fib(n - 1) + fib(n - 2)
```

Function pointer type and assignment.

```
action: mu fn(int, int): int
action = fn(a, b) = a + b
```

#### Parameter Modifiers

| Modifier | Copies/References | Mutable in function |
|:---------|:-----------------|:--------------------|
| *(none)* | Copy | No |
| `mu` | Copy | Yes |
| `ref` | Reference | No |
| `ref mu` | Reference | Yes |
| `out` | Reference | Yes, must be set |

`out` parameters treat the variable as unset on entry and guarantee it is set before the function returns.

```
setInt(out i): void =
    i = 3

x: mu int
setInt(x)

-- Or use `=>` in the parameter to declare inline:
setInt(=> n)
print("{n}")    -- "3"
```

#### Lambda Functions

```
map(array0, fn(x) = x + 1)

map(array0, do fn(x) =
    if x < 2: x - 1
    else: x + 2
end)
```

A name is optional; naming a lambda creates an immutable self-reference for recursion.

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

Functions may declare required context variables with `$name: type`. These are resolved from the calling scope rather than passed as arguments.

```
addContext() =
    $a: int
    $b: int
    $a + $b

$a = 1
$b = 2
result = addContext()   -- Uses $a and $b from context.
```

Optional contextual parameters use `T?`.

```
isThereX() =
    $x: int?
    match $x is
    | Some(_): print("$x exists")
    | None:    print("No $x")
```

### Meta Bindings (`::`)

`::` declares compile-time structural information. All `::` bindings share the same signal: *this is structural, not runtime.*

| Form | Meaning |
|:-----|:--------|
| `Name :: { ... }` | Type definition |
| `Name :: (...)` | Tuple type definition |
| `Name :: impl[P] =` | Protocol implementation |
| `name :: op(a, b) =` | Custom operator |
| `name :: const =` | Compile-time constant |

---

## 5. Types

### Type Notation

| Notation | Meaning |
|:---------|:--------|
| `T` | Plain type |
| `T?` | Option (may be `None`) |
| `T!` or `T!E` | Result (may be exception `E`) |
| `T#` | Dynamic array |
| `T#N` | Fixed array of length N |
| `T##` | 2D array (one `#` per dimension) |
| `V#K` | Dictionary with key type K |
| `T^` | Immutable typed pointer |
| `T^mu` | Mutable typed pointer |
| `fn(T, T): T` | Function pointer |

### Built-in Types

`int`, `uint`, `float`, `bool`, `char`, `str`, `ptr`, `void`

These are not keywords — they follow the naming convention of lowercase for built-in types. User-defined types should use capitalized names.

### Compile-Time Functions

```
typeof[x]           -- Type of x at compile time.
sizeof[T]           -- Size of T in bytes.
default[T]          -- Default value of T.
default[]           -- Infer T from context.
```

### Type Conversion

Use `~` when the target type can be inferred.

```
x: float = 1.5
y: int = ~x         -- Calls int(x) → 1
```

When the target type cannot be inferred, call it explicitly with `~` on the argument.

```
y = float(~x)       -- Convert x to float.
```

### Booleans

`bool` is a built-in enum with members `True` and `False`.

### Numbers

| Type | Meaning | Suffix | Examples |
|:-----|:--------|:------:|:---------|
| `int` | Signed integer | `i` | `1`, `2i`, `0xabcdef` |
| `uint` | Unsigned integer | `u` | `0u`, `5u` |
| `float` | Floating point | `f` | `1.0`, `1.5f`, `2.0e100` |

Numbers are case-insensitive. Underscores may be placed anywhere for readability. Leading zeros are allowed without changing the base.

```
1_234_567
0b1111_0000
0o777
0xFF
2.0e-10
```

Space operators correctly when adjacent to `++` or `--`.

```
1+ +1   -- Addition: 2
1++1    -- Array: [1, 1]
1- -1   -- Subtraction: 2
1--1    -- Just 1 (with a comment)
```

### Characters

Single-byte values written with apostrophes. Arithmetic is allowed.

```
a = 'a'
b = a + 1       -- 'b'
tab = '\t'
unicode = '\uFFFF'
```

### Strings

| Form | Purpose |
|:-----|:--------|
| `"…"` | Regular string with `{expr}` interpolation |
| `"""…"""` | Multiline with interpolation; whitespace trimmed at closing `"""` position |
| `''…''` | Inline raw string, no interpolation or escaping |
| `@"""…"""@` | Multiline raw string; `@` count must match to close |

```
name = "world"
hello = "Hello, {name}!"
escaped = "Literal brace: \{"

path = ''C:\files\on\windows.txt''

doc = """
    Line one.
    Line two.
    """
```

Subsequent string literals are automatically concatenated. Use `++` for runtime concatenation.

```
str1 = "Hello" ", " "world"
str2 = str1 ++ "!"
```

### Arrays

```
list: int#4 = [1, 2, 3, 4]
grid: int#2#2 = [[1, 2], [3, 4]]
item = grid#1#0         -- Row 1, column 0 → 3
```

Spread into an array with `++`.

```
a = [1, 2, 3]
b = [0, ++a, 4]     -- [0, 1, 2, 3, 4]
c = 1 ++ 2 ++ 3     -- [1, 2, 3]
```

The familiar `array[i]` form is also accepted. The compiler resolves it to `array#[i]` based on type. Tooling will suggest migrating to the `#` form.

### Dictionaries

A subtype of arrays where each item has a key. Type notation is `V#K`.

```
dict: int#str = [
    a: 1,
    b: 2,
    ["invalid name"]: 127,
]
print("{dict.b}")           -- "2" (string keys that are valid names allow `.`)
print("{dict#["b"]}")       -- "2"
```

### Tuples

Positional tuples use `()`, named tuples use `{}`. A tuple with only one positional member and no named members is equal to that member.

```
(10) == 10      -- True.
```

Mixed tuples combine both with `&`.

```
(a, b) & {x: 1} = (0, 1, x: 1)
```

Use `&[]` to update named members of a tuple in place.

```
user = User.create()&[
    name: "John Doe",
    dob:  "1970-01-01",
]
```

Use `~&[]` to create a new product type by spreading both sides.

```
tuple = ()
    ~&[a: 1]
    ~&[b: 2]
    ~&[c: 3]
-- tuple is (a: 1, b: 2, c: 3)
```

### Pointers

Opaque pointers use `ptr`. Typed pointers use `T^` or `T^mu`. Dereferencing and pointer arithmetic require an `unsafe:` block.

```
result: ptr = ExternalLib.getSomething()
if result == Null:
    raise NotFound

unsafe:
    mu x = 0
    xPtr: int^mu = ~ptr(x)
    xPtr^ = 1
    print("{x}")    -- "1"
```

| Type | Can reassign pointer | Can mutate memory |
|:-----|:--------------------:|:-----------------:|
| `T^` | No | No |
| `T^mu` | No | Yes |
| `mu T^` | Yes | No |
| `mu T^mu` | Yes | Yes |

---

## 6. Control Flow

All branching constructs share a consistent pattern:

```
-- Block form (`:` after subject)
keyword subject:
    body

-- Inline form (`then` after subject)
keyword subject then expr
```

Variables created in the subject field shadow the parent scope.

### `pass`

Leaves a block empty.

```
block:
    pass
```

### `block`

Creates a new scope. Its value is the last expression evaluated.

```
x = block:
    y = 1
    y + 1       -- block's value is 2
```

Give it a label to enable `break`.

```
block outer:
    break outer
```

### `if` / `else`

```
x = if x > 0 then x else -x

if a and b:
    print("both")
else if a or b:
    print("one")
else:
    print("neither")
```

### `loop`

The universal loop keyword. All loop forms share the same `loop` keyword.

```
-- Unconditional (break manually):
loop:
    body
    break

-- While condition is true:
loop cond:
    body

-- For-each:
loop x in collection:
    body

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

Inlined `loop x in` returns a lazy iterator collected with `++`.

```
doubled = [++ loop x in list then x * 2]
```

Destructuring works in loop variables.

```
loop (x, y, z) in listOfTuples:
    print("{x}, {y}, {z}")
```

### `break` / `continue`

Both accept an optional label to target an outer loop.

```
block outer loop x in 0..100:
    block inner loop y in 0..100:
        if x * y >= 100:
            break inner
        if x * y == 77:
            break outer
```

---

## 7. Pattern Matching

Wherever you see `is`, a pattern follows.

### Pattern Binding Modifiers

| Form | Meaning |
|:-----|:--------|
| `P(x)` | Guaranteed match — `x` is `T` |
| `P(x?)` | Possible non-match — `x` is `T?` |
| `P(x \|\| fallback)` | Possible non-match — `x` is `T`, defaults to `fallback` |

`x?` is required when the compiler cannot guarantee the pattern always matches (e.g., a loop with a reachable `break`). `x || fallback` is sugar for `x?` with an immediate unwrap.

### `match` / `is`

Exhaustive by default. Use `| _:` for a catch-all.

```
match choice is
| First:
    print("First")
| Second(x):
    print("Second: {x}")
| Third{val}:
    print("Third: {val}")
| _:
    print("Other")
```

Inline form.

```
result = match x is | A: 1 | B: 2 | _: 0
```

Multiple patterns per case — all must bind the same variable names and types.

```
match choice is
| Second(val) | Third{val}:
    print("val = {val}")
```

#### `fallthrough`

Proceeds to the next case. The next case must not destructure new values, unless optional bindings are used.

```
match choice is
| First:
    print("First")
    fallthrough
| Second(x?):
    if x is Some(x):
        print("Definitely Second: {x}")
```

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
```

### `if` / `is`

Pattern match a single case with an optional `else`.

```
if getStuff() is Pattern(x):
    print("{x}")
else:
    print("No match")

something = if getSomething() is Some(x) then x else "fallback"
```

### `is` / `then`

Extract a binding inline. Requires a guaranteed match or optional bindings.

```
-- Guaranteed match (exhaustive type):
result = value is Pattern(x) then x

-- With fallback (non-exhaustive):
result = value is Pattern(x || "fallback") then x

-- Multiple bindings:
result = value is Pattern(x || 0, y || 0) then (x, y)

-- Arbitrary expression over bindings:
result = value is Pattern(x || 0, y || 0) then x + y
```

Pairs naturally with pipelining.

```
getValue()
|> $ is Pattern(x || "fallback") then x
|> doSomethingWith($)
```

### `loop` / `is`

Loop while a pattern matches.

```
loop nextValue() is Some(x):
    print("{x}")
```

### `until` / `is`

Loop until a pattern matches. Bindings are in scope below the loop.

```
loop:
    value = getValue()
until value is Some(x)

print("{x}")    -- x is guaranteed set here.
```

If `break` is reachable inside the loop, optional bindings are required.

```
loop:
    if earlyCondition:
        break
until getValue() is Some(x?)

if x is Some(x):
    print("{x}")
```

### `=>` / `else`

Extract a pattern into scope. The `else` block must exit the current scope (`break`, `return`, `raise`, etc.), guaranteeing the binding exists after it.

```
getStuff() => Pattern(x) else:
    raise Error("No match")

print("{x}")    -- x is guaranteed here.
```

With a guard.

```
getStuff() => Pattern(x if x >= 100) else:
    raise Error("x is less than 100")

print("{x}")    -- x is >= 100 here.
```

---

## 8. Monadic Blocks

### `try` / `except`

Unwrap result types with `!` inside a `try` block. Unhandled exceptions propagate upward.

```
try:
    a = doSomething1(x)!
    b = doSomething2(a)!
    b
except Exception(e):
    print("Error: {e}")
    0
```

Inline form.

```
result = try divide(1, 0)! except _ then 0.0
```

Using `!` inside a function automatically infers a result return type.

```
riskyFn(a: int): int! =
    b = step1(a)!
    c = step2(b)!
    c
```

### `opt`

Unwrap option types with `?` inside an `opt` block. If any `?` returns `None`, the block short-circuits.

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

Using `?` inside a function automatically infers an option return type.

```
addStuff(a: int, b: int): int? =
    x = getA()?
    y = getB()?
    x + y
```

Nested options unwrap with multiple `?`.

```
unnest(x: int??): int? = x??
```

Chain `||` to try multiple options with a final fallback.

```
getFirst(a: int, b: int, c: int): int =
    getA(a) || getB(b) || getC(c) || 0
```

---

*This document covers the core language. Standard library features, FFI, the compilation model, and asynchronous programming are outside this scope.*
