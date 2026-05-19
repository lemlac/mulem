# Mu Language Syntax and Semantics Reference

*Version 0.1 (Draft)*

The Mu programming language or *Mulang* is a general-purpose, multi-paradigm programming language with significant whitespace. It targets Python developers who need C-level performance all within the same language. It is expression-oriented where possible and provides explicit control over evaluation strategy, memory model, and error handling. Another goal is to make it minimalist and have opinionated formatting to make it easier for both humans to read and LLMs to produce without over-consuming tokens.

Mulang will be both a compiled and an interpreted language. Unlike Python, you won't need to use another language like C to increase performance. You can instead compile some of it and then call it as a shared library all within the same language. Some use cases for this are AI, systems programming, and game development.

This document will focus on the language itself. Some features may come in a standard library which will not be discussed here.

---

## Lexical Conventions

- **Indentation**: Significant whitespace (4 spaces recommended). 
- **Statements**: Each statement is divided by new lines. Semi-colons (;) can also be used, but there must be another statement after it&mdash;i.e. no trailing semi-colons. Keyword `end` has a special meaning to end a block.
- **Comments**: Double dash (`--`) to end-of-line. Block comments start with `(--` and end with `--)`.
- **String literals**: `"..."` with interpolation with `{expr}` inside. To write a literal curly brace (`{`), escape it with a backslash `\{`. 

## Basic Syntax

Comments are made with double dash `--`.

```mu
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

A Mu program consists of a list of **expressions** separated by new-lines or inlined with semi-colons (`;`). New-lines and semi-colons are interchangeable and treated the same. 

```mu
expr
expr

expr; expr
```

Almost everything is an expression. Some statements can be either inline or block depending on the presence of a new-line. When mixing the two (i.e. a block statement within an inline statement), the `end` keyword is required to exit block mode and return to inline mode. The last statement evaluated in a block is its value. 

To split one expression into multiple lines, you must wrap it in parentheses. The indentation inside the parentheses doesn't matter as long as it's the same as or greater than the opening parenthesis. Semi-colons (`;`) are a syntax error inside parentheses unless inside a block expression within the parentheses.

```mu
(word1
    word2
   word3
      word4)
```

Adding a newline and indentations after a `:` or `=` starts a block. A **block** wraps multiple expressions into one. The last expression evaluated in a block becomes its value. The most basic block type is `then` which runs a block only once.

```mu
then:
    expr
    expr
```

Indentation marks the end of the block. If a block is inside of another expression and not by itself, then the closing `end` is required and must be at the same indentation as the opening part of the block.

```mu

then:
    then:
        expr
                 -- Ends both blocks.

(then:
    then:
        expr
end)             -- Required here since the block is in parentheses.
```

Some keywords can be inlined or blocked based on whether they use a `:` or not. 

```mu
if x then "True" else "False"  -- Inline expression.

if x:           -- `:` + new line here, so start a block.
    expr        -- Next line should be one or more indentations higher.
else:           -- Unindenting exits the block.
    expr        -- This is a new block.

expr            -- Unindenting exits the block.
```

All subsequent expressions within a block should have the same indentation. If an expression has less indentation than the first but more than the opening of the block, then that's a syntax error. If an expression has more indentation than the first, then it must be in a new block or inside parentheses otherwise, it's also a syntax error.

## Operators

The philosophy of Mulang is that symbols should be easy to understand and that generally keywords are preferred over symbols. Most symbols are consistent with their contextual meaning, for example `*` and `/` relate to math, `~` relates to bitwise operators, `!` relates to exceptions, `&` relates to tuples, etc.. There's also a common pattern where repeating an operator gets you a more advanced version of that operator, for example: `+` addition vs `++` concatenation, `*` multiplication vs `**` exponentiation, `/` division vs `//` floor division, and `%` standard modulo vs `%%` floor division modulo. Mulang will check any combination of symbols greedily until the next space or word, so for example `++` would be different from `+ +`. Spaces are required between multiple symbolic operators, much like how spaces are required between words. Symbol characters include any ASCII character that isn't alphanumeric, whitespace, quotation marks, delimiters, or brackets&mdash;in other words these symbols: `~!@#$%^&*-+=|\:<.>/?`.

**Algebra:**

- `lhs + rhs` &mdash; addition
- `lhs - rhs` &mdash; subtraction
- `- rhs` &mdash; sign-flip
- `lhs * rhs` &mdash; multiplication
- `lhs / rhs` &mdash; division
- `lhs // rhs` &mdash; floored division (rounded down)
- `lhs % rhs` &mdash; modulo (sign matches `lhs`)
- `lhs %% rhs` &mdash; floor division modulo (sign matches `rhs`)
- `lhs ** rhs` &mdash; exponential

**Comparison:**

- `lhs == rhs` &mdash; equality
- `lhs != rhs` &mdash; inequality
- `lhs > rhs` &mdash; greater than
- `lhs < rhs` &mdash; less than
- `lhs >= rhs` &mdash; greater than or equals to
- `lhs <= rhs` &mdash; less than or equals to

**Boolean:**

- `lhs and rhs` &mdash; false if any are false
- `lhs or rhs` &mdash; true if any are true
- `not rhs` &mdash; inverts a boolean

**Bitwise:**

- `lhs ~and rhs` &mdash; bitwise AND
- `lhs ~or rhs` &mdash; bitwise OR
- `lhs ~ rhs` &mdash; bitwise XOR
- `~ rhs` &mdash; bitwise NOT
- `lhs << rhs` &mdash; bitshift left
- `lhs >> rhs` &mdash; bitshift right

**Arrays:**

- `lhs # rhs` &mdash; get an item at an index (starting at 0)
- `lhs ++ rhs` &mdash; concatenation or appendation, always returns a new array
- `++ rhs` &mdash; spread an array or iterator into an array or positional tuple
- `lhs .. rhs` &mdash; creates an iterator that starts at the left value and ends just before the right value (exclusive)
- `lhs ..= rhs` &mdash; creates an iterator that starts at the left value and ends with the right value (inclusive)

**Tuples:**

- `lhs & rhs` &mdash; combine two tuples into one
- `& rhs` &mdash; spread a tuple into another tuple

**Pointers:**

- `@ rhs` &mdash; getting the pointer to a variable
- `lhs ^` &mdash; dereferencing a pointer
- `lhs ^. rhs` &mdash; access a member of a pointer (same as `(^lhs).rhs`)
- `lhs ^= rhs` &mdash; set the value of the slot in memory that a pointer is referencing

*Pointer chaining* &mdash; for any `^` operator, you can add `^` to repeatedly dereference a pointer. 

**Options:**

- `lhs ?` &mdash; returns the `Some` value if it's not `None`, otherwise propagate to the nearest `opt` keyword (see [`opt` block](#opt))
- `lhs ?. rhs` &mdash; gets a method or member of an optional type if it has something, otherwise return `None`
- `lhs ?? rhs` &mdash; fallback to another value if the left side is `None`.

**Results**

- `lhs !` &mdash; returns the result value if it's not an exception, otherwise propagate to the nearest `try` keyword (see [`try` block](#try--except))

Some operators also allow an equal sign after it to set a variable based on its previous value. The left-hand side must be an already defined variable. If it's immutable, then this is the same as shadowing it. If it's mutable, then the value is mutated. All of these are void statements, i.e. they return nothing and should only be used in an expression by themselves.

- `lhs += rhs` &mdash; `lhs = lhs + rhs` &mdash; increment
- `lhs -= rhs` &mdash; `lhs = lhs - rhs` &mdash; decrement
- `lhs *= rhs` &mdash; `lhs = lhs * rhs` &mdash; multiplication assignment
- `lhs /= rhs` &mdash; `lhs = lhs / rhs` &mdash; division assignment
- `lhs //= rhs` &mdash; `lhs = lhs // rhs` &mdash; floor division assignment
- `lhs %= rhs` &mdash; `lhs = lhs % rhs` &mdash; modulo assignment
- `lhs %%= rhs` &mdash; `lhs = lhs %% rhs` &mdash; floor division modulo assignment (binds `lhs` to a range in `[0, rhs)` if `rhs` is positive or `(rhs, 0]` if `rhs` is negative)
- `lhs ++= rhs` &mdash; `lhs = lhs ++ rhs` &mdash; append to an array (not allowed if `lhs` is a fixed length array)

## Basic Bindings

There are two types of bindings: basic `=` and meta `::`. See [Meta Bindings](#Meta-Bindings) below for details about `::`.

### Variable Declarations

Variables are declared with just the equals sign (`=`). Type is inferred, but can be declared with a colon (`:`).

```mu
a = 1
b: Int = 1
```

Variables are immutable, but setting it again shadows it.

```mu
a = 1
a = 2
a = 3
a = "hello"   -- The type of a shadowed variable doesn't have to match.
```

You can also shadow a variable using its previous value.

```mu
i = 0
i = i + 1     -- Sets new `i` based on old `i`
i += 1        -- Same as above
```

Using the single equal-sign is a void statement, so using it within an expression and not on its own is a syntax error. This helps prevent the common bug of using `=` when you meant `==`. For inline binding, use `let () in` or `as`. (See [Inline Binding](#Inline-binding-let-in-and-as).)

```mu
(-- Error:
if x = 0:
    print("x is 0")
--)
-- Do this instead:
if x == 0:
    print("x is 0")
```

#### Mutability (`mu`)

Mutable variables are marked with the keyword `mu`, the main star of the language. Just as Go has its `go` keyword, Mulang has `mu`. This was chosen since mutability is a common practice in programming much like functions are&mdash;which is why functions also get their own two letter keyword `fn`. (See [Function Declarations](#Function-Declarations).) The general rule of thumb in Mulang is that *patterns scale with complexity*&mdash;simple things like declaring a variable or function use short patterns, more complex things use bigger patterns.

Declare a mutable variable with `mu type`. Setting it will change the value instead of shadowing it. The type of value when mutating it should match its original type.

```mu
x: mu Int = 0
x = 1          -- `x` is mutated
```

You can also infer the type with `mu _ = _`:

```mu
mu x = 0
x = 1
```

Or declare the type and set it later:

```mu
x: mu Int
doSomething()
x = 1
```

Not setting a mutable variable implies `= undefined` after it which means it cannot be used until it's been set. The exception is passing undefined variables to the `out` parameters of functions. (See [Function Declarations](#Function-Declarations).)

```mu
x: mu Int = undefined
-- `x` cannot be used here.
(--
doSomething(x)   -- This is an error.
--)

x = 1
-- `x` can be used now.
doSomething(x)   -- This is okay.
```

Assigning to the variable for the rest of the scope and any sub-scopes will mutate the variable. The exception to this is functions which always set a new variable unless its been explicitly captured. (See [Capturing](#Capturing-capture).)

```mu
x: mu Int = 0
then:
    x = 1
print("{x}")   -- 1, x was mutated
cantSetX() =
    x = 2
cantSetX()
print("{x}")   -- still 1, cantSetX didn't change it
```

Since using `=` on a mutable variable mutates it, there's no way to create an immutable variable with the same name until you go out of scope. In other languages, you can say `let _ = _` to declare a new variable in scope, but Mulang abandoned this pattern for the simpler `_ = _` one. It's recommended to create a new variable name if you want to make a mutable variable immutable.

```mu
mu x = 0
then:
    shadowX = x
```

#### References (`ref`/`ref mu`)

A reference type points to the same spot in memory as another variable. It's like a lightweight version of a pointer. (See [Pointers](#Pointers).) Both sides of the equation must be variables, i.e. no references that point to a constant. References are immutable by default unless declared with `ref mu`. The syntax follows the same pattern as `mu`:

* `x: ref T = y` = Immutable reference with explicit type.
* `ref x = y` = Immutable reference with inferred type.
* `x: ref mu T = y` = Mutable reference with explicit type.
* `ref mu x = y` = Mutable reference with inferred type. 

```mu
mu x = 0
ref mu xRef = x
xRef = 1
print("x is {x}")   -- "x is 1"
```

See [Pointers](#Pointers) for more details.

#### Destructuring

Tuples can be split into separate variables. Use parentheses (`()`) for positional tuples and curly braces (`{}`) for named tuples. If a tuple is mixed, split each on the left side with `&` like `() & {}` or `{} & ()`. This is to avoid confliction between the different uses of `:`, type notation on the left of the equal sign and key-value pairing on the right of the equal sign. Use `as` to create an alias for named tuples with type notation going after the alias name. See [Tuples](#Tuples) for more information.

```mu
(a, b) = (0, 1)               -- Basic position destructuring.
(a: Int, b: Int) = (0, 1)     -- Type notation.
{x} = {x: 2}                  -- Named destructuring.
{x: Int} = {x: 2}             -- Named destructuring with type notation.
(a, b) & {x} = (0, 1, x: 2)   -- This tuple has both positional and named components.
(a: Int, b: Int) & {x: Int} = (0, 1, x: 2)
{x as y} = {x: 2}             -- Set the `x` component as `y` in this scope.
{x as y: Int} = {x: 2}        -- Type notation goes after `as`.
```

All components of a tuple should be referenced on the left. Use `_` to explicitly skip components. Also put `_` at the end of a named tuple to skip other keys.

```mu
(_, b, _) & {x, _} = (0, 1, 2, x: 3, y: 4)
```

When destructuring a type that isn't anonymous, the type can optionally be put before the parentheses/braces, otherwise it's automatically inferred. An ampersand (`&`) should be placed at the start of the expression so it won't be confused for a function declaration (see next section). This guarantees that you are destructuring based on the correct type. This follows the same schema as pattern matching in `match`/`case` and `try`/`except`. (See [Control Flow](#Control-Flow).)

```mu
Thing :: {x: Int, y: Int}
thing = Thing(x: 1, y: 2)
&Thing{x, y} = thing       -- Split thing into its components.
```

### Function Declarations

Functions are declared with parentheses before the equals sign. The type of the parameters can be either explicitly typed or inferred based on usage.

```mu
add(a: Int, b: Int): Int = a + b
-- Or inferred:
add(a, b) = a + b

result = add(1, 2)
```

Functions can have multiple lines. The last line evaluated is the return value.

```mu
fib(n) = 
    if n < 1:
        0
    else if n < 2:
        1
    else:
        fib(n - 1) + fib(n - 2)
```

Trailing commas are ignored, but leading commas and double commas are considered a syntax error. 

```mu
add3(a, b, c) = a + b + c
-- Also okay:
add3(a, b, c,) = a + b + c

add3(1, 2, 3,)   -- This is okay.
add(
    1,
    2,
    3,           -- Useful if you list arguments in a block.
)
(--
add3(,1,2, ,3,,) -- This is not okay.
-- Uncommenting would get an error. --)
```

Functions can also be declared with `fn` to be set later. This type is a **function pointer.** It lets you treat functions are variables.

```mu
action: mu fn(Int, Int): Int
add(a, b) = a + b
sub(a, b) = a - b
action = add
print("1 + 1 = {action(1, 1)}")  -- 2
action = sub
print("1 - 1 = {action(1, 1)}")  -- 0
```

#### Mutable / Reference parameters

Function parameters can be declared like variables. Likewise, you can modify their mutability and referenceness the same way.

```mu
increment(x: ref mu Int) =
    x += 1

y = 0
increment(y)
```

What each modifier means changes the functionality and the function's purity:

| Modifiers | What it does |
|:--|:--|
| *nothing* | Copies value, immutable in the function. |
| `mu` | Copies value, mutable within the function. |
| `ref` | Passes reference, immutable in the function. |
| `ref mu` | Passes reference, mutable in the function |

Another type of parameter is `out`. This is like `ref mu` but is treated as `undefined` at the start of the function. Use it to set a variable that hasn't been set yet. The parameter must not be `undefined` in any branch within the function. This means either setting it within the function or passing it to another function with an `out` parameter. This ensures that the variable is set after the function has been called. 

```mu
setInt(out i) =
  i = 3

x: mu Int
setInt(x)
print("{x}")    -- 3
```

This works for mutable variables, but what if you wanted to make an immutable variable using `out`? You can do this by using `out` again while calling a function to declare it as an immutable variable in the current scope. 

```mu
setInt(out n)
print("{n}")    -- 3
```

#### Capturing (`capture`)

Immutable variables can be captured without an issue. If you try to set it within a function, it will get shadowed within the scope of the function. This also includes other functions which are also immutable by default.

For better safety, functions don't capture mutable variables by default. Instead, any variable set inside a function is treated as a new variable. This help prevent accidently mutating a variable that you didn't mean to and encorage functional programming practices.

```mu
x = 1

addFromX(y) = x + y
addFromX2(y, z) = addFromX(y) + z

cannotChangeX(newX) =
    x = newX
    print("{x}")

cannotChangeX(2) -- prints 2
print("{x}")     -- prints 1
```

To capture a mutable variable, you must redeclare it in the function with `capture _`. This helps make it easy to see which functions can mutate other variables and which don't. You can capture multiple variables at once with commas `,`. Each captured variable must be listed. This follows the same practice as `import` and `inherit` where all words in a given context are listed out clearly so that there are no accident name collisions or hidden gotchas.

```mu
mu count = 0
mu squared = 1
mu cubed = 1
addCount() =
    capture count, squared, cubed
    count += 1
    squared = count * count
    cubed = squared * count

addCount()
addCount()
addCount()
print("{count}, {squared}, {cubed}") -- 3, 9, 27
```

#### Lambda Functions

You can define a function within an expression with the keyword `fn` in the pattern `fn(_) = _`. This is useful for passing functions to other functions. If the lambda function has multiple lines, it must terminate with `end`. As mentioned before, the purity of the lambda must match the function that it's being passed to.

```mu
map(array, func) = [++for x in array then func(x)]
array0 = [1, 2, 3, 4]
array1 = map(array0, fn(x) = x + 1)
array2 = map(array0, fn(x) =
    if x < 2:
        x - 1
    else:
        x + 2
end)
```

A name is optional. Adding a name creates an immutable reference of the function itself.

```mu
doThing(fn callback(val) =
    if val > 0:
        callback(val - 1)
    else:
        print("done")
end)
```

#### Named Parameters (`&`)

You can declare a named parameter with `&` and a named tuple after it before the `:` or `=`. The named members are marked in curly brackets (`{}`). Parentheses mark a **positional tuple**, and curly braces mark a **named tuple**. (See [Tuples](#Tuples).) Named tuples can be destructored so that their members become variables in the scope. 

```mu
add() & { a: Int, b: Int }: Int =
    a + b

add(a: 1, b: 2)
```

Named parameters can be defined in their own object and then passed in with the `&` operator as well. The named parameters in the function can also be collected into a single variable using `as`. 

```mu
Options :: { enabled: Bool }
doThing(key: Str) & Options as options =
    if options.enabled:
        Some(callApi(Str))
    else:
        None

options = Options(enabled: true)

doThing("foo", &options)   -- Spreads `options` into the arguments.
```

See [Tuples](#Tuples) for more details on the `&` operator.

### Inline Binding (`let () in` and `as`)

You can also bind variables within an expression using `let () in` and `as`. `let () in` is used for a single expression, where as `as` binds for the rest of the scope.

```mu
squared = let (x = getSomething()) in x * x

while next() as val != None:
    print("{val}")
```

Multiple variables can be declared in a `let () in` expression separated by commas.

```mu
sum = let (x = 1, y = 2) in x + y
```

`as` has the same rules as `=` but returns the value in the expression instead of being void. This means that it can also mutate a mutable variable.

```mu
val: mu Choice
if get() as val != Target:
    raise NotTarget(val)
-- `val` is a `Target` here.
```

---

## Types

- **Basic**: `name: type`
- **Function**: `fn(type, type): type`
- **Option**: `type?`
- **Result**: `type!` or `type!error`
- **Arrays**: `type#` or `type#number`
- **Multi-dimensional Array**: `type##` (an extra `#` for each dimension)
- **Pointers**: `^type`
- **Inferred**: omit the annotation entirely

### Built-in Types

Some built-in types include `Int`, `Uint`, `Float`, `Bool`, `Char`, and `Str`. 

```mu
myInt: Int = -1234
myInt: Uint = 5678
myFloat: Float = 12.34
myBool: Bool = True
myChar: Char = 'a'
myStr: Str = "Hello"
```

You can get the type of any variable with the `typeof` keyword.

```mu
x = 0
y: typeof x = 1    -- Ensures that x and y have the same type.
```

You can also get the default value of any type with the keyword `default`. The type needs to have a default value defined which is yet to be determined how, but they're already defined for basic types.

```mu
x = default Int    -- 0
x = default Float  -- 0.0
x = default Bool   -- False
x = default Char   -- '\0'
x = default Str    -- ""
```

`default` can also infer the type. This can be useful in certain situations, like if you want to leave a function that returns somethng empty so that you can implement it later.

```mu
doSomething(): Int =
    -- TODO: Implement this function.
    default
```

You can also get the size of any type with the keyword `sizeof`. It returns a constant `Uint` (unsigned integer) with the number of bytes of memory that type requires. The exact sizes of some types like `Int` or `Float` might vary, but you can rely on `Char` and `Bool` being 1 byte each. There's also the `void` type which represents no data.

```mu
sizeOfBool = sizeof Bool   -- 1
sizeOfChar = sizeof Char   -- 1
sizeOfVoid = sizeof void   -- 0
```

You can call a type as a function to convert types into other types if conversion is possible. 

```mu
x = 1
y = Float(x)
print("{y}")     -- 1.0
```

#### Booleans

`Bool` is an enum type with its only members being `False` and `True`. This means you can also pattern match with a Bool, although it's recommended to use `if`/`else` instead.

```mu
match value
case True:
    print("It's true!")
case False:
    print("It's false!")
```

That's the same as this.

```mu
if value:
    print("It's true!")
else:
    print("It's false!")
```

#### Strings

Strings can be formatted with curly braces (`{expr}`) in the string. Use a backslash to write a literal opening curly brace (`\{`). Note that string insertion and named tuples both use curly braces. This shouldn't be an issue though since they're used in different contexts. 

```mu
name = "world"  
hello = "Hello, {name}!"
helloEscaped = "Hello, \{name}!"
```

Subsequent string literals will automatically concatenate, and the `++` operator can be used to concatenate non-literal strings.

```mu
str1 = "This" " string"
str2 = " is broken"
str3 = str1 ++ str2 ++ " into multiple parts."
print(str3)
-- Prints "This string is broken up into multiple parts."
```

You can also write multi-line strings with `"""`. A common issue in programming languages is how to fix the issue of leading whitespace in a multi-line string. Mulang uses significant whitespace, so unintenting the string wouldn't work. We don't want all the leading whitespace to be in the string, but how to we solve this? Whitespace is trimmed based on the positions of the last `"""`. Anything before it is automatically trimmed. Buch like blocks, having too little indentation is a syntax error inside multi-line strings. This helps keeps things readable and concisitant and solves the whitespace issue inside string.

```mu
myStr =
    """
    Hello.
    This string has multiple lines.
      This line will start with 2 spaces in the string.
    
    The lines above and below this are empty.

    One quotation mark is fine (").
    Two quotation marks are fine too ("").
    But three qutation marks like \""" need to be escaped.
    This is the last line because of the closing quotation marks below it.
    """
```

You can write a raw string with `''...''` (two apostrophes). Athough apostrophes `'` are used for chars, an empty Char isn't possible since the default Char is written `'\0'` (null character). This is a common practice in programming languages where `"` mark formattable strings and `'` mark raw strings, so this should be easy to understand for anyone. When you write `''`, every character after it (including whitespace and intentation) is in the string until the closing `''`. Escaping with backslashes `\` and insertion with curly braces `{}` are ignored.

```mu
rawString = ''It's okay to put an apostrophe (') in the string.''
filePath = ''C:\files\on\windows.txt''
template = ''Insert here -> {{variable}}''
bigDocument = ''
    This
   is
  all
  in
   a
    string
''
```

> What if you have a raw string with two apostrophes in a row inside it?

That's an edge case not worth persuing. The double apostrophes `''` syntax isn't perfect, but it's at least something. For anything more complicated, it's likely better to save the string to a document and open it as a file, which will be handled by a seperate library and lies outside the scope of this document.

#### Arrays

Array types are declared with the hash symbol (`#`). This was chosen because the `#` is commonly used for numbers in English. For example, `#1` is read as `number 1`. A number after the `#` makes it a fixed length array `type#N`. Arrays are fixed length by default for better performance. Items are separated with commas (`,`). The hash symbol is also used for accessing an array, giving a clear visible mirror between the two. Other languages use `[]` for indexing, but that has another meaning in Mulang which is explained in the proceeding example.

```mu
list: Int#4 = [1, 2, 3, 4]
print("length of list: {len(list)}")
compressedList = [list#0 + list#1, list#2 + list#3]
doubleArray: Int#2#2 = [[1, 2], [3, 4]]
```

To make chaining accesses easier, there's a special rule for square brackets: any operator `op` can be expressed with `a[op b]` which is the same as `((a) op (b))`. This can be used together with the index operator `#` to get an item from a matrix or multi-dimensional array. This helps keep the visual association of `#` for arrays and leaves room for `[]` for other possible uses in the future.

```mu
item = doubleArray[#1][#0]  -- 2nd row, 1st column
print("{item}")             -- "3"
```

One other common operator with arrays is `++` which spreads an array into another array. This was chosen because it's also used for concatenation, giving the two operations an obvious connection. Some languages use `...`, but this visually conflicts with the range `..` operator. `++` visually does the job better without any issues.

```mu
a = [1, 2, 3]
b = [0, ++a, 4]    -- == [0, 1, 2, 3, 4]
c = a ++ b         -- == [1, 2, 3, 0, 1, 2, 3, 4]
```

#### Pointers

Although most things can be achieved without manual manipulation of pointers, some low level code requires it. Pointers are marked with a caret (`^`) before the type, and dereferencing them uses the same symbol. More carets mark how many times you need to dereference it: `^^type` = double pointer, `^^^type` = triple pointer, etc.. Get the reference to a variable with `@`. Pointers are immutable by default, so mutating them isn't allowed.

```mu
x: Int = 0
xPtr: ^Int = @x
```

Note that shadowing a pointer's reference doesn't update the pointer. How the program handles the old reference is up to the memory model. It some cases, it may have already been dropped. (See [Memory Models](#Memory-Models).)

```mu
x = 1
print("{^xPtr}")  -- "0", because it's still referencing the old x.
-- This might lead to a crash.
```

Pointers can also be treated as numbers. The resulted type is `^unknown` by default which means it can't be dereferenced unless it's been casted to a known pointer type. Use `^type(_)` to cast a pointer to a different type. Although pointer arithmetic is possible, it's not recommended because it can easily lead to crashes. This should be reserved for low-level code. 

```mu
x = 0
p = @x
next = ^Int(p + 1)
print("next: {^next}")   -- This will probably crash.
prev = ^Int(p - 1)
print("prev: {^prev}")   -- This will probably crash too.
```

Mutable pointers are marked with `^mu type`. The reference must also be a mutable type. Although it's pointing to a mutable variable, the actual pointer variable itself is immutable. You would need another `mu` before the caret to change the pointer, marked as `mu ^type` or `mu ^mu type`.

```mu
x: mu Int = 0
xPtr: ^mu Int = @x
xPtr ^= 1
print("{x}")     -- "1", x was mutated

y = 2
ptr: mu ^Int = @x
print("{^ptr}")  -- "1", points to x
ptr = @y
print("{^ptr}")  -- "2", points to y
```

Consider this a work in progress. This will need more testing to figure out the best way to handle pointers. Some featues like memory allocation and safe pointers will probably be implemented through a standard library. 

#### Unknown Type

There are some cases where the type can't be infered right away in which case the thing in question gets typed as `unknown`. This usually gets resolved eventually, and if it doesn't, the compiler should throw an error. 

Sometimes with FFI, we don't really know nor care what the actual type to a pointer is. We just know that it's a pointer to something. In that case, we can type it as `^unknown`. This pointer type should always be immutable&mdash;i.e. no `^mu unknown`&mdash;and dereferencing it will throw a compile-time error. 

```mu
result: ^unknown = ExternalLib.getSomething()
ExternalLib.doSomethingWith(result)
```

---

## Control Flow

All branching constructs share the same block / inline pattern:

```mu
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

#### `pass`

The keyword `pass` can be put into any body to leave it empty. This might result in a compile-time error when the block is expected to return a value. 

```mu
keyword[ subject]:
    pass
```

#### `undefined`

This marks that something should not be used. If the compiler detects that a code is using an `undefined` somewhere, it throws an error.

```mu
unusedFn() =
    undefined

-- `unusedFn` cannot be used.
```

#### `then`

Marks a block of code with its own scope that runs only once.

```mu
x = 0
then:
   x = 1
   print("{x}")  -- "1", `x` inside the `then` block
print("{x}")     -- "0", `x` outside the `then` block
```

#### `if` / `else`

Basic boolean branching.

```mu
x = if x > 0 then x else -x

if x > 0:
    print("positive")
else:
    print("non-positive")
```

#### `match` / `case`

Enum/exception branching. Exhaustive by default. `case _` for the default case. The indentation of each `case` must be the same as the starting `match`, ending any debate on whether `case` should be indented or not&mdash;Mulang is opinionated after all. This is done not only to save indentation but also to stay consistant. Because `match _` doesn't use `:` or `then`, it logically shouldn't indent the next line.

The patterns match to the type of enum passed to `match`, so you only need to reference the members of that type in each `case` block.

```mu
match self
case First:
    print("First")
case Second(x):
    print("Second({x})")
case Third{val}:
    print("Third {{ val={val} }}")

-- Inline form
message = (match e case OpenError { filename } then "Open error: {filename}" case _ then "Unknown error")
```

#### `if case`

This combines `match` and `if` into one expression. Useful if you just want to handle a single case.

```mu
if case Pattern(x) = value:
    print("value is {x}")
else:
    print("value doesn't match")
```

#### `for` / `in`

Iterates through an array or iterator.

```mu
for x in list:
    print("{x}")
```

If inlined, it returns an iterator which will lazily execute when spread with `++` or passed into another `for _ in` loop. 

```mu
iterator = for x in list then x * 2
list = [++iterator]
```

For an async iterator or an array of async types, you can use `for await` to automatically wait for each item in sequential order. (See [`await`](#await).)

```mu
for await x in asyncIter():
    print("{x}")
```

#### `while`

Repeats a block of code until the condition is `True`.

```mu
while cond:
    body
```

You can also do `while case` just like with `if case`.

```mu
while case Some(x) = nextValue():
    print("value is {x}")
```

#### `loop` / `until`

Repeats a block of code until `break` is called.

```mu
loop:
    print("I'm looping!")
    break
```

Add `until` after a `loop` block to create a condition that will break the loop. Unlike `while`, this will break when the condition is `True`, analogous to `if cond then break`. The loop will run at least once before checking the condition.

```mu
loop:
    body
until cond
```

#### `break` / `continue`

Controls the iteration of any loop type mentioned. `break` exits out of the loop, and `continue` skips to the next iteration.

#### `opt`

Wraps a value in a option type. You can use `?` to unwrap multiple option types within an expression. If one `?` returns `None`, then the whole expression stops and returns `None`.

```mu
x = (opt f(a?))?? "fallback"     -- x is "fallback" if a is none.
```

If a function returns an option type, then use of the `?` is allowed with `opt`. Using an `?` inside a function will automatically infer the return type as an option. The return will be automatically wrapped in `Some(_)`.

```mu
addStuff(a: Int, b: Int): Int? =
    a = getSomething(a)?
    b = getSomething(b)?
    a + b
```

Option types automatically flatten in the following manner:

- `Some(Some(_))` = `Some(_)`
- `Some(None)` = `None`

#### `try` / `except`

Result types are unwrapped with an exclamation mark (`!`) within a try block.

```mu
safeResult = try divide(1, 0)! except _ then 0.0

try:
    risky()!
except e:
    print("{e}")
```

Pattern matching works in the `except` clause like in `case`. If a pattern is missing, the exception is raised to the next block above it. The type of exception in the last `except e then` block is all the possible exceptions minus the caught exceptions. 

```mu
try:
    risky()!
except Exception(e):
    print("Exception {e}")
(-- implied:
except e:
    raise e
--)
```

If a function returns a result type `type!`, then it can also use the `!` like in a `try` block. Use of a `!` in a function automatically infers a result type as the return.

```mu
riskyFunction(a: Int): Int! =
    b = doSomething1(a)!
    c = doSomething2(b)!
    c
```

You can combine `?` and `!` together when the return type is `type?!` (an **option result type**).

```mu
-- With type notation:
doSomething(x: Str?): Str?! =
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

```mu
isThirteen(x) =
    if x == 13:
        return True  -- Exits the function and returns true.
    False            -- Returns false.
```

#### `raise`

Return out of the function with an exception value. The function must return a result type `type!`. If an except type is also declared `type!except`, then the type passed to `raise` must match.

```mu
alwaysFail() =
    raise MyError("error message")

try:
    alwaysFail()!
except e:
    print("{e}")
```

#### `yield`

Exits out of a function with an `Iter` type. The return value of the function must be of type `Iter T` where T is the yield type. When you have `yield` in your function, the actual return value in the function body is discarded, and using `return _` in it is a compile-time error. Use of `yield` will infer the return type as `Iter T`. 

```mu
count(n: Int): Iter Int =
    for i in 0..n:
        yield i
```

#### `await`

Exits out of a function with an `Async` type. The return type of the function must be of type `Async T` where T is the type that the async will return in the end. The return value of the async instance is determined the same way as a non-async function. Use of `await` will infer the return type as `Async T`. 

```mu
asyncFn(a, b): Async Int =
    a = await fetch(a)
    b = await fetch(b)
    a + b
```

Both `yield` and `await` can be used together in an `Iter Async T` type. As the type suggests, each yield is of type `Async T`. Use `for await` to wait for each async value to resolve in sequential order.

```mu
asyncIterFn(n): Iter Async Int =
    for i in 0..n:
        val = await fetch(i)
        yield val

asyncCollect(n): Async Int# =
    ret: mu Int# = []
    for await x in asyncIterFn(n):
        ret ++= x
    ret
```

---

## Error Handling

- `expr!` &mdash; propagate an error upward (Rust/Swift style).
- Exceptions are sum types; the compiler unions all possible exception types from every `!` site in a `try` block.
- `try` / `except` can be an expression or a block and must unify return types (like `if` / `else`).
- Pattern matching on errors works with `match` or `if case Pattern(x) = value`.

## Pattern Matching and Destructuring

- Full `match` / `case` (exhaustive unless `case _` is present).
- `if case Pattern(x) = value then` &mdash; like Rust's `if let`.
- Destructuring supports structs, tuples, enum variants, and wildcards (`_`).

---

## Meta Bindings (`::`)

Variable and functions primarily use the equals sign (`=`) and are for storing actual data within a program, but there's another type of binding used for abstract values for the compiler to know about such as constants, types, inline-functions, and generics. This type of declaration is constant; in other words, they cannot be mutated or shadowed. However, depending on what it is, subsequent `::` of the same name will modify its definition. 

### Constants

Putting a constant value after `::` creates a constant. This holds an unchangeable value that must be known at compile time. Explicit typing isn't necessary since it cannot be changed.

```mu
PI :: 3.1415926535
```

You can also bind a function to a constant. When calling it, it would be the same as defining it inline and then calling. This can be useful if you need to pass a function multiple times but don't want it to be outputted when compiled.

```mu
IDENTITY :: fn(x) = x
addOne :: fn(x) = x + 1
value = addOne(2)               -- Same as (fn(x) = x + 1)(2), result is 3.
array = map([1, 2, 3, 4], addOne)
```

### Aliases

Assigning a type after `::` creates an alias

```mu
numberType :: Int
```

This alias is unique to the scope. Modifying it only affects the alias and not the original type. This prevents accidental conflictions between modules. (See [Implementation](#Implementing-impl).)

You can also create aliases for basic product types or sum types.

```mu
tuple        ::  Int , Float , Char                    -- Also called a "positional tuple".
alsoTuple    :: (Int , Float , Char)                   -- Optional parentheses.
namedTuple   :: {count: Int, scale: Float, code: Char} -- Position not guaranteed.
mixedTuple   :: (Int, Float) & {code: Char}            -- Has both positional and named components.
productUnion ::  Int & Float & Char                    -- Is the size of all types combined.
sumUnion     ::  Int | Float | Char                    -- Is the size of the largest type.
```

#### Tuples

Every opaque type by itself is its own tuple, so for example `Char` and `(Char)` are the same. This means that in the example, `Int & Float & Char` is the same as `(Int, Float, Char)`.

Tuples use commas (`,`) to separate components for both positional (`()`) and named (`{}`) tuples. This follows the same rules as function parameters. (See [Function Declarations](#Function-Declarations).)

Product unions with the `&` operator can be used for both types and values. When combining two or more positional tuples, the positions of subsequent tuples get bumped up by the number of positions in the previous tuples, i.e. `(a, b) & (c, d)` becomes `(a, b, c, d)`. When you combine two or more named tuples, conflicting named parameters override each other with the last tuple taking priority&mdash;much like how shadowing works. So if you have `{x: 1} & {x: 2}`, the result is just `{x: 2}` since it overrides the `x` of the previous tuple. Positional tuples and named tuples can be combined together for example `(0, 1) & {x: 2}`. The shorthand for this is to write named parameters in a positional tuple like `(0, 1, x: 2)`. 

You can think of it as every tuple always having both dimensions, just with most slots empty:

```mu
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

Opaque types such as primitives and enums coerce into a tuple of one, so creating a product type of them creates a positional tuple, e.g. `Int & Float & Char` becomes `(Int, Float, Char)`. Structs convert to named tuples unless declared with an `@opaque` decorator, in which case they behave like opaque types. The `void` type coerces to an empty tuple `()`, and combining empty tuples produces an empty tuple `() & () == ()`. The same is true for empty named tuples `{} & {} == {}`. This also means that empty positional tuples and empty named tuples are equivalent `() == {}`. `void` itself is treated as an equivalent of these types: `void == ()` and `void == {}`.

### Structures (`struct`)

Structs are product types&mdash;or in other words&mdash;plain data containers. They cannot extend other structs, but can inherit members of other structs (see [Inheritance and Visibility](#Inheritance-and-Visibility)).

```mu
MyStruct :: struct
    name: Str
    value: Int
```

Instantiate a struct by calling it like a function. Each member is treated as a named argument.

```mu
myObject = MyStruct(name: "Foobar", value: 1)
```

Structs are transparent. They can be destructured like named arrays. Use `@opaque` if you need to disable this. That will treat the struct type as an opaque type. This will also prevent it from being inheritted with `inherit` since that relies on destructuring. (See [Inheritance and Visibility](Inheritance-and-Visibility).)

```mu
TransparentThing :: struct
    a: int
    b: int

{a, b} = TransparentThing(a: 1, b: 2)
print("a: {a}, b: {b}")

@opaque
OpaqueThing :: struct
    a: int
    b: int

o = OpaqueThing(a: 1, b: 2)
print("a: {o.a}, b: {o.b}")
```

### Enumerables (`enum`)

Enums are sum types. They define a closed set of variants. Variants may carry data turning them into a tagged union.

```mu
MyEnum :: enum
    First
    Second(Int)
    Third{val: Int}
```

Like structs, instantiate by calling the member as a function unless it doesn't carry any data.

```mu
a = MyEnum.First
b = MyEnum.Second(2)
c = MyEnum.Third(val: 3)
```

### Exceptions (`except`)

Exceptions are similar to enums but used for error handling. See "Error Handling" for more details. Instantiation works the same as enums.

```mu
MyException :: except
    OutOfBounds
    DivideByZero(Int)
```

### Prototypes (`proto`)

A `proto` is an abstract interface &mdash; a named contract with no data. It is equivalent to a `virtual class` (C++), `trait` (Rust), or `interface` (Java, TypeScript, etc.) in other languages. Each member is a function, also called a **method**. Methods that have a parameter named `self` at the beginning will be called as methods on the instance of that type, i.e. `instance.method(...)`. This is equivalent to saying `(typeof instance).method(instance, ...)`. `self` is inferred to be type of `Self` which represents the current type implementing this proto. 

```mu
MyPrototype :: proto
    speak(self): Str
```

### Implementing (`impl`)

Methods and trait implementations are added separately with `impl`. Much like `proto`, `self` refers to the current instance and `Self` refers to the current type. You can also add static values that are attached to the type itself. Use `.` to access static values and methods like with structs.

```mu
MyStruct :: impl
    staticValue = 1234
    init(x: Int, y: Int): Self =
        MyStruct(x: x, y: y)

print("{MyStruct.staticValue}")
```

To implement a prototype onto another type, you add the proto's name after `impl`:

```mu
MyStruct :: impl MyPrototype
    speak(self) =
        "I am a MyStruct {{ x={self.x}, y={self.y} }}"

MyEnum :: impl MyPrototype
    speak(self) =
        match self
        case First then
            "I am a MyEnum of First"
        case Second(x) then
            "I am a MyEnum of Second({x})"
        case Third{val} then
            "I am a MyEnum of Third {{ val={val} }}"
```

## Inheritance and Visibility

Even though structs cannot be extended the usual way, they can inherit from other structs using the `inherit` keyword. This works similar to importing. It marks members that map to members of another struct, making conversion possible. It follows the same convention for pattern matching as destructuring. (See [Destructuring](#Destructuring).)

```mu
Vector2 :: struct
    x: Float
    y: Float

Vector3 :: struct
    inherit Vector2{x, y}
    z: Float

v3 = Vector3(x: 1.0, y: 2.0, z: 3.0)

radius2d(v: Vector2) = sqrt(v.x*v.x+v.y*v.y)
print("{radius2d(v3)}") -- This works because Vector3 inherits from Vector2.
```

When you inherit, you don't just pick out some members. The entire super type is inherited, but only some members are visible. 

All members of a type are public by default. When making a subtype, inherited members become private to the subtype unless explicitly redeclared with `inherit`. This encourages separating public and private data into distinct types rather than using access modifiers. If you only inherit some fields but not all, you must put a `_` at the end of the tuple list to mark that not all members are inheritted. 

```mu
PrivateFields :: struct
    val: Int
    secret: Int

PublicFields :: struct
    inherit PrivateFields{val, _}     -- Redeclared, val is `public` and `secret` is private
    other: Int
```

A subtype cannot accidentally expose or clash with a private inherited member because types only see members that have been explicitly declared within them. This mirrors the convention used for imports.

## Meta Functions

Adding a parameter before the double colon (`::`) turns it into a **meta function** which combines the concepts of **inline functions**, **macros**, and **generics**. Parameters are divided with spaces like in functional programming languages such as Haskell or OCaml. The values of parameters can sometimes be inferred based on context. 

Analogous to constant values, you can define an inline function or macro by adding a parameter before the double colons and writing an expression after it. Like constants, they don't output a value in memory when compiled, useful for collecting repeated code. Unlike constant functions, they cannot be passed to another function. They are only for inserting an expression. Each parameter is a variable within the expression, so you don't need to wrap them in parentheses `()` like with C macros. 

```mu
max a b :: if a > b then a else b
min a b :: if a < b then a else b
```

You can also have multi-line macros similar to functions. Each meta function creates a new scope. Defining variables that could bleed into the surrounding scope is not allowed. The last expression is the return value. Call it like a function using `.()`. 

```mu
doSomethingComplicated x ::
    x = x + 1
    x = x / 2
    x * x

value = doSomethingComplicated.(3)
```

This is the same as this:

```mu
value =
    x = (3) + 1
    x = x / 2
    x * x
```

The compiler will read the body of the macro and understand where to insert its parameters, so if a parameter gets shadowed, then it will no longer insert it for the rest of that scope. 

You can also pass a type back to make generic types and functions.

```mu
Option T :: enum
    Some(T)
    None

Some T :: fn(x: T) = Option.(T).Some(x)

maybeInt = Some(1)
```

Type parameters can be omitted at the call site if they can be fully inferred from the value arguments, in which case the call uses parentheses like a regular function. It can also be called explicitly by making an alias for it.

```mu
SomeInt :: Some Int
maybeInt = SomeInt(1)
```

When used as a type, only spaces are needed to delimit arguments just like how it's written when being defined. This is the recommend way of writing meta functions. Sometimes you need to call them in an expression instead of as type notation. In that cause, you need to use `.()` to call it like you would for a normal function. This is awkward in most cases, so it's recommended to use type inference instead. 

```mu
maybeInt = Some.(Int)(1)
```

Some more examples using the `.()` notation:

```mu
max a b :: if a > b then a else b
min a b :: if a < b then a else b
print("{ max.(0, 1) }")           -- "1"
print("{ min.(0, 1) }")           -- "0"
print("{ MAX.(1+2, 3+4) }")       -- "7"

maxAdd a b :: if a > b then fn(c) = a + c else fn(c) = b + c
print("{ ( maxAdd.(f(0), g(0))(1) }")
```

### Where Block

This is used to define what each parameter's type is for an abstract function. It must be the first definition, and any subsequent definitions should have patterns that match the where clause. 

```mu
List T N :: where
    T: type       -- `type` refers to any literal type, i.e. not a value
    N: Int        -- A constant `Int` that must be known at compile time

List T N :: struct
    data: T#N

List T N :: impl
    init() =
        data: T#N = [++for _ in 0..N then default]
        Self(data: data)
```

### Manual Implementation

Generics will automatically generate code based on their parameters, but you can also implement them by hand using pattern matching. If you only want to use the manual implementations for a generic function, you can set its body to `undefined`. This creates a virtual function that can be overloaded later. If you use a function that is defined as `undefined`, it will throw a compile-time error.

```mu
-- Forces every type to have its own implementation
increment T :: fn(c: ref mu T): void =
    undefined

Counter :: struct
    value: Int

-- Specialized for Counter
increment Counter :: fn(c: ref mu Counter): void =
    c.value += 1

-- Specialized for Float
increment Float :: fn(c: ref mu Float): void =
    c += 1.0

c = Counter(value: 2)
f = 3.0
b = True

increment(c)   -- T is inferred as Counter
increment(f)   -- T is inferred as Float
(--
increment(b)   -- T is inferred as Bool which has no implementation, compile-time error
--)
```

---

## Importing and Modules

Use `import _` to import something, optionally giving the import an alias with `as`. You can either import a single export such as `import a.b.c` or multiple at once using destructuring rules `import a.b{c, d}`. (See [Destructuring](#Destructuring).) All imports must be implicitly declared&mdash;no `import a.b.*`. This helps prevent naming conflicts and track where things have been defined.

The most common import will likely be the `print` function, which will be defined somewhere in a standard library.

```mu
import std.print     -- This is just an example and not final.

print("Hello, world!")
```

Modules are named with the keyword `mod` near the top before anything is defined. This is the name you'll use when importing your module. 

```mu
import somewhere{thing}

mod myModule

addThing(x) = x + thing
```

In this example, you would import `addThing` like this:

```mu
import myModule.addThing
```

### Memory Models

Mulang is multi-paradigm: different functions, structs, or modules can use different memory strategies in the same program. The model is controlled per-module via decorators. Boundary crossing between models follows FFI-like rules &mdash; automatic marshalling where possible, explicit escapes otherwise.

Modules define how memory is handled with the `@memory` decorator. By default, modules use a garbage collector. Some options include `Collect(GC)` (default),  `Count(ARC)` (reference counting), `Borrow` (borrow checking), and `Manual`. `GC` and `ARC` represent the standard garbage collector and reference counter respectively, but others can be defined and used instead.

```mu
import std.mem{memory, Count, ARC, _}

@memory(Count(ARC))
mod moduleThatUsesReferenceCounting
```

How this is implemented is outside of the scope of this document. That will be saved for when it's time to make a standard library for Mulang.

---

## Design Philosophy

- **Readability first** &mdash; Python-like syntax with significant whitespace and opinionated formatting.
- **Patterns scale with complexity** &mdash; simple things like declaring a variable or function use short patterns, more complex things use bigger patterns.
- **Performance on demand** &mdash; start with GC; change to a lower level memory model where necessary.
- **Explicit but ergonomic** &mdash; `!` for errors, attributes for memory models, same keywords used between inline and block expressions.
- **Trace and auditability** &mdash; `import`, `inherit`, and `capture` require variables to be listed out to know where they're coming from; no glob-like imports.
- **Unified concepts** &mdash; `capture` for scope capture; `inherit` for member visibility; `::` for all top-level definitions.
- **Python-developer friendly** &mdash; gradual typing, familiar control flow, no second language or FFI layer required.

---

## Keywords

There are 47 keywords in total:

* `and`
* `as`
* `await`
* `break`
* `capture`
* `case`
* `continue`
* `default`
* `else`
* `end`
* `enum`
* `except`
* `fn`
* `for`
* `if`
* `inherit`
* `impl`
* `import`
* `in`
* `let`
* `loop`
* `match`
* `mod`
* `mu`
* `not`
* `out`
* `opt`
* `or`
* `pass`
* `proto`
* `raise`
* `ref`
* `return`
* `self`/`Self`
* `sizeof`
* `struct`
* `then`
* `try`
* `type`
* `typeof`
* `undefined`
* `unknown`
* `until`
* `void`
* `where`
* `while`
* `yield`

---

*This document captures the current state of the Mulang design. The language is still evolving.*
