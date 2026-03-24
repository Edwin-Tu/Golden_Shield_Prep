# picoCTF 2025 - EVEN RSA CAN BE BROKEN??? 教學手冊

## 題目資訊
- 題目名稱：EVEN RSA CAN BE BROKEN???
- 題目類型：Cryptography
- 難度：Easy
- 連線方式：`nc verbal-sleep.picoctf.net 60458`

---

## 題目敘述
題目會透過 netcat 提供三個 RSA 相關數值：
- `N`
- `e`
- `cyphertext`

要求我們只靠這些公開資訊，把加密後的 flag 解回來。

---

## 本題核心觀念
正常 RSA 會選兩個大質數：

- `p`
- `q`

然後計算：

```text
N = p × q
```

通常 RSA 的兩個質數都會是奇數，因此：

```text
奇數 × 奇數 = 奇數
```

也就是說，**正常 RSA 的 `N` 幾乎一定是奇數**。

但這題提供的 `N` 最後是：

```text
...781014
```

代表 `N` 是 **偶數**，因此一定可以被 `2` 整除。

這就表示：

```text
p = 2
q = N // 2
```

只要知道 `p` 和 `q`，就能進一步算出 RSA 私鑰，最後把密文解回 flag。

---

## 題目原始碼重點
題目提供的 `encrypt.py` 內容如下：

```python
from sys import exit
from Crypto.Util.number import bytes_to_long, inverse
from setup import get_primes

e = 65537

def gen_key(k):
    """
    Generates RSA key with k bits
    """
    p,q = get_primes(k//2)
    N = p*q
    d = inverse(e, (p-1)*(q-1))

    return ((N,e), d)

def encrypt(pubkey, m):
    N,e = pubkey
    return pow(bytes_to_long(m.encode('utf-8')), e, N)

def main(flag):
    pubkey, _privkey = gen_key(1024)
    encrypted = encrypt(pubkey, flag) 
    return (pubkey[0], encrypted)

if __name__ == "__main__":
    flag = open('flag.txt', 'r').read()
    flag = flag.strip()
    N, cypher  = main(flag)
    print("N:", N)
    print("e:", e)
    print("cyphertext:", cypher)
    exit()
```

從這段程式可以知道：

1. 公開指數固定是 `e = 65537`
2. `N = p * q`
3. 私鑰是透過 `inverse(e, (p-1)*(q-1))` 算出來
4. 密文是用：

```python
pow(bytes_to_long(m.encode('utf-8')), e, N)
```

因此只要能取得 `p` 和 `q`，整題就能破解。

---

## 本題拿到的數值
本次題目提供：

```text
N = 15759861201924286828177538908735993181346999562841594315449820441559609800133026102982844642962710197834005386835458855908275959919130250880111172891781014
e = 65537
cyphertext = 9135843788912129977870012437926501416502529018223262791329403149861558559550767036818704491081660571951905548123759492372745730891393658276323264820133595
```

觀察 `N` 的尾數是 `14`，因此：

```text
N % 2 == 0
```

所以：

```text
p = 2
q = N // 2
```

---

## 解題步驟

### 第一步：分解 N
因為 `N` 是偶數，所以：

```python
p = 2
q = N // 2
```

---

### 第二步：計算 φ(N)
RSA 中會用到歐拉函數：

```text
φ(N) = (p - 1)(q - 1)
```

代入 `p = 2`：

```text
φ(N) = (2 - 1)(q - 1)
     = q - 1
```

---

### 第三步：計算私鑰 d
RSA 私鑰 `d` 需要滿足：

```text
e × d ≡ 1 (mod φ(N))
```

在 Python 中可直接寫成：

```python
d = pow(e, -1, phi)
```

這行的意思是：求出 `e` 在模 `phi` 下的反元素。

---

### 第四步：解密密文
RSA 解密公式為：

```text
m = c^d mod N
```

Python 寫法：

```python
m = pow(c, d, N)
```

這裡得到的 `m` 還不是字串，而是一個大整數。

---

### 第五步：把整數轉回文字
RSA 解出來後，需要把整數轉成 bytes，再解碼為字串：

```python
flag = m.to_bytes((m.bit_length() + 7) // 8, "big")
print(flag.decode())
```

---

## 完整解題程式
把以下內容存成 `solve.py`：

```python
N = 15759861201924286828177538908735993181346999562841594315449820441559609800133026102982844642962710197834005386835458855908275959919130250880111172891781014
e = 65537
c = 9135843788912129977870012437926501416502529018223262791329403149861558559550767036818704491081660571951905548123759492372745730891393658276323264820133595

# 由於 N 是偶數，可直接推出其中一個因數是 2
p = 2
q = N // 2

# 計算 φ(N)
phi = (p - 1) * (q - 1)

# 求模反元素，取得私鑰 d
# d 滿足 e * d ≡ 1 (mod phi)
d = pow(e, -1, phi)

# RSA 解密
m = pow(c, d, N)

# 轉回原本文字
flag = m.to_bytes((m.bit_length() + 7) // 8, "big")
print(flag.decode())
```

---

## 執行方式
在 WSL / Linux 終端機輸入：

```bash
python3 solve.py
```

---

## 解出結果
執行後可得到：

```text
picoCTF{tw0_1$_pr!m375129bb1}
```

---

## 為什麼這題能秒解
這題不是因為 RSA 本身很弱，而是因為 **金鑰生成錯誤**。

正常情況下：
- `p` 和 `q` 應該都是大型奇質數
- 所以 `N` 應該是奇數

但這題出現偶數 `N`，等於直接透露：

```text
其中一個質因數是 2
```

這讓 RSA 最關鍵的安全前提──「很難分解 N」──直接失效。

---

## 本題學到的重點
1. 看到 RSA 題目時，先做基本檢查，不要急著上複雜工具。
2. 如果 `N` 是偶數，代表它一定能被 `2` 整除。
3. 一旦知道 `p` 和 `q`，就能算出 `φ(N)`。
4. 算出 `φ(N)` 後，就能求得私鑰 `d`。
5. 最後再用 `m = c^d mod N` 解出明文。

---

## 快速檢查模板
以後遇到 RSA 題目，可以先試：

```python
if N % 2 == 0:
    print("N 是偶數，代表其中一個因數是 2")
```

這種基礎檢查，常常就是解題突破口。

---

## 總結
這題的關鍵不在暴力破解，也不在高深數論，而是：

> **先觀察 `N` 的性質。**

當你發現 `N` 是偶數時，題目其實就已經破了一半。

