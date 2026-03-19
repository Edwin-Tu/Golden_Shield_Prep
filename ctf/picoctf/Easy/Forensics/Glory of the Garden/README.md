# CTF 教學文件：使用 `strings` 從圖片檔找出隱藏資訊

## 題目類型

本題屬於非常基礎的 **Forensics / Steganography（鑑識 / 隱寫）** 類型。

題目提供一張圖片檔 `garden.jpg`，而這題的解法非常直接：

> 使用 `strings` 從圖片檔中擷取可讀字串，從中找出隱藏的資訊或 flag。

這類題目常見的出題方式包括：

- 把 flag 直接藏在檔案裡的可讀文字區段
- 將提示字串附加在圖片尾端
- 把關鍵資訊放在 metadata、comment 或其他可被 `strings` 掃到的地方

---

## 觀念說明：什麼是 `strings`？

`strings` 是 Linux / WSL 中很常用的工具，功能是：

> 從二進位檔案中擷取出「人類可讀的文字字串」。

因為圖片、PDF、執行檔、封包檔等本質上都是二進位資料，
如果題目作者把提示、flag、帳密、路徑或註解直接藏在檔案中，
就有機會被 `strings` 直接掃出來。

---

## 適用情境

`strings` 特別適合先拿來檢查以下檔案：

- `.jpg` / `.png`
- `.pdf`
- `.zip`
- `.pcap`
- 可執行檔
- 任何可疑的二進位檔

在 CTF 裡，這通常是最先該嘗試的快速偵查手法之一。

---

## 使用環境

以下示範以 **WSL / Linux 終端機** 為主。

假設題目檔名為：

```bash
 garden.jpg
```

---

## 第一步：確認檔案型態

先用 `file` 看看這個檔案是不是正常圖片：

```bash
file garden.jpg
```

可能看到類似輸出：

```bash
garden.jpg: JPEG image data, JFIF standard 1.01, baseline, precision 8, ...
```

### 這一步的目的

- 確認副檔名不是假的
- 確認它真的是 JPEG 檔
- 建立基本判斷：這是一張圖片，但不代表裡面沒有藏東西

---

## 第二步：直接用 `strings` 掃描

最基本的指令：

```bash
strings garden.jpg
```

如果題目真的很簡單，flag 或提示通常就會直接出現在輸出中。

你也可以搭配分頁查看：

```bash
strings garden.jpg | less
```

---

## 第三步：用 `grep` 篩選關鍵字

因為 `strings` 的輸出可能很多，建議搭配 `grep` 找常見 CTF 關鍵字：

```bash
strings garden.jpg | grep -Ei "flag|ctf|pico|hidden|secret"
```

如果知道比賽 flag 格式，也可以直接搜：

### 例如 picoCTF

```bash
strings garden.jpg | grep "picoCTF"
```

### 例如 flag{...}

```bash
strings garden.jpg | grep -E "flag\{|FLAG\{|ctf\{"
```

---

## 第四步：必要時調整 `strings` 的條件

有些時候可讀字串太短，預設不一定能完整顯示，
這時可以手動降低字串最短長度：

```bash
strings -n 3 garden.jpg
```

或：

```bash
strings -n 4 garden.jpg
```

### 說明

- `-n 3`：顯示長度至少 3 個字元的字串
- `-n 4`：顯示長度至少 4 個字元的字串

這在某些 flag 被切得很短、或提示字樣很短時很有幫助。

---

## 建議實戰流程

這題如果要快速解，建議照這個順序：

```bash
file garden.jpg
strings garden.jpg
strings garden.jpg | grep -Ei "flag|ctf|pico|hidden|secret"
strings -n 3 garden.jpg | less
```

如果題目真的屬於入門題，通常到這裡就能找到答案。

---

## 解題思路整理

這題的核心概念不是圖片內容本身，而是：

> 題目把資訊直接藏在檔案的可讀字串裡。

所以真正的解題關鍵是：

1. **不要只看圖片畫面**  
   圖片看起來再正常，也不代表檔案內部沒有藏資料。

2. **先做低成本偵查**  
   `strings` 是成本很低、速度很快的第一步。

3. **看到圖片檔也不要先急著用複雜工具**  
   很多新手會一開始就去想隱寫、LSB、Steghide、ExifTool，
   但其實不少題目只是把答案直接藏成可讀字串。

---

## 為什麼這題適合新手？

因為它能幫你建立一個很重要的習慣：

> 拿到題目檔案後，先做最基本的檢查，而不是一開始就走複雜路線。

在鑑識題中，常見的基本檢查有：

- `file`
- `strings`
- `xxd` / `hexdump`
- `exiftool`
- `binwalk`

其中 `strings` 幾乎是最簡單、最快、最值得先試的工具之一。

---

## 延伸補充

如果 `strings` 沒有直接找到答案，可以繼續往下檢查：

### 1. 檢查 metadata

```bash
exiftool garden.jpg
```

### 2. 看檔案尾端是否有附加資料

```bash
tail -c 200 garden.jpg
```

### 3. 看十六進位內容

```bash
xxd garden.jpg | less
```

### 4. 掃描是否藏有其他檔案

```bash
binwalk garden.jpg
```

但這一題的重點就是：

> **先用 `strings` 就夠了。**

---

## 範例 writeup 模板

你可以在提交作業或 GitHub 練習紀錄時，寫成這樣：

```markdown
### 解題流程
1. 使用 `file garden.jpg` 確認檔案型態為 JPEG。
2. 使用 `strings garden.jpg` 擷取圖片中的可讀字串。
3. 從輸出結果中搜尋可疑資訊。
4. 成功找到題目隱藏的提示 / flag。

### 使用指令
```bash
file garden.jpg
strings garden.jpg
strings garden.jpg | grep -Ei "flag|ctf|pico|hidden|secret"
```
```

---

## 本題重點總結

- 題目檔案是圖片，但解法不是看圖，而是看檔案內容
- `strings` 能快速擷取二進位檔中的可讀字串
- 遇到圖片、PDF、執行檔、封包檔時，`strings` 都值得先試
- 很多簡單 CTF 題的 flag 其實就直接藏在可讀字串中

---

## 結論

這題是很典型的新手入門題，重點不在複雜技巧，而在於：

> **建立正確的鑑識習慣：先做基本檢查。**

只要想到先用 `strings` 掃一下，這類題目通常都能很快突破。

