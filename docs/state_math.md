# Scalar multiplication
---

Multiplying a state by a scalar modifies the coefficient appropriately:

```julia
julia> k = d" (1+3.4im)| 0 > "
Ket{KroneckerDelta,1,Complex{Float64}} with 1 state(s):
  1.0 + 3.4im | 0 ⟩

julia> k/2
Ket{KroneckerDelta,1,Complex{Float64}} with 1 state(s):
  0.5 + 1.7im | 0 ⟩
```

---
# Addition and Subtraction
---

States can be also be added and subtracted:

```julia
julia> d" | 0 > + | 0 > == 2| 0 > "
true

julia> d" | 0 > - | 0 > == 0| 0 > "
true

julia> d" 1/√3 * (| 0 > + | 1 > - | 2 >) "
Ket{KroneckerDelta,1,Float64} with 3 state(s):
  0.5773502691896258 | 0 ⟩
  -0.5773502691896258 | 2 ⟩
  0.5773502691896258 | 1 ⟩
```

One can conviently sum over an iterable of labels by using Julia's `sum` function in conjunction with the `ket`/`bra` function:

```julia
julia> 1/√5 * sum(ket, 1:5)
Ket{KroneckerDelta,1,Float64} with 5 state(s):
  0.4472135954999579 | 2 ⟩
  0.4472135954999579 | 3 ⟩
  0.4472135954999579 | 5 ⟩
  0.4472135954999579 | 4 ⟩
  0.4472135954999579 | 1 ⟩
```

---
# Normalization
---

In general, QuDirac objects do not automatically normalize themselves.

We can normalize a state in-place by using the `normalize!` function:

```julia
julia> k = sum(i -> d" float(i)| i > ", 1:3)
Ket{KroneckerDelta,1,Float64} with 3 state(s):
  3.0 | 3 ⟩
  2.0 | 2 ⟩
  1.0 | 1 ⟩

julia> normalize!(k)
Ket{KroneckerDelta,1,Float64} with 3 state(s):
  0.8017837257372732 | 3 ⟩
  0.5345224838248488 | 2 ⟩
  0.2672612419124244 | 1 ⟩

julia> norm(k)
1.0
```

The `normalize` function (without the trailing "!") is also provided, which normalizes a copy of the state instead of modifying the original.

---
# Getting a State's Dual
---

One can use the `ctranspose` function to construct the dual of a given state:

```julia
julia> k = d" (im)| 0 > "
Ket{KroneckerDelta,1,Complex{Int64}} with 1 state(s):
  0 + 1im | 0 ⟩

julia> k'
Bra{KroneckerDelta,1,Complex{Int64}} with 1 state(s):
  0 - 1im ⟨ 0 |
  
julia> k'' == k
true
```

For efficiency's sake, Bras are *views* onto their Kets, not copies. Thus, mutating a Bra will result in the mutation of the underlying Ket:

```julia
julia> k = d" 2.3| 1 > + 4.5| 2 > "
Ket{KroneckerDelta,1,Float64} with 2 state(s):
  4.5 | 2 ⟩
  2.3 | 1 ⟩

julia> b = k'
Bra{KroneckerDelta,1,Float64} with 2 state(s):
  4.5 ⟨ 2 |
  2.3 ⟨ 1 |

julia> normalize!(b) # in-place operation mutates b, which mutates k
Bra{KroneckerDelta,1,Float64} with 2 state(s):
  0.8904346821960807 ⟨ 2 |
  0.4551110597891079 ⟨ 1 |

julia> k
Ket{KroneckerDelta,1,Float64} with 2 state(s):
  0.8904346821960807 | 2 ⟩
  0.4551110597891079 | 1 ⟩
```

If you would like a copy of a state instead, you can explicitly construct one via the `copy` function:

```julia
julia> k = d" | 1 > + | 2 > ";

julia> b = copy(k)';

julia> scale!(3,b)
Bra{KroneckerDelta,1,Int64} with 2 state(s):
  3 ⟨ 2 |
  3 ⟨ 1 |

julia> k
Ket{KroneckerDelta,1,Int64} with 2 state(s):
  1 | 2 ⟩
  1 | 1 ⟩
```

---
# Tensor Product
---

One can take a tensor product of states simply by multiplying them:

```julia
julia> d" | 0 > * | 0 > "
Ket{KroneckerDelta,2,Int64} with 1 state(s):
  1 | 0,0 ⟩
```

As you might notice, a tensor product of Kets is itself a Ket, and the result
of the above is the same as if we input `ket(0,0)`. Taking the tensor product of 
more complicated states illustrates the tensor product's cartesian properties:

```julia
julia> d" normalize!(sum(i -> (i^2)| i >, 0:3) * sum(i -> (i/2)| i >, -3:3)) "
Ket{KroneckerDelta,2,Float64} with 18 state(s):
  -0.5154323951168185 | 3,-3 ⟩
  0.3436215967445456 | 3,2 ⟩
  -0.019090088708030313 | 1,-1 ⟩
  0.22908106449636376 | 2,3 ⟩
  -0.07636035483212125 | 2,-1 ⟩
  0.019090088708030313 | 1,1 ⟩
  ⁞
```

---
# Inner Product
---

Similarly to the tensor product, the inner product can be taken simply by multiplying Bras with Kets:

```julia
julia> bra(0) * ket(1)
0

julia> d" < 0 | 1 > " # same as above
0

julia> k = d" 1/√2 * ( | 0,0 > + | 1,1 > ) "; k'*k
0.9999999999999998

julia> d" < 0,0 | * k "
0.7071067811865475
```

---
# Acting a Bra on a specific Ket factor and vice versa
---

It is sometimes useful to take the inner product between a Bra and a *specific factor* of a Ket.
A math example might be:

```
| ψ ⟩ =   c₁ | 0, 1 ⟩ + c₂ | 1, 0 ⟩

⟨ 0₂ | ψ ⟩ =  c₁ ⟨ 0₂ | 0, 1 ⟩ + c₂ ⟨ 0₂ | 1, 0 ⟩

          =  c₁ ⟨ 0 | 1 ⟩| 0 ⟩ + c₂ ⟨ 0 | 0 ⟩| 1 ⟩
```

If these states are orthonormal, our final result is

```
⟨ 0₂ | ψ ⟩ = 0 | 0 ⟩ + c₂ | 1 ⟩ 
          
          = c₂ | 1 ⟩ 
```

QuDirac supports this operation through the use of the `act_on` function:

```julia
julia> ψ = d" normalize!( | 0,1 > + 2.0| 1,0 > ) "
Ket{KroneckerDelta,2,Float64} with 2 state(s):
  0.4472135954999579 | 0,1 ⟩
  0.8944271909999159 | 1,0 ⟩

julia> act_on(d" < 0 | ", ψ, 2) # ⟨ 0₂ | ψ ⟩
Ket{KroneckerDelta,1,Float64} with 2 state(s):
  0.8944271909999159 | 1 ⟩

julia> act_on(d" < 0 | ", ψ, 1) # ⟨ 0₁ | ψ ⟩
Ket{KroneckerDelta,1,Float64} with 2 state(s):
  0.4472135954999579 | 1 ⟩
```

This does, of course, work even when the Bra is a superposition of states:

```julia
julia> ϕ = d" 1/√2 * (< 0 | + < 1 |) "
Bra{KroneckerDelta,1,Float64} with 2 state(s):
  0.7071067811865475 ⟨ 0 |
  0.7071067811865475 ⟨ 1 |

julia> act_on(ϕ, ψ, 2)
Ket{KroneckerDelta,1,Float64} with 2 state(s):
  0.3162277660168379 | 0 ⟩
  0.6324555320336758 | 1 ⟩
```

Additionally, one can call `act_on(k::Ket, b::Bra, i)` to compute `⟨ b | kᵢ ⟩`:

```julia
julia> act_on(d" | 0 > ", ψ', 2)
Bra{KroneckerDelta,1,Float64} with 1 state(s):
  0.8944271909999159 ⟨ 1 |
```

As you can see, the above calculations assume an *orthonormal* inner product for the involved states. This behavior is indicated by the state's type (e.g. `KroneckerDelta` in `Ket{KroneckerDelta,1}`). QuDirac has support for other kinds of inner products as well. To learn more, see the [Working with Inner Products](inner_products.md) section.
