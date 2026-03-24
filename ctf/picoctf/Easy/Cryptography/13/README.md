# ROT13 解題教學：`cvpbPGS{abg_gbb_onq_bs_n_ceboyrz}`

## 題目觀察
題目文字提到：

> 密碼學其實很簡單，你知道 ROT13 是什麼嗎？

並給出一串看起來像 flag 的字串：

```text
cvpbPGS{abg_gbb_onq_bs_n_ceboyrz}
```

看到這種提示時，第一個要聯想到的就是 **ROT13**。

---

## 什麼是 ROT13？
ROT13（Rotate by 13 places）是一種非常基礎的字母替換編碼。

它的規則是：
- 英文字母往後移動 13 個位置
- 超過 `z` 或 `Z` 就從頭開始
- 只會影響英文字母
- 數字、底線、括號通常不會改變

例如：

- `a → n`
- `b → o`
- `c → p`
- `n → a`
- `p → c`

因此：

- `cvpbPGS` 經過 ROT13 之後會變成 `picoCTF`

這也是這題很明顯的線索，因為 `picoCTF{...}` 是常見的 flag 格式。

---

## 解題思路
我們直接把整串文字做 ROT13 轉換：

```text
cvpbPGS{abg_gbb_onq_bs_n_ceboyrz}
```

解碼後得到：

```text
picoCTF{not_too_bad_of_a_problem}
```

---

## 最終 Flag

```text
picoCTF{not_too_bad_of_a_problem}
```

---

## 如何手動判斷這是 ROT13？
在 CTF 中，若題目出現以下特徵，可以優先懷疑是 ROT13：

1. **題目明確提到 ROT13 或 rotation**
2. 字串看起來像英文，但又不是正常可讀單字
3. 解開後很可能出現常見旗標前綴，例如：
   - `picoCTF{...}`
   - `flag{...}`
4. 字母大小寫位置保留，但內容像是被整體平移

例如：

- `cvpbPGS` 看起來不像隨機亂碼
- 套用 ROT13 後變成 `picoCTF`
- 這就能立刻確認方向正確

---

## 實作方式

### 方法一：使用線上工具
搜尋 ROT13 decoder，將字串貼上即可解碼。

---

### 方法二：使用 Python
可以直接用 Python 內建的 `codecs` 模組：

```python
import codecs

cipher = "cvpbPGS{abg_gbb_onq_bs_n_ceboyrz}"
flag = codecs.decode(cipher, "rot_13")
print(flag)
```

輸出結果：

```text
picoCTF{not_too_bad_of_a_problem}
```

---

### 方法三：自行字母對照
也可以用 ROT13 對照表手動轉：

```text
A ↔ N
B ↔ O
C ↔ P
D ↔ Q
E ↔ R
F ↔ S
G ↔ T
H ↔ U
I ↔ V
J ↔ W
K ↔ X
L ↔ Y
M ↔ Z
```

小寫同理。

---

## 這題學到什麼？
這題重點不在複雜密碼學，而是在於：

- 辨識題目提示
- 了解 ROT13 的基本規則
- 知道如何快速驗證猜測

ROT13 是 CTF 中非常常見的入門編碼題，常常會搭配：
- 題目直接提示 ROT13
- 看似亂碼但其實是英文句子
- flag 前綴解出後非常明顯

所以之後看到這類型的字串時，可以先試：
- ROT13
- Caesar cipher
- Base64
- strings

---

## 解題流程總結

1. 讀題目提示，注意到關鍵字 **ROT13**
2. 觀察字串 `cvpbPGS{abg_gbb_onq_bs_n_ceboyrz}`
3. 對整串內容做 ROT13 解碼
4. 得到最終 flag：

```text
picoCTF{not_too_bad_of_a_problem}
```

---

## 結論
這是一題非常典型的 ROT13 入門題。

只要看到題目已經明示 ROT13，就可以直接對密文進行轉換。解出來的 `picoCTF{not_too_bad_of_a_problem}` 也進一步證明方向正確。

這類題目雖然簡單，但很適合熟悉 CTF 中常見的基礎編碼觀念。
