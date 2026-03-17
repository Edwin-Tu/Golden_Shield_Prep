# CTF 練習紀錄（CTF Practice）

本目錄用於記錄各類 CTF（Capture The Flag）題目的解題過程（writeup）、分類整理與學習成果。

---

## 🎯 目的

- 累積解題經驗與思考流程
- 建立團隊知識庫
- 快速複習常見題型與解法
- 作為金盾獎賽前準備資料

---

## 📂 目錄結構

```
ctf/
├─ picoctf/          # PicoCTF 練習
│  ├─ web/
│  ├─ crypto/
│  ├─ reverse/
│  ├─ forensics/
│  └─ binary/
│
└─ other-ctf/        # 其他 CTF（如 HTB、THM、比賽）
```

---

## ✍️ Writeup 撰寫規範

每一題請包含以下內容：

1. **題目資訊**
   - 題目名稱：
   - 題型（Web / Crypto / Reverse / Forensics / OSINT）：
   - 來源（PicoCTF / 比賽名稱）：
   - 難度（Easy / Medium / Hard）：

2. **題目描述**
   - 簡述題目給的資訊（避免直接複製全部題目內容）。

3. **解題思路**
   - 問題分析
   - 嘗試過的方法
   - 關鍵突破點

4. **解題步驟**
   - 請條列式描述：
     - 做了什麼
     - 使用什麼工具
     - 得到什麼結果
   - 👉 必要時附上指令或程式碼：

     ```bash
     nmap -sV target.com
     ```

5. **Flag**
   - `flag{...}`

6. **心得（可選）**
   - 學到什麼
   - 哪裡卡住
   - 下次可以怎麼更快

---

## 🧠 題型分類說明

| 類型       | 說明                  |
|------------|-----------------------|
| Web       | 網頁漏洞（XSS、SQLi 等） |
| Crypto    | 密碼學、編碼          |
| Reverse   | 逆向工程              |
| Forensics | 檔案與封包分析        |
| OSINT     | 開源情報              |
| Binary    | pwn / binary exploitation |

---

## 🛠️ 常用工具

- Burp Suite
- Wireshark
- Nmap
- CyberChef
- Ghidra

---

## 🔄 協作方式

建立 branch

新增 writeup（依分類放置）

發送 Pull Request

經 review 後合併

📌 命名規則
資料夾命名
challenge-name/
檔案命名
README.md

👉 每題一個資料夾 + 一個 README.md

🚫 注意事項

不得上傳敏感資訊（帳密、token）

不得包含非法行為或未授權攻擊內容

尊重 CTF 平台規範（部分比賽禁止公開 writeup）

📈 建議學習方式

每人每週至少完成 2～3 題

寫 writeup（強制）

定期分享解題思路

將關鍵技巧整理至 knowledge/

🚀 最終目標

讓團隊達到：

快速辨識題型

有系統地解題

可複用的解題流程

團隊協作解題能力