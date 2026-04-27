# picoCTF 2025 - FANTASY CTF Writeup

## 題目資訊

- **題目名稱：** FANTASY CTF
- **平台：** picoCTF 2025
- **分類：** General Skills
- **難度：** Easy
- **標籤：** browser_webshell_solvable
- **連線方式：**

```bash
nc verbal-sleep.picoctf.net 57024
```

## 題目描述

題目要求玩家透過 `netcat` 連線到遠端程式，並遊玩一段簡短的互動式文字遊戲。

官方描述大意：

```text
Play this short game to get familiar with terminal applications and some of the
most important rules in scope for picoCTF.
```

這題主要目標不是技術破解，而是讓玩家熟悉：

1. 如何使用終端機互動。
2. 如何使用 `nc` 連線到遠端服務。
3. picoCTF 的基本規則，例如不要共用帳號、不要提交別人的 flag、不要分享 artifact downloads。
4. 在文字互動遊戲中依照提示選擇正確選項。

## 使用工具

本題只需要使用：

```bash
nc
```

也就是 `netcat`。

如果系統沒有安裝，可以使用：

### Ubuntu / WSL / Debian

```bash
sudo apt update
sudo apt install netcat-openbsd -y
```

### Kali Linux

通常已經內建 `nc`，如果沒有：

```bash
sudo apt install netcat-openbsd -y
```

## 解題流程

### 1. 使用 nc 連線

根據題目提供的指令：

```bash
nc verbal-sleep.picoctf.net 57024
```

執行後會進入互動式劇情遊戲。

畫面一開始會顯示：

```text
FANTASY CTF SIMULATION
```

之後會出現多段劇情文字，每段文字後面都會要求：

```text
(Press Enter to continue...)
```

這時只要按 Enter 繼續即可。

## 互動選項分析

### 第一個選項：註冊帳號方式

程式會出現：

```text
Options:
A) *Register multiple accounts*
B) *Share an account with a friend*
C) *Register a single, private account*
[a/b/c] >
```

這裡要選：

```text
c
```

原因：

- picoCTF 不允許註冊多個帳號來取得不公平優勢。
- 不應該跟朋友共用帳號。
- 正確做法是註冊一個自己的私人帳號。

輸入：

```text
c
```

程式會回應這是正確選擇，並說明多帳號或共用帳號可能導致被取消資格。

### 第二個選項：是否搜尋別人的 flag

之後程式會提示 sanity challenge，並出現：

```text
Options:
A) *Play the game*
B) *Search the Ether for the flag*
[a/b] >
```

如果選擇 `b`，程式會提醒：

```text
You don't want to submit a different players flag!
That's against the rules!
```

因此正確觀念是：不要搜尋或提交其他玩家的 flag。

如果誤選 `b`，通常不會直接失敗，程式會給你規則提醒，接著讓你重新選擇。

正確選項是：

```text
a
```

也就是自己玩遊戲取得 flag。

### 第三步：完成遊戲

選擇：

```text
a
```

後，程式會顯示：

```text
Playing the Game
Playing the Game: 100%|██████████████████████████████████████ [time left: 00:00]
Playing the Game completed successfully!
```

代表遊戲完成。

接著繼續按 Enter，就會看到 flag。

## 實際操作紀錄

完整連線指令：

```bash
nc verbal-sleep.picoctf.net 57024
```

重要互動流程如下：

```text
Options:
A) *Register multiple accounts*
B) *Share an account with a friend*
C) *Register a single, private account*
[a/b/c] > c
```

接著按 Enter 繼續劇情。

之後出現：

```text
Options:
A) *Play the game*
B) *Search the Ether for the flag*
[a/b] > b
```

這裡選 `b` 後，程式會提醒不要提交其他玩家的 flag。

接著再次出現：

```text
Options:
A) *Play the game*
B) *Search the Ether for the flag*
[a/b] > A
```

輸入 `A` 或 `a` 都可以。

遊戲完成後出現：

```text
Thanks, Nyx! Here's the flag I found: picoCTF{m1113n1um_3d1710n_3b6c6fab}
```

## Flag

```text
picoCTF{m1113n1um_3d1710n_3b6c6fab}
```

## 一行解法概念

由於這題是互動式劇情，最穩定的方式是手動操作：

```bash
nc verbal-sleep.picoctf.net 57024
```

然後依序：

1. 按 Enter 推進劇情。
2. 第一題選 `c`。
3. 繼續按 Enter。
4. 第二題選 `a`。
5. 繼續按 Enter 直到看到 flag。

如果要自動化，可以使用 `printf` 預先送出多個換行與選項，但由於互動節奏可能會因伺服器輸出而不同，初學者建議手動完成。

## 常見錯誤

### 錯誤 1：以為需要破解程式

這題不是 buffer overflow、密碼破解或檔案分析題。

它的目的主要是教你：

- 使用 `nc`
- 閱讀終端機輸出
- 依照互動式提示輸入選項
- 理解 picoCTF 規則

### 錯誤 2：選擇搜尋別人的 flag

選項：

```text
B) Search the Ether for the flag
```

這代表去找別人的 flag，不符合比賽規則。

正確做法是選：

```text
A) Play the game
```

### 錯誤 3：關閉終端機太早

有些 flag 會在劇情最後才出現，因此完成選項後還要繼續按 Enter，直到看到類似：

```text
Here's the flag I found:
```

## 總結

FANTASY CTF 是一題入門導覽型題目，主要考察：

1. 是否會用 `nc` 連線遠端服務。
2. 是否能閱讀終端機中的互動提示。
3. 是否理解 CTF 比賽規範。
4. 是否能在文字遊戲中選擇正確選項並取得 flag。

最終取得的 flag 為：

```text
picoCTF{m1113n1um_3d1710n_3b6c6fab}
```
