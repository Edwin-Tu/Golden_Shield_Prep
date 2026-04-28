# picoCTF 2026 - bytemancy 0 Writeup

## 題目資訊

- **題目名稱：** bytemancy 0
- **平台：** picoCTF 2026
- **分類：** General Skills
- **難度：** Easy
- **連線方式：**

```bash
nc candy-mountain.picoctf.net 57670
```

## 題目描述

題目要求：

```text
Send me ASCII DECIMAL 101, 101, 101, side-by-side, no space.
```

意思是要輸入三個 ASCII 十進位 `101` 對應的字元，而且要連在一起，中間不能有空格。

## 原始碼分析

題目提供的 Python 程式核心判斷如下：

```python
user_input = input('==> ')
if user_input == "\x65\x65\x65":
  print(open("./flag.txt", "r").read())
  break
```

其中：

```text
\x65
```

代表十六進位 `0x65`。

`0x65` 轉成十進位是 `101`，在 ASCII 表中對應的字元是：

```text
e
```

所以：

```python
"\x65\x65\x65"
```

實際上等於：

```text
eee
```

## 解題觀念

題目說：

```text
ASCII DECIMAL 101, 101, 101
```

查 ASCII 對照：

| Decimal | Hex  | Character |
|---|---|---|
| 101 | 0x65 | e |

因此要輸入：

```text
eee
```

注意不是輸入：

```text
101101101
```

也不是輸入：

```text
\x65\x65\x65
```

而是輸入三個字母 `e`。

## 實際操作紀錄

連線：

```bash
nc candy-mountain.picoctf.net 57670
```

程式顯示：

```text
⊹──────[ BYTEMANCY-0 ]──────⊹

Send me ASCII DECIMAL 101, 101, 101, side-by-side, no space.

==>
```

輸入：

```text
eee
```

成功取得 flag：

```text
picoCTF{pr1n74813_ch4r5_4daf27d8}
```

## 一行解法

也可以直接用 `echo` 搭配 `nc`：

```bash
echo "eee" | nc candy-mountain.picoctf.net 57670
```

或用 Python 產生字串：

```bash
python3 -c 'print("e"*3)' | nc candy-mountain.picoctf.net 57670
```

## 常見錯誤

### 錯誤 1：輸入十進位數字本身

錯誤：

```text
101101101
```

原因：題目要的是 ASCII decimal 101 所代表的字元，不是數字字串。

正確：

```text
eee
```

### 錯誤 2：輸入 Python 轉義字串

錯誤：

```text
\x65\x65\x65
```

原因：`\x65` 是程式碼中的表示法，真正的字元是 `e`。

正確：

```text
eee
```

### 錯誤 3：中間加空格或逗號

錯誤：

```text
e e e
```

或：

```text
e,e,e
```

原因：題目明確要求 `side-by-side, no space`，也就是連在一起、沒有空格。

正確：

```text
eee
```

## 總結

這題主要考的是：

1. 理解 ASCII decimal 與字元的對應關係。
2. 知道 decimal `101` 對應字元 `e`。
3. 看懂 Python 中的 `\x65` 代表十六進位字元。
4. 將三個 `e` 連續輸入即可取得 flag。

最終答案輸入：

```text
eee
```
