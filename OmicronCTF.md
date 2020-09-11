# OmicronCTF
> [https://omicronctf.xyz/](https://omicronctf.xyz/)


## Cryptography


###	Base64

> This challenge will teach you how to decode base64
> 5 points	|	Easy

> `b21pY3Jvbnt3M2xjMG0zfQ==`

Avec :
```bash
$ echo "b21pY3Jvbnt3M2xjMG0zfQ==" | base64 -d
(FLAG)
```

### Binfork
> This challenge is a confusing crypto challenge, can you solve it?
> 5 points  | Easy

> o10101101m10101101{10101101h10101101410101101h10101101410101101h10101101410101101h10101101h10101101_10101101r10101101310101101410101101l10101101_10101101f10101101010101101r10101101k10101101110101101_10101101010101101G10101101s10101101_10101101w10101101110101101l10101101l10101101_10101101k10101101n10101101010101101w10101101_10101101;10101101)10101101}

Il suffit de retirer tout les `10101101` du texte et on obtient le flag.


## OSINT

### Links
> This is is an easy-medium OSINT challenge, while it does take some steps to complete, it doesn't involve any step that is too hard.
> 10 points | Medium

> We heard of this guy bragging about his botnet, he goes under the name of "anomalousitalypseudogangsta" (weird name, huh?). He's also said to be active on social media. Anyway, can you find out more about him? (no website-accounts required!)

