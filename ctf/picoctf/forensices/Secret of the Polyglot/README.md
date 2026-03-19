# PicoCTF Writeup - 變更副檔名取得兩段 Flag

## 題目重點

這一題的核心不是爆破或逆向，而是觀察檔案本身的結構。

同一個檔案其實同時藏了兩種格式的內容：

- 以 **PNG 圖片** 開啟時，可以看到前半段 flag
- 以 **PDF** 開啟時，可以看到後半段 flag

也就是說，這是一個典型的 **polyglot file（多重格式檔案）** 題型。

---

## 解題觀念

有些 CTF 題目會故意把：

- 一個檔案偽裝成另一種副檔名
- 或把兩種檔案格式塞進同一個檔案裡

這時候如果只用副檔名判斷，很容易被誤導。

真正要做的是：

1. 用工具判斷檔案的實際格式
2. 觀察檔案內是否還藏有其他格式特徵
3. 嘗試用不同副檔名或不同程式開啟

---

## 題目檔案分析

假設題目檔名是：

```bash
flag2of2-final.pdf
```

先用 `file` 檢查：

```bash
file flag2of2-final.pdf
```

可能會看到類似結果：

```bash
flag2of2-final.pdf: PNG image data, 50 x 50, 8-bit/color RGBA, non-interlaced
```

這裡非常關鍵：

- 檔名是 `.pdf`
- 但 `file` 卻判斷它其實是 **PNG image data**

代表這個檔案並不單純。

---

## 解法一：改成 PNG 觀看前半段 Flag

先把檔案複製一份，再改成 `.png`：

```bash
cp flag2of2-final.pdf flag_part1.png
```

接著開啟圖片：

```bash
xdg-open flag_part1.png
```

或在 Windows / 檔案總管中直接用圖片檢視器開啟。

打開後可以看到前半段 flag：

```text
picoCTF{f1u3n7_
```

---

## 解法二：以 PDF 方式觀看後半段 Flag

同一個檔案再用 PDF 方式開啟，就可以看到另一段內容。

若系統可直接用 PDF 閱讀器開啟原檔，可直接嘗試：

```bash
xdg-open flag2of2-final.pdf
```

若無法正常顯示，也可以進一步檢查檔案內是否藏有 PDF 標頭：

```bash
strings flag2of2-final.pdf | less
```

你會看到檔案中出現：

```text
%PDF-1.4
```

這代表檔案裡面真的還藏著 PDF 結構。

以 PDF 方式顯示後，可以取得後半段 flag：

```text
1n_pn9_&_pdf_1f991f77}
```

---

## 組合完整 Flag

把前後兩段組合起來：

```text
picoCTF{f1u3n7_1n_pn9_&_pdf_1f991f77}
```

---

## 完整解題流程示範

### 1. 先確認真實檔案類型

```bash
file flag2of2-final.pdf
```

### 2. 複製一份並改成 PNG

```bash
cp flag2of2-final.pdf flag_part1.png
```

### 3. 開啟圖片取得前半段

```bash
xdg-open flag_part1.png
```

看到：

```text
picoCTF{f1u3n7_
```

### 4. 再用 PDF 方式觀察另一半

```bash
xdg-open flag2of2-final.pdf
```

看到：

```text
1n_pn9_&_pdf_1f991f77}
```

### 5. 組合得到完整 flag

```text
picoCTF{f1u3n7_1n_pn9_&_pdf_1f991f77}
```

---

## 為什麼會這樣？

這是因為題目檔案被做成了 **PNG + PDF 的雙格式檔案**。

### PNG 的特性
PNG 檔案有明確的檔頭與區塊結構，只要圖片資料本身完整，某些額外資料附加在尾端時，圖片依然可能正常顯示。

### PDF 的特性
PDF 閱讀器會尋找 `%PDF-` 標頭來解析文件內容，所以即使前面混入了其他資料，只要 PDF 區段仍然完整，仍有機會被讀取。

因此同一份檔案就可能同時被：

- 當作圖片讀取
- 當作 PDF 讀取

這就是這題最有趣的地方。

---

## 進階驗證方法

除了改副檔名，還可以再做幾個驗證。

### 1. 用 `strings` 找 PDF 標頭

```bash
strings flag2of2-final.pdf | grep PDF
```

### 2. 用 `grep -abo` 找 `%PDF` 出現的位置

```bash
grep -abo "%PDF" flag2of2-final.pdf
```

這可以知道 PDF 內容是從哪個位元組開始。

### 3. 用 `dd` 把內嵌 PDF 切出來

假設 `%PDF` 出現在偏移 `914`：

```bash
dd if=flag2of2-final.pdf of=extracted.pdf bs=1 skip=914
```

接著再開：

```bash
xdg-open extracted.pdf
```

這樣就能把內嵌的 PDF 單獨抽出來。

---

## 這題學到什麼

這題可以學到幾個很重要的 CTF 觀念：

1. **不要只相信副檔名**
2. **要用 `file` 檢查真實格式**
3. **看到異常格式時，要懷疑是不是 polyglot file**
4. **同一個檔案可能藏有多種資料格式**
5. **改副檔名只是表面操作，真正關鍵是理解檔案結構**

---

## 常用指令整理

### 判斷檔案格式

```bash
file 檔名
```

### 顯示可讀字串

```bash
strings 檔名
```

### 搜尋特定字串出現位置

```bash
grep -abo "關鍵字" 檔名
```

### 從特定位元組開始切出檔案

```bash
dd if=原始檔 of=輸出檔 bs=1 skip=位移值
```

---

## 最終答案

```text
picoCTF{f1u3n7_1n_pn9_&_pdf_1f991f77}
```

---

## 總結

這題屬於很經典的檔案格式觀察題。

表面上它是一個 `.pdf`，但實際上也能當成 `.png` 使用；兩種開啟方式分別顯示出不同的 flag 片段。只要仔細觀察檔案型態，再把兩段內容組合，就能順利拿到完整 flag。

如果之後遇到：

- 副檔名和真實格式不一致
- 圖片裡藏壓縮檔
- PDF 裡藏圖片
- 一個檔案能被兩種程式打開

都可以往 **polyglot / embedded file** 的方向思考。
