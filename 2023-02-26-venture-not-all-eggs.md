# Venture Not All Eggs In One Basket: Decentralize Your Secrets

**Share &#x2260; publish.**
In the contemporary social networking world the verb to '**share**' means to
'publish and make known' to other parties. In contrast, in this blog to
'**share a secret**' (in the cryptographic sense; we certainly do not want to
publish secrets!) means distributing your secret among several parties in 
a clever way in order to add security,
by preventing a small subset of these parties to complot and reveal/steal
the secret. Even though 'secret sharing' was 
coined in 1979, long before any social networks, maybe to 
'partition/distribute/split/divide' a secret would be a better term than to
'share' a secret. Just keep in mind this meaning of 'sharing a secret' in this
blog.


## 1. Simplest Case: Split a Secret Into Two Parts

How to partition / split / divide / share a secret between two parties so as 
no single party can unilaterally recover the secret from their share?

This simple security hardening trick is very easy and useful to know
and use, but apparently it's so complex that no cloud giants currently 
provide a support for it. But you'll be easily able to roll your own.

Suppose you have a *Secret*, a long bit string, say a standard AES-256
symmetric encryption key, which is just a random bit string of length 256. 

You do not want to keep you *Secret* in just one place and want to share 
it between two locations.

Entrusting your *Secret* to just a single person (entity, KMS, HSM, ...)
is unwise, because this single person may lose it, get bribed, ... Can
we, partition/split/divide/share it between two persons/locations, 
so as any one losing his share does not compromise the whole *Secret*
and both parties are needed to recover the secret? Yes, in this simple
case the solution is very easy.

Generate *Part<sub>1</sub>* as a random bit string of the same length 
as *Secret*. Obviously, if *Part<sub>1</sub>* is lost alone, 
it does not reveal the content of your 
*Secret* (they are simply unrelated). Compute 
*Part<sub>2</sub> = Secret XOR Part<sub>1</sub>*,
where *XOR* is the binary bitwise exclusive *OR* operation (for bits *x*, *y*,
*x XOR y = 1* iff *x* and *y* are opposite). If *Part<sub>2</sub>* 
is lost, it also
reveals nothing about your *Secret*. Why? Because, by construction, each bit
of *Part<sub>2</sub>* is a bit of Secret randomly inverted (*XOR*ed by the
corresponding bit of *Part<sub>1</sub>*).

So, neither *Part<sub>1</sub>*, nor *Part<sub>2</sub>* 
alone lost reveal our *Secret*. But if we combine both: 
*Part<sub>1</sub> XOR Part<sub>2</sub> = Part<sub>1</sub> XOR 
(Secret XOR Part<sub>1</sub>) = Part<sub>1</sub>
XOR Part<sub>1</sub> XOR Secret = Secret*, because *XOR* is associative,
commutative, *X XOR X = 0*, and *X XOR 0 = X*.

You may ask a question: why using bitwise *XOR*, we may simply chop our
256-bit *Secret* in two 128-bit halves, the beginning, and the end. If
one of those get lost, it reveals 128 bits of your Secret and its
strength remains 128 bits, which is probably enough against brute-force attacks by
present-day supercomputers, but not enough against the emerging quantum
computers. Our *XOR* split scheme has 256-bit strength, enough against
any quantum computers.


## 2. How to Benefit From A Secret Split?

On the practical side, if you place *Part<sub>1</sub>* in AWS
and *Part<sub>2</sub>* in Azure,
none of the clouds would learn your original *Secret* encryption key,
but you may use *Secret = Part<sub>1</sub> XOR Part<sub>2</sub>* 
to encrypt your data in
either/both of the clouds. However, you should keep in mind the
following. You can import *Part<sub>1</sub>* and 
*Part<sub>2</sub>* into KMSs or HSMs, but you
**cannot** extract *Part<sub>1</sub>* and *Part<sub>2</sub>* 
back from KMSs or HMSs, because none of
them allow you to extract keys (*Part<sub>1</sub>* or 
*Part<sub>2</sub>*). It's supposed to be so for
your benefit: you can only use KMS or HMS APIs to encrypt or decrypt
messages with a key contained in KMS/HSM, but cannot extract the keys from
there! Therefore, the ideal form of keeping your *Part<sub>1</sub>* 
and *Part<sub>2</sub>* are
*secrets* and *secret management systems* like AWS Secrets manager or Azure
Key Vault. When you need your *Secret*, you fetch 
*Part<sub>1</sub>* from an AWS
Secret Manager, and *Part<sub>2</sub>* from 
Azure Key Vault (make sure that in a
vault you keep a secret rather than a key, the latter cannot leave the
vault).

It does not matter where you keep your encrypted data. It's useless
unless both secret parts are revealed.


**How is it relevant to you?**
Even though cloud cryptography is a powerful thing, it easily 
'breaks on demand'. If
required, cloud providers reveal you secrets. Here's what AWS says.


**Disclosure of customer content.**
AWS will not disclose customer content unless we're required to do so
to comply with the law or a binding order of a government body. If a
governmental body sends AWS a demand for your customer content, we
will attempt to redirect the governmental body to request that data
directly from you. If compelled to disclose your customer content to a
government body, we will give you reasonable notice of the demand to
allow the customer to seek a protective order or other appropriate
remedy unless AWS is legally prohibited from doing so.

So, if one of the two clouds you use discloses all you data, including
one part of your *Secret*, no one can make sense of your encrypted data. 
Both clouds 
should reveal *Part<sub>1</sub>* and *Part<sub>2<sub>* for your data to be 
decrypted. Therefore, your best bet is to use two
clouds from different jurisdictions, like AWS and Alibaba.


## 3. Shamir's *(m, n)*-Threshold Secret Sharing Scheme

Let *2 &#x2264; m &#x2264; n*. We want to partition / distribute / 
split / share a **secret** *a<sub>0</sub>* between *n* participants 
in such a way that at least *m*
participants' shares are needed in order to recover the secret (and
any smaller number of participants is **not enough** for recovering the
secret). This is called the *(m, n)*-threshold SSS (Secret Sharing
Scheme). Previously we've demonstrated the (2, 2)-threshold SSS based
on XOR. The SSS scheme described below belongs to Shamir (1979), the
same as S in RSA.

The scheme is very easy to understand. To distribute a secret 
*a<sub>0</sub>* the
*distributor* chooses *n* different numbers *x<sub>i</sub>* (all 
arithmetic is modulo a big prime number *p*), which are publicly
distributed to *n* participants. Then the distributor secretly
generates *m-1* random polynomial coefficients 
*a<sub>1</sub>, ... ,a<sub>m-1</sub>* and gives
to the *i*-th participant the value 
*y<sub>i</sub>=a(x<sub>i</sub>)* of the polynomial in point 
*x<sub>i</sub>*:

*a(x) = a<sub>0</sub> + a<sub>1</sub>x + a<sub>2</sub>x<sup>2</sup> + ... + 
a<sub>m-1</sub>x<sup>m-1</sup>*.

Thus, the i-th participant has a public *x<sub>i</sub>* and a secret 
share *y<sub>i</sub>*.

I hope you can now see where it all goes: any polynomial of degree *m-1* 
can be easily and uniquely reconstructed by the values 
*(x<sub>i</sub>, y<sub>i</sub>)* in *m* (no fewer!) points 
(reconstructing means recover coefficients *a<sub>i</sub>*, notably
the secret *a<sub>0</sub>*). To see how, suppose *X* and *Y* are vectors of
length m containing public and corresponding private values of *m*
participants who got together to recover the secret. We have *m*
linear equations *a(X<sub>i</sub>)=Y<sub>i</sub>* with respect 
to the unknown coefficients
*a<sub>0</sub>, a<sub>1</sub>,...,a<sub>m-1</sub>*, and 
*X<sub>i</sub>*, *Y<sub>i</sub>* are constants. This *m x m* 
linear system has a **unique** solution,
because its matrix is Vandermonde (every row is 
*(1, x<sub>i</sub>, x<sub>i</sub><sup>2</sup>,...,
x<sub>i</sub><sup>m-1</sup>)*) and its determinant is nonzero for
*different* *x<sub>i</sub>*'s.

There is an even simpler method of recovering the secret *a<sub>0</sub>* 
from *m* points *(x<sub>i</sub>, y<sub>i</sub>)*. 
The well known Lagrange interpolation formula *L(x; X, Y)* gives
an explicit polynomial of *x* going through points *(X, Y)*, taking
coefficients from *X*, *Y*. By substituting *x=0* into this polynomial
you recover the secret *a<sub>0</sub>*.

Now you can implement, e.g., the *(4, 4)*-thereshold SSS for distributing 
your secrets in AWS, Azure, GCP, and Alibaba. Note that *(3, 4)* may not be 
enough since US clouds can simultaneously trade your secrets.
