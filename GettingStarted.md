# So, you want to be a Cryptographer?*

![Creative Commons License: BY](https://i.creativecommons.org/l/by/4.0/88x31.png)
This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/)

\**That's not really a fair title, because this won't teach you how to be a cryptographer,
but one of my favorite games as a kid was [Quest for Glory: So You Want to be a Hero](https://en.wikipedia.org/wiki/Quest_for_Glory:_So_You_Want_to_Be_a_Hero)
and I couldn't pass up the opportunity to pay homage to it.
A far better title would be **"So, you want to start designing and writing cryptographic code and vaguely following what is happening in the broader cryptographic community?"**,
but that doesn't exactly roll off the tongue now, does it?*

## Introduction

For some reason you have decided that you need to learn more cryptography.
Maybe it is personal interest or maybe your job is shifting.
I really don't know.
However, a surprising number of people have asked me "How do I get started?" (or more commonly a variation of "How did you get started?"),
so rather than assemble my notes each time for each person, I'm writing these down _once_ in an easy to share and update format.

Similar to the main [Crypto Gotchas](https://github.com/SalusaSecondus/CryptoGotchas/blob/master/README.md), this is a personal project and definitely informal.
I have no "formal" qualifications other than that I've done work like this since about 2006 in some form or another (and started focusing on it in about 2012).

### Please don't

This isn't an effort at gatekeeping. Anyone can learn this and I really do think that more developers should know more about cryptography.
However, it is also a life goal of mine that developers *don't need* to know about cryptography (or *anything* from the main Crypto Gotcha's list),
that they need to is an indictment of the existing tools and libraries out there.

So why do I discourage this then? **This knowledge can be dangerous.**

Just like I generally discourage people from learning lockpicking (see [Toool](https://toool.nl/Toool) if you want to learn)
or amateur pyrotechnics (no useful links here, sorry),
it is really easy to learn just enough cryptography to get yourself in a lot of trouble.
If you don't know anything about lockpicking, you are unlikely to say "Hmm, I wonder if I can pick this random lock?"
If you know just a little, you might say "Aha, I recognize this lock and can pick it!"
Once you know more, you might say "I could pick, that lock, but if it breaks, I am trapped out of my apartment and the lock isn't technically owned by my anyway, and where I live even doing this is criminal...."
Similarly in cryptography, it is easy to learn just enough to be able to assemble some cryptography which *looks* good but is catestrophically broken.
(When I'm reviewing cryptographic code I love reading comments for why "this thing is safe to do even though it looks dangerous."
The comments are almost always wrong and highlight exactly how to break the system.
The problem is that the authors know just enough to make the arguments but not enough to see why it still isn't safe.)
See also [Schneier's Law](https://www.schneier.com/blog/archives/2011/04/schneiers_law.html) that "Anyone ... can create an algorithm that he himself can’t break."
Also, you will make mistakes and some of these mistakes will matter and hurt people.
This will be despite your best efforts.
Are you okay with that?

So, if this is something you actually want to get *good* at (rather than just learning out of interest) it is something you need to dedicate a lot of time to.
Are you sure that it is worth it? Are there better uses of your time?
Cryptography is rarely the most valuable skill for your projects and there are far more rewarding hobbies.
(I should be spending more time practicing my instruments or bicycling.)

However, with all that said, if you still want to learn this, I hope this helps.

## What this isn't

This is not a course and doesn't actually aim to teach you much of anything.
(I'm [working on a glossary](https://github.com/SalusaSecondus/CryptoGotchas/issues/15) which will help here, but who knows when that will happen.)
Instead this is meant to be a resource list and a jumping off point for you to learn from people who actually know what they are doing and can actually teach you.

If you read *everything* (or even most things) from this list you'll know more about cryptography than the vast majority of security specialists in the world.
However, you still won't be a cryptographer by any stretch of the imagination.
You'll probably be able to find and correct many cryptographic mistakes and maybe even write some simple *high level* cryptographic code,
but you should certainly not be implementing any primitives.
You definitely shouldn't start defining new designs or doing any work on your own.
(I don't care how good or who you are, cryptography is never a task to be done alone.)

## The List

### Courses

1. [Dan Boneh's Cryptography I](https://www.coursera.org/learn/crypto)
  This is an excellent undergraduate-level course in cryptography and everyone should complete this. It should probably be your first stop
1. [Introduction to Cryptography by Christof Paar](https://www.youtube.com/channel/UC1usFRN4LCMcfIV7UjHNuQg/videos)
  (I haven't personally verified this one.)
  I have heard good things about this series of lectures and suspect it to be similar in value to Boneh's course. So, I recommend it as well.

### Books

1. Look at [Crypto 101](https://www.crypto101.io/) by lvh. (Though I haven't personally verified it, I am very familiar with lvh's excellent work.)
1. Get [Serious Cryptography](https://nostarch.com/seriouscrypto) by Jean-Philippe Aumasson. This is one of the best books out there on applied cryptography.
1. (Just for fun) Read [The Code Book](https://www.amazon.com/dp/0385495323) by Singh. I don't think this will help you be a cryptographer, but it is a fun history of the space and light introduction to the topic. If other things here are too serious then this can be a more gentle introduction to help get you ready for the rest.
1. (Just for fun) [The Woman Who Smashed Codes](https://www.amazon.com/dp/B01M0EOI6I) by Fagone is a *fascinating* biography of Elizebeth Smith Friedman. In many ways she was the creator of modern code-breaking in the US and did much unsung work to protect the allies during both world wars.

### Activities

1. [Cryptopals Crypto Challenges](https://cryptopals.com/)
  Here is the first "hands-on" resource. It takes you through building and breaking many standard cryptographic algorithms.
  It starts easy and gets *really hard* at the end. Even if you cannot complete it, you should go as far as you can and keep chipping away at it as you get better.
1. [CryptoHack](https://cryptohack.org/)
  This is similar to "CryptoPals" but newer and flashier. It also looks great but I've only done portions of it.
  My own real complaint is that basically requires that you work in Python and deal with network programming as well.
  While neither of these are big distractions, I'm personally not a fan of Python and like being able to work offline.

### Other reading

1. [How to Learn Cryptography as a Programmer](https://soatok.blog/2020/06/10/how-to-learn-cryptography-as-a-programmer/) by Soatok
1. Read the source of the libraries you use.
1. Read *tons* of specifications. You use AES-GCM? Read [NIST SP 800-38D](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf) You use HMAC? Read [RFC #2104](https://tools.ietf.org/html/rfc2104)
1. Read [The Stick-Figure Guide to AES](http://www.moserware.com/2009/09/stick-figure-guide-to-advanced.html)
1. Read the [Latacora Blog](https://latacora.singles/) (especially the "Right Answers")
1. Read my [Cryptographic Gotchas](./index.md) list.
1. Read [If You’re Typing the Letters A-E-S Into Your Code You’re Doing It Wrong](https://www.e-x-a.org/stuff/articles/typing-a-e-s/) by Thomas Ptacek

### Miscellaneous

1. Find people working in spaces closer to professional cryptography than you and ask them to help you (buy them beers or beverages of choice)
1. Look at public issues on GitHub for libraries you use and see if you can contribute, or at least understand them
1. Follow the [IACR](https://www.iacr.org/). (There are three good Twitter accounts: [official @IACRcrypto](https://twitter.com/IACRcrypto), [official @IACR_News](https://twitter.com/IACR_News), and  [unofficial @IACRePrint](https://twitter.com/IACRePrint) which follows (unreviewed) papers).)
1. Take a look at the [crypto subreddit wiki](https://www.reddit.com/r/crypto/wiki/index) (Excellent when I looked in May 2021.)
  I especially recommend looking at the "How to get more involved section" because becoming part of the community is one of the most useful things.
  Once I became part of the crypto community (back in early 2018) things became much easier for me and I started getting much better.
  This is because when I had questions or was trying to figure things out,
  I could easily look through the people I followed (and even ask questions) to track down the resources I needed.
1. Remember (and try to follow) any company/person mentioned in this list.
1. **KNOW YOUR LIMITS**

## Contributions and Licensing

Please see the [Contributions and Licensing section of the main document](https://github.com/SalusaSecondus/CryptoGotchas/blob/master/README.md#contributions-and-licensing).
