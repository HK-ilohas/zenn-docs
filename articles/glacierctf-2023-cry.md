---
title: "GlacierCTF 2023 writeup [crypto]"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ctf"]
published: true
---

## はじめに

https://play.glacierctf.com/

GlacierCTF 2023 に出て Crypto の簡単な問題を解いたので、そのときのメモを編集して公開します。CSIDH なにもわからないし、執筆当時（2023-11-27）では多分 writeup 出てない…

## intro

### Welcome challenge

From Official Discord.

```
gctf{w3lc0m3_t0_g1ac13rctf_2023}
```

### ARISAI

:::message
I heard that RSA with multiple primes is more secure. My N is very large, so there should not be a problem.
:::

添付ファイル: `chall.py` `output.txt`

Multi Prime の問題。

```python
from Crypto.Util.number import bytes_to_long
from Crypto.Util.number import getPrime

PRIME_LENGTH = 24
NUM_PRIMES = 256

FLAG = b"gctf{redacted}"

N = 1
e = 65537

for i in range(NUM_PRIMES):
    prime = getPrime(PRIME_LENGTH)
    N *= prime

ct = pow(bytes_to_long(FLAG), e, N)

print(f"{N=}")
print(f"{e=}")
print(f"{ct=}")

```

24 ビットの素数なので、簡単に素因数分解できる。以下は sagemath の例。

```python
from output import N

fs = list(factor(N))
print(fs)

```

あとは CRT で復号するだけ。

```python
from Crypto.Util.number import *
from functools import reduce
from output import N, e, ct


def CRT(a, m):
    MM = reduce(lambda a, b: a * b, m)
    x = 0
    for i in range(len(a)):
        Mi = MM // m[i]
        ti = pow(Mi, -1, m[i])
        x += (Mi * ti * a[i]) % MM
    return x % MM

fs = [(8441831, 1), (8450987, 1), (8452019, 1), (8473027, 1), (8476817, 1), (8523661, 1), (8525711, 1), (8608673, 1), (8633423, 1), (8641453, 1), (8725153, 2), (8786017, 1), (8796721, 1), (8824679, 1), (8850601, 1), (8913481, 1), (8933437, 1), (9016037, 1), (9041551, 1), (9075889, 1), (9095939, 1), (9126197, 1), (9142547, 1), (9163981, 1), (9172531, 1), (9196001, 1), (9223867, 1), (9253319, 1), (9265309, 1), (9277921, 1), (9298747, 1), (9300803, 1), (9357883, 1), (9368759, 1), (9405353, 1), (9444839, 1), (9552029, 1), (9569057, 1), (9584371, 1), (9663629, 1), (9696719, 1), (9720223, 1), (9748049, 1), (9770723, 1), (9801269, 1), (9828727, 1), (9836483, 1), (9838117, 1), (9853043, 1), (9873373, 1), (9883469, 1), (9884603, 1), (9905167, 1), (9989579, 1), (10000759, 1), (10064897, 1), (10114409, 1), (10122389, 1), (10213001, 1), (10214591, 1), (10228861, 1), (10235447, 1), (10344643, 1), (10428001, 1), (10433911, 1), (10438013, 1), (10441523, 1), (10476001, 1), (10514083, 1), (10523977, 1), (10605817, 1), (10650929, 1), (10667479, 1), (10699517, 1), (10731407, 1), (10732091, 1), (10754837, 1), (10773781, 1), (10849837, 1), (10861127, 1), (10893173, 1), (10918459, 1), (10943417, 1), (10944433, 1), (11028001, 1), (11049739, 1), (11057621, 1), (11073793, 1), (11084419, 1), (11113789, 1), (11152859, 1), (11156681, 1), (11230451, 1), (11239903, 1), (11369903, 2), (11462177, 1), (11470343, 1), (11504419, 1), (11519971, 1), (11543971, 1), (11559637, 1), (11625619, 1), (11633267, 1), (11661121, 1), (11768401, 1), (11847721, 1), (11909747, 1), (11915809, 1), (11925691, 1), (11928173, 1), (11945093, 1), (11990089, 1), (12010259, 1), (12089663, 1), (12109277, 1), (12231853, 1), (12240667, 1), (12274813, 1), (12319117, 1), (12339689, 1), (12350357, 1), (12358079, 1), (12387329, 1), (12407609, 1), (12407959, 1), (12515033, 1), (12550357, 1), (12599803, 1), (12621067, 1), (12652597, 1), (12705883, 1), (12804707, 1), (12808151, 1), (12824027, 1), (12932669, 1), (12967831, 1), (13046717, 1), (13059269, 1), (13076249, 1), (13128433, 1), (13170671, 1), (13202297, 1), (13227367, 1), (13328803, 1), (13366687, 1), (13371181, 1), (13415921, 1), (13417357, 1), (13424921, 1), (13430423, 1), (13534007, 1), (13561657, 1), (13566431, 1), (13568981, 1), (13587683, 1), (13625263, 1), (13653811, 1), (13655797, 1), (13669967, 1), (13673927, 1), (13755149, 1), (13799299, 1), (13823059, 1), (13865617, 1), (13870601, 1), (13997617, 1), (14013617, 1), (14044937, 1), (14046449, 1), (14086979, 1), (14103413, 1), (14162843, 1), (14217041, 1), (14311291, 1), (14339863, 1), (14340289, 1), (14377679, 1), (14407667, 1), (14423561, 1), (14435203, 1), (14465153, 1), (14466281, 1), (14475521, 1), (14482381, 1), (14535811, 1), (14548939, 1), (14549063, 1), (14588369, 1), (14624459, 1), (14633851, 1), (14650763, 1), (14693927, 1), (14713939, 1), (14738869, 1), (14797501, 1), (14880347, 1), (14910199, 1), (14922409, 1), (14982181, 1), (15005579, 1), (15020413, 1), (15031937, 1), (15103373, 1), (15181499, 1), (15185399, 1), (15209617, 1), (15232961, 1), (15299831, 1), (15365261, 1), (15441739, 1), (15459343, 1), (15470893, 1), (15475193, 1), (15489707, 1), (15501071, 1), (15682181, 1), (15689647, 1), (15689981, 1), (15707093, 1), (15707143, 1), (15748631, 1), (15792169, 1), (15793247, 1), (15798877, 1), (15922301, 1), (15947639, 1), (16032721, 1), (16045049, 1), (16071229, 1), (16080319, 1), (16175597, 1), (16177433, 2), (16198717, 1), (16199101, 1), (16212913, 1), (16225283, 1), (16254883, 1), (16312763, 1), (16336267, 1), (16359283, 1), (16405027, 1), (16432721, 1), (16497373, 1), (16593167, 1), (16594681, 1), (16629163, 1), (16632713, 1), (16643707, 1), (16657153, 1), (16679137, 1), (16701907, 1), (16738913, 1), (16755269, 1)]
ms = []
ns = []
for p, k in fs:
    n = pow(p, k)
    phi = pow(p, k) - pow(p, k - 1)
    d = pow(e, -1, phi)
    m = pow(ct, d, n)
    ms.append(m)
    ns.append(n)

m = CRT(ms, ns)
print(long_to_bytes(m).decode())

```

```
gctf{maybe_I_should_have_used_bigger_primes}
```

## crypto

### Missing Bits

:::message
Somehow your private key lost some bits. Are you able to reconstruct them and decrypt the ciphertext?
:::

添付ファイル：`ciphertext_message` `encode_file.py` `priv.key`

普通の RSA っぽい。

```python
from Crypto.PublicKey import RSA
from Crypto.Util.number import bytes_to_long
from Crypto.Util.number import long_to_bytes

content = open("2048_key_original.priv", "rb").read()
key = RSA.import_key(content)

filecontent = open("plaintext_message", "rb").read()

ct = pow(bytes_to_long(filecontent), key.e, key.n)

open("ciphertext_message", "wb").write(long_to_bytes(ct))

```

しかし、`priv.key` が破損している。

```
xwiBgUBTtaSyUvdrqfgAEduRdnzRbKcwEheMxwIDAQABAoIBAAqaJbojNCwYqykz
n0Fn2sxMsho4PhThPQcX79AGqSxVNxviWK2GXETP7SsnvWGmRXHIRnR6JGOhyHVe
dTDYZxOAOhl85FksVQYVUaygf9Epekja/vSj5OE8NIcAdEBr3aZ6gdLxi+q1a5Kh
1nEmsF6FiYGpsPkN63ovbow/CKt4N7UQJqZEQw380rNA0sOQenmzXRFOpXA8PRFb
G6itGRiP1gB9tpdQnWggQ5n+x8/2k+k3CRW6/xIP9dMAVZh2jVombenLxgnhQCJB
bYaR4I8B0zzYqXqFfeHCMNl+pJmmmFcvs2ZE71fqyjRid6ZDqS4GXtSuRQM0UL7L
UQVBaYECgYEA5BiLN7FjwgOuT4FKxFdzizdq/t5mvRksbmBP/JWk3vxQYeCmMiPQ
xrQUqdHGGxG8iMIwH7dnhNaPa81lrP9fCKyij/caEbe4lmEm+VdM/xZQF+PiCc1f
ziYXphz9wuAc8++kvKxM2EaiDe8F25nsXW+FaxNoXKbJg0zTQLyzKiECgYEA8SLi
hbAwo2l0zal8GMIemzr+APxLw+fmd4aryVAMov8ANkG8KDMwdmvvkn3rL7WaKym5
fakqvXR45/QGPe8niVzx6oaWGSSfijeVan27pG/bzVqyymFHZP9cRhEHW4HN57hO
pXy0kUFqVaxJWCs+thH0LTZoToAepg+sr82FaecCgYEAnJnpQzRsHDEwxO8sqP6t
moBS2mdRPDUDV0iSwgTvrBSpD3oQQM5sMXBD24/lpoIX4gEIz025Ke+xij77trmh
wq/b8GGjqVRsy/opqvjwKRZlqPFRKI+zXjKy+95dryT1W9lFTjAxli9wZYaci/fy
2veNL0Wk2i+8nIPraj/j9mECgYEA0ou6LC7OGTDwKs7sqxV78eBNfoDMis7GPeEZ
x9ocXom3HqjA6HzhuNS/xzIpE2xGo5939c+qoOe81hMNDDDwXZEJLdS75FJE90NX
NDd6iracvi6OZAUSeI47fHZL7UtmhQg5q2c6poXumcWn+NMxm3oLsRqLcteNa0PO
bWZPMksCgYBidblknACvEinDUQB8dvElzROql0ZUMX9hQOsSrg0ju3smun8qujcT
PJQrWctoNw0ZXnQjDkVzaJxog1F3QkKUg6B1Rn2Q0RYuCAeNDn+KqBkTT18Du7yw
+GU/4U6EMw+uL7dNjasD1jjx90ro6RmDDBmGDQuludO0G7h9XQzl+Q==
-----END RSA PRIVATE KEY-----
```

とりあえず、`---` の行を消して Base64 デコード。

```console
$ cp priv.key dump.key
$ vim dump.key
$ cat dump.key | base64 -d > dump.bin
```

この記事を参考にパースした。すると、復号に必要なパラメータが判明した。

https://bearmini.hatenablog.com/entry/2014/02/05/143510

```python
from Crypto.Util.number import bytes_to_long, long_to_bytes


dump = open("dump.bin", "rb").read()

# https://bearmini.hatenablog.com/entry/2014/02/05/143510
data = dump[0x1c:] # start from exp
nums = []
while data:
    type_sym = data[0]
    size_sym = data[1]
    size_msb = (size_sym & 0b10000000) >> 7
    if size_msb == 0:
        size_data = size_sym
        data = data[2:]
    elif size_msb == 1:
        size_sym = size_sym & 0b01111111
        size_data = bytes_to_long(data[2:2+size_sym])
        data = data[2+size_sym:]
    # 2: INTEGER
    if type_sym == 2:
        num = bytes_to_long(data[:size_data])
        nums.append(num)
    data = data[size_data:]

e = nums[0]
d = nums[1]
p = nums[2]
q = nums[3]

ct = open("ciphertext_message", "rb").read()
m = pow(bytes_to_long(ct), d, p*q)
print(long_to_bytes(m).decode())

```

```
Hey Bob this is Alice.
I want to let you know that the Flag is gctf{7hi5_k3y_can_b3_r3c0ns7ruc7ed}
```

### SLCG

:::message
In cryptography class we learned about random numbers and algorithms to create pseudo random number generators. I think I build a solid cipher that nobody can break. This is why I call it SecureLongCiphertextGenerator, SLCG for short
:::

添付ファイル: `ciphertext.txt`, `encrypt.py`

ソースコードを見てみる。普通の線形合同法で何かしてるっぽい。

```python
from __future__ import annotations

import os

FLAG = b"gctf{???????}"


class LCG:
    def __init__(self, mod: int, mult: int, add: int, seed: int):
        self.mod = mod
        self.mult = mult
        self.add = add
        self.value = seed

    def __next__(self) -> int:
        self.value = (self.value * self.mult + self.add) % self.mod
        return self.value

    def __iter__(self) -> LCG:
        return self

    @classmethod
    def random_values(cls):
        return LCG(
            int.from_bytes(os.urandom(16)),
            int.from_bytes(os.urandom(16)),
            int.from_bytes(os.urandom(16)),
            int.from_bytes(os.urandom(16))
        )


class Encryptor:
    def __init__(self):
        self.lcgs: tuple[LCG] = (LCG.random_values(), LCG.random_values())

    def encrypt(self, message: str) -> list[int]:
        result = []
        for ascii_char in message:
            bin_char = list(map(int, list(f"{ascii_char:07b}")))

            for bit in bin_char:
                result.append(next(self.lcgs[bit]))

            self.lcgs = (
                LCG(
                    next(self.lcgs[0]), next(self.lcgs[0]),
                    next(self.lcgs[0]), next(self.lcgs[0])
                ),
                LCG(
                    next(self.lcgs[1]), next(self.lcgs[1]),
                    next(self.lcgs[1]), next(self.lcgs[1])
                )
            )
        return result


def main() -> int:
    encryption = Encryptor()
    print(f"ct = {encryption.encrypt(FLAG)}")

    return 0


if __name__ == "__main__":
    raise SystemExit(main())

```

普通に動かすとエラーが出たので微修正。

```diff
- int.from_bytes(os.urandom(16)),
+ bytes_to_long(os.urandom(16)),
```

一文字ずつバイナリにして、ビットに対応する LCG の出力を暗号文にしている。

```python
bin_char = list(map(int, list(f"{ascii_char:07b}")))
```

LCG は連続する乱数列 5 つ以上でパラメータを復元できるかも。

https://tailcall.net/posts/cracking-rngs-lcgs/

`ciphertext.txt` を `ciphertext.py` にしてソルバーを書く。色々と試したら、最初の文字 `g` でパラメータを復元できた。方針としては、`self.lcgs[1]` がわかったので、それで生成した値と一致したら `1`、そうでなければ `0` として文字の各ビットを特定する。

試行錯誤しながらのプログラムだから、汚いのはご愛嬌。

```python
from functools import reduce
from Crypto.Util.number import inverse, GCD
from ciphertext import ct


# https://tailcall.net/posts/cracking-rngs-lcgs/
def solve_unknown_increment(states, A, M):
    B = (states[1] - A * states[0]) % M
    return B


def solve_unknown_multiplier(states, M):
    A = (states[2] - states[1]) * inverse((states[1] - states[0]), M)
    return A


def solve_unknown_modulus(states):
    diffs = [X_1 - X_0 for X_0, X_1 in zip(states, states[1:])]
    multiples_of_M = [T_2 * T_0 - T_1 ** 2 for T_0,
                      T_1, T_2, in zip(diffs, diffs[1:], diffs[2:])]
    M = reduce(GCD, multiples_of_M)
    return M


def lcg(X_0, M, A, B):
    X_1 = (A * X_0 + B) % M
    return X_1


# ビット1のところだけを取り出す
ascii_char = b"g"[0]
bin_char = list(map(int, list(f"{ascii_char:07b}")))
x = []
for i in range(7):
    if bin_char[i] == 1:
        x.append(ct[i])

# LCGのパラメータを復元
M = solve_unknown_modulus(x)
A = solve_unknown_multiplier(x, M)
B = solve_unknown_increment(x, A, M)

# check
X = [x[0]]
X_0 = x[0]
for _ in range(len(x)-1):
    X_1 = lcg(X_0, M, A, B)
    X.append(X_1)
    X_0 = X_1
assert X == x

# フラグの長さ
SIZE = len(ct) // 7

flag = "g"
X0_0 = x[-1]
M0 = M
A0 = A
B0 = B
for i in range(SIZE-1):
    # change param
    M1 = lcg(X0_0, M0, A0, B0)
    X0_0 = M1
    A1 = lcg(X0_0, M0, A0, B0)
    X0_0 = A1
    B1 = lcg(X0_0, M0, A0, B0)
    X0_0 = B1
    X1_0 = lcg(X0_0, M0, A0, B0)

    bin_char = []
    # ビットが1か0かを特定する
    for j in range(7):
        X1_1 = lcg(X1_0, M1, A1, B1)
        if ct[7*(i+1)+j] == X1_1:
            bin_char.append("1")
            X1_0 = X1_1
        else:
            bin_char.append("0")
    ascii_char = "".join(c for c in bin_char)
    ascii_char = chr(int(ascii_char, 2))
    flag += ascii_char

    X0_0 = X1_0
    M0 = M1
    A0 = A1
    B0 = B1

print(flag)
```

```
gctf{th15_lcg_3ncryp710n_w4sn7_s0_5s3cur3_aft3r_4ll}
```

### Glacier Spirit

:::message
You have climbed the mountain and reached the top. You are now in the presences of the glacier spirit which seems to have an affinity for ASCON. But it seems confused about how to use it.

author: mcsch

`nc chall.glacierctf.com 13379`
:::

添付ファイル: `challenge.py`

```python
#!/usr/bin/env python3

import ascon
import secrets
from secret import FLAG

BLOCK_SIZE = 16


def xor(a, b):
    return bytes([x ^ y for x, y in zip(a, b)])


def split_blocks(message):
    return [message[i:i + BLOCK_SIZE] for i in range(0, len(message), BLOCK_SIZE)]


def mac_creation(key, message):
    assert len(message) % BLOCK_SIZE == 0
    message_blocks = split_blocks(message)
    enc_out = b"\x00" * BLOCK_SIZE
    for message in message_blocks:
        chaining_values = xor(message, enc_out)
        enc_out = ascon.encrypt(key, chaining_values, b'', b'')
    assert len(enc_out) == BLOCK_SIZE
    return enc_out


def pad_message(message):
    first_block_pad = len(message) % BLOCK_SIZE
    first_block_pad = 16 if first_block_pad == 0 else first_block_pad
    return (first_block_pad.to_bytes() * (BLOCK_SIZE - first_block_pad)) + message


def encrypt(key,  message):
    assert len(message) % BLOCK_SIZE == 0
    message_blocks = split_blocks(message)

    assert len(message_blocks) < BLOCK_SIZE
    nonce = secrets.token_bytes(BLOCK_SIZE-1)

    cts = []
    for ctr, message in enumerate(message_blocks):
        cipher_input = nonce + (ctr+1).to_bytes(1, 'little')

        enc = ascon.encrypt(key, cipher_input, b'', b'')

        ct = xor(message, enc)
        cts.append(ct)
    return nonce, b''.join(cts)


def create_message_and_mac(key, message):
    padded_message = pad_message(message)
    nonce, ct = encrypt(key, padded_message)
    tag = mac_creation(key, padded_message)
    return nonce, ct, tag


if __name__ == "__main__":
    print("              Glacier Spirit\n\n")
    print("           ,                  /\.__      _.-\ ")
    print("          /~\,      __       /~    \   ./    \ ")
    print("        ,/  /_\   _/  \    ,/~,_.~''\ /_\_  /'\ ")
    print("       / \ /## \ / V#\/\  /~8#  # ## V8  #\/8 8\ ")
    print("     /~#'#'#''##V&#&# ##\/88#'#8# #' #\#&'##' ##\ ")
    print("    j# ##### #'#\&&'####/###&  #'#&## #&' #'#&#'#'\ ")
    print("   /#'#'#####'###'\&##'/&#'####'### # #&#&##'#'### \ ")
    print("  J#'###'#'#'#'####'\# #'##'#'##'#'#####&'## '#'&'##|\ ")

    key = secrets.token_bytes(BLOCK_SIZE)

    print("The spirit of the glacier gifts you a flag!\n")
    nonce, ct, tag = create_message_and_mac(key, FLAG)
    print(f"{nonce.hex()}, {ct.hex()}, {tag.hex()}")

    print("\nNow you can bring forth your own messages to be blessed by the spirit of the glacier!\n")
    for i in range(8):
        print(f"Offer your message:")
        user_msg = input()
        try:
            msg = bytes.fromhex(user_msg)
        except:
            print(
                "The spirit of the glacier is displeased with the format of your message.")
            exit(0)
        nonce, ct, tag = create_message_and_mac(key, msg)
        print("The spirit of the glacier has blessed your message!\n")
        print(f"{nonce.hex()}, {ct.hex()}, {tag.hex()}")

    print("The spirit of the glacier has left you. You are alone once more.")
```

ASCON という軽量の暗号があるらしい。しかも新しい。これ自体は破れなさそう。

```cardlink
url: https://pypi.org/project/ascon/
title: "ascon"
description: "Lightweight authenticated encryption and hashing"
host: pypi.org
favicon: https://pypi.org/static/images/favicon.35549fe8.ico
image: https://pypi.org/static/images/twitter.abaf4b19.webp
```

接続すると、まず `key` が生成される。もちろん、予測不可である。

```python
key = secrets.token_bytes(BLOCK_SIZE)
```

この `key` を使ってフラグが暗号化される。

```python
nonce, ct, tag = create_message_and_mac(key, FLAG)
print(f"{nonce.hex()}, {ct.hex()}, {tag.hex()}")
```

その後、任意の平文で 8 回同じ処理ができる。`key` は同じである。

```python
for i in range(8):
	print(f"Offer your message:")
	user_msg = input()
	try:
		msg = bytes.fromhex(user_msg)
	except:
		print(
			"The spirit of the glacier is displeased with the format of your message.")
		exit(0)
	nonce, ct, tag = create_message_and_mac(key, msg)
	print("The spirit of the glacier has blessed your message!\n")
	print(f"{nonce.hex()}, {ct.hex()}, {tag.hex()}")
```

`create_message_and_mac` はパディングと暗号化と MAC 生成の 3 ステップである。

```python
def create_message_and_mac(key, message):
    padded_message = pad_message(message)
    nonce, ct = encrypt(key, padded_message)
    tag = mac_creation(key, padded_message)
    return nonce, ct, tag
```

`pad_message` は 16 の倍数になるように調節される。もし 16 の倍数ちょうどであれば、**何も追加されない**という特徴がある。

```python
def pad_message(message):
    first_block_pad = len(message) % BLOCK_SIZE
    first_block_pad = 16 if first_block_pad == 0 else first_block_pad
    return (first_block_pad.to_bytes() * (BLOCK_SIZE - first_block_pad)) + message
```

`encrypt` は、`key` と `nonce + (ctr+1)` で暗号化したものを平文ブロックと XOR したものを暗号文ブロックとしている。ECB モードだ。

```python
def encrypt(key,  message):
    assert len(message) % BLOCK_SIZE == 0
    message_blocks = split_blocks(message)

    assert len(message_blocks) < BLOCK_SIZE
    nonce = secrets.token_bytes(BLOCK_SIZE-1)

    cts = []
    for ctr, message in enumerate(message_blocks):
        cipher_input = nonce + (ctr+1).to_bytes(1, 'little')

        enc = ascon.encrypt(key, cipher_input, b'', b'')

        ct = xor(message, enc)
        cts.append(ct)
    return nonce, b''.join(cts)
```

`mac_creation` は、平文と前結果の XOR を暗号化するという処理を繰り返している。

```python
def mac_creation(key, message):
    assert len(message) % BLOCK_SIZE == 0
    message_blocks = split_blocks(message)
    enc_out = b"\x00" * BLOCK_SIZE
    for message in message_blocks:
        chaining_values = xor(message, enc_out)
        enc_out = ascon.encrypt(key, chaining_values, b'', b'')
    assert len(enc_out) == BLOCK_SIZE
    return enc_out
```

解法としては、パディングの脆弱性を利用すると、1 ブロックだけを送って `mac_creation` で任意の暗号文ブロックを生成することができる。あとは、それを XOR すれば OK。

![](https://storage.googleapis.com/zenn-user-upload/ae4279347619-20231127.png)

```python
from pwn import remote

BLOCK_SIZE = 16
HOST = "chall.glacierctf.com"
PORT = 13379


def xor(a, b):
    return bytes([x ^ y for x, y in zip(a, b)])


def split_blocks(message):
    return [message[i:i + BLOCK_SIZE] for i in range(0, len(message), BLOCK_SIZE)]


def get_params(conn):
    data = conn.recvline().strip().decode()
    nonce, ct, tag = map(bytes.fromhex, data.replace(",", "").split())
    return nonce, ct, tag


conn = remote(HOST, PORT)
conn.recvuntil(b"The spirit of the glacier gifts you a flag!\n\n")
nonce, ct, _ = get_params(conn)

MSG_BLOCK_SIZE = len(ct) // BLOCK_SIZE
encs = []

for i in range(MSG_BLOCK_SIZE):
    m = nonce + (i+1).to_bytes(1, "little")
    conn.recvuntil(b"Offer your message:")
    conn.sendline(m.hex().encode())
    conn.recvuntil(b"The spirit of the glacier has blessed your message!\n\n")
    _, _, tag = get_params(conn)
    encs.append(tag)

ct_blocks = split_blocks(ct)
flag = []
for ct_block, enc in zip(ct_blocks, encs):
    flag.append(xor(ct_block, enc))
flag = b"".join(flag)
print(flag)

```

```
[+] Opening connection to chall.glacierctf.com on port 13379: Done
b'\x0fgctf{CTr_M0d3_cbc_M4C_ASCON_DeF3AT$_TH3_$p1rIT}'
[*] Closed connection to chall.glacierctf.com port 13379
```
