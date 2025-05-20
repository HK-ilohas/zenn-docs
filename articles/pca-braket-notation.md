---
title: "ブラケット記法で極める主成分分析の直感的な統一視点"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["機械学習", "データ分析", "主成分分析", "統計学", "線形代数"]
published: true
---

## はじめに

**主成分分析**（Principal Component Analysis：PCA）とは、多次元空間上に分布するデータ集合を、誤差が最小となる方向へ直交射影することで、データの構造を低次元で簡潔に表現する手法である。

![](/images/pca-braket-notation/image.png)

本記事では、以下の観点から理論的に主成分分析を解説する。

1. **ブラケット記法の採用**
   量子力学で好んで用いられるブラケット記法 $\ket{x}, \bra{y}, \braket{x|y}$ を採用することで、射影や基底表示を直感的に表現する。
2. **「前処理の違い」として共分散行列・相関行列の差異を統一的に捉える**
   従来の解説では共分散行列・相関行列を別々に扱ったものが多い。しかし、本記事では元データに対する前処理の違い（中心化、標準化）として、これらを統一的に説明する。
3. 射影による近似誤差の最小化問題としての定式化
   データ $\ket{x_1}, \ket{x_2}, \dots, \ket{x_N} \in \mathbb{R}^D$ を単位ベクトル $\ket{v} \in \mathbb{R}^D$ の方向へ射影し、その誤差平均を最小化する問題として定式化する。すなわち、

    $$
    \operatorname*{minimize}_{\|\ket{v}\|=1} \frac{1}{N} \sum_{k=1}^N \| \ket{x_k} - \ket{v} \braket{v|x_k} \|^2
    $$
    を主成分軸の決定問題とみなす。
4. 二次形式の最大化問題
   上記の最小化問題は
   
    $$
    \operatorname*{maximize}_{\|\ket{v}\|=1} \bra{v} C \ket{v}, \quad C = \frac{1}{N} \sum_{k=1}^N \ket{x_k} \bra{x_k}
    $$
    という二次形式の最大化問題と同値で、演算子 $C$ の固有ベクトルが解となることを示す。


**想定読者**

- 線形代数（固有値やベクトル空間）の基礎を習得している方
- ブラケット記法に慣れている方（最後に解説を載せたので、未習者でも読み進められる）
- 主成分分析の理論的背景を体系的に深く理解したい方
- これまでの曖昧な解説で混乱した経験があり、明快な説明を求める方

## 主成分分析

$N$ 個の $D$ 次元の実数ベクトル $\ket{x_1}, \ket{x_2}, \dots, \ket{x_N} \in \mathbb{R}^D$ としてデータが与えられたとする。$\ket{v} \in \mathbb{R}^D$ をノルム 1 のベクトルとしたとき、

$$
\operatorname*{minimize}_{\|\ket{v}\|=1} \frac{1}{N} \sum_{k=1}^N \| \ket{x_k} - \ket{v} \braket{v|x_k} \|^2
$$

の最適解 $\ket{v^*}$ を、$N$ 個のデータを最も良く近似する方向として考える。$\|\ket{v}\|=1$ とするのは、この条件が無いと最適解が存在しないからである。$\braket{v|x_k}$ は $\ket{x_k}$ を $\ket{v}$ の方向へ射影したときの成分である。なお、目的関数は類似度を表しているものであれば他にも選び方は考えられるが、主成分分析ではこの誤差平均が用いられる。

目的関数を計算すると、

$$
\begin{align*}
\frac{1}{N} \sum_{k=1}^N \| \ket{x_k} - \ket{v} \braket{v|x_k} \|^2 = \frac{1}{N} \sum_{k=1}^N \left( \braket{x_k|x_k} - \braket{v|x_k} \braket{x_k|v} \right)
\end{align*}
$$

$\braket{x_k|x_k}$ は定数なので、この最適化問題は

$$
\begin{align*}
\operatorname*{maximize}_{\|\ket{v}\|=1} \frac{1}{N} \sum_{k=1}^N \braket{v|x_k} \braket{x_k|v} 
&= \operatorname*{maximize}_{\|\ket{v}\|=1} \bra{v} C \ket{v}, \\
C := \frac{1}{N} \sum_{k=1}^N \ket{x_k} \bra{x_k}
\end{align*}
$$

という二次形式の最大化問題になる。

### 二次形式の最大化問題

演算子 $C$ はエルミート演算子で半正定値なので、次のようにスペクトル分解できる。

$$
C = \sum_{n=1}^D \lambda_n \ket{\lambda_n} \bra{\lambda_n}
$$

ただし、

- $\lambda_1 \ge \lambda_2 \ge \cdots \ge \lambda_D \ge 0$ は演算子 $C$ の実固有値
- 固有値に対応する固有ベクトルを $\ket{\lambda_1}, \ket{\lambda_2}, \cdots, \ket{\lambda_D}$ 
- $\{ \ket{\lambda_n} \}_{n=1}^D$ は $\mathbb{R}^D$ の正規直交基底

とする。このとき、$\| \ket{v} \| = 1$ となる任意の $\ket{v} \in \text{span} \{\ket{\lambda_p}, \ket{\lambda_{p+1}}, \cdots, \ket{\lambda_D}\}$ は

$$
\bra{v} C \ket{v} \le \lambda_p
$$

を満たす。すなわち、$\ket{v^*} = \ket{\lambda_p}$ が $\text{span} \{\ket{\lambda_p}, \ket{\lambda_{p+1}}, \cdots, \ket{\lambda_D}\}$ 上の最適解となる。$\mathbb{R}^D$ ではなく $\ket{\lambda_p}$ 以降の基底で張られる空間にしている理由は、主成分分析では複数の最適な方向（主成分）が欲しいからである。

この結果から、固有値が大きい固有ベクトルほど元データを良く近似する方向になっていることがわかる。なぜなら、$p=1, 2, 3, \cdots$ と順番に

$$
\operatorname*{maximize}_{\|\ket{v}\|=1, \, \text{span} \{\ket{\lambda_p}, \ket{\lambda_{p+1}}, \cdots, \ket{\lambda_D}\}} \bra{v} C \ket{v}
$$

を解くと $\ket{\lambda_1}, \ket{\lambda_2}, \cdots$ となるからだ。

#### 証明

$C$ がエルミート演算子の証明：

$$
C^\dagger = \sum_{n=1}^D \lambda_n^* (\ket{\lambda_n} \bra{\lambda_n})^\dagger = \sum_{n=1}^D \lambda_n \ket{\lambda_n} \bra{\lambda_n} = C
$$

$C$ が半正定値の証明：任意の $\ket{v} \in \mathbb{R}^D$ について

$$
\bra{v} C \ket{v} = \frac{1}{N} \sum_{k=1}^N \braket{v|x_k} \braket{x_k|v} = \frac{1}{N} \sum_{k=1}^N |\braket{v|x_k}|^2 \ge 0
$$

$\bra{v} C \ket{v} \le \lambda_p$ の証明：$\| \ket{v} \| = 1$ となる任意の $\ket{v} \in \text{span} \{\ket{\lambda_p}, \ket{\lambda_{p+1}}, \cdots, \ket{\lambda_D}\}$ は

$$
\begin{align*}
&\ket{v} = c_p \ket{\lambda_p} + \cdots + c_D \ket{\lambda_D}, \\
&|c_p|^2 + \cdots + |c_D|^2 = 1
\end{align*}
$$

と基底展開の形で書ける。よって、

$$
\bra{v} C \ket{v} = |c_p|^2 \lambda_p + \cdots + |c_D|^2 \lambda_D \le \lambda_p (|c_p|^2 + \cdots + |c_D|^2) = \lambda_p
$$

### アルゴリズム

これまでの結果をまとめると、主成分分析のアルゴリズムは次のようになる。

1. 実データ $\ket{x_1}, \ket{x_2}, \dots, \ket{x_N} \in \mathbb{R}^D$ について、

    $$
    C = \frac{1}{N} \sum_{k=1}^N \ket{x_k} \bra{x_k}
    $$
2. 演算子 $C$ の固有値 $\lambda_n$ と固有ベクトル $\ket{\lambda_n}$ を求める。ただし、
    - $\lambda_1 \ge \lambda_2 \ge \cdots \ge \lambda_D \ge 0$ は演算子 $C$ の実固有値
	- 固有値に対応する固有ベクトルを $\ket{\lambda_1}, \ket{\lambda_2}, \cdots, \ket{\lambda_D}$ 
	- $\{ \ket{\lambda_n} \}_{n=1}^D$ は $\mathbb{R}^D$ の正規直交基底
3. 主成分の数を $K$ として、

    $$
    \ket{\lambda_1}, \ket{\lambda_2}, \cdots, \ket{\lambda_K}
    $$
    を主成分の軸とする。
4. 実データ $\ket{x_j}$ の第 $i$ 主成分のスコア $\alpha_{ij}$ は

    $$
    \alpha_{ij} = \braket{\lambda_i | x_j}
    $$
    となる。これをまとめるとスコア $A \in \mathbb{R}^{K \times N}$ が得られる。 

## データの前処理

従来の多くの解説では共分散行列と相関行列を別々に扱い、その違いを個別に説明しがちであるが、本記事では「**データの前処理**」という視点で両者を一括して扱う。これにより、共分散行列・相関行列のいずれを用いる場合でも、同一の手順で主成分分析を理解できる。

### 共分散行列

平均ベクトルを $\ket{\mu} = \frac{1}{N} \sum_{k=1}^N \ket{x_k}$ として、

$$
\ket{\tilde{x}_k} = \ket{x_k} - \ket{\mu}
$$

と**中心化**の変換をしたデータで主成分分析を行う。

このとき、$C=\Sigma$ として

$$
\Sigma = \frac{1}{N} \sum_{k=1}^N \ket{\tilde{x}_k} \bra{\tilde{x}_k} = \frac{1}{N} \sum_{k=1}^N (\ket{x_k} - \ket{\mu}) (\bra{x_k} - \bra{\mu})
$$

は**共分散行列**である。なお、分母は $N-1$ の不偏分散にすることもある。

### 相関行列

第 $i$ 成分（主成分ではない元の標準基底）の分散 $\sigma_i^2$ を計算する。

$$
\sigma_i^2 = \frac{1}{N} \sum_{k=1}^N (x_{ki} - \mu_i)^2
$$

- $x_{ki}$ は $\ket{x_k}$ の第 $i$ 成分
- $\mu_i$ は $\ket{\mu_i}$ の第 $i$ 成分
- 分母は $N-1$ の不偏分散にすることもある

$D = \text{diag} (\sigma_1^2, \sigma_2^2, \cdots, \sigma_D^2)$ とすると、$D^{-1/2} = \text{diag} (\sigma_1^{-1}, \sigma_2^{-1}, \cdots, \sigma_D^{-1})$ であり、

$$
\ket{z_k} = D^{-1/2} \ket{\tilde{x}_k} = D^{-1/2} (\ket{x_k} - \ket{\mu})
$$

と**標準化**の変換をしたデータで主成分分析を行う。

このとき、$C=R$ として

$$
R = \frac{1}{N} \sum_{k=1}^N \ket{z_k} \bra{z_k} = \frac{1}{N} \sum_{k=1}^N D^{-1/2} \ket{\tilde{x}_k} \bra{\tilde{x}_k} D^{-1/2}
$$

から $R = D^{-1/2} \Sigma D^{-1/2}$ となるので、$R$ は**相関行列**である。

## ブラケット記法のおさらい

**ブラケット記法（Dirac の記法）** は、もともと量子力学で使われる記法だが、線形代数の有限次元ベクトル空間にも自然に適用できる。

- **ケット** (Ket)：列ベクトルを表す $\ket{x}$
- **ブラ** (Bra)：ケット $\ket{x}$ の随伴（実数では転置）を表す $\bra{x}$

ブラケット記法を使うことで、射影や基底表示などをわかり易く表現できる。

### ケット

ケット $\ket{x}$ は列ベクトルを表す。ベクトル表示では、

$$
\ket{x} = \begin{bmatrix} x_1 \\ x_2 \\ \vdots \\ x_n \end{bmatrix} = \begin{bmatrix} x_1 & x_2 & \cdots & x_n \end{bmatrix}^T 
$$

と書ける。${}^T$ は転置である。

### ブラ

ブラ $\bra{x}$ はケット $\ket{x}$ の随伴、すなわち双対空間の要素である。ベクトル表示では、

$$
\bra{x} = \ket{x}^\dagger = \begin{bmatrix} x_1^* & x_2^* & \cdots & x_n^* \end{bmatrix} 
$$

と書ける。${}^*$ は共役である。なお、この記事では実数のみを扱うので、ブラはケットの転置、すなわち単なる行ベクトルとなる。

### 内積

$\braket{x|y}$ はブラ $\bra{x}$ とケット $\ket{y}$ の内積を表す。これは、線形代数での内積が $(\mathbf{x}, \mathbf{y}) = \mathbf{x}^\dagger \mathbf{y}$ であることに対応する。なお実数だと、$\braket{x|y} = \braket{y|x}$ である。

### 外積

$\ket{x}\bra{y}$ は**演算子**（行列）を表す。たとえば、$\ket{x}\bra{x}$ は $\ket{x}$ への射影演算子となる。

$$
(\ket{x}\bra{x}) \ket{y} = \braket{x|y} \cdot \ket{x} 
$$

### ノルム

ノルム $\| \ket{x} \|$ は自身の内積の平方根で定義される。

$$
\| \ket{x} \| = \sqrt{\braket{x|x}}
$$

### 基底展開

ベクトル空間の正規直交基底 $\{\ket{i}\}_i$ に対して、任意のケット $\ket{x}$ は

$$
\ket{x} = \sum_{i} c_i \ket{i}, \quad c_i = \braket{i|x}
$$

と展開できる。もし $\ket{x}$ がノルム 1 なら

$$
\sum_{i} |c_i|^2 = 1
$$

となることは重要である。

### スペクトル分解

演算子 $A$ がエルミートであるとは $A^\dagger = A$ を満たすことである。エルミート演算子 $A$ には必ず実数の固有値 $\{\lambda_i\}$ と、互いに直交する正規化された固有ベクトル $\{\ket{\lambda_i}\}$ が存在する。すなわち、**スペクトル分解（有限次元だと対角化）** が可能で、

$$
    A = \sum_{i} \lambda_i \ket{\lambda _i}\bra{\lambda _i}
$$

と書ける。正確な説明は線形代数や量子力学の本を参照すること。

## おわりに

本記事では、ブラケット記法を用い、主成分分析を射影誤差の最小化からスペクトル分解まで一貫して理論的に解説し、中心化・標準化という前処理の違いで分散・相関を統一的に扱った。これにより、主成分分析の本質を線形代数の枠組みで直感的に理解できるようになった。

## 参考文献

この記事は次の書籍を参考に作成した。この記事のオリジナル部分は、ブラケット記法の採用や前処理の枠組みである。現状絶版になっているので、電子版の販売が待ち遠しい一冊である。

- [佐藤一宏『線形代数を基礎とする応用数理入門 最適化理論・システム制御理論を中心に』](https://www.saiensu.co.jp/search/?isbn=978-4-7819-1586-9&y=2023)

主成分分析は、特異値分解の低ランク近似の枠組みとしても捉えることができる。しかも、数値計算の安定性を考えると特異値分解のほうが良いとされる。ただし、個人的には特異値分解のほうがわかりにくいので、今回は主成分分析に絞って解説した。
