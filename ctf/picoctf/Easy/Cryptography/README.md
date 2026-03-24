# 📘 picoCTF 密碼學教學前瞻（六題統整）

## 🎯 本教學目標
透過這六題，你其實已經接觸到 CTF Cryptography 最核心的入門能力：

> **「看到資料 → 判斷類型 → 選工具 → 驗證 → 還原」**

---

# 🧠 一、你學到的核心能力

## 1️⃣ 模式辨識（最重要）
👉 **CTF 密碼學 = pattern recognition（模式辨識）**

---

## 2️⃣ 多層解碼思維
👉 **資料可能不只一層**
Base64 → Base64 → Caesar

---

## 3️⃣ 基礎優先思維
👉 很多題不是難，是你想太難

---

# 🛠️ 二、你已掌握的工具

## 🔹 Linux 指令
- base64 -d
- md5sum / sha1sum / sha256sum
- nc

## 🔹 Python
- codecs.decode
- base64.b64decode
- pow()

---

# 🔐 三、你學到的密碼學類型

## 古典密碼
- ROT13 / Caesar
- A1Z26

## 編碼
- Base64

## Hash
- MD5 / SHA1 / SHA256

## 現代密碼
- RSA

---

# 🧩 四、CTF 解題標準流程

1. 觀察
2. 判斷類型
3. 套工具
4. 驗證結果

---

# 🚀 五、你的能力定位

✅ 已具備 CTF 密碼學入門能力  
✅ 能辨識常見題型  
✅ 能使用工具解題  

---

# 📈 六、下一步建議

- Caesar brute force
- Base32 / Base85
- hashcat
- RSA 攻擊手法

---

# 🧾 總結

👉 密碼學題 = 辨識 + 工具

🔥 一句話：
看到資料 → 判斷 → 解碼 → 得 flag
