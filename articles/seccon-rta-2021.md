---
title: "RTACTF 2021 Crypto並走記録"
emoji: "😸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [CTF]
published: true
---

## はじめに

RTACTF 2021 の Crypto で途中から並走していたので，その記録を残しておきます．最初の辺りはコタツの中でぬくぬくと観戦していましたが，自分でもやってみたかったので外野で参戦しました．

- [RTACTF 2021](https://speedrun.seccon.jp/)

https://youtu.be/VXaROnAmAiY

## Proth RSA [666.46 sec]

![](https://storage.googleapis.com/zenn-user-upload/aab8ea614fff-20211219.png)

### 問題ファイル

```python:chall.py
from Crypto.Util.number import getRandomInteger, getPrime, isPrime
import os

def getProthPrime(n=512):
    # Proth prime: https://en.wikipedia.org/wiki/Proth_prime
    while True:
        k = getRandomInteger(n)
        p = (2*k + 1) * (1<<n) + 1
        if isPrime(p):
            return p, k

if __name__ == '__main__':
    # Plaintext (FLAG)
    m = int.from_bytes(os.getenv("FLAG", "FAKE{sample_flag}").encode(), 'big')

    # Generate key
    p, k1 = getProthPrime()
    q, k2 = getProthPrime()
    n = p * q
    e = 65537
    s = (k1 * k2) % n

    # Encryption
    c = pow(m, e, n)

    # Information disclosure
    print(f"n = 0x{n:x}")
    print(f"e = 0x{e:x}")
    print(f"s = 0x{s:x}")
    print(f"c = 0x{c:x}")

```

### 解法

$k_1$, $k_2$ を乱数として，素数 $p$, $q$ は次のような式で書けます．

$$
\begin{align*}
    p &= (2 k_1 + 1) \cdot 2^{512} + 1 \\
    q &= (2 k_2 + 1) \cdot 2^{512} + 1
\end{align*}
$$

このとき，$n=pq$, $s \equiv k_1 k_2 \pmod{n}$ が情報として与えられます．

$s$ について軽く実験してみると，$k_1 k_2 < n$ だったので，$s = k_1 k_2$ になることがわかりました．ということは，$n$ と $s$ の連立方程式を解いてあげると $k_1$, $k_2$ が求まりそうです．

後で放送のアーカイブを見てみるとグレブナー基底で解かれていましたが，グレブナー基底 1 mm もわからんって感じでした．僕はそんな高等テク？は使えないので，Sage の `solve` を使いました．

```python:solve.sage
from Crypto.Util.number import long_to_bytes


n = 0xa19028b5c0e77e19fc167374358aa346776e6c20c27499505be59c83ea02014e97af631ba0ccbab881313818fd323c15c82dad8793220ba6679ec4b38787e04d0c1fff0880e04423ea288e443660c63a1607532e47dbaad421723d0546c208447f701cd7e9ee1bb43774d132abbb2e91bf50b67be40ed854dbe6c3071ca3ae3307ac03abd76f74e506594106a22795d4b7938611301248a9957e1a637538a9169cf38daf5d60ffc05ae32ea7e638e16d790ffeebfff655a645c99a513616d3ce00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001
e = 0x10001
s = 0x28640a2d7039df867f059cdd0d62a8d19ddb9b08309d265416f96720fa808053a5ebd8c6e8332eae204c4e063f4c8f05720b6b61e4c882e999e7b12ce1e1f812c11cfed72a5c33cfb8f3d34f650e4c19579cf34745f2588aa2fd08a8746257cb789f23ca232346fcf72468a2b160934911902de3f90620aba5874a2d79a33699
c = 0x4595c3c923bd191ba07456611f80e656a197ff528a031e2952adedda532b1fa2caef719c929132a3cdf06d0e55e6a00f7eb1f189a614b26759916ec42f83579a75ab5948186769a1a936b019466f918f29e32852675c464b7f0797c6fdc55efcd54fbe2083761b1df3dde0b9a9a35b96e3b216c54770b444b1f02525f0268c44483c6e84a781fe9111e6912130d69f462c519873043d44e4a3f1f938491feeb591b5831d0abe7399bc87244576decaf2925f287d3c2bb4061d560c919d820e364744f2322c7efd37d42563842bcf9b1d6b46218694dcd49758d311c6896e38cf2b55c7114d78cfdfaeba74720ecf30d9133034799b9735e26ec913cc9f26bb0a

var("x y")
f = ((2*x + 1) * (1 << 512) + 1) * ((2*y + 1) * (1 << 512) + 1) - n
g = x * y - s
res = solve([f, g], x, y)

k1, k2 = res[0]
k1, k2 = k1.roots()[0][0], k2.roots()[0][0]
p = (2*k1 + 1) * (1 << 512) + 1
q = (2*k2 + 1) * (1 << 512) + 1

d = pow(e, -1, (p-1) * (q-1))
m = pow(c, d, n)
print(long_to_bytes(int(m)))

```

```
$ sage solve.sage
b'-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-= RTACON{d1d_U_us3_th3_p0w3r_0f_Groebner_basis?} =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-'
```

## Leaky RSA [リタイア]

40 分ぐらい格闘しましたが，解けませんでした 😢

後でアーカイブや Twitter を見て解き直しましたが，やっぱり悔しいですね．ちなみに，フラグを入力せずに放置しているので，経過時間が恐ろしいことになってます．

![](https://storage.googleapis.com/zenn-user-upload/ae7842803436-20211219.png)

## Neighbor RSA [759.17 sec]

2 秒差で 1st 取ったかと思ったら，途中で追い越されてました．

![](https://storage.googleapis.com/zenn-user-upload/03c3e07320e3-20211219.png)

### 問題ファイル

```python:chall.sage
import os

# Plaintext (FLAG)
plaintext = os.getenv("FLAG", "FAKE{sample_flag}").encode()
plai  = plaintext[:len(plaintext)//2]
ntext = plaintext[len(plaintext)//2:]
m1 = int.from_bytes(plai  + os.urandom(128), 'big')
m2 = int.from_bytes(ntext + os.urandom(128), 'big')

# Generate key
e = 65537
p = random_prime(1<<2048)
q = random_prime(1<<512)
r = random_prime(1<<512)
n1 = p * q
n2 = next_prime(p) * r
assert m1 < n1 and m2 < n2

# Encryption
c1 = pow(m1, e, n1)
c2 = pow(m2, e, n2)

# Information disclosure
print(f"e = {hex(e)}")
print(f"n1 = {hex(n1)}")
print(f"n2 = {hex(n2)}")
print(f"c1 = {hex(c1)}")
print(f"c2 = {hex(c2)}")

```

### 解法

$p$ と $p$ の次の素数の差分を $\varepsilon$ と書くと，$n_1$, $n_2$ は次のように表されます．

$$
\begin{align*}
   n_1 &= pq \\
   n_2 &= (p+\varepsilon) r
\end{align*}
$$

$\varepsilon$ さえなければ $\gcd(n_1, n_2)$ で $p$ が出るのになーって考えていると，「ん？そういえば近似的な $\gcd$ ってあったよな？？？」と思い出してブックマーク一覧を見てみると，ふるつきさんの記事が出てきました．

- [Approximate GCD として解く問題 - RCTF 2021 Uncommon Factor 2 / Midnightsun CTF 2021 Finals Flåarb.tar.xz writeup - ふるつき](https://furutsuki.hatenablog.com/entry/2021/09/23/122340)

この記事と前に試した記憶を参考にして Solver を書きました．

原理はよく理解できていないので，今度暇なときに勉強したいですね．ちょくちょく，次のペーパーを読んでるんですけども，数学よわよわなのでよくわからないです．

- [Algorithms for the Approximate Common Divisor Problem](https://eprint.iacr.org/2016/215.pdf)

```python:solve.sage
from Crypto.Util.number import long_to_bytes


e = 0x10001
n1 = 0xa8ed020c3dd125d503bf124052d643ba1405f2c349244122140e79e7d2244304a1590762c61ac83900c2aced76007b2e3f320464fd51fcfad167ebdc87e69329230869e0a3e153b44ed3b04bfe94174bc8b5ee1a3fa8036b6b9e834666aa07229a431b477e589d94f9a4cfed25b195215b0c694b86e874413b8a00cb064809c8e3677632cde9b43b87a0b812c2024b0c821b5c10764fd4de2d18af55d897d94aeded80b71e36fd73014f75641a8c5b38b36faa020e7cf1327a707bb7d42503bcc28768ef184d66b9ba16efd019b68268885a2da302cd326e78b1d473bcf7cd62442ccd25dc85d23aeb5408922b6b00f13584bea394f1bca4cc431f3c29c5d98ec1683453cc0c526abe4aec08781c7a53f50f2047b4995b9bea6a7a9f6b5425b29be6e867764efaa050799f716af78273041372cfe4f3c88a62329f6f1feff99
n2 = 0x16ea1bde86ea11cdec196a9173258efca235da66f8e3d5437e39e1b2e2574dd3f93d65104ca0225d6119519ae9ea9c035e0f85f02212c0992d0705723fa8b97ed6cff860c4d8fb65f0214a0047feca64e662dcbf025fff47590305e90e5d070d39871880828f5e960ab2ef330129ed5752c3b4debe827376a632b06487740fff4b622a88de23649e3e6993cd332b0284b84eb8765d58527209cc202c89d479421131a2f64ae517ee1e62e6c0f329c306569e427113ec6a8b9d96d73e95580d3a33f6add681f9a9156f0681eb1804183dfa8cebbe921d2fb1d43b256f727d46c5859cc5229f7e555ad25397e5cd14620ebbaefa0a0a520bada3ef8b115481734242af6befbd9b069d4a03281094c0f4aca4e6fdcbe2558b104fc2b383e1c70f0e5a07d1a623f9fc2309ca1d09b69aa1869e280fbc50de2adbada7ea545743b12b
c1 = 0x690e49037fee7649033ffaaa71e4730d2d7143fab97beb22e2afdf6eca449cad3f95b60295f592e7e84833e08b3468d61a34c1d1123f4c683c79d68bbe27dd0af203fc50ef7ebe98b1bc1221918470f058a8fb7645eacb569931835bd7f80494dbb67fbaa592ec19d9b4930c787a2ce1267f8088229b5031e710d6cd5720756923ccb64444939a0f09a51c87488650d4d02551fd4ed7a2fd248825ec34c5df8b6077a6d0d75c5832f9140420c92d3d00cf51e3b0665f5a6d031cb369ddebbb5ce77f2176cd12bb0add5aeda6ae88c4ceade0c1fd0ff3960d3ee36a0c6455ae3027f33e660663d0e2298654e19e8c8a06b4de991fac3b4c1673825b3d9f8f5c675f920a7d137f85ba723bf741321904e0c3c601f5c18d02e1e5b7b118e62e91a7926a9b1eda3cc53e2a6cbc95553e1990ec3f6cceddf283410d6e6849a26f89b
c2 = 0x5c4c7dce82753a68dcdbcdce9af52c9b7af2f561c08b8e23b27c6145d4c3df29d498303bee1bd29829a2e0ae9faaf243b387c39d69daccba07dace7bb420115ffaa69f89a3ea4e1ef0e08eb19043e012a090b79e51d6ae8446ca76e88abe5adbdbe25a731d7ee9aa333a84447edafbc360b505ff293c751571c6bf29dee99fdc443b756f182eb588b4a03de3d35dc4f23736d7239cfbd0ca13fa7b234bc4064a2053ab0045f4833250c8c9de91798502b09d4312ee52f3dc5229dfcb73b42f7c3440932839e6e790bb0db1788fbd7c60365121bbe3858ecedd3d48261d081c380e7ddf6ca570c13cc89c0af2011b4978b22d5456d1122dabd7b2068ab30e301a674809732daede77a27ae13e1bc4779e15d51210f6c10be159907ec1a59bfaf8db6cf290a348f734fd88e3c2b7df6bda84665b810cfe55bc3645d8d118c9172

C = 2 ^ 1024
B = matrix(ZZ, [
    [C, 0, n2],
    [0, C, -n1]
])
res = B.LLL()
v = res[0]
q = v[0] // C
r = v[1] // C
p = n1 // q
_p = n2 // r

d1 = pow(e, -1, (p-1)*(q-1))
d2 = pow(e, -1, (_p-1)*(r-1))
m1 = pow(c1, d1, n1)
m2 = pow(c2, d2, n2)

print(long_to_bytes(int(m1)))
print(long_to_bytes(int(m2)))

```

```
$ sage solve.sage
b"RTACON{1nt3nd3d_s0lut10n_\xa3m\x1c\xf1/w\x8b\xdcH\x15\xe5\xaf#\xc3\xc8G\x10\r\xf8\x00\x00M1\x06^\xfeB\xa1X\xf0\x88I\xd9&\x07\xb2t\xb2%\xfe\xcc\xae\x8aa\x7fxukDEa\xab\x15{\x8e\x9d\x87m\x81\x95\xeb\xe9\xc7l_{U\xae\xbb\x1d}\xa40*\xbe\xad'\xa6k\xdeJ\x0e\xceI\xfd\xb3T\xa9\x7f\\\x08\xc0\x14\x00\xd2\x06\xf2\x1d%\x98Y\xea\xbe\x86\xee\xc1H\xea\xf6\x10\x95CU$\x04\xb9\x08\xb8\xcax\x89x)\xef\xa3\xfbQ\xe4"
b"1s_Approximate_GCD_:pray:}z\xa5I\xec8\xcdF\xc2\xf0\xa4r\xbc\x03\x95\xfb\xf5^GP\x80\xc3X\xe5\xbf\x9a\xbcb\xbc\x89Z?\xcc\xc7\x94\x08\x18\xc91-P\xbf\xda\xb9\n)\x82\xa6\xd2\x8a$\xff\xbc_`\xa4\xaa\xe7\xa4\x10\x1a?\xfd\xd9\x92>_\xd44\x86\x80\x16\x9ebcW\xf7j\x18\xdax4\x83\xf4\x0bNGa\x91\x13.\x8a\xac\x1bA\xfd\xb4\xef\x8a\x88Z0\x91\x83:?\xeecA\x1b\x7f\xda\xbd\xc6\x81\xd7\x04\xfd\xa7\xb7\xc6\xeb\x04\xd0\x83\x7f\xf4'\xf8"
```

## 感想

普段はまったりと CTF を解いているので，RTA 形式になると少しのミスで焦ってしまいますね．特に Proth RSA は焦りに焦りまくってバグのオンパレードでした．しかし，この緊張感がとても楽しかったです（語彙力）．

運営の皆さん，本当にありがとうございました & お疲れ様でした！
