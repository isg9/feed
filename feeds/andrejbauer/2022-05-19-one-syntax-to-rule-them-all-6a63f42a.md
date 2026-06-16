---
title: One syntax to rule them all
url: https://math.andrej.com/2022/05/20/one-syntax-to-rule-them-all/
published: "2022-05-19T22:00:00Z"
feed: andrejbauer
guid: https://math.andrej.com/2022/05/20/one-syntax-to-rule-them-all
---

# One syntax to rule them all

I am at the [Syntax and Semantics of Type Theory](https://europroofnet.github.io/wg6-kickoff-stockholm/) workshop in Stockholm, a kickoff meeting for [WG6](https://europroofnet.github.io/wg6/) of the [EuroProofNet](https://europroofnet.github.io) COST network, where I am giving a talk “One syntax to rule them all” based on joint work with [Danel Ahman](https://danel.ahman.ee).

**Abstract:** The raw syntax of a type theory, or more generally of a formal system with binding constructs, involves not only free and bound variables, but also meta-variables, which feature in inference rules. Each notion of variable has an associated notion of substitution. A syntactic translation from one type theory to another brings in one more level of substitutions, this time mapping type-theoretic constructors to terms. Working with three levels of substitution, each depending on the previous one, is cumbersome and repetitive. One gets the feeling that there should be a better way to deal with syntax.

In this talk I will present a relative monad capturing higher-rank syntax which takes care of all notions of substitution and binding-preserving syntactic transformations in one fell swoop. The categorical structure of the monad corresponds precisely to the desirable syntactic properties of binding and substitution. Special cases of syntax, such as ordinary first-order variables, or second-order syntax with variables and meta-variables, are obtained easily by precomposition of the relative monad with a suitable inclusion of restricted variable contexts into the general ones. The meta-theoretic properties of syntax transfer along the inclusion.

The relative monad is sufficiently expressive to give a notion of intrinsic syntax for simply typed theories. It remains to be seen how one could refine the monad to account for intrinsic syntax of dependent type theories.

**Talk notes:** Here are the hand-written [talk notes](https://math.andrej.com/asset/data/one-syntax-to-rule-them-all.pdf), which cover more than I could say during the talk.

**Formalization:** I have the beginning of a formalization of the higher-rank syntax, but it hits a problem, see below. Can someone suggest a solution? (You can download [`Syntax.agda`](https://math.andrej.com/asset/data/Syntax.agda).)

```
{-
   An attempt at formalization of (raw) higher-rank syntax.

   We define a notion of syntax which allows for higher-rank binders,
   variables and substitutions. Ordinary notions of variables are
   special cases:

   * order 1: ordinary variables and substitutions, for example those of
     λ-calculus
   * order 2: meta-variables and their instantiations
   * order 3: symbols (term formers) in dependent type theory, such as
     Π, Σ, W, and syntactic transformations between theories

   The syntax is parameterized by a type Class of syntactic classes. For
   example, in dependent type theory there might be two syntactic
   classes, ty and tm, corresponding to type and term expressions.
-}

module Syntax (Class : Set) where

  {- Shapes can also be called “syntactic variable contexts”, as they assign to
     each variable its syntactic arity, but no typing information.

     An arity is a binding shape with a syntactic class. The shape specifies
     how many arguments the variable takes and how it binds the argument's variables.
     The class specifies the syntactic class of the variable, and therefore of the
     expression formed by it.

     We model shapes as binary trees so that it is easy to concatenate
     two of them. A more traditional approach models shapes as lists, in
     which case one has to append lists.
  -}

  infixl 6 _⊕_

  data Shape : Set where
    𝟘 : Shape -- the empty shape
    [_,_] : ∀ (γ : Shape) (cl : Class) → Shape -- the shape with precisely one variable
    _⊕_ : ∀ (γ : Shape) (δ : Shape) → Shape -- disjoint sum of shapes

  infix 5 [_,_]∈_

  {- The de Bruijn indices are binary numbers because shapes are binary
     trees. [ δ , cl ]∈ γ is the set of variable indices in γ whose arity
     is (δ, cl). -}

  data [_,_]∈_ : Shape → Class → Shape → Set where
    var-here : ∀ {θ} {cl} → [ θ , cl ]∈  [ θ , cl ]
    var-left :  ∀ {θ} {cl} {γ} {δ} → [ θ , cl ]∈ γ → [ θ , cl ]∈ γ ⊕ δ
    var-right : ∀ {θ} {cl} {γ} {δ} → [ θ , cl ]∈ δ → [ θ , cl ]∈ γ ⊕ δ

  {- Examples:

  postulate ty : Class -- type class
  postulate tm : Class -- term class

  ordinary-variable-arity : Class → Shape
  ordinary-variable-arity c = [ 𝟘 , c ]

  binary-type-metavariable-arity : Shape
  binary-type-metavariable-arity = [ [ 𝟘 , tm ] ⊕ [ 𝟘 , tm ] , ty ]

  Π-arity : Shape
  Π-arity = [ [ 𝟘 , ty ] ⊕ [ [ 𝟘 , tm ] , ty ] , ty ]

  -}

  {- Because everything is a variable, even symbols, there is a single
     expression constructor _`_ which forms and expression by applying
     the variable x to arguments ts. -}

  -- Expressions

  infix 9 _`_

  data Expr : Shape → Class → Set where
    _`_ : ∀ {γ} {δ} {cl} (x : [ δ , cl ]∈ γ) →
            (ts : ∀ {θ} {B} (y : [ θ , B ]∈ δ) → Expr (γ ⊕ θ) B) → Expr γ cl

  -- Renamings

  infix 5 _→ʳ_

  _→ʳ_ : Shape → Shape → Set
  γ →ʳ δ = ∀ {θ} {cl} (x : [ θ , cl ]∈ γ) → [ θ , cl ]∈ δ

  -- identity renaming

  𝟙ʳ : ∀ {γ} → γ →ʳ γ
  𝟙ʳ x = x

  -- composition of renamings

  infixl 7 _∘ʳ_

  _∘ʳ_ : ∀ {γ} {δ} {η} → (δ →ʳ η) → (γ →ʳ δ) → (γ →ʳ η)
  (r ∘ʳ s) x =  r (s x)

  -- renaming extension

  ⇑ʳ : ∀ {γ} {δ} {Θ} → (γ →ʳ δ) → (γ ⊕ Θ →ʳ δ ⊕ Θ)
  ⇑ʳ r (var-left x) =  var-left (r x)
  ⇑ʳ r (var-right y) = var-right y

  -- the action of a renaming on an expression

  infixr 6 [_]ʳ_

  [_]ʳ_ : ∀ {γ} {δ} {cl} (r : γ →ʳ δ) → Expr γ cl → Expr δ cl
  [ r ]ʳ (x ` ts) = r x ` λ { y → [ ⇑ʳ r ]ʳ ts y }

  -- substitution
  infix 5 _→ˢ_

  _→ˢ_ : Shape → Shape → Set
  γ →ˢ δ = ∀ {Θ} {cl} (x : [ Θ , cl ]∈ γ) → Expr (δ ⊕ Θ) cl

  -- side-remark: notice that the ts in the definition of Expr is just a substituition

  -- We now hit a problem when trying to define the identity substitution in a naive
  -- fashion. Agda rejects the definition, as it is not structurally recursive.
  -- {-# TERMINATING #-}
  𝟙ˢ : ∀ {γ} → γ →ˢ γ
  𝟙ˢ x = var-left x ` λ y →  [ ⇑ʳ var-right ]ʳ 𝟙ˢ y

  {- What is the best way to deal with the non-termination problem? I have tried:

     1. sized types: got mixed results, perhaps I don't know how to use them
     2. well-founded recursion: it gets messy and unpleasant to use
     3. reorganizing the above definitions, but non-structural recursion always sneeks in

     A solution which makes the identity substitition compute is highly preferred.

     The problem persists with other operations on substitutions, such as composition
     and the action of a substitution.
  -}

```
