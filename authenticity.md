{% include breadcrumbs.html %}
# Authenticity

In my grand scheme of bundling up what I see as common themes from different areas
(see my notes on [Domain Separation](domain_separation.md) for another example),
here I'm going to be talking about several concepts all related to "authenticity."
In other words, "Is the message I got from who I think it's from?"

We're going to look at a bunch of different constructions and how they work or fail.
While there are purely symmetric solutions (AEADs and MACs), they are relatively straight forward,
so this page will be focused on the *asymmetric* constructions.
Throughout this, I'll be using "ideal" implementations of all the constructions unless I specify a concrete implementation.
So, all of the pieces (hash function, Diffie-Hellman style agreement, AEAD, signature, etc.) work perfectly according to their standard security definitions.

## Signature based solutions

### Unencrypted

Let's start with the simplest case, a simple plaintext message that Alice is sending to Bob.
"I think SalusaSecondus is amazing!"
(We'll call this `M`.)
How does Bob know it's from Alice?
He doesn't.
Not without some cryptographic proof that's it's from her.

So, let's fix this with a [signature](https://en.wikipedia.org/wiki/Digital_signature).

Now Alice has a keypair A<sub>s</sub> and A<sub>p</sub> (her private/secret and public keys respectively).
She can use this to generate a signature \\(sig=\texttt{Sign}(A_s, M)\\) and send that to B along with `M`.
Bob can then just check that \\(\\texttt{verify}(A_p, M, sig)\\) is `true` and know that Alice sent the message.

What are the problems with this?

1. The message isn't encrypted.
   This might not be a problem, but it's certainly a limitation.
2. The message can be forwarded.
   Even if the message is public, maybe Alice wanted to ensure that Carol didn't think the message was for her.
   This signature doesn't capture it.
   Obviously, Alice could change her message to something like "Bob, I think SalusaSecondus is amazing!" but it would be better to have a cryptographic fix.

So, let's encrypt the data.
(It doesn't matter if we are using symmetric or asymmetric encryption so let's assume it's a public key encryption scheme that can use the same keys as signing.)
Do we sign the plaintext or the encrypted data?
Both have problems.

### Encrypt then Sign

Let's say that Alice signs the ciphertext.
So, \\(ct=\texttt{Encrypt}(B_p, M)\\), \\(sig=\texttt{Sign}(A_s, ct)\\), and Alice sends both `ct` and `sig` to Bob.
While this message does have authenticity and confidentiality, it could be resigned by an attacker.

What if the message says something like, "You can trust me because I know the password `hunter2`."
In the good case, Bob receives the message, verifies the signature (from Alice) and then decrypts the message.
He now knows that Alice sent him a message with this secret password and thus he can trust her.

Now Mallory intercepts the traffic from Alice to Bob.
She simply discards `sig` and replaces it with \\(sig\prime=\texttt{Sign}(M_s, M)\\).
Then she forwards on `ct` and `sig'` to Bob.
Bob receives the message, verifies the signature (from Mallory) and then decrypts the message.
He now "knows" that *Mallory* sent him the message with this secret password and thus he can trust her.
This is clearly a bad thing.

The fundamental issue is that Mallory was able to replace the correct signature with her own
despite not knowing the data she was signing.

### Sign then Encrypt

Doing it the other way has problems too.

This time we're going to assume that the secret message is something like, "Will you marry me?"

Alice signs the message \\(sig=\texttt{Sign}(A_s, M)\\) and then encrypts the message \\(ct=\texttt{Encrypt}(B_p, M)\\).
Again, she sends both `sig` and `ct` to Bob to propose marriage.
(It doesn't matter here if the signature is inside the ciphertext or separate.)
Bob receives the message, decrypts the message (so he knows it is intended for him), and then verifies the signature (from Alice).
So he now knows that Alice wants to marry him!

But what if Bob is evil?

Bob can decrypt the message sent to him to recover `M` and then encrypt it to Carlos.
\\(ct\prime=\texttt{Encrypt}(C_p, M)\\)
Bob then sends `ct'` and `sig` to Carlos.
Carlos receives the message, decrypts the message (so he knows it is intended for him), and then verifies the signature (from Alice).
So now Carlos "knows" that Alice wants to marry him!

The fundamental problem is that authenticity does not capture the recipient.

### Signcryption

The solution is something called [signcryption](https://en.wikipedia.org/wiki/Signcryption).
I'm not an expert on it and have never actually used it.
But this has the best of all the above worlds.

1. Data is encrypted to a specific recipient
2. Data is signed by a specific sender
3. Neither of the two above attacks work

Importantly, signcryption also provides [non-repudiation](https://en.wikipedia.org/wiki/Non-repudiation).
(So do the flawed schemes above.)
This means that the resulting message and signature can be used to prove (to a third party) that the sender actually signed the message.
(The recipient might need to reveal their private key as part of the proof, but that's okay in this model.)