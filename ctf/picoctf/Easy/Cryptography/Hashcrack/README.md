# Hashcrack— picoCTF 解題教學手冊

## 題目概述
本題是一道典型的 **Hash Crack / 弱密碼辨識** 題目。

題目會透過 `nc` 連到遠端服務，依序給出三組雜湊值（hash），要求玩家輸入對應的原始密碼。三組分別對應：

1. **MD5**
2. **SHA-1**
3. **SHA-256**

只要依序輸入正確密碼，就能拿到最終 flag。

---

## 題目重點
這題的核心不是「直接把 hash 反推回原文」，而是：

- 先觀察 hash 的長度與格式
- 判斷它可能是哪一種雜湊演算法
- 再用常見弱密碼進行猜測與驗證
- 最後把正確密碼輸入遠端服務

這種作法就是最基本的 **字典攻擊（Dictionary Attack）** 思維。

---

## 使用環境
本題使用 WSL / Linux 終端機進行操作。

### 用到的工具
- `nc`：連線到題目伺服器
- `md5sum`：驗證 MD5
- `sha1sum`：驗證 SHA-1
- `sha256sum`：驗證 SHA-256

---

## 連線方式
依照題目頁面提供的主機與連接埠執行：

```bash
nc verbal-sleep.picoctf.net 59397
```

> 注意：**port 可能會因實例重啟而改變**，請以題目頁面當下顯示的資訊為準。

如果看到類似以下畫面，表示連線成功：

```text
Welcome!! Looking For the Secret?
```

---

## 解題流程

## 第一關：辨識 MD5
題目給出的第一組 hash：

```text
482c811da5d5b4bc6d497ffa98491e38
```

### 觀察特徵
- 長度為 **32 個字元**
- 全部是十六進位字元（`0-9`, `a-f`）

這是很典型的 **MD5** 格式。

### 驗證方式
在終端機輸入：

```bash
echo -n "password123" | md5sum
```

輸出會是：

```text
482c811da5d5b4bc6d497ffa98491e38  -
```

和題目提供的 hash 完全一致，因此第一關答案為：

```text
password123
```

輸入後會顯示：

```text
Correct! You've cracked the MD5 hash with no secret found!
```

---

## 第二關：辨識 SHA-1
第二組 hash：

```text
b7a875fc1ea228b9061041b7cec4bd3c52ab3ce3
```

### 觀察特徵
- 長度為 **40 個字元**
- 也是十六進位字元

這是很典型的 **SHA-1**。

### 驗證方式
輸入：

```bash
echo -n "letmein" | sha1sum
```

輸出：

```text
b7a875fc1ea228b9061041b7cec4bd3c52ab3ce3  -
```

與題目一致，因此第二關答案為：

```text
letmein
```

輸入後會顯示：

```text
Correct! You've cracked the SHA-1 hash with no secret found!
```

---

## 第三關：辨識 SHA-256
第三組 hash：

```text
916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745
```

### 觀察特徵
- 長度為 **64 個字元**
- 為十六進位字元

這是很典型的 **SHA-256**。

### 驗證方式
輸入：

```bash
echo -n "qwerty098" | sha256sum
```

如果輸出為：

```text
916e8c4f79b25028c9e467f1eb8eee6d6bbdff965f9928310ad30a8d88697745  -
```

就代表第三關原文密碼為：

```text
qwerty098
```

輸入後即可取得 flag。

---

## 最終答案輸入順序
成功連線後，依序輸入以下三組密碼：

```text
password123
letmein
qwerty098
```

---

## 最終 Flag
根據實際解題結果，取得的 flag 為：

```text
picoCTF{UseStr0nG_h@shEs_&PaSswDs!_93e052d7}
```

---

## 為什麼可以知道密碼？
這裡要特別理解一個觀念：

**不是直接從 hash「算回去」原文。**

而是透過以下流程得出答案：

1. 觀察 hash 長度
2. 判斷演算法類型
3. 從常見弱密碼開始猜測
4. 使用對應工具驗證 hash 是否一致

這就是最基本的 hash cracking 思路。

例如：

- `32` 碼十六進位 → 常見是 `MD5`
- `40` 碼十六進位 → 常見是 `SHA-1`
- `64` 碼十六進位 → 常見是 `SHA-256`

---

## 常見驗證指令整理

### 驗證 MD5
```bash
echo -n "password123" | md5sum
```

### 驗證 SHA-1
```bash
echo -n "letmein" | sha1sum
```

### 驗證 SHA-256
```bash
echo -n "qwerty098" | sha256sum
```

---

## 為什麼 `echo` 要加 `-n`？
因為如果不加 `-n`，`echo` 會自動加上一個換行字元 `\n`。

也就是說：

```bash
echo "password123" | md5sum
```

實際上算的是：

```text
password123\n
```

這樣算出來的 hash 就會和題目不同。

因此驗證時要使用：

```bash
echo -n "password123" | md5sum
```

---

## 解題觀念補充

### 1. Hash 不是加密（Encryption）
Hash 是單向運算，理論上不能直接逆向還原。

### 2. 弱密碼容易被字典攻擊
像是以下這種密碼都很危險：

- `password`
- `password123`
- `123456`
- `letmein`
- `qwerty`

因為這些都常出現在字典檔中，很容易被快速撞出來。

### 3. 題目其實在提醒安全觀念
本題最終想傳達的是：

- 只用 hash 並不代表安全
- 若搭配弱密碼，仍然很容易被破解
- 應使用**強密碼**並搭配更安全的密碼儲存方式（如加鹽 salt 與現代密碼雜湊方案）

---

## 常見錯誤

### 1. 主機名稱拼錯
例如把：

```text
picoctf.net
```

打成別的字串，會導致：

```text
Name or service not known
```

### 2. 忘記使用 `echo -n`
會讓驗證結果不一致。

### 3. 以為 hash 可以直接反推出原文
實際上多數 CTF 題目是靠：

- 常見密碼猜測
- 字典攻擊
- 線上比對資料庫
- 工具驗證

來找出原文。

---

## 本題完整操作示範

```bash
nc verbal-sleep.picoctf.net 59397
```

接著依序輸入：

```text
password123
letmein
qwerty098
```

即可拿到 flag。

---

## 心得總結
這題雖然難度不高，但很適合練習以下能力：

- 從 hash 長度快速辨識演算法
- 了解弱密碼的風險
- 熟悉 Linux 下的 hash 驗證指令
- 建立字典攻擊的基本思維

對初學者來說，這是一題非常好的入門題。

---

## 速查表

| 長度 | 常見演算法 |
|------|------------|
| 32   | MD5        |
| 40   | SHA-1      |
| 64   | SHA-256    |

---

## 參考指令速查

```bash
# 連線
nc verbal-sleep.picoctf.net 59397

# 驗證 MD5
echo -n "password123" | md5sum

# 驗證 SHA-1
echo -n "letmein" | sha1sum

# 驗證 SHA-256
echo -n "qwerty098" | sha256sum
```
