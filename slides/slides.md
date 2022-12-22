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

---

## Billion-dollar mistake

```java
class Person { ... }

Person bob = null;

...

// calculating full name
String fullName = bob.firstName + bob.lastName; // RUNTIME ERROR
```

<br>

> I call it my **billion-dollar mistake** [...] I couldn’t resist the temptation to put in a **null reference** [...] This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years**.
> 
> – Tony Hoare, inventor of ALGOL W.

<br>

## A way to solve the mistake

```ts{1|4|all}
bob : Person | null

// calculating full name
bob.firstName + bob.lastName // STATIC ERROR: bob may be null
```


---

## Can require properties about code

```ts{1,5,6|2,7|all}
-- given the account balance, an amount
-- and a proof that the withdrawn amount is less than the balance
-- perform the operation
withdraw_from :
  (account : Account) →
  (amount : Nat) →
  (amount <= account.balance) →
  Nat
```


---

# Dependent Types

Consider this code:

```agda{1-3|5,6|8,9|all}
data Nat : Set where
  zero : Nat
  suc : Nat -> Nat

two : Nat            -- right-hand side is a type
two = suc (suc zero) -- right-hand side is a term

-- given two arguments(symbolized by _)
_+_ :: Nat -> Nat -> Nat
zero    + b = b
(suc a) + b = suc (a + b)
```

<br>
<br>
<br>
<br>
<br>
<br>
<br>

*Note: Example code is written in the Agda programming language*

<!--
Begin at 2:41
-->

---

Dependent types allows more flexibility in defining types, because it allows types to *depend* on terms:

```agda{1-2|3-4|6-8|9-15|16-18|all}
-- parameterized type: needs a type A, and a Nat number describing the length of the vector
data Vec (A : Set) : Nat -> Set where
  -- symbolizes the empty array of an implicit type A
  [] : Vec A zero
  
  _::_ :
  	-- given an implicit number n(compiler finds this "n")
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

```agda{1-4|5|all}
-- given two implicit arguments(type of vector A, length n)
-- given a vector of length *greater than* zero
-- returns the head of the *proved* non-empty vector
head : {A : Set}{n : Nat} → Vec A (suc n) → A
head (x :: xs) = x
```

Compared to a Java function that may throw an error at runtime
```java{0|all}
class List<T> {
	T head() {
		if (!this.at(0)) throw new Error("empty list!");
		return this.at(0);
	}
}
```

Concatenate function:
```agda{0|all}
concat : {A : Set}{m n : Nat} -> Vec A m -> Vec A n -> Vec A (m + n)
concat = ...
```


---

## Proofs as Programs

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

# Core Language

## Expressions

$$
\begin{aligned}
        t, T ::= & \ x                     & \text{variable}                \\
        |        & \  \lambda x.t          & \text{abstraction}             \\
        |        & \  t\ t                 & \text{application}             \\
        |        & \ (x : T) \rightarrow T & \text{dependent function type} \\
        |        & \ \textbf{Type}         & \text{the "type" of types}
\end{aligned}
$$

## Module
A module is a list of top-level declarations, that can use other declarations in the same module:

$$
\begin{aligned}
        Decl ::= & \ t : T & \text{type-signature} \\
        |        & \ t = t & \text{definition}
\end{aligned}
$$

<!--
Begin at 8:46
-->

---

# Examples

Identity-function:

```haskell{1|2|4-7|all}
id : (x : Type) -> (y : x) -> x
id = λx. λy. y

id Bool True -- will return True, which is of type Bool
-- id           : (x : Type) -> (y : x) -> x
-- id Bool      : (y : Bool) -> Bool
-- id Bool True : Bool
```


<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

*Note: from now on, example code is written on the language built by this thesis*


---

# Extension of the Language: Data Types

<br>
<br>

$$
\begin{aligned}
        Decl ::= & \ ... \\
        |        & \ data\ TConName\ Telescope\ :\ Type\ where\ ConstructorDef+ & \ \text{data definition}
\end{aligned}
$$

<br>
<br>
<br>

$$
\begin{aligned}
        t, T ::= & \ ... \\
        |        & \ TConName\ t* & \text{constructing a data type by applying a list of arguments}     \\
        |        & \ DConName\ t* & \text{instance of a data type} \\
  		|        & \ case\ t\ of\ Case* & \text{pattern matching of a term}
\end{aligned}
$$


---

# Examples

Natural number:

```haskell{1-4|6-8|9-14|all}
data Nat : Type where {
		Zero,
    Succ of (Nat)
}

one : Nat
one = Succ Zero
two = Succ one

plus : Nat → Nat → Nat
plus = λa. λb. case a of {
  Zero → b,
  Succ a' → Succ (plus a' b)
}
```


---

<!-- # Ideal Typechecker Inference rules

![dep-tt-rules](/imgs/dep-tt-rules-1.png)

-->

<!-- 
# Manually-provided proof tree

Given the program:

```haskell
id : (x : Type) -> (y : x) -> x
id = λx. λy. y
```

We can manually derive this proof tree:

![id-proof-tree](/imgs/id-proof-tree.png)

The goal of our typechecker is to provide this proof tree automatically. However, we need to make a few changes in our typecheckers' rules to be able to do that.

-->

# Type-checking

*Ok, I'm sold, dependent-types sound awesome! How do they work behind the scenes?*

As per any statically-typed language, we begin with the type-checker, which is a ..erm.. program to check types...

It is the typechecker that is responsible to provide all those static guarantees, and it does so by various rules that analyze the given code.

Usually a typechecker runs late in the compiler/language pipeline, after we have the AST(Abstract Syntax Tree).

<!--
Begin at 12:20
-->

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
  inferType :: Context -> Term -> Maybe Type
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

## Check mode

![](/imgs/dep-tt-rules-check.png)


---

## Example

![](/imgs/bidirectional-example.png)


---

# Proof-specific code

There are two types of equality:

- Definitional equality, is used when two type expressions are equal if they beta-reduce to the same term, i.e. `plus Zero (Succ Zero)` is definitionally equal to `Succ Zero`
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

<!--
Begin at 15:25
-->

---

## Proof-specific rules

![](/imgs/dep-tt-rules-3.png)


---

```haskell{1-2|4-5|7-8|10-14|16|18-22|23-25|all}
one_eq_one : 1 = 1
one_eq_one = refl

one_plus_one_eq_two : ((plus 1 1) = 2)
one_plus_one_eq_two = refl

plus_zero_n : (n : Nat) -> ((plus Zero n) = n)
plus_zero_n = λn. refl

-- applies a function to a equality(congruence)
cong : (A B : Type) -> (f : (A -> B)) -> (x y : A) ->
  x = y ->
  (f x) = (f y)
cong = λA B f x y eq_xy. subst refl by eq_xy

suc_fn = λn. Succ n -- because in our language a data constructor is not a function

two_eq_two : 2 = 2
-- two_eq_two can be proved by "refl" or:
-- 2=2 because we're applying cong with the suc function on the 1=1 proof
--           cong suc_fn one_eq_one
two_eq_two = cong Nat Nat suc_fn 1 1 one_eq_one
--                       one_eq_one : 1=1
--           cong suc_fn one_eq_one : (Succ 1)=(Succ 1)
--           cong suc_fn one_eq_one : 2=2

```

<!--
Begin at 16:42
-->

---

<!--
plus_n_zero : (n : Nat) -> ((plus n Zero) = n)
plus_n_zero = λn. case n of {
  Zero -> refl,
  --         cong suc_fn (plus_n_zero n')
  Succ n' -> cong Nat Nat suc_fn (plus n' Zero) n' (plus_n_zero n')
  -- cong Succ (plus_n_zero n'):
  --                   (plus_n_zero n') : (n' + 0) = n'
  --         cong suc_fn (plus_n_zero n') : (Succ n' + 0) = Succ n'
  --         cong suc_fn (plus_n_zero n') : (n + 0) = n
}
-->


```haskell{1-3|4-5|6-7|8-12|all}
-- using cong for a proof by induction
-- plus_assoc : (m + n) + p = m + (n + p)
plus_assoc : (m n p: Nat) -> ((plus (plus m n) p) = (plus m (plus n p)))
plus_assoc = λm. λn. λp. case m of {
  Zero -> refl,
  --         cong suc_fn (plus_assoc m' n p)
  Succ m' -> cong Nat Nat suc_fn ((plus (plus m' n) p)) (plus m' (plus n p)) (plus_assoc m' n p)
  -- cong suc_fn (plus_assoc m' n p):
  --                     (plus_assoc m' n p) : ((m' + n) + p) = (m' + (n + p)))
  --         cong suc_fn (plus_assoc m' n p) : Succ ((m' + n) + p) = Succ (m' + (n + p)))
  --         cong suc_fn (plus_assoc m' n p) : ((Succ m' + n) + p) = (Succ m' + (n + p)))
  --         cong suc_fn (plus_assoc m' n p) : ((m + n) + p) = (m + (n + p)))
}
```

<!--
Begin at 20:17
End at 22:09
  --         cong suc_fn (plus_assoc m' n p) : ((m + n) + p) = (m + (n + p)))
  --         cong suc_fn (plus_assoc m' n p) : ((Succ m' + n) + p) = (Succ m' + (n + p)))
  --         cong suc_fn (plus_assoc m' n p) : (Succ (m' + n) + p) = (Succ m' + (n + p)))
  --         cong suc_fn (plus_assoc m' n p) : Succ ((m' + n) + p) = Succ (m' + (n + p)))
-->
