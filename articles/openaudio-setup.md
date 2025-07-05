---
title: "OpenAudio（旧 Fish-Speech）をWSL2 + uvで動かす ― 高精度・高速Text-to-Speechモデルの導入手順"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["音声合成", "TTS", "AI", "WSL", "uv"]
published: true
---

## OpenAudio (旧 Fish-Speech) とは

https://fish.audio/ja/
https://github.com/fishaudio/fish-speech/tree/main

**OpenAudio** (**旧 Fish-Speech**) は Text-to-Speech (TTS) の最新モデルの一つです。その特徴として以下の点が公式に挙げられています。デモは前述の公式ページから確認できます。

- 高精度：Seed TTS Eval で WER 0.008、CER 0.004（S1）を達成
- 高速：RTX 4060 で約 1:5、RTX 4090 で約 1:15 のリアルタイム係数
- Zero-shot ＆ Few-shot TTS：10～30秒の音声サンプルから高品質な音声を合成可能
- 非音素依存：音素表記なしに任意のスクリプトで動作
- 感情やトーン、エフェクト（笑い超えや泣き声など）に対応
- 多言語/言語横断対応：**英語**、**日本語**、韓国語、中国語、フランス語、ドイツ語、アラビア語、スペイン語
- WebUI（Gradio）／GUI（PyQt6） による手軽なローカル推論環境

本記事では **OpenAudio-S1-mini** をローカルで動作させることを目標とします。ここでは以下の環境を想定しています。Linux であれば基本的に同じ手順で問題ありません。

- WSL2 (Ubuntu)
- [uv (Python)](https://docs.astral.sh/uv/)

また、著者の検証 PC は RTX 4060 を搭載しており、これでもギリギリだったため、RTX 4090 以上が望ましいと感じました。少なくとも推論に GPU メモリは 6 GB 程度ないと厳しいでしょう。手順が異なるため本記事では扱いませんが、どうしても動かしたいという方は量子化版があるのでそちらを利用しましょう。

https://huggingface.co/calcuis/openaudio-gguf

## ライセンスと注意点

GitHub にある fish-speech は **Apache License** ですが、すべてのモデルウェイトは **CC-BY-NC-SA-4.0 License** です。そのため、非商用の利用に限られます。

本記事は技術的な解説を目的としたものであり、音声クローン技術の使用を推奨・助長するものではありません。音声クローンの利用にあたっては、本人の許可なく音声を模倣・再利用することが**プライバシー権・肖像権・著作権などの侵害に該当する可能性がある**ことに十分ご留意ください。また、各国・各地域の法律や本技術の使用方法によっては、それが違法行為とみなされる場合があります。利用者ご自身の責任において、**法令・契約・モラルに従い、慎重にご判断ください**。本記事の内容によって生じたいかなる損害やトラブルについても、著者は一切の責任を負いかねます。

## 導入

### uv

uv が入っていない場合はインストールします。conda でも可能ですがオススメはしません。

```
$ curl -LsSf https://astral.sh/uv/install.sh | sh
```

### huggingface-cli

huggingface-cli が入っていない場合はインストールします。さらに、アカウントを持っていない場合はサインアップし、token を作成してください。

```
$ uv tool install huggingface_hub
$ huggingface-cli login
```

### ライブラリ

音声処理に使用するライブラリをインストールします。`build-essential` が入っていない場合は、あらかじめインストールしておいてください。

```
$ sudo apt install portaudio19-dev libsox-dev ffmpeg
```

Python 3.12 の開発ヘッダファイルのインストールします。

```
$ sudo apt install python3.12-dev portaudio19-dev
```

### fish-speech

fish-speech を GitHub からクローンしてきてセットアップします。

```
$ git clone https://github.com/fishaudio/fish-speech.git
$ cd ./fish-speech
$ uv sync --python 3.12
```

### openaudio-s1-mini

Hugging Face から`fishaudio/openaudio-s1-mini` のモデルをダウンロードします。使用前に同意する必要があります。

https://huggingface.co/fishaudio/openaudio-s1-mini

![](/images/openaudio-setup/1.png)
*左真ん中のフォームに入力して同意する*

```
$ huggingface-cli download fishaudio/openaudio-s1-mini --local-dir checkpoints/openaudio-s1-mini
```

## 推論

WebUI を起動します。一部ライブラリのエラーが出るが無視してください。

```
$ source .venv/bin/activate
$ python -m tools.run_webui \
      --llama-checkpoint-path "checkpoints/openaudio-s1-mini" \
      --decoder-checkpoint-path "checkpoints/openaudio-s1-mini/codec.pth" \
      --decoder-config-name modded_dac_vq
```

少し待つと起動して URL が表示されるので、それを開きます。

```
* Running on local URL:  http://127.0.0.1:7860
```

![](/images/openaudio-setup/2.png)
*WebUI の画面*

あとはテキストを入力したり設定を弄ったりすることで TTS できます。なお、著者の RTX 4060 環境では 113 単語の英文で 2 分程度生成に要しました。