# Parsing

The following document is focused on how parsing works in Mu.

When starting off, the parser begins with **block-level parsing.** Semi-colons (`;`) and new-lines delimit expressions. This forms an **expression sequence.** Empty expressions are dropped from the sequence.

```
expr
expr

expr; expr
```

- A **line** is any sequence of tokens until a new line character.
- An **expression** is sequence of tokens between delimitters, openers and closera.
- A **delimitter** is a token that separates expressions which changes depending on the parsing mode.
- An **opener** is a token that switching parsing mode
- A **closer** is a token that exits a parsing mode back to the parsing mode before it.

If an expression has a comma (`,`) in it, it will start **tuple parsing** until a new-line or closing bracket, during which semi-colons aren't allowed. Components are delimitted with commas — the previous expression being the first component. If a line starts with a comma, it will use the last expression above it as the first component. Multiple lines starting with comma will chain together to the same tuple.

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
