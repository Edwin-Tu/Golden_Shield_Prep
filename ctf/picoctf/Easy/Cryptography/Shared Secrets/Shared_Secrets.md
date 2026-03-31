# Shared Secrets

## Challenge Metadata
- **Challenge Name:** Shared Secrets
- **Category:** Cryptography
- **Difficulty:** Easy
- **Event:** picoCTF 2026
- **Author:** YAHAYA MEDDY

## 一、基本資訊
本題提供兩個檔案：
- `message.txt`
- `encryption.py`

題目描述指出訊息是使用 shared secret 加密，但其中一方的交換資訊發生洩漏。這暗示共享金鑰建立流程存在明顯的資料外洩問題。

## 二、題目概述
本題實際使用的是 **Diffie-Hellman key exchange** 觀念，而非 RSA。流程如下：
1. 公開參數 `g`, `p`
2. Server secret `a`，計算 `A = g^a mod p`
3. Client secret `b`
4. Shared secret `shared = A^b mod p`
5. 使用 `(shared % 256)` 當作單 byte XOR key，加密 flag

題目的關鍵漏洞在於：`message.txt` 中直接洩漏了 client secret `b`。

## 三、分析目標
1. 由 `encryption.py` 理解加密流程。
2. 確認 `message.txt` 中哪些參數已知。
3. 利用洩漏的 `b` 重建 shared secret。
4. 還原 XOR key 並解出 flag。

## 四、原始碼分析

### 4.1 加密腳本核心邏輯
原始碼重要片段如下：

```python
g = 2
p = getPrime(1048)

a = randint(2, p-2)
A = pow(g, a, p)

b = '???'
B = pow(g, b, p)

shared = pow(A, b, p)

flag = b"picoCTF{...}"
enc = bytes([x ^ (shared % 256) for x in flag])
```

### 4.2 重大洩漏點
腳本最後會將 challenge info 寫入檔案，其中包含：

```python
f.write(f"b = {b} \n")
```

這表示本應保密的 client secret `b` 被直接寫進 `message.txt`，使共享祕密可被完整重建。

## 五、解題流程

### 5.1 從 `message.txt` 取得參數
`message.txt` 內包含：
- `g`
- `p`
- `A`
- `b`
- `enc`

由於 `b` 已直接外洩，因此可直接計算：

```python
shared = pow(A, b, p)
key = shared % 256
```

### 5.2 還原 XOR 金鑰
因為 flag 格式已知通常以 `picoCTF{` 開頭，也可以用 known-plaintext 方式快速驗證前幾個 byte，確認 XOR key 為固定單 byte。

最終得到 XOR key：

```text
0xe2
```

### 5.3 解密密文
使用該單 byte key 對整段 `enc` XOR 還原後，即可取得明文 flag。

## 六、解題結果
最終取得的 flag 為：

```text
picoCTF{dh_s3cr3t_32ec2679}
```

## 七、最短解法
由於已知 XOR key 為 `0xe2`，可直接解密：

```python
enc = bytes.fromhex("928b818da1b6a499868abd91d18190d196bdd1d08781d0d4d5db9f")
key = 0xe2
print(bytes(c ^ key for c in enc).decode())
```

輸出：

```text
picoCTF{dh_s3cr3t_32ec2679}
```

## 八、弱點分析
本題的根本問題是 **secret leakage**：
1. Diffie-Hellman 的安全性建立在 `a` 與 `b` 必須保密。
2. 一旦 `b` 被寫入輸出檔，shared secret 可直接算出。
3. 後續加密又只使用 `shared % 256` 作為單 byte XOR key，安全性進一步大幅下降。

## 九、防禦建議
1. 不得將私密指數 `a` / `b` 寫入日誌、輸出檔或 debug 訊息。
2. 不應將高熵 shared secret 簡化成單 byte XOR key。
3. 若需對稱加密，應使用經驗證的加密模式，如 AES-GCM、ChaCha20-Poly1305。
4. 開發流程中應移除 debug 資訊，避免機密材料外洩。

## 十、結論
`Shared Secrets` 是一道典型的密碼協定實作失誤題。雖然題目表面使用了 Diffie-Hellman 交換 shared secret，但由於私密參數 `b` 被直接洩漏，使整個安全性完全失效。最終只需重建 shared secret 或利用 known-plaintext 推回 XOR key，即可還原 flag。

## 十一、附錄：驗證腳本

```python
enc = bytes.fromhex("928b818da1b6a499868abd91d18190d196bdd1d08781d0d4d5db9f")
key = 0xe2
print(bytes(c ^ key for c in enc).decode())
```
