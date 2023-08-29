---
title: "RGBとHSVの相互変換を手計算でする必要があったときのメモ"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["画像処理"]
published: true
---

## はじめに

この記事は、過去に友人から「試験対策のために RGB と HSV の計算方法を教えて」と言われて作成した資料がもとになっています。当初は適当な Web サイトの URL を送り付けてやり過ごそうかと思っていたのですが、スケールが授業と違っていたり、計算方法がわかりにくかったりなど、あまりピンと来るサイトを見つけられなかったので自分で書きました。しかし、試験ではこの問題が出なかったのでショックでした(ToT)

こんなありきたりな内容の記事を公開するかどうか悩みましたが、過去の自分や友人と同じように、どこかの高専・大学生の役に立ったらいいなという思いで公開することにしました。

## 値の範囲

資料によって範囲が異なるが、今回はこの範囲を使用する^[範囲が違う場合はどちらかにスケーリングすれば同じ]。

| R     | G     | B     |
| ----- | ----- | ----- |
| 0-255 | 0-255 | 0-255 |

- $H$ が 0-100
- $S,V$ が 0-100 や 0-255

| H     | S   | V   |
| ----- | --- | --- |
| 0-360 | 0-1 | 0-1 |

## RGB → HSV

$MAX = \text{RGB 中の最大値}$
$MIN = \text{RGB 中の最小値}$

### Hue（色相）

計算結果がマイナス値だったら、360 を加算する。

- $R$ が $MAX$ のとき

  $$
  H = \frac{G - B}{MAX - MIN} \times 60
  $$

- $G$ が $MAX$ のとき

  $$
  H = \frac{B - R}{MAX - MIN} \times 60 + 120
  $$

- $B$ が $MAX$ のとき

  $$
  H = \frac{R - G}{MAX - MIN} \times 60 + 240
  $$

- $R=G=B$ のとき

  $$
  H = 0
  $$

### Satulation（彩度）

$$
S = \frac{MAX - MIN}{MAX}
$$

### Value（明度）

$$
V = \frac{MAX}{255}
$$

## HSV → RGB

### 最大値・最小値

$$
\begin{align*}
MAX &= V \times 255\\
MIN &= MAX \times (1 - S)
\end{align*}
$$

### H が 0-60 の場合

$$
\begin{align*}
R &= MAX\\
G &= (H \div 60) \times (MAX - MIN) + MIN\\
B &= MIN
\end{align*}
$$

### H が 60-120 の場合

$$
\begin{align*}
R &= ((120 - H) \div 60) \times (MAX - MIN) + MIN\\
G &= MAX\\
B &= MIN
\end{align*}
$$

### H が 120-180 の場合

$$
\begin{align*}
R &= MIN\\
G &= MAX\\
B &= ((H - 120) \div 60) \times (MAX - MIN) + MIN
\end{align*}
$$

### H が 180-240 の場合

$$
\begin{align*}
R &= MIN\\
G &= ((240 - H) \div 60) \times (MAX - MIN) + MIN\\
B &= MAX
\end{align*}
$$

### H が 240-300 の場合

$$
\begin{align*}
R &= ((H - 240) \div 60) \times (MAX - MIN) + MIN\\
G &= MIN\\
B &= MAX
\end{align*}
$$

### H が 300-360 の場合

$$
\begin{align*}
R &= MAX\\
G &= MIN\\
B &= ((360 - H) \div 60) \times (MAX - MIN) + MIN
\end{align*}
$$

## 計算例

### ① RGB(100, 200, 50)

$$
\begin{align*}
H &= \frac{50 - 100}{200 - 50} \times 60 + 120 = 100\\
S &= \frac{200 - 50}{200} = 0.75\\
V &= \frac{200}{255} = 0.7843137254901961... \simeq 0.78
\end{align*}
$$

### ② HSV(50, 0.5, 0.9)

$$
\begin{align*}
MAX &= 0.9 \times 255 = 229.5 \simeq 230\\
MIN &= 230 \times (1 - 0.5) = 115\\
\end{align*}
$$

$$
\begin{align*}
R &= MAX = 230\\
G &= (50 \div 60) \times (MAX - MIN) + MIN = 210.8333... \simeq 211\\
B &= MIN = 115
\end{align*}
$$

## 参考

- https://www.peko-step.com/tool/hsvrgb.html
