# 術語表

此術語表包含與 Capture The Flag (CTF) 挑戰、網路安全以及 Golden Shield Prep 項目中使用的相關工具相關的關鍵術語和定義。

## 一般術語

- **CTF (Capture The Flag)**: 一種網路安全競賽，參與者解決挑戰以尋找隱藏的「旗標」（文字字串）。
- **Flag**: 參與者必須找到的秘密字串或代碼，以完成挑戰。
- **Challenge**: CTF 中的任務或謎題，需要特定技能來解決。
- **Category**: 挑戰的分組，例如網頁、密碼學、取證等。
- **Writeup**: 詳細說明挑戰如何被解決的解釋。

## 密碼學

- **Cryptography**: 通過代碼和密碼保護通訊和資料的實踐。
- **Encryption**: 將明文轉換為密文以防止未經授權的存取的過程。
- **Decryption**: 將密文轉換回明文的過程。
- **Symmetric Encryption**: 使用相同金鑰進行加密和解密的加密（例如 AES）。
- **Asymmetric Encryption**: 使用一對金鑰的加密：公鑰和私鑰（例如 RSA）。
- **Hash Function**: 將輸入資料轉換為固定大小字元字串的函數（例如 SHA-256）。
- **Salt**: 在雜湊前添加到密碼的隨機資料，以防止彩虹表攻擊。
- **Rainbow Table**: 用於破解密碼雜湊的預計算雜湊值表。
- **Brute Force**: 嘗試所有可能組合來破解密碼或代碼。
- **Dictionary Attack**: 使用常見單詞或短語列表來猜測密碼。

## 取證

- **Digital Forensics**: 調查和分析數位裝置以獲取證據的過程。
- **Disk Image**: 儲存裝置整個內容的副本。
- **Metadata**: 關於資料的資料，例如檔案建立日期、作者等。
- **PCAP**: 封包捕獲檔案，包含網路流量資料。
- **Steganography**: 在非秘密檔案或訊息中隱藏秘密資訊。

## 網頁安全

- **XSS (Cross-Site Scripting)**: 允許攻擊者將惡意腳本注入網頁的漏洞。
- **SQL Injection (SQLi)**: 利用 SQL 查詢漏洞的代碼注入技術。
- **CSRF (Cross-Site Request Forgery)**: 誘騙使用者在網頁應用程式上執行不想要動作的攻擊。
- **File Upload Vulnerability**: 允許攻擊者將惡意檔案上傳到伺服器的漏洞。

## 逆向工程

- **Reverse Engineering**: 分析軟體以了解其設計、架構或提取資訊。
- **Assembly Language**: 直接對應機器碼的低階程式語言。
- **Strings**: 從二進位檔案中提取可讀字串的工具。
- **Ghidra**: NSA 開發的軟體逆向工程工具。

## OSINT (Open Source Intelligence)

- **OSINT**: 從公開可用來源收集資訊。
- **Social Engineering**: 操縱人們透露機密資訊。

## 工具

- **Nmap**: 用於發現主機和服務的網路掃描工具。
- **Wireshark**: 用於捕獲和檢查封包的網路協定分析器。
- **Burp Suite**: 網頁漏洞掃描器和代理工具。
- **Metasploit**: 用於開發和執行漏洞利用的框架。
- **John the Ripper**: 密碼破解工具。
- **Gobuster**: 目錄和檔案暴力破解工具。
- **SQLMap**: 自動化 SQL 注入工具。

## 網路

- **IP Address**: 分配給網路中每個裝置的唯一位址。
- **MAC Address**: 分配給網路介面的唯一識別碼。
- **DNS (Domain Name System)**: 將網域名稱轉譯為 IP 位址。
- **HTTP/HTTPS**: 用於傳輸網頁的協定（HTTPS 是安全的）。
- **Firewall**: 監控和控制進出流量的網路安全系統。
- **IDS (Intrusion Detection System)**: 監控網路流量以檢測可疑活動。
- **IPS (Intrusion Prevention System)**: 封鎖檢測到的威脅。

## 其他術語

- **Exploit**: 利用漏洞的代碼或技術片段。
- **Vulnerability**: 系統中可以被利用的弱點。
- **Payload**: 漏洞利用中執行惡意動作的部分。
- **Malware**: 設計用來危害或利用系統的惡意軟體。
- **Phishing**: 通過偽裝成可信任實體來獲取敏感資訊的欺詐嘗試。
- **Rootkit**: 隱藏系統上惡意軟體存在的軟體。