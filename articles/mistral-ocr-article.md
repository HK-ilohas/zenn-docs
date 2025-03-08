---
title: "話題のMistral OCRを徹底活用！英語論文を格安で高精度に読み解く最新ワークフロー"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AI", "OCR", "Mistral", "Python"]
published: false
---

## はじめに

本記事では、Mistral AI が提供する最新の OCR 技術「**Mistral OCR**」について解説します。この技術を使って英語論文 PDF からテキストを抽出し、構造化された Markdown に変換、さらに HTML へと変換して日本語化する方法を紹介します。

Mistral OCR は学術論文に含まれるテキスト、数式、表、画像などを高精度で認識します。構造化されたデータとして出力できるため、論文の理解と活用が格段に効率化されます。

このワークフローを使えば、英語論文をより読みやすい形式で日本語閲覧できるようになり、研究活動や学習の効率向上に役立ちます。

## Mistral OCRとは

https://mistral.ai/en/news/mistral-ocr

**Mistral OCR** は、フランスの AI 企業 Mistral AI が 2025 年 3 月 6 日に発表した最新の OCR API です。単なるテキスト抽出ツールではなく、文書の構造、レイアウト、画像、表、数式など、複雑な要素を高精度で認識・抽出できる高度な OCR システムです。

### 公式例

![](/images/mistral-article/formaltest.png)

### Mistral OCR の主な特徴

- 複雑な文書要素（画像、数式、表など）の高精度な理解
- 多言語対応（数千の言語とフォントをサポート）
- ベンチマークテストでトップクラスの性能
- 高速処理（単一ノードで 1 分あたり最大 2000 ページを処理可能）
- 構造化された出力（Markdown/json 形式）
- Self ホスティングオプション（一部の組織向け）

### 性能比較

Mistral OCR は、他の主要 OCR モデルと比較して優れた性能を示しています。以下は公式に発表されたベンチマーク結果です。

| モデル | 総合 | 数式 | 多言語 | スキャン文書 | 表 |
| --- | --- | --- | --- | --- | --- |
| Google Document AI | 83.42 | 80.29 | 86.42 | 92.77 | 78.16 |
| Azure OCR | 89.52 | 85.72 | 87.52 | 94.65 | 89.52 |
| Gemini-1.5-Flash-002 | 90.23 | 89.11 | 86.76 | 94.87 | 90.48 |
| Gemini-1.5-Pro-002 | 89.92 | 88.48 | 86.33 | 96.15 | 89.71 |
| Gemini-2.0-Flash-001 | 88.69 | 84.18 | 85.80 | 95.11 | 91.46 |
| GPT-4o-2024-11-20 | 89.77 | 87.55 | 86.00 | 94.58 | 91.70 |
| **Mistral OCR 2503** | **94.89** | **94.29** | **89.55** | **98.96** | **96.12** |

![](/images/mistral-article/bou.jpeg)
![](/images/mistral-article/gantchart.jpeg)

多言語対応においても、Mistral OCR は他のサービスを上回るパフォーマンスを示しています。

| モデル | 生成におけるFuzzyマッチ |
| --- | --- |
| Google-Document-AI | 95.88 |
| Gemini-2.0-Flash-001 | 96.53 |
| Azure OCR | 97.31 |
| **Mistral OCR 2503** | **99.02** |

ただし、日本語については注意が必要です。以下のように Twitter（X）での報告では、日本語処理に関して精度が低いケースが見られています。

- https://x.com/ZFPhalanx/status/1897839969110311244
- https://x.com/jukenmax1975/status/1898223709808468399
- https://x.com/yakitori_eng/status/1897879134157914540

### 値段設定

- 基本料金：**1000 ページあたり 1 ドル**
- バッチ処理利用時：ページあたりのコストがおよそ半分に
- クラウドおよび推論パートナー経由でも近日提供予定
- 特定の組織向けにオンプレミスでの導入も可能

他の OCR サービスと比較すると、コストパフォーマンスに優れています。特に論文のような複雑な構造を持つ文書の処理においては、この価格設定は非常に魅力的です。

## Mistral OCRの使用方法

### Mistral APIの取得

https://mistral.ai/

1. Mistral AI のウェブサイトにアクセス
2. 「la Plateforme」でアカウントを作成
3. API キーを取得（Console 画面の `API > API Keys`）

検証するだけなら Free プランで十分です。

### ライブラリの Install

Python の場合は pip で導入できます。

```bash
$ pip install mistralai
```

### サンプルコード

ここでは、Mistral OCR を使って論文 PDF を処理する Python プログラムを紹介します。このコードは以下の 4 つの主要な機能を提供します。

- PDF 文書の OCR 処理: Mistral OCR API を使用して文書を解析
- 画像の抽出: 論文中の図表や画像を別ファイルとして保存
- マークダウンの生成: 文書構造を保持したマークダウン形式への変換
- 結果の整理: 処理結果を JSON 形式で保存し、参照しやすく整理

https://docs.mistral.ai/capabilities/document/

:::details Python プログラム

```python
import json
import base64
import re
from pathlib import Path
from mistralai import Mistral


def read_api_key(api_key_file):
    """APIキーをファイルから読み取る"""
    with open(api_key_file, "r") as f:
        api_key = f.read().strip()
        if not api_key:
            raise ValueError("API key is empty")
        return api_key


def process_ocr(pdf_path, api_key):
    """PDFのOCR処理を実行"""
    client = Mistral(api_key=api_key)

    with open(pdf_path, "rb") as f:
        file_content = f.read()

    uploaded_pdf = client.files.upload(
        file={"file_name": pdf_path.name, "content": file_content},
        purpose="ocr",
    )

    signed_url = client.files.get_signed_url(file_id=uploaded_pdf.id)

    ocr_response = client.ocr.process(
        model="mistral-ocr-latest",
        document={"type": "document_url", "document_url": signed_url.url},
        include_image_base64=True,
    )

    return ocr_response.model_dump()


def extract_images(ocr_result, images_dir):
    """OCR結果から画像を抽出して保存"""
    all_saved_images = {}

    for page in ocr_result["pages"]:
        page_index = page["index"]
        page_images = page.get("images", [])
        saved_images = {}

        for img in page_images:
            img_id = img["id"]
            img_base64 = img.get("image_base64", "")

            if not img_base64 or img_base64 == "..." or img_base64.endswith("..."):
                continue

            try:
                if img_base64.startswith("data:"):
                    base64_data = img_base64.split(",", 1)[1]
                else:
                    base64_data = img_base64

                img_data = base64.b64decode(base64_data)
                img_extension = ".jpg"

                if img_base64.startswith("data:image/"):
                    format_part = img_base64.split(";", 1)[0]
                    img_extension = "." + format_part.split("/", 1)[1]

                if img_id.lower().endswith((".jpg", ".jpeg", ".png", ".gif", ".bmp", ".webp")):
                    img_path = images_dir / f"{img_id}"
                else:
                    img_path = images_dir / f"{img_id}{img_extension}"

                with open(img_path, "wb") as img_file:
                    img_file.write(img_data)

                saved_images[img_id] = img_id
            except Exception as e:
                print(f"Failed to save image {img_id}: {e}")

        all_saved_images[page_index] = saved_images

    return all_saved_images


def fix_image_references(markdown_text, image_ids):
    """マークダウンの画像参照を修正"""
    for img_id in image_ids:
        correct_path = f"../images/{img_id}"

        # パターン1: 二重括弧パターン
        markdown_text = re.sub(
            r"!\[(.*?)\]\([^)]*" + re.escape(img_id) + r"[^)]*\)(?:\([^)]*" + re.escape(img_id) + r"[^)]*\))+",
            f"![{img_id}]({correct_path})",
            markdown_text,
        )

        # パターン2: 単一括弧パターン
        markdown_text = re.sub(
            r"!\[(.*?)\]\([^)]*" + re.escape(img_id) + r"[^)]*\)", f"![{img_id}]({correct_path})", markdown_text
        )

        # パターン3: 括弧なしパターン
        markdown_text = re.sub(r"!\[" + re.escape(img_id) + r"\](?!\()", f"![{img_id}]({correct_path})", markdown_text)

    return markdown_text


def save_markdown_files(ocr_result, markdown_dir, saved_images_dict):
    """マークダウンをページごとに保存"""
    # 個別ページの保存
    for page in ocr_result["pages"]:
        page_index = page["index"]
        markdown_text = page["markdown"]
        saved_images = saved_images_dict.get(page_index, {})
        image_ids = list(saved_images.keys())

        if image_ids:
            markdown_text = fix_image_references(markdown_text, image_ids)

        output_file = markdown_dir / f"page_{page_index}.md"
        with open(output_file, "w", encoding="utf-8") as f:
            f.write(markdown_text)

    # 全ページを結合したマークダウンの保存
    combined_file = markdown_dir / "combined.md"
    with open(combined_file, "w", encoding="utf-8") as f:
        all_image_ids = []
        for page_images in saved_images_dict.values():
            all_image_ids.extend(page_images.keys())

        unique_image_ids = list(set(all_image_ids))

        for page in sorted(ocr_result["pages"], key=lambda p: p["index"]):
            page_markdown = page["markdown"]

            if unique_image_ids:
                page_markdown = fix_image_references(page_markdown, unique_image_ids)

            f.write(page_markdown)
            f.write("\n\n")


def main():
    # 設定
    api_key_file = "mistral_api.txt"
    pdf_path = Path("paper.pdf")

    # ディレクトリ作成
    output_dir = Path("output")
    output_dir.mkdir(exist_ok=True)

    images_dir = output_dir / "images"
    images_dir.mkdir(exist_ok=True)

    markdown_dir = output_dir / "markdown"
    markdown_dir.mkdir(exist_ok=True)

    # 処理実行
    api_key = read_api_key(api_key_file)
    ocr_result = process_ocr(pdf_path, api_key)

    # 結果をJSONとして保存
    json_path = output_dir / f"{pdf_path.stem}.json"
    with open(json_path, "w", encoding="utf-8") as f:
        json.dump(ocr_result, f, ensure_ascii=False, indent=2)

    # 画像の抽出と保存
    saved_images_dict = extract_images(ocr_result, images_dir)

    # マークダウンの保存
    save_markdown_files(ocr_result, markdown_dir, saved_images_dict)

    print("Processing complete!")


if __name__ == "__main__":
    main()


```

:::

### 実行手順

このコードを実行する前に、以下の準備が必要です。

- `pip install mistralai` で必要なライブラリをインストール
- `mistral_api.txt` というファイルに Mistral AI API キーを保存
- `paper.pdf` という名前で処理したい PDF を用意（名前を変更する場合はプログラムも修正必要）

### 出力ファイル

実行すると以下のファイルが生成されます。

- `output/paper.json` - OCR 結果の完全な JSON データ
- `output/markdown/page_X.md` - ページごとの Markdown ファイル
- `output/markdown/combined.md` - 全ページを結合した Markdown ファイル
- `output/images/` - 抽出された画像ファイル

具体的にはこのようになります。例に Attention Is All You Need の PDF を使用しました。

https://arxiv.org/abs/1706.03762

```
.
├── mistral_api.txt
├── ocr.py
├── output
│   ├── images
│   ├── markdown
│   └── paper.json
└── paper.pdf
```

### OCR 結果例

結果を 1 ページだけ抜き出してみるとこのようになります。大体のレイアウト構造を維持しながら処理できていることがわかります。ただし、図のクリッピングが完全ではない部分もあります（左上の文字が切れている箇所など）。

![](/images/mistral-article/myocr.png)

公式サンプルほど完璧な結果を期待するのは現実的ではありませんが、論文の構造をほぼ維持したまま抽出できていることがわかります。特に数式や表の認識精度は他の OCR サービスと比較して非常に高いです。

## 応用例‐Markdownを変換して日本語HTMLを作成する

次に、OCR で生成された Markdown を HTML 化し、さらに日本語に翻訳する方法を紹介します。この部分は非常に荒削りなものですが、基本的なワークフローは理解できるでしょう。

以下のスクリプトは Markdown を HTML に変換するものです（※ 大部分は Claude AI によって生成されています）。

https://gist.github.com/HK-ilohas/db296b819a8a40caaacd6f9abbfa12e6

生成された HTML を Gemini（gemini-2.0-flash-thinking-exp-01-21）に投げて日本語に翻訳させました。使用したプロンプトは以下のシンプルなものです。

```
HTMLの文章部分を全て自然な日本語に翻訳したHTMLをください。
ただし、これは論文であることに注意してください。
また、著者名や著作名については英語のままでお願いします。
```

応答が途切れたときは「HTML で続けて」と入力して続きを取得しました。

![](/images/mistral-article/md_to_html.png)
![](/images/mistral-article/trans_html.png)

効率化のためには、生成 AI の API を使って自動化できますが、コスト面を考慮して今回は手動で行いました。将来的には全体のワークフローを自動化することで、さらに効率的に英語論文を日本語化できるでしょう。

### メリット

このアプローチには以下のメリットがあります。

1. 高品質なテキスト抽出: Mistral OCR は高い精度で論文の構造・数式・画像を認識
2. 構造の保持: 論文の章立て、引用、表などの構造が保持される
3. 数式のサポート: 数学的表現が MathJax などで適切に表示可能
4. 多言語対応: 様々な言語で書かれた論文にも対応
5. カスタマイズ性: 必要に応じて出力形式や翻訳方法を調整可能
6. 自動化可能: 大量の論文を一括処理することも可能
7. 低コスト: 従来の翻訳サービスと比較して非常に安価

### デメリット

一方で、いくつかの課題も存在します。

1. コスト: Mistral OCR と生成 AI API の両方の利用料金が発生（ただし従来手法より安価な可能性大）
2. 特殊な記号や表現: 専門分野特有の表記が正確に翻訳されない場合がある
3. API の制限: 各サービスのレート制限に注意が必要
4. 画像の扱い: 論文中の図表の位置が元の論文と変わる可能性がある
5. 翻訳品質: 高度に専門的な内容の翻訳は完璧ではない
6. 日本語対応: Mistral OCR 自体の日本語文書への対応はまだ発展途上

## まとめ

Mistral OCR は、従来の OCR サービスよりも高精度で論文のような複雑な構造を持つ文書を処理できる革新的なツールです。特に数式や表、画像の認識精度は他のサービスを上回っており、学術論文の処理に大きな可能性を示しています。

本記事で紹介したワークフローを使えば、以下のことが可能になります。

- 英語論文 PDF から高精度でテキストと構造を抽出
- Markdown 形式で構造化された文書を生成
- HTML に変換して視覚的に読みやすい形式に整形
- 生成 AI を活用して日本語化

このアプローチにより、研究者や学生はより効率的に英語論文を読解できるようになります。また、1000 ページあたり 1 ドルという低コストでの処理が可能になり、大量の論文処理も現実的なものとなりました。

今後の発展としては、以下のような方向性が考えられます。

- ワークフロー全体の自動化（PDF から HTML 変換、翻訳までをパイプライン化）
- 専門分野ごとの用語辞書の導入による翻訳精度の向上
- 共同研究のための注釈機能の追加
- 複数論文の横断的な分析機能の開発

Mistral OCR は比較的新しいサービスで、今後さらに機能が拡充していくことが期待されます。特に日本語対応が向上すれば、日本語論文の処理にも活用できるようになるでしょう。

## 参考文献

1. Mistral AI 公式ドキュメント - [OCR and Document Understanding](https://docs.mistral.ai/capabilities/document/)
2. Mistral AI 公式ブログ - [Mistral OCR](https://mistral.ai/news/mistral-ocr)
