## Challenge Metadata

- **Platform:** picoCTF
- **Category:** Web Exploitation
- **Difficulty:** Easy
- **Author:** Bryan
- **Date:** 2025-12-12

### 一、基本資訊

**題目名稱：** logon  
**題目類型：** Web Exploitation  
**平台：** picoCTF  
**目標：** 以 Joe 的身份登入系統並取得隱藏的 flag

### 二、題目概述

本題提供一個名為 Factory Login 的網站，使用者可透過登入系統進入平台。

根據題目描述：

> The factory is hiding things from all of its users.
> Can you login as Joe and find what they've been looking at?

題目提示系統對所有使用者隱藏了一些資訊，而攻擊者需要成功 以 Joe 的身份登入 才能看到該資訊。

初步觀察網站時，登入後頁面會顯示：

```
Success: You logged in! Not sure you'll be able to see the flag though.
No flag for you
```

這表示即使成功登入，系統仍然沒有顯示 flag，暗示網站的 身份驗證機制存在漏洞。

### 三、分析目標

本次分析的主要目標如下：

- 分析網站登入流程
- 找出身份驗證機制中的弱點
- 嘗試偽造使用者身份
- 成功以 Joe 的身份登入
- 取得 flag

### 四、初步觀察

在登入任意帳號後，頁面會顯示登入成功訊息，但仍然無法看到 flag。

透過瀏覽器 Developer Tools 檢查 HTTP 回應，可發現網站使用 Cookie 來記錄使用者身份。

例如：

```
username=guest
```

或類似的身份資訊會被儲存在 Cookie 中。

Cookie 是儲存在 客戶端（瀏覽器） 的資料，使用者可以自行修改其內容。

若伺服器未對 Cookie 內容進行驗證，則可能導致 身份偽造（Authentication Bypass）。

### 五、弱點判定

#### 5.1 弱點描述

本題的網站在驗證使用者身份時，直接信任 Cookie 中的資料。

也就是說，伺服器會依照 Cookie 中的值來判斷目前登入的使用者，例如：

```
username=guest
```

當網站收到請求時，可能會使用類似以下的邏輯：

```
if cookie["username"] == "Joe":
    show_flag()
else:
    hide_flag()
```

由於 Cookie 可以被客戶端任意修改，攻擊者只需將 Cookie 中的使用者名稱改為 Joe，即可繞過身份驗證。

#### 5.2 弱點類型

- Broken Authentication
- Client-Side Authentication
- Cookie Manipulation

#### 5.3 風險說明

由於伺服器完全信任來自客戶端的 Cookie，攻擊者可以透過修改 Cookie 偽造任意使用者身份。

在實際環境中，此類漏洞可能導致：

- 非授權帳號存取
- 管理員權限取得
- 敏感資料外洩

### 六、攻擊思路

攻擊流程如下：

1. 使用任意帳號登入網站
2. 開啟瀏覽器 Developer Tools
3. 找到網站設定的 Cookie
4. 修改 Cookie 中的使用者名稱
5. 重新整理頁面

若 Cookie 被改為：

```
username=Joe
```

伺服器便會誤以為目前登入的使用者是 Joe，從而顯示隱藏的資訊。

### 七、利用過程

#### 7.1 登入網站

首先使用任意帳號登入網站。

登入後畫面顯示：

```
Success: You logged in!
No flag for you
```

#### 7.2 開啟 Developer Tools

按下：

- F12

進入瀏覽器開發者工具。

選擇：

- Application → Cookies

找到網站的 Cookie。

#### 7.3 修改 Cookie

將原本的 Cookie：

```
username=guest
```

修改為：

```
username=Joe
```

#### 7.4 重新整理頁面

刷新頁面後，伺服器會依照修改後的 Cookie 判斷使用者身份。

此時系統會將使用者視為 Joe，並顯示隱藏的 flag。

### 八、利用結果

成功取得 flag：

```
picoCTF{...}
```

（實際 flag 依題目實例而定）

### 九、攻擊流程總結

攻擊流程如下：

1. 登入網站
2. 開啟瀏覽器 DevTools
3. 修改 Cookie
4. 偽造身份為 Joe
5. 刷新頁面取得 flag

### 十、成因分析

本題漏洞產生的主要原因是 伺服器將身份驗證邏輯放在客戶端。

問題點包括：

- 直接信任 Cookie
- 伺服器沒有驗證 Cookie 的真實性。
- 缺乏簽章機制
- Cookie 未使用 HMAC 或數位簽章保護。
- 身份驗證邏輯設計錯誤
- 身份資訊應儲存在伺服器端，而非完全依賴客戶端資料。

### 十一、防禦建議

為避免此類漏洞，可採取以下安全措施：

#### 11.1 使用 Server-Side Session

應只在 Cookie 中儲存 session ID，而不是直接儲存使用者身份。

#### 11.2 Cookie 簽章驗證

使用 HMAC 或 JWT 簽章來確保 Cookie 未被竄改。

#### 11.3 最小化客戶端信任

伺服器不應直接信任任何來自客戶端的資料，包括：

- Cookie
- Header
- URL 參數

#### 11.4 實作安全身份驗證機制

可採用成熟框架的驗證系統，例如：

- OAuth
- JWT
- Session-based authentication

### 十二、結論

本題展示了一個典型的 Cookie-based Authentication Bypass 漏洞。

由於網站將使用者身份直接儲存在 Cookie 中，且未對其進行任何驗證，攻擊者只需修改 Cookie 即可偽造身份並取得原本不應顯示的資訊。

此案例說明，在 Web 應用程式中，若身份驗證設計不當，即使沒有複雜漏洞，也可能導致嚴重的安全問題。

### 十三、附錄：關鍵觀念整理

項目 | 說明
---|---
弱點名稱 | Cookie Authentication Bypass
漏洞類型 | Broken Authentication
攻擊方式 | 修改 Cookie 偽造身份
利用條件 | 伺服器直接信任 Cookie
攻擊效果 | 取得未授權資料
本題成果 | 偽造 Joe 身份取得 flag

## 使用工具

- Browser Developer Tools
- Cookie editor
- HTTP analysis
