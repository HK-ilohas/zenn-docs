---
title: "SECCON Beginners CTF 2022 Writeup (Crypto)"
emoji: "ð£"
type: "tech" # tech: æè¡è¨äº / idea: ã¢ã¤ãã¢
topics: ["CTF"]
published: true
---

## ã¯ããã«

VLANouroboros ã® HK-ilohas ã¨ãã¦ãSECCON Beginners CTF 2022 ã«åå ãã¾ãããä»å¹´ã¯ Crypto å¨åã¨ Misc æ°åãè§£ãã¾ãããå»å¹´ã¯ Crypto ã 1 åè§£ããªãã£ãã®ã§ããã£ã¨æé·ãã¦ããã®ã§ãããã

ãã¦ããã®è¨äºã§ã¯ Crypto ã® Writeup ãæ¸ãã¾ããomni-RSA ã¯ small_roots ãä¸æãåºããªãã£ãã®ã§ãããã´ãªæ¼ãã§è§£ãã¦ãã¾ããåäººçã«ã¯ãCommand ã¨ omni-RSA ãå¥½ã¿ã§ããã

![](https://storage.googleapis.com/zenn-user-upload/6a54e3d524a1-20220605.png)

## CoughingFox [beginner]

### åé¡æ¦è¦

```python:problem.py
from random import shuffle

flag = b"ctf4b{XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX}"

cipher = []

for i in range(len(flag)):
    f = flag[i]
    c = (f + i)**2 + i
    cipher.append(c)

shuffle(cipher)
print("cipher =", cipher)

```

```python:output.txt
cipher = [12147, 20481, 7073, 10408, 26615, 19066, 19363, 10852, 11705, 17445, 3028, 10640, 10623, 13243, 5789, 17436, 12348, 10818, 15891, 2818, 13690, 11671, 6410, 16649, 15905, 22240, 7096, 9801, 6090, 9624, 16660, 18531, 22533, 24381, 14909, 17705, 16389, 21346, 19626, 29977, 23452, 14895, 17452, 17733, 22235, 24687, 15649, 21941, 11472]
```

- ãã©ã°ã® $i$ çªç®ã®æå­ $f$ ã«å¯¾ãã¦ã$c = (f + i)^2 + i$ ãè¨ç®ãã
- $c$ ã `cipher` ã«è¿½å ãã
- ãããå¨ã¦ã® $i$ ã«ã¤ãã¦è¡ã
- `cipher` ãã·ã£ããã«ãã

### è§£æ³

`cipher` ã¯ã·ã£ããã«ããã¦ããã®ã§ãã¾ãã¯ä½çªç®ã®æå­ããããªãã¡ $i$ ãç¹å®ããå¿è¦ãããã$c - i = (f + i)^2$ ããã$\sqrt{c-i}$ ã¯æ´æ°ã«ãªãã®ã§ããããå©ç¨ãã¦ $i$ ãæ±ããã$i$ ãæ±ã¾ãã°ã$f = \sqrt{c-i} - i$ ã§ãã©ã°ã® $i$ çªç®ã®æå­ $f$ ãè§£èª­ã§ããã

```python:solve.py
from gmpy2 import iroot

cipher = [12147, 20481, 7073, 10408, 26615, 19066, 19363, 10852, 11705, 17445, 3028, 10640, 10623, 13243, 5789, 17436, 12348, 10818, 15891, 2818, 13690, 11671, 6410, 16649, 15905, 22240, 7096, 9801, 6090, 9624, 16660, 18531, 22533, 24381, 14909, 17705, 16389, 21346, 19626, 29977, 23452, 14895, 17452, 17733, 22235, 24687, 15649, 21941, 11472]
flag = [0 for _ in range(len(cipher))]

for c in cipher:
    for i in range(len(cipher)):
        m = iroot(c-i, 2)
        if m[1] == True:
            flag[i] = chr(m[0] - i)
            break

print("".join(flag))

```

## PrimeParty [easy]

### åé¡æ¦è¦

```python:server.py
from Crypto.Util.number import *
from secret import flag
from functools import reduce
from operator import mul


bits = 256
flag = bytes_to_long(flag.encode())
assert flag.bit_length() == 455

GUESTS = []


def invite(p):
    global GUESTS
    if isPrime(p):
        print("[*] We have been waiting for you!!! This way, please.")
        GUESTS.append(p)
    else:
        print("[*] I'm sorry... If you are not a Prime Number, you will not be allowed to join the party.")
    print("-*-*-*-*-*-*-*-*-*-*-*-*-")


invite(getPrime(bits))
invite(getPrime(bits))
invite(getPrime(bits))
invite(getPrime(bits))

for i in range(3):
    print("[*] Do you want to invite more guests?")
    num = int(input(" > "))
    invite(num)


n = reduce(mul, GUESTS)
e = 65537
cipher = pow(flag, e, n)

print("n =", n)
print("e =", e)
print("cipher =", cipher)

```

- RSA ã®åé¡
- ãµã¼ãã®ãã­ã°ã©ã  `server.py` ãä¸ãããã
- $n$ ã«ä»»æã®ç´ æ° 3 ã¤ãè¿½å ã§ãã

### è§£æ³

$n$ ã«ä»»æã®ç´ æ° 3 ã¤ãè¿½å ã§ããã®ã§ãããç¨åº¦å¤§ããç´ æ° $p, q, r$ ãé©å½ã«é¸ãã§éä¿¡ãã$d_{pqr} \equiv e^{-1} \pmod{\phi (pqr)}$ ã¨ãã¦ã$\text{cipher}^{d_{pqr}} \bmod{pqr}$ ã§å¾©å·ã§ããã

```python:solve.py
from Crypto.Util.number import long_to_bytes, getPrime

# p = getPrime(256)
# q = getPrime(256)
# r = getPrime(256)
# print(f"p = {p}")
# print(f"q = {q}")
# print(f"r = {r}")

p = 103611340022948920047206697682854214284077513484428122456568351468608769960659
q = 71911486628615636174586927761843540515001158245260223140530176276660431137567
r = 105479008010107316750674250979137157990625660512766801522302298758814994591023

n = 66196520127116141129419168771863204498197699090100617898008952155542077844240289174615367937967288080778181177362159673621244315123463561802698289321344353484885009796886233149519618846919924497860199435937085587007336776267143279084451797077208624479145541148899795426904916001166187762498113876746695423637679781606528036125548499783529398999190197975389910891441917180156279826391680741203296851165604745742852366839087754053508669078077771631315157317265913799421882570307855039503106868054788404238731441408058182422113675054937861219
e = 65537
cipher = 33566576020133322586231570563996803200369251297642187422721475027694391986843516359651100073032268756945701275781923531371462155493327898938842249838821366582783997475620244205686649757881121063675796732058891500540164933186284800809696538100273099265914792066856319551280420762951419560070750332302429894816762233817882368660665132760644437588064513427422724479642370406046539986997878809979569992518424465318977273889470832767451069524956028200533574077812645596686119234513133143026954630141439819805709124337764518253230703116114658805

phi_pqr = (p-1)*(q-1)*(r-1)
d_pqr = pow(e, -1, phi_pqr)
flag = pow(cipher, d_pqr, p*q*r)

print(long_to_bytes(flag))

```

## Command [easy]

### åé¡æ¦è¦

```python:chal.py
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
from Crypto.Util.number import isPrime
from secret import FLAG, key
import os


def main():
    while True:
        print('----- Menu -----')
        print('1. Encrypt command')
        print('2. Execute encrypted command')
        print('3. Exit')
        select = int(input('> '))

        if select == 1:
            encrypt()
        elif select == 2:
            execute()
        elif select == 3:
            break
        else:
            pass

        print()


def encrypt():
    print('Available commands: fizzbuzz, primes, getflag')
    cmd = input('> ').encode()

    if cmd not in [b'fizzbuzz', b'primes', b'getflag']:
        print('unknown command')
        return

    if b'getflag' in cmd:
        print('this command is for admin')
        return

    iv = os.urandom(16)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    enc = cipher.encrypt(pad(cmd, 16))
    print(f'Encrypted command: {(iv+enc).hex()}')


def execute():
    inp = bytes.fromhex(input('Encrypted command> '))
    iv, enc = inp[:16], inp[16:]
    cipher = AES.new(key, AES.MODE_CBC, iv)
    try:
        cmd = unpad(cipher.decrypt(enc), 16)
        if cmd == b'fizzbuzz':
            fizzbuzz()
        elif cmd == b'primes':
            primes()
        elif cmd == b'getflag':
            getflag()
    except ValueError:
        pass


def fizzbuzz():
    for i in range(1, 101):
        if i % 15 == 0:
            print('FizzBuzz')
        elif i % 3 == 0:
            print('Fizz')
        elif i % 5 == 0:
            print('Buzz')
        else:
            print(i)


def primes():
    for i in range(1, 101):
        if isPrime(i):
            print(i)


def getflag():
    print(FLAG)


if __name__ == '__main__':
    main()

```

- AES-CBC ã®åé¡
- ãµã¼ãã®ãã­ã°ã©ã  `chal.py` ãä¸ãããã
- 3 ã¤ã®ã³ãã³ã `fizzbuzz`, `primes`, `getflag` ãç¨æããã¦ãã
  - `getflag` ãå®è¡ããã¨ããã©ã°ãè¡¨ç¤ºããã
- `encrypt` ã§ `fizzbuzz` ã `primes` ãæå·åã§ãã
  - `getflag` ã¯é¸æã§ããªã
- `execute` ã§æå·åãããã³ãã³ããå®è¡ã§ãã
  - ããã§ã¯ãæå·æããããã° `getflag` ãä½¿ãã

### è§£æ³

- `fizzbuzz` ã `primes` ã®æå·æãå¥æãã
- AES-CBC ãªã®ã§ãæå·æã«ã¯ IV ãã¤ãã¦ãã
- ä»åãããããããªã®ã¯ IV ã ããªã®ã§ããããèª¿æ´ãã¦å¹³æã getflag ã«ãã

æç« ã§ä¸æãèª¬æã§ããªãã£ãã®ã§ãææ¸ãã®å³ãè¼ãã¦ããã¾ãã

![](https://storage.googleapis.com/zenn-user-upload/0cb31f6d1bae-20220605.png)

---

![](https://storage.googleapis.com/zenn-user-upload/bd2ec64d2e84-20220605.png)

---

![](https://storage.googleapis.com/zenn-user-upload/a8344014923e-20220605.png)

```python:solve.py
from pwn import *
from binascii import unhexlify
from Crypto.Util.Padding import pad

io = remote("command.quals.beginners.seccon.jp", 5555)

io.recvuntil(b">")
io.sendline(b"1")
io.recvuntil(b">")
io.sendline(b"fizzbuzz")
io.recvuntil(b":")
ret = unhexlify(io.recvline().strip())

iv, enc = ret[:16], ret[16:]
old_cmd = pad(b"fizzbuzz", 16)
new_cmd = pad(b"getflag", 16)
iv = xor(iv, old_cmd, new_cmd)

io.recvuntil(b">")
io.sendline(b"2")
io.recvuntil(b">")
io.sendline((iv+enc).hex().encode())

print(io.recvline().decode())

```

## Unpredictable Pad [medium]

### åé¡æ¦è¦

```python:chal.py
import random
import os


FLAG = os.getenv('FLAG', 'notflag{this_is_sample_flag}')


def main():
    r = random.Random()

    for i in range(3):
        try:
            inp = int(input('Input to oracle: '))
            if inp > 2**64:
                print('input is too big')
                return

            oracle = r.getrandbits(inp.bit_length()) ^ inp
            print(f'The oracle is: {oracle}')
        except ValueError:
            continue

    intflag = int(FLAG.encode().hex(), 16)
    encrypted_flag = intflag ^ r.getrandbits(intflag.bit_length())
    print(f'Encrypted flag: {encrypted_flag}')


if __name__ == '__main__':
    main()

```

- `random` ã® `getrandbits`ãããªãã¡ã¡ã«ã»ã³ãã»ãã¤ã¹ã¿ã®åé¡
- $2^{64}$ ä»¥ä¸ã®æ° `inp` ãå¥åãã
- ããã¨ã`inp` ã¨åããããæ°ã®ä¹±æ°ãçæããã`inp` ã¨ XOR ããå¤ãä¸ãããã
- ãã®æä½ã 3 åè¡ãã
- ãã®å¾ããã©ã°ã¨ä¹±æ°ã XOR ããããã®ãæ¸¡ããã

## è§£æ³

åºæ¬çã«ã¯ã[ãããããã¯ãã­ã¸ã¼ Mersenne Twister ã®åºåãæ¨æ¸¬ãã¦ã¿ã](https://inaz2.hatenablog.com/entry/2016/03/07/194147) ã®ã¹ã¯ãªãããä½¿ã£ã¦è§£ããã

ãã®åé¡ã§å¿è¦ãªåå®¹ã¯ä»¥ä¸ã® 2 ã¤ã§ããã

- Python ã® `random` ã¯ã¡ã«ã»ã³ãã»ãã¤ã¹ã¿
- ã¡ã«ã»ã³ãã»ãã¤ã¹ã¿ã¯é£ç¶ãã 624 åã®åºåãããã°ãåé¨ç¶æãå¾©åå¯è½ï¼ããï¼

åºå 1 åã¯ 32 ããããªã®ã§ãè¨ 19968 ããããå¿è¦ã¨ãªãããããã`inp` ã¯ $2^{64}$ ä»¥ä¸ã¨ããæ¡ä»¶ããããããã 3 åããæä½ã§ããªãã®ã§ã19968 ãããã«ã¯å¨ãå±ããªããã¨ããããè² ã®æ°ã«ã¤ãã¦ã¯å¶éããªãã®ã§ããããã¤ãã¦ãããã°ãããå·ä½çã«ã¯ã19968 ã 3 ã§å²ã£ã 6656 ããã $-(2^{6656} - 1)$ ã 3 åéä¿¡ãããã¨ã«ãªãã

ãã¨ã¯ãåç§è¨äºã®ã¹ã¯ãªããã«å½ã¦ã¯ãã¦è§£ããããã©ã°ã®ãããæ°ã¯æå·æã®ãããæ°ã¨åãã¨ããä»®å®ã§ã¹ã¯ãªãããæ¸ããã®ã§ãå ´åã«ãã£ã¦ã¯ç ´æãã¦ããã

```python:solve.py
import random
from pwn import *
from Crypto.Util.number import long_to_bytes


io = remote("unpredictable-pad.quals.beginners.seccon.jp", 9777)
inp = -(2**6656 - 1)

io.sendlineafter(b": ", str(inp).encode())
io.recvuntil(b": ")
output1 = int(io.recvline().strip()) ^ inp

io.sendlineafter(b": ", str(inp).encode())
io.recvuntil(b": ")
output2 = int(io.recvline().strip()) ^ inp

io.sendlineafter(b": ", str(inp).encode())
io.recvuntil(b": ")
output3 = int(io.recvline().strip()) ^ inp

io.recvuntil(b": ")
cipher = int(io.recvline().strip())

outputs = []
for i in range(208):
    x = (output1 & ((2**32 - 1) << (32*i))) >> (32*i)
    outputs.append(x)
for i in range(208):
    x = (output2 & ((2**32 - 1) << (32*i))) >> (32*i)
    outputs.append(x)
for i in range(208):
    x = (output3 & ((2**32 - 1) << (32*i))) >> (32*i)
    outputs.append(x)


# https://inaz2.hatenablog.com/entry/2016/03/07/194147
def untemper(x):
    x = unBitshiftRightXor(x, 18)
    x = unBitshiftLeftXor(x, 15, 0xefc60000)
    x = unBitshiftLeftXor(x, 7, 0x9d2c5680)
    x = unBitshiftRightXor(x, 11)
    return x

def unBitshiftRightXor(x, shift):
    i = 1
    y = x
    while i * shift < 32:
        z = y >> shift
        y = x ^ z
        i += 1
    return y

def unBitshiftLeftXor(x, shift, mask):
    i = 1
    y = x
    while i * shift < 32:
        z = y << shift
        y = x ^ (z & mask)
        i += 1
    return y


mt_state = tuple([untemper(x) for x in outputs] + [624])
random.setstate((3, mt_state, None))

print(long_to_bytes(cipher ^ random.getrandbits(cipher.bit_length())))

```

## omni-RSA [hard]

### åé¡æ¦è¦

```python:problem.py
from Crypto.Util.number import *
from flag import flag

p, q, r = getPrime(512), getPrime(256), getPrime(256)
n = p * q * r
phi = (p - 1) * (q - 1) * (r - 1)
e = 2003
d = inverse(e, phi)

flag = bytes_to_long(flag.encode())
cipher = pow(flag, e, n)

s = d % ((q - 1)*(r - 1)) & (2**470 - 1)

assert q < r
print("rq =", r % q)

print("e =", e)
print("n =", n)
print("s =", s)
print("cipher =", cipher)

```

- $n = pqr$ ã® RSA
- $s = (d \bmod{(q-1)(r-1)}) \And (2^{470} - 1)$ ã¨ $r \bmod{q}$ ãä¸ãããã

### è§£æ³

$d_{qr} = d \bmod{(q-1)(r-1)}$ ã¨ããã$s$ ãæ¬¡ã®ããã«æ¸ãæããã$k$ ã¯ãã¹ã¯ã«ããç ´æãã $d_{qr}$ ã®åé ­ãããã§ããããã 42 ãããã§ããã

$$
s = d_{qr} - 2^{470}k \Rightarrow d_{qr} = s + 2^{470}k
$$

$d_{qr}$ ã¯æ³ $(q-1)(r-1)$ ã«ããã $e$ ã®éåã§ãããã¨ãããæ¬¡ã®ããã«å¤å½¢ã§ããã

$$
\begin{align*}
    ed_{qr} &\equiv 1 \pmod{(q-1)(r-1)} \\
    ed_{qr} &= 1 + l(q-1)(r-1) \\
    e(s+2^{470}k) &= 1 + l(q-1)(r-1)
\end{align*}
$$

ãã®ã¨ãã$l$ ã¯ååå°ããæ¢ç´¢å¯è½ã§ãããã¨ããå®é¨ãã¦ã¿ãã¨ããã«ãããã

ããã§ã$a = r \bmod{q}$ ãä¸ãããã¦ãããã¨ãæãåºããã$r$ ã¨ $q$ ã¯åããããæ°ã§ $r > q$ ã§ãã£ããã¨ãããæ¬¡ã®ããã«æ¸ãããã¨ãæå¾ã§ããã

$$
r = q + a
$$

ãããä¸ã®å¼ã«ä»£å¥ããã

$$
e(s+2^{470}k) = 1 + l(q-1)(q+a-1)
$$

ä½åè Writeup ã§ã¯ãããã $\text{mod}\,\, q$ ããã¦ `small_roots` ã§è§£ãã¦ããããåã¯ä¸æãåºããªãã£ãã®ã§éãæ¹æ³ã§è§£ãã¦ã¿ãã

$l$ ã¯ç·å½ããå¯è½ãªã®ã§æ¢ç¥ã¨ããã¨ãç¾ç¶ã®æªç¥å¤æ°ã¯ $k$ ã¨ $q$ ã® 2 ã¤ã§ãããä¸è¿°ã®ã¨ããã$q$ ãæ¶ãã®ã¯ä¸æãã§ããªãã£ãã®ã§ãä»åº¦ã¯ $k$ ãæ¶ããã¨ã«ããã

$$
es \equiv 1 + l(q-1)(q+a-1) \pmod{2^{470}}
$$

å¾ã¯ã`solve_mod` ã§ $q$ ãå°åºããããã ãã¹ã¯ãªããã®æ¸ãæ¹ãæªãããã§ãã«ã¼ããç´ 380 åãã¨ã«ããªã¼ãºãããã©ãããã°ãããã ...ï¼

```python:solve_q.sage
rq = 7062868051777431792068714233088346458853439302461253671126410604645566438638
e = 2003
n = 140735937315721299582012271948983606040515856203095488910447576031270423278798287969947290908107499639255710908946669335985101959587493331281108201956459032271521083896344745259700651329459617119839995200673938478129274453144336015573208490094867570399501781784015670585043084941769317893797657324242253119873
s = 1227151974351032983332456714998776453509045403806082930374928568863822330849014696701894272422348965090027592677317646472514367175350102138331
cipher = 82412668756220041769979914934789463246015810009718254908303314153112258034728623481105815482207815918342082933427247924956647810307951148199551543392938344763435508464036683592604350121356524208096280681304955556292679275244357522750630140768411103240567076573094418811001539712472534616918635076601402584666

X = var("X")
for l in range(1, e):
    print(f"l = {l}")
    results = solve_mod([1 + l*(X-1)*(X+rq-1) == e*s], 2^470)
    for x in results:
        q = ZZ(x[0])
        if n % q == 0:
            print(q)
            break

#l = 1576
#108719400953000878740030929903618126158486070837750092259928673760881189657243

```

```python:solve.py
from Crypto.Util.number import long_to_bytes

rq = 7062868051777431792068714233088346458853439302461253671126410604645566438638
e = 2003
n = 140735937315721299582012271948983606040515856203095488910447576031270423278798287969947290908107499639255710908946669335985101959587493331281108201956459032271521083896344745259700651329459617119839995200673938478129274453144336015573208490094867570399501781784015670585043084941769317893797657324242253119873
s = 1227151974351032983332456714998776453509045403806082930374928568863822330849014696701894272422348965090027592677317646472514367175350102138331
cipher = 82412668756220041769979914934789463246015810009718254908303314153112258034728623481105815482207815918342082933427247924956647810307951148199551543392938344763435508464036683592604350121356524208096280681304955556292679275244357522750630140768411103240567076573094418811001539712472534616918635076601402584666

q = 108719400953000878740030929903618126158486070837750092259928673760881189657243
r = q + rq
p = n // (q * r)

phi = (p - 1) * (q - 1) * (r - 1)
d = pow(e, -1, phi)
m = pow(cipher, d, n)
print(long_to_bytes(m))

```
