---
title: "【Marp for VS Code】スライド作成の実践的なノウハウ"
emoji: "🦁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["markdown", "marp", "vscode"]
published: false
---

## はじめに

## Marp とは

https://marp.app/

[Marp](https://marp.app/) は、Markdown 記法と CSS を用いてプレゼンテーションスライドを作成するためのエコシステムです。Marp を使うためには、VS Code の拡張機能 [Marp for VS Code](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode) と [Marp CLI](https://github.com/marp-team/marp-cli/releases) の 2 つの方法が公式から提供されています。本記事では Marp for VS Code を使用します。

#todo スクショ

さらに、Marp は HTML、PDF、PowerPoint などの形式でエクスポートすることができます。そのため、Marp で作成したスライドはほとんどの環境で使用でき、資料の配布も容易です。

著者が考える Marp を使用するメリット・デメリットは以下の通りです。

### メリット

#### ① テキストベース (Markdown) で作成できる

テキストベースでスライドを作成できることから、PowerPoint や Keynote などと違って以下のことが容易にできます。

- git での管理
- Markdown の Linter が使える
- テキストベースの資料からのコピペ

#### ② VS Code で作成できる

VS Code の拡張機能として Marp を使用するため、PowerPoint や Keynote が無い環境でもスライドを作成できます。また、他の VS Code の機能も使えるため、スライド作成環境を自分好みにカスタマイズできます。

#### ③ 数式 (KaTeX) が使える

Web 用の TeX レンダラーである KaTeX に対応しているため、PowerPoint よりも数式を簡単に記述できます。ただし、数式番号には対応していません。

#todo スクショ

#### ④ コードブロックが使える

コードブロックが使えるので、プログラム多めのスライドだと便利です。さらに、Marp のコードブロックはオートスケールに対応しているので、雑に貼り付けられます。

#todo スクショ

#### ⑤ SVG を貼り付けられる

図に SVG が使えるので、高解像度の図や外部ツール（Draw.io など）を使用できます。この点については後で掘り下げます。

#### ⑥ サクッと作成できる

最低限のクオリティのスライドを短い時間で簡単に作成できます。特に使い捨てレベルのスライドであれば Marp で十分なことが多いです。

### デメリット

#### ① 凝ったクオリティのスライドを作成するのが難しい

PowerPoint と比べて、凝ったスライドを作ろうと思うと相当時間がかかります。また、**レイアウトの自由度が低い**ため、実際にスライドを作成してみると困ることが多々あります。

#### ② アニメーションや動画が使えない

Marp ではアニメーションや動画が使えないので、プレゼンテーションの内容によっては困ることがあります。一応、スライドの[遷移アニメーション](https://github.com/marp-team/marp-cli/blob/main/docs/bespoke-transitions/README.md)には対応しているようです。

#### ③ 図機能が弱い

図のキャプションや配置などが大変で、1 ページ 1 つが基本になります。この点については後で掘り下げます。
