{% include breadcrumbs.html %}
# How to read a (crypto) research paper

![Creative Commons License: BY](https://i.creativecommons.org/l/by/4.0/88x31.png)
This work is licensed under a [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/)

## Introduction

One of the most valuable skills I learnt isn't specific to cryptography but is simply how to read a research paper.
Once you can do this your ability to teach yourself massively improves and you can keep up with the latest advances in whatever your sub-field is.
(There is a reason I list [IACR's ePrint Archive](https://eprint.iacr.org/) on my [Getting Started](GettingStarted.md) page.)
There have been many times at work when teams have come to me and said "What does this mean?" and handed me a dense paper to work through.
Being able to make sense of them has been a god-send.

Unfortunately reading a research paper can be very hard and is a different set of skills from reading most other things.
(For example, you don't start at the beginning and read to the end.)
This means that lots of people (like me) will try to read some and then quickly give up when it just doesn't make sense.
This page aims to save people the frustration that I've had to deal with.

**This guidance is _not_ aimed at those actually in academia.**
While it may be useful to you, please talk with your advisor or colleagues rather than following the random ramblings on this by someone with limited academic background.
Your needs and goals will be different and so how you approach papers will be different.

The guidance here is split into two sections.
The first should apply to (almost) all research papers, regardless of field.
The second is specific to cryptography.

## Reading a research paper

Before you read a research paper you need to know what _your_ goal is for reading it.

* Are you trying to learn more about the general field?
* Are you trying to figure out if the paper matters to you?
* Are you trying to learn the specific results from the paper?
* Are you trying to understand and evaluate the work that lead to the result?
* ...

Each goal requires a different focus and approach.
The strategies laid out below work reasonably well for each, but you'll need to customize them depending on your specific needs.
What do you take notes on? What do you skip?

### Generic strategies

A research paper generally consists of the following sections in order:

1. Title
2. Abstract
3. Introduction
4. (Optional) Background
5. Actual body of the paper
6. Conclusion
7. References/Citations

For those of us used to reading "normal" things we instinctually want to try this in order.
That doesn't work well.
Instead, it's best to generally read them in the order below.

As you read, be sure to take notes, save references you want to read later (or possibly first, if they have important prerequesites), etc.

Another thing to consider is that many papers have been presented at conferences.
Take a look at the conference's website or YouTube to see if you can find a recording of the author explaining it.
Sometimes that is easier.

#### 1. Title

There really isn't anything to say here.
You need to know what you're reading.

#### 2. Abstract

This will give you the shape of the paper and let you know if it's worth your time.
Sometimes it is worth bailing on a paper here if it is not relevant, too far beyond you, or just horribly written

### 3. Introduction

This is like the abstract but meatier.
Your goal here is to have a rough idea of what the paper is talking about and how it gets there.
There should definitely be some sort of problem statement and hopefully you are convinced that paper might actually address it.

Some specific things to look out for:

* Unrealistic claims (You might not be an expert, but you should already be able to sniff out some nonsense)
* Solutions which don't match the problem (It's the classic "Everything looks like a nail when all you have is a hammer" except that the hammer is whatever cryptographic tool the authors like. That and people will commonly handwave over real-world complications for an easier technical paper. Depending on the paper, this might be alright.)
* Are there weasel words in the claims? (Perhaps the results are really interesting, but only in very specific cases you don't care about or the authors want to make things more impressive than they really are.)

**This is your first really good stopping point.**
By now you should know if the paper is worth your time.
You might even have some other (more interesting or useful) references to chase down.

#### 4. Background

This section is optional.
If you already know the subject matter well, just skip it.

But, if you don't, this might be the most valuable part of the paper.
Here you'll learn *what* the paper is talking about and start getting a feel for the context.
If the paper is good, you'll really be able to build up your knowledge and collect a good list of references for further reading.
(Maybe you'll realize that this paper is beyond you right now but there are some others you can read instead.)

I still consider myself a beginner in many ways, so the "background" remains one of my favorite sections of a well-written paper.

#### 5. References

This section is optional.

If the background was useful, you should probably skim the references to see what to read next (or first).
If you background wasn't needed, then you know enough to know if the references are helpful.

#### 6. Conclusion

Time to jump to the end!

What are the conclusions of the paper? Do they make sense? (Are they interesting?)

In many cases the conclusion of the paper is its entire reason to exist.
This is why the paper was written and published.

**This is your next really good stopping point.**

Perhaps all you care about is the conclusions.

* "Algorithm FOO is broken!" Okay, let's not use it.
* "Algorithm BAR is useful to do BAZ." Well, I don't do BAZ, so I don't care.
* etc.

One of my common jobs is to look at research papers and figure out if we care about them.
Do we need to change our code or wrangle security teams to defend against new attacks?
Usually, once I've read the conclusion I know enough to triage the papers and figure out if they are worth our time and I might never need the details.

#### 7. Body

*Finally* we reach the body of the paper.
(Or maybe we didn't reach it because you stopped before you got here. That's okay too.)

The important thing to understand is that right now your goal _isn't to read the body_, it's just to skim it.
Get a feel for what they are saying and how they did their work.
Don't worry about the proofs or any detailed data-sets.
Right now, you just want to understand the overall argument and are simply *trusting* that the authors can support their arguments.
You're not trying to check their work.

**This is another good place to stop.**

In fact this is my most common stopping point.
I'm not good enough (yet) to understand, much less check, the detailed security proofs in the papers I read.
Most of the math for the asymmetric algorithms is far beyond me.
So, it isn't worth my time to fight through the details.

#### 8. Body (detailed read)

Now is when you re-read the body, including all detailed proofs and data with an eye to understand exactly what they did, how, and catch errors if there are any.
You know the shape of the paper and their arguments so you can see how everything fits together.
When a proof is presented, you can see how it will support the later pieces.

**Congratulations, you're done.**

### Tricks for reading cryptographic papers

Cryptography is hard and, like many sciences, has its own vocabulary.
Lots of concepts have detailed and extremely precise definitions, but, if you're using this guide, they rarely matter.
Instead, what you need is an _intuition_.
Most of the time all you need is a rough mental approximation and familiarity with some standard notation and terms to wade through many papers.

So, here following my incredibly informal (and probably inaccurate) mental model for a bunch of crypto things.

* Cryptographic Assumptions: *Most* of the time we don't claim that an algorithm is simply "secure" but rather that it is secure relative to some assumptions.
  * The most common assumption is that it is computationally (or semantically) secure relative to some security parameter. This simply means that you can brute force the algorithm in time roughly proportional to \\(2^\\lambda\\) (two to the power of the security parameter). Another assumption might be that the [discrete log is hard](https://en.wikipedia.org/wiki/Discrete_logarithm#Cryptography).
  * A few algorithms are [information theoretically secure](https://en.wikipedia.org/wiki/Information-theoretic_security) which means that they truly are "secure".  There is no way to brute-force or crack them regardless of the power of the attacker. While interesting, these algorithms are rarely useful in the real world. (Yes, I know that [Shamir's Secret Sharing](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing) is used in the real world, but most people will and should never touch it.)
  * Many (most?) papers use security reductions. If they proof a security reduction from Foo to Bar, then it means that (subject to usually polynomial approximation), Foo is as secure as Bar (and by implication any attack on Foo could be used to attack Bar as well). This is nice when we are really confident in the security of Bar. (It will also be used for theoretically perfect constructs. So, the proof might be relative to a Random Function with a reduction that means that when you substitue in a PRF, your construction will be as secure as that PRF is a good approximation of a real random function.)
* Model: A Model is a a way for cryptographers to assume things which may not be true but are often close enough to true (and useful for proofs). They can be our version of [spherical cows](https://en.wikipedia.org/wiki/Spherical_cow).
  * Random Oracle Model: We get to assume that we have access to [Random Oracles](https://en.wikipedia.org/wiki/Random_oracle) (ROs). Incredibly weird and critical to cryptographic proofs, but you can just think of it like a "perfect" cryptographic hash function (possibly with variable length output). As this is an imaginary construct anyway, there are an infinite number of different ROs which each map their inputs to ourputs differently. Generally, you don't care which you have as long as you have one. (This also means that papers might use more than one RO within the same paper for different purposes.) In security proofs both the challenger and adversary can query it and the adversary has the ability to pre-program its responses.
  * Common Reference String Model: Everyone gets to know some common string. This solves the problem of ensuring everyone can get access to some public (trusted) value.
  * Algebraic Group Model: Group elements output by an adversary can only have been derived by multiplying and inverting the adversary’s input group elements
* Random Function: It's like a Random Oracle (but don't ask me what the difference  is). Imagine you have a set of all possible functions from a set of inputs to a set of outputs. Then, you select one of them completely at random (uniform distribution).
* Random Permutation: Same as a random function, but the input and output sets are the same and there is a one-to-one mapping between them.
* PRF: A [Pseudorandom Function](https://en.wikipedia.org/wiki/Pseudorandom_function_family). Basically a fake random oracle. At a first mental approximation, don't worry about the differences. (They often take a key which lets them simulate different ROs.)
* PRP: A [Psuedorandom Permutation](https://en.wikipedia.org/wiki/Pseudorandom_permutation). Like a PRF, except the input and output spaces are identical and it defines a permutation. Think a "perfect" block-cipher. (Like PRFs, they often take keys.)
* \\(\\xleftarrow{\\$}\\) (A left pointing arrow with a dollar sign over it) means to randomly select an element from a set. Unless otherwise stated, all elements are equally likely to be selected. This is commonly used to select nonces or keys.
* Games:  A game is some set sequence of allowed actions between a challenger and an adversary. The majority of security definitions are given by describing a game and saying something is secure if no PPT adversary can win the game with more than negligible probability
  * An "Adversary" is some limited (usually normal probability polynomial time) algorithm which is trying to break an algorithm. (Probabilistic polynomial time just means it can runs relatively quickly and has access to random bits.)
  * A "Challenger" generally uses an algorithm (being proven to have certain properties) and random bits to pose some sort of challenge to an adversary in the context of a game. For example "Is this string the result of encrypting the data you gave me under a random key OR is it just completely random data?"
* The security parameter for a proof is usually notated as \\(\\lambda\\) and is that many `1`s. (So, AES-128's security parameter might be notated as \\(1^{128}\\) or a string of 128 ones.) Why? Because it is easier to say "polynomial in the size of the input" (which is normal for [big-O notation](https://en.wikipedia.org/wiki/Big_O_notation)) rather than "polynomial time in the security parameter" (which would be different and special for us).
* Negligibable Probability: Basically, as values (usually the security paramter) get sufficient large, the probability of something happen becomes some small you can (mostly) ignore it. A function \\(f(n)\\) is negligible if for any polynomial \\(p(n)\\), \\(\|f(n)\| \< 1/p(n)\\) for all sufficiently large \\(n\\). You can think of negligible functions as getting really small at the rate of an inverse exponential.
* Peer Review: Papers published at conferences or in journals have undergone peer review. You should think of this as a high-level check on the claims made by the paper and its quality of communication. Importantly, it does not mean that the results are correct, just that they are interesting and not obviously wrong. (Anyone can post papers on [ePrint](https://eprint.iacr.org/) so you cannot assume that they have undergone any sort of review.)
* Proofs: Many papers will include proof or proof-sketches of a particular theorem. The level of detail included varies substantially by author and by venue. Mistakes or omissions in notation are quite common, so don't be too surprised if a step seems to be missing and needs to be filled in by the reader.
* (Yes, I know I'm missing lots of things, please tell me what)

Finally, I've also heard that Boneh's and Shoup's [A Graduate Course in Applied Cryptography](https://toc.cryptobook.us/) is an *excellent* (free) reference and is a good stepping stone towards engaging with research papers. I haven't read it yet but it is now high on my personal list.

## Contributions and Licensing

Please see the [Contributions and Licensing section of the main document](index.md#contributions-and-licensing).
