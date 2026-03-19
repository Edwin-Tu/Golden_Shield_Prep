# 🕵️‍♂️ CTF Writeup：JPG 隱寫術（Steganography）

## 📌 題目描述

你會得到一張看似普通的 JPG 圖片，但圖片內部隱藏著一些不為人知的資料。  
你的任務是找出隱藏的有效載荷並提取出 flag。

---

## 🧠 解題思路

這題屬於典型的：

> 📂 Steganography（隱寫術） + Metadata + Base64

---

## 🧰 使用工具

- `file`：檢查檔案型態
- `binwalk`：掃描是否有嵌入檔案
- `strings`：提取可讀字串
- `exiftool`：查看圖片 metadata
- `base64`：解碼
- `steghide`：提取隱藏資料

---

## �️ 工具說明

### binwalk
`binwalk` 是一個用於分析和提取嵌入在檔案中的其他檔案的工具。它可以掃描檔案的二進制內容，識別出可能的嵌入檔案，如 ZIP、PNG 等。  
**安裝方式**：`sudo apt install binwalk`（Ubuntu/Debian）或 `brew install binwalk`（macOS）。  
**基本用法**：`binwalk <檔案名稱>`，用於掃描檔案結構。

### steghide
`steghide` 是一個用於在圖片或音訊檔案中隱藏和提取資料的隱寫術工具。它使用密碼保護隱藏的資料。  
**安裝方式**：`sudo apt install steghide`（Ubuntu/Debian）或 `brew install steghide`（macOS）。  
**基本用法**：
- 隱藏資料：`steghide embed -cf <載體檔案> -ef <隱藏檔案> -p <密碼>`
- 提取資料：`steghide extract -sf <載體檔案> -p <密碼>`

---

## 🔍 Step 1：檢查檔案型態

```bash
file img.jpg
```

**輸出：**

```
JPEG image data, JFIF standard 1.01
```

✔ 確認為正常 JPEG 檔案

---

## 🔍 Step 2：使用 binwalk 掃描

```bash
binwalk img.jpg
```

**輸出：**

```
DECIMAL       HEXADECIMAL     DESCRIPTION
-------------------------------------------------------
0             0x0             JPEG image data
```

✔ 沒有發現內嵌檔案（如 zip、png）

👉 判斷：不是 binwalk 類型題

---

## 🔍 Step 3：查看 metadata（關鍵步驟）

```bash
exiftool img.jpg
```

**關鍵輸出：**

```
Comment : c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9
```

👉 發現疑似 Base64 字串

---

## 🧪 Step 4：Base64 解碼（第一層）

```bash
echo "c3RlZ2hpZGU6Y0VGNmVuZHZjbVE9" | base64 -d
```

**輸出：**

```
steghide:cEF6endvcmQ=
```

👉 得到：

- 工具名稱：steghide
- 密碼（還需再解碼）

---

## 🧪 Step 5：Base64 解碼（第二層）

```bash
echo "cEF6endvcmQ=" | base64 -d
```

**輸出：**

```
p@zword
```

👉 最終密碼：p@zword

---

## 🔐 Step 6：使用 steghide 提取隱藏資料

```bash
steghide extract -sf img.jpg
```

**輸入密碼：**

```
p@zword
```

---

## 📦 Step 7：取得隱藏檔案

成功後會顯示：

```
wrote extracted data to "flag.txt"
```

---

## 🏁 Step 8：取得 flag

```bash
cat flag.txt
```

🎉 成功取得 flag！

---

## 🧠 總結

本題完整解題流程如下：

JPEG 圖片
   ↓
metadata（exiftool）
   ↓
Base64（雙層）
   ↓
取得 steghide 密碼
   ↓
steghide extract
   ↓
flag
💡 重點觀念整理
🔸 1. Metadata 是重要藏資料位置

Comment

Author

Description

🔸 2. Base64 常見特徵

結尾有 =

看起來像亂碼但有規律

🔸 3. 常見題型 pattern
metadata → base64 → 工具:密碼 → stego 解密
🔸 4. binwalk 沒東西 ≠ 沒藏東西

👉 要換方向思考（metadata / stego）

🚀 延伸練習

你可以嘗試：

使用 strings 找 base64

使用 xxd 分析檔案尾巴

嘗試 steghide info 查看是否有隱藏資料

🏆 結語

這題屬於：

👉 ⭐ 初中階 Steganography 綜合題

掌握以下技能：

metadata 分析

base64 判斷與解碼

steghide 使用

就能穩定解出此類題目 💪