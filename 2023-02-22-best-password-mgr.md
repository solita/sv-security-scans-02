# Best Password Managers Do Not Know Nor Send Away Your Passwords

## Introduction

1. Is it possible? YES!

2. Does it make password leaking impossible? YES!

3. So, finally I can use the same (strong) password 
for all websites? YES.

4. Is it difficult to understand? NO, just basic 
knowledge about hash functions, symmetric and asymmetric
cryptography is enough. No maths. No oblivious pseudorandom 
functions, zero-knowledge proofs of possession, ...
5. TL;DR: start by section 2.


## 1. How Traditional Password Authentication Works (Reminder)

1. During registration, you send your `username` and password `pwd` to
   a website, the password is being hashed with a salt `h(pwd, salt)`,
   and a triple
   ```
   username | salt | h(pwd, salt)
   ```
   is being saved on the website, while `pwd` is **erased** and 
   never stored.
2. During login, you send your `username` and `pwd`, the website 
   computes the salted hash and if it happens to be the same as
   `h(pwd, salt)` kept in the triple above, you are successfully
   authenticated. You prove that you know `pwd`, since
   it produces the same hash as expected. 
   Hash verification is based on the 
   **collision-free** property
   of good hashing: it's impossible to produce two different 
   passwords with the same hash:
   ```
   h(pwd1, salt) != h(pwd2, salt) for pwd1 != pwd2
   ```
3. Why this authentication is bad? Because (even though the
   website does not store your password) you need to send your 
   password to the website on each login (hopefully with HTTPS)
   for hash verification/authentication.
   This exposes your password, which may be leaked somewhere
   on the website side. Therefore all these well-known 
   recommendations about not reusing the same password
   on different websites.

4. The Web is full of horror stories about never ending passwords leaks.


## 2. No More Sending Away Passwords!

1. During registration, you take your `username`, password `pwd`
   and compute several things:
   - password hash `k=h(pwd)`, which will be used as a 
     **symmetric key**,
   - generate an **asymmetric key pair** `pub, pri`,
   - encrypt `enc(pri, k)` the private key with the symmetric
     key `k=h(pwd)` above.
   
   The following triple is saved on the website (only the
   middle element is cryptotext):

   ```
   username | enc(pri, k) | pub
   ```

2. During login, you send your `username` and get back your
   encrypted private key `enc(pri, k)`. Since you know your 
   password `pwd`, you can compute your symmetric key
   `k=h(pwd)` and decrypt your asymmetric private key
   `pri=enc(enc(pri, k), k)` (in symmetric cryptography you encrypt 
   and decrypt with the same key `k`).

   Now you have your asymmetric private key `pri`, the website
   has the corresponding public key `pub`, and you may start 
   secure communication.

3. Note that:
   - your password is never sent away from your computer;
   - it is only used on your computer for calculating `h(pwd)`
     and erased thereafter;
   - the difficult-to-remember secret `pri` is kept in encrypted 
     form `enc(pri, k)` on the website; you decrypt it 
     using an easy to remember `pwd` and key `k=h(pwd)`;
   - of course, you can start by registering `pub` on the
     website, keep `pri` with you and use this key pair 
     for all communication with the server, but remembering `pri`
     is difficult, especially if you use many devices (copying 
     needed, which increases possibilities of losing `pri`); 
     in the scheme described you just need to remember `pwd`.

## Conclusions

1. So, in a better world you do not need to send away
   your passwords for authentication.

2. Of course, all the above is oversimplified for the wide audience
to understand the basic principles and get convinced that a more 
secure password management is needed and possible.

3. Of course, just password managers cannot implement the 
   authentication schemes similar to the above. Client-server
   protocols need to be upgraded. We just wanted to stress
   the idea that servers do not need to know your passwords
   for authentication.

4. Once it is done, you can finally start reusing your single 
   (strong) password on many websites, since they cannot leak it.


## Appendix 1: Passkeys

### Q: What is a passkey?

### A: Cryptographically, it's just a store for asymmetric private keys locked by a master key.

1. It is locked by something you know (password, pin, swipe 
   pattern) or/and biometrically (fingerprint, iris, voice).

2. Otherwise, it just contains a list of the websites with 
   associated private keys you created at registration with
   each website:
   ```
   web-url_1, pri_1
   web-url_2, pri_2
   web-url_3, pri_3
   ...
   ```
   The corresponding public keys `pub_i` are kept by the 
   websites.

3. No surprise: authentication happens using public key 
   cryptography, and, obviously, private keys are **never**
   exposed/leaked to websites.
4. So, passkeys are just convenient means to store big sets 
   of impossible-to-remember private keys, protected.
5. No rocket science.

By contrast, what we described in section 2 is how you **can 
avoid** remembering private keys, storing them encrypted 
on the websites, and easily decrypting with your hashed 
password.
