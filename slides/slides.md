---
---

# Dependently-Typed Language

---

# Types

Types provides guarantees even before running the program

## Improper assignment
<br>

```js{1|3|all}
string message = "hello";

int result = message; // ERROR: message is not an int
```

<br>
<br>

## Can solve the billion-dollar mistake
<br>

```ts{1|4|all}
bob : Person | null

// calculating full name
bob.firstName + bob.lastName // ERROR: bob may be null
```


---

# Dependent Types

Allows more flexibility in defining types, because it allows types to *depend* on terms:

```agda{1-3|5,6|all}
data Nat : Set where
  zero : Nat
  suc : Nat -> Nat

-- given two arguments(symbolized by _)
_+_ :: Nat -> Nat -> Nat
zero    + b = b
(suc a) + b = suc (a + b)
```


---

```agda{1-2|3-4|6-8|9-15|16-18|all}
-- parameterized type: needs a type A, and a Nat number describing the length of the vector
data Vec (A : Set) : Nat -> Set where
  -- symbolizes the empty array of an implicit type A
  [] : Vec A zero
  
  _::_ :
  	-- given an implicit number n
  	{n : Nat} ->
    -- given an element of type A
    A ->
    -- given a vector of type A, with length n
    Vec A n ->
    -- return a vector of type A, with length n+1
    Vec A (suc n)

vec : Vec Bool (suc (suc zero))
vec = False :: (True :: [])
```


---

Notice which kind of functions we can write with dependent-types:

```agda{1-4|5|7-8|all}
-- given two implicit arguments(type of vector A, length n)
-- given a vector of length *greater than* zero
-- returns the head of the *proved* non-empty vector
head : {A : Set}{n : Nat} → Vec A (suc n) → A
head (x :: xs) = x

concat : {A : Set}{m n : Nat} -> Vec A m -> Vec A n -> Vec A (m + n)
concat = ...
```


---

# Proofs as Programs

We can even use them to **express** *arbitrary* properties about our code, like that addition is associative:

```agda
+-assoc :
 -- given three numbers
 forall (a b c : Nat) →
  -- we provide a type that expresses the following equality
 (a + b) + c ≡ a + (b + c)
```

Dependent-types allows us to **express** those equalities and arbitrary properties, however we can only **prove** those properties to be true, if we provide a proof of that.

As per the famous Curry-Howard correspondence, we can write Proofs as Programs, i.e. we have to provide an instance of that type for the proof to be complete

```agda
+-assoc a b c =
  -- construct the equality proof
  -- ommitted for brevity
```

---

# The Language

## Expressions

$$
\begin{aligned}
        t, T ::= & \ x                     & \text{variable}                \\
        |        & \  \lambda x.t          & \text{abstraction}             \\
        |        & \  t\ t                 & \text{application}             \\
        |        & \ (t : T) \rightarrow T & \text{dependent function type} \\
        |        & \ \textbf{Type}         & \text{the "type" of types}
\end{aligned}
$$


---

# Module

A module is a list of declarations, that can use other declarations in the same module:

$$
\begin{aligned}
        Decl ::= & \ t : T & \text{type-signature} \\
        |        & \ t = t & \text{definition}     \\
        |        & \ DataDef & \text{data definition}
\end{aligned}
$$

---

# Examples

Identity-function:

```haskell{1|2|all}
id : (x : Type) -> (y : x) -> x
id = λx. λy. y

id Bool True
```


---

# Examples

Church-numeral:

```haskell{1-2|4-6|8-10|12-13|15-16|all}
nat : Type
nat = (x : Type) -> x -> (x -> x) -> x

-- zero
z : nat
z = λx. λzf. λsf. zf

-- successor function
s : nat -> nat
s = λn. λx. λzf. λsf. sf (n x zf sf)

one = s z
two = s one

plus : nat -> nat -> nat
plus = λx. λy. x nat y s
```


---

# Ideal Typechecker Inference rules

![dep-tt-rules](/imgs/dep-tt-rules-1.png)


---

# Manually-provided proof tree

Given the program:

```haskell
id : (x : Type) -> (y : x) -> x
id = λx. λy. y
```

We can manually derive this proof tree:

![id-proof-tree](/imgs/id-proof-tree.png)

The goal of our typechecker is to provide this proof tree automatically. However, we need to make a few changes in our typecheckers' rules to be able to do that.


---

# Type-checking

*Ok, I'm sold, dependent-types sound awesome! How do they work behind the curtains?*

As per any statically-typed language, we begin with the type-checker, which is a ..erm.. program to check types...

It is the typechecker that is responsible to provide all those static guarantees, and it does so by various rules that analyze the given code.

Usually a typechecker runs late in the compiler/language pipeline, after we have the AST(Abstract Syntax Tree).

---

It's important to note that we have some ways of building a typechecker, that directly affects the end-user:

- Explicitly requiring the programmer to provide types for every term
  ```haskell
  a, b, c : Nat
  b = 3
  c = 10
  a = b + c
  ```
  or more java-like:
  ```haskell
  Nat b = 3
  Nat c = 10
  Nat a = b + c
  ```
- Not requiring the programmer to provide types for the terms:
  ```haskell
  b = 3     -- b is inferred to Nat
  c = 10    -- c is inferred to Nat
  a = b + c -- a is inferred to Nat
  ```

---

# Bidirectional Type-checking

We'll be using a mix of the two ways called "Bidirectional type-checking", which will run on two modes:
- **infer**, given a term $x$ (inside a context $\Gamma$), return its type $A$.

	$\Gamma \vdash x \Rightarrow A$
  ```haskell
  inferType :: Context -> Term -> Type
  ```
- **check**, given a term $x$ and a type $A$(inside a context), verify if the term $x$ is of type $A$.

	$\Gamma \vdash x \Leftarrow A$
  ```haskell
  checkType :: Context -> Term -> Type -> Bool
  ```

<br>
<br>

*Note: context is list that contains all previous calculated type-signatures, variable and data definitions*

---

# Bidirectional type-checker inference rules

## Infer mode

![](/imgs/dep-tt-rules-infer.png)

---

# Check mode

![](/imgs/dep-tt-rules-check.png)

---

# Example

![](/imgs/bidirectional-example.png)


---

# Proof-specific code

There are two types of equality:

- Definitional equality, is used when two type expressions are equal if they beta-reduce to the same term, i.e. `plus z (s z)` is definitionally equal to `s z`
- Propositional equality, is used when the programmer defines new equality propositions and can use them for posterior theorems, i.e. given that `a+b = b+a`, we can use that to prove that `a+(b+c)` is propositionally equal to `(b+c)+a`.

For that we'll need to add:

$$
\begin{aligned}
        t, T ::= & \ ...                                                 \\
        |        & \ T = T            & \text{propositional-equality}    \\
        |        & \ refl             & \text{introduction of prop. eq.} \\
        |        & \ subst\ t\ by\ t  & \text{elimination of prop. eq.}
\end{aligned}
$$

---

# Proof-specific rules

![](/imgs/dep-tt-rules-3.png)


---

```haskell{1-2|4-5|6-12|13-15|16,22|17,20-25|18,19,26,27|all}
one_plus_one_eq_two : ((plus (s z) (s z)) = (s (s z)))
one_plus_one_eq_two = refl

plus_zero_n : (n : nat) -> ((plus z n) = n)
plus_zero_n = λn. refl

eq_trans :
  (A : Type) ->
  (x : A) -> (y : A) -> (z : A) ->
  (x = y) ->
  (y = z) ->
  (x = z)
eq_trans = λA x y z eq_xy eq_yz.
  -- goal is to arrive in a term with type (x = z)
  -- we only have eq_xy(x = y) and eq_yz(y = z)
  -- refl is an instance of type (x = x), for any x
  -- if we manipulate refl by using eq_xy (the proof that (x = y)), we get (x = y)
  -- if we manipulate it by using eq_yz, we get (x = z), which is our goal
  subst
    (
  	  subst
    	refl  -- x=x
      by
        eq_xy -- x=y
    )     -- x=y
  by
    eq_yz -- y=z
```
