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

__Parsing Modes:__

* _Normal:_
  * __block__
  * __block-line__
  * __bracket__
  * __do-line__
* _Special:_
  * __string__
  * __comment__

When starting off, the parser begins in block parsing mode with the root sequence. The starting indentation must be zero at the first non-empty line.

__While in a normal parsing mode:__\
If a comment opener appears — switch to comment mode.\
If a string opener appears — switch to string mode.

If a line starts with a colon `:`, all previous whitespace will be ignored. The sequence will continue from the previous one.

Each sequence in normal parsing mode *(except for the root sequence)* has a **bracket parent.** For bracket sequences, this is a self-reference. For non-bracket sequences, this is the same as their parents' bracket parent. A bracket sequence starts when there's an opening bracket `(`/`[`/`{`. Whenever a closing bracket `)`/`]`/`}` appears, it must match the barcket parent of that sequence. If it does, all sequences with that bracket parent will close and the parser continues at the parent of that bracket sequence. If it doesn't, an error will be thrown and parsing ends.

### Block Parsing Mode

Each block starts with indentation, the root block having an indentation of 0. Expressions in this sequence must have matching indentation. A line with less indentation ends the block. 

While parsing each line, if the parser finds a semi-colon or comma, it will switch to block-line mode for the rest of the line.

### Block-Line Parsing Mode

Block-line sequences can have either semi-colons or commas as delimiters, but not both. New-lines closes the sequence, as well as other normal parsing closers.

While in block or block-line mode – if the `do` token appears, a new sequence will be added to the stack.

* If there's a new-line after `do` — it will switch to block parsing with increased indentation.
* If another token appears — it will switch into **do-line** parsing until its closer appears.

__While in a comment mode:__\
Comment mode has two modes: **line** and **block**. Comment mode also carries an index of how many nested block comments are in it.

Opener:
* `--` — line comment mode
* `(--` — block comment mode

* While in line comment mode, all tokens until a new-line are consumed.
* While in block comment mode:
  * Start with nested-block-comments of 1.
  * An opening comment token `(--` appears: nested-block-comments increment by one.
  * A closing comment token `--)` appears: nested-block-comments decrement by one.
  * Block comment ends when nested-block-comments reaches 0.

When a comment finishes, the parser goes back to its parent and removes the comment from the sequence.

## Do-Line Mode

A do-line sequence starts with `do` + a non-new-line token and takes any expression delimitted by semi-coloms `;` until it reaches its closer: comma, new-line, or parent bracket closer. 

```
do a; b; c        -- 1 do-line sequence of 3
x, do w; y, z     -- 1 open-tuple of 3, 2nd expression is a do-line sequence of 2
```

If another do-line sequence starts while in do-line mode, it will flatten to the current sequence. Flattening only happens when the immediate top of the parsing stack is a do-line sequence. 

```
do a; do b; do c     -->  do a; b; c
do a; (do b; do c)   -->  do a; (do b; c)
```

```
x; y; z       -- valid
x, y, z       -- valid
x; y, y       -- invalid
x, y; z       -- invalid
do x; y, z    -- valid
x, do y; z    -- valid
do x, y; z    -- invalid
```

- **Block line rule**: track delimiter type (unset, `;`, or `,`). First delimiter sets it. Mismatch is an error.
- **Do-line**: a `;`-delimited sub-sequence, closed by `,` / newline / bracket closer. Can appear as an item at any level.
