# Mu Language Syntax and Semantics Reference

*Version 0.1 (Draft)*

The Mu programming language or *Mulang* is a general-purpose, multi-paradigm programming language with significant whitespace. It targets Python developers who need C-level performance all within the same language. It is expression-oriented where possible and provides explicit control over evaluation strategy, memory model, and error handling.

Mulang will be both a compiled and an interpreted language. Unlike Python, you won't need to use another language like C to increase performance. You can instead compile some of it and then call it as a shared library all within the same language. Some use cases for this are AI, systems programming, and game development.

This document will focus on the language itself. Some features may come in a standard library which will not be discussed here.

---

## Lexical Conventions

- **Indentation**: Significant whitespace (4 spaces recommended). 
- **Statements**: Each statement is divided by new lines. Semi-colons (;) can also be used. 
- **Comments**: Double dash (`--`) to end-of-line. Block comments start with `(--` and end with `--)`.
- **String literals**: `"..."` with interpolation with `{expr}` inside. To write a literal curly brace (`{`), escape it with a backslash `\{`. 

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

Almost everything is an expression. Some statements can be either inline or block depending on the presence of a colon (`:`) or equals sign (`=`) followed by a new-line.

Parts of an expression are divided into 4 catagories:

1. __Words:__ variable names: `x`, `PI; numeric constants: `1`, `3.14`, `0xABCDEF`;
2. __Char/String Literals:__ things surrounded in quotes: `'a'`, `"foo"`, `"""big string"""`, `#''raw string''#`
3. __Delimiters:__ commas `,` for tuples and arrays; `;` for subsequent expressions
4. __Symbols:__ operators and anything with these characters: `~!@#$%^&*-+=\|:<.>/?` *but excluding the symbol for a comment `--`*
5. __Bracket Expressions:__ anything in parentheses `()`, square brackets `[]`, or curly braces `{}`; *note: the term __bracket__ means any of these characters `()[]{}`, and the term __square bracket__ means just these characters `[]`*
6. __Whitespace:__ spaces, tabs `\t`, new lines `\n`/`\r`, etc.

Whitespace is significant. **Indentation** is used to mark when blocks start and end. Indentation can be any whitespace character except for new-lines. Inside any block, expressions should start with the same indentation through out. This includes the number and value of indentations. You could mix tabs and spaces, but each subsequent expression in a block must have the same number of tabs and spaces and in the same order. It's recommended to use **4 spaces** per indentation, adding 4 more for each block.

A single new-lines can be either Carriage Return (`\r`), Line Feed (`\n`), or both (`\r\n`). A new-line marks the end of an expression *unless...*

1. The line contains an open bracket or quotation mark.
2. The next line immediately after it and after indentation starts with a *symbol,* known as **expression splitting.**

If one of these rules is broken, *then indentation is ignored.*

Examples:

```
-- This is two expressions:
expr1
expr2

-- This is one expression, ignores indentation until closing bracket:
(expr3p1
    expr3p2
   expr3p3
      expr3p4)

-- This is one expression, ignores indentation for each line that starts with a symbol:
expr4p1
   + expr4p2
  + expr4p3
      + expr4p4

-- These are seperate expressions, 2 new lines disable expression splitting:
expr5

+ expr6

+ expr7

+ expr8
```

**Expression splitting** is when you split an expression into multiple parts. Each line should start with a **symbol.** The common use case for this is for *method chaining.*

```
object.method1()
      .method2()
      .method3()
      .method4()
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

`do` signals to switch to block mode. After the `do` comes a block keyword, and then there needs to be a `:` (or `=` for functions) at the end of the line. When the keyword `end` appears, that's the signal to switch back to inline mode. The expression between `do` and `end` is known as an **inline-block expression.**

```
(do block:       -- `do` --> Switch to block mode.
    expr         -- Inside the inline-block expression, significant whitepace here.
end)             -- `end` --> Switch to inline mode.

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

Not all blocks need to be wrapped in `do`...`end` all of the time. Only when they need to switch from inline mode to block mode. Most blocks in Mulang can be **inlined** though. Some are based on whether they use a `:` or not. If a block has a *subject* component, then the `:` goes after the subject in block mode but is swapped for the keyword `then` in inline mode. *(See [Control Flow](#Control-Flow) for more details.)*

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

The philosophy of Mulang is that symbols should be easy to recognize and understand. Generally, keywords are preferred over symbols to make it easier to read, but symbols can also help make code easier to both write and read at times. Many are semantically grouped based on their contextual usage, for example `*` and `/` relate to math, `%` relates to bitwise operators, `?` relates to options, `&` relates to tuples, etc.. There's also a common pattern where repeating an operator gives you a more technical or complex version of that operator, for example: `+` addition vs `++` concatenation, `*` multiplication vs `**` exponentiation, `/` division vs `//` floor division, and `%` standard modulo vs `%%` floor division modulo.

Mulang will check any combination of symbols greedily until the next space or word except for the reserved comment token (`--`). For example `++` would be different from `+ +`, but `++--` would be considered `++` and a comment `--`. Spaces are required between multiple symbolic operators, much like how spaces are required between words. Symbol characters include any ASCII character that isn't alphanumeric, whitespace, quotation marks, delimiters, or brackets&mdash;in other words these symbols: `~!@#$%^&*-+=|\:<.>/?`.

There may be operator overloading in the future. Even if an operator could be broken into smaller parts, it's considered a unique operator even though it may not be defined&mdash;just like how you can technically write a method that may not be defined, but they're both considered errors. Because of that, long sequences of operators need to be broken up into their separate symbols. From example `a???b`, the compiler will see `???` as one symbol instead of as `?` and `??`. You would need to put at least one space here `a? ??b` to make sure it's clear that it's 2 operators and not one. This not only helps the parser but also helps the coder when reading the code. The exception to this rule is `--` which is **always** a comment no matter what.

__Arithmetic:__

- `lhs + rhs` &mdash; addition
- `lhs - rhs` &mdash; subtraction
- `+ rhs` &mdash; keeps the sign the same; *so does nothing*
- `- rhs` &mdash; sign-flip
- `lhs * rhs` &mdash; multiplication
- `lhs / rhs` &mdash; exact division; *returns a floating point number*
- `lhs // rhs` &mdash; floored division (rounded down); *returns an integer*
- `lhs % rhs` &mdash; modulo (sign matches `lhs`)
- `lhs %% rhs` &mdash; floor division modulo (sign matches `rhs`); *result is between the range `[0, rhs)` if `rhs` is positive or `(rhs, 0]` if `rhs` is negative*
- `lhs ** rhs` &mdash; exponential

__Comparison:__

- `lhs == rhs` &mdash; equality
- `lhs != rhs` &mdash; inequality
- `lhs > rhs` &mdash; greater than
- `lhs < rhs` &mdash; less than
- `lhs >= rhs` &mdash; greater than or equals to
- `lhs <= rhs` &mdash; less than or equals to

__Boolean:__

- `lhs and rhs` &mdash; false if any are false
- `lhs or rhs` &mdash; true if any are true
- `not rhs` &mdash; inverts a boolean
- `and rhs` &mdash; do `and` on each component of a tuple: `and (True, False, True)` -> `True and False and True` -> `False
- `or rhs` &mdash; do `or` on each component of a tuple: `or (True, False, True)` -> `True or False or True` -> `True`

__Bitwise:__

- `lhs %& rhs` &mdash; bitwise `AND`
- `lhs %| rhs` &mdash; bitwise `OR`
- `lhs %^ rhs` &mdash; bitwise `XOR`
- `%~ rhs` &mdash; bitwise `NOT`
- `lhs << rhs` &mdash; bitshift left
- `lhs >> rhs` &mdash; bitshift right

*NOTE: `%` is reused for bitwise operators to keep with the pattern of `%` as symbol for computer-related math. Most people don't learn about the modulo operator `%` unless they learn computer science. Therefore, it makes sense to associate it with other computer-related math operations as well. You may think that this is overloading the meaning of `%`, but the trade-off is that this makes it semantically clear that when you see `%`, some kind of computer-science related math is going on. In general, bitwise operations are only used for very low-level programming, so this will not be something that new users have to worry about.*

__Arrays:__

- `lhs # rhs` &mdash; get an item from `lhs` at an index `rhs` (index starting at 0)
- `lhs ++ rhs` &mdash; concatenation, returns a new array
- `++ rhs` &mdash; spread an array or iterator into an array or positional tuple
- `lhs .. rhs` &mdash; creates an iterator that starts at the left value and ends just before the right value (exclusive)
- `lhs ..= rhs` &mdash; creates an iterator that starts at the left value and ends with the right value (inclusive)
- `lhs in rhs` &mdash; checks if an item exists in an array, returns a boolean

__Tuples:__

- `lhs . rhs` &mdash; access a member/component
- `lhs & rhs` &mdash; combine two tuples into one
- `& rhs` &mdash; spread a tuple into another tuple

__Options:__

- `lhs ?` &mdash; returns the `Some` value if it's not `None`, otherwise propagate to the nearest `opt` keyword *(see [`opt` block](#opt))*
- `lhs ?. rhs` &mdash; gets a method or member of an optional type if it has something, otherwise return `None`
- `lhs ?# rhs` &mdash; applies `#` to an optional type if it has something, otherwise return `None`
- `lhs ?? rhs` &mdash; fallback to another value if the left side is `None`.

__Results:__

- `lhs !` &mdash; returns the result value if it's not an exception, otherwise propagate to the nearest `try` keyword *(see [`try` block](#try--except))*

__Functional:__

- `lhs -> rhs` &mdash; pipelining, disregards `lhs` and returns `rhs`, but `_` becomes the value of `lhs` in the expression of `rhs`
- `~ rhs` &mdash; inferred type conversion

__Assignment:__

- `lhs = rhs` &mdash; assignment or inferred-type declaration *(See [Variable Declarations](#Variable-Declaration).)*
- `lhs := rhs` &mdash; always inferred-type declaration
- `lhs: T = rhs` &mdash; explicit-type declaration
- `lhs as rhs` &mdash; inline for `:=` but returns the assigned value instead of being void; the operands are inverted (variable goes on the right); creates an immutable variable by default *(See [Inline Binding](#Inline-Binding).)*

Operators that return the same type as their left-hand side have assignment alternatives by adding an equals sign `=` after it. This sets a variable based on its previous value. The left-hand side must be an already defined variable. If it's immutable, then this is the same as shadowing it. If it's mutable, then the value is mutated. *(See [Mutability(#Mutability-mu).)* All of these are void statements, i.e. they return nothing and should only be used in an expression by themselves.

- `lhs += rhs` &mdash; `lhs = lhs + rhs` &mdash; increment
- `lhs -= rhs` &mdash; `lhs = lhs - rhs` &mdash; decrement
- `lhs *= rhs` &mdash; `lhs = lhs * rhs` &mdash; multiplication assignment
- `lhs /= rhs` &mdash; `lhs = lhs / rhs` &mdash; division assignment
- `lhs //= rhs` &mdash; `lhs = lhs // rhs` &mdash; floor division assignment
- `lhs %= rhs` &mdash; `lhs = lhs % rhs` &mdash; modulo assignment
- `lhs %%= rhs` &mdash; `lhs = lhs %% rhs` &mdash; floor division modulo assignment (binds `lhs` to a range in `[0, rhs)` if `rhs` is positive or `(rhs, 0]` if `rhs` is negative)
- `lhs %&= rhs` &mdash; `lhs = lhs %& rhs` &mdash; bitwise `AND` assignment
- `lhs %|= rhs` &mdash; `lhs = lhs %| rhs` &mdash; bitwise `OR` assignment
- `lhs %^= rhs` &mdash; `lhs = lhs %^ rhs` &mdash; bitwise `XOR` assignment
- `lhs <<= rhs` &mdash; `lhs = lhs << rhs` &mdash; bitshift left assignment
- `lhs >>= rhs` &mdash; `lhs = lhs >> rhs` &mdash; bitshift right assignment
- `lhs ++= rhs` &mdash; `lhs = lhs ++ rhs` &mdash; append to an array (not allowed if `lhs` is a fixed length array)

All assignment operators *(except for `as`)* are **void.** This is intentional to prevent bugs and difficult to read code. It's recommend to put all assignment statements on their own separate lines for clarity. 

### Order of Operations

Organizing operator precedence requires balancing mathematical convention with developer intuition. Below is the standard, battle-tested hierarchy for a modern programming language like Mu, ordered from highest precedence (binds tightest) to lowest precedence (binds loosest).

1. __Member Accessing/Inline-Binding (Highest):__ `lhs . rhs`, `lhs ?. rhs`, `lhs as rhs`
2. __Primary & Postfix Operators:__ `lhs # rhs`, `lhs ?`, `lhs ?#`, `lhs !`
3. __Unary Operators:__ `+ rhs`, `- rhs`, `not rhs`, `and rhs`, `or rhs`, `%~ rhs`, `++ rhs`, `& rhs`, `~ rhs`
5. __Exponentiation:__ `lhs ** rhs` *(right-associative: `2**3**2` is `2**(3**2)`)*
6. __Multiplicative & Bitshift Operators:__ `lhs * rhs`, `lhs / rhs`, `lhs // rhs`, `lhs % rhs`, `lhs %% rhs`, `lhs << rhs`, `lhs >> rhs`
7. __Additive & Vector Operators:__ `lhs + rhs`, `lhs - rhs`, `lhs ++ rhs`, `lhs & rhs`
8. __Bitwise Logic:__ `lhs %& rhs`, `lhs %^ rhs`, `lhs %| rhs`
9. __Range & Interval Operators:__ `lhs .. rhs`, `lhs ..= rhs`
10. __Comparisons & Membership:__ `lhs == rhs`, `lhs != rhs`, `lhs > rhs`, `lhs < rhs`, `lhs >= rhs`, `lhs <= rhs`, `lhs in rhs`
11. __Logical AND:__ `lhs and rhs`
12. __Logical OR:__ `lhs or rhs`
13. __None Coalescing:__ `lhs ?? rhs`
14. __Pipelining (Lowest):__ `lhs -> rhs`
15. __Assignment (Lowest):__ `lhs = rhs`, *etc. (excluding `as`)*

### Operation chaining with `[]`

**Infix operator** are operators who have both an `lhs` and `rhs`. For any non-void infix operator `op`, you can write it as `lhs op[rhs]` (an operator + an array literal). This is shorthand for writing `((lhs) op (rhs))` and has the same precedence as accessing with `.` or wrapping parentheses around the expression. This is primarily used for arrays, but has many potential use cases which can be explored.

```
a * -b + [c] * [d] - [e] * f
-- That is equivalent to this:
a * -(((b + c) * d) - e) * f
```

Operation chaining has the same precedence has member accessing `.`. This allows you to treat any operator as a member access. The most useful use cases for this are for array indexing `#` and piplining `->`.

```
array#[0]#[1]              -- => ( ( array # 0 ) # 1 )
fn1()->[fn2(_)]->[fn3(_)]  -- => ( ( fn1() -> fn2(_) ) -> fn3(_) ) => fn3( fn2( fn1() ) )
```

Pipelining can be particularly useful when combined with `[]` for inlining a variable that's repeated in an expression or method-chaining on an object with non-method functions.

```
-- Repeated value:
onePlusTwoCubed = (1+2)->[_*_*_]
print("{onePlusTwoCubed}")       -- Prints "27"

-- Method chaining:
object.method1()->[fn1(_)].method2()->[fn2(_)]
-- Becomes...
fn2( fn1( object.method1() ).method2() )
```

Although arrays and tuples are treated differently, their syntaxes are analogous to each other&mdash;sit's just that one uses square brackets `[array]` and the other uses parentheses `(tuple)` or curly braches `{tuple}`. We can use this to our advantage to give operation chaining another feature: **tuple generation**. If the array literal after an operation has more than one indexes (e.g. `[expr, expr]`) or one or more named indexes (e.g. `[name: expr]`), then it will return a **tuple.** *For more information on tuples, see [Tuples](#Tuples).*

```
-- Apply `x ==` to each value, returns a tuple of bools.
-- Prefix `or` takes a tuple of bools and goes if any member is true.
if or x == [0, 1, 2, 3, 4]:
    print("x is 0 or 1 or 2 or 3 or 4")
```

Another example:

```
x = 1
y = x -> [_, _ + 1, _ + 2, _ + 3] -> [_.0 * _.1 * _.2 + _.3]    -- Split `x` into 3 components and collect.
```

`y` simplifies like this:

1. `x -> [_, _ + 1, _ + 2, _ + 3] -> [_.0 * _.1 * _.2 + _.3]`
2. `(x, x + 1, x + 2) -> [_.0 * _.1 * _.2 + _.3]`
3. `(x * (x + 1) * (x + 2) + (x + 3))`
4. `(1 * (1 + 1) * (1 + 2) + (1 + 3))`
5. `(1 * 2 * 3 + 4)`
6. `(10)`

Tuples are transparent by default, and a tuple with only one position component and no named components is equal to just its position component. *For an explaination why, see [Tuples](#Tuples).*

- `(10) == (10).0`
- `(10).0 == 10`
- _Therefore:_ `(10) == 10`

Most programming languages use `[]` by itself for indexing an array. While this is still possible in Mulang, operation chaining gives the coder a lot more options and ability to write expressive code. The transition from array indexing with `[]` in other languages like C or Python to array indexing with `#[]` in Mulang is as simple as adding `#` before each square bracket.

- `array[0][1]` -> `array#[0]#[1]`

This frees `name[]` to take on a new meaning in Mulang: **meta functions.** *For more on that, see [Meta Functions](#Meta-Functions).*

## Basic Bindings

There are two types of bindings: basic `=` and meta `::`. See [Meta Bindings](#Meta-Bindings) below for details about `::`.

### Variable Declarations

Variables are declared with just the equals sign (`=`). Type is inferred, but can be declared with a colon (`:`). You can also use the `:=` operator instead to declare and infer the type at the same time. This is useful for shadowing mutable variables. *(See [Mutability](#Mutability).)* For now, just know that anytime you see `:` before `=`, *it always declares a new variable,* and if you see just `=`, *it's either declaring or mutating a variable.*

```
a = 1       -- Implicit declaration
b: int = 1  -- Explicit declaration with type.
c := 2      -- Explicit declaration but infer its type.
```

Variables are immutable, but declaring it again shadows it. Any subsequent `=` of an immutable variable is an implicit declaration. Redeclaring a variable with the same name is known as **shadowing.** This makes Mulang flexable like a dynamicly typed language like Python while still having the advantages of being statically typed.

```
a = 1
a = 2
a = 3
a = "hello"   -- The type of a shadowed variable doesn't have to match.
```

You can also shadow a variable using its previous value.

```
i = 0
i = i + 1     -- Sets new `i` based on old `i`
i += 1        -- Same as above
```

Using the single equal-sign is a void statement. If you use it within an expression and not on its own, it's a syntax error. This helps prevent the common bug of using `=` when you meant `==`. For inline binding, use `let`/`then` or `as` instead. *(See [Inline Binding](#Inline-binding).)*

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

Mutable variables are marked with the keyword `mu`, the main star of the language. Just as Go has its `go` keyword, Mulang has `mu`. This was chosen since mutability is a common practice in programming much like functions are&mdash;which is why functions also get their own two letter keyword `fn`. *(See [Function Declarations](#Function-Declarations).)* The general rule of thumb in Mulang is that *patterns scale with complexity*&mdash;simple things like declaring a variable or function use short patterns, more complex things use bigger patterns.

You might think the choice of `mu` as a keyword&mdash;which is the same name as the language itself&mdash;will be a potential source of confusion. However, Go doesn't have this problem with its `go` keyword. When searching or discussing about *Mu*, it's sometimes preferable to write it as *Mulang* for clarity, just as other languages can have "lang" at the end of their names such as *Go* -> *Golang* or *D* -> *Dlang*.

Declare a mutable variable with `mu type`. Setting it will change the value instead of shadowing it. The type of value when mutating it must match its original type.

```
x: mu int = 0
x = 1          -- `x` is mutated
```

You can also infer the type with `mu _ = _`:

```
mu x = 0   -- Same as `x: mu int = 0`.
x = 1
```

Or you can declare the type and set it later:

```
x: mu int
doSomething()
x = 1
```

Not setting a mutable variable implies `= undefined` after it which means it cannot be used until it's been set. The exception is passing undefined variables to the `out` parameters of functions. *(See [Function Declarations](#Function-Declarations).)*

```
x: mu int = undefined
-- `x` cannot be used here.
(--
doSomething(x)   -- This is an error.
--)

x = 1
-- `x` can be used now.
doSomething(x)   -- This is okay.
```

See [`undefined`](#undefined) for more details about what it means and its uses.

`mu` variables cannot be shadowed by `=`. They can only be mutated. Assigning to the variable for the rest of the scope and any sub-scopes will mutate the variable&mdash;except for functions which always set a new variable unless its been explicitly captured. *(See [Capturing](#Capturing-capture).)*

```
mu x = 0
block:
    x = 1
    print("{x}")  -- Prints "1", same as outer `x`.
print("{x}")      -- Prints "1", `x` was mutated.
cantSetX() =
    x = 2
cantSetX()
print("{x}")      -- Prints "1" again, cantSetX didn't change it.
```

If you wish to shadow it, you can redeclare the variable with `: T =` or `:=`.

```
mu x = 0
block:
    x := 1        -- Delcare new `x` in this block.
    print("{x}")  -- Prints "1", inner `x` was shadowed.
                  -- Exit block
print("{x}")      -- Prints "0", outer `x` was not mutated.
```

#### References (`ref`/`ref mu`)

A reference type points to the same spot in memory as another variable. It's like a lightweight version of a pointer. *(See [Pointers](#Pointers).)* Lifetimes are inferred via borrow-checking when the memory model is set to `Borrow`. *(See [Memory Models](#Memory-Models).)* The right hand side has be something stored in memory, i.e. not a constant. References are immutable by default unless declared with `ref mu`. The syntax follows the same pattern as `mu`:

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

When destructuring a type that isn't anonymous, the type can optionally be put before the parentheses/braces, otherwise it's automatically inferred. This could conflict with function declarations which uses a similar pattern `name(param) = body`. *(See [Function Declarations](#Function-Declarations).)* Because functions are far more common than destructured assignments, they take the simpler pattern. An empty tuple and ampersand (`()&`) should be placed at the start of the expression so it won't be confused for a function declaration. This guarantees that you are destructuring based on the correct type. This follows the same schema as pattern matching such as in `case` and `except` lines. *(See [Control Flow](#Control-Flow).)*

```
Thing :: {x: int, y: int}
thing = Thing(x: 1, y: 2)
()& Thing{x, y} = thing       -- Split thing into its components.
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

The arguments of a function are a **tuple.** The share the same sintax. Arguments are separated by commas `,`. Trailing commas are ignored, but leading commas and double commas are considered a syntax error. 

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

Functions can also be declared with type `fn` to be set later. This type is known as a **function pointer.** It lets you treat functions as values.

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

Function parameters can be declared like variables. Likewise, you can modify their mutability and referenceness the same way.

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

Another type of parameter is `out`. This is like `ref mu` but is treated as `undefined` at the start of the function. Use it to set a variable that hasn't been set yet. The parameter must not be `undefined` in any branch within the function. This means either setting it within the function or passing it to another function with an `out` parameter. This ensures that the variable is set after the function has been called. 

```
setInt(out i) =
    i = 3

x: mu int
setInt(x)
print("{x}")    -- Prints "3"
```

This works for mutable variables, but what if you wanted to make an immutable variable using `out`? You can write `out as` while calling a function to declare it as an immutable variable in the current scope. This has the same rules as just `as`. *(See [Inline Binding](#Inline-Binding).)

```
setInt(out as n)   -- Declare a new variable `n` that gets set by `setInt`.
print("{n}")       -- Prints "3"
```

#### Capturing (`@capture`)

Immutable variables can be captured without an issue. If you try to set it within a function, it will get shadowed within the scope of the function. This also includes other functions which are also immutable by default.

**By default, functions cannot capture mutable variables.**

This is an intentional decision for better safety in Mulang. Functions don't automatically capture mutable variables. Instead, any variable set inside a function is treated as a new variable. This helps prevent accidentally mutating a variable that you didn't mean to and encourage good functional programming practices.

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

To capture a mutable variable, you must least each variable in the `@capture` decorator. This helps make it easy to see which functions can mutate other variables and which don't. You can capture multiple variables at once with commas `,`. Each captured variable must be listed. This follows the same practice as `import` and `inherit` where all words in a given context are listed out clearly so that there are no accident name collisions or hidden gotchas.

```
mu count = 0
mu squared = 1
mu cubed = 1

@capture(count, squared, cubed)
addCount() =
    count += 1
    squared = count * count
    cubed = squared * count

addCount()
addCount()
addCount()
print("{count}, {squared}, {cubed}") -- Prints "3, 9, 27"
```

#### Lambda Functions

You can define a function within an expression with the keyword `fn` in the pattern `fn(_) = _`. This is useful for passing functions to other functions. If the lambda function has multiple lines, it must be wrapped in `do`...`end`.

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

The same keyword for creating functions is also used for function type notation. If this seems confusing, just remember where the context is: if it's being used as a type, it means a *function pointer type*; if it's being used as a value, it's a *lambda function*.

```
action: mu fn(int, int): int    -- Declaring a variable with a function type.
action = fn(a, b) = a + b       -- Passing a function to the variable as a value.
```

Note that if you try to declare a function with the name `fn`, it will throw an error. This prevents potential gotchas and silent errors. The exception is if it's the last value in a block, then the return value of that block is a function.

```
(-- This is an error because you are trying to declare a function with the name `fn`:
fn(a, b) =
    a + b
fn(1, 2)
--)

-- This is not an error, it's a function that returns another function:
curryAdd(a: int): fn(int): fn(int): int =
    fn(b) =
        fn(c) =
            a + b + c

curryAdd(1)(2)(3)
```

Sometimes, you may need to add decorators to a lambda function thats being passed into another function. To do this, write `do block:` instead of `do fn() =`. This will create an inline-block. The last expression evaluated is the return value, so define a lambda function with `fn` inside of it.

```
mu count = 0
forEach([1, 2, 3, 4], do block:
    @capture(count)
    fn(x) =
        count += x
end)
```

#### Named Parameters (`&`)

Functions declared with a named tuple `{}` require each parameter to be named in order to call them. Named tuples can be destructored so that their members become variables in the scope. Call the function with parentheses like before but with the name of each parameter inside.

```
add{ a: int, b: int }: int =
    a + b

add(b: 1, a: 2)     -- Order doesn't matter.
```

You can also have both positional and named parameters. Split the positionals into `()` and named into `{}` and place a `&` between them. Parentheses mark a **positional tuple**, and curly braces mark a **named tuple**. *(See [Tuples](#Tuples).)* This works the same way as **destructuring.** *(See [Destructuring](#Destructuring).)* Named parameters don't take up any position, so they can be placed anywhere in the parameters.

```
add(x: int) & { a: int, b: int }: int =
    x + a + b

-- Order doesn't matter, `x` is the same for all of these:
add(1, a: 2, b: 3)
add(a: 2, 1, b: 3)
add(a: 2, b: 3, 1)
```

Named parameters can be defined in their own object and then passed in with the `&` operator as well. The named parameters in the function can also be collected into a single variable using `as`. Like with desctructuring, a `()&` should be placed before the typed name parameters even if the function doesn't have any positional parameters.

```
Settings :: { enabled: bool }
doThing(key: str) & Settings as settings =
    if settings.enabled:
        Some(callApi(key))
    else:
        None

settings = Settings(enabled: True)

doThing("foo", &settings)      -- Spreads `settings` into the arguments.
doThing("foo", enabled: True)  -- Or passed as a named parameter.
```

See [Tuples](#Tuples) for more details on the `&` operator.

### Inline Binding

You can also bind variables within an expression using `let`/`then` and `as`. `let`/`then` is used for a single expression, whereas `as` binds for the rest of the scope.

#### `let`/`then`

```
squared = let x = getSomething() then x * x

while next() as val != None:
    print("{val}")
```

All variables after `let` are new declarations, much like variables in a non-capturing function; they cannot mutate any existing variables. In other words, they have an implicit `:=` even when they use `=`. 

```mu
mu x = 0
y = let x = 1 then x + x   -- Is the same as `y = let x := 1 then x + x`
print("x = {x}, y = {y}")  -- Prints "x = 0, y = 2"
```

Multiple variables can also be declared at once. You must surround the variables with square brackets like `let [] then`. Each variable is seperated by commas. 

```
sum = let [x = 1, y = 2] then x + y
```

`[]` are used instead of `()` to distinguish it from a regular function call. It also follows the pattern of `name[]` meaning a compile-time function, which `let []` sort of is a compile-time function since it's running at compile-time. However, it's technically a part of the syntax and not a function.

#### `as`

`as` sets a value within an expression as a variable within a block's scope. It returns the value of the left-hand side, the right-hand side should be a valid variable name. Like `let`, it's an explicit declaration like `:=`, so it can't mutate. Note that this is slightly different from but consistent with the `as` that's used for aliasing. *`as` the operator* is only used in **expressions**; meanwhile, *`as` for aliases* is only used in **patterns.** *For information on how patterns work, see [Destructuring](#Destructuring).*

The simplest use case for `as` is to pair it with a `while` loop to get a value on each iteration.

```
while next() as val != None:
    print("{val}")
```

Another is to use while method chaining to get a result of one of the methods. Note that `as` has the same order of operations as `.`. *(See [Order of Operations](#Order-of-Operations).)*

```
object.method1()
    .method2() as result
    .method3()
print("{result}")
```

Type must be inferred. This is to avoid using `:` in an expression, which could be mistaken for the key name of a tuple or a declaration.

```
action(get() as result: int) -- Is this a named component? Did you forget the commas?
```

However, mutability can be set with `as mu`.

```
object.method1()
    .method2() as mu result   -- New mutable variable.
    .method3()
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
- **Maps**: `type#type`
- **Inferred**: omit the annotation entirely

### Built-in Types

Some built-in types include `int`, `uint`, `float`, `bool`, `char`, `str`, and `ptr`. Note that although built-in types use lowercase names, they are not *keywords*. This is just a naming convention. It's recommended that users create custom types with capitalized names to differentiant from built-in types.

```
myInt: int = -1234
myInt: uint = 5678
myFloat: float = 12.34
myBool: bool = True
myChar: char = 'a'
myStr: str = "Hello"
myPtr: ptr = ExternalLib.getSomething()
```

You can get the type of any variable with the copile-time function `typeof`. This fetches the type of that symbol at that point during compile time. *(See [Meta Functions](#Meta-Functions).)*

```
x = 0
y: typeof[x] = 1    -- Ensures that x and y have the same type.
```

You can also get the default value of any type with the copile-time function `default`. The type needs to have a default value defined which is yet to be determined how, but they're already defined for basic types.

```
x = default[int]    -- == 0
x = default[float]  -- == 0.0
x = default[bool]   -- == False
x = default[char]   -- == '\0'
x = default[str]    -- == ""
x = default[ptr]    -- == Null
```

`default` can also infer the type. This can be useful in certain situations, like if you want to leave a function that returns something empty so that you can implement it later.

```
implementLater(): int = default
```

There will be an API for defining the default value of custom types, but that is outside the scope of this document. Here is an example of what that might look like:

```
MyType :: struct =
    value: int

MyType :: impl(Default) =            -- Implement the Default proto
    getDefault() = MyType(value: 0)  -- Default value for this type
```

*This API is subject to change.*

You can also get the size of any type with the copile-time function `sizeof`. It returns a constant `uint` (unsigned integer) with the number of bytes of memory that type requires. The exact sizes of some types like `int` or `float` might vary, but you can rely on `char` and `bool` being 1 byte each. There's also the `void` type which represents no data. `ptr` depends on the pointer size of the system. 

```
sizeOfBool = sizeof[bool]   -- == 1
sizeOfChar = sizeof[char]   -- == 1
sizeOfVoid = sizeof[void]   -- == 0
sizeOfPtr  = sizeof[ptr]    -- == 4 or 8
```

You can call a type as a function to convert types into other types if conversion is possible. 

```
x = 1
y = float(x)
print("{y}")     -- Prints "1.0"
```

Or implicitly convert types using `~` if the result type of the expression is known.

```
x: float = 1.5  -- `x` is a float.
y: int = ~x     -- `y` is expecting an int, so call `int(x)`
print("{y}")    -- Prints "1"
```

#### Booleans

`bool` is a built-in enum type with its only members being `False` and `True`. This means you can also pattern match with a bool, although it's recommended to use `if`/`else` instead. Enum-members are usually capitalized, and this matches Python's `True` and `False` convention. 

```
match value
| True:
    print("It's true!")
| False:
    print("It's false!")
```

That's the same as this.

```
if value:
    print("It's true!")
else:
    print("It's false!")
```

#### Numbers

There are several number types. More may be added in the future, but for now we'll focus on the 3 main types. For any of these types, you can add a suffix to explicitly declare a number as a particular type. However, suffixes are disabled if the number already has letters in it. 

| Type | Meaning | Suffix | Examples |
|:--|:--|:--|:--|:--|
| `int` | signed integer | `i` | `1`, `2i`, `10i`, `0xabcdef` |
| `uint` | unsigned integer | `u` | `1`, `0u`, `5u`, `0x10ff` |
| `float` | floating ploint number | `f` | `1.0`, `1.5f`, `100f`, `2.0e100` |

A number literal starts with a digit `0123456789` followed by zero or more other digits, letters, or underscores `_`. All number literals are case-insensitive, so `1.0f == 1.0F`.

The sign is considered an operator and not a part of the constant itself. This gets automatically calculated in constant expressions at compile-time so that it seems like it's a part of the constant.

- `- 1` -> `minus` + `1` -> `signflip(1)` -> `-1`

The exception to this rule is in scientific notation: the sign after `e` is a part of the number itself. There must not be a space between `e` and the sign `-`/`+`. The sign is optional for positive exponent values.

- `2e-100`, `5.0e+128`, `6.8e-128`, `10e3`

You can place underscores `_` anywhere in a number to break it up into segments. This doesn't change the value.

- `1_234`, `1_000_000`, `0b1111_0000`, `0xab_cd_ef`

Leading zeros are allowed as well and don't effect the value. Unless there's a base letter, it's still in base 10, unlike in most C languages where a leading 0 switches to base 8.

- `001`, `009`, `000100`, `000u`, `010.0f`

Changing the base invovles adding a `0` and a then the letters `b`, `o`, or `x` to the start of the number.

| Prefix | Base | Valid Digts |
|:-:|:-:|:-:|
| `0b` | 2 | `01` |
| `0o` | 8 | `01234567` |
| `0x` | 16 | `0123456789abcdef` (case insensitive) |

As said before, it's essential to put spaces between operators so that they don't get confused for something else.

```
1++1  -- Creates an array [1, 1].
1+ +1 -- Add 1+1 => 2.
1--1  -- Just 1 with a comment
1- -1 -- Subtract 1-(-1) => 1+1 => 2.
```

It's not likely anyone would mark a number as positive and add it on the right hand side, and neither is it likely anyone would flip the sign of a number and subtract it on the right-hand side. Most people whould write both `1+(+1)` and `1-(-1)` as just `1+1`. 

#### Characters

Characters or `char` are written with apostrophes (`'_'`) *(also called single quotes).* They store 1 byte of data. You can also do arithmatic on them like with numbers.

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

You can write a raw string with `''...''` (two apostrophes). Although apostrophes `'` are used for chars, an empty char isn't possible since the default char is written `'\0'` (null character). This is a common practice in programming languages where `"` mark formattable strings and `'` mark raw strings, so this should be easy to understand for any programmer. When you write `''`, every character after it (including whitespace and indentation) is in the string until the closing `''`. Escaping with backslashes `\` and insertion with curly braces `{}` are ignored.

```
rawString = ''It's okay to put an apostrophe (') in the string.''
filePath = ''C:\files\on\windows.txt''
template = ''Insert here -> {{variable}}''
```

To escape `''` within a raw string, add a hash `#` before and after the apostrophes. The number of `#`s must match to close the raw string.

```
-- Add a `#` to escape the `''` within the string.
bigDocument = #''
    This  '
   is   ''      ''
  all       ''
  in  '         '
   a     '''' '
    string
''#
-- Maching number of `#` closes the string.

-- `##''` to escape the `''#` within the string.
nestedDocument = ##''
bigDocument = #''
    This  '
   is   ''      ''
  all       ''
  in  '         '
   a     '''' '
    string
''#                 
''##
-- `''##` closes the matching `##''`.
```

While this is possible, this is not recommended for the sake of legibility. For anything more complicated, it's recommended to save the string in a document and open it as a file, which will be handled by a separate library and lies outside the scope of this document.

#### Arrays

Array types are declared with the hash symbol (`#`). This was chosen because the `#` is commonly used for numbers in English. For example, `#1` is read as `number 1`. A number after the `#` makes it a fixed length array `type#N`. Arrays are statically sized when written as `type#N`; `type#` is the dynamic form. Items are separated with commas (`,`). The hash symbol is also used for accessing an array, giving a clear visible mirror between the two. 

```
list: int#4 = [1, 2, 3, 4]
print("length of list: {len(list)}")
compressedList = [list#0 + list#1, list#2 + list#3]
doubleArray: int#2#2 = [[1, 2], [3, 4]]
```

This builds on the visible simatry between type notation and their value expressions:

| Type | Notation | Expression |
|:--|:-:|:-:|
| **Results** | `T!` | `x!` |
| **Options** | `T?` | `x?` |
| **Arrays** | `T#N` | `x#n` |

Other languages use `[]` for indexing, but that has another meaning in Mulang. Instead, you can use operation chaining to do the same thing as `[]` in other languages. *(See [Operation Chaining](#Operation-chaining-with).)*

```
item = doubleArray#[1]#[0]  -- The 2nd row, 1st column
print("{item}")             -- Prints "3"
```

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

This follows a symmetry with tuples and the `&` operator:

| Type | Join | Spread |
|:--|:--|:--|
| Arrays | '[a, b] ++ c` | `[a, b, ++c]` |
| Tuples | `(a, b) & c` | `(a, b, &c)` |

Even though they behave similarly, arrays and tuples are fundamentally different, so their operations are kept seperate to make intention clear. 

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

A standard library will be made to safely handle pointer dereferencing and do pointer arithmatic, but that is outside the scope of this document. Here is an example of how it might work:

```
mu x = 0             -- Create a local mutable variable.
xPtr = getMuPtr(x)?  -- Map `Null` to a option type, branch if it's `None`, return `Some(ptr)` if it's not and unwrap it with `?`.
xPtr.set(1)!         -- Safely set the pointer and branch if there's an error.
print("{x}")         -- "1", the pointer successfully mutated `x`.
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

The presence of `:` signifies if a keyword is in block mode or inline mode. `then` is used to separate a subject and expression when a keyword block is inlined. 

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

You can put `block _` before any other block type to give it a label. This can be useful for nested loops.

```
block loop for x in 0..100:     -- Label this block `loop`.
    for y in 0..100:
        prod = x * y
        print("{x} * {y} == {prod}")
        if prod >= 100:
            break               -- Break inner loop, continue outer loop.
        if prod == 77:
            break loop          -- Break outer loop, loop ends.
```

#### `if` / `else`

Basic boolean branching.

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

#### `if or`/`if and`

The alternative is to put `or`/`and` immediately after `if`, it will check a tuple of booleans instead of a single boolean.

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

This can be paied with `==[]` to check multiple values at once. *(See [Operation Chaining](#Operation-chaining-with).)*

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

Note that `if and`/`if or`/`if not and`/`if not or` isn't actually a special syntax. It's that the `or`/`and` prefix operators compresses a tuple of bools to a single bool. 

```
x = or (True, False, True)
print("{x}")              -- Prints "True"
```

#### `match`

Enum/exception branching. Exhaustive by default. `| _:` for the default case. It's syntax is a bit different than most blocks. There's no `:` after the subject line in `match subject`. Each case starts with `|`. That's because it's parsily inlined, relying on the rule that symbols at the start of the line are part of the same expression. The colons are found at the end of each case, with each case being its own block. 

The patterns map to the type passed in after `match`, so you only need to reference the members of that type in each case block.

```
match choice             -- No `:`. No extra indentation is necessary.
| First:                 -- Each case is at the same indentation or more.
    print("First")       -- Each case marks a new block, indent here.
| Second(x):             -- Continue this for each case.
    print("Second({x})") -- ...
| Third{val}:            -- ...
    print("Third \{ val={val} }")
                         -- All choices were exhausted, so no `| _:` is necessary.
```

The inline form switches `:` for `->`, unlike other blocks which use `then` for inline form. This makes it easier to read and take up less space. Patterns are usually words, so you can visually sequence it into pattern/expression pairs: `| ptrn -> expr | ptrn -> expr | ptrn -> expr` etc.. This won't clash with the `->` operator because each case is a pattern, not an expression. Semantically, it makes sense because it does the same job&mdash;in `lhs -> rhs`, `rhs` is always returned, just like how the case returns that expression when the pattern matches.

```
message = match e | OpenError{filename} -> "Open error: {filename}" | _ -> "Unknown error"
```

You can have multiple patterns match to one case. If any of the case's patterns destructurs with a variable, the same variable name and type must be in all patterns. If not, use a wildcard `_` in each tuple or omit the tuples entirely to disable destructuring.

```
match choice
| First:
    print("First")
| Second(val) | Third{val}:            -- `val` must be in all patterns
    print("Second or Third, val={val}")

match choice
| First | Second | Third:              -- `First` doesn't have any values, so destructuring must be disabled.
    print("First, Second, or Third")
```

#### `if case`

This combines `match` and `if` into one expression. Useful if you want to destructur a single case of a sum type. This must be a pattern that matches the to type of value after `=`.

```
if case Pattern(x) = value:
    print("value is {x}")
else:
    print("value doesn't match")
```

#### `for` / `in`

Iterates through an array or iterator.

```
for var in expr:
    body
```

If inlined, it returns an iterator `iter[T]` which is lazily executed. It won't activate until it's collected in an array with `++` or passed into another `for _ in` loop. 

```
iterator = for x in list then x * 2
list = [++iterator]
```

The items can also be destructured. *(See [Destructuring](#Destructuring).)*

```
listOfTuples = [(1, '2', "three"), (4, '5', "six")]

for (x, y, z) in listOfTuples:
    print("{x}, {y}, {z}")
```

Unlike with destructuring, you don't need `()&` if you want to destruct from a known type.

```
for Thing{x, y} in getThings():
    print("Thing\{x={x}, y={y}}")
```

For an async iterator or an array of async types, you can use `for await` to automatically wait for each item in sequential order. *(See [`await`](#await).)*

```
for await x in asyncIter():
    print("{x}")
```

#### `for case`

Use `for case` to only iterate on enums that match a given pattern. This is the same as wrapping the body in an `if case` block. This only works for block-level `for` expressions. For inline, you need to use an `if case` or `match` so that the inputed iterator maps 1-to-1 with the outputed iterator.

```
for case Some(x) in iterGet():
    print("{x}")

-- That's the same as this:
for o in iterGet():
    if case Some(x) = o:
        print("{x}")
    else:
        continue
```


__NOTE:__ There is no inline version of `for case`. You have to use `if case` instead, and each yield must resolve to the same type. To make an inline version of the above example, we can unwrap each option type or yield the default value of its type (whatever it is), like this:

```
inlineForCase = for o in iterGet() then if case Some(x) = o then x else default
```

You can combine `for await` and `for case` into `for await case`. This will resolve each asynchronous instance and only run the body of the loop if the case matches, skipping any where the case doesn't match. 

```
for await case Some(x) in asyncIterGet():
    print("{x}")

-- Short for this:
for await o in asyncIterGet():
    if case Some(x) = o:
        print("{x}")
    else:
        continue
```

#### `while`

Repeats a block of code until the condition is `True`.

```
while cond:
    body
```

The same things that apply to `if` apply to `while` such as the `or == []` pattern.

```
mu x = 0
while or x ==[0, 2, 3, 4]:
    print("{x}")
    x = randInt(-1, 4)
```

`else` after a `while` block will run if the loop never ran even once.

```
x = False
while x:
    pass
else:
    print("The loop failed")
```

#### `while`+`as`

Some programming languages let you do something like `while value = getValue()`. Mulang doesn't allow this because `=` is a void statement. Instead, you can use `as` to achieve the same thing but more explicitly.

```
while None != getValue() as value:
    print("value = {value}")
```

#### `while case`

You can also do `while case` just like with `if case`. This will loop until the pattern breaks. 

```
while case Some(x) = nextValue():
    print("value is {x}")
```

#### `loop` / `until`

Repeats a block of code until `break` is called.

```
loop:
    print("I'm looping!")
    break
```

Add `until` after a `loop` block to create a condition that will break the loop. Unlike `while`, this will break when the condition is `True`, analogous to `if cond then break`. This would be a `do { body } while (!cond);` loop in other languages like C. The reason it doesn't use `while` is because that would start another block, and `do` already has a different meaning in Mulang, so `loop` and `until` were chosen instead. This makes it clear that `loop` and `until` are semantically connected since `until` is only used with `loop`.

```
loop:
    body
until cond
-- The loop will run at least once before checking the condition.
```

`loop`/`until` is arguably a better choice than `do`/`while` because it's more descriptive of what it's doing. *"Loop until this is true."* You can visually see that the condition is true after the `loop`/`until` block is finished, whereas you would need to invert the condition in your head for a `do`/`while` loop. It acts as a blockade to wait until a certain condition is true.

```
mu i = 0
loop:
    i += 1
until i >= 10
-- i is >=10 at this point
```

#### `until case`

You can do `until case`, but it's behavior is a bit different than the other `_ case` blocks. Instead of destructuring and creating a new variable in the loop body, it creates a new variable in the scope outside of the loop. This can be useful if you want to repeatedly call a function until you get something.

```
value: mu int?

loop:
    value = getValue()
until case Some(x) = value

print("value = {x}")
```

#### `break` / `continue`

Controls the iteration of any loop type mentioned. `break` exits out of the loop, and `continue` skips to the next iteration. This is only allowed in block-level loops. Inlined `for` loops are not allowed to skip iterations. This keeps the mapping between iterators 1-to-1.

Afer `break` or `continue`, you can give it a label to break. It must be the name of a `block` in the same scope or higher. 

```
block label loop:
    loop:
        print("Double loop! How do I break?")
        break label

print("I'm free!")
```

#### `switch`

Sometimes, there just isn't a clean way to write a C-style `switch` block with only the other control flow patterns mentioned. Mulang lets you do that with some aditional features to make it feel both modern and safe. The basic `switch` pattern is similar to `match`, but replace `|` at the start of each case with the keyword `case`:

```
switch choice:               -- Colon here becase `switch` is a block.
    case First:              -- Another indent for each case.
        print("First")       -- Each case marks a new block, indent here.
    case Second(x):          -- Continue this for each case.
        print("Second({x})") -- ...
    case Third{val}:         -- ...
        print("Third \{ val={val} }")

-- Inline:
score = 2
grade = switch x case 4 then 'a' case 3 then 'b' case 2 then 'c' case 1 then 'd' case _ then 'f'
```

`switch` can match both patterns and expressions, so the inline form uses `case _ then _` instead of `| _ -> _`. Whether the condition of each `case` is a pattern or an expression depends on the type passed to `switch`: enum/exceptions -> pattern, other types -> expression.

In the block form, the entire `switch` is a block with each `case` being a sub block. Any code in a `switch` outside of a `case` will always run. This can be useful for setting up variables that are shared between case blocks.

```
x = 2
switch x:
    mu y: int
    case 0:
        print("Inside first case!")
        y = 0b0001
    print("After first case.")
    case 1:
        print("Inside second case!")
        y = 0b0010
    print("After second case.")
    case 2:
        print("Inside third case!")
        y = 0b0100
    print("After third case.")
    case 3:
        print("Inside forth case!")
        y = 0b1000
    print("After forth case.")
    case _:
        print("Inside last case!")
        y = 0b1111
    print("After last case. y = {y}")
```

What it prints:

```
After first case.
After second case.
Inside third case!
After third case.
After forth case.
After last case. y = 4
```

Only the first case that matches will run by default. If you want to keep going to the next case, you can put `fallthrough` at the end of it. Put `fallthrough` at the bottom of every case to run all of them, starting at the first one that matches. This achieves the same effect as a C-style `switch` with no `break` statements but is safer since it's explicit rather than implicit.

```
x = 0
switch x:
    case 0:
        print("Fall through!")
        fallthrough
    case 1:
        print("Fall through!")
        fallthrough
    case 2:
        print("Fall through!")
        fallthrough
    case _:
        print("Fall through!")
        fallthrough
```

Use this to match multiple cases into one block.

```
x = 3
switch x:
    case 1:
        fallthrough
    case 2:
        fallthrough
    case 3:
        fallthrough
    case 4:
        print("x is 1 or 2 or 3 or 4")
        -- No fallthrough here so it breaks.
    case _:
        print("x is not 1 or 2 or 3 or 4")
```

The alternative is to use `|` to seperate case values like with the `match` block.

```
x = 3
switch x:
    case 1 | 2 | 3 | 4:
        print)"x is 1 or 2 or 3 or 4")
        -- No fallthrough here so it breaks.
    case _:
        print("x is not 1 or 2 or 3 or 4")
```

#### `opt`

Wraps a single expression in a option type. After `opt`, you can use `?` within to expression to unwrap multiple option values. If one `?` returns `None`, then the whole `opt` expression stops and returns `None`.

```
x = ( opt f(a?) ) ?? "fallback"     -- x is "fallback" if a is none.
```

If a function returns an option type, then use of the `?` is allowed without `opt`. Using an `?` inside a function will automatically infer the return type as an option. The return will be automatically wrapped in `Some(_)`. If there isn't a return value in the function, then it should be a type `void?`. Unwrapping it gives an empty tuple `()`. *[See [Tuples](#Tuples).)*

```
addStuff(a: int, b: int): int? =
    a = getSomething(a)?
    b = getSomething(b)?
    a + b
```

Option types automatically flatten in the following manner:

- `Some(Some(_))` = `Some(_)`
- `Some(None)` = `None`

This is true even for deeply nested options:

- `Some(Some(Some(Some(Some(_)))))` = `Some(_)`
- `Some(Some(Some(Some(Some(Some(Some(Some(None))))))))` = `None`

If this is undesired, you as a Mu programmer can make your own option-like enum type and use that instead. *(See [Meta Functions](#Meta-Functions).)*

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

If a function returns a result type `type!`, then it can also use the `!` like in a `try` block. Use of a `!` in a function automatically infers a result type as the return. If the function doesn't have a return value, like with options, the type is `void!` which unwraps into an empty tuple `()`. *[See [Tuples](#Tuples).)*

```
riskyFunction(a: int): int! =
    b = doSomething1(a)!
    c = doSomething2(b)!
    c
```

You can combine `?` and `!` together when the return type is `type?!` (an **option result type**).

```
-- With type notation:
doSomething(x: str?): void?! =
    x = x?
    doSomethingElse(x)!

-- Or inferred:
doSomething(x) =
    x = x?
    doSomethingElse(x)!
```

Result types flatten similarly to option types. The rules go as follows:

- For every `raise` or `!` in a `try` block, an exception is added to the exception sum type of the result.
- For every `except` after the `try` block, an exception is removed from the exception sum type of the result.

When all exceptions have been handled, the result is `type!void` which automatically converges to just `type`.

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

Exits out of a function with an `iter[_]` type. The return value of the function must be of type `iter[T]` where T is the yield type. When you have `yield` in your function, the actual return value in the function body is discarded, and using `return _` in it is a compile-time error. Use of `yield` will infer the return type as `iter[_]`. 

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

Exits out of a function with an `async[_]` type. The return type of the function must be of type `async[T]` where T is the type that the asynchronous value will resolve in the end. The return value of the asynchronous instance is determined the same way as a non-asynchronous function. Use of `await` will infer the return type as `async[_]`. 

```
asyncFn(a, b): async[int] =
    a = await fetch(a)
    b = await fetch(b)
    a + b
```

Both `yield` and `await` can be used together in an `iter[async[_]]` type. As the type suggests, each yield is of type `async[_]`. Use `for await` to wait for each async value to resolve in sequential order.

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

Runs after a function done. For iterator functions, this is when the iterator was broken or exhausted. For asynchronous functions, this is when the asynchronous type is resolved or rejected. Each `defer` statement go in reverse order: *first-in last-out*. It can be one line `defer _` or a block `defer:`. Generally though it's just one line such as `defer cleanUp()`. 

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

## Error Handling

- `expr!` &mdash; propagate an error upward (Rust/Swift style).
- Exceptions are sum types; the compiler unions all possible exception types from every `!` site in a `try` block.
- `try` / `except` can be an expression or a block and must unify return types (like `if` / `else`).
- Pattern matching on errors works with `match` or `if case Pattern(x) = value`.

## Pattern Matching and Destructuring

- Full `match` (exhaustive unless `case _` is present).
- `if case Pattern(x) = value then` &mdash; like Rust's `if let`.
- Destructuring supports structs, tuples, enum variants, and wildcards (`_`).

---

## Meta Bindings (`::`)

Variable and functions primarily use the equals sign (`=`) and are for storing actual data within a program, but there's another type of binding used for abstract values for the compiler to know about such as constants, types, inline-functions, and generics. This type of declaration is **constant**; in other words, they cannot be **mutated** or **shadowed**. Depending on what it is, subsequent `::` of the same name will modify its definition. The most common is `:: impl` with adds methods and static variables to a meta binding. 

### Constants

Putting a constant value after `::` creates a constant. This holds an unchangeable value that must be known at compile time. Explicit typing isn't necessary since it cannot be changed.

```
PI :: 3.1415926535
```

You can also bind a function to a constant. When calling it, it would be the same as defining it inline and then calling. This can be useful if you need to pass a function multiple times but don't want it to be outputted when compiled.

```
IDENTITY :: fn(x) = x
addOne :: fn(x) = x + 1
value = addOne(2)               -- Same as (fn(x) = x + 1)(2), result is 3.
array = map([1, 2, 3, 4], addOne)
```

### Aliases

Assigning a type after `::` creates an alias

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

Every opaque type by itself is its own tuple, so for example `char` and `(char)` are the same. This means that in the example, `int & float & char` is the same as `(int, float, char)`.

Tuples use commas (`,`) to separate components for both positional (`()`) and named (`{}`) tuples. This follows the same rules as function parameters. *(See [Function Declarations](#Function-Declarations).)*

Product unions with the `&` operator can be used for both types and values. When combining two or more positional tuples, the positions of subsequent tuples get bumped up by the number of positions in the previous tuples, i.e. `(a, b) & (c, d)` becomes `(a, b, c, d)`. When you combine two or more named tuples, conflicting named parameters override each other with the last tuple taking priority&mdash;much like how shadowing works. So if you have `{x: 1} & {x: 2}`, the result is just `{x: 2}` since it overrides the `x` of the previous tuple. Positional tuples and named tuples can be combined together for example `(0, 1) & {x: 2}`. The shorthand for this is to write named parameters in a positional tuple like `(0, 1, x: 2)`. 

You can think of it as every tuple always having both dimensions, just with most slots empty:

```
(0, 1)           -- positional: (0, 1), named: {}
{x: 2}           -- positional: (),     named: {x: 2}
(0, 1) & {x: 2}  -- positional: (0, 1), named: {x: 2}
{x: 2} & (0, 1)  -- positional: (0, 1), named: {x: 2} -- identical
```

So `&` has different commutativity rules depending on what's being combined:

| Combination | Commutative? | Rule |
|:--|:--|:--|
| Positional & Positional | No | Positions concatenate in order |
| Named & Named | No | Conflicts resolve last-wins |
| Positional & Named | Yes | Orthogonal, no interaction |

This makes the algebra quite principled. The only cases where order matters are also the cases where a conflict is actually possible &mdash; two positional slots or two named slots with the same key. When there's no possible conflict, order is irrelevant.

It also means the shorthand `(0, 1, x: 2)` isn't really special syntax. It's the natural representation of a tuple that has both dimensions populated, which any `&` expression across the two types would produce anyway.

Opaque types such as primitives and enums coerce into a tuple of one, so creating a product type of them creates a positional tuple, e.g. `int & float & char` becomes `(int, float, char)`. Structs convert to named tuples unless declared with an `@opaque` decorator, in which case they behave like opaque types. The `void` type coerces to an empty tuple `()`. *(See [Decorators](#Decorators)* for more informations on available decorators.)

Combining empty tuples produces an empty tuple `() & () == ()`. The same is true for empty named tuples `{} & {} == {}`. This also means that empty positional tuples and empty named tuples are equivalent `() == {}`. Both tuples have zero dimensions in both positional and named components; therefore they are equivalent. Saying `() & {} & ()` or `{} & ()` and any combination of empty tuples all produce an empty tuple. If you think about it, this makes sense. They all represent nothing, like a cup that can holds 0mL of water. If you stacked a bunch of 0mL cups, you would still not be able to hold any water in it. Is it even a cup then? That's why empty tuples are treated as `void`, which verbally represents nothing, because they're all nothing. Therefore `void == () == {}`. The 3 types are all the same. 

This also means that a void function and a function that returns an empty tuple are the same. *Empty positional tuples, empty named tuples, and void are fully interchangeable at the type level.*

```
voidFn(): void = ()
voidFn(): () = ()
voidFn(): {} = ()
```

We say that a void function returns nothing. Well, that's what an empty tuple is: *nothing*. So there isn't any issue here. Maybe in other languages there would be in an issue, but not in Mulang. 

### Definition Blocks

Some keywords after `::` start a **definition block.** They can only be used in `::` definitions. These are special blocks used for abstract data such as types and static variables. Inside any definition block, the word `Self` is a self-reference to the abstract data that's being defined. 

#### Structures (`struct`)

Structs are product types&mdash;or in other words&mdash;plain data containers. They cannot extend other structs, but can inherit members of other structs. *(See [Inheritance and Visibility](#Inheritance-and-Visibility).)*

Put an equals sign after the `struct`. This makes it easier to tell type definitions from aliases and lets the parser know that it's starting a block since a block always starts after a `:` or `=`. `=` was chosen over `:` to show that the block is some sort of data rather than control flow. This makes it clear that when you see `=` at the end of a line, something is being defined. It also resembles the familiar `var: type = value` but the `::` makes it clear that this isn't a run-time value. 

```
MyStruct :: struct =
    name: str
    value: int
```

You can write it with one line, seperating each member with a comma `,`.

```
MyStruct :: struct = name: str, value: int
```

Instantiate a struct by calling it like a function. Each member is treated as a named argument.

```
myObject = MyStruct(name: "Foobar", value: 1)
```

Structs are transparent. They can be destructured like named arrays. Use `@opaque` if you need to disable this. That will treat the struct type as an opaque type and prevent it from being inherited with `inherit`. *(See [Inheritance and Visibility](Inheritance-and-Visibility).)*

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

#### Enumerables (`enum`)

Enums are sum types. They define a closed set of variants. Variants may carry data turning them into a tagged union. Like with `struct`, place an `=` after `enum` before starting the block. 

```
MyEnum :: enum =
    First
    Second(int)
    Third{val: int}
```

The inline version works the same.

```
NyEnum :: enum = First, Second(int), Third{val: int}
```

Like structs, instantiate by calling the member as a function unless it doesn't carry any data.

```
a = MyEnum.First
b = MyEnum.Second(2)
c = MyEnum.Third(val: 3)
```

When pattern match, the fill path to the type doesn't need to named on each case, only the name of each member. Use `_` while destructuring to discard the members data.

```
match a
| First:
    print("first!")
| Second(_):
    print("second!")
| Third{_}:
    print("third!")
```

#### Exceptions (`except`)

Exceptions are similar to enums but used for error handling. See "Error Handling" for more details. Instantiation works the same as enums.

```
MyException :: except =
    OutOfBounds
    DivideByZero(int)
```

Becaues esceptions group together to form custom exception types per `try` block, each member must use their full name or be given an alias. This helps prevent potential name-clashses.

```
try
    risky()!
except OutOfBounds:
    print("Out of bounds!")
except DivideByZero(x):
    print("Can't divide {x} by zero!")
```

#### Prototypes (`proto`)

A `proto` is an abstract interface &mdash; a named contract with no data. It is equivalent to a `virtual class` (C++), `trait` (Rust), or `interface` (Java, TypeScript, etc.) in other languages. Each member is a function, also called a **method**. Methods that have a parameter named `self` at the beginning will be called as methods on the instance of that type, i.e. `self.method(...)`. This is equivalent to saying `typeof[self].method(self, ...)` or `Self.method(self, ...)`. The type of `self` is `Self` which represents the current type implementing this proto. 

```
MyPrototype :: proto =
    speak(self): str
```

#### Implementing (`impl`)

Methods and trait implementations are added separately with `impl`. Much like `proto`, `self` refers to the current instance and `Self` refers to the current type. You can also add static values that are attached to the type itself. Use `.` to access static values and methods like with structs.

```
MyStruct :: impl =
    staticValue = 1234
    init(name: str, value: int): Self =
        MyStruct(name: name, value: value)

print("{MyStruct.staticValue}")
```

To implement a prototype onto another type, you add the proto's name after `impl`. Each implementation gets their own `impl` block. 

```
MyStruct :: impl(MyPrototype) =
    speak(self) =
        "I am a MyStruct \{ name={self.name}, value={self.value} }"

MyEnum :: impl(MyPrototype) =
    speak(self) =
        match self
        case First then
            "I am a MyEnum of First"
        case Second(x) then
            "I am a MyEnum of Second({x})"
        case Third{val} then
            "I am a MyEnum of Third \{ val={val} }"
```

### Inheritance and Visibility

Even though structs cannot be extended the usual way, they can **inherit** from other structs using the `inherit` keyword. This works similar to **importing.** It marks members that map to members of another struct, making conversion possible. It follows the same convention for pattern matching as destructuring. *(See [Destructuring](#Destructuring).)* The struct being inherited must not be `@opaque` or else it's a compile-time error. *(See [Decorators](#Decorators) for more informations on available decorators.)*

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

When you inherit, you don't just pick out some members. The entire parent struct is exists in the child struct in memory, but only some members are visible. 

All members of a type are public by default. When making a subtype, inherited members become private to the subtype unless explicitly redeclared with `inherit`. This encourages separating public and private data into distinct types rather than using access modifiers. If you only inherit some fields but not all, you must put a `_` at the end of the tuple list to mark that not all members are inherited. 

```
PrivateFields :: struct =
    val: int
    secret: int

PublicFields :: struct =
    inherit PrivateFields{val, _}     -- Redeclared, val is `public` and `secret` is private
    other: int
```

A subtype cannot accidentally expose or clash with a private inherited member because types only see members that have been explicitly declared within them. This mirrors the convention used for imports.

## Meta Functions

Adding a parameter before the double colon (`::`) turns it into a **meta function** which combines the concepts of **inline functions**, **macros**, and **generics**. Parameters are put in square brackets `[]` to distinguish them from regular functions which use parentheses `()`. The types of parameters can be inferred based on context. If the meta function has no parameters or the values of those parameters can be inferred based on context, then you don't have to use `[]` when calling it. 

Analogous to constant values, you can define an inline function or macro by adding a parameter before the double colons and writing an expression after it. Like constants, they don't output a value in memory when compiled, useful for collecting repeated code. Unlike constant functions, they cannot be passed to another function. They are only for inserting an expression. Each parameter is a variable within the expression, so you don't need to wrap them in parentheses `()` like with C macros. 

```
max[a, b] :: if a > b then a else b
min[a, b] :: if a < b then a else b
```

You can also have multi-line macros similar to functions. Each meta function creates a new scope. Defining variables that could bleed into the surrounding scope is not allowed. The last expression is the return value. Call it like a function using `[]`. 

```
doSomethingComplicated[x] ::
    x = x + 1
    x = x / 2
    x * x

value = doSomethingComplicated[3]
```

This is the same as this:

```
value =
    x = (3) + 1
    x = x / 2
    x * x
```

The compiler will read the body of the macro and understand where to insert its parameters, so if a parameter gets shadowed, then it will no longer insert it for the rest of that scope. 

You can also pass a type back to make generic types and functions.

```
-- Note that this is not the actual defintion for an option type `type?`. This is just a user-defined enum that uses the same pattern.
Maybe T :: enum =
    Some(T)
    None

Some[T] :: fn(x: T) = Maybe[T].Some(x)

maybeInt = Some(1)
```

Type parameters can be omitted at the call site if they can be fully inferred from the value arguments, in which case the call uses parentheses like a regular function. It can also be called explicitly by making an alias for it or calling with both at the same time `[]()`

```
SomeInt :: Some[int]
maybeInt = SomeInt(1)

maybeInt = Some[int](1)
```

Some more examples using the `[]` notation:

```
max[a, b] :: if a > b then a else b
min[a, b] :: if a < b then a else b
print("{ max[0, 1] }")           -- Prints "1"
print("{ min[0, 1] }")           -- Prints "0"
print("{ max[1+2, 3+4] }")       -- Prints "7"

f(x) = x * x
g(x) = x + 2
maxAdd[a, b] :: if a > b then fn(c) = a + c else fn(c) = b + c
print("{ maxAdd[ f(0), g(0) ] (1) }")
```

The syntax `[]` was chosen so that generic type inferrence will take precedence. `neta(a, b)` means to *call the instantiated function that `neta` returns with inferred types* where as `neta[a, b]` means to *call the abstract function `neta` with these exact values.* This also makes it easy to distinguish actual function calls from macros/inlining.

### Where Block

This is not required for all meta functions but is useful for defining what patterns each parameter is expected to be. It must be the first definition, and any subsequent definitions should have patterns that match the where clause. One common pattern is simply `type` which indicates that a parameter is any literal type. 

```
List[T, N] :: where =
    T: type         -- `type` refers to any literal type, i.e. not a value
    N: int          -- A constant `int` that must be known at compile time

List[T, N] :: struct =
    data: T#N

List[T, N] :: impl =
    init() =
        data: T#N = [++for _ in 0..N then default]
        Self(data: data)
```

### Manual Implementation

Generics will automatically generate code based on their parameters, but you can also implement them by hand using pattern matching. If you only want to use the manual implementations for a generic function, you can set its body to `undefined`. This creates a virtual function that can be overloaded later. If you use a function that is defined as `undefined`, it will throw a compile-time error. *(See [`undefined`](#undefined)* for more details.)

```
-- Forces every type to have its own implementation
increment[T] :: fn(c: ref mu T): void =
    undefined

Counter :: struct =
    value: int

-- Specialized for Counter
increment[Counter] :: fn(c: ref mu Counter): void =
    c.value += 1

-- Specialized for float
increment[float] :: fn(c: ref mu float): void =
    c += 1.0

c = Counter(value: 2)
f = 3.0
b = True

increment(c)   -- T is inferred as Counter
increment(f)   -- T is inferred as float
(--
increment(b)   -- T is inferred as bool which has no implementation, compile-time error
--)
```

---

## Importing and Modules

Use `import` to import something, optionally giving the import an alias with `as`. You can either import a single export such as `import a.b.c` or multiple at once using destructuring rules `import a.b{c, d}`. *(See [Destructuring](#Destructuring).)* Note that there is no `.` before the `{`. This follows the same convention as destructuring with tuples. All imports must be **explicitly** declared&mdash;no `import a.b._`. This helps prevent naming conflicts and track where things have been defined.

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

Mulang is multi-paradigm: different functions, structs, or modules can use different memory strategies in the same program. The model is controlled per-module via decorators. Boundary crossing between models follows FFI-like rules &mdash; automatic marshalling where possible, explicit escapes otherwise.

Modules define how memory is handled with the `@memory` decorator. *(See [Decorators](#Decorators) for more informations on available decorators.)* By default, modules use a garbage collector. Some options include `Collect(GC)` (default),  `Count(ARC)` (reference counting), `Borrow` (borrow checking), and `Manual`. `GC` and `ARC` represent the standard garbage collector and reference counter respectively, but others can be defined and used instead.

```
import std.mem{memory, Count, ARC, _}

@memory(Count(ARC))
mod moduleThatUsesReferenceCounting
```

How this is implemented is outside of the scope of this document. That will be saved for when it's time to make a standard library for Mulang. For now, Mulang will focus on only implementing the garbage collector which will work in both interpretted and compiled mode.

---

## Decorators

There have been a few examples of decorators in this document such as `@opaque` and `@memory`. These are compile-time functions that communicate to the compiler directly and can alter the behavior of things. The API for defining your own decorators is not set in stone yet. More information on them will be available in the future. The syntax for adding decorators goes like this:

```
@decorator1      -- No arguments.
@decorator2(arg) -- With arguments.
expr             -- Modifies whatever this expression is.
```

Decorators can be stacked and will run in reverse order. *Closest decorator to the expression runs, then the next one above that, then the next one, etc.*

Built-in decorators so far include `@capture`, `@opaque`, and `@memory`. More will be added in the future.

```
@memory(Manual) -- Changes what memory model a module uses, default is `Collect(GC)` the garbage collector.
mod myModule

@opaque         -- Marks a type as opaque, it can't be spread or destructured.
Thing :: struct =
    value: int
```

Some other ideas for built-in decorators include:

- `@local` &mdash; locks a symbol to only be used within its module.
- `@static` &mdash; make a variable global but only available within the scope that it was defined in.
- `@inline` &mdash; marks that a regular function should inline itself like a meta function.
- `@pure` &mdash; enforces pure function programming practices: *no `ref mu`, no `capture`, no `out`, etc.*
- `@safe` &mdash; enforces borrow-checking at compile time for this function.
- `@override` &mdash; marks that a previously implemented method will be overridden.

This is a work in progress though. How these decorators are implemented and their API are subject to change.

---

## Design Philosophy

- **Readability first** &mdash; Python-like syntax with significant whitespace and opinionated formatting.
- **Patterns scale with complexity** &mdash; simple things like declaring a mutable variable (`mu`), making a function (`fn`), or wrapping a block (`do`+`end`) use short patterns, more complex things use bigger patterns.
- **Performance on demand** &mdash; start with GC; change to a lower level memory model where necessary.
- **Explicit but ergonomic** &mdash; `!` for errors, attributes for memory models, same keywords used between inline and block expressions.
- **Trace and auditability** &mdash; `import`, `inherit`, and `capture` require variables to be listed out to know where they're coming from; no glob-like imports.
- **Unified concepts** &mdash; `capture` for scope capture; `inherit` for member visibility; `::` for all top-level definitions.
- **Python-developer friendly** &mdash; gradual typing, familiar control flow, no second language or FFI layer required.

---

## Keywords

1. `and`
2. `as`
3. `await`
4. `break`
5. `capture`
6. `case`
7. `continue`
8. `defer`
9. `do`
10. `else`
11. `end`
12. `enum`
13. `except`
14. `fallthrough`
15. `fn`
16. `for`
17. `if`
18. `impl`
19. `import`
20. `inherit`
21. `in`
22. `let`
23. `loop`
24. `match`
25. `mod`
26. `mu`
27. `not`
28. `opt`
29. `or`
30. `out`
31. `pass`
32. `proto`
33. `raise`
34. `ref`
35. `return`
36. `self`
37. `struct`
38. `then`
39. `try`
40. `until`
41. `where`
42. `while`
43. `yield`

*NOTE: Built-in types, values, and functions such as `int`, `void`, `True`, `False`, `Some`, `None`, `Null`, `default`, `undefined` etc. are not considered keywords. `Self` (uppercase) is a type/value and not a keyword, not to be confused with `self` (lowercase) which is a keyword used to signify a method can be used on an instance of a type. See [Implementation](#Implementing-impl).*

---

*This document captures the current state of the Mulang design. The language is still evolving.*
