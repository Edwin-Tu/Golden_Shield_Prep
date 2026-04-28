# picoCTF 2024 - Super SSH 解題教學

## 題目資訊

- 題目名稱：Super SSH
- 題目類型：General Skills / SSH
- 難度：簡單
- 目標：使用 SSH 連線到 picoCTF 提供的遠端主機，成功登入後取得 flag。

---

## 題目描述

題目提供一組 SSH 連線資訊，要求我們透過終端機連上遠端伺服器。

題目畫面提供的資訊如下：

```text
ssh ctf-player@titan.picoctf.net -p 52622
```

密碼如下：

```text
f3b61b38
```

SSH 指令格式為：

```bash
ssh 使用者名稱@主機位址 -p 連接埠
```

套用到本題：

```bash
ssh ctf-player@titan.picoctf.net -p 52622
```

---

## 解題環境

本次操作環境為 WSL / Linux 終端機：

```bash
edwintu@EdwinTu:/mnt/c/Users/hc105/Downloads$
```

---

## 解題流程

### Step 1：第一次嘗試連線錯誤

一開始輸入了：

```bash
ssh ctf-player@titan.picoctf.net 52662
```

終端機回應：

```text
ctf-player@titan.picoctf.net: Permission denied (publickey).
```

這個錯誤的原因是 SSH 指令格式不正確。

在 SSH 指令中，如果要指定 port，必須使用 `-p` 參數。

錯誤寫法：

```bash
ssh ctf-player@titan.picoctf.net 52662
```

正確寫法應為：

```bash
ssh ctf-player@titan.picoctf.net -p 52662
```

---

### Step 2：使用錯誤的 port

接著輸入：

```bash
ssh ctf-player@titan.picoctf.net -p 52662
```

終端機回應：

```text
ssh: connect to host titan.picoctf.net port 52662: Connection refused
```

這代表 SSH 指令格式已經正確，但是 port 號碼錯了，或該 port 沒有開放服務。

根據題目截圖，正確的 port 是：

```text
52622
```

不是：

```text
52662
```

---

### Step 3：使用正確 SSH 指令連線

輸入題目提供的正確指令：

```bash
ssh ctf-player@titan.picoctf.net -p 52622
```

第一次連線時，系統會出現主機驗證訊息：

```text
The authenticity of host '[titan.picoctf.net]:52622' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

這是 SSH 第一次連到該主機時的安全確認。

輸入：

```text
yes
```

接著系統會要求輸入密碼：

```text
ctf-player@titan.picoctf.net's password:
```

輸入題目提供的密碼：

```text
f3b61b38
```

注意：在 Linux / WSL 終端機輸入密碼時，畫面不會顯示任何字元，這是正常現象。

---

## 成功取得 Flag

成功登入後，遠端主機直接顯示 flag：

```text
Welcome ctf-player, here's your flag: picoCTF{s3cur3_c0nn3ct10n_3e293eea}
```

因此本題 flag 為：

```text
picoCTF{s3cur3_c0nn3ct10n_3e293eea}
```

---

## 完整操作紀錄

```bash
edwintu@EdwinTu:/mnt/c/Users/hc105/Downloads$ ssh ctf-player@titan.picoctf.net 52662
The authenticity of host 'titan.picoctf.net (3.139.174.234)' can't be established.
ED25519 key fingerprint is SHA256:ouhiHD6XQKuHntnkUUSngcHUBWF2ZN+NYh6D5p565nM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'titan.picoctf.net' (ED25519) to the list of known hosts.
ctf-player@titan.picoctf.net: Permission denied (publickey).

edwintu@EdwinTu:/mnt/c/Users/hc105/Downloads$ ssh ctf-player@titan.picoctf.net -p 52662
ssh: connect to host titan.picoctf.net port 52662: Connection refused

edwintu@EdwinTu:/mnt/c/Users/hc105/Downloads$ ssh ctf-player@titan.picoctf.net -p 52622
The authenticity of host '[titan.picoctf.net]:52622 ([3.139.174.234]:52622)' can't be established.
ED25519 key fingerprint is SHA256:4S9EbTSSRZm32I+cdM5TyzthpQryv5kudRP9PIKT7XQ.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[titan.picoctf.net]:52622' (ED25519) to the list of known hosts.
ctf-player@titan.picoctf.net's password:
Welcome ctf-player, here's your flag: picoCTF{s3cur3_c0nn3ct10n_3e293eea}
Connection to titan.picoctf.net closed.
```

---

## 指令觀念整理

### SSH 是什麼？

SSH，全名是 Secure Shell，是一種安全連線到遠端電腦的方式。

CTF 題目常會提供：

- 使用者名稱
- 主機位址
- port
- 密碼

讓我們透過 SSH 登入遠端環境。

---

### SSH 指令格式

```bash
ssh username@hostname -p port
```

本題對應如下：

| 欄位 | 內容 |
|---|---|
| username | ctf-player |
| hostname | titan.picoctf.net |
| port | 52622 |
| password | f3b61b38 |

所以完整指令為：

```bash
ssh ctf-player@titan.picoctf.net -p 52622
```

---

## 常見錯誤說明

### 錯誤 1：忘記加 `-p`

錯誤指令：

```bash
ssh ctf-player@titan.picoctf.net 52662
```

問題原因：

`52662` 不會被 SSH 當成 port，而可能被視為其他參數，因此連線方式錯誤。

正確方式：

```bash
ssh ctf-player@titan.picoctf.net -p 52622
```

---

### 錯誤 2：port 打錯

錯誤指令：

```bash
ssh ctf-player@titan.picoctf.net -p 52662
```

錯誤訊息：

```text
Connection refused
```

問題原因：

該 port 沒有開放 SSH 服務，或題目提供的 port 不是這個。

解法：

回到題目頁面確認正確 port。

本題正確 port 是：

```text
52622
```

---

### 錯誤 3：輸入密碼時看不到字

這不是錯誤。

Linux / WSL 輸入密碼時不會顯示任何字元，也不會顯示 `*`。

只要正常輸入密碼後按 Enter 即可。

---

## 解題重點

本題重點不是漏洞利用，而是熟悉基本 SSH 操作：

1. 看懂題目提供的 SSH 連線資訊。
2. 使用 `ssh username@host -p port` 格式連線。
3. 第一次連線時輸入 `yes` 接受主機驗證。
4. 輸入題目提供的密碼。
5. 成功登入後取得 flag。

---

## 最終答案

```text
picoCTF{s3cur3_c0nn3ct10n_3e293eea}
```
