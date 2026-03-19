# CTF Writeup：unknown.zip / ukn_reality.jpg Metadata 解題教學

## 題目類型
- Forensics
- ZIP 解壓縮
- JPEG Metadata Analysis
- Base64 Decoding

---

## 題目概述

這題先提供一個壓縮檔 `unknown.zip`。  
解壓後可以得到一張圖片 `ukn_reality.jpg`。  
圖片本身看起來是一般 JPEG，但透過檢查 metadata，可以在 `Attribution URL` 欄位中找到一段 **Base64** 字串。  
將它解碼後，即可取得 flag。

這題的核心流程是：

1. 解壓 ZIP 檔
2. 確認解壓出的檔案類型
3. 用 `exiftool` 檢查圖片 metadata
4. 找出可疑的 Base64 字串
5. 解碼取得 flag

---

## 使用環境
- Windows + WSL
- Ubuntu / Debian 類 Linux
- `unzip`
- `file`
- `exiftool`
- `base64`

---

## 題目檔案
- `unknown.zip`

---

# 解題流程

## 1. 先解壓縮 ZIP 檔

你執行了：

```bash
unzip unknown.zip
```

實際輸出：

```bash
Archive:  unknown.zip
  inflating: ukn_reality.jpg
```

### 判斷
代表壓縮檔內含一個檔案：

- `ukn_reality.jpg`

這表示下一步應該針對這張圖片做分析。

---

## 2. 查看目前目錄中的檔案

你執行了：

```bash
ls
```

實際輸出：

```bash
desktop.ini  ukn_reality.jpg  unknown.zip
```

### 判斷
可以確認：

- 原始壓縮檔 `unknown.zip`
- 解壓後得到的圖片 `ukn_reality.jpg`

---

## 3. 使用 `file` 確認圖片格式

你執行了：

```bash
file ukn_reality.jpg
```

實際輸出：

```bash
ukn_reality.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, baseline, precision 8, 4308x2875, components 3
```

### 判斷
這表示：

- 檔案是一張正常的 JPEG 圖片
- 沒有副檔名偽裝問題
- 接下來可以從圖片的 metadata 著手分析

---

## 4. 檢查 metadata

你先打錯指令：

```bash
exitool ukn_reality.jpg
```

系統提示：

```bash
Command 'exitool' not found, did you mean:
  command 'exiftool' from deb libimage-exiftool-perl (12.67+dfsg-1)
```

### 判斷
這裡只是單純拼字錯誤，正確工具名稱是：

```bash
exiftool
```

---

## 5. 使用 `exiftool` 查看圖片 metadata

你執行了：

```bash
exiftool ukn_reality.jpg
```

實際輸出重點如下：

```text
XMP Toolkit                     : Image::ExifTool 11.88
Attribution URL                 : cGljb0NURntNRTc0RDQ3QV9ISUREM05fZGVjYTA2ZmJ9Cg==
```

### 關鍵觀察

最可疑的是這一行：

```text
Attribution URL : cGljb0NURntNRTc0RDQ3QV9ISUREM05fZGVjYTA2ZmJ9Cg==
```

### 為什麼可疑？

因為這串文字：

- 看起來不像正常網址
- 只包含英文字母、數字與 `=`
- 很像 **Base64** 編碼後的內容

因此可以合理推測：

> flag 被藏在圖片 metadata 的 `Attribution URL` 欄位中，而且經過 Base64 編碼。

---

## 6. 第一次嘗試解碼失敗

你先解了一段**不完整**的字串：

```bash
echo 'cGljb0NURntNRTc0RDQ3QV9ISUREM05fZGVjYTA2Zm' | base64 -d
```

實際輸出：

```text
picoCTF{ME74D47A_HIDD3N_deca06fbase64: invalid input
```

### 判斷
這表示：

- 你選的方向是正確的
- 因為已經成功解出前半段 `picoCTF{...}`
- 但因為 Base64 字串不完整，所以結尾出現：

```text
base64: invalid input
```

這是非常典型的「少複製了幾個字元」情況。

---

## 7. 重新複製完整字串後解碼

完整字串應為：

```text
cGljb0NURntNRTc0RDQ3QV9ISUREM05fZGVjYTA2ZmJ9Cg==
```

你執行了：

```bash
echo 'cGljb0NURntNRTc0RDQ3QV9ISUREM05fZGVjYTA2ZmJ9Cg==' | base64 -d
```

實際輸出：

```text
picoCTF{ME74D47A_HIDD3N_deca06fb}
```

---

# Flag

```text
picoCTF{ME74D47A_HIDD3N_deca06fb}
```

---

# 完整操作記錄整理

```bash
unzip unknown.zip

ls

file ukn_reality.jpg

exiftool ukn_reality.jpg

echo 'cGljb0NURntNRTc0RDQ3QV9ISUREM05fZGVjYTA2ZmJ9Cg==' | base64 -d
```

---

# 解題思路整理

## 第一步：先解壓縮
題目先給的是 `unknown.zip`，所以要先用 `unzip` 把內容取出。

## 第二步：確認解壓出的檔案
使用 `ls` 與 `file` 確認得到的是 JPEG 圖片 `ukn_reality.jpg`。

## 第三步：檢查圖片 metadata
使用 `exiftool` 檢查圖片資訊，尋找可疑欄位。

## 第四步：找出可疑字串
在 `Attribution URL` 欄位看到一段不像正常網址、卻很像 Base64 的文字。

## 第五步：進行 Base64 解碼
用 `base64 -d` 解碼後，成功得到 flag。

---

# 知識補充

## 為什麼解壓後要先用 `file`？

因為有些 CTF 題目會故意：

- 偽裝副檔名
- 把檔案藏在壓縮包裡
- 解壓後再做第二層誤導

所以先用 `file` 看真實格式，是很重要的基本習慣。

---

## 為什麼 metadata 值得檢查？

圖片不只包含畫面本身，還常帶有額外資料，例如：

- EXIF
- IPTC
- XMP
- 作者資訊
- 版權資訊
- 描述欄位
- 自訂欄位

這些地方很常被拿來藏：

- flag
- 提示
- 編碼過的字串

這題就是藏在：

```text
Attribution URL
```

---

## 怎麼判斷一串文字可能是 Base64？

你可以觀察幾個特徵：

- 由英文字母、數字、`+`、`/`、`=` 組成
- 看起來像規則亂碼
- 長度通常是 4 的倍數
- 結尾可能有 `=` 或 `==`

這題的：

```text
cGljb0NURntNRTc0RDQ3QV9ISUREM05fZGVjYTA2ZmJ9Cg==
```

就很符合 Base64 特徵。

---

## 為什麼第一次解碼會出現 `invalid input`？

因為第一次你輸入的是不完整字串：

```text
cGljb0NURntNRTc0RDQ3QV9ISUREM05fZGVjYTA2Zm
```

雖然前面已經能解出部分內容，但後面不完整，因此 Base64 解碼器就會報錯。

這提醒我們：

> 做 Base64 解碼時，務必要確認整串字都有完整複製下來。

---

# 心得

這題是很典型的圖片 metadata 題，而且多了一個 ZIP 解壓縮的前置步驟。  
整體難度不高，但很適合建立完整的鑑識題習慣：

1. 先解壓
2. 看檔案類型
3. 查 metadata
4. 找可疑字串
5. 試常見編碼解碼

真正重要的不是工具本身，而是解題流程的順序。

---

# 本題學到的技能

- 使用 `unzip` 解壓縮題目檔案
- 使用 `ls` 檢查解壓結果
- 使用 `file` 確認檔案真實格式
- 使用 `exiftool` 檢查圖片 metadata
- 辨識 Base64 字串
- 使用 `base64 -d` 解碼
- 建立「解壓 → 觀察 → metadata → 解碼」的思考流程

---

# 建議延伸練習

完成這題後，可以進一步練習：

1. 使用 `zip` / `unzip` 操作更多壓縮檔題型
2. 練習在 JPEG 的不同 metadata 欄位中藏字串
3. 比較 `strings` 與 `exiftool` 的用途差異
4. 練習辨識 Base64、Hex、ROT13、URL encode 等常見編碼
5. 嘗試自己做一張藏有 metadata flag 的圖片

---
