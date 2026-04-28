# picoCTF 2026 - bytemancy 1 解題教學

## 題目資訊

- **題目名稱**：bytemancy 1
- **平台**：picoCTF 2026
- **分類**：General Skills
- **難度**：Easy
- **目標**：根據程式要求，送出正確的 bytes / 字元內容以取得 flag。

---

## 題目描述

題目提示：

> Can you conjure the right bytes? The program's source code can be downloaded here.

意思是這題會提供原始碼，我們需要閱讀程式邏輯，找出它要求輸入什麼內容。

---

## 原始碼重點分析

程式會不斷顯示提示，要求使用者輸入：

```text
Send me ASCII DECIMAL 101 1751 times, side-by-side, no space.
```

接著程式讀取使用者輸入：

```python
user_input = input('==> ')
```

真正的判斷條件是：

```python
if user_input == "\x65"*1751:
    print(open("./flag.txt", "r").read())
    break
```

這代表只要輸入內容等於 `"\x65" * 1751`，程式就會印出 `flag.txt`。

---

## 關鍵概念：ASCII DECIMAL 101 是什麼？

ASCII decimal `101` 對應的字元是：

```text
e
```

而 Python 裡的：

```python
"\x65"
```

代表十六進位 `0x65`，也就是十進位 `101`，同樣是字母 `e`。

所以：

```python
"\x65" * 1751
```

等於：

```text
1751 個 e，全部連在一起，中間不能有空格
```

---

## 錯誤觀念

不要直接輸入：

```text
"\x65"*1751
```

因為這只是 Python 表達式的文字，不是程式要的答案。

程式真正要的是這個表達式「執行後的結果」，也就是：

```text
eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee...
```

總長度必須是 1751 個 `e`。

---

## 本機測試方式

如果你有下載 `app.py`，可以先在本機建立一個測試用的 `flag.txt`：

```bash
echo 'picoCTF{test_flag}' > flag.txt
```

執行程式：

```bash
python3 app.py
```

手動貼上 1751 個 `e` 很容易出錯，所以建議用 Python 自動產生輸入：

```bash
python3 -c 'print("e"*1751)' | python3 app.py
```

如果成功，會看到 `flag.txt` 的內容。

---

## 遠端解題方式

題目啟動 instance 後，通常會提供類似這樣的連線方式：

```bash
nc HOST PORT
```

將 `HOST` 和 `PORT` 換成題目實際給的主機與埠號。

解題指令：

```bash
python3 -c 'print("e"*1751)' | nc HOST PORT
```

範例格式：

```bash
python3 -c 'print("e"*1751)' | nc example.picoctf.net 12345
```

成功後，遠端程式會輸出 flag。

---

## 檢查輸入長度

如果想確認自己產生的字串長度是否正確，可以使用：

```bash
python3 -c 'import sys; sys.stdout.write("e"*1751)' | wc -c
```

正確結果應該是：

```text
1751
```

注意：

```bash
python3 -c 'print("e"*1751)' | wc -c
```

會得到：

```text
1752
```

因為 `print()` 會多輸出一個換行字元。不過遠端程式使用 `input()` 讀取輸入時，最後的 Enter / 換行只代表送出輸入，不會算進 `user_input` 裡，所以用 `print("e"*1751)` 搭配 `nc` 是可行的。

---

## 完整解題指令整理

### 1. 下載或查看原始碼後，確認條件

```python
if user_input == "\x65"*1751:
```

### 2. 產生 1751 個 e

```bash
python3 -c 'print("e"*1751)'
```

### 3. 傳給遠端服務

```bash
python3 -c 'print("e"*1751)' | nc HOST PORT
```

### 4. 取得 flag

遠端程式會印出：

```text
picoCTF{...}
```

---

## 解題重點總結

這題主要考：

1. 能看懂 Python 字串表示法。
2. 知道 ASCII decimal `101` 是字母 `e`。
3. 知道 `"\x65"` 也是字母 `e`。
4. 知道 `* 1751` 代表重複 1751 次。
5. 能用指令自動產生長字串並送給遠端服務。

一句話解法：

```bash
python3 -c 'print("e"*1751)' | nc HOST PORT
```

---

## 常見問題

### Q1：為什麼不能輸入 `"\x65"*1751`？

因為遠端程式不是在執行你輸入的 Python 程式碼，它只是把你的輸入當成一般文字比較。它要的是 1751 個 `e`，不是 Python 表達式本身。

### Q2：為什麼 `\x65` 是 `e`？

`\x65` 是十六進位表示法，`0x65` 等於十進位 `101`，而 ASCII 編碼中十進位 `101` 對應字母 `e`。

### Q3：如果 `nc` 沒反應怎麼辦？

先確認 instance 已經啟動，並且 `HOST`、`PORT` 是題目目前提供的最新資訊。picoCTF 的動態 instance 可能會過期，需要重新啟動。
