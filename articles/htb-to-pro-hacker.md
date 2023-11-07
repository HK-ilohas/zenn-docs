---
title: "【Hack The Box】無課金学生が2ヶ月でPro Hackerになるまでの軌跡"
emoji: "🙌"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Security", "hackthebox", "htb"]
published: true
---

## はじめに

![](https://storage.googleapis.com/zenn-user-upload/d41afcbaac1a-20231107.png)

Profile: https://app.hackthebox.com/profile/1660611

Hack The Box (HTB) を始めてから 2 ヶ月で Pro Hacker に昇格できた経験をもとに、その道のりを振り返ります。括弧内の日数は、要した時間を示しています。

## 当初のスペック

- 情報系の高専生
- 英語以外の資格は特に持っていない
- セキュリティについては一部学んでいた
- Hack The Box については名前だけ知っていた
- CTF の Crypto を時折やっている
- 研究分野はセキュリティとはほぼ関係ない

## Hack The Box (HTB) とは

https://www.hackthebox.com/

Hack The Box (HTB) は、ゲームのようにペネトレーションテストをトレーニングできるオンラインプラットフォームです。脆弱なマシンが用意されており、実際に攻撃・侵入することで様々なスキルを学ぶことができます。HTB の問題は大きく 2 つのカテゴリに分かれます。

- Machine：脆弱なマシンを攻撃・侵入して、`user.txt` と `root.txt` を入手する問題
- Challenge：Jeopardy 形式の flag を入手する問題

さらに、これらの問題は 2 種類に分類されます。

- Active：加点対象の新しい問題。writeup^[問題の解法についての記事。HTB では Walkthrough とも呼ばれます。] の公開が禁止されています。
- Retired：加点対象外の古い問題。writeup の公開が許可されています。

加点対象の問題を解いてポイントを貯め、Rank を上げることができます。Rank 制度は下から順番に Noob, Script Kiddie, Hacker, Pro Hacker, Elite Hacker, Guru, Omniscient となっています。上位 2 つの Rank は特に次元が違うイメージです。

月 14 ドルの VIP 会員以上になると、100 台以上ある Retired の問題を解けるようになる特典が付いてきます。一応、無料会員でも最新の Retired 数問は解くことができます。僕は今のところ**無課金**です。

## HTB に登録する

9 月末、唐突に HTB をやりたくなったのでアカウントを作成しました。僕が Hack The Box を知った数年前の時点では、登録するために簡単な問題を解く必要がありましたが、今ではその仕組みが廃止されているそうです。

HTB に登録後、まずは **Hacker** になることを目標に設定しました。理由としては、以下のような記事を読んで Hacker が HTB における登竜門的な Rank だと感じたからです。
https://qiita.com/daihi_t/items/95d5adfc4b2cea1d4c5c
https://sanposhiho.com/posts/be-hacker-in-hackthebox/

## 初めて Active Machine を攻略するまで（2 日）

### 攻撃環境の構築

HTB といえば Kali Linux というイメージがあったので、VMware で Kali Linux の仮想環境を準備しました。設定は以下の記事を参考にしました。
https://qiita.com/v_avenger/items/c85d946ed2b6bf340a84

### VPN で接続する

HTB で Machine を攻略するためには、VPN 接続が必要です。始めたての僕はどうやって OpenVPN を使用して接続すればよいのかがわからず、非常に混乱しました。色々な選択肢があるけど何を選べばいいのだろう…

ちなみに結論としては、Protocol を `TCP 443` に設定し、 `.ovpn` をダウンロードし、次のコマンドを実行するだけでした。実はこの Protocol の変更が**重要**で、これをしなかった僕はこのあと苦労します。

```
$ sudo openvpn lab_<username>.ovpn
```

### Starting Point の攻略

HTB には Starting Point というチュートリアル的な Machine があります。最初はこれらを解いて HTB の遊び方を学びました。僕は公式の writeup を読みながら進めました。

しかし、4 台解いたところで Stating Point に**飽きました**！

### 無謀にも Active Machine に挑む

ということで、一番簡単そうな Active Machine に挑みました。しかし、知識が不足していたため、次のような初歩的な問題にぶつかりました。

- 名前解決が出来ない
  `/etc/hosts` に手動で追加する必要があることに気づかず。
- Burp の使い方を忘れてしまった
- Reverse Shell がわからなかった

結局、`user.txt` すら手に入れることができず、撃沈しました。

### 勉強してリベンジ

一度、Retired Machine の writeup をいくつか読んで勉強することにしました。具体的には「hack the box writeup」で検索したり、Qiita や Zenn の `Hack The Box` タグを調べたりしました。異なる問題でも、定石やツールなどを学べるので大変参考になりました。

ある程度知識を増やした段階でさきほどの問題にリベンジすると、なんとか解けました！　今でも Active なので詳細について明かせないのが残念です。

## Hacker になるまで（2 日）

まず、Active Machine の EASY をすべて解きました。わからない問題もありましたが、公式フォーラムのヒントを頼りに乗り越えました。

EASY を埋めても Hacker までのポイントが少し足りないため、Challenge の Crypto を解きました。Challenge は一般的な CTF 形式ですが、想像以上に問題の質が高くて驚きました。有名どころの CTFer が携わっているのかな？

そして、あっさりと目標の Hacker に**昇格**しました！
![](https://storage.googleapis.com/zenn-user-upload/ef487f0238cd-20231107.png)

## 初めて MEDIUM を攻略するまで（4 日）

今では Retired になっていますが、Format という MEDIUM の Machine に挑みました。ちなみに、この Machine については僕も writeup を公開しています。
https://zenn.dev/hk_ilohas/articles/htb-format-linux-medium

EASY から難易度が一段と上がり、苦戦しました。具体的には、次のような点が挙げられます。

- 問題構造がより複雑になった
- 攻撃方針を誤っていて行き詰まった
- `www-data` から `user` といった水平方向の権限昇格というステップが増えた

時間はかかりましたが、奇跡的に解けました。その要因は以下の通りです。

- ピンポイントな Redis の記事を偶然発見した
- ふと思いついた攻撃方針が成功した
- Python の Bypass に慣れていた

## HTB に復帰するまで（約 1 ヶ月）

### 問題発生

MEDIUM 1 台解けた後、以下の問題が発生しました。

- 他の MEDIUM が全く解けない
- Kali Linux の仮想環境（VMware）が重すぎて、文字入力すらままならなくなる

特に 2 つ目が致命的で、ストレスでモチベーションが消失しました。おそらく、PC のスペック不足が原因なので、どうしようもありませんでした。

というわけで、ここから約 1 ヶ月間 HTB から**離れました**。

### 天啓が降りる

数週間が経った頃、天啓が降りました。「**WSL** で Kali Linux を動かしたらいいのでは？」と。以前は容量が大きすぎて踏み切れなかったのですが、別の HDD に移動できることを知り、自分の環境でも対応できました。

Kali Linux の簡易インストール後は、次の手順で行いました。

```
PS C:\Users\HK> wsl --export kali-linux D:\wsl\kali-linux.tar
エクスポートが進行中です。これには数分かかる場合があります。
この操作を正しく終了しました。
PS C:\Users\HK> wsl --unregister kali-linux
登録解除。
この操作を正しく終了しました。
PS C:\Users\HK> wsl --import kali-linux D:\wsl\ D:\wsl\kali-linux.tar
インポート中です。この処理には数分かかることがあります。
この操作を正しく終了しました。
```

あとは Win-KeX や ツールをインストールしました。`root` で入れるとバグったので、`kali` に切り替えて操作をしました（`kali` は簡易インストール時に作成した user）。
https://www.kali.org/docs/wsl/win-kex/

起動も `kali` で次のコマンドを実行しました。

```
$ kex --win -m
```

WSL にすることで、期待通りサクサク動作するようになりました！

### 大量の writeup を読み漁る

HTB のモチベーションは復活しましたが、このままだと MEDIUM は解けないので、大量の writeup を読み漁ることにしました。個人的なオススメは次の通りです。英語では優れた writeup がたくさんありますが、特に 0xdf 氏のものは道中の思考を含めて非常に洗練されており、勉強になりました。

#### 日本語

https://qiita.com/schectman-hell
https://qiita.com/v_avenger

#### 英語

https://0xdf.gitlab.io/

### 新しい EASY を解く

ちょうど Active Machine が入れ替わり、新しい EASY Machine が登場しました。リハビリとして挑戦することにしました。新しい環境で実際に Machine を攻略していると、新たな 2 つの問題に直面しました。

一つ目は、SSH ができなかったことです。調べてみたところ、新しい Kali Linux のデフォルト鍵交換方式と HTB の鍵交換方式が一致していないことが[原因](https://x.com/Azalea47520138/status/1709477431755821530)だとわかりました。解決するためには、オプションを付ける必要があります。具体的には次の通りです。

```
$ ssh -o KexAlgorithms=ecdh-sha2-nistp256 user@host
```

二つ目は、ローカルとマシン間の通信に不具合があったことです。原因は不明でしたが、OpenVPN のプロトコルを `TCP 443` に変更すると解決しました。次の記事は別の問題ですが、これを参考にすると上手くいきました。
https://hack-lab-256.com/hackthebox-evil-winrm-error/1080/

面倒な問題に遭遇しましたが、無事に EASY を解くことができ、HTB に完全復帰しました。

## Pro Hacker になるまで（10 日）

せっかく復帰したので、**Pro Hacker** になることを目標に設定しました。

まずは、MEDIUM を全て埋めることにしました。Linux Machine は休暇中の勉強成果が表れ、時間はかかりましたが問題なく解けました！

Windows Machine の知識は皆無でしたので、一旦 writeup を読み漁りました。Windows は Linux に比べて問題構造が複雑ですが、定石が比較的ハッキリしているため理解しやすかったです。そして、無事に MEDIUM を埋めることに成功しました！

次は、HARD に挑戦しました。MEDIUM より更に難しかったので大苦戦しましたが、writeup 読み漁りのおかげか 1 台奇跡的に解けました。

Hacker のときと同様に、Pro Hacker まで少しポイントが足りなかったので、Challenge の Crypto を解きました。深夜 2 時ごろだったので、RTA のように問題を解きまくりました。

そして、無事に Pro Hacker に**昇格**しました！

## 効果的だったこと

Pro Hacker になるまでに様々なことをしましたが、特に効果的だったのは以下の 2 つです。

### writeup を読み漁ること

他の人が書いた writeup を読むことには以下のメリットがあります。

- 定石やツールなどを学べる
- 自分の知識をアップデートできる

個人的な事例で説明すると、`nmap` を次のように使っていました。

```
$ nmap -p- -sV -sC <IP>
```

`nmap` を使ったことがある人ならわかると思いますが、このオプションでは非常に時間がかかります。回線状況によっては 40, 50 分ぐらいかかりました。しかし、writeup に書いてあった方法に変更すると、数分で済むようになりました。

```
$ nmap -p- --min-rate 10000 <IP>
$ nmap -p <PORTS> -sCV <IP>
```

他にも色々な事例がありますが、このように思いがけない学びもあります。

### 自分用の writeup を雑に作成しながら解くこと

**雑に**というのが肝です。公開するレベルで作成すると非常にめんどくさいのですが、コマンドや一言コメント、スクリーンショットを貼り付ける程度のメモであれば、割と楽に行なえます。

writeup を書くことで思考を整理でき、正答率が上がった気がします。また、後日「あのコマンド何だったっけ？」というようなときに簡単に見返すことができ、とても便利です。

僕の場合、HackMD でメモを取っています。日常だと Notion を使っていますが、プログラムや数式には難があるので、HackMD を使用しています。

https://hackmd.io/
![](https://storage.googleapis.com/zenn-user-upload/88299403622e-20231108.png)

HackMD の無料版だと画像のアップロードが 1 MB までという制限があり、スクリーンショットだと引っかかることがあります。そのときは Squoosh で圧縮しています。Squoosh はローカルで処理をするため、セキュリティ的にも安心です。
https://squoosh.app/

## おわりに

HTB を初めて 2 ヶ月で無事 Pro Hacker になれました！　次は Elite Hacker を目指して解いていきたいです。Hacker から Pro Hacker へのときよりも大変そうですが…
