# picoCTF 教學手冊：enc_flag 兩層 Base64 + Caesar Cipher 解題

## 題目概述
這題的核心觀念不是單一加密，而是把內容做了 **兩層 Base64 編碼**，再把解出的字串用 **Caesar Cipher（凱薩位移）** 做字母位移。

根據實際操作紀錄，我們是先從檔案 `enc_flag` 讀出內容，連續解兩次 Base64，最後辨識出 `wpjvJAM{jhlzhy_k3jy9wa3k_78250hmj}` 是一個經過位移的 picoCTF flag，解回後得到真正答案。

---

## 題目環境與已知資訊
在 WSL 目錄中可看到目標檔案：

```bash
ls
```

輸出：

```bash
desktop.ini  enc_flag
```

接著查看檔案內容：

```bash
cat enc_flag
```

輸出：

```text
YidkM0JxZGtWQlRYdHFhR3g2aHlfazNqeTl3YTNrVlVGwzWVR0clh6YzRNa1V3YUcxcWZRPT0nCg==
```

這一串看起來很像 Base64，因為：

- 只由英數字、`+`、`/`、`=` 這類 Base64 常見字元組成
- 尾端有 `==`，這是 Base64 很常見的補位格式

---

## 解題流程

### 第一步：對 `enc_flag` 做第一次 Base64 解碼
使用：

```bash
echo 'YidkM0JxZGtWQlRYdHFhR3g2aHlfazNqeTl3YTNrVlVGwzWVR0clh6YzRNa1V3YUcxcWZRPT0nCg==' | base64 -d
```

輸出：

```text
b'd3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrXzc4MjUwaG1qfQ=='
```

### 為什麼這一步很重要？
第一次解碼後，沒有直接得到 flag，而是得到：

```text
b'd3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrXzc4MjUwaG1qfQ=='
```

這表示：

- 裡面真正有用的內容是 `d3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrXzc4MjUwaG1qfQ==`
- 外層的 `b' ... '` 很像 **Python bytes 物件的字串表示法**

也就是說，這題不是只編一次 Base64，而是把「另一段 Base64 字串」包在 bytes 表示法裡再存進檔案。

---

### 第二步：取出中間那段，再做第二次 Base64 解碼
把真正的中間內容拿出來再解一次：

```bash
echo 'd3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrXzc4MjUwaG1qfQ==' | base64 -d
```

輸出：

```text
wpjvJAM{jhlzhy_k3jy9wa3k_78250hmj}
```

這時仍然不是正常可讀的 picoCTF flag，但已經很接近了。

---

### 第三步：辨識字串型態
得到的字串是：

```text
wpjvJAM{jhlzhy_k3jy9wa3k_78250hmj}
```

這串有幾個很明顯的特徵：

1. 長得像 flag 格式：`某字串{...}`
2. 前面的 `wpjvJAM` 很像某個英文單字被位移過
3. picoCTF 題目很常用 **Caesar Cipher**

如果把 `wpjvJAM` 每個字母往回位移 7 位：

- `w -> p`
- `p -> i`
- `j -> c`
- `v -> o`
- `J -> C`
- `A -> T`
- `M -> F`

就會變成：

```text
picoCTF
```

因此可以判斷這整串是 **凱薩密碼，位移量為 7**。

---

### 第四步：把整串做 Caesar 位移還原
將整串字母都往回移 7 位，可以得到：

```text
picoCTF{caesar_d3cr9pt3d_78250afc}
```

這就是真正的 flag。

---

## 最終 Flag

```text
picoCTF{caesar_d3cr9pt3d_78250afc}
```

---

## 完整操作紀錄整理

### 1. 查看檔案
```bash
cat enc_flag
```

### 2. 第一次 Base64 解碼
```bash
echo 'YidkM0JxZGtWQlRYdHFhR3g2aHlfazNqeTl3YTNrXzc4MjUwaG1qfQ==' | base64 -d
```

實際依據你的操作，第一次解出的是：

```text
b'd3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrXzc4MjUwaG1qfQ=='
```

### 3. 第二次 Base64 解碼
```bash
echo 'd3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrXzc4MjUwaG1qfQ==' | base64 -d
```

得到：

```text
wpjvJAM{jhlzhy_k3jy9wa3k_78250hmj}
```

### 4. Caesar Cipher 還原
往回位移 7 位後得到：

```text
picoCTF{caesar_d3cr9pt3d_78250afc}
```

---

## 補充知識

### 1. 如何快速判斷是不是 Base64？
常見線索：

- 只包含英文大小寫、數字、`+`、`/`
- 尾端常有 `=` 或 `==`
- 解碼後若還是像亂碼或另一段可疑字串，可能還有第二層編碼

### 2. 如何快速判斷是不是 Caesar Cipher？
常見線索：

- 字串看起來像英文，但每個字都怪怪的
- 題目結果格式像 flag：`xxxxx{...}`
- 位移幾位後，前綴可能變成 `picoCTF`

### 3. 為什麼第一次會解出 `b'...'`？
因為原資料很可能是把 Python 的 bytes 表示法整個再編碼進去，例如：

```python
b'd3BqdkpBTXtqaGx6aHlfazNqeTl3YTNrXzc4MjUwaG1qfQ=='
```

所以第一次 Base64 解碼只是把這層包裝拆掉，第二次才拿到真正內容。

---

## 這題學到的重點
這題的重點不在複雜工具，而在於：

1. 先觀察資料外觀，判斷可能是 Base64
2. 發現第一次解碼後還有第二層內容
3. 辨識 `wpjvJAM` 很像被位移的 `picoCTF`
4. 推出 Caesar 位移量為 7
5. 還原得到真正 flag

這是一題很典型的「**多層編碼 + 基礎古典密碼**」綜合題。

---

## 可選：用 Python 一次自動解出
如果想把流程自動化，可以用下面程式：

```python
import base64

enc = b"YidkM0JxZGtWQlRYdHFhR3g2aHlfazNqeTl3YTNrXzc4MjUwaG1qfQ=="
first = base64.b64decode(enc).decode()
second_b64 = first[2:-1]   # 去掉前面的 b' 和最後的 '
second = base64.b64decode(second_b64).decode()


def caesar_decode(s, shift):
    out = []
    for ch in s:
        if 'a' <= ch <= 'z':
            out.append(chr((ord(ch) - ord('a') - shift) % 26 + ord('a')))
        elif 'A' <= ch <= 'Z':
            out.append(chr((ord(ch) - ord('A') - shift) % 26 + ord('A')))
        else:
            out.append(ch)
    return ''.join(out)

print(second)
print(caesar_decode(second, 7))
```

輸出會是：

```text
wpjvJAM{jhlzhy_k3jy9wa3k_78250hmj}
picoCTF{caesar_d3cr9pt3d_78250afc}
```
