# buc

+++\
title = "$ buc · Structures"\
weight = 20

\[glossaryEntry.buc]\
name = "buc"\
symbol = "$"\
usage = "Structures"\
desc = "Runes used for defining custom types."

\[glossaryEntry.bucbar]\
name = "bucbar"\
symbol = "$|"\
usage = "Structures"\
slug = "#-bucbar"\
desc = "`[%bcbr p=spec q=hoon]`: structure that satisfies a validator."

\[glossaryEntry.buccab]\
name = "buccab"\
symbol = "$_"_\
_usage = "Structures"_\
_slug = "#_-buccab"\
desc = "`[%bccb p=hoon]`: structure that normalizes to an example."

\[glossaryEntry.buccen]\
name = "buccen"\
symbol = "$%"\
usage = "Structures"\
slug = "#-buccen"\
desc = "`[%bccn p=(list spec)]`: structure which recognizes a union tagged by head atom."

\[glossaryEntry.buccol]\
name = "buccol"\
symbol = "$:"\
usage = "Structures"\
slug = "#-buccol"\
desc = "`[%bccl p=(list spec)]`: form a cell type."

\[glossaryEntry.bucgal]\
name = "bucgal"\
symbol = "$<"\
usage = "Structures"\
slug = "#-bucgal"\
desc = "`[%bcgl p=spec q=spec]`: restrict a mold by excluding some given mold."

\[glossaryEntry.bucgar]\
name = "bucgar"\
symbol = "$>"\
usage = "Structures"\
slug = "#-bucgar"\
desc = "`[%bchp p=spec q=spec]`: filter a mold to obtain a new mold."

\[glossaryEntry.buchep]\
name = "buchep"\
symbol = "$-"\
usage = "Structures"\
slug = "#--buchep"\
desc = "`[%bchp p=spec q=spec]`: structure that normalizes to an example gate."

\[glossaryEntry.bucket]\
name = "bucket"\
symbol = "$^"\
usage = "Structures"\
slug = "#-bucket"\
desc = "`[%bckt p=spec q=spec]`: structure which normalizes a union tagged by head depth (cell)."

\[glossaryEntry.buclus]\
name = "buclus"\
symbol = "$+"\
usage = "Structures"\
slug = "#buclus"\
desc = "`[%bcls p=stud q=spec]`: specify a shorthand type name for prettyprinting."

\[glossaryEntry.bucsig]\
name = "bucsig"\
symbol = "$\~"\
usage = "Structures"\
slug = "#-bucsig"\
desc = "`[%bcsg p=hoon q=spec]`: define a custom type default value"

\[glossaryEntry.bucpam]\
name = "bucpam"\
symbol = "$&"\
usage = "Structures"\
slug = "#-bucpam"\
desc = "`[%bcpm p=spec q=hoon]`: repair a value of a tagged union type"

\[glossaryEntry.bucpat]\
name = "bucpat"\
symbol = "$@"\
usage = "Structures"\
slug = "#-bucpat"\
desc = "`[%bcpt p=spec q=spec]`: structure which normalizes a union tagged by head depth (atom)."

\[glossaryEntry.buctis]\
name = "buctis"\
symbol = "$="\
usage = "Structures"\
slug = "#-buctis"\
desc = "`[%bcts p=skin q=spec]`: structure which wraps a face around another structure."

\[glossaryEntry.bucwut]\
name = "bucwut"\
symbol = "$?"\
usage = "Structures"\
slug = "#-bucwut"\
desc = "`[%bcwt p=(list spec)]`: form a type from a union of other types."

+++

The `$` family of runes is used for defining custom types. Strictly speaking,\
these runes are used to produce `spec`s, which we call 'structures'.

### Overview

Structures are abstract syntax trees for `type`s (see the documentation on[basic](../../../../../language/hoon/reference/basic/) and[advanced](../../../../../language/hoon/reference/advanced/) types for the\
precise definition of `type`). Structures are compile-time values of `type` which\
at runtime may be used to produce a 'mold'.

A mold is a function from nouns to nouns used to validate values of the type to\
which the structure defines. A mold can do two things at runtime. First, it may\
'clam' a noun, which validates the shape of the noun to be one that fits the\
abstract syntax tree given by the `spec` that produced the mold. Failing this\
validation results in a crash. Secondly, a mold may also be used to produce an\
example value of the type to which is corresponds, called the 'bunt value'. The\
bunt value is used as a placeholder for sample values that may be passed to a\
gate that accepts the corresponding type.

A correct mold is a **normalizer**: an idempotent function across all nouns. If\
the sample of a gate has type `%noun`, and its body obeys the constraint that\
for any x, `=((mold x) (mold (mold x)))`, it's a normalizer and can be used as a\
mold. Hoon is not dependently typed and so can't check idempotence statically,\
so we can't actually tell if a mold matches this definition perfectly. This is\
not actually a problem.

In any case, since molds are just functions, we can use functional programming\
to assemble interesting molds. For instance, `(map foo bar)` is a table from\
mold `foo` to mold `bar`. `map` is not a mold; it's a function that makes a\
mold. Molds and mold builders are generally described together.

`spec`s contain more information and draw finer distinctions than `type`s,\
which is to say that a given type may have more than one valid `spec` defining\
it, and thus downconversion from `spec` to `type` is lossy. Thus structure\
validation (done with [`$|`](buc.md#-bucbar), which is a more restrictive validation\
than that performed by molds, is a rare use case. Except for direct raw input,\
it's generally a faux pas to validate structure at runtime -- or even in userspace.\
Nonetheless they are sometimes utilized for types that will be more performant\
if they satisfy some validating gate.

***

### `$|` "bucbar"

Structure that satisfies a validator.

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  $|  p
  q
  ```

***

* Wide
* ```hoon
  $|(p q)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%bcbr p=spec q=hoon]
```

**Discussion**

`$|` is used for validation of values at a finer level than that of types.\
Recall that a given value of `type` can be equivalently defined by more than one`spec`. For performance reasons, it may be beneficial to restrict oneself to\
values of a given type that adhere to an abstract syntax tree specified by some\
subset of those `spec`s that may be used to define a given type.

`$|` takes two arguments: a structure `a` and a gate `b` that produces a `flag`\
that is used to validate values produced by the mold generated by `a` at\
runtime. `$|(a b)` is a gate that takes in a noun `x` and first pins the product\
of clamming `a` with `x`, call this `foo`. Then it calls `b` on `foo`. It\
asserts that the product of `(b foo)` is `&`, and then produces `foo`. This is\
equivalent to the following (which is not how `$|` is actually defined but has\
the same behavior):

```hoon
|=  x=*
=/  foo  ;;(a x)
?>  (b foo)
foo
```

For example, the elements of a `set` are treated as being unordered, but the\
values will necessarily possess an order by where they are in the memory. Thus\
if every `set` is stored using the same order scheme then faster algorithms\
involving `set`s may be written. Furthermore, if you just place elements in the`set` randomly, it may be mistreated by algorithms already in place that are\
expecting a certain order. This is not the same thing as casting - it is forcing\
a type to have a more specific set of values than its mold would suggest. This\
rune should rarely be used, but it is extremely important when it is.

**Examples**

```
> =foo $|  (list @)
       |=(a=(list) (lth (lent a) 4))
```

This creates a structure `foo` whose values are `list`s with length less than 4.

```
> (foo ~[1 2 3])
~[1 2 3]

> (foo ~[1 2 3 4])
ford: %ride failed to execute:
```

The definition of `+set` in `hoon.hoon` is the following:

```hoon
++  set
  |$  [item]
  $|  (tree item)
  |=(a=(tree) ?:(=(~ a) & ~(apt in a)))
```

Here [`|$`](../../../../../language/hoon/reference/rune/bar/#-barbuc) is used to define a mold\
builder that takes in a mold (given the face `item`) and creates a structure\
consisting of a `tree` of `item`s with `$|` that is validated with the gate`|=(a=(tree) ?:(=(~ a) & ~(apt in a)))`. `in` is a door in `hoon.hoon` with\
functions for handling `set`s, and `apt` is an arm in that door that checks that\
the values in the `tree` are arranged in the particular way that `set`s are\
arranged in Hoon, namely 'ascending `+mug` hash order'.

***

### `$_` "buccab"

Structure that normalizes to an example.

**Syntax**

One argument, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  $_  p
  ```

***

* Wide
* ```hoon
  $_(p)
  ```

***

* Irregular
* ```
    _p
  ```

**AST**

```hoon
[%bccb p=hoon]
```

**Expands to**

```hoon
|=(* p)
```

**Discussion**

`$_` discards the sample it's supposedly normalizing and produces its**example** instead.

**Examples**

```
> =foo $_([%foobaz %moobaz])

> (foo %foo %baz)
[%foobaz %moobaz]

> `foo`[%foobaz %moobaz]
[%foobaz %moobaz]

> $:foo
[%foobaz %moobaz]
```

***

### `$%` "buccen"

Structure which recognizes a union tagged by head atom.

**Syntax**

A variable number of arguments.

* Form
* Syntax

***

* Tall
* ```hoon
  $%  [%p1 ...]
      [%p2 ...]
      [%p3 ...]
      [%pn ...]
  ==
  ```

***

* Wide
* ```hoon
  $%([%p1 ...] [%p2 ...] [%p3 ...] [%pn ...])
  ```

***

* Irregular
* None.

Each item may be an atom or (more commonly) a cell. The atom or head of the cel&#x6C;_&#x6D;ust_ be a constant (`%foo`, `%1`, `%.y`, etc).

**AST**

```hoon
[%bccn p=(list spec)]
```

**Defaults to**

The default of the last item `i` in `p`. Crashes if `p` is empty.

**Discussion**

A `$%` is a tagged union, a common data model.

Make sure the last item in your `$%` terminates, or the default will\
be an infinite loop! Alteratively, you can use `$~` to define a custom\
type default value.

**Examples**

```
> =foo $%([%foo p=@ud q=@ud] [%baz p=@ud])

> (foo [%foo 4 2])
[%foo p=4 q=2]

> (foo [%baz 37])
[%baz p=37]

> $:foo
[%baz p=0]
```

***

### `$:` "buccol"

Form a cell type.

**Syntax**

A variable number of arguments.

* Form
* Syntax

***

* Tall
* ```hoon
  $:  p1
      p2
      p3
      pn
  ==
  ```

***

* Wide
* ```hoon
  $:(p1 p2 p3 pn)
  ```

***

* Irregular (noun mode)
* ```hoon
  ,[p1 p2 p3 pn]
  ```

***

* Irregular (structure mode)
* ```hoon
    [p1 p2 p3 pn]
  ```

**AST**

```hoon
[%bccl p=(list spec)]
```

**Normalizes to**

The tuple the length of `p`, normalizing each item.

**Defaults to**

The tuple the length of `p`.

**Examples**

```
> =foo $:(p=@ud q=@tas)

> (foo 33 %foo)
[p=33 q=%foo]

> `foo`[33 %foo]
[p=33 q=%foo]

> $:foo
[p=0 q=%$]
```

***

### `$<` "bucgal"

Filters a pre-existing mold to obtain a mold that excludes a particular\
structure.

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  $<  p
  q
  ```

***

* Wide
* ```hoon
  $<(p q)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%bcgl p=spec q=spec]
```

**Discussion**

This can be used to obtain type(s) from a list of types `q` that do not satisfy a\
requirement given by `p`.

**Examples**

```
> =foo $%([%bar p=@ud q=@ud] [%baz p=@ud])

> =m $<(%bar foo)

> (m [%bar 2 4])
ford: %ride failed to execute:

> (m [%baz 2])
[%baz p=2]

> ;;($<(%foo [@tas *]) [%foo 1])
ford: %ride failed to execute:

> ;;($<(%foo [@tas *]) [%bar 1])
[%bar 1]
```

***

### `$>` "bucgar"

Filters a mold to obtain a new mold matching a particular structure.

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  $>  p
  q
  ```

***

* Wide
* ```hoon
  $>(p q)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%bcgr p=spec q=spec]
```

**Discussion**

This can be used to obtain type(s) from a list of types `q` that satisfy a\
requirement given by `p`.

**Examples**

Examples with `$%`:

```
> =foo $%([%bar p=@ud q=@ud] [%baz p=@ud])

> =m $>(%bar foo)

> (m [%bar 2 4])
[%bar p=2 q=4]

> (m [%baz 2])
ford: %ride failed to execute:
```

Examples with `;;`:

```
> ;;([@tas *] [%foo 1])
[%foo 1]

> ;;([@tas *] [%bar 1])
[%bar 1]

> ;;($>(%foo [@tas *]) [%foo 1])
[%foo 1]

> ;;($>(%foo [@tas *]) [%bar 1])
ford: %ride failed to execute:
```

***

### `$-` "buchep"

Structure that normalizes to an example gate.

**AST**

```hoon
[%bchp p=spec q=spec]
```

**Expands to**

```hoon
$_  ^|
|=(p $:q)
```

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  $-  p
  q
  ```

***

* Wide
* ```hoon
  $-(p q)
  ```

***

* Irregular
* None.

`p` is the type the gate takes and `q` is the type the gate produces.

**Discussion**

Since a `$-` reduces to a [`$_`](buc.md#_-buccab), it is not useful for normalizing, just for typechecking. In particular, the existence of `$-`s does **not** let us send gates or other cores over the network!

**Examples**

```
> =foo $-(%foo %baz)

> ($:foo %foo)
%baz
```

***

### `$^` "bucket"

Structure which normalizes a union tagged by head depth (cell).

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  $^  p
  q
  ```

***

* Wide
* ```hoon
  $^(p q)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%bckt p=spec q=spec]
```

**Normalizes to**

Default, if the sample is an atom; `p`, if the head of the sample\
is an atom; `q` otherwise.

**Defaults to**

The default of `p`.

**Examples**

```
> =a $%([%foo p=@ud q=@ud] [%baz p=@ud])

> =b $^([a a] a)

> (b [[%baz 33] [%foo 19 22]])
[[%baz p=33] [%foo p=19 q=22]]

> (b [%foo 19 22])
[%foo p=19 q=22]

> $:b
[%baz p=0]
```

***

### `$+` "buclus"

Specify a shorthand type name for use in prettyprinting.

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  $+  p
  q
  ```

***

* Wide
* ```hoon
  $+(p q)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%bcls p=stud q=spec]
```

**Examples**

```
> =/  my-type  $+(my-alias [@ @])

> -:!>(*my-type)
#t/#my-alias
```

***

### `$&` "bucpam"

Repair a value of a tagged union type.

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  $&  p
  q
  ```

***

* Wide
* ```hoon
  $&(p q)
  ```

***

* Irregular
* None.

```hoon
$&(combined-mold=spec normalizing-gate=hoon)
```

Here `combined-mold` is a tagged union type (typically made with `$%`) and`normalizing-gate` is a gate which accepts values of `combined-mold` and\
normalizes them to be of one particular type in `combined-mold`.

**AST**

```hoon
[%bcpm p=spec q=hoon]
```

**Normalizes to**

The product of the normalizing gate and sample.

**Defaults to**

The default of the last type listed in `p`, normalized with the normalizing gate.

**Discussion**

This rune is used to "upgrade" or "repair" values of a structure, typically from\
an old version to a new version. For example, this may happen when migrating\
state after updating an app.

**Examples**

```hoon
+$  old  [%0 @]
+$  new  [%1 ^]
+$  combined  $%(old new)
+$  adapting  $&(combined |=(?-(-.a %0 [%1 1 +.a], %1 a)))
```

Here `adapting` is a structure that bunts to `[%1 ^]` but also normalizes from`[%0 @]` if called on such a noun.

***

### `$~` "bucsig"

Define a custom type default value.

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  $~  p
  q
  ```

***

* Wide
* ```hoon
  $~(p q)
  ```

***

* Irregular
* None.

`p` defines the default value, and `q` defines everything else about the\
structure.

**AST**

```hoon
[%bcsg p=hoon q=spec]
```

**Product**

Creates a structure (custom type) just like `q`, except its default value is `p`.

**Defaults to**

The product of `p`.

**Discussion**

You should make sure that the product type of `p` nests under `q`. You can check\
the default value of some structure (custom type) `r` with `*r`. (See the [`^*`\
rune](../../../../../language/hoon/reference/rune/ket/#-kettar).)

Do not confuse the `$~` rune with the constant type for null, `$~`. (The latter\
uses older Hoon syntax that is still accepted. Preferably it would be `%~`.)

**Examples**

First, let's define a type without using `$~`:

```
> =b $@(@tas $%([%two *] [%three *]))

> `b`%hello
%hello

> `b`[%two %hello]
[%two 478.560.413.032]

> *b

%$

> *@tas
%$
```

Using `$~`:

```
> =c $~(%default-value $@(@tas $%([%two *] [%three *])))

> `c`%hello
%hello

> `c`[%two %hello]
[%two 478.560.413.032]

> *c
%default-value
```

***

### `$@` "bucpat"

Structure which normalizes a union tagged by head depth (atom).

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  $@  p
  q
  ```

***

* Wide
* ```hoon
  $@(p q)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%bcpt p=spec q=spec]
```

**Normalizes to**

`p`, if the sample is an atom; `q`, if the sample is a cell.

**Defaults to**

The default of `p`.

**Produces**

A structure which applies `p` if its sample is an atom, `q` if its sample is a\
cell.

**Examples**

```
> =a $@(%foo $:(p=%baz q=@ud))

> (a %foo)
%foo

> `a`[%baz 99]
[p=%baz q=99]

> $:a
%foo
```

***

### `$=` "buctis"

Structure which wraps a face around another structure.

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  $=  p
  q
  ```

***

* Wide
* ```hoon
  $=(p q)
  ```

***

* Irregular
* ```
    p=q
  ```

**AST**

```hoon
[%bcts p=skin q=spec]
```

**Expands to**

```hoon
|=  *
^=(p %-(q +6))
```

**Discussion**

Note that the Hoon compiler is at least slightly clever about\
compiling structures, and almost never has to actually put in a gate\
layer (as seen in the expansion above) to apply a `$=`.

**Examples**

```
> =a $=(p %foo)

> (a %foo)
p=%foo

> (a %baz)
ford: %ride failed to execute:
```

***

### `$?` "bucwut"

Form a type from a union of other types.

**Syntax**

Variable number of arguments.

* Form
* Syntax

***

* Tall
* ```hoon
  $?  p1
      p2
      p3
      pn
  ==
  ```

***

* Wide
* ```hoon
  $?(p1 p2 p3 pn)
  ```

***

* Irregular
* ```hoon
    ?(p1 p2 p3 pn)
  ```

**AST**

```hoon
[%bcwt p=(list spec)]
```

**Normalizes to**

The last item in `p` which normalizes the sample to itself.

Void, if `p` is empty.

**Defaults to**

The last item in `p`.

**Discussion**

For a union of atoms, a `$?` is fine. For more complex nouns, always try to use\
a [`$%`](buc.md#-buccen), [`$@`](buc.md#-bucpat) or [`$^`](buc.md#-bucket), at least if you expect\
your structure to be used as a normalizer.

**Examples**

```
> =a ?(%foo %baz %baz)

> (a %baz)
%baz

> (a [37 45])
ford: %ride failed to execute:

> $:a
%baz
```
