# Markdown experiments

Foo bar

| <span class="iconify" data-icon="charm:person"></span> [gotchas.salusa.dev](https://gotchas.salusa.dev/) | <span class="iconify" data-icon="tabler:brand-github"></span> [github.com/SalusaSecondus/](https://github.com/SalusaSecondus/) | <span class="iconify" data-icon="tabler:brand-signal"></span> [Signal: SalusaSecondus.3514](https://signal.me/#eu/NaQ8xeUOpB4g_coC2NUe-VFZGp108FRTjkQwtWiQpUuc0q7UC3NM06BCFooBopci) |
|-|-|-|
| | | |

H~2~0 or 2^32^.

In-line code: `H~2~0 or 2^32^`.
Code block:

```plain
H~2~0 or 2^32^
```

H<sub>2</sub>0 or 2<sup>32</sup>.

In-line code: `H<sub>2</sub>0 or 2<sup>32</sup>`.
Code block:

```plain
H<sub>2</sub>0 or 2<sup>32</sup>
```

\(\lambda\)

Now Alice has a keypair A<sub>s</sub> and A<sub>p</sub> (her private/secret and public keys respectively).
She can use this to generate a signature \(sig=\texttt{Sign}(A_s, M)\) and send that to B along with `M`.
Bob can then just check that \(\texttt{verify}(A_p, M, sig)\) is `true` and know that Alice sent the message.
