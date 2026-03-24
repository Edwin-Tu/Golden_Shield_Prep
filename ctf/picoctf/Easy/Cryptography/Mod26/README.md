# picoCTF 教學手冊：ROT13 Flag 解碼

## 題目概述

這一題提供了一個檔案 `values.txt`，檔案內容是一串看起來像 flag 的字串：

```text
cvpbPGS{arkg_gvzr_V'yy_gel_2_ebhaqf_bs_ebg13_45559noq}
```

這類題目常見於 **Cryptography / Encoding** 類型，重點通常不在爆破，而是在辨識字串的轉換方式。

---

## 觀察重點

先看字串前綴：

```text
cvpbPGS
```

在 picoCTF 題目中，flag 幾乎都會以：

```text
picoCTF{...}
```

開頭。

因此，當看到 `cvpbPGS` 時，可以懷疑它可能是 `picoCTF` 經過某種簡單位移加密後的結果。

---

## 判斷加密方式：ROT13

這題最適合先猜 **ROT13**。

### 什麼是 ROT13？

ROT13 是一種特殊的凱薩密碼（Caesar Cipher），規則是：

- 每個英文字母往後位移 13 個位置
- 超過 `z` 或 `Z` 就從頭繞回
- 大小寫分開處理
- 數字、底線、符號通常不變

例如：

- `c -> p`
- `v -> i`
- `p -> c`
- `b -> o`
- `P -> C`
- `G -> T`
- `S -> F`

所以：

```text
cvpbPGS -> picoCTF
```

只要這一段對上，幾乎就能確定整串是用 ROT13 加密。

---

## 解題方式一：手動推算

原始內容：

```text
cvpbPGS{arkg_gvzr_V'yy_gel_2_ebhaqf_bs_ebg13_45559noq}
```

對整串做 ROT13 後可得：

```text
picoCTF{next_time_I'll_try_2_rounds_of_rot13_45559abd}
```

---

## 解題方式二：使用 Linux 指令

在 WSL / Linux 終端機中，可以直接用 `tr` 進行 ROT13 轉換：

```bash
cat values.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

如果只想直接測試字串，也可以：

```bash
echo 'cvpbPGS{arkg_gvzr_V'
```

上面這種寫法會因為字串中含有單引號 `'` 而出現問題，所以建議改用雙引號：

```bash
echo "cvpbPGS{arkg_gvzr_V'yy_gel_2_ebhaqf_bs_ebg13_45559noq}" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

執行後就會得到：

```text
picoCTF{next_time_I'll_try_2_rounds_of_rot13_45559abd}
```

---

## 解題方式三：使用 Python

也可以用 Python 快速解碼：

```python
import codecs

s = "cvpbPGS{arkg_gvzr_V'yy_gel_2_ebhaqf_bs_ebg13_45559noq}"
print(codecs.decode(s, 'rot_13'))
```

輸出結果：

```text
picoCTF{next_time_I'll_try_2_rounds_of_rot13_45559abd}
```

---

## 為什麼這題要先想到 ROT13？

因為它有幾個很明顯的特徵：

1. 字串格式很像 flag
   - 有大括號 `{}`
   - 內容由英文字母、數字、底線組成

2. 開頭 `cvpbPGS` 很像 `picoCTF`
   - 這在 picoCTF 題目裡是很常見的提示

3. 字母分布自然
   - 不像隨機亂碼，反而像單純位移過的英文

當你看到類似：

```text
cvpbPGS
```

通常就可以優先猜：

- ROT13
- Caesar cipher

---

## 最終答案

```text
picoCTF{next_time_I'll_try_2_rounds_of_rot13_45559abd}
```

---

## 這題學到的重點

### 1. 看到 `cvpbPGS` 要想到 `picoCTF`
這是很多 picoCTF ROT13 題的典型特徵。

### 2. ROT13 是 Caesar cipher 的特例
位移量固定為 13，因此解碼與加密使用同一套規則。

### 3. 題目不一定需要複雜工具
有些題目只需要觀察、判斷格式，再配合簡單指令就能解出。

---

## 延伸練習

之後如果看到類似這種字串，也可以優先檢查：

- 是否是 ROT13
- 是否是一般 Caesar 位移
- 是否是 Base64
- 是否是十六進位 / ASCII 轉換

這些都是 CTF 初學者很常遇到的編碼與古典密碼題型。
