# CTF Writeup：cat.jpg Metadata 解題教學

## 題目類型
- Forensics
- Metadata Analysis
- Base64 Decoding
- JPEG / EXIF / XMP

---

## 題目概述

這題提供一張名為 `cat.jpg` 的圖片，表面上看起來只是一般的貓咪照片。  
但透過檢查圖片的 metadata，可以發現其中某個欄位藏有一段 **Base64** 字串。  
將其解碼後，即可取得 flag。

這題的核心觀念是：

- 先確認檔案格式
- 檢查圖片 metadata
- 找出可疑欄位
- 判斷字串是否為 Base64
- 解碼取得 flag

---

## 使用環境
- Windows + WSL
- Ubuntu / Debian 類 Linux
- `file`
- `strings`
- `exiftool`
- `base64`

---

## 題目檔案
- `cat.jpg`

---

# 解題流程

## 1. 先確認目錄中的檔案

```bash
ls
```

你的實際輸出：

```bash
README.md  cat.jpg  desktop.ini  red.png
```

### 判斷
可以確認題目主檔案為 `cat.jpg`。

---

## 2. 用 `file` 確認檔案格式

```bash
file cat.jpg
```

你的實際輸出：

```bash
cat.jpg: JPEG image data, JFIF standard 1.02, aspect ratio, density 1x1, segment length 16, baseline, precision 8, 2560x1598, components 3
```

### 判斷
這表示：

- 檔案是一張正常的 JPEG 圖片
- 沒有明顯的副檔名偽裝問題
- 下一步可以檢查圖片內部資訊（metadata）

---

## 3. 使用 `exiftool` 查看 metadata

```bash
exiftool cat.jpg
```

你的實際輸出重點如下：

```text
Copyright Notice                : PicoCTF
License                         : cGljb0NURnt0aGVfbTN0YWRhdGFfMXNfbW9kaWZpZWR9
Rights                          : PicoCTF
```

### 關鍵觀察

其中最可疑的是這一行：

```text
License : cGljb0NURnt0aGVfbTN0YWRhdGFfMXNfbW9kaWZpZWR9
```

### 為什麼可疑？

因為這串文字：

- 只包含英文字母、數字
- 看起來像亂碼
- 很像常見的 **Base64** 格式

因此可以合理懷疑：

> flag 被藏在 metadata 的 `License` 欄位中，並且經過 Base64 編碼。

---

## 4. 用 `strings` 輔助確認

你也有執行：

```bash
strings cat.jpg | head
```

你的實際輸出：

```text
JFIF
0Photoshop 3.0
8BIM
PicoCTF
http://ns.adobe.com/xap/1.0/
<?xpacket begin='
' id='W5M0MpCehiHzreSzNTczkc9d'?>
<x:xmpmeta xmlns:x='adobe:ns:meta/' x:xmptk='Image::ExifTool 10.80'>
<rdf:RDF xmlns:rdf='http://www.w3.org/1999/02/22-rdf-syntax-ns#'>
 <rdf:Description rdf:about=''
```

### 判斷
`strings` 沒有直接給出 flag，但可以幫助我們確認：

- 這張圖裡確實含有 XMP / metadata 相關資訊
- `PicoCTF` 出現在字串中，表示方向正確
- 這題確實值得從 metadata 著手

---

## 5. 嘗試 Base64 解碼

你先嘗試解碼了一段**不完整**的字串：

```bash
echo 'cGljb0NURnt0aGVfbTN0YWRhdGFfMXNfbW9kaWZpZW' | base64 -d
```

輸出為：

```text
picoCTF{the_m3tadata_1s_modifiebase64: invalid input
```

### 判斷
這代表前面確實已經解出部分可讀內容，但因為字串不完整，所以 `base64` 提示：

```text
invalid input
```

也就是說：

- 方向是正確的
- 只是你複製到的 Base64 字串少了一部分

---

## 6. 重新複製完整 Base64 字串並解碼

完整正確的字串是：

```text
cGljb0NURnt0aGVfbTN0YWRhdGFfMXNfbW9kaWZpZWR9
```

執行：

```bash
echo 'cGljb0NURnt0aGVfbTN0YWRhdGFfMXNfbW9kaWZpZWR9' | base64 -d
```

你的實際輸出：

```text
picoCTF{the_m3tadata_1s_modified}
```

---

# Flag

```text
picoCTF{the_m3tadata_1s_modified}
```

---

# 完整操作記錄整理

```bash
ls

file cat.jpg

exiftool cat.jpg

strings cat.jpg | head

echo 'cGljb0NURnt0aGVfbTN0YWRhdGFfMXNfbW9kaWZpZWR9' | base64 -d
```

---

# 解題思路整理

## 第一步：確認檔案格式
用 `file` 檢查 `cat.jpg`，確認它是正常 JPEG。

## 第二步：檢查 metadata
用 `exiftool` 列出完整 metadata，尋找可疑欄位。

## 第三步：找出可疑字串
在 `License` 欄位中發現一串疑似 Base64 的內容。

## 第四步：進行解碼
使用 `base64 -d` 將字串解碼。

## 第五步：取得 flag
成功還原出：

```text
picoCTF{the_m3tadata_1s_modified}
```

---

# 知識補充

## 什麼是 metadata？

Metadata 是「描述檔案本身的資料」，例如：

- 作者
- 版權資訊
- 建立工具
- 相機資訊
- XMP / EXIF 欄位
- 自訂文字欄位

在 CTF 題中，常常會把 flag 或提示藏在這些欄位裡。

---

## 為什麼這題要先用 `exiftool`？

因為：

- `file` 只能看出檔案類型
- `strings` 只能看到部分可讀內容
- `exiftool` 能完整列出 JPEG 的 EXIF / IPTC / XMP 欄位

因此在圖片 metadata 題中，`exiftool` 通常是最直接有效的工具之一。

---

## 怎麼判斷一串字是不是 Base64？

常見特徵包括：

- 只包含 `A-Z`、`a-z`、`0-9`、`+`、`/`
- 有時會以 `=` 結尾
- 長度常看起來像規則亂碼
- 解碼後常出現 `picoCTF{...}`、`flag{...}` 之類格式

這題的 `License` 欄位就非常符合這種特徵。

---

## 為什麼第一次會出現 `invalid input`？

因為第一次貼進去解碼的字串是：

```text
cGljb0NURnt0aGVfbTN0YWRhdGFfMXNfbW9kaWZpZW
```

它少了最後一段，因此解碼只完成一部分就報錯。

這提醒我們：

> 在做 Base64 解碼時，要確認複製的是完整字串。

---

# 心得

這題是很典型的 metadata 題型，難度不高，但非常適合練習：

- 用 `file` 確認格式
- 用 `exiftool` 看 metadata
- 用 `strings` 做輔助觀察
- 用 `base64` 解碼可疑欄位

重點不是圖片本身，而是圖片攜帶的附加資訊。  
之後遇到類似題目時，可以優先想到：

1. `file`
2. `exiftool`
3. `strings`
4. 嘗試 Base64 / Hex / URL decode

---

# 本題學到的技能

- 使用 `file` 判斷圖片格式
- 使用 `exiftool` 分析圖片 metadata
- 使用 `strings` 輔助觀察字串內容
- 辨識 Base64 字串
- 使用 `base64 -d` 解碼
- 建立「觀察 → 判斷 → 驗證 → 解碼」的基本流程

---

# 建議延伸練習

完成這題後，可以進一步練習：

1. 使用 `exiftool` 修改圖片 metadata
2. 練習在 `Author`、`Comment`、`License` 等欄位中藏字串
3. 嘗試辨識 Hex、Base64、ROT13 等常見編碼
4. 練習結合 `strings`、`binwalk`、`zsteg` 做更深入的圖片分析

---
