# 學習路線圖（Roadmap）

本文件用於規劃團隊在金盾獎與 CTF 競賽前的學習方向、階段目標與技能成長路徑。

---

## 🎯 總體目標

- 建立完整資安知識體系
- 提升 CTF 解題能力（速度 + 正確率）
- 熟悉常見工具與攻擊手法
- 建立團隊協作與解題流程
- 為金盾獎比賽做好實戰準備

---

## 🧭 學習階段

### 🟢 Phase 1：基礎建立（1～2 週）

📌 目標：建立資安基本觀念 + 熟悉環境

**內容：**
- Linux 基礎操作（檔案、權限、指令）
- 基本網路概念（TCP/IP、HTTP、DNS）
- Python 基礎（for parsing / scripting）
- 常見編碼（Base64、Hex、ASCII）

**工具：**
- Wireshark（基本封包觀察）
- CyberChef（編碼轉換）

**練習平台：**
- PicoCTF（入門題）
- OverTheWire（Bandit）

---

### 🟡 Phase 2：核心能力（2～4 週）

📌 目標：掌握 CTF 常見題型

#### 🌐 Web Security
- XSS（反射型 / 儲存型）
- SQL Injection
- CSRF
- File Upload 漏洞

**工具：**
- Burp Suite
- Gobuster
- sqlmap

---

#### 🔐 Cryptography
- 古典密碼（Caesar、Vigenère）
- XOR
- Hash（MD5、SHA）
- 基本 RSA 概念

---

#### 🔍 Forensics
- 檔案 metadata 分析
- 圖片隱寫（Steganography）
- PCAP 分析

---

#### 🧠 Reverse Engineering
- strings / 基本分析
- Ghidra / IDA 基礎
- 簡單 assembly 概念

---

### 🔵 Phase 3：實戰強化（3～6 週）

📌 目標：提升解題速度與整合能力

**內容：**
- 跨領域題型（Web + Crypto / Forensics + OSINT）
- 題目拆解與思考流程訓練
- 常見題型套路整理

**實作：**
- 每週 mock CTF（團隊解題）
- Writeup 撰寫與分享
- 建立解題模板（templates）

---

### 🔴 Phase 4：比賽準備（2～3 週）

📌 目標：最佳化團隊戰力

**內容：**
- 分工策略（Web / Crypto / Reverse / Forensics）
- 解題流程標準化
- 快速資訊共享（Notion / Markdown / Discord）

**模擬：**
- 限時 CTF 模擬（3～6 小時）
- 壓力測試（時間限制 + 題量）

---

## 🛠️ 工具學習路線

| 類別 | 工具 |
|------|------|
| Web | Burp Suite, Gobuster, sqlmap |
| Network | Wireshark, tcpdump |
| Forensics | Autopsy, exiftool |
| Reverse | Ghidra, IDA |
| General | CyberChef |

---

## 📚 學習資源

- PicoCTF
- Hack The Box（HTB）
- TryHackMe（THM）
- OverTheWire

---

## 🧩 團隊運作建議

- 每週至少 1 次 CTF 練習
- 每人負責 1～2 個領域（專精）
- 建立 writeup 知識庫
- 定期分享學習成果

---

## 📈 進度追蹤

建議使用 `team/progress-tracker.md` 記錄：

- 完成題數
- 掌握技能
- 工具熟悉度
- 弱項分析

---

## 🚫 注意事項

- 僅在合法與授權環境中練習
- 不使用資安技能進行非法行為
- 不外流比賽題解（依規定）

---

## 🧠 最終目標

讓團隊達到：

- 能快速辨識題型
- 有系統地拆解問題
- 熟練使用工具
- 具備團隊協作解題能力
