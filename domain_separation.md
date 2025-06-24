# Domain Separation

If you ask cryptographers whether domain separation matters, they'll almost always agree. There have even been several security problems with protocols because it's been missing. And in the world of formal security proofs (or formal verification) researches absolutely pay attenion to it.

The problem is when you actually ask anyone to define it.

Even assembling a *good* list of incidents based on it is hard.
This is because it means different things to different people and actually covers an extremely wide range of different constructions, problems, and mitigations.
Lots of problems have occurred but they are generally thought of in more specific cases.

So, what is "Domain Separation?"
My *personal* definition is:
> Domain separation is ensuring that things can only be used for a specific intended purposes.

(What are "things?" Data, functions, keys, etc.)

By analogy it is very similar to strict (runtime) type-safety in a programming language.
It protects you against tons of theoretical problems by making them simply impossible.
(This is why byte-arrays are a horrible data type for most not super-low-level things. An AES-GCM key, ChaCha20/Poly1305 key, UTF-8 encoded string, and an MPEG4 video are all different things. Very few methods should accept more than one of them despite them all being just bytes.)

## Things which use domain separation

Though domain separation as a concept can be used with just abouut anything, there are a few primitives where you see (and need) it more commonly:

- [Random Oracles](https://en.wikipedia.org/wiki/Random_oracle) / [Hash Functions](https://en.wikipedia.org/wiki/Hash_function):
  These are a building block of many other cryptographic constructions and if you use the same hash function in different parts of the protocol, sometimes public parts of the protocol can reveal useful state about secret parts.
  This exact problem is described in [Separate Your Domains: NIST PQC KEMs, Oracle Cloning and Read-Only Indifferentiability](https://eprint.iacr.org/2020/241) where Bellare et. al. broke a bunch of the PQ-KEM submissions.
- Ciphertexts:
  If two different systems encrypt ciphertexts without domain separation, then an attacker can take a ciphertext from one system and hand it to the other (which may misinterpret it).
  I've personally witnessed a system which used the same key to encrypt trusted URLs in a redirect flow and customer data in a different flow.
  There was an attack where someone could encrypt data that looked like a URL and then cause systems to redirect to malicious sites.
- Signed data:
  Identical issues to encrypted data
- Authentication flows:
  [Reflection attacks](https://en.wikipedia.org/wiki/Reflection_attack) on authentication flows can be thought of as a case of domain separation where the domains are initiator / responder.

## Types of Domain Separation

### Different keys

If you are doing two different things, you should use two different keys.
(They might be derived from single source.)
This alone protrects you from many things.

Examples:
- TLS uses different keys for each direction of traffic.
This means that an attacker cannot simply reflect traffic back to a sender and make them accept it.
- ***???***

### Different AAD

Assuming that the encryptor and decryptors properly set the AAD to unambiguously contain the domain, this can work too.

- I have designed protocols which did use the same AES-GCM key in both directions but the first character of the AAD designated direction.
(example: "C" for sent by the client, and "S" for by the server.)
- A web-server might use a single AES-GCM to encrypt both cookies and other state sent to clients.
To ensure nothing can be swapped, the type of data (cookie/form-field/etc) and a name are included in the AAD.

Generally, different keys is a better solution than different AAD.
You (usually) approach the best of both words by using a KDF to derive domain-separated keys from a single root key.
This also leaves the AAD for more standard usage.

### Including the Domain as input

The "different aad" case is really just a special case of this one.
Like that one this requires that each domain be assigned an unambiguous unique string.
This is often harder than it seems, but there is a simple, common, case.
If you have a limited number of domains known from the start, you can just assign each a unique name *of the same length.*
It is critically important that all the names are of equal length to avoid canonicalization confusion.
(Technically, all that is required is that the domain names are a [prefix-free code](https://en.wikipedia.org/wiki/Prefix_code).
Having them all be of the same length is a trivial way to accomplish this.
Another way is to have a designated terminal element of all the names.
This is one of those places where consulting an expert is a good idea.)

One of the nice things about this pattern is that it can be applied to many different algorithms which would be otherwise challenging to separate:

- You can prepend the domain-separator to the data you're going to sign. SLH-DSA ([FIPS 205](https://csrc.nist.gov/pubs/fips/205/final), based on SPHINCS<sup>+</sup>) does this with a single byte value to indicate if the signed data was pre-hashed or not.
- You can prepend your plaintext with the domain separator.
- You can include the domain separator with data you're going to hash. HPKE and SHL-DSA do this throughout. ML-DSA includes the parameters when deriving keys to provide separation there as well.

### Length Separation

**CAUTION: This can be hard to get right**

If each domain only handles data of unique lengths such that you can unambiguously identify the domain just from the length of the data, then this can be sufficient separation.

### Data format separation

**CAUTION: This can be *extremely* hard to get right**

If there is absolutely no way to misinterpret the data from one domain as data from another domain, then you achieve separation.
(Length Separation is just a special case of this.)
Unfortunately, this is extremely hard to get right.
Many different data formats can coexist in a single file (sometimes called a [polyglot file](https://en.wikipedia.org/wiki/Polyglot_(computing))).
Even if you have a single data type with different content, different parsers may extra different fields.
(Consider an invalid JSON object with duplicate key names.
Some parsers may use the first value, some the last, others might concatenate.)

In some ways the [including as input](#including-the-domain-as-input) strategy is a special case of this one.

## Educational Examples

- [HKDF](https://www.rfc-editor.org/rfc/rfc9180.html) does an excellent job of domain separation.
  This is why both `LabeledExtract` and `LabeledExpand` exist. 
- Overall, [SPHINCS+](https://sphincs.org/) does a good job with domain separation.
  It's a [hash based signature scheme](https://en.wikipedia.org/wiki/Hash-based_cryptography) and so hashes lots of data in different contexts.
  Most of the time it includes an "address" in the data being hashed so that each use of the hash function is distinct.
  Unfortunately, the designers forgot to include this separation *once* (when they were using some data directly without a hash first) and that resulted in an [attack](https://eprint.iacr.org/2022/1061.pdf).
  While this is not described as a "domain separation" flaw, I consider it one because it matches the pattern we see elsewhere.
- [ML-KEM (FIPS 203 / Kyber)](https://csrc.nist.gov/pubs/fips/203/final) [added domain separation](https://www.federalregister.gov/documents/2024/08/14/2024-17956/announcing-issuance-of-federal-information-processing-standards-fips-fips-203-module-lattice-based) to its key-derivation algorithm right before before publication due to [public comment](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/5CT4NC_6zRI/m/QZ7jLXEiCAAJ)
  The concern was that someone might (incorrectly) use the same (seed) key with multiple algorithms (or parameter sets with ML-KEM) and get related keys out when they should be completely separate.
  By including domain separation, ML-KEM ensures that different algorithms will always derive unrelated cryptographic keys.