# 🧩 Challenge: Flag in Flame

## 📌 題目敘述

在最近一次安全漏洞事件後，安全營運中心（SOC）團隊發現了一個異常龐大的日誌檔案。打開後，他們發現裡面並非普通的日誌，而是一大段程式碼文字。

這裡面是否隱藏著什麼秘密？你的任務是檢查這個文件，揭示它的真正用途。請從這份異常日誌中挖掘出任何隱藏的訊息。

📎 提供檔案：

* logs.txt

📌 提示：

* 使用 base64 解碼資料並產生圖像檔案 

---

## 📁 提供檔案分析

打開 `logs.txt` 後，可以看到內容如下（節錄）：

```
iVBORw0KGgoAAAANSUhEUgAAA4AAAASACAIAAAAh8bSO...
```

---

## 🧠 解題思路（逐步推理）

### 🔍 Step 1：觀察檔案內容

這個「日誌檔案」並不像正常 log，反而是一長串無意義字元。

觀察特徵：

* 由 A–Z、a–z、0–9、`+`、`/` 組成
* 字串非常長
* 幾乎沒有空格或可讀文字

👉 這是 **base64 編碼的典型特徵**

---

### 🧩 Step 2：關鍵線索辨識

開頭：

```
iVBORw0K
```

這是一個非常重要的特徵！

👉 這代表：

* 很可能是 **PNG 圖片的 base64 編碼**

---

### 💡 Step 3：推測解法

根據提示與觀察：

* 資料是 base64 編碼
* 解碼後應該是一個圖片

👉 解題方向：

**將 logs.txt 進行 base64 解碼 → 還原圖片**

---

## 🛠 解題步驟（實作教學）

### ✅ 方法一：Linux / WSL

```bash
base64 -d logs.txt > output.png
```

---

### ✅ 方法二：Python

```python
import base64

with open("logs.txt", "r") as f:
    data = f.read()

decoded = base64.b64decode(data)

with open("output.png", "wb") as f:
    f.write(decoded)
```

---

### ✅ 方法三：線上工具

使用：
https://base64.guru/converter/decode

步驟：

1. 貼上 logs.txt 內容
2. 點擊 decode
3. 下載輸出檔案

---

## 🖼 結果驗證

解碼後得到：

```
output.png
```

可以使用以下指令確認：

```bash
file output.png
```

若顯示：

```
PNG image data
```

👉 表示成功還原圖片

打開圖片後即可看到 flag。

---

## 🚩 Flag

```
（在圖片中取得）
```

---

## 📚 延伸知識（重要加分）

### 🔹 什麼是 Base64？

Base64 是一種將「二進位資料」轉為「文字」的編碼方式，常見用途：

* 圖片嵌入 HTML
* Email 傳輸
* CTF 隱藏資料

---

### 🔹 如何快速辨識 Base64？

特徵：

* 字元只包含：

  * A–Z / a–z / 0–9 / + /
* 通常以 `=` 結尾（padding）
* 字串很長且無空格

---

### 🔹 PNG 檔案 magic number

PNG 檔案開頭（hex）：

```
89 50 4E 47
```

轉成 base64：

```
iVBORw0K
```

👉 因此看到 `iVBORw0K` 幾乎可以確定是 PNG

---

### 🔹 常見 CTF 類型（本題屬於）

* Encoding / Decoding
* File Reconstruction
* Steganography（延伸）

---

## 💡 解題心得

這題的關鍵在於：

* 能辨識 base64 特徵
* 能從開頭判斷檔案格式（PNG）

👉 在 CTF 中，**辨識資料型態比工具更重要**

---

## 🧠 技巧總結（必記）

* `iVBORw0K` → PNG
* base64 長字串 → 優先嘗試 decode
* log 檔不一定是 log（常為偽裝）

---

## 🚀 延伸練習建議

* 嘗試將圖片再用 `strings` 或 `binwalk` 分析
* 練習其他 encoding（hex / rot13 / gzip）

---
