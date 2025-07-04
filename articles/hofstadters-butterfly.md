---
title: "【量子フラクタル図形】ホフスタッターの蝶（Hofstadter's butterfly）を計算してみた【Python】"
emoji: "🦋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Python", "フラクタル", "量子力学"]
published: true
---

![](/images/hofstadters-butterfly/hofstadter.png)
*ホフスタッターの蝶（Hofstadter's butterfly）の計算結果*

## はじめに

**ホフスタッターの蝶**（Hofstadter's butterfly）とは、周期的ポテンシャルと 2 次元格子上の垂直な磁場下で運動する電子のスペクトル特性のグラフです。これは Douglas Hofstadter が 1976 年の論文で初めて示したもので、量子力学内で初めて発見された**フラクタル図形**になっています。フラクタル図形について調べていると、量子力学上でも存在することを知り、それで試しに計算してみた結果を記事化しました。

本記事では、ホフスタッターの蝶を**簡単に計算してみることだけを目的としています**。厳密に理解しようとすると相当な前提知識が求められますし、正直なところ私自身も理解できていない部分が多々あります。それでも、数式からグラフを描くという行為自体はそれほど難しくありません。そのため、この記事はあくまで入門的な試みに留まるものであり、厳密性や学術的な正確さを求める方の期待には応えられないかもしれないことを、あらかじめご承知おきください。

## モデル

まず、次のようなハミルトニアンを考えます。

$$
H = e^{ix} + e^{-ix} + e^{ip} + e^{-ip}
$$

これを2次元格子状に離散化します。色々と計算すると、次の Harper 方程式が得られます。

$$
\psi_{n+1} + \psi_{n-1} + 2 \cos (2 \pi \alpha n + k_j) \psi_n = E_n \psi_n
$$

- $n$ はサイト
- $E_n$ はサイト $n$ のエネルギー
- $\psi_n$ はサイト $n$ の波動関数
- $\alpha = a/b$ は有理数で既約分数
- $k_j \in [0, 2\pi)$ は波数で $k_j = 2 \pi j / b$ ($j = 0, \cdots b-1$) と分割される

次に、$\alpha$ の分母 $b$ ごとに $b \times b$ の Harper 行列

$$
H_{m,n}(k_j) = \delta_{m,n+1} + \delta_{m+1,n} + 2 \cos (2 \pi \alpha m + k_j) \delta_{m,n}
$$

を計算します。ここで、周期境界条件として $H_{0,b-1} = H_{b-1,0} = 1$ を設定します。

## 計算方法

ホフスタッターの蝶では、横軸に格子あたりの磁束 $\alpha$、縦軸にエネルギー $E_n$ をプロットします。量子力学ではハミルトニアン（この場合はHarper 行列）の固有値がエネルギーなので、すべての $(a, b, k_j)$ で $H$ の固有値を計算すればよいということになります。蝶の外形を見るだけであれば、$b$ の最大値は  40-60 程度で十分なので、今回は Python の `scipy` を用いて計算しました。結果は最初に載せた図と一致します。

```python
import numpy as np
import scipy.linalg as la
import matplotlib.pyplot as plt

# Hofstadterの蝶
points = []
max_b = 60
for b in range(1, max_b + 1):
    for a in range(0, b+1):
        if np.gcd(a, b) != 1:
            continue
        alpha = a / b

        for j in range(b):
            k = 2 * np.pi * j / b
            # 対角要素
            diag = 2 * np.cos(2 * np.pi * alpha * np.arange(b) + k)
            # Harper行列の構築
            H = np.diag(diag)
            H += np.diag(np.ones(b - 1), 1) + np.diag(np.ones(b - 1), -1)
            # 周期境界条件
            H[0, -1] = H[-1, 0] = 1
            # 固有値計算
            E = la.eigvalsh(H)
            for e in E:
                points.append((alpha, e))
alpha_vals, E_vals = zip(*points)

# プロット設定
plt.style.use('dark_background')
fig, ax = plt.subplots(figsize=(10, 8), dpi=400)

sc = ax.scatter(
    alpha_vals, E_vals,
    c=E_vals,
    cmap='plasma',
    s=0.8,
    alpha=0.6,
    linewidths=0
)

ax.set_title("Hofstadter's Butterfly", pad=12, fontsize=16, color='white')
ax.set_xlabel("Flux α", fontsize=14, color='white')
ax.set_ylabel("Energy E", fontsize=14, color='white')
ax.set_xlim(0, 1)
ax.set_ylim(-4, 4)

# 軸の色設定
ax.tick_params(colors='white', which='both')
for spine in ax.spines.values():
    spine.set_color('white')

# カラーバー
cbar = plt.colorbar(sc, ax=ax, shrink=0.8)
cbar.set_label('Energy E', color='white', fontsize=12)
cbar.ax.yaxis.set_tick_params(color='white')
plt.setp(plt.getp(cbar.ax.axes, 'yticklabels'), color='white')

plt.tight_layout()
plt.show()

```

## おわりに

本記事では、量子力学上のフラクタル図形「ホフスタッターの蝶」を計算しました。今回は細かいところはすべて省略して式だけを載せ、それに基づいて固有値を計算してプロットしました。

ホフスタッターの蝶は単にビジュアル的に美しいというだけのものではなく、2016 年にノーベル物理学賞を受賞したトポロジカル的な話とも関連するようです。また、理論的なものだけでなく、最近では実験的にホフスタッターの蝶を観測することにも成功しているようです。私は物性物理の人間ではないのでよくわかりませんが、理論的にも面白そうな雰囲気を感じました。

## 参考文献

- [Hofstadter's butterfly - Wikipedia](https://en.wikipedia.org/wiki/Hofstadter's_butterfly)
- [Yasuyuki Hatsuda et al, Hofstadter’s butterfly in quantum geometry, New J. Phys. 18 103023 (2016)](https://iopscience.iop.org/article/10.1088/1367-2630/18/10/103023)
- [Koki Shimura, Hofstadterの蝶 (2025)](https://lectarica.github.io/notes/hofstadter.pdf)
- [物質中のフラクタル Hofstadterの蝶 – 初貝・溝口・曽根 研究室：量子物性理論](https://patricia.ph.tsukuba.ac.jp/2021/09/10/hofstadter/)
