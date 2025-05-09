# M-typecheck

+++\
title = "12. Type Checking"\
weight = 22\
nodes = \[183]\
objectives = \["Use assertions to enforce type constraints."]\
+++

_In this module we'll cover how the Hoon compiler infers type, as well_\
_as various cases in which a type check is performed._

### Type Casting

Casting is used to explain to the Hoon compiler exactly what it is we\
mean with a given data structure. As you get in the habit of casting\
your data structures, it will not only help anyone reading your code,\
but it will help you in hunting down bugs in your code.

is\
a mold builder that is used to produce a \{% tooltip label="mold"\
href="/glossary/mold" /%\}, i.e. a list of a particular type (like \`(list\
@)\` for a list of atoms). A list can be thought of as an ordered\
arrangement of zero or more elements terminated by a \`\~\` (null). There\
is a difference to Hoon, however, between something explicitly tagged as\
a \`list\` of some kind and a null-terminated tuple.

```hoon
> -:!>(~[1 2 3])
#t/[@ud @ud @ud %~]

> -:!>(`(list @)`~[1 2 3])
#t/it(@)
```

The former is inflexible and doesn't have the `i`/`t` faces that a list\
presents. By marking the type explicitly as a `(list @)` for the\
compiler, we achieve some stronger guarantees that many of the `list`\
operators require.

However, we still don't get the \{% tooltip label="faces"\
href="/glossary/face" /%\} for free:

```hoon
> =a `(list @)`~[1 2 3]

> i.a
-find.i.a
find-fork
dojo: hoon expression failed
```

What's going on? Formally, a \{% tooltip label="list"\
href="/glossary/list" /%\} can be either null or non-null. When the list\
contains only `~` and no items, it's the null list. Most lists are,\
however, non-null lists, which have items preceding the `~`. Non-null\
lists, called _lests_, are \{% tooltip label="cells"\
href="/glossary/cell" /%\} in which the head is the first list item, and\
the tail is the rest of the list. The tail is itself a list, and if\
such a list is also non-null, the head of this sublist is the second\
item in the greater list, and so on. To illustrate, let's look at a\
list `[1 2 3 4 ~]` with the cell-delineating brackets left in:

```hoon
[1 [2 [3 [4 ~]]]]
```

It's easy to see where the heads are and where the nesting tails are.\
The head of the above list is the atom `1` and the tail is the list `[2 [3 [4 ~]]]`, (or `[2 3 4 ~]`). Recall that whenever \{% tooltip\
label="cell" href="/glossary/cell" /%\} brackets are omitted so that\
visually there appears to be more than two child \{% tooltip\
label="nouns" href="/glossary/noun" /%\}, it is implicitly understood\
that the right-most nouns constitute a cell.

You can construct

of\
any type. `(list @)` indicates a list of atoms, `(list ^)` indicates a\
list of cells, `(list [@ ?])` indicates a list of cells whose head is an\
atom and whose tail is a flag, etc.

```hoon
> `(list @)`~
~

> `(list @)`[1 2 3 4 5 ~]
~[1 2 3 4 5]

> `(list @)`[1 [2 [3 [4 [5 ~]]]]]
~[1 2 3 4 5]

> `(list @)`~[1 2 3 4 5]
~[1 2 3 4 5]
```

Notice how the last Dojo command has a different construction, with the`~` in front of the bracketed items. This is just another way of\
writing the same thing; `~[1 2]` is semantically identical to `[1 2 ~]`.

Back to our earlier example:

```hoon
> =a `(list @)`~[1 2 3]

> i.a
-find.i.a
find-fork
dojo: hoon expression failed
```

Any time we see a `find-fork` error, it means that the type checker\
considers the value to be underspecified. In this case, it can't\
guarantee that `i.a` exists because although `a` is a list, it's not\
known to be a non-null lest. If we enforce that constraint, then\
suddenly we can use the faces:

```hoon
> ?:  ?=(~ a)  !!  i.a
1
```

It's important to note that performing tests like this will actually\
transform a

into a`lest`, a non-null list. Because `lest` is a different type than`list`, performing such tests can come back to bite you later in\
non-obvious ways when you try to use some standard library functions\
meant for lists.

#### Casting Nouns (`^` ket Runes)

As the Hoon compiler compiles your Hoon code, it does a type check on\
certain expressions to make sure they are guaranteed to produce a value\
of the correct type. If it cannot be proved that the output value is\
correctly typed, the compile will fail with a \{% tooltip\
label="nest-fail" href="/language/hoon/reference/hoon-errors#nest-fail"\
/%\} crash. In order to figure out what type of value is produced by a\
given expression, the compiler uses type inference on that code.

Let's enumerate the most common cases where a type check is called for\
in Hoon.

The most obvious case is when there is a casting `^` \{% tooltip\
label="ket" href="/language/hoon/reference/rune/ket" /%\} rune in your\
code. These runes don't directly have any effect on the compiled result\
of your code; they simply indicate that a type check should be performed\
on a piece of code at compile-time.

**`^-` kethep Cast with a Type**

You've already seen one rune that calls for a type check:`^-` \{% tooltip label="kethep"\
href="/language/hoon/reference/rune/ket#--kethep" /%\}:

```hoon
> ^-(@ 12)
12

> ^-(@ 123)
123

> ^-(@ [12 14])
nest-fail

> ^-(^ [12 14])
[12 14]

> ^-(* [12 14])
[12 14]

> ^-(* 12)
12

> ^-([@ *] [12 [23 43]])
[12 [23 43]]

> ^-([@ *] [[12 23] 43])
nest-fail
```

**`^+` ketlus Cast with an Example Value**

The rune `^+` \{% tooltip label="ketlus"\
href="/language/hoon/reference/rune/ket#-ketlus" /%\} is like `^-` \{%\
tooltip label="kethep" href="/language/hoon/reference/rune/ket#--kethep"\
/%\}, except that instead of using a type name for the cast, it uses an\
example value of the type in question. E.g.:

```hoon
> ^+(7 12)
12

> ^+(7 123)
123

> ^+(7 [12 14])
nest-fail
```

The `^+` \{% tooltip label="ketlus"\
href="/language/hoon/reference/rune/ket#-ketlus" /%\} rune takes two\
subexpressions. The first subexpression is evaluated and its type is\
inferred. The second subexpression is evaluated and its inferred type\
is compared against the type of the first. If the type of the second\
provably nests under the type of the first, the result of the `^+`\
ketlus expression is just the value of its second subexpression.\
Otherwise, the code fails to compile.

This rune is useful for casting when you already have a noun—or an\
expression producing a noun—whose type you may not know or be able to\
construct easily. If you want your output value to be of the same type,\
you can use `^+` ketlus.

More examples:

```hoon
> ^+([12 13] [123 456])
[123 456]

> ^+([12 13] [123 [12 14]])
nest-fail

> ^+([12 [1 2]] [123 [12 14]])
[123 12 14]
```

#### Arm Checks

Whenever an

is evaluated\
in Hoon it expects to have some version of its parent \{% tooltip\
label="core" href="/glossary/core" /%\} as the \{% tooltip label="subject"\
href="/glossary/subject" /%\}. Specifically, a type check is performed\
to see whether the arm subject is of the appropriate type. We see this\
in action whenever a

or a multi-arm

is\
called.

A gate is a one-armed core with a \{% tooltip label="sample"\
href="/glossary/sample" /%\}. When it is called, its `$` buc arm is\
evaluated with (a mutated copy of) the gate as the \{% tooltip\
label="subject" href="/glossary/subject" /%\}. The only part of the core\
that might change is the \{% tooltip label="payload"\
href="/glossary/payload" /%\}, including the sample. Of course, we want\
the sample to be able to change. The sample is where the argument(s) of\
the function call are placed. For example, when we call \{% tooltip\
label="add" href="/language/hoon/reference/stdlib/1a#add" /%\} the `$`\
buc arm expects two

for the sample, i.e., the two numbers to be added. When the type check\
occurs, the payload must be of the appropriate type. If it isn't, the\
result is a \{% tooltip label="nest-fail"\
href="/language/hoon/reference/hoon-errors#nest-fail" /%\} crash.

```hoon
> (add 22 33)
55

> (add [10 22] [10 33])
nest-fail

> (|=(a=@ [a a]) 15)
[15 15]

> (|=(a=@ [a a]) 22)
[22 22]

> (|=(a=@ [a a]) [22 22])
nest-fail
```

We'll talk in more detail about the various kinds of type-checking that\
can occur at arm evaluation [when we discuss type\
polymorphism](../../../courses/hoon-school/R-metals/).

This isn't a comprehensive list of the type checks in Hoon: for\
instance, some other runes that include a type check are `=.` \{% tooltip\
label="tisdot" href="/language/hoon/reference/rune/tis#-tisdot" /%\} and`%_` \{% tooltip label="cencab"\
href="/language/hoon/reference/rune/cen#\_-cencab" /%\}.

### Type Inference

Hoon infers the type of any given expression. How does this inference\
work? Hoon has available various tools for inferring the type of any\
given expression: literal syntax, cast expressions, gate sample\
definitions, conditional expressions, and more.

#### Literals

[Literals](https://en.wikipedia.org/wiki/Literal_\(computer_programming\))\
are expressions that represent fixed values. \{% tooltip label="Atom"\
href="/glossary/atom" /%\} and \{% tooltip label="cell"\
href="/glossary/cell" /%\} literals are supported in Hoon, and every\
supported

has an\
unambiguous representation that allows the parser to directly infer the\
type from the form. Here are a few examples of auras and associated\
literal formats:

| Type         | Literal               |
| ------------ | --------------------- |
| `@ud`        | `123`, `1.000`        |
| `@ux`        | `0x1234`, `0x12.3456` |
| `@ub`        | `0b1011.1110`         |
| `[@ud @ud]`  | `[12 14]`             |
| `[@ux @t ?]` | `[0x1f 'hello' %.y]`  |

#### Casts

Casting with `^` \{% tooltip label="ket"\
href="/language/hoon/reference/rune/ket" /%\} runes also shape how Hoon\
understands an expression type, as outlined above. The inferred type of\
a cast expression is just the type being cast for. It can be inferred\
that, if the cast didn't result in a \{% tooltip label="nest-fail"\
href="/language/hoon/reference/hoon-errors#nest-fail" /%\}, the value\
produced must be of the cast type. Here are some examples of cast\
expressions with the inferred output type on the right:

| Type        | Cast                            |
| ----------- | ------------------------------- |
| `@ud`       | `^-(@ud 123)`                   |
| `@`         | `^-(@ 123)`                     |
| `^`         | `^-(^ [12 14])`                 |
| `[@ @]`     | `^-([@ @] [12 14])`             |
| `*`         | `^-(* [12 14])`                 |
| `@ud`       | `^+(7 123)`                     |
| `[@ud @ud]` | `^+([7 8] [12 14])`             |
| `[@ud @ud]` | `^+([44 55] [12 14])`           |
| `[@ux @ub]` | `^+([0x1b 0b11] [0x123 0b101])` |

You can also use the irregular `` ` `` syntax for casting in the same\
way as `^-` \{% tooltip label="kethep"\
href="/language/hoon/reference/rune/ket#--kethep" /%\}; e.g., `` `@`123 `` for `^-(@ 123)`.

Since casts can throw away type information, if the cast type is more\
general, then the more specific type information is lost. Consider the\
literal `[12 14]`. The inferred type of this expression is `[@ @]`,\
i.e., a

of two \{%\
tooltip label="atoms" href="/glossary/atom" /%\}. If we cast over `[12 14]` with `^-(^ [12 14])` then the inferred type is just `^`, the set of\
all cells. The information about what kind of cell it is has been\
thrown away. If we cast over `[12 14]` with `^-(* [12 14])` then the\
inferred type is `*`, the set of all \{% tooltip label="nouns"\
href="/glossary/noun" /%\}. All interesting type information is thrown\
away on the latter cast.

It's important to remember to include a cast \{% tooltip label="rune"\
href="/glossary/rune" /%\} with each \{% tooltip label="gate"\
href="/glossary/gate" /%\} and \{% tooltip label="trap"\
href="/glossary/trap" /%\} expression. That way it's clear what the\
inferred product type will be for calls to that core.

#### (Dry) Gate Sample Definitions

By now you've used the `|=` \{% tooltip label="bartis"\
href="/language/hoon/reference/rune/bar#-bartis" /%\} rune to define\
several

. This rune\
is used to produce a _dry gate_, which has different type-checking and\
type-inference properties than a _wet gate_ does. We won't explain the\
distinction until [a later module](../../../courses/hoon-school/R-metals/)—for\
now, just keep in mind that we're only dealing with one kind of gate\
(albeit the more common kind).

The first subexpression after the `|=` defines the \{% tooltip\
label="sample" href="/glossary/sample" /%\} type. Any faces used in this\
definition have the type declared for it in this definition. Consider\
an addition generator `/gen/sum.hoon`:

```hoon

|=  [a=@ b=@]
^-  @
?:  =(b 0)
  a
$(a +(a), b (dec b))
```

We run it in the

using\
a cell to pass the two arguments:

```hoon
> +sum [12 14]
26

> +sum 22
nest-fail
-need.[a=@ b=@]
-have.@ud
```

If you try to call this gate with the wrong kind of argument, you get\
a \{% tooltip label="nest-fail"\
href="/language/hoon/reference/hoon-errors#nest-fail" /%\}. If the call\
succeeds, then the argument takes on the type of the \{% tooltip\
label="sample" href="/glossary/sample" /%\} definition: `[a=@ b=@]`.\
Accordingly, the inferred type of `a` is `@`, and the inferred type of`b` is `@`. In this case some type information has been thrown away;\
the inferred type of `[12 14]` is `[@ud @ud]`, but the addition program\
takes all atoms, regardless of \{% tooltip label="aura"\
href="/glossary/aura" /%\}.

#### Inferring Type (`?` wut Runes)

**Using Conditionals for Inference by Branch**

You have learned about a few conditional runes (e.g., `?:` \{% tooltip\
label="wutcol" href="/language/hoon/reference/rune/wut#-wutcol" /%\} and`?.` \{% tooltip label="wutdot"\
href="/language/hoon/reference/rune/wut#-wutdot" /%\}), but other runes\
of the `?` family are used for branch-specialized type inference. The`?@` \{% tooltip label="wutpat"\
href="/language/hoon/reference/rune/wut#-wutpat" /%\}, `?^` \{% tooltip\
label="wutket" href="/language/hoon/reference/rune/wut#-wutket" /%\}, and`?~` \{% tooltip label="wutsig"\
href="/language/hoon/reference/rune/wut#-wutsig" /%\} conditionals each\
take three subexpressions, which play the same basic role as the\
corresponding subexpressions of `?:` wutcol—the first is the test\
condition, which evaluates to a flag `?`. If the test condition is\
true, the second subexpression is evaluated; otherwise the third. These\
second and third subexpressions are the ‘branches’ of the conditional.

There is also a `?=` \{% tooltip label="wuttis"\
href="/language/hoon/reference/rune/wut#-wuttis" /%\} rune for\
pattern-matching expressions by type, returning `%.y` for a match and`%.n` otherwise.

**`?=` wuttis Non-recursive Type Match Test**

The `?=` \{% tooltip label="wuttis"\
href="/language/hoon/reference/rune/wut#-wuttis" /%\} rune takes two\
subexpressions. The first subexpression should be a type. The second\
subexpression is evaluated and the resulting value is compared to the\
first type. If the value is an instance of the type, `%.y` is produced.\
Otherwise, `%.n`. Examples:

```hoon
> ?=(@ 12)
%.y

> ?=(@ [12 14])
%.n

> ?=(^ [12 14])
%.y

> ?=(^ 12)
%.n

> ?=([@ @] [12 14])
%.y

> ?=([@ @] [[12 12] 14])
%.n
```

`?=` \{% tooltip label="wuttis"\
href="/language/hoon/reference/rune/wut#-wuttis" /%\} expressions ignore

information:

```hoon
> ?=(@ud 0x12)
%.y

> ?=(@ux 'hello')
%.y
```

We haven't talked much about types that are made with a type constructor\
yet. We'll discuss these more soon, but it's worth pointing out that\
every list type qualifies as such, and hence should not be used with`?=`:

```hoon
> ?=((list @) ~[1 2 3 4])
fish-loop
```

Using these non-basic constructed types with the `?=` \{% tooltip\
label="wuttis" href="/language/hoon/reference/rune/wut#-wuttis" /%\} rune\
results in a `fish-loop` error.

The `?=` \{% tooltip label="wuttis"\
href="/language/hoon/reference/rune/wut#-wuttis" /%\} rune is\
particularly useful when used with the `?:` \{% tooltip label="wutcol"\
href="/language/hoon/reference/rune/wut#-wutcol" /%\} rune, because in\
these cases Hoon uses the result of the `?=` wuttis evaluation to infer\
type information. To see how this works lets use `=/` \{% tooltip\
label="tisfas" href="/language/hoon/reference/rune/tis#-tisfas" /%\} to\
define a

, `b`, as a\
generic noun:

```hoon
> =/(b=* 12 b)
12
```

The inferred type of the final `b` is just `*`, because that's how `b`\
was defined earlier. We can see this by using `?` in the Dojo to see\
the product type:

```hoon
> ? =/(b=* 12 b)
  *
12
```

(Remember that `?` isn't part of Hoon -- it's a Dojo-specific\
instruction.)

Let's replace that last `b` with a `?:` \{% tooltip label="wutcol"\
href="/language/hoon/reference/rune/wut#-wutcol" /%\} expression whose\
condition subexpression is a `?=` \{% tooltip label="wuttis"\
href="/language/hoon/reference/rune/wut#-wuttis" /%\} test. If `b` is an`@`, it'll produce `[& b]`; otherwise `[| b]`:

```hoon
> =/(b=* 12 ?:(?=(@ b) [& b] [| b]))
[%.y 12]
```

You can't see it here, but the inferred type of `b` in `[& b]` is `@`.\
That subexpression is only evaluated if `?=(@ b)` evaluates as true;\
hence, Hoon can safely infer that `b` must be an \{% tooltip label="atom"\
href="/glossary/atom" /%\} in that subexpression. Let's set `b` to a\
different initial value but leave everything else the same:

```hoon
> =/(b=* [12 14] ?:(?=(@ b) [& b] [| b]))
[%.n 12 14]
```

You can't see it here either, but the inferred type of `b` in `[| b]` is`^`. That subexpression is only evaluated if `?=(@ b)` evaluates as\
false, so `b` can't be an atom there. It follows that it must be\
a

.

**The Type Spear**

What if you want to see the inferred type of `b` for yourself for each\
conditional branch? One way to do this is with the _type spear_. The`!>` \{% tooltip label="zapgar"\
href="/language/hoon/reference/rune/zap#-zapgar" /%\} rune takes one\
subexpression and constructs a \{% tooltip label="cell"\
href="/glossary/cell" /%\} from it. The subexpression is evaluated and\
becomes the tail of the product cell, with a `q` \{% tooltip label="face"\
href="/glossary/face" /%\} attached. The head of the product cell is the\
inferred type of the subexpression.

```hoon
> !>(15)
[#t/@ud q=15]

> !>([12 14])
[#t/[@ud @ud] q=[12 14]]

> !>((add 22 55))
[#t/@ q=77]
```

The `#t/` is the pretty-printer's way of indicating a type.

To get just the inferred type of a expression, we only want the head of\
the `!>` product, `-`. Thus we should use our mighty weapon, the type\
spear, `-:!>`.

```hoon
> -:!>(15)
#t/@ud

> -:!>([12 14])
#t/[@ud @ud]

> -:!>((add 22 55))
#t/@
```

Now let's try using `?=` \{% tooltip label="wuttis"\
href="/language/hoon/reference/rune/wut#-wuttis" /%\} with `?:` \{%\
tooltip label="wutcol" href="/language/hoon/reference/rune/wut#-wutcol"\
/%\} again. But this time we'll replace `[& b]` with `[& -:!>(b)]` and`[| b]` with `[| -:!>(b)]`. With `b` as `12`:

```hoon
> =/(b=* 12 ?:(?=(@ b) [& -:!>(b)] [| -:!>(b)]))
[%.y #t/@]
```

… and with `b` as `[12 14]`:

```hoon
> =/(b=* [12 14] ?:(?=(@ b) [& -:!>(b)] [| -:!>(b)]))
[%.n #t/[* *]]
```

In both cases, `b` is defined initially as a generic \{% tooltip\
label="noun" href="/glossary/noun" /%\}, `*`. But when using `?:` with`?=(@ b)` as the test condition, `b` is inferred to be an \{% tooltip\
label="atom" href="/glossary/atom" /%\}, `@`, when the condition is true;\
otherwise `b` is inferred to be a \{% tooltip label="cell"\
href="/glossary/cell" /%\}, `^` (identical to `[* *]`).

**`mint-vain`**

Expressions of the form `?:(?=(a b) c d)` should only be used when the\
previously inferred type of `b` isn't specific enough to determine\
whether it nests under `a`. This kind of expression is only to be used\
when `?=` can reveal new type information about `b`, not to confirm\
information Hoon already has.

For example, if you have a wing expression (e.g., `b`) that is already\
known to be an atom, `@`, and you use `?=(@ b)` to test whether `b` is\
an atom, you'll get a \{% tooltip label="mint-vain"\
href="/language/hoon/reference/hoon-errors#mint-vain-and-mint-lost" /%\}\
crash. The same thing happens if `b` is initially defined to be a \{%\
tooltip label="cell" href="/glossary/cell" /%\} `^`:

```hoon
> =/(b=@ 12 ?:(?=(@ b) [& b] [| b]))
mint-vain

> =/(b=^ [12 14] ?:(?=(@ b) [& b] [| b]))
mint-vain
```

In the first case it's already known that `b` is an atom. In the second\
case it's already known that `b` isn't an atom. Either way, the check\
is superfluous and thus one of the `?:` wutcol branches will never be\
taken. The `mint-vain` crash indicates that it's provably the case one\
of the branches will never be taken.

**`?@` wutpat Atom Match Tests**

The `?@` \{% tooltip label="wutpat"\
href="/language/hoon/reference/rune/wut#-wutpat" /%\} rune takes three\
subexpressions. The first is evaluated, and if its value is an instance\
of `@`, the second subexpression is evaluated. Otherwise, the third\
subexpression is evaluated.

```hoon
> =/(b=* 12 ?@(b %atom %cell))
%atom

> =/(b=* [12 14] ?@(b %atom %cell))
%cell
```

If the second `?@` \{% tooltip label="wutpat"\
href="/language/hoon/reference/rune/wut#-wutpat" /%\} subexpression is\
evaluated, Hoon correctly infers that `b` is an \{% tooltip label="atom"\
href="/glossary/atom" /%\}. if the third subexpression is evaluated,\
Hoon correctly infers that `b` is a \{% tooltip label="cell"\
href="/glossary/cell" /%\}.

```hoon
> =/(b=* 12 ?@(b [%atom -:!>(b)] [%cell -:!>(b)]))
[%atom #t/@]

> =/(b=* [12 14] ?@(b [%atom -:!>(b)] [%cell -:!>(b)]))
[%cell #t/[* *]]
```

If the inferred type of the first `?@` \{% tooltip label="wutpat"\
href="/language/hoon/reference/rune/wut#-wutpat" /%\} subexpression nests\
under `@` then one of the conditional branches provably never runs.\
Attempting to evaluate the expression results in a \{% tooltip\
label="mint-vain"\
href="/language/hoon/reference/hoon-errors#mint-vain-and-mint-lost" /%\}:

```hoon
> ?@(12 %an-atom %not-an-atom)
mint-vain

> ?@([12 14] %an-atom %not-an-atom)
mint-vain

> =/(b=@ 12 ?@(b %an-atom %not-an-atom))
mint-vain

> =/(b=^ [12 14] ?@(b %an-atom %not-an-atom))
mint-vain
```

`?@` \{% tooltip label="wutpat"\
href="/language/hoon/reference/rune/wut#-wutpat" /%\} should only be used\
when it allows for Hoon to infer new type information; it shouldn't be\
used to confirm type information Hoon already knows.

**`?^` wutket Cell Match Tests**

The `?^` \{% tooltip label="wutket"\
href="/language/hoon/reference/rune/wut#-wutket" /%\} rune is just like`?@` \{% tooltip label="wutpat"\
href="/language/hoon/reference/rune/wut#-wutpat" /%\} except it tests for\
a cell match instead of for an atom match. The first subexpression is\
evaluated, and if the resulting value is an instance of `^` the second\
subexpression is evaluated. Otherwise, the third is run.

```hoon
> =/(b=* 12 ?^(b %cell %atom))
%atom

> =/(b=* [12 14] ?^(b %cell %atom))
%cell
```

Again, if the second subexpression is evaluated Hoon infers that `b` is\
a cell; if the third, Hoon infers that `b` is an atom. If one of the\
conditional branches is provably never evaluated, the expression crashes\
with a \{% tooltip label="mint-vain"\
href="/language/hoon/reference/hoon-errors#mint-vain-and-mint-lost" /%\}:

```hoon
> =/(b=@ 12 ?^(b %cell %atom))
mint-vain

> =/(b=^ 12 ?^(b %cell %atom))
nest-fail
```

**Tutorial: Leaf Counting**

can be understood as\
binary trees in which each 'leaf' of the tree is an \{% tooltip\
label="atom" href="/glossary/atom" /%\}. Let's look at a program that\
takes a noun and returns the number of leaves in it, i.e., the number of\
atoms.

```hoon

|=  a=*
^-  @
?@  a
  1
(add $(a -.a) $(a +.a))
```

Save this as `/gen/leafcount.hoon` in your fakeship's \{% tooltip\
label="pier" href="/glossary/pier" /%\} and run it from the \{% tooltip\
label="Dojo" href="/glossary/dojo" /%\}:

```hoon
> +leafcount 12
1

> +leafcount [12 14]
2

> +leafcount [12 [63 [829 12] 23] 13]
6
```

This program is pretty simple. If the noun `a` is an atom, then it's a\
tree of one leaf; return `1`. Otherwise, the number of leaves in `a` is\
the sum of the leaves in the head, `-.a`, and the tail, `+.a`.

We have been careful to use `-.a` and `+.a` only on a branch for which`a` is proved to be a

\-- then it's safe to treat `a` as having a head and a tail.

**Tutorial: Cell Counting**

Here's a program that counts the number of cells in a noun:

```hoon

<div data-gb-custom-block data-tag="copy" data-0='true'></div>

|=  a=*
=|  c=@
|-  ^-  @
?@  a
  c
%=  $
  c  $(c +(c), a -.a)
  a  +.a
==
```

Save this as `/gen/cellcount.hoon` and run it from the Dojo:

```hoon
> +cellcount 12
0

> +cellcount [12 14]
1

> +cellcount [12 14 15]
2

> +cellcount [[12 [14 15]] 15]
3

> +cellcount [[12 [14 15]] [15 14]]
4

> +cellcount [[12 [14 15]] [15 [14 22]]]
5
```

This code is a little more tricky. The basic idea, however, is simple.\
We have a counter value, `c`, whose initial value is `0` (`=|` \{%\
tooltip label="tisbar" href="/language/hoon/reference/rune/tis#-tisbar"\
/%\} pins the

of the\
value with the given \{% tooltip label="face" href="/glossary/face"\
/%\}). We trace through the noun \`a\`, adding \`1\` to \`c\` every time we\
come across a cell. For any part of the noun that is just an atom, \`c\`\
is returned unchanged.

What makes this program is little harder to follow is that it recurses\
within a recursion call. The first recursion expression on line 6 makes\
changes to two face values: `c`, the counter, and `a`, the input noun.\
The new value for `c` defined in the line `$(c +(c), a -.a)` is another\
recursion call (this time in irregular syntax). The new value for `c`\
is to be the result of running the same function on the the head of `a`,`-.a`, and with `1` added to `c`. We add `1` because we know that `a`\
must be a

. Otherwise,\
we're asking for the number of cells in the rest of `-.a`.

Once that new value for `c` is computed from the head of `a`, we're\
ready to check the tail of `a`, `+.a`. We've already got everything we\
want from `-.a`, so we throw that away and replace `a` with `+.a`.

#### Lists

You learned about lists earlier in the chapter, but we left out a little\
bit of information about the way Hoon understands \{% tooltip\
label="list" href="/glossary/list" /%\} types.

A non-null list is a cell. If `b` is a non-null list then the head of`b` is the first item of `b` _with an `i` face on it_. The tail of `b`\
is the rest of the list. The 'rest of the list' is itself another lis&#x74;_&#x77;ith a `t`_

_on it_.\
We can (and should) use these `i` and `t` faces in list functions.

To illustrate: let's say that `b` is the list of the atoms `11`, `22`,\
and `33`. Let's construct this in stages:

```hoon
[i=11 t=[rest-of-list-b]]

[i=11 t=[i=22 t=[rest-of-list-b]]]

[i=11 t=[i=22 t=[i=33 t=~]]]
```

(There are lists of every type. Lists of `@ud`, `@ux`, `@` in general,`^`, `[^ [@ @]]`, etc. We can even have lists of lists of `@`, `^`, or`?`, etc.)

**Tutorial: List Spanning Values**

Here's a program that takes atoms `a` and `b` and returns a list of all\
atoms from `a` to `b`:

```hoon

|=  [a=@ b=@]
^-  (list @)
?:  (gth a b)
  ~
[i=a t=$(a +(a))]
```

This program is very simple. It takes two `@` as input, `a` and `b`,\
and returns a `(list @)`, i.e., a list of `@`. If `a` is greater than`b` the

is finished:\
return the null list \`\~\`. Otherwise, return a non-null list: a pair in\
which the head is \`a\` with an \`i\` \{% tooltip label="face"\
href="/glossary/face" /%\} on it, and in which the tail is another list\
with the \`t\` face on it. This embedded list is the product of a\
recursion call: add \`1\` to \`a\` and run the function again.

Save this code as `/gen/gulf.hoon` and run it from the Dojo:

```hoon
> +gulf [1 10]
~[1 2 3 4 5 6 7 8 9 10]

> +gulf [10 20]
~[10 11 12 13 14 15 16 17 18 19 20]

> +gulf [20 10]
~
```

Where are all the `i`s and `t`s??? For the sake of neatness the Hoon\
pretty-printer doesn't show the `i` and `t` faces of lists, just the\
items.

In fact, we could have left out the `i` and `t` faces in the program\
itself:

```hoon

|=  [a=@ b=@]
^-  (list @)
?:  (gth a b)
  ~
[a $(a +(a))]
```

Because there is a cast to a `(list @)` on line 2, Hoon will silently\
include `i` and `t` faces for the appropriate places of the \{% tooltip\
label="noun" href="/glossary/noun" /%\}. Remember that \{% tooltip\
label="faces" href="/glossary/face" /%\} are recorded in the type\
information of the noun in question, not as part of the noun itself.

We called this program `gulf.hoon` because it replicates the \{% tooltip\
label="gulf" href="/language/hoon/reference/stdlib/2b#gulf" /%\} function\
in the Hoon standard library:

```hoon
> (gulf 1 10)
~[1 2 3 4 5 6 7 8 9 10]

> (gulf 10 20)
~[10 11 12 13 14 15 16 17 18 19 20]
```

**`?~` wutsig Null Match Test**

The `?~` \{% tooltip label="wutsig"\
href="/language/hoon/reference/rune/wut#-wutsig" /%\} rune is a lot like`?@` \{% tooltip label="wutpat"\
href="/language/hoon/reference/rune/wut#-wutpat" /%\} and `?^` \{% tooltip\
label="wutket" href="/language/hoon/reference/rune/wut#-wutket" /%\}. It\
takes three subexpressions, the first of which is evaluated to see\
whether the result is `~` null. If so, the second subexpression is\
evaluated. Otherwise, the third one is evaluated.

```hoon
> =/(b=* ~ ?~(b %null %not-null))
%null

> =/(b=* [12 13] ?~(b %null %not-null))
%not-null
```

The inferred type of `b` must not already be known to be null or\
non-null; otherwise, the expression will crash with a \{% tooltip\
label="mint-vain"\
href="/language/hoon/reference/hoon-errors#mint-vain-and-mint-lost" /%\}:

```hoon
> =/(b=~ ~ ?~(b %null %not-null))
mint-vain

> =/(b=^ [10 12] ?~(b %null %not-null))
mint-vain

> ?~(~ %null %not-null)
mint-vain
```

Hoon will infer that `b` either is or isn't null based on which `?~`\
branch is evaluated after the test.

**Using `?~` wutsig With Lists**

`?~` \{% tooltip label="wutsig"\
href="/language/hoon/reference/rune/wut#-wutsig" /%\} is especially\
useful for working with lists. Is a list null, or not? You probably\
want to do different things based on the answer to that question.\
Above, we used a pattern of `?:` \{% tooltip label="wutcol"\
href="/language/hoon/reference/rune/wut#-wutcol" /%\} and `?=` \{% tooltip\
label="wuttis" href="/language/hoon/reference/rune/wut#-wuttis" /%\} to\
answer the question, but `?~` wutsig will let us know in one step.\
Here's a program using `?~` wutsig to calculate the number of items in a\
list of atoms:

```hoon

<div data-gb-custom-block data-tag="copy" data-0='true'></div>

|=  a=(list @)
=|  c=@
|-  ^-  @
?~  a
  c
$(c +(c), a t.a)
```

This function takes a list of `@` and returns an `@`. It uses `c` as a\
counter value, initially set at `0` on line 2. If `a` is `~` (i.e., a\
null list) then the computation is finished; return `c`. Otherwise `a`\
must be a non-null

, in\
which case there is a recursion to the \`|-\` \{% tooltip label="barhep"\
href="/language/hoon/reference/rune/bar#--barhep" /%\} on line 3, but with\
\`c\` incremented, and with the head of the list \`a\` thrown away.

It's important to note that if `a` is a list, you can only use `i.a` and`t.a` after Hoon has inferred that `a` is non-null. A null list has no`i` or `t` in it! You'll often use `?~` to distinguish the two kinds of\
list (null and non-null). If you use `i.a` or `t.a` without showing\
that `a` is non-null you'll get a `find-fork` crash.

A non-null `list` is called a `lest`.

Save the above code as `/gen/lent.hoon` and run it from the Dojo:

```hoon
> +lent ~[11 22 33]
3

> +lent ~[11 22 33 44 55 77]
6

> +lent ~[0xff 0b11 'howdy' %hello]
4
```

**Tutorial: Converting a Noun to a List of its Leaves**

Here's a program that takes a noun and returns a \{% tooltip label="list"\
href="/glossary/list" /%\} of its 'leaves' (atoms) in order of their\
appearance:

```hoon

|=  a=*
=/  lis=(list @)  ~
|-  ^-  (list @)
?@  a
  [i=a t=lis]
$(lis $(a +.a), a -.a)
```

The input noun is `a`. The list of atoms to be output is `lis`, which\
is given an initial value of `~`. If `a` is just an atom, return a\
non-null list whose head is `a` and whose tail is `lis`. Otherwise, the\
somewhat complicated recursion `$(lis $(a +.a), a -.a)` is evaluated, in\
effect looping back to the `|-` with modifications made to `lis` and`a`.

The modification to `lis` in line 6 is to `$(a +.a)`. The latter is a\
recursion to `|-` but with `a` replaced by its tail. This evaluates to\
the list of `@` in the tail of `a`. So `lis` becomes the list of atoms\
in the tail of `a`, and `a` becomes the head of `a`, `-.a`.

Save the above code as `/gen/listleaf.hoon` and run it from the Dojo:

```hoon
> +listleaf [[[[12 13] [33 22] 12] 11] 33]
~[12 13 33 22 12 11 33]
```

#### Other Kinds of Type Inference

So far you've learned about four kinds of type inference:

1. literals
2. explicit casts
3. gate sample definitions
4. branch specialization using runes in the `?` family

There are several other ways that Hoon infers type. Any rune expression\
that evaluates to a `?` flag, e.g., `.=` \{% tooltip label="dottis"\
href="/language/hoon/reference/rune/dot#-dottis" /%\}, will be inferred\
from accordingly. The `.+` \{% tooltip label="dotlus"\
href="/language/hoon/reference/rune/dot#-dotlus" /%\} rune always\
evaluates to an `@`, and Hoon knows that too. The cell constructor\
runes, `:-` \{% tooltip label="colhep"\
href="/language/hoon/reference/rune/col#--colhep" /%\}, `:+` \{% tooltip\
label="collus" href="/language/hoon/reference/rune/col#-collus" /%\},`:^` \{% tooltip label="colket"\
href="/language/hoon/reference/rune/col#-colket" /%\}, and `:*` \{%\
tooltip label="coltar" href="/language/hoon/reference/rune/col#-coltar"\
/%\} are all known to produce cells.

More subtly, the `=+` \{% tooltip label="tislus"\
href="/language/hoon/reference/rune/tis#-tislus" /%\}, `=/` \{% tooltip\
label="tisfas" href="/language/hoon/reference/rune/tis#-tisfas" /%\}, and`=|` \{% tooltip label="tisbar"\
href="/language/hoon/reference/rune/tis#-tisbar" /%\} runes modify the \{%\
tooltip label="subject" href="/glossary/subject" /%\} by pinning values\
to the head. Hoon infers from this that the subject has a new type: a\
cell whose head is the type of the pinned value and whose tail is the\
type of the (old) subject.

In general, anything that modifies the subject modifies the type of the\
subject. Type inference can work in subtle ways for various\
expressions. However, we have covered enough that it should be\
relatively clear how to anticipate how type inference works for the vast\
majority of ordinary use cases.

### Auras as 'Soft' Types

It's important to understand that Hoon's type system doesn't enforce \{%\
tooltip label="auras" href="/glossary/aura" /%\} as strictly as it does\
other types. Auras are 'soft' type information. To see how this works,\
we'll take you through the process of converting the aura of an \{%\
tooltip label="atom" href="/glossary/atom" /%\} to another aura.

Hoon makes an effort to enforce that the correct aura is produced by an\
expression:

```hoon
> ^-(@ud 0x10)
nest-fail

> ^-(@ud 0b10)
nest-fail

> ^-(@ux 100)
nest-fail
```

But there are ways around this. First, you can cast to a more general\
aura, as long as the current aura nests under the cast aura. E.g., `@ub`\
to `@u`, `@ux` to `@u`, `@u` to `@`, etc. By doing this you're\
essentially telling Hoon to throw away some aura information:

```hoon
> ^-(@u 0x10)
16

> ? ^-(@u 0x10)
  @u
16

> ^-(@u 0b10)
2

> ? ^-(@u 0b10)
  @u
2
```

In fact, you can cast any atom all the way to the most general case `@`:

```hoon
> ^-(@ 0x10)
16

> ? ^-(@ 0x10)
  @
16

> ^-(@ 0b10)
2

> ? ^-(@ 0b10)
  @
2
```

Anything of the general \{% tooltip label="aura" href="/glossary/aura"\
/%\} `@` can, in turn, be cast to more specific auras. We can show this\
by embedding a cast expression inside another cast:

```hoon
> ^-(@ud ^-(@ 0x10))
16

> ^-(@ub ^-(@ 0x10))
0b1.0000

> ^-(@ux ^-(@ 10))
0xa
```

Hoon uses the outermost cast to infer the type:

```hoon
> ? ^-(@ub ^-(@ 0x10))
  @ub
0b1.0000
```

As you can see, an atom with one aura can be converted to another aura.\
For a convenient shorthand, you can do this conversion with irregular\
cast syntax, e.g. `` `@ud` ``, rather than using the `^-` \{% tooltip\
label="kethep" href="/language/hoon/reference/rune/ket#--kethep" /%\}\
rune twice:

```hoon
> `@ud`0x10
16

> `@ub`0x10
0b1.0000

> `@ux`10
0xa
```

This is what we mean when we call auras 'soft' types. The above examples\
show that the programmer can get around the type system for auras by\
casting up to `@` and then back down to the specific aura, say `@ub`; or\
by casting with `` `@ub` `` for short.

**Note**: there is currently a type system issue that causes some of\
these functions to fail when passed a list `b` after some type inference\
has been performed on `b`. For an illustration of the bug, let's set`b` to be a `(list @)` of `~[11 22 33 44]` in the Dojo:

```hoon
> =b `(list @)`~[11 22 33 44]

> b
~[11 22 33 44]
```

Now let's use `?~` \{% tooltip label="wutsig"\
href="/language/hoon/reference/rune/wut#-wutsig" /%\} to prove that `b`\
isn't null, and then try to snag it:

```hoon
> ?~(b ~ (snag 0 b))
nest-fail
```

The problem is that \{% tooltip label="++snag"\
href="/language/hoon/reference/stdlib/2b#snag" /%\} is expecting a raw\
list, not a list that is known to be non-null.

You can cast `b` back to `(list)` to work around this:

```hoon
> ?~(b ~ (snag 0 `(list)`b))
11
```

#### Pattern Matching and Assertions

To summarize, as values get passed around and checked at various points,\
the Hoon compiler tracks what the possible data structure or \{% tooltip\
label="mold" href="/glossary/mold" /%\} looks like. The following runes\
are particularly helpful when inducing the compiler to infer what it\
needs to know:

* `?~` \{% tooltip label="wutsig"\
  href="/language/hoon/reference/rune/wut#-wutsig" /%\} asserts non-null.
* `?^` \{% tooltip label="wutket"\
  href="/language/hoon/reference/rune/wut#-wutket" /%\} asserts cell.
* `?@` \{% tooltip label="wutpat"\
  href="/language/hoon/reference/rune/wut#-wutpat" /%\} asserts atom.
* `?=` \{% tooltip label="wuttis"\
  href="/language/hoon/reference/rune/wut#-wuttis" /%\} tests for a\
  pattern match in type.

There are two additional assertions which can be used with the type\
system:

* `?>` \{% tooltip label="wutgar"\
  href="/language/hoon/reference/rune/wut#-wutgar" /%\} is a positive\
  assertion (`%.y` or crash).
* `?<` \{% tooltip label="wutgal"\
  href="/language/hoon/reference/rune/wut#-wutgal" /%\} is a negative\
  assertion (`%.n` or crash).

If you are running into `find-fork` errors in more complicated data\
structures (like

or\
JSONs), consider using these assertions to guide the typechecker.
