---
title: What is a Matrix? A Miserable Pile of Coefficients!
url: /2023/09/29/what-is-a-matrix
published: "2023-09-29T00:00:00Z"
feed: mcyoung
guid: /2023/09/29/what-is-a-matrix
---

# What is a Matrix? A Miserable Pile of Coefficients!

Linear algebra is undoubtedly the most useful field in all of algebra. It finds applications in all kinds of science and engineering, like quantum mechanics, graphics programming, and machine learning. It is the “most well-behaved” algebraic theory, in that other abstract algebra topics often try to approximate linear algebra, when possible.

For many students, linear algebra means vectors and matrices and determinants, and complex formulas for computing them. Matrices, in particular, come equipped with a fairly complicated, and *a fortiori* convoluted, multiplication operation.

This is not the only way to teach linear algebra, of course. Matrices and their multiplication appear complicated, but actually are a natural and compact way to represent a particular type of *function*, i.e., a linear map (or linear transformation).

This article is a short introduction to viewing linear algebra from the perspective of abstract algebra, from which matrices arise as a computational tool, rather than an object of study in and of themselves. I do assume some degree of familiarity with the idea of a matrix.

## [Linear Spaces](\#linear-spaces)

Most linear algebra courses open with a description of vectors in Euclidean space: Rn. Vectors there are defined as tuples of real numbers that can be added, multiplied, and scaled. Two vectors can be combined into a number through the dot product. Vectors come equipped with a notion of magnitude and direction.

However, this highly geometric picture can be counterproductive, since it is hard to apply geometric intuition directly to higher dimensions. It also obscures how this connects to working over a different number system, like the complex numbers.

Instead, I’d like to open with the concept of a *linear space*, which is somewhat more abstract than a vector space[1](#fn:1).

First, we will need a notion of a “coefficient”, which is essentially something that you can do arithmetic with. We will draw coefficients from a designated *ground field* K. A field is a setting for doing arithmetic: a set of objects that can be added, subtracted, and multiplied, and divided in the “usual fashion” along with special 0 and 1 values. E.g. a+0=a, 1a=a, a(b+c)=ab+ac, and so on.

Not only are the real numbers R a field, but so are the complex numbers C, and the rational numbers Q. If we drop the “division” requirement, we can also include the integers Z, or polynomials with rational coefficients Q\[x\], for example.

Having chosen our coefficients K, a linear space V *over* K is another set of objects that can be added and subtracted (and including a special value 0)[2](#fn:2), along with a *scaling operation*, which takes a coefficient c∈K and one of our objects v∈V and produces a new cv∈V.

The important part of the scaling operation is that it’s compatible with addition: if we have a,b∈K and v,w∈V, we require that

a(v+w)=av+aw(a+b)v=av+bv​

[LaTeX](#code:1)

This is what makes a linear space “linear”: you can write equations that look like first-degree polynomials (e.g. ax+b), and which can be *manipulated* *like first-degree polynomials*.

These polynomials are called linear because their graph looks like a line. There’s no multiplication, so we can’t have x2, but we do have multiplication by a coefficient. This is what makes linear algebra is “linear”.

Some examples: n-tuples of elements drawn from any field are a linear space over that field, by componentwise addition and scalar multiplication; e.g., R3. Setting n=1 shows that every field is a linear space over itself.

Polynomials in one variable over some field, K\[x\], are also a linear space, since polynomials can be added together and scaled by a any value in K (since lone coefficients are degree zero polynomials). Real-valued functions also form a linear space over R in a similar way.

### [Linear Transformations](\#linear-transformations)

A linear map is a function f:V→W between two linear spaces V and W over K which “respects” the linear structure in a particular way. That is, for any c∈K and v,w∈V,

f(v+w)=f(v)+f(w)f(cv)=c⋅f(v)​

[LaTeX](#code:2)

We call this type of relationship (respecting addition and scaling) “linearity”. One way to think of this relationship is that f is kind of like a different kind of coefficient, in that it distributes over addition, which commutes with the “ordinary” coefficients from K. However, applying f produces a value from W rather than V.

Another way to think of it is that if we have a linear polynomial like p(x)=ax+b in x, then f(p(x))=p(f(x)). We say that f *commutes* with all linear polynomials.

The most obvious sort of linear map is scaling. Given any coefficient c∈K, it defines a “scaling map”:

μc​:V→Vv↦cv​

[LaTeX](#code:3)

It’s trivial to check this is a linear map, by plugging it into the above equations: it’s linear because scaling is distributive and commutative.

Linear maps are the essential thing we study in linear algebra, since they describe all the different kinds of relationships between linear spaces.

Some linear maps are complicated. For example, a function from R2→R2 that rotates the plane by some angle θ is linear, as are operations that stretch or shear the plane. However, they can’t “bend” or “fold” the plane: they are all fairly rigid motions. In the linear space Q\[x\] of rational polynomials, multiplication by *any* polynomial, such as x or x2−1, is a linear map. The notion of “linear map” depends heavily on the space we’re in.

Unfortunately, linear maps as they are quite opaque, and do not lend themselves well to calculation. However, we can build an explicit representation using a *linear basis*.

## [Linear Basis](\#linear-basis)

For any linear space, we can construct a relatively small of elements such that any element of the space can be expressed as some linear function of these elements.

Explicitly, for any V, we can construct a sequence[3](#fn:3)ei​ such that for any v∈V, we can find ci​∈K such that

v=i∑​ci​ei​.

[LaTeX](#code:4)

Such a set ei​ is called a *basis* if it is linearly independent: no one ei​ can be expressed as a linear function of the rest. The *dimension* of V, denoted dimV, is the number of elements in any choice of basis. This value does not depend on the choice of basis[4](#fn:4).

Constructing a basis for any V is easy: we can do this recursively. First, pick a random element e1​ of V, and define a new linear space V/e1​ where we have identified all elements that differ by a factor of e1​ as equal (i.e., if v−w=ce1​, we treat v and w as equal in V/e1​).

Then, a basis for V is a basis of V/e1​ with e1​ added. The construction of V/e1​ is essentially “collapsing” the dimension e1​ “points” in, giving us a new space where we’ve “deleted” all of the elements that have a nonzero e1​ component.

However, this only works when the dimension is finite; more complex methods must be used for infinite-dimensional spaces. For example, the polynomials Q\[x\] are an infinite-dimensional space, with basis elements {1,x,x2,x3,...}. In general, for any linear space V, it *is* always possible to arbitrarily choose a basis, although it may be infinite[5](#fn:5).

Bases are useful because they give us a concrete representation of any element of V. Given a fixed basis ei​, we can represent any w=∑i​ci​ei​ by the coefficients ci​ themselves. For a finite-dimensional V, this brings us back *column vectors*: (dimV)-tuples of coefficients from K that are added and scaled componentwise.

​c0​c1​⋮cn​​​given ei​:=​i∑​ci​ei​

[LaTeX](#code:5)

The ith basis element is represented as the vector whose entries are all 0 except for the ith one, which is 1. E.g.,

​10⋮0​​given ei​=​e1​,​01⋮0​​given ei​=​e2​,...

[LaTeX](#code:6)

It is important to recall that the choice of basis is *arbitrary*. From the mathematical perspective, any basis is just as good as any other, although some may be more computationally convenient.

Over R2, (1,0) and (0,1) are sometimes called the “standard basis”, but (1,2) and (3,−4) are also a basis for this space. One easy mistake to make, particularly when working over the tuple space Kn, is to confuse the actual elements of the linear space with the coefficient vectors that represent them. Working with abstract linear spaces eliminates this source of confusion.

### [Representing Linear Transformations](\#representing-linear-transformations)

Working with finite-dimensional linear spaces V and W, let’s choose bases ei​ and dj​ for them, and let’s consider a linear map f:V→W.

The powerful thing about bases is that we can more compactly express the information content of f. Given any v∈V, we can decompose it into a linear function of the basis (for some coefficients), so we can write

f(v)=f(i∑​ci​ei​)=i∑​f(ci​ei​)=i∑​ci​⋅f(ei​)

[LaTeX](#code:7)

In other words, to specify f, we *only* need to specify what it does to each of the dimV basis elements. But what’s more, because W also has a basis, we can write

f(ei​)=j∑​Aij​dj​

[LaTeX](#code:8)

Putting these two formulas together, we have an explicit closed form for f(v), given the coefficients Aij​ of f, and the coefficients ci​ of v:

f(v)=i,j∑​ci​Aij​dj​

[LaTeX](#code:9)

Alternatively, we can express v and f(v) as column vectors, and f as the A matrix with entires Aij​. The entries of the resulting column vector are given by the above explicit formula for f(v), fixing the value of j in each entry.

A​A0,0​A1,0​⋮A0,m​​A1,0​A1,1​⋮A1,m​​⋯⋯⋱⋯​An,0​An,1​⋮An,m​​​​​v​c0​c1​⋮cn​​​​​=Av​∑i​ci​Ai,0​∑i​ci​Ai,1​⋮∑i​ci​Ai,m​​​​​

[LaTeX](#code:10)

(Remember, this is all dependent on the choices of bases ei​ and dj​!)

Behold, we have derived the matrix-vector multiplication formula: the jth entry of the result is the dot product of the vector and the jth row of the matrix.

But it is crucial to keep in mind that we had to choose bases ei​ and dj​ to be entitled to write down a matrix for f. The values of the coefficients depend on the choice of basis.

If your linear space happens to be Rn, there is an “obvious” choice of basis, but not every linear space over R is Rn! Importantly, the actual linear algebra *does not* change depending on the basis[6](#fn:6).

## [Matrix Multiplication](\#matrix-multiplication)

So, where does matrix multiplication come from? An n×m[7](#fn:7) matrix A *represents* some linear map f:V→W, where dimV=n, dimW=m, and appropriate choices of basis (ei​, dj​) have been made.

Keeping in mind that linear maps are supreme over matrices, suppose we have a third linear space U, and a map g:U→V, and let ℓ=dimU. Choosing a basis hk​ for U, we can represent g as a matrix B of dimension ℓ×n.

Then, we’d like for the matrix product AB to be the same matrix we’d get from representing the composite map fg:U→W as a matrix, using the aforementioned choices of bases for U and W (the basis choice for V should “cancel out”).

Recall our formula for f(v) in terms of its matrix coefficients Aij​ and the coefficients of the input v, which we call ci​. We can produce a similar formula for g(u), giving it matrix coefficients Bki​, and coefficients bk​ for u. (I appologize for the number of indices and coefficients here.)

f(v)g(u)​=i,j∑​ci​Aij​dj​=k,i∑​bk​Bki​ei​​

[LaTeX](#code:11)

If we write f(g(u)), then ci​ is the coefficient ei​ is multiplied by; i.e., we fix i, and drop it from the summation: ci​=∑k​bk​Bki​.

Substituting that into the above formula, we now have something like the following.

f(g(u))f(g(u))​=i,j∑​k∑​bk​Bki​Aij​dj​=k,j∑​bk​(i∑​Aij​Bki​)dj​​(⋆)​

[LaTeX](#code:12)

In (⋆), we’ve rearranged things so that the sum in parenthesis is the (k,j)th matrix coefficient of the composite fg. Because we wanted AB to represent fg, it must be an ℓ×m matrix whose entries are

(AB)kj​=i∑​Aij​Bki​

[LaTeX](#code:13)

*This* is matrix multiplication. It arises naturally out of composition of linear maps. In this way, the matrix multiplication formula is not a definition, but a *theorem* of linear algebra!

> [theorem](#thm:1)
>
> Given an n×m matrix A and an ℓ×n matrix B, both with coefficients in K, then AB is an ℓ×m matrix with entires
>
> (AB)kj​=i∑​Aij​Bki​
>
> [LaTeX](#code:14)

If the matrix dimension is read as n→m instead of n×m, the shape requirements are more obvious: two matrices A and B can be multiplied together only when they represent a pair of maps V→W and U→V.

### [Other Consequences, and Conclusion](\#other-consequences-and-conclusion)

The identity matrix is an n×n matrix:

In​=​1​1​⋱​1​​

[LaTeX](#code:15)

We want it to be such that for any appropriately-sized matrices A and B, it has AIn​=A and In​B=B. Lifted up to linear maps, this means that In​ should represent the identity map V→V, when dimV=n. This map sends each basis element ei​ to itself, so the columns of In​ should be the basis vectors, in order:

​10⋮0​​​01⋮0​​⋯​00⋮1​​

[LaTeX](#code:16)

If we shuffle the columns, we’ll get a *permutation matrix*, which shuffles the coefficients of a column vector. For example, consider this matrix.

​010​100​001​​

[LaTeX](#code:17)

This is similar to the identity, but we’ve swapped the first two columns. Thus, it will swap the first two coefficients of any column vector.

Matrices may seem unintuitive when they’re introduced as a subject of study. Every student encountering matrices for the same time may ask “If they add componentwise, why don’t they multiply componentwise too?”

However, approaching matrices as a computational and representational tool shows that the convoluted-looking matrix multiplication formula is a direct consequence of linearity.

f(v+w)=f(v)+f(w)f(cv)=c⋅f(v)​

[LaTeX](#code:18)

---

1. In actual modern mathematics, the objects I describe are still called vector spaces, which I think generates unnecessary confusion in this case. “Linear space” is a bit more on the nose for what I’m going for. [↩︎](#fnref:1)

2. This type of structure (just the addition part) is also called an “abelian group”. [↩︎](#fnref:2)

3. Throughout i, j, and k are indices in some unspecified but ordered indexing set, usually {1,2,...,n}. I will not bother giving this index set a name. [↩︎](#fnref:3)

4. This is sometimes called the [*dimension theorem*](https://en.wikipedia.org/wiki/Dimension_theorem_for_vector_spaces), which is somewhat tedious to prove. [↩︎](#fnref:4)

5. An example of a messy infinite-dimensional basis is R considered as linear space over Q (in general, every field is a linear space over its subfields). The basis for this space essentially has to be “1, and all irrational numbers” except if we include e.g. e and π we can’t include e+21​π, which is a Q-linear combination of e and π.

On the other hand, C is two-dimensional over R, with basis 1,i.

Incidentally, this idea of “view a field K as a linear space over its subfield F” is such a useful concept that it is called the “degree of the field extension K/F”, and given the symbol \[K:F\].

This, \[R:Q\]=∞ and \[C:R\]=2. [↩︎](#fnref:5)

6. You may recall from linear algebra class that two matrices A and B of the same shape are *similar* if there are two appropriately-sized square matrices S and R such that SAR=B. These matrices S and R represent a *change of basis*, and indicate that the linear maps A,B:V→W these matrices come from do “the same thing” to elements of V.

Over an algebraically closed field like C (i.e. all polynomials have solutions), there is an even stronger way to capture the information content of a linear map via [*Jordan canonicalization*](https://en.wikipedia.org/wiki/Jordan_normal_form), which takes any square matrix A and produces an almost-diagonal square matrix that only depends on the eigenvalues of A, which is the same for similar matrices, and thus basis-independent. [↩︎](#fnref:6)

7. Here, as always, matrix dimensions are given in RC (row-column) order. You can think of this as being “input dimension” to “output dimension”. [↩︎](#fnref:7)
