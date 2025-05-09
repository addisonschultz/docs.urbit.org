# bar

+++\
title = "| bar · Cores"\
weight = 10

\[glossaryEntry.bar]\
name = "bar"\
symbol = "|"\
usage = "Cores"\
desc = "Runes used to produce cores."

\[glossaryEntry.barbuc]\
name = "barbuc"\
symbol = "|$"\
usage = "Cores"\
slug = "#-barbuc"\
desc = "Declare a wet gate mold builder."

\[glossaryEntry.barcab]\
name = "barcab"\
symbol = "|_"_\
_usage = "Cores"_\
_slug = "#_-barcab"\
desc = "Produce a **door** (a core with a sample)."

\[glossaryEntry.barcen]\
name = "barcen"\
symbol = "|%"\
usage = "Cores"\
slug = "#-barcen"\
desc = "Produce a core, `[battery payload]`."

\[glossaryEntry.barcol]\
name = "barcol"\
symbol = "|:"\
usage = "Cores"\
slug = "#-barcol"\
desc = "Produce a gate with a custom sample."

\[glossaryEntry.bardot]\
name = "bardot"\
symbol = "|."\
usage = "Cores"\
slug = "#-bardot"\
desc = "Produce a trap (a core with one arm `$`)."

\[glossaryEntry.barhep]\
name = "barhep"\
symbol = "|-"\
usage = "Cores"\
slug = "#--barhep"\
desc = "Produce a trap (a core with one arm `$`) and evaluate it."

\[glossaryEntry.barket]\
name = "barket"\
symbol = "|^"\
usage = "Cores"\
slug = "#-barket"\
desc = "Produce a core whose battery includes a `$` arm and compute the latter."

\[glossaryEntry.barsig]\
name = "barsig"\
symbol = "|\~"\
usage = "Cores"\
slug = "#-barsig"\
desc = "Produce an iron gate."

\[glossaryEntry.bartar]\
name = "bartar"\
symbol = "|\*"\
usage = "Cores"\
slug = "#-bartar"\
desc = "Produce a wet gate (one-armed core with sample)."

\[glossaryEntry.bartis]\
name = "bartis"\
symbol = "|="\
usage = "Cores"\
slug = "#-bartis"\
desc = "Produce a gate (a one-armed core with a sample)."

\[glossaryEntry.barpat]\
name = "barpat"\
symbol = "|@"\
usage = "Cores"\
slug = "#-barpat"\
desc = "Produce a 'wet' core `[battery payload]`."

\[glossaryEntry.barwut]\
name = "barwut"\
symbol = "|?"\
usage = "Cores"\
slug = "#-barwut"\
desc = "Produce a lead trap."

+++

Core expressions produce cores. A core is a cell of `[battery payload]`.\
The `battery` is code, a battery of Nock formulas. The `payload` is the data\
needed to run those formulas correctly.

Five core runes (`|=`, `|.`, `|-`, `|*`, and `|$`) produce a core with a single\
arm, named `$`. As with all arms, we can recompute `$` with changes, which is\
useful for recursion among other things.

### `|$` "barbuc"

Declares a mold builder wet gate with one or more molds as its sample.

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  |$  sample
  body
  ```

***

* Wide
* ```hoon
  |$(sample body)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%brbc sample=(lest term) body=spec]
```

**Expands to**

```hoon
|$  [a b]
body
```

becomes

```hoon
|*  [a=$~(* $-(* *)) b=$~(* $-(* *))]
^:
body
```

**Semantics**

`|$` is used to declare a wet gate mold builder that is polymorphic in its input\
molds. `a` is a `lest` of `term` used as identifiers for the input molds. `b` is\
a structure built from elements of `a`. The output of `|$` is a mold builder\
obtained by substituting the input molds parameterized by `a` into `b`.

**Discussion**

A mold builder is a wet gate from one or more molds to a mold. A mold is a\
function from nouns to nouns with types that may be partial, is always\
idempotent, and usually the identity function on the noun itself.

`|$` is a restricted form of `|*`. The use of `|$` over `|*` is one of style, as\
either could be used to make wet gates that are mold builders. The buc in `|$`\
is a hint that `|$` is closely related to buc runes, and thus `|$` should be\
used to make mold builders, while `|*` should be used for any other sort of wet\
gate. Unlike `|*`, the body of `|$` is parsed in pattern mode to a `$spec`.\
Thus, the second argument of `|$` is frequently a buc rune. For further\
discussion of wet gates, see the entry for [`|*`](bar.md#-bartar).

Like other single-arm cores, the arm for `|$` is named `$` and this can be used to\
define recursive structures. Note however that Hoon is evaluated eagerly, and so\
infinite structures are not permitted.

Proper style for `|$` is to enclose the first argument with brackets, even if it\
is only a single term. The interpeter will accept a single term without brackets\
just fine, but this style is for consistency with the fact that the first\
argument is a `lest`.

**Examples**

```
> =foo |$([a b] [b a])

> =bar (foo [@ @tas])

> (bar %cat 3)
[%cat 3]
```

***

### `|_` "barcab"

Produce a **door** (a core with a sample).

**Syntax**

One fixed argument, then a variable number of `+`-family expressions.

* Form
* Syntax

***

* Tall
* ```hoon
  |_  a=spec
  ++  b=term  c=hoon
  ++  d=term  e=hoon
       ...
  ++  f=term  g=hoon
  --
  ```

***

* Wide
* None.

***

* Irregular
* None.

Note: The `++` rune may be replaced with any other rune in the `+` family.

**AST**

```hoon
[%brcb p=spec q=alas r=(map term tome)]
```

**Expands to**

```hoon
=|  a=spec
|%
++  b=term  c=hoon
++  d=term  e=hoon
       ...
++  f=term  g=hoon
--
```

**Semantics**

The product of a `|_` expression is a **door**, a core with one or more arms\
whose payload includes a sample. That is, a door is a cell of `[battery [sample context]]`, where the `battery` contains one or more arms.

`a` defines the door sample type and usually includes a name assignment (e.g.,`n=@`). `a` is followed by a series of arm definitions, each of which begins\
with a rune in the `+` family (most of `++`). There must be at least one arm,\
but there may be arbitrarily many. Each arm must include a name (`b`, `d`, and`f` above), which is followed by the expression (`c`, `e`, and `g` above) that\
defines the arm product.

The context of the door is the subject of the `|_` expression.

**Discussion**

A door is the general case of a gate (function). A gate is a door with only one\
arm, which has the name `$`.

Calling a door is like calling a gate except the caller also needs to specify\
the arm to be computed. So, for example, if you have some door `door` which\
contains some arm `arm`, and you want to pass some argument (i.e., input value`arg`), you would call it with `~(arm door arg)`.

Because gates are also doors, you can call them the same way. To call the gate`foo` as a door, instead of `(foo baz)` we would write `~($ foo baz)`. This is\
an irregular form for `%~($ foo baz)`,[%\~](../../../../../language/hoon/reference/rune/cen/#-censig).

**Examples**

A trivial door:

```
> =mol |_  a=@ud
       ++  succ  +(a)
       ++  prev  (dec a)
       --

> ~(succ mol 1)
2

> ~(succ mol ~(succ mol ~(prev mol 5)))
6
```

A more interesting door, from the kernel library:

```hoon
++  ne
  |_  tig=@
  ++  d  (add tig '0')
  ++  x  ?:((gte tig 10) (add tig 87) d)
  ++  v  ?:((gte tig 10) (add tig 87) d)
  ++  w  ?:(=(tig 63) '~' ?:(=(tig 62) '-' ?:((gte tig 36) (add tig 29) x)))
  --
```

The `ne` door prints a digit in base 10, 16, 32 or 64:

```
~zod:dojo> `@t`~(x ne 12)
'c'
```

***

### `|:` "barcol"

Produce a gate with a custom sample.

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  |:  a
  b
  ```

***

* Wide
* ```hoon
  |:(a b)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%brcl p=hoon q=hoon]
```

**Semantics**

`a` is a Hoon expression whose product type defines which values the gate accepts, and it usually includes a name (e.g., `n=1`). The product of `a` also serves as the default value of the sample. `b` is a Hoon expression that determines the product value of the gate.

**Expands to**

```hoon
=+  a
|.  b
```

**Discussion**

Pick your own default value for the sample. Note that `a` is an ordinary\
expression, not a type; `|:` doesn't bunt a sample as [`|=`](bar.md#-bartis) does.

This is useful if you want a gate to have a sample of a particular type, but you don't want the default value of the gate to be the default value of that type.

**Examples**

```
> =add-ten |:(n=`@`2 (add n 10))

> (add-ten 10)
20

> (add-ten)
12
```

***

### `|%` "barcen"

Produce a core, `[battery payload]`.

**Syntax**

Argument: a variable number of `+`-family expressions.

* Form
* Syntax

***

* Tall
* ```hoon
  |%
  ++  a=term  b=hoon
  ++  c=term  d=hoon
       ...
  ++  e=term  f=hoon
  --
  ```

***

* Wide
* None.

***

* Irregular
* None.

Note: The `++` rune may be replaced with any other rune in the `+` family.

**AST**

```hoon
[%brcn p=(unit term) q=(map term tome)]
```

**Semantics**

The product of a `|%` expression is a dry core with one or more arms in the\
battery.

The `|%` rune is followed by a series of arm definitions, each of which begins\
with a rune in the `+` family (most of `++`). There must be at least one arm,\
but there may be arbitrarily many. Each arm must include a name (`a`, `c`, and`e` above), which is followed by the expression (`b`, `d`, and `f` above) that\
defines the arm product.

The core payload is the subject of the `|%` expression.

**Discussion**

A core is a cell of `[battery payload]`, where the `battery` is code and the`payload` is data. The `battery` is one or more arms. An arm is a computation\
that takes the core itself as its subject.

The `|%` rune is used to construct a core from a series of arm definitions. Each\
arm definition in the expression begins with an arm rune (`++`, `+$`, or `+*`).\
These arms make up the `battery`. The subject of the `|%` expression is used to\
make the core's `payload`.

A core is like an "object" in a conventional language, but its attributes\
(_arms_) are functions on the core, not the core and an argument. A "method" on\
a core is an arm that produces a gate.

**Examples**

A trivial core:

```
> =foo =+  x=58
       |%
       ++  n  (add 42 x)
       ++  g  |=  b=@
              (add b n)
       --

> n.foo
100

> (g.foo 1)
101
```

***

### `|.` "bardot"

Produce a trap (a core with one arm `$`).

**Syntax**

One argument, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  |.  a
  ```

***

* Wide
* ```hoon
  |.(a)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%brdt p=hoon]
```

**Expands to**

```hoon
|%  ++  $  a=hoon
--
```

**Semantics**

A `|.` expression produces a core with a single arm, `$`. The core isn't\
explicitly given a sample. `a` is a Hoon expression that defines the computation\
of the `$` arm.

The payload of the core is the subject of the `|.` expression.

**Discussion**

A trap is generally used to defer a computation.

**Examples**

A trivial trap:

```
> =foo |.(42)

> $:foo
42

> (foo)
42
```

A more interesting trap:

```
> =foo =/  reps  10
       =/  step  0
       =/  outp  0
       |.
       ?:  =(step reps)
         outp
       $(outp (add outp 2), step +(step))

> (foo)
20
```

Note that we can use `$()` to recurse back into the\
trap, since it's a core with an `$` arm.

> `$(...)` expands to `%=($ ...)` (["centis"](../../../../../language/hoon/reference/rune/cen/#-centis)).

***

### `|^` "barket"

Produce a core whose battery includes a `$` arm and compute the latter.

**Syntax**

One fixed argument, then a variable number of `+`-family expressions.

* Form
* Syntax

***

* Tall
* ```hoon
  |^  a=hoon
  ++  b=term  c=hoon
  ++  d=term  e=hoon
       ...
  ++  f=term  g=hoon
  --
  ```

***

* Wide
* None.

***

* Irregular
* None.

***

**AST**

```hoon
[%brkt p=hoon q=(map term tome)]
```

**Expands to**

```hoon
=>  |%
    ++  $  a=hoon
    ++  b=term  c=hoon
    ++  d=term  e=hoon
           ...
    ++  f=term  g=hoon
    --
$
```

**Semantics**

A `|^` expression produces a multi-arm core whose battery includes a `$` arm,\
which is evaluated immediately. `a` is a Hoon expression that defines the\
product of the `$` arm. `a` is followed by a series of arm definitions for the\
rest of the core battery arms. There must be at least one arm other than the `$`\
arm.

**Discussion**

The `|^` rune is useful when you define a multi-arm core in your code and a\
particular arm in it is to be evaluated immediately.

**Examples**

A trivial example:

```
> |^
  (add n g)
  ++  n  42
  ++  g  58
  --
100
```

***

### `|-` "barhep"

Produce a trap (a core with one arm `$`) and evaluate it.

**Syntax**

One argument, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  |-  a
  ```

***

* Wide
* ```hoon
  |-(a)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%brhp p=hoon]
```

**Expands to**

```hoon
=<($ |.(a=hoon))
```

**Semantics**

A `|-` expression produces a core with one arm named `$` and immediately\
evaluates `$`. `a` is a Hoon expression that determines what `$` evaluates to.

**Discussion**

The `|-` rune can be thought of as a 'recursion point' or a 'loop starting\
point'. Since `|-` makes a `|.` (["bardot"](bar.md#-bardot), a core with one arm named`$`, we can recurse back into it with `$()`.

> `$(...)` expands to `%=($ ...)` (["centis"](../../../../../language/hoon/reference/rune/cen/#-centis)).

**Examples**

A trivial computation doesn't recurse:

```
> |-(42)
42
```

The classic loop is a decrement:

```
> =foo =/  a  42
       =/  b  0
       |-
       ?:  =(a +(b))
         b
       $(b +(b))

> foo
41
```

***

### `|~` "barsig"

Produce an iron gate.

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  |~  a
  b
  ```

***

* Wide
* ```hoon
  |~(a b)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%brsg p=spec q=hoon]
```

**Expands to**

```hoon
^|  |=(a b)
```

**Semantics**

A `|~` expression produces an iron gate. `a` defines the sample, and `b` defines\
the output value of the gate.

**Discussion**

See [this discussion of core variance models](../../../../../language/hoon/reference/advanced/)

**Examples**

```
> =>  ~  ^+(|~(a=@ *@) |=(a=* *@))
<1|usl {a/@ $~}>
```

***

### `|*` "bartar"

Produce a wet gate (one-armed core with sample).

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  |*  a  b
  ```

***

* Wide
* ```hoon
  |*(a b)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%brtr p=spec q=hoon]
```

**Expands to**

```hoon
=|  a
|@
++  $
  b
--
```

**Semantics**

A `|*` expression produces a wet gate. `a` defines the gate's sample, and `b` is\
a Hoon expression that determines the output value of the gate.

**Discussion**

In a normal (dry) gate, your argument is converted into the\
sample type. In a generic (wet) gate, your argument type\
passes through the function, rather as if it were a macro (there\
is still only one copy of the code, however).

Genericity is a powerful and dangerous tool. Use wet gates only if\
you know what you're doing.

Just as with a [gate](bar.md#-bartis), we can recurse back into a wet gate\
with `$()`.

> `$(...)` expands to `%=($ ...)` (["centis"](../../../../../language/hoon/reference/rune/cen/#-centis)).

`|*` can be used to make wet gates that produce structures, but this usage is\
discouraged in favor of `|$`.

**Examples**

Wet and dry gates in a nutshell:

```
> =foo |=([a=* b=*] [b a])

> =bar |*([a=* b=*] [b a])

> (foo %cat %dog)
[6.778.724 7.627.107]

> (bar %cat %dog)
[%dog %cat]
```

The dry gate does not preserve the type of `a` and `b`; the wet\
gate does.

***

### `|=` "bartis"

Produce a gate (a one-armed core with a sample).

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  |=  a
  b
  ```

***

* Wide
* ```hoon
  |=(a b)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%brts p=spec q=hoon]
```

**Expands to**

```hoon
=+  ^~(*a=spec)
|%  ++  $  b=hoon
--
```

**Definition**

The product of a `|=` expression is a dry **gate**, i.e., a Hoon function.

`p` defines the gate sample type -- i.e., the input value type -- and usually\
includes a sample name assignment (e.g., `a=@`). `q` is an expression that\
determines the output value of the gate.

**Discussion**

Dry gates are used for the vast majority of ordinary functions in Hoon.

A gate is a core with one arm named `$`, so we can recurse back into it with `$()`.

> `$(...)` expands to `%=($ ...)` (["centis"](../../../../../language/hoon/reference/rune/cen/#-centis)).

**Examples**

A trivial gate:

```
> =foo |=(a=@ +(a))

> (foo 20)
21
```

A slightly less trivial gate:

```
> =foo |=  [a=@ b=@]
       (add a b)

> (foo 30 400)
430
```

***

### `|@` "barpat"

Produce a 'wet' core `[battery payload]`.

**Syntax**

Arguments: a variable number of `+`-family expressions.

* Form
* Syntax

***

* Tall
* ```hoon
  |@
  ++  a=term  b=hoon
  ++  c=term  d=hoon
       ...
  ++  e=term  f=hoon
  --
  ```

***

* Wide
* None.

***

* Irregular
* None.

Note: The `++` rune may be replaced with any other rune in the `+` family.

**AST**

```hoon
[%brpt p=(unit term) q=(map term tome)]
```

**Semantics**

A `|@` expression produces a 'wet' core whose payload is the expression's\
subject. The various arms in the battery are each named (`a`, `c`, and `e`\
above) and defined explicitly with a Hoon expression (with `b`, `d`, and `f`\
above).

**Discussion**

The `|@` rune is just like the `|%` rune except that instead of producing a\
'dry' core, it produces a 'wet' one. This allows for type polymorphism of its\
arms, using 'genericity'. See [Advanced types](../../../../../language/hoon/reference/advanced/).

***

### `|?` "barwut"

Produce a lead trap.

**Syntax**

One argument, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  |?  a
  ```

***

* Wide
* ```hoon
  |?(a)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%brwt p=hoon]
```

**Expands to**

```hoon
^?  |.  a
```

**Semantics**

A `|?` expression produces a lead trap (i.e., a lead core with one arm named`$`). `a` is a Hoon expression that defines what the `$` arm does.

**Discussion**

See this [discussion of the core variance model](../../../../../language/hoon/reference/advanced/).

**Examples**

```
> =>  ~  ^+  |?(%a)  |.(%a)
<1?pqz $~>

> =>  ~  ^+  |?(%a)  |.(%b)
nest-fail
```
