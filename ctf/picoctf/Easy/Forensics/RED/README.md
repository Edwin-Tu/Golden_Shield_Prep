# CTF Writeup：Red PNG LSB 隱寫分析（README.md）

## 題目敘述
- RED, RED, RED, RED

## 題目類型
- Steganography
- PNG 隱寫
- LSB（Least Significant Bit）
- Base64 解碼

---

## 題目概述

這題提供一張看起來幾乎只是**純紅色**的 PNG 圖片。  
表面上圖片沒有任何明顯內容，但透過觀察檔案中的文字線索，可以判斷題目提示我們去檢查圖片的 **LSB（最低有效位）**。  
進一步抽出像素資料後，可以取得一段 Base64 字串，解碼後得到 flag。

---

## 解題目標

1. 從圖片的 metadata / 字串資訊找到提示
2. 判斷題目是在考 LSB 隱寫
3. 取出圖片像素中的最低有效位資料
4. 還原隱藏字串
5. 進行 Base64 解碼取得 flag

---

## 使用環境
- Windows + WSL
- Ubuntu / Debian 類 Linux
- Python 3
- Pillow
- NumPy

---

## 可能會用到的工具

### `file`
用來確認圖片格式。

### `strings`
用來檢查圖片中是否有可讀字串、提示訊息或 metadata。

### `python3`
用來分析圖片像素、抽取 LSB 與解碼資料。

### `base64`
用來解碼萃取出的 Base64 字串。

---

# 解題流程

## 1. 先觀察圖片

題目給的是一張看起來只有紅色的圖片：

- 畫面沒有直接顯示 flag
- 顏色非常單一
- 這類題目常見方向有：
  - metadata 提示
  - 顏色通道隱藏資訊
  - LSB 隱寫
  - 圖片附加資料

因此第一步不是直接猜答案，而是先檢查檔案本身。

---

## 2. 先確認檔案資訊

```bash
file red.png
strings red.png | less
```

### 你要觀察什麼？

- `file red.png`：確認它是 PNG 圖片
- `strings red.png | less`：檢查是否存在文字提示

---

## 3. 從 `strings` 找到關鍵線索

在 `strings` 的輸出中，可以看到一段像詩一樣的文字：

```text
Crimson heart, vibrant and bold,
Hearts flutter at your sight.
Evenings glow softly red,
Cherries burst with sweet life.
Kisses linger with your warmth.
Love deep as merlot.
Scarlet leaves falling softly,
Bold in every stroke.
```

---

## 4. 判斷這段詩的真正用途

這段詩表面上像描述紅色，但如果看每一行的第一個字母：

- **C**rimson
- **H**earts
- **E**venings
- **C**herries
- **K**isses
- **L**ove
- **S**carlet
- **B**old

組起來就是：

```text
CHECKLSB
```

### 判斷

這代表題目在明確提示你：

> 去檢查圖片的 LSB（Least Significant Bit）

所以這題不是單純看畫面，而是要分析像素資料。

---

## 5. 什麼是 LSB？

LSB 是 **Least Significant Bit（最低有效位）**。

以一個顏色值來說：

- `254` 的二進位最低位是 `0`
- `255` 的二進位最低位是 `1`

這兩個值肉眼看起來差異極小，但電腦可以利用最低位偷偷藏資訊。  
這就是圖片隱寫中很常見的技巧。

---

## 6. 檢查圖片像素是否有 LSB 特徵

用 Python 檢查圖片模式、尺寸與各色彩通道的值：

```bash
python3 - <<'PY'
from PIL import Image
import numpy as np

img = Image.open("red.png")
arr = np.array(img)

print("mode:", img.mode)
print("size:", img.size)

for i, name in enumerate("RGBA"):
    vals = np.unique(arr[:, :, i])
    print(name, vals)
PY
```

### 預期觀察

你會發現這張圖的像素值很可疑，通常會接近這樣：

- `R` 只有 `254`、`255`
- `G` 只有 `0`、`1`
- `B` 只有 `0`、`1`
- `A` 只有 `254`、`255`

### 判斷

這種「只差 1」的數值分布非常符合 LSB 隱寫的特徵。

---

## 7. 萃取圖片中的 LSB 資料

既然題目提示 `CHECKLSB`，接下來就把每個像素通道的最低位取出。

這題實際上可以用 RGBA 每個通道的最低位來組字串。

```bash
python3 - <<'PY'
from PIL import Image
import numpy as np

img = Image.open("red.png")
arr = np.array(img)

bits = ""
for x in range(arr.shape[1]):
    for c in range(4):   # RGBA
        bits += str(arr[0, x, c] & 1)

out = ""
for i in range(0, len(bits), 8):
    byte = bits[i:i+8]
    out += chr(int(byte, 2))

print(out)
PY
```

---

## 8. 觀察萃取結果

你會得到一段像這樣的字串：

```text
cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==
```

### 判斷

這看起來非常像 Base64，原因包括：

- 只包含英數字與 `=`
- 結尾有 `==`
- 長度和格式很典型

所以下一步應該做 Base64 解碼。

---

## 9. Base64 解碼

### 用指令解碼

```bash
echo 'cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==' | base64 -d
```

### 或用 Python 解碼

```bash
python3 - <<'PY'
import base64

s = "cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ=="
print(base64.b64decode(s).decode())
PY
```

---

## 10. 得到 flag

解碼後可得：

```text
picoCTF{r3d_1s_th3_ult1m4t3_cur3_f0r_54dn355_}
```

---

# Flag

```text
picoCTF{r3d_1s_th3_ult1m4t3_cur3_f0r_54dn355_}
```

---

# 完整操作指令整理

## 先看檔案與字串提示

```bash
file red.png
strings red.png | less
```

## 檢查像素值分布

```bash
python3 - <<'PY'
from PIL import Image
import numpy as np

img = Image.open("red.png")
arr = np.array(img)

print("mode:", img.mode)
print("size:", img.size)

for i, name in enumerate("RGBA"):
    vals = np.unique(arr[:, :, i])
    print(name, vals)
PY
```

## 抽出 LSB 並顯示原始隱藏內容

```bash
python3 - <<'PY'
from PIL import Image
import numpy as np

img = Image.open("red.png")
arr = np.array(img)

bits = ""
for x in range(arr.shape[1]):
    for c in range(4):
        bits += str(arr[0, x, c] & 1)

data = ""
for i in range(0, len(bits), 8):
    data += chr(int(bits[i:i+8], 2))

print(data)
PY
```

## Base64 解碼

```bash
echo 'cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==' | base64 -d
```

---

# 一次完成的 Python 版本

```bash
python3 - <<'PY'
from PIL import Image
import numpy as np
import base64

img = Image.open("red.png")
arr = np.array(img)

bits = ""
for x in range(arr.shape[1]):
    for c in range(4):  # RGBA
        bits += str(arr[0, x, c] & 1)

data = ""
for i in range(0, len(bits), 8):
    data += chr(int(bits[i:i+8], 2))

print("Extracted:", data)
print("Decoded:", base64.b64decode(data).decode())
PY
```

---

# 解題思路整理

## 第一步：先觀察畫面
看到的是一張純紅圖片，肉眼沒有資訊，因此懷疑是隱寫題。

## 第二步：查 `strings`
發現一首詩，表示圖片中有文字提示。

## 第三步：看藏頭
每行首字母拼出 `CHECKLSB`，明確指向 LSB。

## 第四步：檢查像素值
發現各通道只在差 1 的值之間變動，符合 LSB 隱寫特徵。

## 第五步：萃取 LSB
把 RGBA 通道的最低位依序取出並組成位元流。

## 第六步：還原資料
將位元流每 8 bit 轉為字元，得到一段 Base64。

## 第七步：Base64 解碼
成功取得 flag。

---

# 知識補充

## 為什麼要先用 `strings`？

因為有些圖片題不會直接把資料藏在像素裡，而是先用 metadata、註解或提示文字告訴你解題方向。  
這題就是非常典型的例子：

- `strings` 先給你提示
- 提示再引導你去做 LSB 分析

---

## 為什麼 LSB 適合藏資料？

因為修改最低位通常不會讓圖片外觀發生明顯變化。  
例如：

- `255` 和 `254`
- `0` 和 `1`

對人眼來說幾乎看不出差別，但可以拿來表示 `1` 與 `0`。

---

## 為什麼結果還要再做 Base64？

因為很多 CTF 題目不會直接把 flag 原文放進 LSB，  
而是會先藏一段經過編碼的字串，例如：

- Base64
- Hex
- ASCII
- 壓縮資料

這題就是先藏 Base64，再由你自行解碼。

---

# 心得

這題是很典型的「兩階段提示式」隱寫題：

1. 先從字串或 metadata 找到提示
2. 再根據提示去分析圖片內容

如果一開始只盯著圖片看，會很難找到方向。  
但只要先做基本檢查：

- `file`
- `strings`
- 像素分析

就能很快掌握題目重點。

---

# 本題學到的技能

- 使用 `file` 檢查圖片格式
- 使用 `strings` 擷取隱藏提示
- 理解 LSB（最低有效位）概念
- 使用 Python 分析 PNG 像素
- 將位元流轉成字串
- 辨識並解碼 Base64
- 建立「觀察 → 判斷 → 驗證 → 解碼」的解題流程

---

# 建議延伸練習

完成這題後，可以繼續練習：

1. 嘗試只修改 R 通道或 Alpha 通道的 LSB
2. 練習把自己的文字藏進 PNG 中
3. 學習使用 `zsteg` 分析 PNG 隱寫
4. 練習辨識 Base64、Hex、ASCII 等常見編碼
5. 比較不同通道順序對結果的影響（如 RGBA、ARGB、RGB）

---
