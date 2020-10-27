# Crypto Gotchas!
![Creative Commons License: BY](https://i.creativecommons.org/l/by/4.0/88x31.png)
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/)

## What is this?

It has occurred to me that there are lots of counter-intuitive "gotchas" in cryptography.
While some of them are relatively well-known (such as "encryption does not imply integrity"),
there are many others which are less well-known and can trip up even relatively experienced people.
So, this is a brain-dump which I hope will help other people who review and design cryptographic systems.

## What this is not

This isn't an attempt to tell people how to do anything.
There are already lots of excellent guides out there including [Latacora's Cryptographic Right Answers](https://latacora.micro.blog/2018/04/03/cryptographic-right-answers.html) or libraries which do most of what you need for you such as the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/introduction.html).
In fact, this document isn't really ever going to try to tell you what to do, just what *not* to do.
This also means that the target audience for this isn't really a junior engineer (though I hope they'll still find it useful and will help them advance).
If you are a junior engineer, you really shouldn't be working at this level at all.
Instead, please look at the earlier links in the paragraph for better guidance about what you *should* be doing.
I also strongly endorse both [Dan Boneh's Online Cryptography Course](https://crypto.stanford.edu/~dabo/courses/OnlineCrypto/) and [The Cryptopals Crypto Challenges](http://www.cryptopals.com/) (both of which are free).
Instead, this is aimed at the more experienced person who already knows cryptography pretty well but hasn't memorized the ins and outs of every single algorithm and quirk in the world.

# The Gotchas

## The Basics
These are the standard things you should watch out for. Hopefully you've already learnt these all, but just in case, here they are.

* **READ THE STANDARDS**. Many standards documents contain explicit instructions on how to do things (such as construct nonces, check input, or what limits to apply). If you are going to use an algorithm, *read the standard defining it*.
* Encryption doesn't (necessarily) provide integrity/authenticity.
  * No asymmetric modes provide authenticity (anyone can encrypt).
  * Unless it _explicitly_ [Authenticated Encryption](https://en.wikipedia.org/wiki/Authenticated_encryption) (e.g., AES-GCM) it doesn't provide any integrity or authenticity.
* Never use a key for more than one thing (this rule is sometimes, very carefully, violated).
  * Keys may only be used for a single algorithm. Don't use the same key for both AES and HMAC (or CMAC, or anything else).
  * This also applies to different modes of the same algorithm. Don't use the same key for AES-GCM, AES-CBC, and AES-CTR. In the asymmetric world you shouldn't use the same key for signatures, key agreement, and encryption. (Yes, this rule is commonly violated with RSA keys and while it is *usually* safe has also created problems in the past.)
  * Even with identical algorithms and modes, you still shouldn't use the same key for different purposes. For example, if you have a bidirectional channel with another party and both directions are encrypted using AES-GCM, you probably should be using separate keys for each direction.
* Always use a [Cryptographically secure pseudorandom number generator](https://en.wikipedia.org/wiki/Cryptographically_secure_pseudorandom_number_generator).
  It doesn't matter if you aren't using it for anything which should have security implications. Just always use it.
  If you must use an insecure one (perhaps for performance reasons), name the variable something like `insecureRandom` to ensure you don't accidentally cross-wire something.
  For guidance in how to do this in many languages, please see Paragon Initiative Enterprises' [excellent blog post](https://paragonie.com/blog/2016/05/how-generate-secure-random-numbers-in-various-programming-languages) on this exact topic.
* Hashes never provide authenticity.
  Since hashes don't incorporate a secret an attacker can just compute the correct hash over tampered data and you'll never know.
  Don't try to incorporate a secret yourself either because it won't work. (Please see [Length Extension Attacks](https://en.wikipedia.org/wiki/Length_extension_attack) and [Flickr's problems with this](http://netifera.com/research/flickr_api_signature_forgery.pdf)).
  If you need something like this, you should use a construction which handles the secret properly for you (such as [HMAC](https://en.wikipedia.org/wiki/HMAC) or AES-GCM).
* HMAC keys should be the same length as the hash output.
  * While HMAC keys can safely be up to the length of the underlying block (512 bits for SHA-1 and SHA-256, and 1024 bits for SHA-384 and SHA-512) there is no real value in being larger than the output size
  * If an HMAC key is larger than the underlying block-size it is hashed before use. This means that `HMAC(H(K), m) = HMAC(K, m)` for *all* `K` and `m` provided that `K` is sufficiently large. This will violate your security expectations.
* Cryptographic keys can "wear out".
  The easiest solution for this is regular key rotation.
  If this looks like it will still be an issue for you, seek out a mode/library designed to avoid this (such as the [AWS Encryption SDK](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/introduction.html)) or find an expert. Working around this problem is beyond the current scope of this document.
  * Generally, only worry about symmetric keys.
  * With rare exceptions is this bound by the [Birthday Paradox](https://en.wikipedia.org/wiki/Birthday_problem) which means that using a key for more than 2<sup>b/2</sup> operations degrades your security. You should generally remain well below this limit. I personally recommend keeping it below about 2^48 operations.
  * For anything based on a block-cipher (such as almost any use of AES), this will be a limit on the number of *blocks* encrypted with a given key.
  * For MACs, this will be the number of tags generated.
  * Some modes (such as AES-GCM) will have additional limits on usage. Check them out carefully.
 * Most ciphers are not "committing".
   This means that a single ciphertext can be decrypted (using different keys) to different *valid* plaintexts.
   This is true even for AEAD ciphers such as AES-GCM and chacha20/poly1305.
   AES-GCM not being committing [broke some security properties of Facebook  Messenger](https://eprint.iacr.org/2019/016).
 *  [Key Derivation Functions](https://en.wikipedia.org/wiki/Key_derivation_function) (KDFs) may not generate different outputs when only the length is varied.
    ([HKDF](https://en.wikipedia.org/wiki/HKDF), my favorite KDF, is an example of this.) Most KDFs take in both an Initial Keying Material (`IKM`) an a per-derived-key `Info` value and `Length`. (They may take in other parameters but these can be ignored safely for this gotcha.) If all inputs _except_ the `Length` are kept constant, the outputs may be related. For example, here are the outputs of `HKDF(IKM=0x0102030405060708, Salt="mysalt", Info="myinfo", Length=X)` for `Length=16` and then `Length=32`:

        0x6adb5cbd648b0af649d1f507543df984
        0x6adb5cbd648b0af649d1f507543df98484ed986c43cfcec47056b1d49795d944
  * Don't assume that any encoding is canonical unless it is explicitly designed to be so. While there are the obvious cases which ignore whitespace (hex, base64, yaml, json, xml, etc.) many also ignore capitalization (hex, xml, etc.). Interestingly, [Base64](https://en.wikipedia.org/wiki/Base64) (even ignoring whitespace) isn't canonical either! Since the trailing padding (which is often optional) causing you to ignore bits, the ignored bits can be anything. For example, while `example` would normally be encoded as `ZXhhbXBsZQ==`, there are many other possible values for it including `ZXhhbXBsZR==`, `ZXhhbXBsZY==`, and `ZXhhbXBsZf==`.

## Nonces/IVs
[Nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce) (or [Initialization Vector (IV)](https://en.wikipedia.org/wiki/Initialization_vector)) are two different names for essentially the same thing.
While some people (myself included) _try_ to draw a distinction between them, the fact is that this is basically a lost cause and you cannot assume anything about a Nonce/IV based on the name alone.
While there is some work being done on "nonce reuse resistant" cryptography, you should still try to avoid ever reusing these values. Just to be safe.

* Nonces/IVs must **never** repeat (in a given context, usually relative to a key).
  If you are using a random nonce/IV, a good rule of thumb is no more than *2<sup>(n-32)/2</sup>* random values where *n* is the number of bits in the nonce. For example, when using AES-GCM with a 12-byte (96-bit) random nonce, your limit is *2<sup>32</sup>* random values.
* You can never go wrong with a cryptographically random nonce from a general-purpose source of cryptographic entropy.
  Any construction other than this might cause problems.
* Many nonces have non-obvious requirements.  **Read the algorithm/protocol specific documentation** if you are doing anything other than an independent random value
  * Related or shorter than expected nonces in (EC)DSA are as bad as repeated, see [how researchers managed to extract blockchain private keys by exploiting biased signature nonces](https://eprint.iacr.org/2019/023).
  * Predictable nonces in CBC gave rise to the [BEAST Attack](https://en.wikipedia.org/wiki/Transport_Layer_Security#BEAST_attack).
  * Generating a nonce by using the same cryptographic key used elsewhere has caused problems (TODO: Add reference).

## Algorithm/Key Selection for Decryption/Verification
This section isn't about selecting the proper algorithm or key for your design (see the introduction).
It's about how to select the correct parameters for _this specific use_ of your design when decrypting or verifying data.
Now, I know that there is a major push in the cryptographic community to eliminate as much of this negotiation as possible, and I sympathize with it, but it isn't always possible.
If you need to decrypt/verify data from some time in the past, or rotate keys, or be _able_ to move off of a deprecated algorithm, you need some way to signal this.
Of course, there are lots of gotchas here (which is the main reason the community is trying to eliminate this in the first place).

* Don't let your adversary select a completely arbitrary key.
  If an adversary can get you to use an arbitrary key, then it could be one they control and that you'd never expect to use.
  This may sound ridiculous, but many asymmetric signatures come with the public key for verification alongside of them.
  Instead, all keys must be extremely carefully checked to ensure they are trusted before you use them.
  Two ways accomplishing this are a PKI (powerful, hard to get right, only works for asymmetric keys) or giving the keys a unique identifier and using that.
  Of course, even with the unique identifier you need to be very careful and might need to whitelist which ones you accept because there may be keys with valid identifiers which shouldn't be used for this use case.
  (An example of this would be any of the massive cloud/corporate KMS systems where _every_ key has a unique identifier.)
* Don't let your adversary select a completely arbitrary algorithm.
  If an adversary can get you to use an arbitrary algorithm, then they can select an insecure or completely broken one.
  (One example of this is [Downgrade Attacks](https://en.wikipedia.org/wiki/Downgrade_attack).)
  At most this can be from a pre-approved white-list.
  Ensure this algorithm is appropriate for the key! (I explicitly recommend people **not** use JWT and this type of [historical issue](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/) is part of why.)
  
### How to do this
This is one of the very few times I'll provide advice on how to do something as opposed to simply saying what not to do.
This is because it is really important to support key rotation and be able to change algorithms.
So, if you are encrypting data and will need to decrypt it later I recommend the following (simple) solution.

1. Prepend your ciphertext with a version number (4-byte integer?)
2. Each version number corresponds to the immutable set of keys needed to decrypt that message (usually just one, but if you have multiple keys for confidentiality/encryption, this can signal both) and the exact algorithms used.
3. Whitelist exactly which versions you'll accept and prune this list whenever you can.

This will let you rotate your keys (by incrementing the version) and even move to new algorithms if needed (by incrementing the version) while avoiding all of the normal complexity around protocol negotiation.
It still isn't fool-proof but is a reasonable design which works for many _simple_ cases.
If this isn't sufficient for your design, please seek out experts to talk to.

## AES-GCM/GMAC

[AES-GCM](https://en.wikipedia.org/wiki/Galois/Counter_Mode) is a really popular mode and one of the better ones for people to select. However, it doesn't always act intuitively.

* Reusing a (nonce/IV, key) combination not only destroys confidentiality for the impacted ciphertexts but results in loss of integrity/authenticity for *all* ciphertexts protected by the key. (TODO: Add reference)
  Thus, as per "key wear out" and "nonces" above, for 96 bit nonces (the recommended case) you shouldn't use generate more than 2<sup>32</sup> ciphertexts with a given key and random nonces.
* While nonces can be re-used across different keys, this is a more advanced design and should only be done cautiously.
  Note that a nonce is *still required* even if it doesn't vary. I strongly recommend 96 bits of zeros.
  Nonces of zero-length are not compliant with the specification and *extremely insecure and must not be used.*
* As AES-GCM uses an underlying counter mode you cannot encrypt more than 2<sup>63</sup>-32 bytes at once as it will cause the counter to overflow.
* AAD is also restricted in length to 2<sup>61</sup>-1 bytes due to internal encodings of the data
* If you use different length tags with the same key, you lower the security of *all* tags produced by that key, not just the short ones. (TODO: Add reference)
* The tags produced can't be treated as "random" values (e.g., like the outputs of a random function or a hash function). Any of the properties you expect (collision resistance, non-invertibility, etc.) may not be there. The only property you can assume they have is that specifically promised by the definition of a [MAC](https://en.wikipedia.org/wiki/Message_authentication_code). 
    * As an example, it is trivial for someone who *knows the key* to craft a message with any arbitrary tag.
    * This implies that it is trivial for someone who *knows the key* to craft multiple messages with the *same* tag
    * Contrast this with tags generated by HMAC which do generally act as people expect (in that they are collision resistant and act like output from a [Random Oracle](https://en.wikipedia.org/wiki/Random_oracle))
* AES-GCM is not committing. (See the discussion under "The Basics" earlier).
* *Do not use the plaintext before you've validated the tag!* This is called "releasing unverified plaintext" and is very bad.
  Some implementations (I'm looking at you OpenSSL and BouncyCastle) release the plaintext before the tag has been verified (e.g., before the `doFinal` call). While this is great from a performance perspective, the plaintext must be considered *completely untrusted* until the tag is verified. This means you mustn't do anything with it which you cannot fully roll-back. In fact, it's better to just ignore it altogether. If you are decrypting the data and handing it on to some other component, you should probably just buffer up all of the plaintext until you know it's valid.
* Not knowing the AAD does *not* stop someone with the key from just decrypting the data.
  AES-GCM is just AES-CTR (an unauthenticated mode) plus GMAC. This means that an attacker who has the key can just decrypt the ciphertext in AES-CTR mode and ignore the extra GMAC tag. That or they can use a library like OpenSSL which will release unverified plaintext (see prior point) and get the data that way.

## Signatures
[Digital Signatures](https://en.wikipedia.org/wiki/Digital_signature) are generally safe to use, but many people assume they have properties that they do not. At the core all they mean is that *without the private key, an attacker cannot find a signature over an arbitrary value for which they don't already know a signature*.

* Many signatures are malleable. This means that given a *valid* signature, an attacker can often find other valid signatures over the same message.
  * (EC)DSA have two different encodings: [ASN.1](https://en.wikipedia.org/wiki/Abstract_Syntax_Notation_One) and "raw" (my name). This means that an attacker can convert a valid signature in one form to the other. 
  * While the ASN.1 encoding is supposed to be [DER](https://en.wikipedia.org/wiki/X.690#DER_encoding) encoded, many libraries accept any (semi-)valid [BER](https://en.wikipedia.org/wiki/X.690#BER_encoding) encoding. This means that an attacker can often use the flexibility of BER to craft an essentially infinite number of valid signatures (for the same message) once they know a single one.
  * ECDSA is mathematically malleable as well. It consists of two values `(r, s)` and given one signature it is trivial to calculate a new `s'` (equal to the order minus the original `s`) which results in a new valid signature `(r, s')` over the same message.
* An attacker mustn't be allowed to select the actual value being validated in the signature. (The hashing step in all standard signatures defends against that as they can only select the hash pre-image, not the value of the hash.) If they could then they could trivially craft a valid signature for an arbitrary public key by (essentially) generating a random signature and seeing what message *would* be verified by that signature and then returning that message/signature pair.
* Just because a signature is valid for a given message doesn't mean it isn't valid for other messages.
* Just because a signature is valid for a given public key doesn't mean it isn't valid for other public keys.
  For that matter, given a signature and a message, an attacker can craft a public key which validates signature and message.
  This is called a [Duplicate Signature Key Selection attack](https://www.agwa.name/blog/post/duplicate_signature_key_selection_attack_in_lets_encrypt).
* Some signatures are randomized (e.g., the (EC)DSA family and RSA-PSS) and some are deterministic (e.g., RSA PKCS#1 v1 and v1.5). This means that assuming either property will trip you up.
* Signatures do not (necessarily) hide the message they sign. They'll commonly reveal the hash (or more) of the data being signed.
* Signatures do not hide the public key used to verify the message.
  * "Public Key Recovery" is a standardized part of ECDSA. (See [SEC2](https://secg.org/sec1-v2.pdf), Section 4.1.6-4.1.7.)
  * This is slightly more challenging with RSA but is still doable as [described here](https://crypto.stackexchange.com/a/26190) and [implemented here](https://gist.github.com/divergentdave/40ac9c7224b382166c905e76595bcf73).

For more information about some of these gotchas, please see [How Not to Use ECDSA](https://yondon.blog/2019/01/01/how-not-to-use-ecdsa/). I also recommend skimming through [Seems Legit: Automated Analysis of Subtle Attacks on Protocols that Use Signatures](https://eprint.iacr.org/2019/779) (there is some heavy formal verification in there, but also some readable writeups of weird signature properties).

## Side-Channels
To [quote SwiftOnSecurity](https://twitter.com/SwiftOnSecurity/status/832055497251487744), "Cryptography is nightmare magic math that cares what kind of pen you use."
Nowhere is this more true than in the area of [side-channels](https://en.wikipedia.org/wiki/Side-channel_attack).
It's not enough get the right answer and perform the correct calculation, you *must do it in the correct way*.
I truly believe that writing good side-channel-free implementations is a specialty unto itself within the larger (highly specialized) field of cryptographic development and I am definitely *not* an expert.
So, please take all of this advice as not only coming from that particular perspective but also aimed at other non-experts (like myself).

* _Any_ behavior which varies based on a secret value can be (potentially) exploited by an attacker.
  This includes time, memory access, power draw, bits flowing down a wire, and *literally anything else*.
* This means that any branches, memory access, or any logic beyond very simple math based on a secret value is a problem (and even simple math can be dangerous).
* The most common case of this for standard developers is comparing a secret value (such as a MAC tag) against another value.
  This must be done with a timing-safe equals check.
  Many languages and frameworks already have one. If you must write one yourself, please seek out an expert.
  For that matter, in cryptographic code I recommend using a timing-safe equality check for all array comparisons.
  Why? Not because it is necessary most of the time but because it both means you won't accidentally omit it (when you need it) and you don't need to waste your time continually re-reviewing every unsafe check to determine if it is actually safe *in this exact case and nothing has changed*.
  Save yourself the stress and just always use the constant-time implementation.
  (ToDo: Create/find a single page listing the best patterns for as many languages/frameworks as possible.)
* If you find that you need to worry about side-channel protection other than constant time equality checks, seek out an expert or refactor your design to make it a non-issue.
  Generally, the best strategy is to make this someone else's problem by depending on a library you trust.
  For example, you might depend on OpenSSL or Microsoft's CNG for cryptographic implementations and simply state "We trust their implementations to be side-channel free."
  Now, they won't be. Not perfectly at least.
  But they are likely to be far better than anything you can write and more likely to be reviewed and patched as problems are found.

## X.509 Certificates

Now, X.509 Certificates aren't cryptography any more than a car is an engine. But just as people who are experts with engines will spend a lot of time worrying about and fixing cars, so too will cryptographers (unfortunately) need to deal with X.509 Certificates. Here are just a *few* of many gotchas related to these horrors. As always, if you actually need to work with them, you should read the specification ([RFC 5280](https://tools.ietf.org/html/rfc5280) and many others).

* X.509 Certificates are *supposed* to be [DER](https://en.wikipedia.org/wiki/X.690#DER_encoding) encoded [ASN.1](https://en.wikipedia.org/wiki/Abstract_Syntax_Notation_One), but most systems will happily accept any (semi-)valid [BER](https://en.wikipedia.org/wiki/X.690#BER_encoding) encoding. This means these technically invalid certificates will almost always work, except with they don't.
  (ASN.1 is a nightmare in itself. Truthfully, I have a soft spot in my heart for it because I think that it fills a really useful function. But I also enjoy Perl and C++, so perhaps my taste is questionable. Three of the best resources for dealing with ASN.1 are [A Layman's Guide to a Subset of ASN.1, BER, and DER](http://luca.ntop.org/Teaching/Appunti/asn1.html), [A Warm Welcome to ASN.1 and DER](https://letsencrypt.org/docs/a-warm-welcome-to-asn1-and-der/), and the [ASN.1 JavaScript decoder](https://lapo.it/asn1js/). These saved me more than once.)
* There is nothing special about a root certificate. All that makes a certificate a "root" certificate is that its key is saved some place safe and tagged as "a root of trust."
  * Notice that I say "key" here. Commonly the key is the real the root of trust and the "root certificate" is just convenient way to package the key.
  * Roots don't need to be self-signed
  * Roots will often be signed by other roots (usually for migration reasons)
  * Signatures, experiations, certificate limitations, etc. on roots are commonly ignored. (Remember what I said about the "key" being the real root?)
  * Root stores often have custom logic and restrictions around how specific roots can be used, except when they don't and just trust the world.
  * Not all "Root CAs" are publicly trusted. You may be most familiar with those in your browser and in the [CA/B Forum](https://en.wikipedia.org/wiki/CA/Browser_Forum), but there are many others out there.
  * While some (publicly-trusted) roots are good and well managed, others just pay a bunch of auditors money to claim they can be trusted. From the outside, it can be hard to know which is which.
* Certificate path building is complicated, never build it yourself.
  * Trust an existing library or your platform. They may get it wrong, but you'll usually do even worse than they do.
  * There can be more than one valid path from a leaf certificate to a root. (Sometimes to the same root, sometimes to different roots.)
  * Not all certificate constraints are always enforced properly if at all (such as name constraints or policies)
* More than one certificate can have the same public key.
  * This is common when a CA needs to be renewed but they want to keep everything signed by it valid.
  * It does happen in other cases too
* Certificate validation (is this the correct certificate and well formed?) is complicated, never build it yourself.
  * The "Common Name" (CN) on server certificates really should be ignored now and only the [Subject Alternative Names (SANs)](https://tools.ietf.org/html/rfc5280#section-4.2.1.6) respected. Still, when something goes wrong, it's still often to blame.
  * Wild cards only apply to the single left-most level. So `*.example.com` matches `foo.example.com` and `bar.example.com`, but not `foo.bar.example.com`. For that you need  `*.bar.example.com`. This means that both `*.*.example.com` and `foo.*.example.com` are invalid.
  * [Top-Level Domains (TLDs)](https://en.wikipedia.org/wiki/Top-level_domain) are weird and need to be handled specially. (For example, the following certificates are all invalid: `*.com`, `*.uk`, and `*.ac.uk`.)
  * Let's not even talk about [Internationalized Domain Names](https://en.wikipedia.org/wiki/Internationalized_domain_name)
  * If you aren't dealing with HTTPS, you can often throw everything you know about name/host validation out the window, because it not longer applies.
  * Key-usages (and other extensions) can be important here and they are an entire area unto themselves. Consult detailed specification or leave this to an expert. (Note, this caution applies equally to path building.)
* Key and Signature types can (and often do) differ within the same chain. So an ECDSA certificate might be issued by an RSA (intermediate) CA which is then issued by a DSA CA. (Though if you find a DSA CA, you should probably run screaming to something slightly newer.)


# Contributions and Licensing

I'm always interested in receiving feedback. [Issues](https://github.com/SalusaSecondus/CryptoGotchas/issues) or [pull requests](https://github.com/SalusaSecondus/CryptoGotchas/pulls) are probably best, but any (reasonable) way to reach out will work.

Is there a mistake? Tell me!

Am I missing something important? Please let me know!

Can you help flesh out my references or otherwise improve this? I want to hear from you!

Several people have already shared pre-publication papers or other non-public materials with me to help me in drafting accurate and helpful _public_ documentation. If you have something you want to share in confidence with me but don't want me to share it on, please reach out to me so we can chat.

The license I've attached to this document is somewhat of a placeholder and exists solely to have something well defined attached. If it causes problems for you for _any reason at all_ please let me know and I'll work with you to come to some mutually agreeable solution

![Creative Commons License: BY](https://i.creativecommons.org/l/by/4.0/88x31.png)
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/)

# Acknowledgements
A special thank you to the following people and groups who have helped me with this.

* The MVP Slack
