---
title: "Meta AIの数式対応OCR『Nougat』を試してみた"
emoji: "😽"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AI", "機械学習", "OCR", "PDF", "Nougat"]
published: true
---

## Nougat とは

https://facebookresearch.github.io/nougat/

Nougat（Neural Optical Understanding for Academic Documents）は、Meta AI が開発した英語の論文解析に特化した OCR です。Nougat を使うと、論文 PDF を Mathpix Markdown と互換性がある形式（`.mmd`）に変換できます。

Nougat の最大の特徴は、数式に対応していることです。実際、Nougat の Example Pages には衝撃的な例が示されています。分数や、$\hbar$ や $\eta_{r,s}^{qQ}$ のような細かいところまである程度正確に読み取っています。

![](https://storage.googleapis.com/zenn-user-upload/99f8f4e222b3-20230907.png)

![](https://storage.googleapis.com/zenn-user-upload/7e7a8adfad4b-20230907.png)

この記事では、実際に Nougat でいくつか PDF を読み取り、Demo 以外の PDF だとどうなるか確かめていきます。

## Nougat の使い方

https://github.com/facebookresearch/nougat

### 動作環境

Nougat を使うためには高性能の GPU が必要です。CPU だけでも一応動作はしますが、非常に遅く大抵クラッシュします。そのため、GPU を積んだ計算機を有していない人には、[Google Colab](https://colab.research.google.com/) のようなサービスを推奨します。

手っ取り早く試したい人には、Huggingface Community Demo がおすすめです。

https://huggingface.co/spaces/ysharma/nougat

### インストール

Google Colab を使っている場合は、まず GPU を有効にしましょう。また、下記コマンドの行頭に `!` をつけてください。

`pip` でインストールできます。

```
$ pip install nougat-ocr
```

レポジトリからも可能です。

```
$ pip install git+https://github.com/facebookresearch/nougat
```

API や dataset の機能を使う場合には少し変わります。

```
$ pip install "nougat-ocr[api]"
または
$ pip install "nougat-ocr[dataset]"
```

### 使用方法

`nougat` の後に PDF のパスを入力するだけです。

```
$ nougat path/to/file.pdf
```

オプションは以下の通りです。たとえば、`--out .` をつけるとカレントディレクトリに結果が出力されます。

```
usage: nougat [-h] [--batchsize BATCHSIZE] [--checkpoint CHECKPOINT] [--out OUT] [--recompute] [--markdown] pdf [pdf ...]

positional arguments:
  pdf                   PDF(s) to process.

options:
  -h, --help            show this help message and exit
  --batchsize BATCHSIZE, -b BATCHSIZE
                        Batch size to use.
  --checkpoint CHECKPOINT, -c CHECKPOINT
                        Path to checkpoint directory.
  --out OUT, -o OUT     Output directory.
  --recompute           Recompute already computed PDF, discarding previous predictions.
  --markdown            Add postprocessing step for markdown compatibility.
```

[出力形式](https://github.com/Mathpix/mathpix-markdown-it)は Mathpix Markdown と互換性がある `.mmd` です。$\LaTeX$ の table といった、通常の Markdown (`.md`) では対応していない記法が使用されているため、`.md` では正確にプレビューができません。

`.mmd` は [Snip Notes](https://snip.mathpix.com/) や [VS Code のプラグイン](https://mathpix.com/docs/mathpix-markdown/how-to-mmd-vscode) などを使用することでプレビューできます。Snip Notes の場合、拡張子を `.md` にして読み込む必要があります。

## 検証

実際に Nougat で様々な PDF を変換してみました。左が Nougat による変換後の Markdown、右が元の PDF です。

### 最近の論文 PDF

#### ["Attention Is All You Need" (Vaswani et al., 2017)](https://arxiv.org/abs/1706.03762)

Google Colab (T4) での実行時間：1 分 57 秒
ページ数：15

全体を見ると、所々正しく認識されていませんが、十分な精度だと思います。特に、図のキャプションが本来の位置ではなく、 Figure 2 について言及している箇所に移動していることが驚きです。

![](https://storage.googleapis.com/zenn-user-upload/46b6aa22c7e7-20230907.png)

#### ["A Grand Unification of Quantum Algorithms" (Martyn et al., 2021)](https://arxiv.org/abs/2105.02859)

Google Colab (T4) での実行時間：5 分 38 秒
ページ数：39

二段組の PDF でも上手く認識できました。

![](https://storage.googleapis.com/zenn-user-upload/22ea35523b33-20230907.png)

### 古い論文 PDF

#### ["Simulating Physics with Computers" (Feynman, 1981)](https://s2.smu.edu/~mitch/class/5395/papers/feynman-quantum-1981.pdf)

Google Colab (T4) での実行時間：1 分 25 秒
ページ数：22

精度はやや落ちていますが、ある程度正確に認識できています。

![](https://storage.googleapis.com/zenn-user-upload/d34137bb5931-20230907.png)

ですが、p.10 はエラーになっていました。

![](https://storage.googleapis.com/zenn-user-upload/cc6a08238fec-20230907.png)

#### ["Riemann's hypothesis and tests for primality\*" (Miller, 1976)](https://www.sciencedirect.com/science/article/pii/S0022000076800438)

Google Colab (T4) での実行時間：1 分 47 秒
ページ数：18

最近の論文 PDF と比べると誤変換が目立ちますが、ページによっては上手くいっていました。

![](https://storage.googleapis.com/zenn-user-upload/9bdb21bc674c-20230907.png)

### 最近の書籍 PDF

全ページを変換すると非常に時間がかかるため、適当に 1-2 ページを切り出して変換しました。

#### ["The Elements of Statistical Learning Second Edition" (Hastie et al., 2009)](https://hastie.su.domains/ElemStatLearn/)

単純なレイアウトのページだと、ほとんど完璧に変換できています。

![](https://storage.googleapis.com/zenn-user-upload/0632d9b00677-20230907.png)

#### ["Pattern Recognition and Machine Learning" (Bishop, 2006)](https://www.microsoft.com/en-us/research/uploads/prod/2006/01/Bishop-Pattern-Recognition-and-Machine-Learning-2006.pdf)

同様にほとんど完璧に変換出来ています。

![](https://storage.googleapis.com/zenn-user-upload/0025a05c1b9a-20230907.png)

### 古い書籍 PDF

#### ["Mechanics" (Landau and Lifshitz, 1976)](https://archive.org/details/landau-and-lifshitz-physics-textbooks-series/Vol%201%20-%20Landau%2C%20Lifshitz%20-%20Mechanics%20%283rd%20ed%2C%201976%29/)

数式だけでなく英語のスペルすら怪しい箇所が多いです。なぜか大量にバーがついています。

![](https://storage.googleapis.com/zenn-user-upload/9b8f807e1764-20230907.png)

#### ["The Principles of Quantum Mechanics" (Dirac, 1930)](https://archive.org/details/principlesquantummechanics1stdirac1930/mode/1up)

ページによっては一切認識しませんでした。さきほどの "Mechanics" もページの切り出し方によっては同様のことが起きました。

![](https://storage.googleapis.com/zenn-user-upload/c9cec3d4def1-20230907.png)

## 所感

あくまで個人の所感なので、参考程度にしてください。

- 既存の OCR と比べるとかなりよさそう
- 最近の論文 PDF についてはかなり精度高い
- 古い論文 PDF だとスキャンの質と組版次第では悪くない
- 最近の書籍 PDF についても、レイアウトが単純なら十分な精度
- 古い書籍 PDF については難あり
- 認識不可なページが多々ある

SNS 上では昔の論文や技術文書への適用が期待されていましたが、そういった用途だと今の Nougat は非実用的でしょう。ですが、今後の OCR に期待させてくれる手法だと思います。

## 参考

- [Nougat](https://facebookresearch.github.io/nougat/)
- [facebookresearch/nougat - github](https://github.com/facebookresearch/nougat)
- [ysharma/nougat - Huggingface Community Demo](https://huggingface.co/spaces/ysharma/nougat)
- [Nougat: Neural Optical Understanding for Academic Documents](https://arxiv.org/abs/2308.13418)

## 関連記事

- [数式や文章がぐにゃぐにゃに曲がった論文 PDF でもくっきり認識する画期的な OCR『Nougat』― AIDB](https://aiboom.net/archives/54869)
- [数式を含むスキャン画像の PDF を OCR してマークダウン形式に変換できる。Nougat を試す ― はまち](https://note.com/hamachi_jp/n/n7f5f35b38768)
