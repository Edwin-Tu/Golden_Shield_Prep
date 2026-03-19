# Golden Shield Prep

這是一個用於金盾獎 / CTF / 資安學習合作練習的 GitHub 儲存庫。

## 目標
- 建立團隊共同學習與解題紀錄
- 整理 PicoCTF 與其他 CTF 的練習成果
- 累積資安工具使用筆記
- 建立常見資安知識庫
- 作為金盾獎賽前複習與協作平台

## 目錄
- `ctf/`：CTF 題目練習紀錄與 writeup
- `tools/`：資安工具使用說明與筆記
- `knowledge/`：資安知識整理
- `practice/`：練習腳本、lab 紀錄、個人筆記模板
- `team/`：團隊成員、分工、進度追蹤
- `docs/`：路線圖、會議紀錄、參考資料

## 協作方式
1. 建立自己的 branch
2. 新增或修改筆記
3. 提交 Pull Request
4. 由其他成員 review 後合併

## 建議 commit 類型
- `docs:` 文件更新
- `feat:` 新增內容或模板
- `fix:` 修正錯誤
- `chore:` 結構整理

## 練習方向
- Web Security
- Cryptography
- Reverse Engineering
- Forensics
- OSINT
- Network Security

## 注意事項
- 不要上傳敏感資訊、帳密、token、真實目標資料
- 練習內容以合法平台與授權環境為主
- writeup 應以學習紀錄為目的

## 儲存庫骨架

golden-shield-prep/
├─ README.md
├─ CONTRIBUTING.md
├─ CODE_OF_CONDUCT.md
├─ LICENSE
├─ .gitignore
├─ docs/
│  ├─ roadmap.md
│  ├─ meeting-notes/
│  │  └─ README.md
│  ├─ references/
│  │  └─ useful-links.md
│  └─ glossary.md
├─ ctf/
│  ├─ README.md
│  ├─ picoctf/
│  │  ├─ README.md
│  │  ├─ web/
│  │  │  └─ README.md
│  │  ├─ crypto/
│  │  │  └─ README.md
│  │  ├─ reverse/
│  │  │  └─ README.md
│  │  ├─ forensics/
│  │  │  └─ README.md
│  │  ├─ binary/
│  │  │  └─ README.md
│  │  └─ templates/
│  │     └─ writeup-template.md
│  └─ other-ctf/
│     └─ README.md
├─ tools/
│  ├─ README.md
│  ├─ burpsuite/
│  │  └─ burpsuite-basics.md
│  ├─ wireshark/
│  │  └─ wireshark-basics.md
│  ├─ nmap/
│  │  └─ nmap-basics.md
│  ├─ gobuster/
│  │  └─ gobuster-basics.md
│  ├─ sqlmap/
│  │  └─ sqlmap-basics.md
│  ├─ john/
│  │  └─ john-the-ripper-basics.md
│  └─ metasploit/
│     └─ metasploit-basics.md
├─ knowledge/
│  ├─ README.md
│  ├─ web-security/
│  │  ├─ xss.md
│  │  ├─ sqli.md
│  │  ├─ csrf.md
│  │  └─ file-upload.md
│  ├─ cryptography/
│  │  ├─ encodings.md
│  │  ├─ classical-ciphers.md
│  │  └─ hash-and-salt.md
│  ├─ reverse-engineering/
│  │  ├─ strings-ghidra-basics.md
│  │  └─ assembly-intro.md
│  ├─ forensics/
│  │  ├─ metadata.md
│  │  ├─ pcap-basics.md
│  │  └─ disk-image-basics.md
│  └─ osint/
│     └─ osint-basics.md
├─ practice/
│  ├─ labs/
│  │  └─ README.md
│  ├─ scripts/
│  │  ├─ README.md
│  │  └─ examples/
│  │     └─ sample-parser.py
│  └─ notes/
│     └─ personal-template.md
├─ team/
│  ├─ members.md
│  ├─ roles.md
│  ├─ weekly-plan.md
│  └─ progress-tracker.md
└─ assets/
   └─ images/
      └─ README.md