# 📚 CTF 教學：檔案分析與 Metadata 解題（以 PDF 為例）

---

嗨，勇敢的偵探！📄🔍 你偶然發現了一份奇怪的 PDF 文件，裡面似乎全是亂碼。但要小心！事情並非表面看起來那麼簡單。在這片混亂之中，隱藏著一份寶藏——一面等待你發現的神秘旗幟。

在此處找到 PDF 文件「隱藏的機密文件」，並在元資料中發現標記。

**附加檔案：** `confidential.pdf`

## 🧩 一、題目類型說明

本題屬於 **檔案分析（Forensics）** 類型，常見特徵：

- 檔案看似正常（PDF / JPG / PNG）
- 內容沒有明顯 flag
- flag 藏在：
  - metadata（中繼資料）
  - 編碼字串（Base64 / Hex）
  - 或隱寫術（steganography）

---

## 🧠 二、標準解題流程（非常重要）

當你拿到檔案時，請固定執行以下步驟：

```bash
file 檔案
strings 檔案
exiftool 檔案
binwalk 檔案
```

👉 這是 CTF 檔案題的「黃金流程」

---

## 🛠️ 三、工具教學

### 🔍 1. file（檔案類型辨識）

**功能：**  
確認檔案真實格式（避免被副檔名騙）。

**指令：**  
```bash
file confidential.pdf
```

**輸出範例：**  
```
PDF document, version 1.4
```

**重點：**  
- `.pdf` 不一定是 PDF
- `.jpg` 可能藏 zip  
👉 第一步一定要做

### 🔎 2. strings（抽取可讀字串）

**功能：**  
從二進位檔案中抽出「人類可讀文字」。

**指令：**  
```bash
strings confidential.pdf
```

或搭配分頁：  
```bash
strings confidential.pdf | less
```

**觀察重點：**  
找以下內容：  
- `flag`
- `picoCTF`
- `/Author`
- `/Title`
- 長字串（像亂碼）

**範例：**  
```
/Author (cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV84N2JlNjBjMH0=)
```

👉 這就是關鍵線索！

### 🧾 3. exiftool（Metadata 查看神器）

**功能：**  
查看檔案的中繼資料（metadata）。

**安裝（WSL）：**  
```bash
sudo apt update
sudo apt install libimage-exiftool-perl
```

**使用方式：**  
```bash
exiftool confidential.pdf
```

**輸出範例：**  
```
Author : cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV84N2JlNjBjMH0=
```

**進階指令：**  
```bash
exiftool -a -u -g1 confidential.pdf
```

👉 顯示更多隱藏欄位（CTF 常用）

### 🧱 4. binwalk（檔案內嵌分析）
📌 功能

分析檔案內是否藏有其他檔案（例如 zip、png）

📌 安裝
sudo apt install binwalk
📌 使用方式
binwalk confidential.pdf
📌 如果有東西
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
1234          0x4D2           Zip archive data

👉 表示裡面藏東西！

📌 解壓內嵌檔案
binwalk -e confidential.pdf
🔐 四、Base64 判斷與解碼（重點）
📌 如何判斷 Base64？

觀察字串：

cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV84N2JlNjBjMH0=
✔ 判斷特徵：
特徵	說明
有 = 結尾	Base64 padding
字元為英數 + +/	合法 Base64
長度為 4 的倍數	編碼規則

👉 符合 → 幾乎就是 Base64

📌 解碼方式
方法 1：Linux
echo cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV84N2JlNjBjMH0= | base64 -d
方法 2：Python
import base64

s = "cGljb0NURntwdXp6bDNkX20zdGFkYXRhX2YwdW5kIV84N2JlNjBjMH0="
print(base64.b64decode(s).decode())
🚩 五、取得 Flag

解碼後得到：

picoCTF{puzzl3d_m3tadata_f0und!_87be60c0}
🧠 六、解題關鍵整理
📌 1. 不要只看檔案內容

PDF ≠ 只有文字

圖片 ≠ 只有畫面

👉 metadata 很常藏 flag

📌 2. 編碼識別能力
編碼	特徵
Base64	有 =
Hex	0-9 + a-f
ROT13	看似英文但怪
📌 3. 工具使用順序
file
strings
exiftool
binwalk

👉 幾乎所有 Forensics 題都適用

🚀 七、延伸學習

你可以進一步學：

steganography（隱寫術）

XOR 編碼

壓縮檔藏檔（zip inside file）

進階 metadata 分析