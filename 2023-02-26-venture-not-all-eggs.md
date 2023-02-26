# Venture Not All Eggs In One Basket: Decentralize Your Secrets

In the contemporary social networking world to '**share**' means to
publish and make known to other parties. In contrast, in this blog to
'**share a secret**' (in the cryptographic sense) means distributing your
secret to several parties in order to add security. Even though 
'secret sharing' was coined in 1979, long before any social networks, may be
'partition/distribute/split' a secret would be a better term than to
'share' a secret. Just keep in mind this sense of 'sharing' in this
blog.


## Shamir's Threshold Secret Sharing Scheme

Let `2<=m<=n`. We want to partition/distribute/split/share a secret
`a0` between `n` participants in such a way that at least `m`
participants' shares are needed in order to recover the secret (and
any smaller subset of participants is not enough for recovering the
secret). This is called the `(m, n)`-threshold SSS (Secret Sharing
Scheme). Previosly we've demostrated the `(2, 2)`-threshold SSS based
on XOR. The SSS scheme described below belongs to Shamir (1979), the
same as S in RSA.

The scheme is very easy to understand. To distribute a secret `a0` the
distributor chooses `n` different numbers `xi` (all numbers and
arithmetic are modulo big prime number `p`), which are publicly
distributed to `n` participants. Then the distributor secretly
generates `m-1` random polynomial coefficients `a1,...,am-1` and gives
each participant number `i` polynomial value `yi=a(xi)` in point `xi`,
where
```
a(x) = a~0~ + a<sub>1</sub>*x + a2*x^2 + ... + am-1*x^(m-1).
```

Now each participant `i` has a public `xi` and a secret `yi`.

I hope you can see where it goes: any degree `m-1` polynomial can be
easily and uniquely reconstructed by the values `(xi,yi)` in `m` (no
less!) points (reconstructing means recover coefficients `ai`, notably
the secret `a0`). To see how, suppose `X` and `Y` are vectors of
length `m` containing public and corresponding private values of `m`
participants who got together to recover the secret. We have `m`
linear equations `a(Xi)=Yi` with respect to unknown coefficients
`a0,a1,...,am-1`. This `m * m` system has a **unique** solution,
because it's matrix is Vandermonde and its determinant is nonzero for
different `xi`s.

There is an even simpler method of recovering the secret `a0` from `m`
points `(xi, yi)`. A Lagrange interpolation formula `L(x; X, Y)` gives
an explicit polynomial going through points `(X, Y)`, taking
coefficients from `X`, `Y`. Substituting `x=0` into this polynomial
you recover the secret `a0`.






not venture all his eggs in one basket." (from "Don Quixote")




