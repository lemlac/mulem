# Parsing

The following document is focused on how parsing works in Mu.

* __token__: The smallest meaningful unit in an expression.
* __expression__: A collection of tokens, sub-expressions.
* __sequence__: An expression that contains sub-expressions seperated by delimitters.
* __parsing mode__: Detlermines how a sequence will be parsed and what tokens are valid or invalid.
* __parsing stack__: A list of parsing mode sequence pairs and indentations.
* __opener__: A token that opens a new sequence and pushes the sequence and a new parsing mode to the parsing stack.
* __delimiter__: A token that seperates expressions in a sequence.
* __closer__: A token that closes a sequences and pops the current parsing stack off by 1.
* __indentation__: A sequence of space characters (except new-lines) that are at the beginning of a line.

* __block__: A sequence delimited by new-lines.
* __line__: A sequence delimited by semicolons.
* __tuple__: A sequence delimited by commas.

| Parsing Mode | Opener | Delimiters | Closer |
|:--|:--|:--|:--|
| __block parsing__ | Certain keywords at the end of a new line | new-lines | Decreased indentation |
| __line parsing__ | new line | semi-colons | new-line |
| __tuple parsing__ | Brackets `(`/`[`/`{` | commas | Matching closing bracket `)`/`]`/`}` |
| __`do`-parsing__ | `do` | semi-colons | new-line or comma |

When starting off, the parser begins in block parsing mode. Empty expressions are dropped from the sequence. If it finds a semi-colon or comma, it will switch modes for the rest of the line: semi-colon – line parsing, comma – tuple parsing. If in line parsing mode, empty expressions are also dropped. If in tuple parsing mode, empty expressions are only dropped if they are at the end of the sequence, otherwise it's error. Colons `:` at the beginning of a line will apped the expression to the end of the previous sequence in the same mode. 

```
expr
expr

expr;
expr;

expr; expr
; expr;;;

expr, expr

expr, expr,
expr, expr,,,

expr,,, expr   -- Error
, expr, expr  -- Error

expr
: expr
```

If the `do` token appears, a new sequence will be added to the stack. If there's a new line after, it will switch to block parsing until indentation ends. If another token appears, it will switch into line parsing until the end of the line or another delimiter ot closer appears. 

```
do expr; expr
do expr; expr, expr
expr, do expr; expr
expr, do expr; expr, expr
do
    expr
expr; do
    expr
expr, do
    expr
```

If an expression has a comma (`,`) in it, it will start tuple parsing until a new-line or closing bracket, during which semi-colons aren't allowed. Each expression is a commponent in the tuple, starting with the expression before the comma. If a line starts with a comma, it will append to the previous tuple being parsed or start a new one with the expression on the previous line as the first component. 

```
expr, expr  -- Tuple of 2

expr
, expr      -- Tuple of 2

expr, expr
, expr      -- Tuple of 3

expr, expr
, expr, expr
, expr      -- Tuple of 5
```

- `expr, expr, expr` — This line contains a tuple of 3.
- `expr; expr, expr` — This line contains a tuple of 2. First expression is dropped.
- `expr, expr: expr` — Error: semi-colon in tuple expression.

An assignment operator (`=`, `+=`, `-=`) in an expression starts **assignment parsing.** If the left side has a modifier + name or name + colon (`:`) or name + parameter (`()`), it's a **declaration.** Otherwise, the left side is evaluated to determine if it's an expression that returns a reference or an immutable variable. The right side is an expression.

```
name = expr
```

axCommas and brackets start **tuple parsing** until a newline. Semicolons are not allowed during 
