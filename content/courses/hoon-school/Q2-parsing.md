# Q2-parsing

+++\
title = "17. Text Processing III"\
weight = 27\
nodes = \[283, 383]\
objectives = \["Tokenize text simply using `find` and `trim`.", "Identify elements of parsing: `nail`, `rule`, etc.", "Use `++scan` to parse `tape` into atoms.", "Construct new rules and parse arbitrary text fields."]\
+++

_This module covers text parsing. It may be considered optional and_\
_skipped if you are speedrunning Hoon School._

We need to build a tool to accept a \{% tooltip label="tape"\
href="/glossary/tape" /%\} containing some characters, then turn it into\
something else, something computational.

For instance, a calculator could accept an input like `3+4` and return`7`. A command-line interface may look for a program to evaluate (like\
Bash and `ls`). A search bar may apply logic to the query (like Google\
and `-` for `NOT`).

The basic problem all parsers face is this:

1. You need to accept a character string.
2. You need to ingest one or more characters and decide what they\
   “mean”, including storing the result of this meaning.
3. You need to loop back to #1 again and again until you are out of\
   characters.

### The Hoon Parser

We could build a simple parser out of a \{% tooltip label="trap"\
href="/glossary/trap" /%\} and \{% tooltip label="++snag"\
href="/language/hoon/reference/stdlib/2b#snag" /%\}, but it would be\
brittle and difficult to extend. The Hoon parser is very sophisticated,\
since it has to take a file of ASCII characters (and some UTF-8 strings)\
and turn it via an AST into \{% tooltip label="Nock"\
href="/glossary/nock" /%\} code. What makes parsing challenging is that\
we have to wade directly into a sea of new types and processes. To wit:

* A

is the string to\
be parsed.

* A `hair` is the position in the text the parser is at, as a cell of\
  column & line, `[p=@ud q=@ud]`.
* A `nail` is parser input, a cell of `hair` and `tape`.
* An `edge` is parser output, a cell of `hair` and a `unit` of `hair`\
  and `nail`. (There are some subtleties around failure-to-parse here\
  that we'll defer a moment.)
* A `rule` is a parser, a gate which applies a `nail` to yield an`edge`.

Basically, one uses a `rule` on `[hair tape]` to yield an `edge`.

A substantial swath of the standard library is built around parsing for\
various scenarios, and there's a lot to know to effectively use these\
tools. **If you can parse arbitrary input using Hoon after this lesson,**\
**you're in fantastic shape for building things later.** It's worth\
spending extra effort to understand how these programs work.

There is a [full guide on parsing](../../../language/hoon/guides/parsing/) which\
goes into more detail than this quick overview.

### Scanning Through a `tape`

parses\
a \`tape\` or crashes, simple enough. It will be our workhorse. All we\
really need to know in order to use it is how to build a \`rule\`.

Here we will preview using \{% tooltip label="++shim"\
href="/language/hoon/reference/stdlib/4f#shim" /%\} to match characters\
with in a given range, here lower-case. If you change the character\
range, e.g. putting `' '` in the `++shim` will span from ASCII `32`, `' '` to ASCII `122`, `'z'`.

```hoon
> `(list)`(scan "after" (star (shim 'a' 'z')))  
~[97 102 116 101 114]  

> `(list)`(scan "after the" (star (shim 'a' 'z')))
{1 6}  
syntax error  
dojo: hoon expression failed
```

### `rule` Building

The `rule`-building system is vast and often requires various components\
together to achieve the desired effect.

#### `rule`s to parse fixed strings

*   takes\
    in a single \`char\` and produces a \`rule\` that attempts to match that\
    \`char\` to the first character in the \`tape\` of the input \`nail\`.

    ```hoon
    > ((just 'a') [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ [p='a' q=[p=[p=1 q=2] q="bc"]]]]
    ```
*   matches\
    a \`cord\`. It takes an input \`cord\` and produces a \`rule\` that\
    attempts to match that \`cord\` against the beginning of the input.

    ```hoon
    > ((jest 'abc') [[1 1] "abc"])
    [p=[p=1 q=4] q=[~ [p='abc' q=[p=[p=1 q=4] q=""]]]]

    > ((jest 'abc') [[1 1] "abcabc"])
    [p=[p=1 q=4] q=[~ [p='abc' q=[p=[p=1 q=4] q="abc"]]]]

    > ((jest 'abc') [[1 1] "abcdef"])
    [p=[p=1 q=4] q=[~ [p='abc' q=[p=[p=1 q=4] q="def"]]]]
    ```

    (Keep an eye on the structure of the return `edge` there.)
*   parses\
    characters within a given range. It takes in two atoms and returns a \`rule\`.

    ```hoon
    > ((shim 'a' 'z') [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ [p='a' q=[p=[p=1 q=2] q="bc"]]]]
    ```
*   is\
    a simple \`rule\` that takes in the next character and returns it as the parsing result.

    ```hoon
    > (next [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ [p='a' q=[p=[p=1 q=2] q="bc"]]]]
    ```

#### `rule`s to parse flexible strings

So far we can only parse one character at a time, which isn't much\
better than just using \{% tooltip label="++snag"\
href="/language/hoon/reference/stdlib/2b#snag" /%\} in a \{% tooltip\
label="trap" href="/glossary/trap" /%\}.

```hoon
> (scan "a" (shim 'a' 'z'))  
'a'  

> (scan "ab" (shim 'a' 'z'))  
{1 2}  
syntax error  
dojo: hoon expression failed
```

How do we parse multiple characters in order to break things up\
sensibly?

*   will\
    match a multi-character list of values.

    ```hoon
    > (scan "a" (just 'a'))
    'a'

    > (scan "aaaaa" (just 'a'))
    ! {1 2}
    ! 'syntax-error'
    ! exit

    > (scan "aaaaa" (star (just 'a')))
    "aaaaa"
    ```
*   takes\
    the \`nail\` in the \`edge\` produced by one rule and passes it to the\
    next \`rule\`, forming a \{% tooltip label="cell" href="/glossary/cell"\
    /%\} of the results as it proceeds.

    ```hoon
    > (scan "starship" ;~(plug (jest 'star') (jest 'ship')))
    ['star' 'ship']
    ```
*   tries\
    each \`rule\` you hand it successively until it finds one that works.

    ```hoon
    > (scan "a" ;~(pose (just 'a') (just 'b')))
    'a'

    > (scan "b" ;~(pose (just 'a') (just 'b')))
    'b'

    > (;~(pose (just 'a') (just 'b')) [1 1] "ab")
    [p=[p=1 q=2] q=[~ u=[p='a' q=[p=[p=1 q=2] q=[i='b' t=""]]]]]
    ```
*   parses\
    a delimiter (a \`rule\`) in between each \`rule\` and forms a \{% tooltip\
    label="cell" href="/glossary/cell" /%\} of the results of each\
    non-delimiter \`rule\`. Delimiters representing each symbol used in\
    Hoon are named according to their \[aural ASCII]\(/glossary/aural-ascii)\
    pronunciation. Sets of characters can also be used as delimiters, such\
    as \`prn\` for printable characters (\[more\
    here]\(/language/hoon/reference/stdlib/4i)).

    ```hoon
    > (scan "a b" ;~((glue ace) (just 'a') (just 'b')))  
    ['a' 'b']

    > (scan "a,b" ;~((glue com) (just 'a') (just 'b')))
    ['a' 'b']

    > (scan "a,b,a" ;~((glue com) (just 'a') (just 'b')))
    {1 4}
    syntax error

    > (scan "a,b,a" ;~((glue com) (just 'a') (just 'b') (just 'a')))
    ['a' 'b' 'a']
    ```
*   The `;~` \{% tooltip label="micsig"\
    href="/language/hoon/reference/rune/mic#-micsig" /%\} will create`;~(combinator (list rule))` to use multiple `rule`s.

    ```hoon
    > (scan "after the" ;~((glue ace) (star (shim 'a' 'z')) (star (shim 'a' 'z'))))  
    [[i='a' t=<|f t e r|>] [i='t' t=<|h e|>]

    > (;~(pose (just 'a') (just 'b')) [1 1] "ab")  
    [p=[p=1 q=2] q=[~ u=[p='a' q=[p=[p=1 q=2] q=[i='b' t=""]]]]]
    ```

At this point we have two problems: we are just getting raw `@t` atoms\
back, and we can't iteratively process arbitrarily long strings. \{%\
tooltip label="++cook" href="/language/hoon/reference/stdlib/4f#cook"\
/%\} will help us with the first of these:

*   will\
    take a \`rule\` and a

    to apply to the successful parse.

    ```hoon
    > ((cook ,@ud (just 'a')) [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ u=[p=97 q=[p=[p=1 q=2] q="bc"]]]]

    > ((cook ,@tas (just 'a')) [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ u=[p=%a q=[p=[p=1 q=2] q="bc"]]]]

    > ((cook |=(a=@ +(a)) (just 'a')) [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ u=[p=98 q=[p=[p=1 q=2] q="bc"]]]]

    > ((cook |=(a=@ `@t`+(a)) (just 'a')) [[1 1] "abc"])
    [p=[p=1 q=2] q=[~ u=[p='b' q=[p=[p=1 q=2] q="bc"]]]]
    ```

However, to parse iteratively, we need to use the \{% tooltip\
label="++knee" href="/language/hoon/reference/stdlib/4f#knee" /%\}\
function, which takes a noun as the \{% tooltip label="bunt"\
href="/glossary/bunt" /%\} of the type the `rule` produces, and produces\
a `rule` that recurses properly. (You'll probably want to treat this as\
a recipe for now and just copy it when necessary.)

```hoon

<div data-gb-custom-block data-tag="copy" data-0='true'></div>

|-(;~(plug prn ;~(pose (knee *tape |.(^$)) (easy ~))))
```

There is an example of a calculator [in the parsing\
guide](../../../language/hoon/guides/parsing/#recursive-parsers) that's worth a\
read at this point. It uses \{% tooltip label="++knee"\
href="/language/hoon/reference/stdlib/4f#knee" /%\} to scan in a set of numbers\
at a time.

#### Example: Parse a String of Numbers

A simple \{% tooltip label="++shim"\
href="/language/hoon/reference/stdlib/4f#shim" /%\}-based parser:

```hoon
> (scan "1234567890" (star (shim '0' '9')))  
[i='1' t=<|2 3 4 5 6 7 8 9 0|>]
```

A refined \{% tooltip label="++cook"\
href="/language/hoon/reference/stdlib/4f#cook" /%\}/

/parser:

```hoon
> ((cook (cury slaw %ud) (jest '1')) [[1 1] "123"])  
[p=[p=1 q=2] q=[~ u=[p=[~ 1] q=[p=[p=1 q=2] q="23"]]]]  

> ((cook (cury slaw %ud) (jest '12')) [[1 1] "123"])
[p=[p=1 q=3] q=[~ u=[p=[~ 12] q=[p=[p=1 q=3] q="3"]]]]
```

#### Example: Hoon Workbook

More examples demonstrating parser usage are available in the [Hoon\
Workbook](../../../language/hoon/examples/), such as the [Roman\
Numeral](../../../language/hoon/examples/roman/) tutorial.
