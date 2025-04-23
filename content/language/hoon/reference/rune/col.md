# col

+++\
title = ": col · Cells"\
weight = 40

\[glossaryEntry.col]\
name = "col"\
symbol = ":"\
usage = "Cells"\
desc = "Runes used to produce cells, which are pairs of nouns."

\[glossaryEntry.colcab]\
name = "colcab"\
symbol = ":_"_\
_usage = "Cells"_\
_slug = "#_-colcab"\
desc = "`[%clcb p=hoon q=hoon]`; construct a cell, inverted."

\[glossaryEntry.colcol]\
name = "colcol"\
symbol = "::"\
usage = "Cells"\
slug = "#-colcol"\
desc = "Code comment."

\[glossaryEntry.colhep]\
name = "colhep"\
symbol = ":-"\
usage = "Cells"\
slug = "#--colhep"\
desc = "`[%clhp p=hoon q=hoon]`: construct a cell (2-tuple)."

\[glossaryEntry.colket]\
name = "colket"\
symbol = ":^"\
usage = "Cells"\
slug = "#-colket"\
desc = "`[%clkt p=hoon q=hoon r=hoon s=hoon]`: construct a quadruple (4-tuple)."

\[glossaryEntry.collus]\
name = "collus"\
symbol = ":+"\
usage = "Cells"\
slug = "#-collus"\
desc = "`[%clls p=hoon q=hoon r=hoon]`: construct a triple (3-tuple)."

\[glossaryEntry.colsig]\
name = "colsig"\
symbol = ":\~"\
usage = "Cells"\
slug = "#-colsig"\
desc = "`[%clsg p=(list hoon)]`: construct a null-terminated list."

\[glossaryEntry.coltar]\
name = "coltar"\
symbol = ":\*"\
usage = "Cells"\
slug = "#-coltar"\
desc = "`[%cltr p=(list hoon)]`: construct an n-tuple."

+++

The `:` ("col") expressions are used to produce cells, which are pairs of\
values. E.g., `:-(p q)` produces the cell `[p q]`. All `:` runes reduce to `:-`.

### `:-` "colhep"

Construct a cell (2-tuple).

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  :-  p
  q
  ```

***

* Wide
* ```hoon
  :-(p q)
  ```

***

* Irregular #1
* ```hoon
  [p q]
  ```

***

* Irregular #2
* ```
    p^q
  ```

**AST**

```hoon
[%clhp p=hoon q=hoon]
```

**Produces**

The cell of `p` and `q`.

**Discussion**

Hoon expressions actually use the same "autocons" pattern as Nock\
formulas. If you're assembling expressions (which usually only the\
compiler does), `[a b]` is the same as `:-(a b)`.

**Examples**

```
> :-(1 2)
[1 2]

~zod:dojo> 1^2
[1 2]
```

***

### `:_` "colcab"

Construct a cell, inverted.

**Syntax**

Two arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  :_  p
  q
  ```

***

* Wide
* ```hoon
  :_(p q)
  ```

***

* Irregular
* None.

**AST**

```hoon
[%clcb p=hoon q=hoon]
```

**Expands to**

```hoon
:-(q p)
```

**Examples**

```
> :_(1 2)
[2 1]
```

***

### `:+` "collus"

Construct a triple (3-tuple).

**Syntax**

Three arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  :+  p
    q
  r
  ```

***

* Wide
* ```hoon
  :+(p q r)
  ```

***

* Irregular
* ```hoon
    [p q r]
  ```

**AST**

```hoon
[%clls p=hoon q=hoon r=hoon]
```

**Expands to:**

```hoon
:-(p :-(q r))
```

**Examples**

```
> :+  1
    2
  3
[1 2 3]

> :+(%a ~ 'b')
[%a ~ 'b']
```

***

### `:^` "colket"

Construct a quadruple (4-tuple).

**Syntax**

Four arguments, fixed.

* Form
* Syntax

***

* Tall
* ```hoon
  :^    p
      q
    r
  s
  ```

***

* Wide
* ```hoon
  :^(p q r s)
  ```

***

* Irregular
* ```hoon
    [p q r s]
  ```

**AST**

```hoon
[%clkt p=hoon q=hoon r=hoon s=hoon]
```

**Expands to**

```hoon
:-(p :-(q :-(r s)))
```

**Examples**

```
> :^(1 2 3 4)
[1 2 3 4]

> :^    5
      6
    7
  8
[5 6 7 8]
```

***

### `:*` "coltar"

Construct an n-tuple.

**Syntax**

Variable number of arguments.

* Form
* Syntax

***

* Tall
* ```hoon
  :*  p1
      p2
      p3
      pn
  ==
  ```

***

* Wide
* ```hoon
  :*(p1 p2 p3 pn)
  ```

***

* Irregular
* ```
    [p1 p2 p3 pn]
  ```

**AST**

```hoon
[%cltr p=(list hoon)]
```

**Expands to**

**Pseudocode**: `a`, `b`, `c`, ... as elements of `p`:

```hoon
:-(a :-(b :-(c :-(... z)))))
```

**Desugaring**

```hoon
|-
?~  p
  !!
?~  t.p
  i.p
:-  i.p
$(p t.p)
```

**Examples**

```
> :*(5 3 4 1 4 9 0 ~ 'a')
[5 3 4 1 4 9 0 ~ 'a']

> [5 3 4 1 4 9 0 ~ 'a']
[5 3 4 1 4 9 0 ~ 'a']

> :*  5
      3
      4
      1
      4
      9
      0
      ~
      'a'
  ==
[5 3 4 1 4 9 0 ~ 'a']
```

***

### `:~` "colsig"

Construct a null-terminated list.

**Syntax**

Variable number of arguments.

* Form
* Syntax

***

* Tall
* ```hoon
  :~  p1
      p2
      p3
      pn
  ==
  ```

***

* Wide
* ```hoon
  :~(p1 p2 p3 pn)
  ```

***

* Irregular
* ```
    ~[p1 p2 p3 pn]
  ```

**AST**

```hoon
[%clsg p=(list hoon)]
```

**Expands to**

**Pseudocode**: `a`, `b`, `c`, ... as elements of `p`:

```hoon
:-(a :-(b :-(c :-(... :-(z ~)))))
```

**Desugaring**

```hoon
|-
?~  p
  ~
:-  i.p
$(p t.p)
```

**Discussion**

Note that this does not produce a `list` type, it just produces a\
null-terminated n-tuple. To make it a proper `list` it must be cast or molded.

**Examples**

```
> :~(5 3 4 2 1)
[5 3 4 2 1 ~]

> ~[5 3 4 2 1]
[5 3 4 2 1 ~]

> :~  5
      3
      4
      2
      1
  ==
[5 3 4 2 1 ~]
```

***

### `::` "colcol"

Code comment.

**Syntax**

```hoon
::  any text you like!
```

**Examples**

```hoon
::
::  this is commented code
::
|=  a=@         ::  a gate
(add 2 a)       ::  that adds 2
                ::  to the input
```
