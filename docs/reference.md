# Mu Language Syntax and Semantics Reference

*Version 0.1 (Draft)*

The Mu programming language or *Mulang* is a general-purpose, multi-paradigm programming language with significant whitespace. It targets programmers who want to be able to write expressive code but also have low-level control. It is expression-oriented where possible and provides explicit control over evaluation strategy, memory model, and error handling. Mulang is not afraid to break conventions and try new things. That's what making a new programming language from scratch is all about. It's experimental—it's meant to make you think—and may not be for everyone. 

Mulang is planned to be both a compiled and an interpreted language. You won't need to use another language like C to increase performance. You can instead compile some of it and then call it like a shared library all within the same language. Some use cases for this are AI, systems programming, and game development.

This document will focus on the language itself. Some features may come in a standard library which will not be discussed here.

---

## Lexical Conventions

- **Indentation**: Significant whitespace (4 spaces recommended). 
- **Statements**: Each statement is divided by new lines. Semi-colons (;) can also be used. 
- **Comments**: Double dash (`--`) to end-of-line. Block comments start with `(--` and end with `--)`.
- **String literals**: `"…"` with interpolation with `{expr}` inside. To write a literal curly brace (`{`), escape it with a backslash `\{`. 

## Basic Syntax

Comments are made with double dash `--`.

```
-- Comment until new line.

(--
Multi-line
comment.
--)

(--
    (--
        Nesting is allowed.
    --)
--)
```

A Mu program consists of a list of **expressions** separated by new-lines or semi-colons (`;`). New-lines and semi-colons are interchangeable and treated the same. 

```
expr
expr

expr; expr
```

Almost everything is an expression. Some statements can be either inline or block depending on the presence of a colon (`:`) or equals sign (`=`) followed by a new line.

Parts of an expression are divided into 4 categories:

1. __Words:__ variable names: `x`, `PI; numeric constants: `1`, `3.14`, `0xABCDEF`;
2. __Char/String Literals:__ things surrounded in quotes: `'a'`, `"foo"`, `"""big string"""`, `$''raw string''$`
3. __Delimiters:__ commas `,` for tuples and arrays; `;` for subsequent expressions
4. __Symbols:__ operators and anything with these characters: `~!@#$%^&*-+=\|:<.>/?` *but excluding the symbol for a comment `--`*
5. __Bracket Expressions:__ anything in parentheses `()`, square brackets `[]`, or curly braces `{}`; *note: the term __bracket__ means any of these characters `()[]{}`, and the term __square bracket__ means just these characters `[]`*
6. __Whitespace:__ spaces, tabs `\t`, new lines `\n`/`\r`, etc.

Whitespace is significant. **Indentation** is used to mark when blocks start and end. Indentation can be any whitespace character except for new lines. Inside any block, expressions should start with the same indentation throughout. This includes the number and value of indentations. You could mix tabs and spaces, but each subsequent expression in a block must have the same number of tabs and spaces and in the same order. It's recommended to use **4 spaces** per indentation, adding 4 more for each block.

A single new lines can be either Carriage Return (`\r`), Line Feed (`\n`), or both (`\r\n`). A new line marks the end of an expression *unless the line contains an open bracket or quotation mark.* When you wrap a single expression in parentheses, this is called *expression splitting.*

Examples:

```
-- This is two expressions:
expr1
expr2

-- Split one expression into multiple lines, ignores indentation until closing bracket:
(expr3p1
    expr3p2
   expr3p3
      expr3p4)
```

**Expression splitting** is when you split an expression into multiple parts. Line breaks are ignored when an expression is in-between any kind of brackets. The common use case for this is for *method chaining.*

```
(object.method1()
       .method2()
       .method3()
       .method4())
```

Adding a newline and indentations after a `:` or `=` starts a block. A **block** wraps multiple expressions into one. *The last expression evaluated in any block becomes its value.* Expression splitting is also disabled for the first line in a block. For this demonstration, we'll use `block:` to represent any block-type. *(See [Control Flow](#Control-Flow) for more details about different types of block expressions.)*

```
block:
    expr
    expr
```

Whitespace is significant. Indentation marks the where a block starts and ends. All subsequent expressions within a block must have the same indentation or more. If an expression has less indentation than the first but more than the opening of the block, then that's a syntax error. If an expression has more indentation than the first, then it must be in a new block or split expression, otherwise it's also a syntax error.

```
block:
    block:
        expr
                 -- Ends both blocks.
```

While significant whitespace saves a lot of typing so many `end` or closing curly braces `}`, this can also make it difficult to write expressive code at times. *What if you wanted to pass a function to another function?* In Python, you might define the function within a function like this:

```py
def doThing():
    def callback(result):
        if result > 0:
            print(f"Success! {result}")
        else:
            print(f"Failure! {result}")
    apiFetch(callback)
```

But you can't *inline* the function directly like you can in other languages. You need some way switch between significant whitespace for **block mode** and insignificant whitespace for **inline mode.** Mulang's solution is the `do`/`end` pattern. That's the signal to make the switch between block mode and inline mode. **`end` always closes the nearest unclosed `do` block, and `do` must always close with an `end`.** The two keywords are semantically linked.

```
doThing() =
    apiFetch(do fn(result) =
        if result > 0:
            print("Success! {result}")
        else:
            print("Failure! {result}")
    end)
```

`do` signals to switch to block mode. After the `do` comes a block keyword, and then there needs to be a `:` (or `=` for functions) at the end of the line. When the keyword `end` appears, that's the signal to switch back to inline mode. The expression between `do` and `end` is called an **inline-block expression.** This allows you to write expressive code depending on your needs. 

```
(do block:       -- `do` → Switch to block mode.
    expr         -- Inside the inline-block expression, significant whitepace here.
end)             -- `end` → Switch to inline mode.

(do block:       -- This starts a block, so indentation is significant.
    block:       -- This starts another block inside that block.
        expr     -- Inside the second block.
    expr         -- Unindent to escape the second block.
end)             -- Use only one `end` to end to whole block.

-- Another example with more nested blocks:
(do block:
    block:
        block:
            block:
                expr
end)

-- Nesting also works:
(do block:
    block:
        (do block:
            expr
        end)
end)
```

Not all blocks need to be wrapped in `do`…`end` all of the time. Only when they need to switch from inline mode to block mode. Most blocks in Mulang can be **inlined** though. Some are based on whether they use a `:` or not. If a block has a *subject* component, then the `:` goes after the subject in block mode but is swapped for the keyword `then` in inline mode. *(See [Control Flow](#Control-Flow) for more details.)*

```
if x then "True" else "False"  -- Inline expression.

if x:           -- `:` + new line here, so start a block.
    expr        -- Next line should be one or more indentations higher.
else:           -- Unindenting exits the block.
    expr        -- This is a new block.

expr            -- Unindenting exits the block.

(do if x:       -- `do` starts an inline-block expression.
    expr
else:           -- You can also start another block until you reach `end`.
    expr
end)            -- `end` finishes the inline-block expression.
```

*See [Control Flow](#Control-Flow) for more information on the types of blocks.*

## Operators

The philosophy of Mulang is that symbols should be easy to recognize and understand. Good symbols can also help make code easier to both write and read at times. Many are semantically grouped based on their contextual usage, for example `*` and `/` relate to math, `?` relates to options, `&` relates to tuples, etc.. There's also a common pattern where repeating an operator gives you a more technical or complex version of that operator, for example: `+` addition vs `++` concatenation, `*` multiplication vs `**` exponentiation, `/` division vs `//` floor division, and `%` standard modulo vs `%%` floor division modulo.

| Operation      | Meaning | Order |
|:---------------|:--|:--|
| __(Arithmetic)__ | — | — |
| `lhs + rhs`    | addition | 6 |
| `lhs - rhs`    | subtraction | 6 |
| `+ rhs`        | keeps the sign the same; *so does nothing* | 9 |
| `- rhs`        | sign-flip | 9 |
| `lhs * rhs`    | multiplication | 7 |
| `lhs / rhs`    | exact division; *returns a floating-point number* | 7 |
| `lhs // rhs`   | floored division (rounded down); *returns an integer* | 7 |
| `lhs % rhs`    | modulo (sign matches `lhs`) | 7 |
| `lhs %% rhs`   | floor division modulo (sign matches `rhs`); *result is between the range `[0, rhs)` if `rhs` is positive or `(rhs, 0]` if `rhs` is negative* | 7 |
| `lhs ** rhs`   | exponential *(right associative: `2**3**2` is `2**(3**2)`)* | 8 |
| __(Comparison)__ | — | — |
| `lhs == rhs`   | equality | 3 |
| `lhs != rhs`   | inequality | 3 |
| `lhs > rhs`    | greater than | 3 |
| `lhs < rhs`    | less than | 3 |
| `lhs >= rhs`   | greater than or equals to | 3 |
| `lhs <= rhs`   | less than or equals to | 3 |
| __(Boolean)__  | — | — |
| `lhs and rhs`  | false if any are false | 2 |
| `lhs or rhs`   | true if any are true | 1 |
| `not rhs`      | inverts a boolean or bitwise-`NOT` when `rhs` is a number | 9 |
| `and rhs`      | do `and` on each component of a tuple: `and (True, False, True)` → `True and False and True` → `False` | 9 |
| `or rhs`       | do `or` on each component of a tuple: `or (True, False, True)` → `True or False or True` → `True` | 9 |
| __(Bitwise)__  | — | — |
|  `lhs /\ rhs`  | bitwise-`AND` *(resembles a wedge* $\land$ *, the symbol for logical AND; also resembles a capital A)* | 5 |
| `lhs \/ rhs`   | bitwise-`OR` *(resembles a vee* $\lor$ *, the symbol for logical OR; also invert of `/\`)* | 5 |
| `lhs >< rhs`   | bitwise-`XOR` *(resembles an X for XOR)* | 5 |
| `lhs << rhs`   | bitshift-left | 7 |
| `lhs >> rhs`   | bitshift-right | 7 |
| `lhs >>> rhs`  | bitshift-right (unsigned) | 7 |
| __(Arrays)__   | — | — |
| `lhs # rhs`    | get an item from `lhs` at an index `rhs` (index starting at 0) | 10 |
| `lhs ++ rhs`   | concatenation, returns a new array | 6 |
| `++ rhs`       | spread an array or iterator into an array or positional tuple | 10 |
| `lhs .. rhs`   | creates an iterator that starts at the left value and ends just before the right value (exclusive) | 4 |
| `lhs ..= rhs`  | creates an iterator that starts at the left value and ends with the right value (inclusive) | 4 |
| `lhs in rhs`   | checks if an item exists in an array, returns a boolean | 3 |
| __(Tuples)__   | — | — |
| `lhs . rhs`    | access a member/component | 11 |
| `& rhs`        | spread a tuple into another tuple | 10 |
| __(Options)__  | — | — |
| `lhs ?`        | returns the `Some` value if it's not `None`, otherwise propagate to the nearest `opt` keyword *(see [`opt` block](#opt))* | 10 |
| `lhs || rhs`   | `None`-coalescing; if `lhs` is `Some(x)` then `x` else `rhs` | 1 |
| __(Results)__  | — | — |
| `lhs !`        | returns the result value if it's not an exception, otherwise propagate to the nearest `try` keyword *(see [`try` block](#try--except))*        | 10 |
| __(Pointers)__ | — | — |
| `lhs ^`        | dereferences a typed pointer | 10 |
| __(Functional)__ | — | — |
| `lhs \|> rhs`  | pipelining, disregards `lhs` and returns `rhs` | 1 |
| `~ rhs`        | inferred type conversion | 10 |
| __(Assignment)__ | — | — |
| `lhs = rhs`    | assignment or inferred-type declaration *(See [Variable Declarations](#Variable-Declaration).)* | 0 |
| `lhs := rhs`   | always inferred-type declaration | 0 |
| `lhs: T = rhs` | explicit-type declaration | 0 |
| `lhs => rhs`   | inline for `:=` but returns the assigned value instead of being void; the operands are inverted (variable goes on the right); creates an immutable variable by default *(See [Inline Binding](#Inline-Binding).)* | 0 |
| `lhs += rhs`   | increment | 0 |
| `lhs -= rhs`   | decrement | 0 |
| `lhs *= rhs`   | multiplication assignment | 0 |
| `lhs /= rhs`   | division assignment | 0 |
| `lhs //= rhs`  | floor division assignment | 0 |
| `lhs %= rhs`   | modulo assignment | 0 |
| `lhs %%= rhs`  | floor division modulo assignment (binds `lhs` to a range in `[0, rhs)` if `rhs` is positive or `(rhs, 0]` if `rhs` is negative) | 0 |
| `lhs **= rhs`  | exponential assignment | 0 |
| `lhs /\= rhs`  | bitwise-`AND` assignment | 0 |
| `lhs \/= rhs`  | bitwise-`OR` assignment | 0 |
| `lhs ><= rhs`  | bitwise-`XOR` assignment | 0 |
| `lhs <<= rhs`  | bitshift-left assignment | 0 |
| `lhs >>= rhs`  | bitshift-right assignment | 0 |
| `lhs >>>= rhs` | unsigned bitshift-right assignment | 0 |
| `lhs ++= rhs`  | append to an array (not allowed if `lhs` is a fixed length array) | 0 |

__Order of Operations:__

12-  0. Assignment/Pipelining (Lowest)
11-  1. Logical OR
10-  2. Logical AND
9 -  3. Comparisons/Membership
8 -  4. Range/Interval Operators
7 -  5. Bitwise Logic
6 -  6. Additive/Vector Operators
5 -  7. Multiplicative/Bit Shift Operators
4 -  8. Exponentiation *(right-associative: `2**3**2` is `2**(3**2)`)*
3 -  9. Unary Operators
2 - 10. Primary/Postfix Operators
1 - 11. Member Accessing/Inline-Binding (Highest)

Some operators have assignment alternatives by adding an equals sign `=` after it. These are reserved for the operators that aren't keywords and return the same type that their left-hand side is. This sets a variable based on its previous value. The left-hand side must be an already defined variable. If it's immutable, then this shadows it. If it's mutable, then the value is mutated. *(See [Mutability(#Mutability-mu).)* All of these are void statements, i.e. they return nothing and should only be used in an expression by themselves.

All assignment operators *(except for `=>`)* are **void.** This is intentional to prevent bugs and difficult to read code. It's recommend to put all assignment statements on their own separate lines for clarity. 

__On the choice of the unique operators...__

Keywords bitwise operators use different symbols than their conventional `&|^~` in other languages because those symbols have different meanings by themselves in Mulang: `&` → tuples, `|` → pattern matching, `~` → type conversion. `&&` and `||` are usually Boolean operators and would also be ambiguous, even though those operations are handled by the keywords `and` and `or` instead. Bitwise-AND and bitwise-OR need to stay separate because their evaluation strategies differ from `and` / `or`, but NOT is unified `not` because it doesn't have that problem; it only differs by type. `/\`, `\/`, and `><` seem like the right blend of uniqueness without being too ambiguous. Being symbols, they also have assignment versions `/\=`, `\/=`, and `><=`—useful for low-level bit manipulation. Keywords such as `band`, `bor`, or `bxor` wouldn't work here. How would their assignment operators look?

```
x band= 0b10  -- Did you mean `x: band = 0b10`?
x /\= 0b10    -- Clearly an assignment operation.
```

That's why only non-keyword operators are allowed to have assignment forms.

The operators are logically motivated — `/\` resembles ∧, `\/` resembles ∨, `><` looks like an X for XOR. The problem isn't that they're unmotivated, it's that the motivation isn't visible at a glance. Good syntax highlighting, inline hints, and a well-written guide would do more work here than any syntactic change. Bitwise operators resemble arrows which give them both visual and semantic unity.

This may break some of the usual habits of other languages, but that frees the usual symbols to do novel things. This is an experimental language, so it's not obligated to follow normal conventions.

### Pipelining `|>`

When placed at the start of a line in a block, the pipelining operator `|>` takes on a special meaning. It takes the value of the previous line and inserts it into the its line as `_`. This makes it easy to chain functions one after another in sequence.

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
|> fetchB(_)
|> fetchC(_)
|> print("{_}")
```

This makes it easier to see what gets called in what order. It reads like a list written in plain English:

- `fetchA`
- *then* `fetchB`
- *then* `fetchC`
- *then* `print`

### Operation chaining `[]`

**Infix operator** are operators who have both an `lhs` and `rhs`. For any non-void infix operator `op`, you can write it like `lhs op[rhs]` (an operator + an array literal). This is shorthand for writing `((lhs) op (rhs))` and has the same precedence like accessing with `.` or wrapping parentheses around the expression. This is primarily used for arrays, but has many potential use cases which can be explored.

Operation chaining has the same precedence has member accessing `.`. This allows you to treat any operator like a member access. The most useful use cases for this are for array indexing `#` and pipelining `|>`.

```
array#[0]#[1]                  -- → ( ( array # 0 ) # 1 )
fn1() |> [fn2(_)] |> [fn3(_)]  -- → ( ( fn1() |> fn2(_) ) |> fn3(_) ) → fn3( fn2( fn1() ) )
```

Pipelining can be particularly useful when combined with `[]` for inlining a variable that's repeated in an expression or method-chaining on an object with non-method functions.

```
-- Repeated value:
onePlusTwoCubed = (1+2) |> [_*_*_]
print("{onePlusTwoCubed}")       -- Prints "27"

-- Method chaining:
object.method1() |> [fn1(_)].method2() |> [fn2(_)]
-- Becomes…
fn2( fn1( object.method1() ).method2() )
```

Although arrays and tuples are treated differently, their syntaxes are analogous to each other—sit's just that one uses square brackets `[array]` and the other uses parentheses `(tuple)` or curly braces `{tuple}`. We can use this to our advantage to give operation chaining another feature: **tuple generation**. If the array literal after an operation has more than one indexes (e.g. `[expr, expr]`) or one or more named indexes (e.g. `[name: expr]`), then it will return a **tuple.** *For more information on tuples, see [Tuples](#Tuples).*

```
-- Apply `x ==` to each value, returns a tuple of bools.
-- Prefix `or` takes a tuple of bools and goes if any member is true.
if or x == [0, 1, 2, 3, 4]:
    print("x is 0 or 1 or 2 or 3 or 4")
```

You can create an named tuple instead of a position tuple too. Members can have names by adding the name and a colon `:` in front of each index. It must be a valid variable name. If you define a member with `name:`, you can access it in the next expression with `_.name`. 

```
-- Split `x` into 3 components and collect.
x = 1
y = x |> [
    a: _,
    b: _ + 1,
    c: _ + 2,
    d: _ + 3,
] |> [
    _.a * _.b * _.c + _.d
]    
```

`y` simplifies like this:

1. `x |> [ a: _, b: _ + 1, c: _ + 2, d: _ + 3 ] |> [ _.a * _.b * _.c + _.d ]`
2. `( a: x, b: x + 1, c: x + 2, d: x + 3 ) |> [ _.a * _.b * _.c + _.d ]`
3. `( x * (x + 1) * (x + 2) + (x + 3) )`
4. `( 1 * (1 + 1) * (1 + 2) + (1 + 3) )`
5. `( 1 * 2 * 3 + 4 )`
6. `( 10 )`

Tuples are transparent by default, and a tuple with only one position component and no named components is equal to just its position component. *For an explanation why, see [Tuples](#Tuples).* Semantically, this makes sese. If you have a parenthetical expressions, you essentially have a tuple of one. Some languages make a distinction between tuples and parenthetical expressions with syntax like `(10,)`, but this is not necessary in Mulang. 

- `(10) == (10).0`
- `(10).0 == 10`
- _Therefore:_ `(10) == 10`

**Mu the programming language is a novel and experimental one.** It has no obligation to follow normal conventions. Most other languages use `[]` by itself for indexing an array. While this is still possible in Mulang, operation chaining gives the coder a lot more options and ability to write expressive code. This is a break from the traditional style of programming, but it opens the door for more pragmatic and ergonomic code. This makes it instantly clear that when you see `#`, it's an array, just like `?` for options and `!` for results, staying consistent with itself rather than with other languages.

The transition from array indexing with `[]` in other languages to array indexing with `#[]` in Mulang is simple: add `#` before each square bracket. In most cases, the square brackets aren't even needed, making the `#` pattern more compacted than the `[]` one.

- `array[0][1]` → `array#[0]#[1]` → `array#0#1`

This frees `name[]` to take on a new meaning in Mulang. *For more on that, see [Meta Functions](#Meta-Functions).* *For more information on array indexes, see [Arrays](#Arrays).*

## Basic Bindings

There are two types of bindings: basic `=` and meta `::`. See [Meta Bindings](#Meta-Bindings-) below for details about `::`.

### Variable Declarations

Variables are declared with just the equals sign (`=`). Type is inferred, but can be declared with a colon (`:`). You can also use the `:=` operator instead to declare and infer the type at the same time. This is useful for shadowing mutable variables. *(See [Mutability](#Mutability).)* For now, just know that anytime you see `:` before `=`, *it always declares a new variable,* and if you see just `=`, *it's either declaring or mutating a variable.*

```
a = 1       -- Implicit declaration
b: int = 1  -- Explicit declaration with type.
c := 2      -- Explicit declaration but infer its type.
```

Variables are immutable, but declaring it again shadows it. Any subsequent `=` of an immutable variable is an implicit declaration. Redeclaring a variable with the same name is called **shadowing.** This makes Mulang flexible while still having the advantages of being statically typed.

```
a = 1
a = 2
a = 3
a = "hello"   -- The type of a shadowed variable doesn't have to match.
```

You can also shadow a variable using its previous value.

```
i = 0
i = i + 1     -- Sets new `i` based on old `i`.
i += 1        -- Does the same.
```

Using the single equal-sign is a void statement. If you use it within an expression and not on its own, it's a syntax error. This helps prevent the common bug of using `=` when you meant `==`. For inline binding, use `let`/`then` or `=>` instead. *(See [Inline Binding](#Inline-binding).)*

```
(-- Error:
if x = 0:
    print("x is 0")
--)
-- Do this instead:
if x == 0:
    print("x is 0")
```

Adding a new line and indentation after the `=` starts a block. The last expression evaluated in the block is the value of that variable.

```
lunch =
    if getDayOfWeek() == "Tuesday":
        "tacos"
    else:
        "sandwich"
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
doSomething()
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

`mu` variables cannot be shadowed by `=`. They can only be mutated. Assigning to the variable for the rest of the scope and any sub-scopes will mutate the variable—except for functions which always set a new variable unless its been explicitly captured. *(See [Capturing](#Capturing).)*

```
mu mut = 0
block:
    mut = 1
    print("{mut}") -- Prints "1", same outer `mut`.
print("{mut}")     -- Prints "1", `mut` was mutated.
cantSetMut() =
    mut = 2
cantSetMut()
print("{mut}")     -- Prints "1" again, cantSetMut didn't change it.
```

If you wish to shadow it, you can redeclare the variable with `: T =` or `:=`.

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

A reference type points to the same spot in memory that another variable has. It's like a lightweight version of a pointer. *(See [Pointers](#Pointers).)* Lifetimes are inferred via borrow-checking when the memory model is set to `Borrow`. *(See [Memory Models](#Memory-Models).)* The right hand side has be something stored in memory, i.e. not a constant. References are immutable by default unless declared with `ref mu`. The syntax follows the same pattern that `mu` does:

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

All components of a tuple should be referenced on the left. Use `_` to explicitly skip components. Also put `_` at the end of a named tuple to skip other keys.

```
(_, b, _) & {x, _} = (0, 1, 2, x: 3, y: 4)
```

When destructuring a type that isn't anonymous, the type can optionally be put after the parentheses/braces, otherwise it's automatically inferred. 

```
Thing :: {x: int, y: int}
thing = Thing(x: 1, y: 2)
{x, y}: Thing = thing         -- Split thing into its components.
{x, y} = thing                -- Or just infer the type.
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

Functions can also be declared with type `fn` to be set later. This type is called a **function pointer.** It lets you treat functions that same way you do with variables.

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

| Modifiers | What It Does | Mutability |
|:--|:--|:--|
| *nothing* | Copies value | Immutable *(in the function)* |
| `mu` | Copies value | Mutable *(in the function)* |
| `ref` | Passes reference | Immutable |
| `ref mu` | Passes reference | Mutable |

Another type of parameter is `out`. This is like `ref mu` but is treated like `unset[_]` at the start of the function. Use it to set a variable that hasn't been set yet. The parameter must not be `unset[_]` in any branch within the function. This means either setting it within the function or passing it to another function with an `out` parameter. This ensures that the variable is set after the function has been called. 

```
setInt(out i) =
    i = 3

x: mu int
setInt(x)
print("{x}")    -- Prints "3"
```

This works for mutable variables, but what if you wanted to make an immutable variable using `out`? You can write `out` while calling a function to declare an immutable variable in the current scope. This has the same rules that `=>` does. *(See [Inline Binding](#Inline-Binding).)

```
setInt(out n)   -- Declare a new variable `n` that gets set by `setInt`.
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

### Inline Binding

You can also bind variables within an expression using `let`/`then` and `as`. `let`/`then` is used for a single expression, whereas `as` binds for the rest of the scope.

#### `let`/`then`

```
squared = let x = getSomething() then x * x

loop if next() => val != None:
    print("{val}")
```

All variables after `let` are new declarations, much like variables in a non-capturing function; they cannot mutate any existing variables. In other words, they have an implicit `:=` even when they use `=`. 

```mu
mu x = 0
y = let x = 1 then x + x   -- Or also `y = let x := 1 then x + x`, the same thing.
print("x = {x}, y = {y}")  -- Prints "x = 0, y = 2".
```

Multiple variables can also be declared at once. You must surround the variables with parentheses like `let () then`. Each variable is separated by commas. 

```
sum = let (x = 1, y = 2) then x + y
```

#### Inline Assignment Operator `=>`

The inline assignment operator `=>` sets a value within an expression as a variable within a block's scope. It returns the value of the left-hand side, the right-hand side should be a valid variable name. Like `let`, it's an explicit declaration like `:=`, so it can't mutate. Note that this is slightly different from but consistent with the `as` that's used for aliasing. *`=>` the operator* is only used in **expressions**; meanwhile, *`as` for aliases* is only used in **patterns.** *For information on how patterns work, see [Destructuring](#Destructuring).*

The simplest use case for `=>` is to pair it with a `loop if` loop to get a value on each iteration.

```
loop if next() => val != None:
    print("{val}")
```

This reads like an English sentence: "Loop if next value is not none…"

*Note that `let` wouldn't work here because it only works on a single expression.*

```
loop if let value == getValue() then value != None:
    print("{value}")   -- Error: `value` is not defined.
```

Another use case for `=>` is to use while method chaining to get a result of one of the methods. Note that `=>` has the same order of operations as `.` when using `[]`. This makes it possible to method chain without adding parentheses around it. `a() =>[b].c` is the same as `(a() => b).c`. *(See [Operation Chaining](#Operation-Chaining).)*

```
(object.method1()
    .method2() =>[result]
    .method3())
print("{result}")
```

Type must be inferred. This is to avoid using `:` in an expression, which could be mistaken for the key name of a tuple or a declaration.

```
(-- Syntax Error:
action(get() => result: int) -- Is this a named component? Did you forget the commas?
--)
```

However, mutability can be set with `=> mu` / `=>[mu]`.

```
(object.method1()
    .method2() =>[mu result]   -- New mutable variable.
    .method3())
result += 1
print("{result}")
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
match value then | True:
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

There are several number types. More may be added in the future, but for now we'll focus on the 3 main types. For any of these types, you can add a suffix to explicitly declare a number with a particular type. However, suffixes are disabled if the number already has letters in it. 

| Type | Meaning | Suffix | Examples |
|:--|:--|:--|:--|
| `int` | signed integer | `i` | `1`, `2i`, `10i`, `0xabcdef` |
| `uint` | unsigned integer | `u` | `1`, `0u`, `5u`, `0x10ff` |
| `float` | floating ploint number | `f` | `1.0`, `1.5f`, `100f`, `2.0e100` |

A number literal starts with a digit `0123456789` followed by zero or more other digits, letters, or underscores `_`. All number literals are case-insensitive, so `1.0f == 1.0F`.

The sign is considered an operator and not a part of the constant itself. This gets automatically calculated in constant expressions at compile-time so that it seems like it's a part of the constant.

- `- 1` → `minus` + `1` → `signflip(1)` → `-1`

The exception to this rule is in scientific notation: the sign after `e` is a part of the number itself. There must not be a space between `e` and the sign `-`/`+`. The sign is optional for positive exponent values.

- `2e-100`, `5.0e+128`, `6.8e-128`, `10e3`

You can place underscores `_` anywhere in a number to break it up into segments. This doesn't change the value.

- `1_234`, `1_000_000`, `0b1111_0000`, `0xab_cd_ef`

Leading zeros are allowed also and don't affect the value. Unless there's a base letter, it's still in base 10, unlike in most C languages where a leading 0 switches to base 8.

- `001`, `009`, `000100`, `000u`, `010.0f`

Changing the base involves adding a `0` and a then the letters `b`, `o`, or `x` to the start of the number.

| Prefix | Base | Valid Digts |
|:-:|:-:|:-:|
| `0b` | 2 | `01` |
| `0o` | 8 | `01234567` |
| `0x` | 16 | `0123456789abcdef` (case insensitive) |

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

Characters can be escaped with a backslash `\` in the string. Some letters have special values like `\t` for tabs, `\n` for new lines, etc. All the standard stuff you would expect. 

```
apostrophe = '\''
tab = '\t'
newLine = '\n'
nullChar = '\0'
unicode = '\uFFFF'
```

#### Strings

Strings are marked with quotation marks (`"_"`) *(also called double quotes)* and can be formatted with curly braces (`{expr}`) in the string. Use a backslash to write a literal opening curly brace (`\{`). Note that string insertion and named tuples both use curly braces. This shouldn't be an issue though since they're used in different contexts. Expression are implicitly converted to strings, so using `str()` or `~` isn't necessary.

```
name = "world"  
hello = "Hello, {name}!"
helloEscaped = "Hello, \{name}!"
```

Subsequent string literals will automatically concatenate, and the `++` operator can be used to concatenate non-literal strings.

```
str1 = "This" " string"
str2 = " is broken"
str3 = str1 ++ str2 ++ " into multiple parts."
print(str3)
-- Prints "This string is broken into multiple parts."
```

You can also write multi-line strings with `"""`. A common issue in programming languages is how to fix the issue of leading whitespace in a multi-line string. Mulang uses significant whitespace, so unindenting the string wouldn't work. We don't want all the leading whitespace to be in the string, but how do we solve this? Whitespace is trimmed based on the positions of the last `"""`. Anything before it is automatically trimmed. Much like blocks, having too little indentation is a syntax error inside multi-line strings. This helps keep things readable and consistent and solves the whitespace issue inside strings.

```
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

You can write a raw string with `''…''` (two apostrophes). Although apostrophes `'` are used for chars, an empty char isn't possible since the default char is written `'\0'` (null character). This is a common practice in programming languages where `"` mark formattable strings and `'` mark raw strings, so this should be easy to understand for any programmer. When you write `''`, every character after it (including whitespace and indentation) is in the string until the closing `''`. Escaping with backslashes `\` and insertion with curly braces `{}` are ignored.

```
rawString = ''It's okay to put an apostrophe (') in the string.''
filePath = ''C:\files\on\windows.txt''
template = ''Insert here → {{variable}}''
```

To escape `''` within a raw string, add a dollar sign `$` before and after the apostrophes. The number of `$`s must match to close the raw string.

```
-- Add a `$` to escape the `''` within the string.
bigDocument = $''
    This  '
   is   ''      ''
  all       ''
  in  '         '
   a     '''' '
    string
''$
-- Matching number of `$` closes the string.

-- `$$''` to escape the `''$` within the string.
nestedDocument = $$''
bigDocument = $''
    This  '
   is   ''      ''
  all       ''
  in  '         '
   a     '''' '
    string
''$                
''$$
-- `''$$` closes the matching `$$''`.
```

While this is possible, this is not recommended for the sake of legibility. For anything more complicated, it's recommended to save the string in a document and open the file inside your program, which will be handled by a separate library and lies outside the scope of this document.

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
|:--|:-:|:-:|
| **Results** | `T!` | `x!` |
| **Options** | `T?` | `x?` |
| **Pointers** | `T^` | `x^` |
| **Arrays** | `T#N` | `x#n` |

Other languages use `[]` for indexing, but that has another meaning in Mulang. Instead, you can use operation chaining to do the same thing like `[]` in other languages. *(See [Operation Chaining](#Operation-Chaining).)*

```
item = doubleArray#[1]#[0]  -- The 2nd row, 1st column
print("{item}")             -- Prints "3"
```

Habits can be hard to break. Many programmers have i internalized the `array[i]` format to mean "array index". To help with that, Mu allows the familiar `array[i]` format to be the same as `array#[i]`, but special care has to be taken into consideration to make sure it doesn't clash with [meta functions](#Meta-Functions). This rules out any tokens that looks like `name[]` or `name[a, b]` since array indexes normally only take one argument in other languages.

```
metaOrArray[0]   -- Is this an array or a meta function?
```

If Mulang thinks something might be an old-fashioned array index access like `metaOrArray[0]`, it will check the context to determine what the type of `metaOrArray` is to resolve it. If it's an array in this context, it will automatically treat it the same as `metaOrArray#[0]` without any fuss. This lets Mulang have the familiar syntax while also having the power and potential that comes with other features like *operation chaining* and *meta functions.*

You may want to do an operation with an array literal on the right hand side of an operation. In that case, you can either store the array in a variable or surround it in parentheses `()`.

```
print("{ 0 in ([1, 2, 3, 4]) }")  -- Prints "False"
a = [0, -1, -2]
print("{ 0 in a }")               -- Prints "True"
```

If the left hand side of `++` isn't an array, it will automatically create one. A non-array on the right-hand side will push it to the end of the resulting array. In this way, you can abandon square brackets entirely and just use `++`.

```
a = 1 ++ 2 ++ 3 ++ 4  -- == [1, 2, 3, 4]
b = 0 ++ a ++ 5       -- == [0, 1, 2, 3, 4, 5]
c = b ++ 6 ++ 7 ++ 8  -- == [0, 1, 2, 3, 4, 5, 6, 7, 8]
```

Another use for `++` is to spread an array into another array. This was chosen because it's also used for concatenation, giving the two operations an obvious connection. Some languages use `...`, but this visually conflicts with the range `..` operator. `++` conveys the meaning better without any issues. 

```
a = [1, 2, 3]
b = [0, ++a, 4]    -- == [0, 1, 2, 3, 4]
c = a ++ b         -- == [1, 2, 3, 0, 1, 2, 3, 4]
```

This means that you can either spread or concatenate just by putting the right-hand side inside or outside of the array literal.

```
a = [ 4, 5 ]
b = [ 1, 2, 3 ] ++a
c = [ 1, 2, 3, ++a ]
b == c             -- True
```

Some languages use `...` for the spread operator, but this visually clashes with the range operator `..`. *What if you wanted to spread a range into an array?*

```
tenDigits = [...0..10 ]
tenDigits = [ ++0..10 ]
```

The second makes it clear that it's two operations: range (`..`) and spread (`++`). 

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

If the key type is `str` and a key is a valid variable name, then the square brackets before `:` can be omitted.

```
dict: int#str = [
    a: 1,
    b: 2,
    c: 3,
    ["invalid name"]: 127,
]
print("{ dict#["b"] }")   -- Prints "2"
```

How dictionaries are implemented is yet to be determined. This document only focuses on their syntax in the language. 

#### Pointers

Although most things can be achieved without manual manipulation of pointers, some low level code requires it. Raw pointers use the type `ptr`. This represents an opaque pointer where the type it represents is unknown. It's ideal for FFI where you need to pass a pointer a around and let an external library handle it. 

```
result: ptr = ExternalLib.getSomething()
ExternalLib.doSomethingWith(result)
```

You can manually check if the pointer is `Null` and give a help error message in scripts.

```
result: ptr = ExternalLib.getSomething()
if result == Null:
    print("No result found")
    raise NotFound
```

A standard library will be made to safely handle pointer dereferencing and do pointer arithmetic, but that is outside the scope of this document. Here is an example of how it might work:

```
mu x = 0             -- Create a local mutable variable.
xPtr = getMuPtr(x)?  -- Map `Null` to a option type, branch if it's `None`, return `Some(ptr)` if it's not and unwrap it with `?`.
xPtr.set(1)!         -- Safely set the pointer and branch if there's an error.
print("{x}")         -- "1", the pointer successfully mutated `x`.
```

Sometimes, it's necessary to dig deep into the unsafe territory. The `^` is the symbol associated with pointers, analogues to `?` for options, `!` for results, and `#` for arrays. It can be used in type notation, but it's also the operator to dereference a pointer. Thy type must be known at compile-time. Dereferencing an opaque pointer `ptr` is a compile-time error. In the type notation, `T^` prevents the pointer from mutating its memory or `T^mu` allows mutation with `^ =` (dereference + assignment). 

```
mu x = 0                 -- `ptr` type takes a reference and creates a generic pointer.
xPtr: int^mu = ~ptr(x)   -- Convert `ptr` to `int^mu`, type is known.
xPtr^ = 1                -- Mutate the memory.
print("{xPtr^}")         -- Prints "1".
print("{x}")             -- Prints "1".
```

Please know that this is only an example and is subject to change. Some more testing is required to figure out the best way to handle pointers. Consider this a work in progress.

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

#### `if or` / `if and`

The alternative is to put `or`/`and` immediately after `if`, it will check a tuple of `bool` values instead of a single `bool` value.

```
if or (True, False, False):
    print("One of these is true.")

if and (True, True, True):
    print("All of these are true.")
```

If you pass an empty tuple, the condition will always be false.

```
if or ():
    print("This will never run")
else if and ():
    print("This will never run either")
```

Put `not` before the `or`/`and` to invert the condition.

```
if not or (False, False, False):
    print("None of these are true.")

if not and (False, True, True):
    print("One of these is not true.")
```

This can be pained with `==[]` to check multiple values at once. *(See [Operation Chaining](#Operation-Chaining).)*

```
x = "foo"

if or x ==["foo", "bar"]:
    print("foobar")
```

The condition doesn't have to be a multi-component tuple though. You can just leave it with one match and insert more when you need them.

```
if or status ==[404]:    -- Add other status numbers when you need to.
    print("error status: {status]")
```

Note that `if and`/`if or`/`if not and`/`if not or` isn't a special syntax. It's that the `or`/`and` prefix operators compress a tuple of bools to a single bool. 

```
x = or (True, False, True)
print("{x}")              -- Prints "True"
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

Like `block`, you can give a label after `loop` to use instead.

```
loop label:
    if cond:
        break label
```

`loop` has several varients to change what condition will break. The pattern is `loop keyword`. A label can be optionally put between with `loop label keyword`. This makes it clear that whenever you see `loop`, then you have a loop. A programmer briefly can see the word `loop` and think *"Ah, there's a loop here."* 

#### `loop for` / `in`

Iterates through an array or iterator.

```
loop for var in expr:
    body
```

If in-lined, it returns an iterator `iter[T]` which is lazily executed. It won't activate until it's collected in an array with `++` or passed into another `loop for _ in`. 

```
iterator = loop for x in list then x * 2
list = [++iterator]
```

The items can also be destructured. *(See [Destructuring](#Destructuring).)*

```
listOfTuples = [(1, '2', "three"), (4, '5', "six")]

loop for (x, y, z) in listOfTuples:
    print("{x}, {y}, {z}")
```

Destructuring also works.

```
loop for {x, y}: Thing in getThings():
    print("Thing\{x={x}, y={y}}")
```

For an async iterator or an array of async types, you can use `loop for await` to automatically wait for each item in sequential order. *(See [`await`](#await).)*

```
loop for await x in asyncIter():
    print("{x}")
```

#### `loop if`

Repeats a block of code until the condition is `True`.

```
loop if cond:
    body
```

The same things that apply to `if` apply to `loop if` like the `or == []` pattern.

```
mu x = 0
loop if or x ==[0, 2, 3, 4]:
    print("{x}")
    x = randInt(-1, 4)
```

The `else` after a `loop if` block will run if the loop never ran even once. This is analogous to `if` / `else`. 

```
x = False
loop if x:
    pass
else:
    print("The loop failed")
```

Some programming languages let you do something like `while (value = getValue()) {body}`. Mulang doesn't allow this because `=` is a void statement. Instead, you can use `=>` to achieve the same thing but more explicitly.

```
loop if None != (getValue() => value):
    print("value = {value}")
```

#### `loop` / `until if`

Postpone the conditional check until the end of the loop. Add `until if` after a `loop` block to create a condition that will break the loop. Unlike `loop if`, this will break when the condition is `True`, analogous to `if cond then break` inside the body. This would be a `do { body } while (!cond);` loop in other languages like C. The reason it doesn't use `while` is because that would get confused for the start of a block, and `do` already has a different meaning in Mulang, so `loop` and `until` were chosen instead. This makes it clear that `loop` and `until` are semantically connected since `until` can only come after `loop`.

```
loop:
    body
    if cond:
        break

-- Transforms into:

loop:
    body
until if cond
```

The loop will run at least once before checking the condition.

`loop`/`until` is arguably a better choice than `do`/`while` because it's more descriptive of what it's doing. *"Loop until if this is true."* You can visually see that the condition is true after the `loop`/`until` block is finished, whereas you would need to invert the condition in your head for a `do`/`while` loop. It acts like a blockade to wait until a certain condition is true.

```
mu i = 0
loop:
    i += 1
until if i >= 10
-- i is >=10 at this point
```

#### `break` / `continue`

Controls the iteration of any loop type mentioned. `break` exits out of the loop, and `continue` skips to the next iteration. This is only allowed in block-level loops. Inlined `for` loops are not allowed to skip iterations. This keeps the mapping between iterators 1-to-1.

After `break` or `continue`, you can give it a label to break. It must be the name of a `block`/`loop` in the same scope or higher. 

```
loop label:
    loop:
        print("Double loop! How do I break?")
        break label

print("I'm free!")
```

```
loop outerFor for x in 0..100:      -- Label this block `outerFor`.
    loop innerFor for y in 0..100:  -- Label this block `innerFor`.
        prod = x * y
        print("{x} * {y} == {prod}")
        if prod >= 100:
            break innerFor          -- Break inner loop, continue outer loop.
        if prod == 77:
            break outerFor          -- Break outer loop, loop ends.
```

#### `match`

Enum/exception branching. Exhaustive by default. `| _:` for the default case.

```
match expr then | ptrn:
    body
| ptrn:
    body
    –
| _:
    body
```

Each pattern starts with `|`. This was chosen because pattern matching is a core feature in Mulang and a core identity of functional programming. This keeps it much briefer than the usual `switch`/`case` statement, closer to the pattern matching found in functional programming languages. 

Its syntax is a bit different than most blocks. You start with `match expr then` with no colon. Each case starts with `|`. This is a special case since each pattern starts a block. The colons are put at the end of the pattern on each case, with each case being its own block. 

The patterns map to the type passed in after `match`, so you only need to reference the members of that type in each pattern.

```
match choice then | First: -- Each pattern case starts its on block.
    print("First")         -- Ident for the new block.
| Second(x):               -- Continue this for each case.
    print("Second({x})")   -- ……
| Third{val}:              -- ……
    print("Third \{ val={val} }")
                           -- All choices were exhausted, so no `| _:` is necessary.
```

The first case can also be put on the next like this, making it easy to line up all the patterns:

```
match choice then
| First:
    print("First")
| Second(x):
    print("Second({x})")
| Third{val}:
    print("Third \{ val={val} }")
```

The inline form switches keeps `:` after patterns. This makes it easier to read and take up less space. Patterns are usually words, so you can visually sequence it into pattern/expression pairs: `| ptrn: expr | ptrn: expr | ptrn: expr` etc.. Other symbols like `=>` wouldn't work because it would clash with operators. `:` also follows the `key: value` pattern that tuples and dictionaries use. 

```
tuple = (a: 1, b: 2)
dict = [x: 3, y: 4]
restult = match x then | Ptrn1: 5 | Ptrn2: 6 | _: 7
```

```
message = match e then | OpenError{filename}: "Open error: {filename}" | _: "Unknown error"
```

You can have multiple patterns match to one case. If any of the patterns destructure with a variable, the same variable name and type must be in all patterns. If not, use a wildcard `_` in each pattern or omit the tuples part entirely to disable destructuring.

```
match choice then
| First:
    print("First")
| Second(val) | Third{val}:                  -- `val` must be in all patterns
    print("Second or Third, val={val}")

match choice then | First | Second | Third:  -- `First` doesn't have any values, so destructuring must be disabled.
    print("First, Second, or Third")
```

Unlike old-style `switch` blocks, each pattern block breaks automatically without the need for a `break`. Instead you can use the keyword `fallthrough` to go to the next case. The next case can't have any destructured values if `fallthrough` is used.

```
match choice then
| First:
    print("First!")
    fallthrough
| Second(_):              -- Destructuring is disabled.
    print("And second!")
    fallthrough
| Third:                  -- `(_)` is optional.
    print("And third!")
```

#### `match` + `if`

Inside any pattern, you can add `if` to conditionally check the variable and only match when the condition is met. Only the first condition to pass will go *(unless there's a `fallthrough`).*

```
match choice then
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
match choice then | Third{val as x if x == 1}:
    print("Third.val is one: {x}")
| _:
    print("Either Third.val isn't one or choice isn't Third.")
```

#### `case`

This does the same job that a single `|` in a `match` block does. Most languages use `case` for the `switch`/`match` block, but since Mulang uses `|`, that frees up the keyword `case` to be used for other useful patterns.

`case` / `else` extracts an enum and binds its value to the scope. You must have `else` at the end, and the block must break out of the scope like with `break`, `continue`, `return`, `raise`, etc.. This is analogous to the `let pattern(x) = value else {}` pattern found in other languages.

```
block label:
    case Pattern(x) = getStuff() else:
        -- `x` is unset here.
        break label
    -- `x` is set here.
    print("{x}")

getStuffPlease() =
    case Pattern(x) = getStuff() else:
        raise Error("It doesn't match")
    print("{x}")

mightGetSomething() =
    case Some(x: int) = getSomething() else return None
    Some(x + 1)
```

Like with `match`, you can also conditionally check with `if`. This works for all other pattern matching blocks too.

```
getStuffPlease() =
    case Pattern(x if x >= 100) = getStuff() else:
        raise Error("It either doesn't match or x ({x}) is less than 100")
    print("{x} is definitely 100 or more.")
```

#### `when`

Combines the conditional branching of `if` with pattern matching of `case`. Useful if you want to destructure a single case of a sum type. This must be a pattern that matches the type of the value after `=`.

```
when Pattern(x) = getStuff():
    print("value is {x}")
else:
    print("value doesn't match")

something = when Some(x) = getSomething() then x else "fallback"
```

This makes it clear that you are pattern matching instead of checking a `bool` value. We want to avoid using `=` in an `if` block because it could lead to potential ambiguity issues.

```
x = 0

if Some(x) = getStuff():  -- Did you mean `==`? Are you creating a new `x` or wrapping the existing `x`?
```

Multiple patterns can be checked for at once with `|` like with a `match` block. All patterns must go on the left of the `=` sign. Any destructured variables must match in name and type.

```
when ResultType1(val: int) | ResultType2{data as val: int} = getResult():
    print("val = {val}")
```

`x if` is also available like before and follows the same rules. You can iteratively match nested enums. This means there's no need for `x when` because it would be redundant.

```
when Some(Some(Some(Some(x if x >= 0)))) = nestedOpt:
    print("Phew! That was a lot of unwrapping for {x}!")
else:
    print("Either none of those nested options matched or x is negative.")
```

#### `loop when`

Are you sensing a pattern? We have `if` and `when`, so that means we also get `loop if` and… `loop when`! This will loop until the pattern breaks. In some cases, this might be more desirable than the `loop if` + `=>` format.

```
loop when Some(x) = nextValue():
    print("value is {x}")
```

#### `loop` / `until when`

Like how `if` has `when`, `until if` has `until when`. This creates a new variable below the loop when the condition is met. This is because it appears at the end of the block instead of the top, so it won't be set *until* it matches which itself is the condition for the loop to break. This can be useful if you want to repeatedly call a function *until* you get something. This means that the loop cannot conditionally break inside the body because then the variable would not be set when it breaks. This ensures that the destructured variables are defined after the loop finishes. The loop runs until the pattern matches, and when it does, the matched value is your result. The scoping rule follows directly from the semantics. The variable is visible *below* the `when`. You can't use the value inside the `loop` body since `until when` is a termination condition, so there's only one place the binding could go — outside. *The scope of bindings mirrors the position of the pattern.*

```
value: mu int?

loop:
    value = getValue()             -- Try to get something.
until when Some(x: int) = value    -- Loop again if it fails. Exit here if it works.
                                   -- Below the loop, `value` is guaranteed to be `Some(x)`, and `x` is guaranteed to be set.
print("value = {x}")               -- Print the unwrapped value.
```

The reason it can't break is because then the condition wouldn't be guaranteed, and any destructured variables would not be set.

```
loop:
    if someCondition:
        break                      -- Compile-time error: cannot break inside `loop`/`until when`
until when Some(x) = getValue()
```

If you wish to use `break`, you should disable destructuring so that there's no need for a guarantee.

```
loop:
    if someCondition:
        break                     -- Okay!
until when Some(_) = getValue()   -- No new variables. This is fine now. 
```

#### `try` / `except`

Result types are unwrapped with an exclamation mark (`!`) within a try block.

```
safeResult = try divide(1, 0)! except _ then 0.0

try:
    risky()!
except e:
    print("{e}")
```

Pattern matching works in the `except` clause like in `case`. If a pattern is missing, the exception is raised to the next block above it. The type of exception in the last `except e then` block is all the possible exceptions minus the caught exceptions. 

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
    for i in 0..n:
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

Both `yield` and `await` can be used together in an `iter[async[_]]` type. The type suggests it—each yield is of type `async[_]`. Use `for await` to wait for each async value to resolve in sequential order.

```
asyncIterFn(n): iter[async[int]] =
    for i in 0..n:
        val = await fetch(i)
        yield val

asyncCollect(n): async[int#] =
    ret: mu int# = []
    for await x in asyncIterFn(n):
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
- `loop if` + `loop when` — loops *when* the condition is true.
- `loop` / `until` — loops *until* the condition is true.
- `loop for` — loops through an iterator or array.

## Error Handling

- `expr!` — propagate an error upward (Rust/Swift style).
- Exceptions are sum types; the compiler unions all possible exception types from every `!` site in a `try` block.
- `try` / `except` can be an expression or a block and must unify return types (like `if` / `else`).
- Pattern matching on errors works with `match` or `ifcase Pattern(x) = value`.

## Pattern Matching and Destructuring

- Full `match` (exhaustive unless `| _:` is present).
- `when` — like Rust's `if let`.
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
match a then
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

To implement from a prototype, add the proto's name in the square brackets. Each implementation gets their own `impl[]` block. 

```
MyStruct :: impl[MyPrototype] =
    speak(self) =
        "I am a MyStruct \{ name={self.name}, value={self.value} }"

MyEnum :: impl[MyPrototype] =
    speak(self) =
        match self then
        | First:
            "I am a MyEnum of First"
        | Second(x):
            "I am a MyEnum of Second({x})"
        | Third{val}:
            "I am a MyEnum of Third \{ val={val} }"
```

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

List[T, N] :: impl[] =
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

Modules are named with the keyword `mod` near the top before anything is defined. This is the name you'll use when importing your module. **There can only be one `mod` declaration per file.** Multiple `mod` declarations is a syntax error.  

```
import somewhere{thing}

mod myModule

addThing(x) = x + thing
```

In this example, you would import `addThing` like this:

```
import myModule.addThing
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

Built-in decorators demonstrated so far include `@capture`, `@opaque`, and `@memory`. More planned for the future.

```
@memory(Manual) -- Call it like a function to pass a variable.
mod myModule

@opaque         -- No function needed if there are no arguments.
Thing :: struct =
    value: int

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

1. `and`
2. `as`
3. `await`
4. `break`
5. `case`
6. `continue`
7. `defer`
8. `do`
9. `else`
10. `end`
11. `enum`
12. `except`
13. `fallthrough`
14. `fn`
15. `for`
16. `if`
17. `impl`
18. `import`
19. `inherit`
20. `in`
21. `let`
22. `loop`
23. `match`
24. `mod`
25. `mu`
26. `not`
27. `opt`
28. `or`
29. `out`
30. `pass`
31. `proto`
32. `raise`
33. `ref`
34. `return`
35. `self`
36. `struct`
37. `then`
38. `try`
39. `until`
40. `when`
41. `where`
42. `yield`

*NOTE: Built-in types, values, functions, and decorators like `int`, `True`, `Some`, `default`, `print`, `@memory` etc. are not considered keywords.*

---

*This document captures the current state of the Mulang design. The language is still evolving.*
