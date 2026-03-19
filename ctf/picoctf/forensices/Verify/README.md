# picoCTF 2024 - Verify

## 題目類型
- Forensics
- shell / grep / sha256sum

## 題目目標
題目提供：
- 一個 `checksum.txt`
- 一個解密腳本 `decrypt.sh`
- 一個放了許多檔案的 `files/` 目錄

你的任務是：
1. 找出 `files/` 目錄中，哪一個檔案的 **SHA-256 雜湊值** 與 `checksum.txt` 相同
2. 再用 `decrypt.sh` 去解密那個正確的檔案
3. 取得 flag

---

## 題目提示整理
從題目畫面可知：

- 題目已經給你正確檔案的 SHA-256 值
- 題目明確提示你執行：

```bash
./decrypt.sh files/<file>
```

這代表這題的關鍵不是自己研究加密演算法，而是：

- 先驗證哪個檔案是真的
- 再把正確檔案交給腳本處理

---

## 解題觀念
這題主要練習三個 Linux 指令：

- `cat`：查看檔案內容
- `sha256sum`：計算檔案 SHA-256 雜湊值
- `grep`：從大量輸出中找出符合條件的結果

核心思路如下：

1. 讀出 `checksum.txt` 裡的雜湊值
2. 對 `files/` 底下所有檔案做 `sha256sum`
3. 用 `grep` 比對與 `checksum.txt` 相同的那一筆
4. 找到正確檔名後，執行 `./decrypt.sh 檔名`

---

## 你的操作中卡住的地方
你先前輸入了：

```bash
echo decrypt.sh | ASCII -d
```

這條指令有兩個問題：

### 1. `ASCII` 不是 Linux 指令
系統回傳：

```bash
-bash: ASCII: command not found
```

代表系統裡根本沒有這個指令。

### 2. `echo decrypt.sh` 只是輸出字串，不是讀檔
這條指令只會把文字 `decrypt.sh` 印出來，不會查看 `decrypt.sh` 的內容。

如果你想看腳本內容，應該用：

```bash
cat decrypt.sh
```

或：

```bash
less decrypt.sh
```

---

## 正確解題步驟

### Step 1：確認目前有哪些檔案
```bash
ls
```

你會看到類似：

```bash
checksum.txt  decrypt.sh  files
```

---

### Step 2：查看 checksum
```bash
cat checksum.txt
```

你會得到一串 SHA-256 值，例如：

```text
03b52eabed517324828b9e09cbbf8a7b0911f348f76cf989ba6d51acede6d5d8
```

---

### Step 3：對 `files/` 中所有檔案計算 SHA-256
```bash
sha256sum files/*
```

這會輸出很多行，每一行格式大致如下：

```bash
<sha256值>  <檔案名稱>
```

但因為輸出很多，直接閱讀不方便，所以要搭配 `grep`。

---

### Step 4：用 `grep` 找出符合 checksum 的檔案
```bash
sha256sum files/* | grep 03b52eabed517324828b9e09cbbf8a7b0911f348f76cf989ba6d51acede6d5d8
```

如果比對成功，會出現類似：

```bash
03b52eabed517324828b9e09cbbf8a7b0911f348f76cf989ba6d51acede6d5d8  files/00011a60
```

這表示：

- 正確檔案就是 `files/00011a60`

---

### Step 5：使用解密腳本處理正確檔案
```bash
./decrypt.sh files/00011a60
```

執行後就可以得到 flag。

---

## 一行解法
也可以把 `checksum.txt` 的內容直接帶進去：

```bash
sha256sum files/* | grep "$(cat checksum.txt)"
```

這樣就不用手動複製 checksum。

找到正確檔案後再執行：

```bash
./decrypt.sh files/00011a60
```

---

## 預期 flag
```text
picoCTF{trust_but_verify_00011a60}
```

---

## 完整指令流程
以下是本題可以直接照著做的一組流程：

```bash
ls
cat checksum.txt
cat decrypt.sh
sha256sum files/* | grep "$(cat checksum.txt)"
./decrypt.sh files/00011a60
```

---

## 額外補充

### `file decrypt.sh` 是做什麼？
你有使用：

```bash
file decrypt.sh
```

它的用途是判斷檔案型態，例如會顯示：

```bash
decrypt.sh: ASCII text
```

這表示它是一個文字檔，通常是 shell script。

但要注意：
- `file` 只能告訴你檔案類型
- **不能幫你解題內容本身**
- 真正重要的是 `cat decrypt.sh` 看它怎麼處理輸入檔案

---

## 這題學到的重點

### 1. 題目提示通常已經給了主要方向
這題直接告訴你：
- 有 checksum
- 有 decrypt script
- 要解密某個檔案

所以思路就是「先驗證，再解密」。

### 2. 雜湊值可以用來驗證檔案真偽
相同的 SHA-256 幾乎可以視為同一份檔案內容，因此可以用來確認哪個檔案是正確目標。

### 3. `grep` 很適合從大量輸出中找答案
當 `sha256sum files/*` 輸出太多行時，`grep` 可以快速鎖定正確結果。

### 4. 看檔案內容要用 `cat`，不是 `echo`
- `echo`：輸出你手動輸入的字串
- `cat`：讀取檔案內容

這是初學 Linux 時很常見的差異。

---

## 總結
這題屬於非常典型的初階 CTF 題：

- 不需要複雜逆向
- 不需要自己實作解密
- 重點在看懂題目提示、使用 Linux 基本指令、把流程串起來

解題流程可以濃縮成一句話：

> 先用 `sha256sum` + `grep` 找到與 `checksum.txt` 相符的檔案，再把該檔案交給 `decrypt.sh` 解密。

---

## 參考操作摘要
```bash
cat checksum.txt
sha256sum files/* | grep "$(cat checksum.txt)"
./decrypt.sh files/00011a60
```

---

## Flag
```text
picoCTF{trust_but_verify_00011a60}
```
