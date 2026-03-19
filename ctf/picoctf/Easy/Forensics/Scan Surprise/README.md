# PicoCTF Writeup - QR Code Flag Challenge

## 題目概念

這一題 PicoCTF 需要先透過 `ssh` 連線到遠端主機，進入挑戰環境後，題目會以 **QR Code** 的形式顯示 flag。  
解題重點在於：

1. 成功使用 `ssh` 連線到題目主機
2. 進入遠端環境後觀察畫面輸出
3. 將畫面中的 QR Code 解碼，取得 flag

---

## 題目環境資訊

從題目提供的資訊與操作紀錄可知：

- 主機：`atlas.picoctf.net`
- Port：`54544`
- 使用者：`ctf-player`

連線指令如下：

```bash
ssh -p 54544 ctf-player@atlas.picoctf.net
```

---

## 操作過程

### 1. 使用 SSH 連線

在終端機中輸入：

```bash
ssh -p 54544 ctf-player@atlas.picoctf.net
```

畫面會顯示：

```text
The authenticity of host '[atlas.picoctf.net]:54544 ([18.217.83.136]:54544)' can't be established.
ED25519 key fingerprint is SHA256:hVmbk/OaNT4902bBr7h26wfhmBuJWi4tub8AJqoAJCM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

這是 SSH 第一次連線到該主機時的正常提示，意思是：

- 目前這台遠端主機尚未存在本機的 `known_hosts`
- SSH 正在詢問你是否信任這台主機

若確定是題目提供的官方主機，可以輸入：

```bash
yes
```

之後會看到：

```text
Warning: Permanently added '[atlas.picoctf.net]:54544' (ED25519) to the list of known hosts.
```

表示這台主機的金鑰已被記錄到本機，下次連線通常就不會再詢問。

---

### 2. 輸入密碼

接著系統會要求輸入密碼：

```text
ctf-player@atlas.picoctf.net's password:
```

此時輸入題目頁面提供的密碼即可。

> 注意：Linux/WSL 終端機輸入密碼時，畫面通常**不會顯示任何字元**，這是正常現象。  
> 即使你有輸入，畫面看起來也像是空白，直接輸入完按 Enter 即可。

---

### 3. 觀察遠端主機輸出內容

根據題意，這題的 flag 是一個 **QR Code**。  
所以登入成功後，通常會出現以下幾種情況之一：

- 終端機直接印出一個 QR Code
- 顯示由特殊字元組成的方塊圖樣
- 顯示某個檔案，裡面內容其實是 QR Code
- 顯示 ASCII/Unicode 方塊字元拼成的 QR Code

這時候不要只盯著文字本身，**要從「圖像資訊」的角度觀察畫面**。

---

## 解題思路

### 方法一：直接截圖後掃描 QR Code

如果登入後終端機畫面直接出現 QR Code，最直觀的方法就是：

1. 將終端機畫面完整截圖
2. 用手機相機或 QR Code 掃描工具掃描
3. 取得掃描結果中的 flag

#### 適合情境

- QR Code 已經能清楚顯示在終端機中
- 方塊對齊正常，沒有跑版
- 螢幕截圖後仍可辨識

---

### 方法二：調整終端機字型與視窗大小

如果 QR Code 看起來變形、擠壓、比例怪異，可以先調整：

- 終端機字型大小
- 視窗寬度
- 行距
- 縮放比例

因為 QR Code 很依賴方塊排列是否整齊，如果終端機顯示比例失真，掃描器可能無法辨識。

#### 建議

- 將終端機視窗放大
- 使用等寬字型
- 讓 QR Code 完整顯示在畫面中
- 再重新截圖掃描

---

### 方法三：若題目將 QR Code 存成檔案，先找檔案再下載/解碼

有些題目登入後不一定直接把 QR Code 顯示在畫面上，而是放在某個檔案裡。  
這時可以先檢查目前目錄內容：

```bash
ls
```

若想看詳細資訊：

```bash
ls -la
```

若發現像是圖片檔、文字檔、輸出檔，可再用以下方式觀察：

```bash
file 檔名
cat 檔名
strings 檔名
```

如果是圖片檔，例如：

- `qr.png`
- `flag.png`
- `code.png`

可以考慮將檔案下載到本機後再掃描。

---

## 可能會用到的指令

### 查看目前資料夾內容

```bash
ls
```

### 查看隱藏檔與詳細資訊

```bash
ls -la
```

### 判斷檔案類型

```bash
file 檔名
```

### 顯示純文字內容

```bash
cat 檔名
```

### 抽取可讀字串

```bash
strings 檔名
```

---

## 如果 QR Code 是由文字方塊組成

有時候題目不是給真正的 PNG 圖片，而是直接在終端機用字元印出 QR Code，例如：

- `█`
- `▀`
- `▄`
- 空白
- Unicode block characters

這種情況下，仍然可以：

1. 放大終端機
2. 確保畫面完整
3. 截圖
4. 用手機掃描

若手機掃不出來，可以把截圖存成圖片後，再使用本機工具辨識，例如：

```bash
zbarimg screenshot.png
```

> 前提是你的系統有安裝 `zbarimg`

安裝方式（Ubuntu / WSL）：

```bash
sudo apt update
sudo apt install zbar-tools
```

之後即可解碼：

```bash
zbarimg screenshot.png
```

---

## 這題的核心觀念

這題其實不難，重點不是爆破或逆向，而是：

- 知道如何用 `ssh` 進入題目環境
- 看到奇怪輸出時，能聯想到它可能是 **QR Code**
- 知道 QR Code 本質上也是一種資訊載體
- 會使用截圖、掃描工具或解碼工具取得結果

也就是說，這題主要在考：

- 基本遠端連線能力
- 對終端機輸出的觀察力
- 對 QR Code 形式的敏感度

---

## 操作紀錄整理

以下是本題的實際連線紀錄：

```bash
edwintu@EdwinTu:/mnt/c/Users/hc105/Downloads/home/ctf-player/drop-in$ ssh -p 54544 ctf-player@atlas.picoctf.net
The authenticity of host '[atlas.picoctf.net]:54544 ([18.217.83.136]:54544)' can't be established.
ED25519 key fingerprint is SHA256:hVmbk/OaNT4902bBr7h26wfhmBuJWi4tub8AJqoAJCM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[atlas.picoctf.net]:54544' (ED25519) to the list of known hosts.
ctf-player@atlas.picoctf.net's password:
```

由這段紀錄可以確認：

1. 已成功連線到 PicoCTF 遠端主機
2. 已接受主機金鑰
3. 下一步就是輸入密碼並進入挑戰環境
4. 進入後需特別注意終端機畫面是否出現 QR Code

---

## 解題總結

這題的標準流程可整理為：

1. 使用 `ssh` 連線到指定主機與 port
2. 第一次連線時接受主機金鑰
3. 輸入題目提供的密碼
4. 觀察登入後畫面
5. 發現 flag 是以 **QR Code** 形式呈現
6. 透過截圖、手機掃描或 `zbarimg` 等工具解碼
7. 取得 picoCTF flag

---

## 補充知識

### 什麼是 SSH？
SSH（Secure Shell）是一種安全的遠端登入協定，可以讓使用者透過加密連線操作遠端主機。

常見用途包括：

- 遠端登入 Linux 主機
- 傳輸檔案
- 執行遠端指令
- 參與 CTF 題目環境

---

### 什麼是 QR Code？
QR Code 是一種二維條碼，可以儲存：

- 網址
- 文字
- 帳號資訊
- 驗證碼
- CTF flag

在 CTF 中，QR Code 常被拿來當作：

- 視覺隱藏資訊
- 非純文字型 flag 載體
- 圖像類題目的線索

---

## 學到的重點

做完這題後，可以學到：

- 如何使用 `ssh -p` 指定 port 連線
- 第一次 SSH 連線時的主機驗證流程
- Linux 密碼輸入時不會顯示字元是正常現象
- CTF 題目不一定把 flag 明文顯示，可能會用 QR Code 包裝
- 當看到特殊字元方塊排列時，要想到它可能是一張可掃描的圖

---

## 建議延伸練習

若想更熟悉這類題型，可以再練習：

- 使用 `file` 判斷檔案類型
- 使用 `strings` 抽取可見字串
- 使用 `zbarimg` 解碼 QR Code
- 練習將終端機輸出轉換成可辨識圖片
- 熟悉更多常見的 CTF 載體形式（圖片、音訊、metadata、壓縮檔、編碼資料）

---

## 備註

本文件是依據提供的 SSH 操作紀錄整理而成。  
由於紀錄中尚未包含登入後實際看到的 QR Code 畫面，因此本文將後續步驟整理為通用且合理的解題流程，方便作為教學與 writeup 使用。
