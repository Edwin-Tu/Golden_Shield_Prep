# CTF 檔案題前置教學指引

> 這份文件不是單一題的 writeup，而是整合多種練習題後形成的「前置判讀手冊」。
> 目標是幫助你在拿到題目檔案後，能先判斷題型、辨識常見編碼、選對工具，再進入正式解題。

---

## 一、這份指引適合什麼時候用？

當你在 CTF 中拿到以下檔案時，都很適合先打開這份 README：

- 圖片：`.png`、`.jpg`、`.jpeg`
- 文件：`.pdf`
- 封包：`.pcap`
- 壓縮檔：`.zip`、`.gz`
- 磁碟映像：`.dd`、`.img`
- 沒有副檔名的可疑檔案：`file`、`data`、`unknown`
- 一堆檔案 + checksum / script 題
- 遠端登入後看到奇怪輸出（例如 QR Code）

這類題目通常不考複雜 exploitation，而是考你是否有正確的**鑑識習慣**。

---

## 二、核心觀念：先觀察，再判斷，再驗證

新手最常犯的錯，是一拿到檔案就直接亂猜工具。

比較好的順序是：

1. **先確認檔案真實類型**
2. **先做低成本偵查**
3. **找可疑字串 / metadata / 檔頭 / 結構**
4. **判斷題型**
5. **再用對應工具深入**
6. **最後驗證結果**

你可以把它記成：

> **觀察 → 判斷 → 驗證 → 解碼 / 提取 / 修復**

---

## 三、CTF 檔案題黃金流程

拿到任何檔案時，先做這一組：

```bash
ls -lah
file 檔名
strings 檔名 | less
exiftool 檔名
binwalk 檔名
```

如果題目是可疑檔案或 `file` 顯示不出格式，再補：

```bash
xxd -l 32 檔名
hexdump -C 檔名 | head
```

如果是壓縮檔或磁碟映像，再補：

```bash
unzip 檔名.zip
gunzip 檔名.gz
parted 檔名.dd print
```

如果是封包檔，再補：

```bash
file traffic.pcap
strings traffic.pcap | less
```

如果是很多檔案配合 checksum，再補：

```bash
sha256sum files/*
```

---

## 四、常用工具總整理

## 1. `file`：先確認真實格式

### 作用
用來判斷檔案真正的類型，而不是只相信副檔名。

### 為什麼重要？
因為在 CTF 中很常出現：

- `.pdf` 其實是 PNG
- `.jpg` 其實藏 ZIP
- 沒副檔名但其實是 JPEG / ELF / PDF
- 檔頭被破壞導致只顯示 `data`

### 常用指令
```bash
file 檔名
```

### 你要注意什麼？
- 副檔名和真實格式是否一致
- 是否只顯示 `data`
- 是否是 `ASCII text`、`PDF document`、`JPEG image data`、`PNG image data`
- 是否顯示 filesystem、disk image、pcap

---

## 2. `strings`：最低成本偵查工具

### 作用
從二進位檔案中擷取可讀字串。

### 適合用在哪裡？
- 圖片
- PDF
- 執行檔
- 壓縮檔
- 磁碟映像
- 封包檔

### 常用指令
```bash
strings 檔名
strings 檔名 | less
strings 檔名 | grep -Ei "flag|ctf|pico|secret|hidden"
strings -n 3 檔名
```

### 什麼時候特別有用？
- flag 直接藏在可讀字串中
- metadata 其實能被 strings 掃到
- 題目把提示字串放在檔尾
- 檔案格式壞掉，但內部關鍵字還在
- disk image 的 raw bytes 內有明文 flag

### 常見關鍵字
- `JFIF`
- `PNG`
- `%PDF`
- `PK`
- `flag`
- `picoCTF`
- `Comment`
- `Author`
- `License`

---

## 3. `exiftool`：metadata 檢查神器

### 作用
查看檔案中繼資料（metadata）。

### 特別適合
- JPG / JPEG
- PNG
- PDF
- 有 XMP / EXIF / Comment / Author / License 的檔案

### 常用指令
```bash
exiftool 檔名
exiftool -a -u -g1 檔名
```

### 常見藏法
- `Comment`
- `Author`
- `License`
- `Copyright`
- `Attribution URL`
- XMP 欄位
- PDF Author / Title

### 看到什麼要警覺？
- 很長、看似亂碼的字串
- 只包含英數與 `=` 的內容
- 看起來不像正常網址或作者名稱的欄位值

---

## 4. `binwalk`：掃描是否藏有其他檔案

### 作用
分析檔案內是否嵌入其他格式或壓縮資料。

### 常用指令
```bash
binwalk 檔名
binwalk -e 檔名
```

### 適合題型
- 圖片裡藏 zip
- PDF 裡藏 png / zip
- 一個檔案裡塞另一個檔案
- 多層封裝題

### 你會看到的線索
- `Zip archive data`
- `PNG image data`
- `PDF document`
- `gzip compressed data`

### 什麼時候不用先用它？
如果題目明顯是檔頭破壞（magic bytes 異常），先看 `file`、`strings`、`xxd` 通常更快。

---

## 5. `xxd` / `hexdump`：檢查 magic bytes

### 作用
直接看檔案前幾個位元組，判斷真實格式、檔頭是否被破壞。

### 常用指令
```bash
xxd -l 32 檔名
hexdump -C 檔名 | head
```

### 常見 magic bytes
```text
JPEG : ff d8 ff
PNG  : 89 50 4e 47 0d 0a 1a 0a
PDF  : 25 50 44 46
ZIP  : 50 4b 03 04
GZIP : 1f 8b
```

### 典型用途
- 無副檔名檔案判型
- 修復被破壞的 JPG / PNG / PDF 檔頭
- 比對副檔名與真實格式
- 找出內嵌格式的起始位置

---

## 6. `base64`：最常見的編碼解碼工具

### 作用
解碼或編碼 Base64 內容。

### 常用指令
```bash
echo '字串' | base64 -d
base64 -d input.txt > output.bin
```

### 典型用途
- metadata 中藏 Base64
- 封包 payload 是 Base64
- LSB 抽出來後得到 Base64
- 假 log 其實是一大段 Base64 圖片

---

## 7. `sha256sum`：檔案比對與驗證

### 作用
計算 SHA-256 雜湊值，比對哪個檔案才是正確目標。

### 常用指令
```bash
sha256sum 檔名
sha256sum files/*
sha256sum files/* | grep "$(cat checksum.txt)"
```

### 適合題型
- 題目給 checksum
- 一個資料夾有很多檔案
- 需要找出哪一個才是真的
- 找到正確檔案後再交給 script 處理

---

## 8. `steghide`：JPG / 音訊隱寫提取

### 作用
從圖片或音訊中提取隱藏資料。

### 常用指令
```bash
steghide extract -sf img.jpg
steghide extract -sf img.jpg -p 密碼
```

### 適合題型
- metadata 裡先給你提示或密碼
- JPG 表面正常，但實際藏有附件
- 題目明示或暗示使用 steghide

---

## 9. `Wireshark`：PCAP 圖形化分析

### 作用
用圖形介面檢查封包內容、協定、payload 與時間順序。

### 你要觀察什麼？
- 封包數量是否少
- 協定是否明顯集中
- payload 中是否有可疑字串
- 是否能從時間順序拼接資料
- 是否有檔案傳輸、登入資訊或明文線索

---

## 10. Disk Forensics 工具組

### 常見工具
```bash
parted 檔名.dd print
fdisk -l 檔名.dd
losetup -Pf 檔名.dd
mount -o loop 檔名.dd /mnt/disko
fls -r 檔名.dd
icat 檔名.dd inode
```

### 用途
- 看 partition
- 判斷 filesystem
- 掛載映像
- 列出檔案
- 從 inode 抽出隱藏檔案

### 核心觀念
Disk image 題不要只看掛載結果，還要考慮：
- raw bytes
- deleted space
- slack space
- hidden file
- 多個 partition 混淆

---

## 11. 其他實用工具

### `grep`
快速從大量輸出中過濾關鍵字。
```bash
grep -Ei "flag|ctf|pico|secret"
```

### `dd`
從特定位元組切出內嵌檔案。
```bash
dd if=原始檔 of=輸出檔 bs=1 skip=位移
```

### `zbarimg`
掃描 QR Code。
```bash
zbarimg screenshot.png
```

### `python3`
當需要：
- 修復檔頭
- 分析像素
- 解碼資料
- 批次處理輸出
時非常好用。

---

## 五、常見編碼 / 特徵判讀

## 1. Base64

### 常見特徵
- 只包含 `A-Z`、`a-z`、`0-9`、`+`、`/`
- 常以 `=` 或 `==` 結尾
- 長度通常是 4 的倍數
- 看起來像亂碼，但很整齊

### 例子
```text
cGljb0NURg==
```

### 第一反應
```bash
echo '字串' | base64 -d
```

---

## 2. Hex

### 常見特徵
- 只由 `0-9` 和 `a-f` / `A-F` 組成
- 偶數長度很常見
- 看起來像：
```text
666c6167
```

### 第一反應
```bash
echo '666c6167' | xxd -r -p
```

---

## 3. ASCII / 純文字線索

### 常見特徵
- 可直接用 `strings` 掃出
- 可能是旗標、路徑、帳密、提示
- 常出現在檔尾、註解、metadata 或 raw bytes

### 第一反應
- 直接閱讀
- 配合 `grep`
- 注意是否只是提示，不一定是最終 flag

---

## 4. URL Encode

### 常見特徵
- 含 `%20`、`%3D`、`%2F` 等
- 常出現在 web / pcap 題

### 第一反應
- 用線上解碼器、Python，或先判斷是否需 URL decode

---

## 5. Gzip / Zip / 壓縮資料

### 常見特徵
- `file` 顯示 `gzip compressed data`、`Zip archive data`
- `binwalk` 掃到壓縮資料
- 檔名可能是 `.gz`、`.zip`
- 也可能藏在 disk image 或 polyglot file 裡

### 第一反應
```bash
gunzip 檔名.gz
unzip 檔名.zip
```

---

## 6. QR Code

### 常見特徵
- 遠端登入後畫面出現方塊圖形
- 截圖後可被 QR 掃描器辨識
- 有時題目不是給圖片，而是終端機文字拼出的 QR

### 第一反應
- 截圖
- 用手機掃
- 或：
```bash
zbarimg screenshot.png
```

---

## 六、依題目類型判斷方向

## 1. PNG 題

### 常見方向
- LSB 隱寫
- 顏色通道藏資料
- metadata / 文字提示
- 附加資料
- Base64 藏在像素裡

### 先做什麼？
```bash
file red.png
strings red.png | less
exiftool red.png
binwalk red.png
```

### 如果畫面很單純怎麼想？
- 純色圖很可能是 LSB
- strings 出現詩、提示文、藏頭句，要小心
- 可用 Python / Pillow 分析像素值是否只差 1

---

## 2. JPG / JPEG 題

### 常見方向
- metadata 藏 Base64
- steghide 隱寫
- strings 直接有 flag
- 檔頭破壞
- 圖片內嵌檔案

### 先做什麼？
```bash
file cat.jpg
strings cat.jpg | less
exiftool cat.jpg
binwalk cat.jpg
```

### 額外注意
- 若 `file` 顯示不正常，但 strings 看見 `JFIF`，可能是 JPEG 檔頭壞了
- 若 metadata 先給你一個密碼線索，下一步很可能是 `steghide`

---

## 3. PDF 題

### 常見方向
- metadata 藏 Base64
- strings 能掃出 `/Author`、`/Title`、`/Producer`
- PDF 內嵌 zip / image
- polyglot file
- 看起來亂碼，但其實 flag 不在頁面內容而在 metadata

### 先做什麼？
```bash
file confidential.pdf
strings confidential.pdf | less
exiftool confidential.pdf
binwalk confidential.pdf
```

### 額外注意
- PDF 本身很常只是載體
- 不要只打開 PDF 用肉眼看頁面

---

## 4. `file` / `data` / 無副檔名 題

### 常見方向
- 副檔名被拿掉
- 檔頭損毀
- 真實格式被偽裝
- 需要修復 magic bytes

### 先做什麼？
```bash
file file
strings file | less
xxd -l 32 file
```

### 重點觀念
- `file` 顯示 `data` 不代表沒資訊
- 先看 strings 與 magic bytes
- 如果出現 `JFIF`、`PNG`、`%PDF`、`PK`，就能反推原始格式

---

## 5. PCAP 題

### 常見方向
- payload 藏 Base64
- 封包片段需要依時間順序拼接
- 找 HTTP / DNS / TCP 明文資料
- 抓帳密、檔案、cookie、flag

### 先做什麼？
```bash
file myNetworkTraffic.pcap
strings myNetworkTraffic.pcap | less
```

### 再做什麼？
- 用 Wireshark 打開
- 看協定
- 看 Data / payload
- 看是否有重複模式
- 看封包順序能不能拼資料

---

## 6. ZIP / GZ 題

### 常見方向
- 壓縮後才是正題
- 解壓後得到圖片 / PDF / disk image
- 題目真正線索在下一層

### 先做什麼？
```bash
unzip unknown.zip
gunzip disko-1.dd.gz
file 解壓後檔案
```

### 重點
拿到壓縮檔不要急著在壓縮層浪費時間，先解開再判型。

---

## 7. Disk Image 題（`.dd` / `.img`）

### 常見方向
- raw bytes 藏明文 flag
- 多個 partition 混淆
- 正確 filesystem 才有答案
- flag 壓縮後藏在某個 inode / log 檔裡

### 基本流程
```bash
gunzip disko-1.dd.gz
file disko-1.dd
parted disko-1.dd print
```

之後依題目狀況分兩條線：

### 路線 A：先掛載看檔案
```bash
sudo mkdir -p /mnt/disko
sudo mount -o loop disko-1.dd /mnt/disko
ls -lah /mnt/disko
find /mnt/disko -type f | less
```

### 路線 B：直接掃 raw data
```bash
strings disko-1.dd | grep -Ei "flag|ctf|pico"
```

### 額外提醒
如果題目有多個 partition，不要直接掃整個 disk 就相信第一個結果；要先判斷哪個 filesystem 才是正確目標。

---

## 8. Checksum / Script 題

### 常見方向
- 題目給 checksum
- 有一堆檔案
- 有解密腳本
- 核心是先找到正確檔案再處理

### 基本流程
```bash
ls
cat checksum.txt
cat decrypt.sh
sha256sum files/* | grep "$(cat checksum.txt)"
./decrypt.sh 正確檔案
```

### 重點
- `file decrypt.sh` 只能告訴你它是文字檔
- 真正要理解腳本內容，應該用 `cat` 或 `less`

---

## 9. Polyglot / 多重格式檔案

### 常見方向
- 一個檔案能被兩種程式打開
- 改副檔名就能看到不同內容
- `file` 和副檔名不一致
- `strings` 可同時看到 `%PDF`、`PNG`、`JFIF` 等特徵

### 基本流程
```bash
file 檔名
strings 檔名 | less
grep -abo "%PDF" 檔名
```

### 可以怎麼做？
- 改副檔名測試
- 用 `dd` 從內嵌格式起始位移切出檔案
- 分別用圖片檢視器 / PDF 閱讀器開啟

---

## 七、基本解題步驟模板

下面是一個最通用的模板。

## Step 1：先看檔案和目錄
```bash
ls -lah
file 檔名
```

### 你要回答的問題
- 這真的是它表面上的格式嗎？
- 是單一檔案，還是壓縮包 / disk image / pcap？

---

## Step 2：先做低成本掃描
```bash
strings 檔名 | less
exiftool 檔名
binwalk 檔名
```

### 你要找什麼？
- 可疑字串
- metadata
- 內嵌檔案
- 題目提示
- 可疑編碼

---

## Step 3：辨識特徵
### 問自己：
- 這像 Base64 嗎？
- 這像 Hex 嗎？
- 這是 metadata 線索嗎？
- 這是 magic bytes 問題嗎？
- 這像 LSB / steghide / disk raw data 題嗎？
- 這是不是需要換副檔名或切出內嵌檔？

---

## Step 4：依題型深入
### 例如：
- metadata 題 → `exiftool`
- 檔頭修復題 → `xxd`
- LSB 題 → Python / Pillow
- pcap 題 → Wireshark
- disk 題 → `parted` / `mount` / `fls` / `icat`
- checksum 題 → `sha256sum`
- QR 題 → 截圖 + `zbarimg`

---

## Step 5：取得結果後驗證
### 例如：
- 解出來是否像 flag 格式？
- 圖片是否能正常打開？
- hash 是否吻合？
- 抽出的檔案是否真的能被 `file` 正確辨識？
- 封包片段是否要重新排序？

---

## 八、實戰判斷速查表

| 看到的現象 | 優先懷疑方向 | 優先工具 |
| --- | --- | --- |
| 副檔名和內容怪怪的 | 偽裝格式 / polyglot | `file`, `strings`, `xxd` |
| `file` 只顯示 `data` | magic bytes 損壞 / 偽裝 | `strings`, `xxd`, `hexdump` |
| metadata 欄位有亂碼 | Base64 / Hex / 提示 | `exiftool`, `base64` |
| 純色 PNG、肉眼看不到東西 | LSB / 通道隱寫 | `strings`, Python, Pillow |
| JPG 很正常但沒線索 | metadata / steghide / binwalk | `exiftool`, `steghide`, `binwalk` |
| PDF 頁面沒東西 | metadata / embedded file | `strings`, `exiftool`, `binwalk` |
| pcap 裡有整齊亂碼片段 | Base64 payload / 分段拼接 | Wireshark, `base64` |
| `.dd` 掛載後看不到 flag | raw bytes / deleted space | `strings`, `fls`, `icat` |
| 一堆檔案 + checksum | hash 驗證 | `sha256sum`, `grep` |
| SSH 登入後是方塊圖 | QR Code | 截圖, `zbarimg` |

---

## 九、常見錯誤觀念

## 1. 只看副檔名
錯。CTF 很常故意騙你。

## 2. 只用肉眼打開圖片 / PDF 看內容
錯。很多 flag 根本不在畫面上，而在 metadata、字串、結構或像素裡。

## 3. 看到亂碼就直接放棄
錯。先想是不是：
- Base64
- Hex
- URL encode
- 壓縮
- XOR 後資料
- QR / 圖片資料

## 4. disk image 只會掛載、不會掃 raw
錯。掛載看不到，不代表資料不存在。

## 5. 直接對整個 disk 掃描就相信第一個 flag
錯。多 partition 題常有 fake flag。

## 6. 用 `echo 檔名`
錯。那只是輸出字串，不是讀檔。  
要讀檔請用：
```bash
cat 檔名
less 檔名
```

---

## 十、推薦安裝工具（WSL / Ubuntu）

```bash
sudo apt update
sudo apt install file binutils exiftool binwalk steghide zbar-tools python3 python3-pip
sudo apt install wireshark
```

如果要做圖片分析：
```bash
pip install pillow numpy
```

如果要做 disk forensics：
```bash
sudo apt install sleuthkit
```

---

## 十一、最小可行解題清單

如果你只想背最精簡版本，先記這組：

```bash
file 檔名
strings 檔名 | less
exiftool 檔名
binwalk 檔名
xxd -l 32 檔名
```

再根據結果分流：

- 像 Base64 → `base64 -d`
- 像 metadata 題 → `exiftool`
- 像壓縮 → 解壓
- 像 LSB → Python 分析像素
- 像 pcap → Wireshark
- 像 disk → `parted` / `mount` / `fls` / `icat`
- 像 checksum 題 → `sha256sum`

---

## 十二、總結

CTF 檔案題真正重要的，不是背很多零散工具，而是建立正確順序：

1. **先判型**
2. **先低成本偵查**
3. **辨識編碼與結構特徵**
4. **選對工具**
5. **逐步驗證**

只要你養成這套流程，之後遇到：

- PNG / JPG / PDF
- `file` / `data`
- PCAP
- ZIP / GZ
- Disk image
- Polyglot file
- Checksum / Script 題

都會更快找到切入點。

---

## 十三、建議你之後的實戰習慣

每次做題都先記錄這四件事：

1. 題目檔名與 `file` 結果
2. `strings` 有沒有關鍵字
3. `exiftool` 有沒有可疑 metadata
4. 下一步為什麼選某個工具

這樣你的 writeup 會更清楚，解題速度也會穩定提升。
