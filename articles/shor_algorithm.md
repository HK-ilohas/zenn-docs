---
title: "Shorのアルゴリズムを用いた素因数分解のメモ"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["量子コンピュータ", "Quantum"]
published: true
---

## 1．はじめに

Shor のアルゴリズムは，RSA 暗号や楕円曲線暗号などといった離散対数問題に基づいている暗号を多項式時間で解読できる量子アルゴリズムとしてよく知られています．この記事では，Shor のアルゴリズムを用いた素因数分解について解説します^[勉強中の学生が書いた内容なので，間違っているかもしれません．]．

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
    \frac{1}{\sqrt{r}} \sum_{s=0}^{r-1} \ket{u_s} = \ket{1}
$$

この式を示すためには $k \ne 0$ のとき，

$$
    \sum_{s=0}^{r-1} e^{-2 \pi i sk/r} = 0
$$

が成り立つことを用います^[$\left(e^{-2 \pi i sk/r}\right)^r=1$ より，$e^{-2 \pi i sk/r}$ は複素数平面上における正 $r$ 角形の頂点になります．そのため，この式は正 $r$ 角形の重心ということになり，対称性から $0$ が出てきます．もし，$0$ でなかったら，重心が中心からズレてしまいます．式の形は少し異なりますが，[1 の n 乗根の導出と複素数平面](https://manabitimes.jp/math/1045) と同じ話です．]．

$$
\begin{align*}
    \frac{1}{\sqrt{r}} \sum_{s=0}^{r-1} \ket{u_s} &= \frac{1}{r} \sum_{s=0}^{r-1} \sum_{k=0}^{r-1} e^{-2 \pi i sk/r} \ket{a^k \bmod{N}} \\
    &= \frac{1}{r} \left(\sum_{s=0}^{r-1}\ket{1} + \sum_{s=0}^{r-1} \sum_{k=1}^{r-1} e^{-2 \pi i sk/r} \ket{a^k \bmod{N}} \right) \\
    &= \ket{1}
\end{align*}
$$

この性質から，$\ket{1}$ で量子位相推定をすると，位相 $s/r$ を計算できます（ただし，$s$ は $0 \leq s \leq r-1$ のランダムな整数）．そして，$s/r$ を連分数展開すると $r$ が求まります．$s$ によっては失敗するので，繰り返す必要があるかもしれません．

![](https://storage.googleapis.com/zenn-user-upload/e446c18953da-20211204.png)

### 2.3．冪剰余演算の構成方法

$U$ についての量子位相推定をするためには，$U^n \ket{x} = \ket{a^n x \bmod{N}}$ という冪剰余演算を行う制御ユニタリゲートを構成する必要があり，量子位数発見において最も重い演算です．構成方法は様々ですが，大雑把な流れは次の通りです．

1. $\ket{a+b}$ のような加算回路を作る．
2. $\ket{(a+b) \bmod{N}}$ のような制御-剰余加算回路を作る．
3. $\ket{ax \bmod{N}}$ のような制御-剰余乗算回路を作る．
4. $\ket{a^n x \bmod{N}}$ のような制御-冪剰余回路を作る．

個人的にわかり易い構成方法の例を以下に挙げておきます．

- [V. Vedral, A. Barenco and A. Ekert, "Quantum Networks for Elementary Arithmetic Operations", arXiv:quant-ph/9511018, 1995.](https://arxiv.org/pdf/quant-ph/9511018.pdf)
- [S. Beauregard, "Circuit for Shor's algorithm using 2n+3 qubits", arXiv:quant-ph/0205095, 2003.](https://arxiv.org/pdf/quant-ph/0205095.pdf)

"Quantum Networks for Elementary Arithmetic Operations" は古典的な回路を移植したようなものです．他の方の記事ですが，[量子アルゴリズムの基本：算術演算の確認（べき剰余）](https://qiita.com/SamN/items/cc44c2fb26633567ab5f)で丁寧に解説されています．

"Circuit for Shor's algorithm using 2n+3 qubits" は QFT ベースのもので，量子位相推定の逆量子フーリエ変換を半古典的にすることで量子ビットの数を削減しています．

## 3．素因数分解

さきほどの量子位数発見を用いて，合成数 $N$ の因数を求めます^[あくまでも，これは因数を求めるアルゴリズムであって，素因数分解をするアルゴリズムではありません．RSA 暗号で使われるような $N=pq$ という形なら，因数を求めることと素因数分解をすることは同等ですが，$N=pqr$ のような形だと素数が $1$ 個求まるだけです．]．

### 3.1．アルゴリズム

1. $x \in \{2, 3, \dots, N-1\}$ をランダムに選びます．
2. $\gcd(x,N)$ を計算します．$\gcd(x, N) \ne 1$ なら，$\gcd(x, N)$ を出力して終了します．
3. $x^r \equiv 1 \pmod{N}$ となる位数 $r$ を量子位数発見で求めます．
4. $r$ が偶数なら，$x^{r/2} + 1 \not\equiv 0 \pmod{N}$ を確認し，$\gcd(x^{r/2} \pm 1, N)$ を因数として出力して終了します．失敗したら，ステップ 1 に戻ります．

#### 3.1.1．ステップ 1, 2 について

$x=0, 1$ は何の情報も与えないので除きます．もし，$\gcd(x,N) \ne 1$ なら，$N$ の因数が $\gcd(x, N)$ ということになります．

#### 3.1.2．ステップ 3, 4 について

位数 $r$ が偶数のとき，$x^r \equiv 1 \pmod{N}$ を因数分解すると，

$$
(x^{r/2} + 1)(x^{r/2} - 1) \equiv 0 \pmod{N}
$$

が成り立ちます．また，位数の定義より，$x^{r/2} - 1 \not\equiv 0 \pmod{N}$ です^[位数 $r$ は $x^r \equiv 1 \pmod{N}$ が成り立つ最小の正の整数なので，$r$ よりも小さい $r/2$ では成り立ちません．]．

さらに，$x^{r/2} + 1 \not \equiv 0 \pmod{N}$ なら，$x^{r/2} \pm 1$ のそれぞれに $N$ の因数が含まれているということになります．そのため，$N$ との $\gcd$ を計算すると因数を取り出せます．

ちなみに，位数 $r$ が偶数でかつ $x^{r/2} + 1 \not \equiv 0 \pmod{N}$ となる確率は，少なくとも $1/2$ あることが知られています．

### 3.2．計算量

$N=2^n$ として，通常の Shor のアルゴリズムの計算量は $O(n^3 \log n)$ です．さらに，高速乗算を用いると計算量が $O(n^2 (\log n)^2 \log \log n)$ になります．

既存の古典コンピュータの素因数分解アルゴリズムでは，一般数体篩法が最速です．一般数体篩法の計算量は $O(e^{(c+o(1))n^{1/3}(\log n)^{2/3}})$ という式で評価され，$N$ の桁長の指数時間に近い準指数時間と呼ばれるものです．

このことから，既存の古典アルゴリズムと比べて，Shor のアルゴリズムは非常に効率的だということがわかります．

### 3.3．必要な量子ビット数と実行時間の概算

Shor のアルゴリズムを用いた 2048 ビットの RSA 暗号の解読にかかる時間については，様々な研究によって概算されています．文献によって仮定や実装方法が異なるので，目についたものを一通り載せておきます．

#### 量子ビット数：856 万個，28.63 時間

『米国科学・工学・医学アカデミーによる量子コンピュータの進歩と展望』（原著者：米国科学・工学・医学アカデミー, 訳者：西森秀稔）に載っていた数値です．

#### 量子ビット数：2000 万個（ノイズ付き），8 時間

[C. Gidney and M. Ekerå, "How to factor 2048 bit RSA integers in 8 hours using 20 million noisy qubits", Quantum, Vol. 5, p. 433, 2021.](https://arxiv.org/pdf/1905.09749.pdf)

様々な工夫がされていますが，一応 Shor のアルゴリズムとしてカウントしました．

#### 量子ビット数：13436 個，177 日

[É. Gouzien and N. Sangouard, "Factoring 2048-bit RSA Integers in 177 Days with 13436 Qubits and a Multimode Memory", Phys. Rev. Lett., Vol. 127, p. 140503, 2021.](https://arxiv.org/pdf/2103.06159.pdf)

実行時間が増える代わりに，量子ビット数を減らしているようです．これも一応 Shor のアルゴリズムとしてカウントしました．
