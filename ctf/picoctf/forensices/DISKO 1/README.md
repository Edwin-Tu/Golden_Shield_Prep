# CTF Writeup - Disk Image String Search

## 題目類型
- Digital Forensics
- Disk Image Analysis
- Strings Search

---

## 題目描述
題目提供了一個壓縮檔：

```bash
disko-1.dd.gz
```

解壓後可得到：

```bash
disko-1.dd
```

這是一個 `.dd` 磁碟映像檔，需要從中找出隱藏的 flag。

---

## 解題目標
從磁碟映像檔 `disko-1.dd` 中找出題目的 flag。

---

## 先備知識

### 1. 什麼是 `.dd` 檔？
`.dd` 通常是**磁碟映像檔（disk image）**，也就是把整個磁碟、隨身碟、記憶卡或分割區的內容完整複製成一個檔案。

它和一般壓縮檔不同，因為它可能包含：

- 檔案系統
- 真正的檔案內容
- 已刪除但尚未覆寫的資料
- 開機區資訊
- 其他隱藏在磁碟中的內容

在數位鑑識與 CTF 中，`.dd` 檔很常拿來當作分析目標。

---

### 2. 什麼是 `.gz`？
`.gz` 是 `gzip` 壓縮格式。

所以：

```bash
disko-1.dd.gz
```

代表的是：

- `disko-1.dd`：原始磁碟映像檔
- `.gz`：外層用 gzip 壓縮過

因此解題第一步通常是先解壓縮。

---

## 解題流程

### Step 1. 解壓縮檔案

如果想保留原始壓縮檔，可以使用：

```bash
gzip -dk disko-1.dd.gz
```

參數說明：

- `-d`：解壓縮
- `-k`：保留原本的 `.gz` 檔案

解壓後會得到：

```bash
disko-1.dd
```

---

### Step 2. 確認檔案類型

使用 `file` 指令查看檔案型態：

```bash
file disko-1.dd
```

可能看到類似結果：

```bash
disko-1.dd: DOS/MBR boot sector, code offset 0x58+2, OEM-ID "mkfs.fat", ...
```

這表示它不是普通文字檔，而是一個帶有檔案系統資訊的磁碟映像。

---

### Step 3. 使用 `strings` 搜尋可讀字串

在 CTF 中，如果懷疑 flag 是以明文存在檔案裡，可以先嘗試 `strings`：

```bash
strings -a disko-1.dd | less
```

#### 指令說明
- `strings`：從二進位檔案中抽出可讀文字
- `-a`：掃描整個檔案，而不是只掃描預設資料區段
- `| less`：讓輸出可以一頁一頁查看

由於磁碟映像通常很大，直接看全部內容會很亂，所以可以搭配 `grep` 篩選。

---

### Step 4. 直接搜尋 flag 格式

這題是 picoCTF，因此很適合直接搜尋：

```bash
strings -a disko-1.dd | grep 'picoCTF{'
```

輸出結果：

```bash
picoCTF{1t5_ju5t_4_5tr1n9_c63b02ef}
```

---

## Flag

```text
picoCTF{1t5_ju5t_4_5tr1n9_c63b02ef}
```

---

## 解題思路說明

這題的關鍵在於：  
**flag 並沒有經過複雜加密，而是直接以字串形式存在磁碟映像中。**

因此最有效率的方式不是一開始就掛載映像、分析檔案系統，  
而是先用最基本也最常見的鑑識手法：

1. `file` 確認檔案型態
2. `strings` 擷取可讀字串
3. `grep` 搜尋 flag 格式

這是一種很典型的 CTF 起手式，尤其對於：

- binary 題
- memory dump 題
- disk image 題
- 壓縮檔藏 flag 題

都很常有效。

---

## 補充知識

### 1. 為什麼 `strings` 對這題有用？
因為雖然 `.dd` 是二進位檔案，但其中常常仍然包含可讀文字，例如：

- 檔名
- 路徑
- 指令字串
- 程式訊息
- 明文 flag
- 已刪除檔案殘留內容

如果 flag 是以純文字方式存在，`strings` 很容易直接抓出來。

---

### 2. 如果 `grep 'picoCTF{'` 找不到怎麼辦？
可以改試以下方式：

#### 搜尋 `flag`
```bash
strings -a disko-1.dd | grep -i flag
```

#### 搜尋常見大括號格式
```bash
strings -a disko-1.dd | grep '{'
```

#### 直接對原始二進位檔搜尋
```bash
grep -a 'picoCTF{' disko-1.dd
```

#### 顯示位移位置
```bash
grep -aob 'picoCTF{' disko-1.dd
```

參數說明：
- `-a`：把二進位當文字
- `-o`：只顯示符合的字串
- `-b`：顯示位元組 offset

這對後續用 `xxd` 或 `dd` 定位資料很有幫助。

---

### 3. 如果題目更難，下一步可以做什麼？
如果 `strings` 沒找到，可以繼續做：

#### 看十六進位內容
```bash
xxd disko-1.dd | less
```

#### 掃描隱藏結構
```bash
binwalk disko-1.dd
```

#### 列出分割區資訊
```bash
fdisk -l disko-1.dd
```

#### 掛載映像檔進行檔案系統分析
若是有分割區或完整檔案系統，可以再進一步掛載。

---

## 本題使用到的指令整理

### 解壓縮
```bash
gzip -dk disko-1.dd.gz
```

### 確認檔案型態
```bash
file disko-1.dd
```

### 抽取可讀字串
```bash
strings -a disko-1.dd | less
```

### 搜尋 flag
```bash
strings -a disko-1.dd | grep 'picoCTF{'
```

---

## 本題重點整理

- `.dd` 是磁碟映像檔
- `.gz` 只是外層壓縮格式
- 拿到磁碟映像後，不一定要先掛載
- 如果題目設計較簡單，可以先用 `strings` 快速搜尋明文內容
- 搭配 `grep` 搜特定 flag 格式，常能快速破題

---

## 學習心得
這題讓我理解到，遇到磁碟映像檔時不需要一開始就把流程想得太複雜。  
先從最基本的檔案辨識與字串搜尋開始，往往就能快速發現關鍵資訊。

也因此建立了一個很重要的觀念：

> 在 CTF 中，先使用成本最低、速度最快的檢查方式，再逐步深入分析。

像是 `file`、`strings`、`grep`，就是非常值得優先嘗試的工具。

---

## 可延伸練習
之後遇到類似題目時，可以依照這樣的順序操作：

```bash
file 檔名
strings -a 檔名 | less
strings -a 檔名 | grep 'flag'
strings -a 檔名 | grep 'picoCTF{'
xxd 檔名 | less
binwalk 檔名
fdisk -l 檔名
```

這會是非常實用的數位鑑識 CTF 基本流程。

---
