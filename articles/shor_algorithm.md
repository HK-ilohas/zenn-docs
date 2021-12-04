---
title: "Shorのアルゴリズムのメモ"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["量子コンピュータ", "Quantum"]
published: false
---

## 1．はじめに

Shor のアルゴリズムは，RSA 暗号や楕円曲線暗号などといった離散対数問題に基づいている暗号を多項式時間で解読できる量子アルゴリズムとしてよく知られています．この記事では，Shor のアルゴリズムを用いた素因数分解について解説します．

## 2．量子位数発見

具体的な方法は後で述べますが，素因数分解の問題は位数を求める問題に変換できます．そして，Shor のアルゴリズムでは量子コンピュータを使って位数を計算します．

### 2.1．位数とは

$a$, $N$ を互いに素な正の整数とします．このとき，

$$
a^r \equiv 1 \pmod{N}
$$

を満たす最小の正の整数 $r$ を $a$ の位数といいます．

### 2.2．アルゴリズム

まず，次のような変換 $U$ を考えます．

$$
   U \ket{x} = \ket{ax \bmod{N}}
$$

$\gcd(a, N)=1$ なら逆元 $a^{-1}$ が存在するので，$U$ はユニタリ変換です．

このとき，$0 \leq s \leq r-1$ を満たす整数 $s$ に対して，

$$
\ket{u_s} = \frac{1}{\sqrt{r}} \sum_{k=0}^{r-1} e^{-2 \pi i sk/r} \ket{a^k \bmod N}
$$

と定義される量子状態 $\ket{u_s}$ は $U$ の固有状態となり，固有値は $e^{2\pi i s /r}$ です．

$$
\begin{align*}
    U \ket{u_s} &= \frac{1}{\sqrt{r}} \sum_{k=0}^{r-1} e^{-2 \pi i sk/r} \ket{a^{k+1} \bmod{N}} \\
    &= \frac{1}{\sqrt{r}} \sum_{l=1}^{r} e^{-2 \pi i s(l-1)/r} \ket{a^{l} \bmod{N}} \\
    &= e^{2\pi i s/r} \frac{1}{\sqrt{r}} \sum_{l=1}^{r} e^{-2 \pi i sl/r} \ket{a^{l} \bmod{N}} \\
    &= e^{2\pi i s/r} \frac{1}{\sqrt{r}} \sum_{k=0}^{r-1} e^{-2 \pi i sk/r} \ket{a^{k} \bmod{N}} \\
    &= e^{2 \pi i s /r} \ket{u_s}
\end{align*}
$$

$\ket{u_s}$ で量子位相推定^[量子位相推定については以前に書いた [量子位相推定のメモ](https://zenn.dev/hk_ilohas/articles/cee39bacafba4d) をご覧ください．]をすると，位相 $s/r$ が計算できます．しかし，$\ket{u_s}$ を用意するためには位数 $r$ を知っている必要があります．

そこで，次のような性質を利用します．

$$
    \frac{1}{\sqrt{r}} \sum_{s=0}^{r-1} = \ket{1}
$$

この式を示すためには $k \ne 0$ のとき，

$$
    \sum_{s=0}^{r-1} e^{-2 \pi i sk/r} = 0
$$

が成り立つことを用います^[$\left(e^{-2 \pi i sk/r}\right)^r=1$ より，$e^{-2 \pi i sk/r}$ は複素数平面上における正 $r$ 角形の頂点になります．そのため，この式は正 $r$ 角形の重心ということになり，対称性から $0$ が出てきます．もし，$0$ でなかったら，重心が中心からズレてしまいます．式の形は少し異なりますが，[1 の n 乗根の導出と複素数平面](https://manabitimes.jp/math/1045) と同じ話です．]．

$$
\begin{align*}
    \frac{1}{\sqrt{r}} \sum_{s=0}^{r-1} &= \frac{1}{r} \sum_{s=0}^{r-1} \sum_{k=0}^{r-1} e^{-2 \pi i sk/r} \ket{a^k \bmod{N}} \\
    &= \frac{1}{r} \left(\sum_{s=0}^{r-1}\ket{1} + \sum_{s=0}^{r-1} \sum_{k=1}^{r-1} e^{-2 \pi i sk/r} \ket{a^k \bmod{N}} \right) \\
    &= \ket{1}
\end{align*}
$$

この性質から，$\ket{1}$ で量子位相推定をすると，位相 $s/r$ を計算できます（ただし，$s$ は $0 \leq s \leq r-1$ のランダムな整数）．そして，$s/r$ を連分数展開すると $r$ が求まります．

![](https://storage.googleapis.com/zenn-user-upload/e446c18953da-20211204.png)

### 2.3．冪剰余演算の構成方法

$U$ についての量子位相推定をするためには，$U^n \ket{x} = \ket{a^n x \bmod{N}}$ という冪剰余演算を行う制御ユニタリゲートを構成する必要があり，量子位数発見において最も重い演算です．構成方法は様々ですが，大雑把な流れは次の通りです．

1. $\ket{a+b}$ のような加算回路を作る．
2. $\ket{(a+b) \bmod{N}}$ のような制御-剰余加算回路を作る．
3. $\ket{ax \bmod{N}}$ のような制御-剰余加算回路を作る．
4. $\ket{a^n x \bmod{N}}$ のような制御-冪剰余回路を作る．

個人的にわかり易い構成方法の例を以下に挙げておきます．

- [V. Vedral, A. Barenco and A. Ekert, "Quantum Networks for Elementary Arithmetic Operations", arXiv:quant-ph/9511018, 1995.](https://arxiv.org/pdf/quant-ph/9511018.pdf)
- [S. Beauregard, "Circuit for Shor's algorithm using 2n+3 qubits", arXiv:quant-ph/0205095, 2003.](https://arxiv.org/pdf/quant-ph/0205095.pdf)

"Quantum Networks for Elementary Arithmetic Operations" は古典的な回路を移植したようなものです．他の方の記事ですが，[量子アルゴリズムの基本：算術演算の確認（べき剰余）](https://qiita.com/SamN/items/cc44c2fb26633567ab5f)で丁寧に解説されています．

"Circuit for Shor's algorithm using 2n+3 qubits" は QFT ベースのもので，半古典的な QFT を用いることで量子ビットの数を削減しています．

## 3．素因数分解
