---
title: "【Obsidian】Pandocが画像のパスを正しく認識しない問題について"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["obsidian", "pandoc"]
published: true
---

## Pandoc Plugin とは

https://github.com/OliverBalfour/obsidian-pandoc

[pandoc](https://pandoc.org/) とはドキュメント変換ツールである。公式ページ上で「a universal document converter」と呼んでいるだけあって対応フォーマットは豊富であり、Markdown や HTML、LaTeX、Word、EPUB などがある。

pandoc のインストール方法は下記の公式ページに載っている。

https://pandoc.org/installing.html

Obsidian の Pandoc Plugin は pandoc を使って変換するプラグインである。もちろん、使用するためには pandoc をインストールしている必要がある。

![](https://storage.googleapis.com/zenn-user-upload/3442b6b6f67c-20231128.png)

## 画像パスを認識しない問題

https://github.com/OliverBalfour/obsidian-pandoc/issues/198

執筆当時（2023-11-28）の Pandoc Plugin には画像パスを認識しない問題がある。例えば、`vault/attachments/` にある画像 `image.png` をノートにリンクする場合、大抵は次のようになるだろう。

```
[[image.png]]
```

しかし、この状態で Pandoc Plugin を使って export すると、`valut/image.png` は存在しないというエラーを出力する。つまり、`valut` 直下しか対応していない。

## 暫定的な解決方法

Pandoc Plugin に添付ファイルのパスを指定する方法があればよいのだが、現状だとそういった機能は提供されていない。今回の問題に関連していそうな Pull requests も出ているが、Pandoc Plugin の最終更新が Sep 26, 2022 なので望みは薄そうだ。

https://github.com/OliverBalfour/obsidian-pandoc/pull/128

なので、現状考えられる解決方法は次の通りである。

- `valut` 直下に画像を配置する
- リンクを `valut` からの相対パスにする

前者は issue の主が取っている方法だが、相当不便だろう（実際困ってそうな発言をしている）。今回オススメするのは後者の方法である。

さきほどの `valut/attachments/image.png` の例を使って説明する。エラーが出ていたのは `image.png` とだけ記載していたときなので、これを `valut` からの相対パスにすることでエラーを回避できる。具体的には次の通りである。

```
[[attachments/image.png]]
```

クリップボードからの paste だと毎回追記するのもめんどくさいが、現状だとこれ以外に方法はないと思われる。pandoc で変換したいなら、最初から変換できるようにノートを取るしかないだろう。
