# CTF 教學文件：從 PCAP 封包中找出隱藏的 Flag

## 題目概念

這題提供了一個封包擷取檔 `myNetworkTraffic.pcap`，目標是從網路封包中找出被隱藏的 `flag`。

這類題目常出現在 CTF 的：

- Forensics（鑑識）
- Network（網路分析）
- Beginner Web / Misc 延伸題

本題的核心不是分析完整的 TCP 連線流程，而是：

**從封包的 payload 中找出可疑字串，判斷是否為 Base64，解碼後再依時間順序拼接出完整 flag。**

---

## 題目檔案

- 檔名：`myNetworkTraffic.pcap`
- 類型：PCAP（Packet Capture）封包擷取檔

### PCAP 是什麼？

PCAP 是一種記錄網路封包的檔案格式，可以把它理解成：

> **網路世界的錄影檔**

它記錄了某段時間內網路通訊的封包內容，例如：

- 來源 IP / 目的 IP
- 使用的協定（TCP、UDP、HTTP、DNS…）
- 傳輸時間
- 封包內資料（payload）

在 CTF 中，常見用途有：

- 找登入帳密
- 找 HTTP 傳輸的檔案
- 找 DNS 查詢紀錄
- 從 payload 內挖出 flag

---

## 本題解題思路

一開始不要急著假設是 HTTP 或 DNS 題，應該先做基本檢查：

1. 確認檔案格式
2. 看封包數量多不多
3. 觀察封包是否有明顯協定特徵
4. 檢查 payload 是否有可疑字串
5. 判斷是否需要解碼（如 Base64、Hex、URL encode）

這題的特徵是：

- 檔案不大
- 封包數量少
- 多數封包長得很像
- payload 內出現很多類似 Base64 的字串

所以合理推斷：

> 題目很可能把 flag 切成數段藏在封包資料內。

---

## 使用工具

本題可使用以下工具：

- `file`：辨識檔案類型
- `Wireshark`：圖形化觀察封包
- `strings`：快速看可見字串（可選）
- `base64`：解碼 Base64
- `Python`：自動化萃取 payload（進階）

---

## 步驟一：確認檔案類型

在 WSL / Linux 中輸入：

```bash
file myNetworkTraffic.pcap
```

預期會看到它是一個封包擷取檔。

這一步的目的不是解題，而是先確認：

- 檔案是不是正常
- 題目是不是給錯副檔名
- 檔案是否需要用網路分析工具打開

---

## 步驟二：用 Wireshark 開啟檔案

如果你有安裝 Wireshark，直接開啟 `myNetworkTraffic.pcap`。

### 在 Wireshark 中要看什麼？

先觀察：

- 封包總數是否很少
- 是否集中在某種協定（例如 TCP）
- 每個封包的資料區有沒有可疑內容

點進單一封包後，可以看下方詳細資訊：

- `Internet Protocol Version 4`
- `Transmission Control Protocol`
- `Data` 或最底部的原始內容

如果你發現 payload 中有這種字串：

```text
cGljb0NURg==
ezF0X3c0cw==
XzM0c3lfdA==
fQ==
```

就要立刻想到：

> 這很像 Base64。

---

## 步驟三：判斷 Base64 特徵

Base64 常見特徵：

- 由英文字母、數字、`+`、`/` 組成
- 長度通常看起來很整齊
- 常以 `=` 或 `==` 結尾

例如：

```text
cGljb0NURg==
```

這就是非常典型的 Base64 字串。

可以直接在終端機中測試：

```bash
echo 'cGljb0NURg==' | base64 -d
```

輸出：

```text
picoCTF
```

再測試另一段：

```bash
echo 'ezF0X3c0cw==' | base64 -d
```

輸出：

```text
{1t_w4s
```

這時你就可以確定：

> 這題的 flag 被拆成多段後，再用 Base64 編碼藏進封包裡。

---

## 步驟四：把有效片段一段一段解出來

從封包中可找出數個有效的 Base64 字串，解碼後會得到以下內容：

```text
picoCTF
{1t_w4s
nt_th4t
_34sy_t
bh_4r_d
1065384
}
```

乍看之下已經很接近完整 flag，但還有一個重點：

> **不能只依照你隨手看到的順序拼接。**

因為有些題目會故意把片段打亂，必須依封包時間排序。

---

## 步驟五：依封包時間排序

在 Wireshark 中，每個封包都有時間欄位。

你應該依照封包的時間順序，重新排列這些解碼後的片段。

排序後正確順序如下：

```text
picoCTF
{1t_w4s
nt_th4t
_34sy_t
bh_4r_d
1065384
}
```

將它們直接拼接後，得到完整 flag：

```text
picoCTF{1t_w4snt_th4t_34sy_tbh_4r_d1065384}
```

---

## 最終答案

```text
picoCTF{1t_w4snt_th4t_34sy_tbh_4r_d1065384}
```

---

## WSL 手動解題流程範例

以下示範一個比較偏新手的手動流程。

### 1. 確認檔案

```bash
file myNetworkTraffic.pcap
```

### 2. 視需要看可見字串

```bash
strings myNetworkTraffic.pcap
```

有時候這一步就能先看到可疑的 Base64 字串。

### 3. 把可疑字串逐個解碼

```bash
echo 'cGljb0NURg==' | base64 -d
```

```bash
echo 'ezF0X3c0cw==' | base64 -d
```

```bash
echo 'XzM0c3lfdA==' | base64 -d
```

### 4. 依照時間排序拼回完整字串

最後得到：

```text
picoCTF{1t_w4snt_th4t_34sy_tbh_4r_d1065384}
```

---

## Python 自動化做法（進階）

如果你想練習自動化分析，可以用 Python 直接從 pcap 中取出 TCP payload，並嘗試 Base64 解碼。

```python
import struct
import base64

pcap_file = "myNetworkTraffic.pcap"

with open(pcap_file, "rb") as f:
    data = f.read()

offset = 24  # 略過 pcap global header
packets = []

while offset < len(data):
    ts_sec, ts_usec, incl_len, orig_len = struct.unpack_from("<IIII", data, offset)
    offset += 16
    pkt = data[offset:offset + incl_len]
    offset += incl_len

    # IPv4 header 長度
    ihl = (pkt[0] & 0x0F) * 4
    ip_total_len = struct.unpack("!H", pkt[2:4])[0]
    ip_payload = pkt[ihl:ip_total_len]

    # TCP header 長度
    tcp_data_offset = (ip_payload[12] >> 4) * 4
    tcp_payload = ip_payload[tcp_data_offset:]

    try:
        raw = tcp_payload.decode().strip()
        decoded = base64.b64decode(raw).decode("ascii")
        if all(32 <= ord(c) < 127 for c in decoded):
            packets.append((ts_sec, ts_usec, raw, decoded))
    except Exception:
        pass

# 依時間排序
packets.sort(key=lambda x: (x[0], x[1]))

for ts_sec, ts_usec, raw, decoded in packets:
    print(f"{ts_sec}.{ts_usec} | {raw} -> {decoded}")

flag_parts = []
started = False

for _, _, _, decoded in packets:
    if decoded == "picoCTF":
        started = True
    if started:
        flag_parts.append(decoded)
    if decoded == "}":
        break

print("\nFLAG =", "".join(flag_parts))
```

### 這段程式做了什麼？

- 讀取 pcap 檔
- 跳過 pcap 標頭
- 逐包解析 IPv4 與 TCP
- 取出 TCP payload
- 嘗試把 payload 當成 Base64 解碼
- 篩出可列印字元
- 依時間排序
- 拼接出 flag

這種做法很適合：

- 封包很多時
- 想避免手動一個一個點
- 想練習寫自己的簡易鑑識工具

---

## 這題學到的重點

### 1. PCAP 題不一定是分析協定流程

有些題目不是要你看三向交握、ACK、HTTP response，而是要你直接看 payload。

### 2. 看到 `=`、`==` 結尾要想到 Base64

Base64 是 CTF 非常常見的基礎編碼方式。

### 3. Flag 可能被切片儲存

不要期待一次就看到完整 flag，很多題目都會把它拆成數段。

### 4. 封包順序不一定等於資料正確順序

這題需要注意時間欄位，不能亂拼。

### 5. Wireshark 與終端工具可以互補

- Wireshark 適合觀察
- `base64` 適合快速驗證
- Python 適合批次處理

---

## 延伸練習建議

如果你想把這題真正學熟，可以再做以下練習：

1. 嘗試只用 Wireshark 完成，不寫程式
2. 嘗試只用終端機工具完成
3. 修改 Python 程式，讓它自動只抓 Base64 樣式的資料
4. 練習其他編碼判斷：
   - Hex
   - URL Encode
   - ROT13
   - XOR

---

## 常見誤區

### 誤區 1：看到 TCP 就只盯著連線流程

這樣容易忽略真正藏資料的 payload。

### 誤區 2：沒有先判斷資料格式

如果看見可疑字串卻沒想到 Base64，會卡很久。

### 誤區 3：解出片段就直接亂拼

有些題目就是故意利用順序混淆你。

### 誤區 4：過度依賴單一工具

CTF 很常需要：

- Wireshark + Linux 指令
- Linux 指令 + Python
- 觀察 + 驗證 交互進行

---

## 結語

這題是很典型、很適合初學者練習的 PCAP 題，因為它同時訓練了：

- 封包觀察能力
- 對資料格式的敏感度
- Base64 判斷與解碼
- 對封包時間順序的理解
- 使用 Wireshark 與命令列工具的基本能力

只要你能從這題建立一個習慣：

> **先看 payload，再判斷資料格式，再做解碼驗證**

之後遇到很多類似的 Network / Forensics 題都會更快上手。

---

## 參考答案

```text
picoCTF{1t_w4snt_th4t_34sy_tbh_4r_d1065384}
```
