# Binary Digits

## Challenge Metadata
- **Challenge Name:** Binary Digits
- **Category:** Forensics
- **Difficulty:** Easy
- **Event:** picoCTF 2026
- **Author:** YAHAYA MEDDY

## 一、基本資訊
本題提供一個僅由 `0` 與 `1` 組成的檔案 `digits.bin`。題目提示指出此檔案看似只是大量的 1 與 0，但實際上並非隨機雜訊，而是可被還原成有意義內容的資料。

## 二、題目概述
本題的核心在於將 bitstream 還原成原始位元組資料，再根據檔頭判斷檔案型別，最後從還原後的內容中取得 flag。這是一道標準的資料重建（data reconstruction）與檔頭識別（file signature identification）題。

## 三、分析目標
1. 確認 `digits.bin` 是否為純二進位字串資料。
2. 將 0/1 bitstream 以每 8 bit 為一組還原成 bytes。
3. 識別還原後檔案的實際格式。
4. 從輸出內容中擷取最終 flag。

## 四、分析過程

### 4.1 Bitstream 還原
將檔案中的 `0` 與 `1` 連續讀出，忽略其他非 bit 字元，然後每 8 個 bit 切分為 1 byte，轉回原始位元組序列。

範例腳本如下：

```python
from pathlib import Path

data = Path("digits.bin").read_text()
bits = ''.join(c for c in data if c in '01')
out = bytes(int(bits[i:i+8], 2) for i in range(0, len(bits), 8))
Path("decoded.jpg").write_bytes(out)
print("saved to decoded.jpg")
```

### 4.2 檔頭識別
還原後的資料開頭為：

```text
FF D8 FF E0 00 10 4A 46 49 46
```

此 signature 對應標準 JPEG / JFIF 檔頭，表示 `digits.bin` 實際上封裝的是一張 JPEG 圖片。

### 4.3 內容檢視
將還原出的檔案存成 `decoded.jpg` 後開啟，即可從圖片中直接讀出 flag。

## 五、解題結果
最終取得的 flag 為：

```text
picoCTF{h1dd3n_1n_th3_b1n4ry_8d00e35f}
```

## 六、關鍵觀念
- **Bitstream Reconstruction：** 將 0/1 文本還原為原始 bytes。
- **Magic Bytes / File Signature：** 利用檔頭判定實際檔案型別。
- **Forensic Recovery：** 題目表面上是文本，實際上隱藏完整媒體檔案。

## 七、弱點與成因分析
此題並非傳統漏洞利用，而是檔案內容被以低階格式隱藏。出題重點在於：
1. 攻擊者或分析者不應只以檔案外觀判斷內容。
2. 看到規律性極高的 `0/1` 資料時，應立即聯想到 bitstream、ASCII、bitmap 或其他編碼形式。
3. 還原後再做檔頭識別是標準鑑識分析流程。

## 八、防禦建議
1. 在檔案分析流程中加入自動 bitstream 偵測與重建模組。
2. 建立 magic bytes 檢查機制，不只依副檔名或表面內容判定檔案類型。
3. 在 DFIR 或 CTF 工具鏈中加入自動 artifact reconstruction 能力，以提升隱藏內容的檢出率。

## 九、結論
`Binary Digits` 是一道基礎但具有代表性的數位鑑識題。解題關鍵不在複雜演算法，而在於辨識資料模式、還原原始位元組並正確識別檔案格式。完成 bitstream 還原後即可從 JPEG 圖片中擷取 flag。

## 十、附錄：快速操作指令

```bash
python3 - <<'PY'
from pathlib import Path
data = Path("digits.bin").read_text()
bits = ''.join(c for c in data if c in '01')
out = bytes(int(bits[i:i+8], 2) for i in range(0, len(bits), 8))
Path("decoded.jpg").write_bytes(out)
print("saved to decoded.jpg")
PY
```
