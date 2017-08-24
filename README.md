# plt-errata
Collection of errata for the book _[Implementing Programming Languages](http://www1.digitalgrammars.com/ipl-book)_ by Aarne Ranta.

To add a new erratum, create an issue or pull request.
Please use Github Markdown syntax and adhere to the style of this page.
I will then add the erratum below.

## Known errata

This includes the errata listed on the book website.

### Chapter 1, Compilation Phases

p. 10 (and also later): it is stated that Python is an untyped language. By this we mean that Python has no compile-time type checking. But it does have a run-time notion of types, known as dynamic typing.

### Chapter 2, Grammars

p. 25: last line: `show (interpret e)` should be `show (eval e)`.

p. 27: too many classes in the Java example have the name `EAdd`. Should be `EAdd`, `ESub`, `EMul`, `EDiv`.

### Chapter 3, Lexing and Parsing

p. 41: The Empty construction could be simplified, by making the
initial state final and saving the ε-transition.

p. 41: The Sequence construction can be simplified by using the
initial state of the first automaton as initial state of the sequence
and the final state of the second automaton as the final state of the
sequence.  This saves 2 ε-transitions (and would correspond to the
example on p. 43).

p. 43: This NFA is not generated by the algorithm on p. 41.  It misses
ε-transitions.  On both paths, there should be 5 ε-transitions, as 2
are generated by the Union and 3 by the Sequence construction.

p. 43: The result of the subset construction should have `0,1,5` as
initial state instead of just `0`.  Again on p. 44.

p. 46: The figure should say `m b's` and `n b's` on the arcs going to
the final states (instead of `m a's` and `n a's`).

p. 53: The line with `%start_pExp` should be deleted from the table (or explained).
It is rather confusing than helpful.

### Chapter 4, Type Checking

#### 4.7 The validity of statements and function definitions

The judgement for checking statements should be formulated as

```
Γ ⊢ s ⇒ Γ'
```

Declarations such as `int x;` extend the typing context.
This would allow to define checking of a sequence of statements
in the natural way.  Actually the Haskell implementation in 4.11
does it exactly like I suggest here.

4.9 would have to be rewritten.  What is going on in 4.9 currently
is that the state monad formulation `Γ ⊢ s ⇒ Γ'` is replaced by a
context monad formulation `Γ ⊢ ss valid` at the cost of modularity:
we can only handle statement sequences.

#### 4.8 Declarations and block structures

This section should discuss the scopes for `if` and `while` (see
errata for Section 5.3).

#### 4.10 Annotating type checkers

p. 69: the pseudo-code for `infer(Γ,a+b)` is wrong in that it removes the annotations from the subexpressions of the addition expression.  The correct return statement would be

```haskell
return [['a : t] + ['b : t] : t]
```

p. 70: the pseudo-code for `infer(Γ,a+b)` has the same problem as on p. 69

#### 4.11 Type checker in Haskell

p. 73: in `checkExp` code, `if (typ2 = typ)` should be `if typ2 == typ`.
It could also be written as

```haskell
unless (typ2 == typ) $ fail $
  "type of " ++ ...
```

In `checkStm`, there are several errors.  The correct code is:

```haskell
checkStm :: Env -> Stm -> Err Env
checkStm env s = case s of
  SExp exp -> do
    inferExp env exp
    return env
  SDecl typ x ->
    updateVar env x typ
  SWhile exp stm -> do
    checkExp env Type_bool exp
    checkStm (newBlock env) stm
    return env
```

p. 74, lines 1-3: Use `s` instead of `x` as variable name for statement.

### Chapter 5, Interpreters

p. 82: Rule `γ ⊢ x ⇓ v`: The `v` is type-set in the wrong font.

#### 5.3 Statements

It is not clear how the statement interpreter `γ ⊢ s ⇓ γ'` would deal
with `return` statements which are not at the end of the function (or
`break` statements in while loops).  I suggest to change it to

```
γ ⊢ s ⇓ ⟨r,γ'⟩
```

where `r ::= continue | return v` is the result of the statement:
usually `continue`, but `return v` for a return statement.  The
execution of sequencing of statements will discard the rest of the
statements once the result is `return v`.

p. 84: The specified interpreter gives the wrong result for

```c
int x = 0;
int y = 0;
while (x++ < 1) int y = 1;
return y;
```

It gives `1`, while the correct result is `0`.
The problem is that the body of the `while` will overwrite the value of the shadowed `y`.
To fix this, the body of a `while` has to be treated as if in its own block.

See http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf:

> An iteration statement is a block whose scope is a strict subset of
> the scope of its enclosing block.  The loop body is also a block
> whose scope is a strict subset of the scope of the iteration
> statement. (Section 6.8.5, sentence 5, page number 135, absolute page 147)

Possible fix: replace premise `γ′ ⊢ s ⇓ γ″` by  `γ′. ⊢ s ⇓ γ″.γ₀` in the
first rule for `while`.

`if` has to be fixed in a similar way, see section 6.8.4, sentence 3.

```c
int y = 0;
if (1) int y = 1; else int y = 2;
return y;
```

This should return `0`, but the current interpreter will return `1`.

Possible fix: replace premise `γ′ ⊢ s ⇓ γ″` by  `γ′. ⊢ s ⇓ γ″.γ₀` in the
first rule for `if`.  Analogously for the second rule.

#### 5.7 Interpreting Java bytecode

p. 92: in the last rule for `ifeq L`, the code pointer should become `P+1` when `v != 0`

### Chapter 6, Code Generation

p. 108:  there is a `&lt;` in the class file template which should
just be `<`.

p. 112: "just a dummy `Object`".  Now, Java has class `Void` for that
purpose.  Change would affect the following code.

p. 105: The generated code for the while loop in the middle column contains `ifeq goto END`. It should be `ifeq END` without the goto.

### Chapter 7, Functional Programming Languages

#### 7.3 Anonymous functions

p. 128: C++ has had lambda functions since C++11.  This is also true for many other mainstream imperative languages nowadays, such as Java and C# -- so the comment about imperative languages on the previous page is a little misleading.

#### 7.9 Polymorphic type checking with unification

p. 142: in `infer(f,a)`: before `infer(a)`, substitution `γ₁` has to
be applied to the typing context.

### Chapter 8, The Language Design Space

p. 170: the `lin` rules for `TAll` and `TAny` generate a bogus condition. The proper rules are:

```
TAll kind = parenth ("\\p -> and [p x | x <-" ++ kind ++ "]") ;
TAny kind = parenth ("\\p -> or  [p x | x <-" ++ kind ++ "]") ;
```

p. 157: BNFC was not ported to Java, C, C++, etc.; rather, the mentioned languages were added as supported backends.

### Appendix A

p. 175: The arcs in this diagram are not really traceable.

### Appendix A

p. 193: dcmpl explanation should be "compare if >*"
