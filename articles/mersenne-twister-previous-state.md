---
title: "Mersenne twister (MT19937) で未来と過去の乱数列を予測してみる【Python】"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Python", "乱数", "CTF"]
published: false
---

## はじめに

**Mersenne twister** (**MT**, メルセンヌ・ツイスタ) は擬似乱数を生成するアルゴリズムの一種です。その中でも、**MT19937** という手法には次のような長所があります。

- 周期が $2^{19937}-1$ と非常に長い。
- 高次元（$623$ 次元）でも均等に分布する。
- 擬似乱数の生成が高速。

このような特徴から、様々なプログラミング言語の標準的な擬似乱数生成器に Mersenne twister が採用されています。たとえば、Python の `random` には MT19937 が採用されています。

しかし、Mersenne twister は**暗号論的擬似乱数**ではありません。暗号論的擬似乱数とは、暗号論的に安全な擬似乱数のことを指し、具体的には予測不可能であることが求められます。つまり、Mersenne twister は一定の条件下で予測可能な擬似乱数です。

この記事では、MT19937 によって出力された乱数列から、未来と過去の乱数列を予測します。より詳細には次の 2 ステップを取ります。

- 現在の内部状態を復元する。
- 過去の内部状態を復元する。

## MT19937

http://www.math.sci.hiroshima-u.ac.jp/m-mat/MT/MT2002/CODES/mt19937ar.c

数学的な背景はともかく、MT19937 のアルゴリズムを知るためには上記の C 言語実装を読むのが最速だと思います。ここでは、ソースコードを適宜抜粋しながら簡単に解説します^[Copyright (C) 1997 - 2002, Makoto Matsumoto and Takuji Nishimura, All rights reserved.]。

### ライブラリと定数

`stdio.h` を include していますが、これは `main` 関数でのサンプル用です。そのため、`mt19937ar.c` は単体で動作します。

```C
#include <stdio.h>
```

次に、乱数生成に必要な定数が定義されています。`N` は内部状態を表す状態ベクトルのサイズ、他は内部状態の更新で使われる定数です。

```C
#define N 624
#define M 397
#define MATRIX_A 0x9908b0dfUL   /* constant vector a */
#define UPPER_MASK 0x80000000UL /* most significant w-r bits */
#define LOWER_MASK 0x7fffffffUL /* least significant r bits */
```

### 状態ベクトル

内部状態を表す状態ベクトル `mt` は次のように定義されます。`mti` は次に生成される乱数を示す index です。`mti` が `N+1` の場合、内部状態が初期化されていない、つまり後述の `init_genrand` が一度も呼び出されていないことを示します。

```C
static unsigned long mt[N]; /* the array for the state vector  */
static int mti=N+1; /* mti==N+1 means mt[N] is not initialized */
```

### 内部状態の初期化関数

このプログラムで定義されている初期化の関数は以下の通りです。

- `init_genrand`
  シード値を受け取り、内部状態を初期化する。初期化後の `mti` は `N` になります。
- `init_by_array`
  シード値を配列とその長さとして与えて、内部状態を初期化する。

### 乱数生成関数

このプログラムで定義されている乱数生成の関数は以下の通りです。この記事では `genrand_int32` を解説します。

- `genrand_int32`
  32 ビットの符号なし整数を生成する。
- `genrand_int31`
  31 ビットの符号付き整数を生成する。
- `genrand_real1`
  $[0,1]$ の浮動小数点数を生成する。
- `genrand_real2`
  $[0,1)$ の浮動小数点数を生成する。
- `genrand_real3`
  $(0,1)$ の浮動小数点数を生成する。
- `genrand_res53`
  $[0,1)$ の浮動小数点数（53-bit resolution）を生成する。

### genrand_int32

`genrand_int32` は 32 ビットの符号なし整数を生成する関数です。

#### 変数の定義

`y` は後で生成される乱数を格納、`mag01` は内部状態を更新するために使います。

```C
unsigned long y;
static unsigned long mag01[2]={0x0UL, MATRIX_A};
/* mag01[x] = x * MATRIX_A  for x=0,1 */
```

#### 内部状態の更新

内部状態 `mt` は、初期化または `N` 個の 32 ビット整数が出力されて枯渇したときに更新されます。`init_genrand` が呼ばれていない場合（`mti` が `N+1`）には、デフォルトのシード値 `5489` を使って `init_genrand` を呼び出します。更新後、`mti` は `0` にリセットされます。

```C
if (mti >= N) { /* generate N words at one time */
    int kk;

    if (mti == N+1)   /* if init_genrand() has not been called, */
        init_genrand(5489UL); /* a default initial seed is used */

    for (kk=0;kk<N-M;kk++) {
        y = (mt[kk]&UPPER_MASK)|(mt[kk+1]&LOWER_MASK);
        mt[kk] = mt[kk+M] ^ (y >> 1) ^ mag01[y & 0x1UL];
    }
    for (;kk<N-1;kk++) {
        y = (mt[kk]&UPPER_MASK)|(mt[kk+1]&LOWER_MASK);
        mt[kk] = mt[kk+(M-N)] ^ (y >> 1) ^ mag01[y & 0x1UL];
    }
    y = (mt[N-1]&UPPER_MASK)|(mt[0]&LOWER_MASK);
    mt[N-1] = mt[M-1] ^ (y >> 1) ^ mag01[y & 0x1UL];

    mti = 0;
}
```

#### 乱数の生成

内部状態 `mt` から値を取り出し、`mti` を更新します。

```C
y = mt[mti++];
```

最後に、**Tempering** と呼ばれる操作が実行されます。これにより、生成された乱数が均等に分布するように調整されます。

```C
/* Tempering */
y ^= (y >> 11);
y ^= (y << 7) & 0x9d2c5680UL;
y ^= (y << 15) & 0xefc60000UL;
y ^= (y >> 18);

return y;
```

## 必要な乱数列の生成

この記事では Python の `random` を用いて、MT19937 の乱数予測を行います。必要な乱数の長さは、内部状態に対応して $32 \times 624 = 19968$ ビットです。ここでは、32 ビットずつ生成しています^[もし、乱数が 32 ビットずつではない場合は、下位から順にマスクして分割してください。]。

```python
import random

N = 624 # 状態ベクトルのサイズ
xs0 = [random.getrandbits(32) for _ in range(N)] # 検証用の乱数列（過去）
xs1 = [random.getrandbits(32) for _ in range(N)] # 復元に使用する乱数列
xs2 = [random.getrandbits(32) for _ in range(N)] # 検証用の乱数列（未来）
```

## 現在の内部状態の復元

https://inaz2.hatenablog.com/entry/2016/03/07/194147

現在の内部状態を復元するためには、各乱数に Tempering の逆操作をする必要があります。実装は上記のサイトから引用しています。

```python
def untemper(x):
    x = unBitshiftRightXor(x, 18)
    x = unBitshiftLeftXor(x, 15, 0xefc60000)
    x = unBitshiftLeftXor(x, 7, 0x9d2c5680)
    x = unBitshiftRightXor(x, 11)
    return x


def unBitshiftRightXor(x, shift):
    i = 1
    y = x
    while i * shift < 32:
        z = y >> shift
        y = x ^ z
        i += 1
    return y


def unBitshiftLeftXor(x, shift, mask):
    i = 1
    y = x
    while i * shift < 32:
        z = y << shift
        y = x ^ (z & mask)
        i += 1
    return y
```

これらの関数を用いて Tempering の逆操作をし、`random.setstate` で内部状態をセットします。

```python
mt_state = [untemper(x) for x in xs]
random.setstate((3, tuple(mt_state + [N]), None))
```

`random.setstate` の引数として与えられるタプル（またはリスト）の要素は次の通りです。[ドキュメント](https://docs.python.org/3/library/random.html)にほとんど記載されていないので、[CPython の実装](https://github.com/python/cpython/blob/3.11/Lib/random.py)を参考にしました。

- `version`
  バージョンごとに処理を分岐させる。`3` または `2`。通常使用では `3` でよさそう。
  > In version 2, the state was saved as signed ints, which causes inconsistencies between 32/64-bit systems. The state is really unsigned 32-bit ints, so we convert negative ints from version 2 to positive longs for version 3.
- `internalstate`
  内部状態。`mt19937ar.c` における `mt` と `mti` のタプル。
- `gauss_next`
  ガウス分布の計算で使われるパラメータ。通常使用では `None` でよさそう。$\mu + z \sigma$ の $z$ に対応する。

最後に、復元した内部状態から生成される乱数列が、検証用の乱数列（未来） `xs2` と一致するか検証します。結果は無事一致しました。ちなみに、`mti` を `0` にすると `predicted` は `xs1` と一致します。

```python
predicted = [random.getrandbits(32) for _ in range(N)]
print(predicted == xs2)
```

```
True
```

まとめるとプログラムは次の通りです。

:::details プログラム

```python
import random


def untemper(x):
    x = unBitshiftRightXor(x, 18)
    x = unBitshiftLeftXor(x, 15, 0xefc60000)
    x = unBitshiftLeftXor(x, 7, 0x9d2c5680)
    x = unBitshiftRightXor(x, 11)
    return x


def unBitshiftRightXor(x, shift):
    i = 1
    y = x
    while i * shift < 32:
        z = y >> shift
        y = x ^ z
        i += 1
    return y


def unBitshiftLeftXor(x, shift, mask):
    i = 1
    y = x
    while i * shift < 32:
        z = y << shift
        y = x ^ (z & mask)
        i += 1
    return y


N = 624 # 状態ベクトルのサイズ
xs0 = [random.getrandbits(32) for _ in range(N)] # 検証用の乱数列（過去）
xs1 = [random.getrandbits(32) for _ in range(N)] # 復元に使用する乱数列
xs2 = [random.getrandbits(32) for _ in range(N)] # 検証用の乱数列（未来）

mt_state = [untemper(x) for x in xs1]
random.setstate((3, tuple(mt_state + [N]), None))
# random.setstate((3, tuple(mt_state + [0]), None))

predicted = [random.getrandbits(32) for _ in range(N)]
print(predicted == xs2)
# print(predicted == xs1)
```

:::

## 過去の内部状態の復元

https://ctf.zeyu2001.com/2021/zh3ro-ctf-v2/twist-and-shout#recovering-the-previous-state

過去の内部状態を復元するためには、内部状態の更新の逆操作をする必要があります。実装は上記のサイトから引用しています。

```python
def get_prev_state(state):
    for i in range(623, -1, -1):
        result = 0
        tmp = state[i]
        tmp ^= state[(i + 397) % 624]
        if ((tmp & 0x80000000) == 0x80000000):
            tmp ^= 0x9908b0df
        result = (tmp << 1) & 0x80000000
        tmp = state[(i - 1 + 624) % 624]
        tmp ^= state[(i + 396) % 624]
        if ((tmp & 0x80000000) == 0x80000000):
            tmp ^= 0x9908b0df
            result |= 1
        result |= (tmp << 1) & 0x7fffffff
        state[i] = result
    return state
```

まずはさきほどの方法で現在の内部状態を復元します。その後、`get_prev_state` で内部状態を遡り、検証用の乱数列（過去） `xs0` と比較します。今回は `mti` を `0` にします。

```python
mt_state = [untemper(x) for x in xs1]
prev_mt_state = get_prev_state(mt_state)
random.setstate((3, tuple(prev_mt_state + [0]), None))

predicted = [random.getrandbits(32) for _ in range(N)]
print(predicted == xs0)
```

```
True
```

まとめるとプログラムは次の通りです。

:::details プログラム

```python
import random


def untemper(x):
    x = unBitshiftRightXor(x, 18)
    x = unBitshiftLeftXor(x, 15, 0xefc60000)
    x = unBitshiftLeftXor(x, 7, 0x9d2c5680)
    x = unBitshiftRightXor(x, 11)
    return x


def unBitshiftRightXor(x, shift):
    i = 1
    y = x
    while i * shift < 32:
        z = y >> shift
        y = x ^ z
        i += 1
    return y


def unBitshiftLeftXor(x, shift, mask):
    i = 1
    y = x
    while i * shift < 32:
        z = y << shift
        y = x ^ (z & mask)
        i += 1
    return y


def get_prev_state(state):
    for i in range(623, -1, -1):
        result = 0
        tmp = state[i]
        tmp ^= state[(i + 397) % 624]
        if ((tmp & 0x80000000) == 0x80000000):
            tmp ^= 0x9908b0df
        result = (tmp << 1) & 0x80000000
        tmp = state[(i - 1 + 624) % 624]
        tmp ^= state[(i + 396) % 624]
        if ((tmp & 0x80000000) == 0x80000000):
            tmp ^= 0x9908b0df
            result |= 1
        result |= (tmp << 1) & 0x7fffffff
        state[i] = result
    return state


N = 624 # 状態ベクトルのサイズ
xs0 = [random.getrandbits(32) for _ in range(N)] # 検証用の乱数列（過去）
xs1 = [random.getrandbits(32) for _ in range(N)] # 復元に使用する乱数列
xs2 = [random.getrandbits(32) for _ in range(N)] # 検証用の乱数列（未来）

mt_state = [untemper(x) for x in xs1]
prev_mt_state = get_prev_state(mt_state)
random.setstate((3, tuple(prev_mt_state + [0]), None))

predicted = [random.getrandbits(32) for _ in range(N)]
print(predicted == xs0)
```

:::

## おわりに

この記事では、Mersenne twister の MT19937 について説明し、乱数の出力結果から内部状態を復元し、更新前の内部状態に遡る方法を示しました。数値シミュレーションのように予測不可能性を必要としない場合には、Mersenne twister のような擬似乱数は有用でしょう。しかし、暗号のように予測不可能性を必要とする場合には不適切で、代わりに暗号論的擬似乱数を使用すべきです。実際、この記事で示したように、条件が揃うと簡単に予測できます。

## 参考

- [メルセンヌ・ツイスタ（Wikipedia）](https://ja.wikipedia.org/wiki/%E3%83%A1%E3%83%AB%E3%82%BB%E3%83%B3%E3%83%8C%E3%83%BB%E3%83%84%E3%82%A4%E3%82%B9%E3%82%BF)
- [暗号論的擬似乱数生成器（Wikipedia）](https://ja.wikipedia.org/wiki/%E6%9A%97%E5%8F%B7%E8%AB%96%E7%9A%84%E6%93%AC%E4%BC%BC%E4%B9%B1%E6%95%B0%E7%94%9F%E6%88%90%E5%99%A8)
- [mt19937ar.c](http://www.math.sci.hiroshima-u.ac.jp/m-mat/MT/MT2002/CODES/mt19937ar.c)
- [Mersenne Twister の出力を推測してみる - ももいろテクノロジー](https://inaz2.hatenablog.com/entry/2016/03/07/194147)
- [random - Generate pseudo-random numbers](https://docs.python.org/3/library/random.html)
- [Lib/random.py](https://github.com/python/cpython/blob/3.11/Lib/random.py)
- [Twist and Shout - Zeyu's CTF Writeups](https://ctf.zeyu2001.com/2021/zh3ro-ctf-v2/twist-and-shout#recovering-the-previous-state)
