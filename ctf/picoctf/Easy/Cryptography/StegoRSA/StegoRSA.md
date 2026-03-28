# StegoRSA

## Challenge Metadata
- **Challenge Name:** StegoRSA
- **Category:** Cryptography
- **Difficulty:** Easy
- **Event:** picoCTF 2026
- **Author:** YAHAYA MEDDY

## 一、基本資訊
本題提供兩個檔案：
- `flag.enc`
- `image.jpg`

題目描述指出訊息是以 RSA 加密，但 public key 不見了；同時暗示有人在處理 private key 時可能不夠謹慎。這表示關鍵材料極有可能被錯誤地藏在另一個附件中。

## 二、題目概述
本題結合了：
1. **Steganography / Hidden Data Discovery**
2. **RSA Private Key Recovery**
3. **Asymmetric Decryption**

核心思路為：從圖片中找出被隱藏的 RSA 私鑰，再使用該私鑰對 `flag.enc` 進行解密。

## 三、分析目標
1. 檢查 `image.jpg` 是否存在 metadata 或附加隱藏資料。
2. 從圖片中還原 RSA private key。
3. 使用 recovered private key 解密 `flag.enc`。
4. 取得最終 flag。

## 四、分析過程

### 4.1 檢查圖片 metadata
使用 `exiftool` 檢查圖片時，發現 JPEG 的 `Comment` 欄位包含一大串 hex 字串。其開頭片段如下：

```text
2d2d2d2d2d424547494e2050524956415445204b45592d2d2d2d2d
```

### 4.2 將 Comment 由 hex 還原成文字
上述 hex 轉換成 ASCII 後可得到：

```text
-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----
```

這表示 JPEG comment 中實際藏有一把完整的 RSA private key。

### 4.3 匯出並重建私鑰
操作流程如下：

```bash
exiftool -b -Comment image.jpg > comment_hex.txt
xxd -r -p comment_hex.txt > private.pem
head private.pem
```

驗證結果顯示 `private.pem` 為有效 PEM 格式私鑰。

### 4.4 解密密文
確認密文檔名為 `flag.enc` 後，使用 OpenSSL 進行 RSA 解密：

```bash
openssl pkeyutl -decrypt -inkey private.pem -in flag.enc
```

成功得到明文 flag。

## 五、解題結果
最終取得的 flag 為：

```text
picoCTF{rs4_k3y_1n_1mg_66388eb3}
```

## 六、關鍵觀念
- **Metadata Abuse：** 攻擊者或操作者可能把敏感資料寫入圖片 metadata。
- **Hex-Encoded Secret Storage：** 雖然表面看似一般 comment，實際內容經 hex 編碼後藏入。
- **RSA Decryption Flow：** 有了 private key 即可對 RSA ciphertext 進行解密。

## 七、成因分析
本題的安全失誤在於：
1. 將 RSA private key 以 comment metadata 形式藏入圖片。
2. 僅做 hex 編碼，未提供真正保護。
3. 一旦分析者檢查 metadata，即可恢復私鑰並解密密文。

這屬於典型的 **secret material exposure** 問題。

## 八、防禦建議
1. 私鑰不得儲存在圖片、文件 metadata 或其他非金鑰儲存系統中。
2. 發佈媒體檔前應清除 EXIF / Comment / XMP 等 metadata。
3. 對金鑰檔進行權限控管與安全保存，不可依賴「看不出來」作為防護。
4. 在安全稽核流程中加入 metadata 掃描，檢查是否有明文或可逆編碼的敏感資料。

## 九、結論
`StegoRSA` 是一道將資訊隱寫與非對稱解密結合的 easy 題。解題關鍵在於主動檢查影像 metadata，而非只專注於密文本身。只要成功從圖片 comment 恢復 private key，後續 RSA 解密流程即相當直接。

## 十、附錄：快速操作指令

```bash
exiftool -b -Comment image.jpg > comment_hex.txt
xxd -r -p comment_hex.txt > private.pem
openssl pkeyutl -decrypt -inkey private.pem -in flag.enc
```
