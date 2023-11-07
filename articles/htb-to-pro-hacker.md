---
title: "【Hack The Box】無課金学生が2ヶ月でPro Hackerになるまでの軌跡"
emoji: "🙌"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Security", "hackthebox", "htb"]
published: false
---

## はじめに

![](https://storage.googleapis.com/zenn-user-upload/d41afcbaac1a-20231107.png)

Profile: https://app.hackthebox.com/profile/1660611

Hack The Box (HTB) を始めて 2 ヶ月で Pro Hacker に昇格したので、その記念と備忘録としてそれまでの軌跡を記事にしました。カッコの日数は、要した時間を表しています。

## 当初のスペック

- 情報系の高専生
- 英語以外の資格は特に持っていない
- セキュリティは多少学んだことある
- Hack The Box は名前ぐらいしか知らない
- ときどき CTF の Crypto をやっている
- 研究分野はセキュリティほぼ関係ない

## Hack The Box (HTB) とは

https://www.hackthebox.com/

Hack The Box (HTB) は、ゲームのようにペネトレーションテストをトレーニングできるオンラインプラットフォームです。VM 上に脆弱なマシンが用意されており、実際に攻撃・侵入することで様々なスキルを学ぶことができます。HTB の問題は大きく分けて 2 種類あります。

- Machine：脆弱なマシンを攻撃・侵入して、`user.txt` と `root.txt` を入手する問題
- Challenge：Jeopardy 形式の flag を入手する問題

さらに、これらの問題は 2 種類に分類されます。

- Active：加点対象の新しい問題。writeup^[問題の解法についての記事。HTB では Walkthrough とも呼ばれます。] の公開は禁止されている。
- Retired：加点対象外の古い問題。writeup の公開が許可されている。

加点対象の問題を解いてポイントを貯めていくことで、Rank を上げることができます。Rank 制度は下から順番に Noob, Script Kiddie, Hacker, Pro Hacker, Elite Hacker, Guru, Omniscient となっています。上 2 つは特に次元が違うイメージです。

月 14 ドルの VIP 会員以上になると、100 台以上ある Retired の問題を解けるようになる特典が付いてきます。一応、無料会員でも最新の Retired 数問は解くことができます。僕は今のところ**無課金**です。

## HTB に登録する

9 月末、唐突に HTB をやりたくなったのでアカウントを作成しました。僕が Hack The Box を知った数年前時点では、登録するために簡単な問題を解く必要があったのですが、今だとその仕組みは廃止されているそうです。

HTB 登録後は、とりあえず **Hacker** になることを目標に設定しました。理由としては、以下のような記事を読んで Hacker が登竜門的な Rank だと感じたからです。
https://qiita.com/daihi_t/items/95d5adfc4b2cea1d4c5c
https://sanposhiho.com/posts/be-hacker-in-hackthebox/

## 初めて Active Machine を攻略するまで（2 日）

### 攻撃環境の構築

HTB といえば Kali Linux というイメージがあったので、VMware で Kali Linux の仮想環境を用意しました。設定は以下の記事を参考にしました。
https://qiita.com/v_avenger/items/c85d946ed2b6bf340a84

### VPN で接続する

Hack The Box で Machine を攻略するためには、VPN で接続する必要があります。始めたての僕はどうやって OpenVPN で接続すればよいのかがわからず、非常に混乱しました。色々な選択肢があるけど何を選べばいいのだろう…

ちなみに結論としては、Protocol を `TCP 443` の方にチェックを入れて `.ovpn` をダウンロードし、次のコマンドを実行するだけでした。実はこの Protocol の変更が**重要**で、これをしなかった僕はこのあと苦戦します。

```
$ sudo openvpn lab_<username>.ovpn
```

### Starting Point の攻略

HTB には Starting Point というチュートリアル的な Machine があります。最初はこれらを解いていくことで HTB の遊び方を学びます。僕は公式の writeup を読みながら進めました。

ですが、4 台解いたところで Stating Point に**飽きました**！

### 無謀にも Active Machine に挑む

ということで、一番簡単そうな Active Machine に挑みました。しかし、知識がない状態で挑んだせいで、次のような初歩的な問題にぶつかりました。

- 名前解決が出来ない
  `/etc/hosts` に自分で追加する必要があることに中々気が付かなかったです。
- Burp の使い方を忘れた
- Reverse Shell がわからない

もちろん `user.txt` すら手に入れることができず、撃沈しました。

### 勉強してリベンジ

一旦、Retired Machine の writeup をいくつか読んで勉強することにしました。具体的には「hack the box writeup」で検索したり、Qiita や Zenn の `Hack The Box` タグを調べたりしました。問題は別でも、定石やツールなどを学べるので大変参考になりました。

ある程度知識を増やした段階でさきほどの問題にリベンジすると、なんとか解けました！　今も Active なので詳細について明かせないのが残念です。

## Hacker になるまで（2 日）

まずは Active Machine の EASY を全部解きました。よくわからない問題もありましたが、公式フォーラムのヒントを頼りになんとかなりました。

EASY を埋めても Hacker までは数ポイント足りないので、Challenge の Crypto を解きました。Challenge はよくある CTF の形式になっているのですが、想像以上に問題の質が高くて驚きました。有名どころの CTFer が携わっているのかな？

というわけで、あっさりと目標の Hacker に**昇格**しました！
![](https://storage.googleapis.com/zenn-user-upload/ef487f0238cd-20231107.png)

## 初めて MEDIUM を攻略するまで（4 日）

今では Retired になっている Format という MEDIUM の Machine に挑みました。ちなみに、この Machine については僕も writeup を公開しています。
https://zenn.dev/hk_ilohas/articles/htb-format-linux-medium

EASY から難易度が跳ね上がっていて苦戦しました。たとえば、次のような点が挙げられます。

- 問題構造がより複雑になっている
- 攻撃方針を間違えて沼にハマる
- `www-data` から `user` といった水平方向の権限昇格というステップが増える

時間はかかりましたが、奇跡的に解けました。その要因は以下の通りです。

- ピンポイントな Redis の記事を偶然発見する
- なんとなく思いついた攻撃方針がそのまま刺さる
- Python の Bypass には元々慣れていた

## HTB に復帰するまで（約 1 ヶ月）

### 問題発生

MEDIUM 1 台解けたまでは良かったのですが、そのあと次のような問題が発生しました。

- 他の MEDIUM が全然解けない
- Kali Linux の仮想環境（VMware）が重すぎて、文字入力すらままならなくなる

特に 2 つ目が致命的で、ストレスでモチベーションが消滅しました。おそらく、PC のスペックが不足しているせいなので、どうしようもありませんでした。

というわけで、ここから約 1 ヶ月間 HTB から**離れました**。

### 天啓が降りる

数週間が経った頃、天啓が降りました。「**WSL** で Kali Linux を動かしたらいいのでは？」と。それまで、容量が大きすぎて WSL で Kali Linux を導入するのは躊躇していました。ですが、別の HDD に移動できることを知り、自分の環境でも対応できました。

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

あとは Win-KeX や ツールを入れました。個人的に詰まったポイントとしては、`root` で入れるとバグったので、`kali` に切り替えて操作をしました。
https://www.kali.org/docs/wsl/win-kex/

起動も `kali` で次のコマンドで行いました。

```
$ kex --win -m
```

WSL にすることで、期待通りサクサク動作するようになりました！

### 大量に writeup を読み漁る

HTB のモチベーションは復活しましたが、このままだと MEDIUM は解けないので、大量に writeup を読み漁ることにしました。個人的なオススメは次のとおりです。英語だと優れた writeup がたくさんありますが、特に 0xdf 氏のは道中の思考を含めて非常に洗練されており、ものすごく勉強になりました。

#### 日本語

https://qiita.com/schectman-hell
https://qiita.com/v_avenger

#### 英語

https://0xdf.gitlab.io/

### 新しい EASY を解く

ちょうど Active Machine が入れ替わって、EASY の新しい Machine が登場しました。リハビリがてら挑戦することにしました。新しい環境で実際に Machine を攻略していると、新しい問題に 2 つ遭遇しました。

一つ目は、SSH ができないことです。調べてみたところ、Kali Linux のデフォルト鍵交換方式と HTB の方式が一致していないことが[原因](https://x.com/Azalea47520138/status/1709477431755821530?s=20)らしいです。解決するためには、オプションを付ける必要があります。具体的には以下の通りです。

```
$ ssh -o KexAlgorithms=ecdh-sha2-nistp256 user@host
```

二つ目は、ローカルとマシン間の通信に不具合があることです。原因はよくわかりませんでしたが、OpenVPN のプロトコルを `TCP 443` に変更すると解決しました。次の記事は別問題ですが、これを参考にすると上手くいきました。
https://hack-lab-256.com/hackthebox-evil-winrm-error/1080/

面倒な問題に遭遇しましたが、無事に EASY を解くことができ、HTB に完全復帰しました。

## Pro Hacker になるまで（10 日）

せっかく復帰したので、**Pro Hacker** になることを目標に設定しました。

まずは、MEDIUM を全て埋めることにしました。Linux Machine は休み期間の勉強成果が出たのか、多少時間はかかりましたが問題なく解けました！

Windows Machine の知識は皆無だったので、一旦 writeup を読み漁りました。Windows は Linux に比べて問題構造が複雑ですが、定石が比較的ハッキリしているため理解しやすかったです。そして、無事に MEDIUM を埋めることに成功しました！

次は、HARD に挑戦しました。MEDIUM より更に難しかったので大苦戦しましたが、writeup 読み漁りのおかげか 1 台奇跡的に解けました。

Hacker のときと同様に、Pro Hacker まで少しポイントが足りなかったので、Challenge の Crypto を解きました。深夜 2 時ごろだったので、RTA のように問題を解きまくりました。

そして、無事に Pro Hacker に**昇格**しました！

## 効果的だったこと

Pro Hacker になるまでに色々なことをしましたが、特に効果的だったのは次の 2 つです。

### writeup を読み漁ること

他の人が書いた writeup を読むことには以下のメリットがあります。

- 定石やツールなどを学べる
- 自分の知識をアップデートできる

個人的な事例で説明すると、`nmap` を次のように使っていました。

```
$ nmap -p- -sV -sC <IP>
```

`nmap` を使ったことがある人ならわかると思いますが、このオプションだと非常に時間がかかります。回線状況によっては 40, 50 分ぐらいかかりました。ですが、writeup に書いてあった方法にすると数分で済むようになりました。

```
$ nmap -p- --min-rate 10000 <IP>
$ nmap -p <PORTS> -sCV <IP>
```

他にも色々な事例がありますが、このように思いがけない学びもあります。

### 自分用の writeup を雑に作成しながら解くこと

**雑に**というのが肝です。公開するレベルで作成すると非常にめんどくさいのですが、コマンドや一言コメント、スクリーンショットを貼り付ける程度のメモであれば、割と楽に行なえます。

writeup を書くことで思考の整理になり、正答率が上がった気がします。また、後日「あのコマンド何だったっけ？」というようなときに簡単に見返すことができ、とても便利です。

僕の場合、HackMD でメモを取っています。日常だと Notion を使っていますが、プログラムや数式には難があるので、HackMD を使用しています。

https://hackmd.io/
![](https://storage.googleapis.com/zenn-user-upload/88299403622e-20231108.png)

HackMD の無料版だと画像のアップロードが 1 MB までという制限があり、スクリーンショットだと引っかかることがあります。そのときは Squoosh で圧縮しています。Squoosh はローカルで処理をするため、セキュリティ的にも安心です。
https://squoosh.app/

## おわりに

HTB を初めて 2 ヶ月で無事 Pro Hacker になれました！　次は Elite Hacker を目指して解いていきたいです。Hacker から Pro Hacker へのときよりも大変そうですが…
