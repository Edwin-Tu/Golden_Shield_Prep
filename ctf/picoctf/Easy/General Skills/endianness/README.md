# picoCTF 2024 - endianness 解題教學

## 題目資訊

- 題目名稱：endianness
- 分類：General Skills
- 難度：Easy
- 題目描述：

```text
Know of little and big endian?
```

連線方式：

```bash
nc titan.picoctf.net 54054
```

---

## 題目目標

連線到伺服器後，程式會隨機給出一個 5 個字母的單字，例如：

```text
Word: snexs
```

我們需要分別輸入：

1. Little Endian representation
2. Big Endian representation

兩個都答對後，即可取得 flag。

---

## 程式邏輯分析

根據題目提供的原始碼，程式會先隨機產生一個 5 個小寫字母組成的單字，接著分別要求使用者輸入 Little Endian 與 Big Endian 的表示法。

其中 Big Endian 的處理方式是按照原本字串順序，將每個字元轉成 ASCII 十六進位。Little Endian 則是將字串順序反過來後，再轉成 ASCII 十六進位。

---

## Endian 觀念說明

假設題目給的單字是：

```text
snexs
```

每個字母對應的 ASCII 十六進位如下：

| 字母 | ASCII Hex |
|---|---|
| s | 73 |
| n | 6E |
| e | 65 |
| x | 78 |
| s | 73 |

---

## Big Endian

Big Endian 是照原本順序排列。

原字串：

```text
s n e x s
```

轉成十六進位：

```text
73 6E 65 78 73
```

合併後得到：

```text
736E657873
```

---

## Little Endian

Little Endian 是將字元順序反過來。

原字串：

```text
s n e x s
```

反過來：

```text
s x e n s
```

轉成十六進位：

```text
73 78 65 6E 73
```

合併後得到：

```text
7378656E73
```

---

## 實際操作紀錄

使用 `nc` 連線到題目伺服器：

```bash
nc titan.picoctf.net 54054
```

伺服器回應：

```text
Welcome to the Endian CTF!
You need to find both the little endian and big endian representations of a word.
If you get both correct, you will receive the flag.
Word: snexs
Enter the Little Endian representation:
```

一開始誤輸入原字串：

```text
Snexs
```

結果錯誤：

```text
Incorrect Little Endian representation. Try again!
```

接著又誤輸入十進位 ASCII：

```text
115 110 101 120 115
```

以及：

```text
115110101120115
```

這些都錯，因為題目要求的是 **十六進位 Hex**，不是十進位 Decimal。

---

## 正確答案

題目給的 word 是：

```text
snexs
```

Little Endian：

```text
7378656E73
```

Big Endian：

```text
736E657873
```

輸入後結果如下：

```text
Enter the Little Endian representation: 7378656E73
Correct Little Endian representation!
Enter the Big Endian representation: 736E657873
Correct Big Endian representation!
Congratulations! You found both endian representations correctly!
Your Flag is: picoCTF{3ndi4n_sw4p_su33ess_02999450}
```

---

## Flag

```text
picoCTF{3ndi4n_sw4p_su33ess_02999450}
```

---

## 快速解法

可以使用 Python 快速轉換。

假設題目給：

```text
snexs
```

執行：

```bash
python3 -c 'w="snexs"; print("Little:", w[::-1].encode().hex().upper()); print("Big:", w.encode().hex().upper())'
```

輸出：

```text
Little: 7378656E73
Big: 736E657873
```

---

## Python 輔助腳本

也可以建立一個簡單的工具：

```python
word = input("Word: ")

little_endian = word[::-1].encode().hex().upper()
big_endian = word.encode().hex().upper()

print("Little Endian:", little_endian)
print("Big Endian:", big_endian)
```

執行方式：

```bash
python3 endian.py
```

輸入題目給的單字後，就能快速得到 Little Endian 與 Big Endian。

---

## 解題重點整理

1. 題目要的是 ASCII 的十六進位表示法。
2. 不要輸入原字串。
3. 不要輸入十進位 ASCII。
4. Big Endian 是原本順序。
5. Little Endian 是反向順序。
6. 輸入時不要加空格，直接輸入完整 Hex 字串。

---

## 本題學到的觀念

這題主要考察 Endianness 的基本概念。

在電腦系統中，Endian 代表資料在記憶體中的儲存順序：

- Big Endian：高位元組放在前面
- Little Endian：低位元組放在前面

在這題中，可以簡化理解成：

```text
Big Endian    = 字串原順序轉 Hex
Little Endian = 字串反過來轉 Hex
```
