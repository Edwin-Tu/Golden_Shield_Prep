# picoCTF Reverse Engineering Writeup
## 題目：16-bit 打包字元逆向還原

---

## 題目給的關鍵程式

```python
''.join([chr((ord(flag[i]) << 8) + ord(flag[i + 1])) for i in range(0, len(flag), 2)])
```

---

## 題目在做什麼？

這段程式的作用是：

> **把原本的 flag 每兩個字元合併成一個新的 Unicode 字元**

也就是把兩個 8-bit 字元，打包成一個 16-bit 整數，再轉成一個字元。

---

## 拆解原理

假設原本有兩個字元：

```python
flag[i] = 'p'
flag[i+1] = 'i'
```

先轉成 ASCII / Unicode 整數：

```python
ord('p') = 112
ord('i') = 105
```

題目做的事是：

```python
(ord(flag[i]) << 8) + ord(flag[i + 1])
```

也就是：

- 第一個字元放到高 8 位
- 第二個字元放到低 8 位

像這樣：

```python
(112 << 8) + 105
```

再把這個數字變成一個字元：

```python
chr(...)
```

所以原本兩個字元，最後會變成一個新的字元。

---

## 為什麼可以逆向？

因為它不是雜湊，也不是不可逆加密，
它只是單純做了 **位元拼接（bit packing）**。

如果加密時是：

```python
combined = (high << 8) + low
```

那解密時只要拆回：

- 高 8 位：`ord(c) >> 8`
- 低 8 位：`ord(c) & 0xff`

就能還原原始兩個字元。

---

## 逆向公式

```python
''.join(chr(ord(c) >> 8) + chr(ord(c) & 0xff) for c in enc)
```

---

## 題目中的密文

題目給的密文如下：

```text
灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸形㝦㘲捡㕽
```

---

## 解題程式

### 直接寫死密文版本

```python
enc = "灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸形㝦㘲捡㕽"
flag = ''.join(chr(ord(c) >> 8) + chr(ord(c) & 0xff) for c in enc)
print(flag)
```

---

### 從檔案讀取版本

```python
with open("enc", "r", encoding="utf-8") as f:
    enc = f.read().strip()

flag = ''.join(chr(ord(c) >> 8) + chr(ord(c) & 0xff) for c in enc)
print(flag)
```

---

## 執行結果

```text
picoCTF{16_bits_inst34d_of_8_b7f62ca5}
```

---

## 最終 Flag

```text
picoCTF{16_bits_inst34d_of_8_b7f62ca5}
```

---

## 這題學到的重點

### 1. `ord()` 與 `chr()` 的互轉
- `ord(c)`：字元轉整數
- `chr(n)`：整數轉字元

### 2. 位元運算
- `<< 8`：左移 8 位，相當於乘上 256
- `>> 8`：右移 8 位，取高位元組
- `& 0xff`：取低 8 位

### 3. 題型辨識
這類題目看到：

- `ord(...)`
- `chr(...)`
- `<<`
- `>>`
- 每兩個字元合併 / 拆開

就要立刻想到：

> 這很可能是 **字元打包 / 位元拆解** 題

---

## 一句話總結

這題不是加密學，而是：

> **把兩個字元包成一個字元，再把它拆回來。**

---

## 精簡版解法

```python
enc = "灩捯䍔䙻ㄶ形楴獟楮獴㌴摟潦弸形㝦㘲捡㕽"
print(''.join(chr(ord(c)>>8) + chr(ord(c)&0xff) for c in enc))
```
