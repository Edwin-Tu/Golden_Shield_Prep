## Challenge Metadata

- **Platform:** picoCTF
- **Category:** Forensics
- **Difficulty:** Easy
- **Author:** Bryan
- **Date:** 2025-12-07

### 一、基本資訊

**題目名稱：** DISKO 1  
**題目類型：** Forensics  
**平台：** picoCTF 2025  
**目標：** 從題目提供的磁碟映像檔中找出隱藏的 flag

### 二、題目概述

本題提供一個壓縮過的磁碟映像檔 disko-1.dd.gz，題目要求分析該 disk image，找出其中隱藏的 flag。

初步觀察可知，這不是一般直接提供文字、執行檔或網頁服務的題目，而是典型的數位鑑識（Digital Forensics）題型。攻擊者或分析者必須先解壓縮映像檔，再確認其檔案系統結構，進一步掛載磁碟內容並搜尋可能的 flag。

實際分析後可發現，本題的 flag 並不是以一般檔案形式明顯存在於掛載後的目錄中，而是直接藏在磁碟映像的原始位元組資料（raw disk data）裡。因此，若只依賴掛載後瀏覽檔案內容，將無法找到 flag；必須進一步對整個 .dd 映像進行字串掃描，才能成功取得答案。

### 三、分析目標

本次分析的主要目標如下：

- 確認題目提供的檔案類型與磁碟映像格式
- 分析磁碟映像的分割區與檔案系統
- 掛載磁碟映像並檢查可見檔案內容
- 找出 flag 的實際儲存位置
- 說明本題的解題思路與鑑識重點

### 四、初步分析

首先確認題目檔案內容，可看到提供的是：

```
disko-1.dd.gz
```

因此第一步需先解壓縮：

```sh
gunzip disko-1.dd.gz
```

解壓後得到：

```
disko-1.dd
```

接著使用 parted 檢查磁碟映像資訊：

```sh
sudo parted disko-1.dd print
```

分析結果如下：

- 磁碟大小：約 52.4MB
- Sector size：512B
- Partition Table：loop
- File system：fat32
- 分割區範圍：0.00B ~ 52.4MB

由上述資訊可得知：

- 本題提供的是一個可直接掛載的 FAT32 磁碟映像
- 分割區從 offset 0 開始，因此不需要額外計算偏移量
- 可以直接以 loop 方式掛載整個映像檔

### 五、掛載與檔案檢查

由於該磁碟映像為 FAT32，且分割區起始位置為 0，因此可直接掛載：

```sh
sudo mkdir -p /mnt/disko
sudo mount -o loop disko-1.dd /mnt/disko
```

掛載完成後，檢查根目錄內容：

```sh
ls -lha /mnt/disko
```

結果顯示根目錄中主要只有一個 bin 目錄：

```
/mnt/disko
└── bin
```

進一步列出 /bin 內容：

```sh
ls -lha /mnt/disko/bin
```

可以看到大量可執行檔與系統工具，例如：

- bash
- apt
- grep
- curl
- find
- tmux
- python3.12
- containerd

然而在可見檔案中，並未發現任何明顯與 flag 相關的檔案，例如：

- flag.txt
- secret.txt
- 含有 picoCTF 字樣的檔名

此外，直接對掛載後目錄進行搜尋也沒有結果：

```sh
grep -Ri "picoctf" /mnt/disko
find /mnt/disko -type f -name "*flag*"
```

這代表 flag 並不在一般可見檔案中。

### 六、關鍵判定

#### 6.1 判定依據

當掛載後只能看到正常的 FAT32 檔案系統內容，卻找不到任何 flag 檔案時，需考慮以下可能性：

- flag 藏在隱藏檔案中
- flag 藏在已刪除資料或 slack space 中
- flag 並非作為檔案存在，而是直接寫入磁碟映像的原始位元組資料中

本題中，經過可見檔案搜尋後沒有結果，因此最合理的下一步是直接掃描整個 .dd 檔案的可讀字串。

#### 6.2 分析方法

使用 strings 對磁碟映像進行字串提取，再以 grep 篩選與 picoCTF 相關的內容：

```sh
strings disko-1.dd | grep -i pico
```

此方法可從二進位資料中撈出 ASCII 可讀字串，適合找出：

- 明文藏在 raw data 中的 flag
- 未被掛載系統顯示的殘留文字
- 不是以一般檔案形式存在的敏感資訊

### 七、利用過程

1. 解壓縮磁碟映像

   ```sh
   gunzip disko-1.dd.gz
   ```

2. 檢查分割區與檔案系統

   ```sh
   sudo parted disko-1.dd print
   ```

3. 掛載磁碟映像

   ```sh
   sudo mkdir -p /mnt/disko
   sudo mount -o loop disko-1.dd /mnt/disko
   ```

4. 檢查掛載內容

   ```sh
   ls -lha /mnt/disko
   ls -lha /mnt/disko/bin
   ```

5. 搜尋一般檔案中的 flag

   ```sh
   grep -Ri "picoctf" /mnt/disko
   find /mnt/disko -type f -name "*flag*"
   ```

6. 直接掃描磁碟映像原始字串

   ```sh
   strings disko-1.dd | grep -i pico
   ```

### 八、利用結果

執行字串掃描後，成功找到 flag：

由此可確認：

- flag 並不在掛載後可見的普通檔案中
- flag 是以明文字串藏在 .dd 映像的 raw bytes 裡

本題重點在於 disk image 的基礎鑑識與字串掃描，而非檔案還原或複雜的分割區分析。

### 九、完整解題指令

```sh
gunzip disko-1.dd.gz

sudo parted disko-1.dd print

sudo mkdir -p /mnt/disko
sudo mount -o loop disko-1.dd /mnt/disko

ls -lha /mnt/disko
ls -lha /mnt/disko/bin

grep -Ri "picoctf" /mnt/disko
find /mnt/disko -type f -name "*flag*"

strings disko-1.dd | grep -i pico
```

### 十、成因分析

本題並非典型漏洞利用題，而是數位鑑識題，因此「成因分析」應理解為 flag 為何能夠不透過一般檔案系統被發現。

關鍵原因如下：

- disk image 會完整保留磁碟的原始位元組資料
- .dd 並不是單純的檔案集合，而是整個磁碟內容的逐位元組複製。
- 掛載檔案系統只能看到「檔案系統願意顯示」的內容
- 若資料未被正式記錄成檔案，或已脫離正常目錄結構，掛載後通常看不到。
- strings 能直接從二進位資料中提取可讀文字
- 若 flag 以 ASCII 明文形式存在於磁碟映像任意位置，即使不屬於某個檔案，仍可能被掃描出來。

換言之，本題的核心並不是還原被刪除檔案，也不是破解某種加密，而是理解：

- 磁碟映像 ≠ 單純的可見檔案集合，而是完整的底層資料副本。

### 十一、鑑識重點與建議

#### 11.1 先確認磁碟結構

在分析 .dd 類型題目時，應先使用：

- file
- fdisk
- parted

了解分割區、檔案系統與 offset。

#### 11.2 掛載後找不到，不代表資料不存在

很多初學者在掛載後沒看到 flag.txt 就會誤以為方向錯了，但在 forensics 題型中，資料常藏在：

- raw bytes
- deleted space
- slack space
- metadata
- hidden structures

#### 11.3 善用 strings 作為第一輪掃描工具

對於中小型 disk image，strings 幾乎是最有效率的初步檢查方式之一：

```
strings image.dd | grep -i pico
```

這通常能快速判斷 flag 是否以明文存在。

#### 11.4 養成由淺入深的分析順序

建議順序如下：

1. 解壓縮
2. 看檔案型態
3. 看 partition
4. 掛載
5. 搜尋檔案
6. 掃 raw strings
7. 必要時再進一步做 forensic carving

### 十二、結論

本題 DISKO 1 的核心考點在於基本磁碟鑑識觀念。分析過程顯示，題目提供的是一個 FAT32 格式的磁碟映像，雖然可成功掛載並看到部分目錄內容，但 flag 並不存在於一般可見檔案中。

透過進一步對整個 .dd 映像做字串掃描，最終成功找到flag

本題說明了一個重要觀念：
在 disk forensics 題型中，掛載後看到的檔案只是一部分資訊；真正的敏感資料可能直接存在於磁碟映像的原始位元組中。

因此，遇到 disk image 題目時，不應只停留在目錄瀏覽，而應同時具備從 raw data 角度進行分析的意識。

### 十三、附錄：關鍵觀念整理

項目 | 說明
---|---
題目類型 | Disk Forensics
檔案格式 | .dd.gz 解壓後為 .dd raw disk image
檔案系統 | FAT32
掛載方式 | mount -o loop
一般搜尋結果 | 未在可見檔案中找到 flag
實際解法 | 對整個映像使用 strings 掃描
flag 儲存形式 | 明文字串藏於 raw disk data
本題成果 | 成功取得 flag

## 使用工具

- strings
- grep
- parted
- mount
- Linux command line