---
title: "OpenFHEのPython wrapperで準同型暗号を試してみた"
emoji: "🌟"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["準同型暗号", "OpenFHE", "Python"]
published: true
---

## はじめに

[OpenFHE](https://www.openfhe.org/) とは、完全準同型暗号（Fully Homomorphic Encryption, FHE）のオープンソースライブラリです。FHE は暗号化されたデータを復号せずに加法や乗法、回転などの計算を実行できます。今回は [OpenFHE の Python wrapper](https://github.com/openfheorg/openfhe-python/tree/main) をインストールし、いくつかサンプルプログラムを動かしてみました。

## インストール

https://github.com/openfheorg/openfhe-python/tree/main

公式の README に従ってインストールします。OpenFHE の Python wrapper をビルドするためには、予め以下のものをインストールする必要があります。

- [OpenFHE](https://github.com/openfheorg/openfhe-development)
- [Python 3.6+](https://www.python.org/)
- [pybind11](https://pybind11.readthedocs.io/en/stable/installing.html)

### OpenFHE

https://github.com/openfheorg/openfhe-development

[ドキュメント](https://openfhe-development.readthedocs.io/en/latest/sphinx_rsts/intro/installation/installation.html)に従ってインストールします。少々時間がかかります。

```
$ git clone https://github.com/openfheorg/openfhe-development.git
$ cd openfhe-development
$ mkdir build
$ cd build
$ cmake ..
$ make
$ make install
```

### Python 3.6+

この記事では Python の導入方法については省略します。簡単に試すだけであれば [Google Colab](https://colab.research.google.com/) を使用するのもありでしょう。各種コマンドの文頭に `!` `%` を挿入すれば同じです。ちなみに動作確認済みです。

### pybind11

```
$ pip install "pybind11[global]"
```

### openfhe-python

ビルドが終わると、`build` の中に `*.so`（たとえば `openfhe.cpython-310-x86_64-linux-gnu.so`） があります。正常に OpenFHE をインポートできなかった場合は、このファイルを自分のプロジェクトにコピーしたり、システムパスに追記したりすることで改善できると思います。

```
$ git clone https://github.com/openfheorg/openfhe-python.git
$ cd openfhe-python
$ mkdir build
$ cd build
$ cmake ..
$ make
$ make install
```

## サンプルプログラム

### 整数演算（BGV）

このサンプルプログラムでは [BGV](https://eprint.iacr.org/2011/277) で整数の加算・乗算・回転を行っています。それぞれ、復号することなく暗号文のまま計算が行われ、その結果を復号すると確かに計算が出来ていることがわかります。

https://github.com/openfheorg/openfhe-python/blob/main/examples/pke/simple-integers-bgvrns.py

```
Plaintext #1: ( 1 2 3 4 5 6 7 8 9 10 11 12 ... )
Plaintext #2: ( 3 2 1 4 5 6 7 8 9 10 11 12 ... )
Plaintext #3: ( 1 2 5 2 5 6 7 8 9 10 11 12 ... )

Results of homomorphic computations
#1 + #2 + #3 = ( 5 6 9 10 15 18 21 24 27 30 33 36 ... )
#1 * #2 * #3 = ( 3 8 15 32 125 216 343 512 729 1000 1331 1728 ... )
Left rotation of #1 by 1 = ( 2 3 4 5 6 7 8 9 10 11 12 ... )
Left rotation of #1 by 2 = ( 3 4 5 6 7 8 9 10 11 12 ... )
Right rotation of #1 by 1 = ( 0 1 2 3 4 5 6 7 8 9 10 11 ... )
Right rotation of #1 by 2 = ( 0 0 1 2 3 4 5 6 7 8 9 10 ... )
```

### 整数演算（BFV）

このサンプルプログラムでは [BFV](https://eprint.iacr.org/2012/144) で整数の加算・乗算・回転を行っています。この場合、結果は BGV と同じになります。

https://github.com/openfheorg/openfhe-python/blob/main/examples/pke/simple-integers.py

```
Plaintext #1: ( 1 2 3 4 5 6 7 8 9 10 11 12 ... )
Plaintext #2: ( 3 2 1 4 5 6 7 8 9 10 11 12 ... )
Plaintext #3: ( 1 2 5 2 5 6 7 8 9 10 11 12 ... )

Results of homomorphic computations
#1 + #2 + #3 = ( 5 6 9 10 15 18 21 24 27 30 33 36 ... )
#1 * #2 * #3 = ( 3 8 15 32 125 216 343 512 729 1000 1331 1728 ... )
Left rotation of #1 by 1 = ( 2 3 4 5 6 7 8 9 10 11 12 ... )
Left rotation of #1 by 2 = ( 3 4 5 6 7 8 9 10 11 12 ... )
Right rotation of #1 by 1 = ( 0 1 2 3 4 5 6 7 8 9 10 11 ... )
Right rotation of #1 by 2 = ( 0 0 1 2 3 4 5 6 7 8 9 10 ... )
```

### 実数演算（CKKS）

このサンプルプログラムでは [CKKS](https://eprint.iacr.org/2016/421) で実数の加算・減算・スカラー倍・要素ごとの積・回転が行われています。出力は `x1` の復号結果、スカラー倍、要素ごとの積、回転のみとなっています。

https://github.com/openfheorg/openfhe-python/blob/main/examples/pke/simple-real-numbers.py

```
The CKKS scheme is using ring dimension: 16384
Input x1: (0.25, 0.5, 0.75, 1, 2, 3, 4, 5,  ... ); Estimated precision: 50 bits

Input x2: (5, 4, 3, 2, 1, 0.75, 0.5, 0.25,  ... ); Estimated precision: 50 bits

Results of homomorphic computations:
x1 = (0.25, 0.5, 0.75, 1, 2, 3, 4, 5,  ... ); Estimated precision: 43 bits

Estimated precision in bits: 43.0
4 * x1 = (1, 2, 3, 4, 8, 12, 16, 20,  ... ); Estimated precision: 42 bits

x1 * x2 = (1.25, 2, 2.25, 2, 2, 2.25, 2, 1.25,  ... ); Estimated precision: 42 bits

In rotations, very small outputs (~10^-10 here) correspond to 0's:
x1 rotated by 1 = (0.5, 0.75, 1, 2, 3, 4, 5, 0.25,  ... ); Estimated precision: 43 bits

x1 rotated by -2 = (4, 5, 0.25, 0.5, 0.75, 1, 2, 3,  ... ); Estimated precision: 43 bits
```

### ブール演算（FHEW/TFHE）

このサンプルプログラムではブール演算が行われています。

https://github.com/openfheorg/openfhe-python/blob/main/examples/binfhe/boolean.py

```
Generating the bootstrapping keys...

Result of encrypted computation of (1 AND 1) OR (1 AND (NOT 1)) = 1
```

## 最後に

この記事では、OpenFHE とその Python wrapper をインストールし、いくつかサンプルプログラムを動かしました。OpenFHE は C++ のライブラリなので、Python wrapper だとかなり記述が楽になるのでは？と期待しましたが、正直あまり差がないように感じました。現時点では、単に OpenFHE を使うだけであれば、ドキュメントやサンプルプログラムなどが豊富な C++ のほうがいい気がしました。

## 参考

### OpenFHE

- https://www.openfhe.org/
- https://github.com/openfheorg/openfhe-python/tree/main
- https://github.com/openfheorg/openfhe-development
- https://openfhe-development.readthedocs.io/en/latest/sphinx_rsts/intro/installation/installation.html

### Python

- https://www.python.org/
- https://pybind11.readthedocs.io/en/stable/installing.html
- https://colab.research.google.com/

### Scheme

- BGV: https://eprint.iacr.org/2011/277
- BFV: https://eprint.iacr.org/2012/144
- CKKS: https://eprint.iacr.org/2016/421
