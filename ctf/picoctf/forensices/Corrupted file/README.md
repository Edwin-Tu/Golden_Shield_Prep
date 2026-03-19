# CTF Writeup：Restoring Bytes（README.md）

## 題目類型
- File Forensics
- Magic Bytes
- 檔案修復

---

## 題目概述

這題提供一個無法直接辨識格式的檔案，目標是透過觀察檔案內容、判斷檔案真實類型、修復異常的檔頭，最後成功取得隱藏在檔案中的 flag。

這題的重點不在複雜工具，而在於：

- 先觀察檔案狀態
- 根據線索推測真實格式
- 檢查 magic bytes（檔頭）
- 修復檔案後重新驗證

---

## 解題目標

找出檔案真正的格式，修復後開啟內容，取得 flag。

---

## 使用環境
- Windows + WSL
- Ubuntu / Debian 類 Linux 環境
- Python 3

---

## 可能會用到的工具

### `file`
用來判斷檔案類型。

### `strings`
用來擷取檔案中的可讀字串，快速發現格式線索。

### `xxd` 或 `hexdump`
用來檢視檔案前幾個位元組（magic bytes）。

### `python3`
用來修復檔案內容。

---

# 解題流程

## 1. 先觀察檔案

假設題目檔案名稱為：

```bash
file
```

先查看目前目錄與檔案資訊：

```bash
ls -l
file file
```

### 預期觀察

你可能會發現：

```bash
file: data
```

### 判斷

這代表：

- `file` 工具無法正常辨識格式
- 檔案可能被破壞、偽裝、加密或混淆
- 下一步應該檢查可讀字串與檔頭

---

## 2. 用 `strings` 尋找線索

```bash
strings file | head -20
```

或：

```bash
strings file | less
```

### 可能看到的關鍵字

```text
JFIF
```

### 判斷

`JFIF` 是 JPEG 圖片常見的標記之一。  
即使 `file` 無法辨識，只要看到 `JFIF`，就可以合理懷疑：

> 這個檔案本來應該是一張 JPEG 圖片，只是檔頭可能有問題。

---

## 3. 檢查檔案前幾個位元組

使用 `xxd`：

```bash
xxd -l 16 file
```

或使用 `hexdump`：

```bash
hexdump -C file | head
```

也可以用 Python 直接查看：

```bash
python3 - <<'PY'
with open("file", "rb") as f:
    print(f.read(16))
PY
```

### 觀察結果

你可能會看到類似：

```text
5c 78 ff e0 00 10 4a 46 49 46 ...
```

或：

```python
b'\\x\xff\xe0\x00\x10JFIF...'
```

---

## 4. 比對正常 JPEG 檔頭

正常 JPEG 檔案開頭通常是：

```text
ff d8 ff e0
```

但這題開頭卻變成：

```text
5c 78 ff e0
```

### 進一步分析

- `5c` = `\`
- `78` = `x`

也就是：

```text
\x
```

這表示原本應該存在的 JPEG 檔頭被破壞了，前兩個位元組被替換成了字元 `\x`。

---

## 5. 做出推論

目前已知：

1. `strings` 看到了 `JFIF`
2. 正常 JPEG 應該以 `ff d8 ff e0` 開頭
3. 題目實際開頭是 `5c 78 ff e0`

因此可以合理推測：

> 這份檔案本來是 JPEG，但前兩個位元組被故意改掉了。  
> 只要把前兩個位元組修正回 `ff d8`，檔案就有機會恢復正常。

---

## 6. 修復檔案

使用 Python 建立修復後的圖片：

```bash
python3 - <<'PY'
from pathlib import Path

data = Path("file").read_bytes()

# 將前兩個錯誤位元組改回 JPEG 正常開頭 ff d8
fixed = b'\xff\xd8' + data[2:]

Path("fixed.jpg").write_bytes(fixed)
print("fixed.jpg created")
PY
```

---

## 7. 驗證修復結果

```bash
file fixed.jpg
```

### 若修復成功，會看到類似：

```bash
fixed.jpg: JPEG image data, JFIF standard ...
```

這表示你的推理正確，檔案已恢復成正常 JPEG。

---

## 8. 開啟圖片取得 flag

在 WSL 中可使用：

```bash
xdg-open fixed.jpg
```

如果無法直接開啟，也可以將 `fixed.jpg` 複製回 Windows 後再開啟。

打開圖片後，即可看到 flag。

---

# Flag

```text
picoCTF[r3st0r1ng_th3_byt3s_2326ca93]
```

---

# 完整操作指令整理

```bash
# 1. 檢查檔案類型
file file

# 2. 查看可讀字串
strings file | head -20

# 3. 檢查前 16 bytes
xxd -l 16 file

# 4. 修復檔頭
python3 - <<'PY'
from pathlib import Path
data = Path("file").read_bytes()
fixed = b'\xff\xd8' + data[2:]
Path("fixed.jpg").write_bytes(fixed)
print("fixed.jpg created")
PY

# 5. 驗證修復後的類型
file fixed.jpg

# 6. 開啟圖片
xdg-open fixed.jpg
```

---

# 解題思路整理

這題的正確思考流程如下：

## 第一步：先看檔案能不能被辨識
使用 `file` 發現它只是 `data`，表示格式異常。

## 第二步：找內容中的格式線索
使用 `strings` 後發現 `JFIF`，推測它原本是 JPEG。

## 第三步：確認檔頭是否被破壞
使用 `xxd` 或 `hexdump` 發現開頭不是正常 JPEG 的 `ff d8 ff e0`，而是 `5c 78 ff e0`。

## 第四步：理解異常位元組
`5c 78` 是 ASCII 的 `\x`，代表檔頭被惡意替換。

## 第五步：修復檔案
將前兩個位元組改回 `ff d8`。

## 第六步：重新驗證並開啟
修復後成功辨識為 JPEG，開圖得到 flag。

---

# 知識補充

## 什麼是 Magic Bytes？

Magic Bytes 是檔案開頭的一組固定標記，用來表示檔案格式。

例如：

### JPEG
```text
ff d8 ff
```

### PNG
```text
89 50 4e 47 0d 0a 1a 0a
```

### PDF
```text
25 50 44 46
```

### ZIP
```text
50 4b 03 04
```

當副檔名不可信或檔案被破壞時，Magic Bytes 是判斷真實格式的重要依據。

---

## 為什麼 `strings` 很重要？

因為有些檔案即使外部格式壞掉，內部仍可能保留關鍵字，例如：

- `JFIF`
- `PNG`
- `PDF`
- `flag`
- `PK`

這些都能幫助你快速判斷下一步該往哪個方向分析。

---

## 為什麼這題不需要先用 `binwalk`？

因為這題的主要問題出在檔頭，而不是內嵌壓縮包或多層封裝。  
只要先掌握 `file`、`strings`、`xxd` 的基本分析流程，就足以解出這題。

---

# 心得

這題雖然不難，但非常適合練習最基本也最重要的檔案分析能力。  
在 CTF 中，很多題目並不需要一開始就使用很複雜的工具，而是先從：

- 檔案類型
- 可讀字串
- 檔頭內容

這三件事開始觀察。  
只要能正確推理，就能快速發現問題並修復檔案。

---

# 參考技能整理

本題可學到的技能：

- 使用 `file` 判斷檔案類型
- 使用 `strings` 擷取可讀字串
- 使用 `xxd` / `hexdump` 檢查 magic bytes
- 理解 JPEG 檔頭格式
- 使用 Python 修復二進位檔案
- 建立「觀察 → 判斷 → 驗證 → 修復」的解題流程

---

# 建議延伸練習

完成這題後，你可以進一步練習：

1. 嘗試修改 PNG 或 PDF 的檔頭，再自行修復
2. 練習辨識常見檔案格式的 magic bytes
3. 接觸簡單的 steganography 題型
4. 練習用 `binwalk`、`exiftool`、`zsteg` 等工具分析檔案

---
