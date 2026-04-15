## Challenge Metadata

- **Platform:** picoCTF
- **Category:** Web Exploitation
- **Difficulty:** Easy
- **Author:** Bryan
- **Date:** 2026-3-27

## 一、基本資訊

**題目名稱：** Old Sessions  
**題目類型：** Web Exploitation  
**平台：** picoCTF 2026  
**目標：** 取得管理員身分並讀出頁面中的 flag

## 二、題目概述

本題提供一個登入與註冊頁面，題目敘述強調站點設計成 once you login, you never have to log-out again，並暗示 session 可能長期有效且沒有正確過期。

玩家需要在網站中觀察登入後的 session 行為，找到管理員的有效 session，並透過替換 cookie 的方式冒用管理員身分以取得 flag。

本題的核心並不在於帳密爆破，而是在於：

- 理解 session 是登入狀態的真正憑證
- 觀察 cookie 中的 session 值
- 發現敏感的 session 洩漏點
- 透過 session hijacking 冒用 admin 身分

## 三、分析目標

本次分析的主要目標如下：

- 建立一個普通使用者帳號並登入
- 觀察登入後產生的 session cookie
- 找出網站暴露 session 的功能路徑
- 取得 admin 對應的有效 session
- 替換 cookie 並取得 flag

## 四、環境觀察

題目首頁為標準的 Login / Register 頁面。未登入前，Cookies 區域為空；登入成功後，瀏覽器會收到一個名為 session 的 cookie。

**觀察結果**

登入後 Cookies 中出現：

session = hZ08eNf5IPkso5r-pFiS-QI3WlskE875yPioDpN3klY

這表示網站以 session cookie 作為使用者身分識別依據。

## 五、第一層弱點：Session 長期有效

### 5.1 題目暗示

題目文字明確指出：

- once you login, you never have to log-out again
- session may remain active indefinitely

這表示該應用可能將 session 設為永久有效。

### 5.2 安全意義

若 session 不會過期，則：

- 同一瀏覽器的後續使用者可能繼承登入狀態
- 被竊取的 token 可長期重用
- session hijacking 的風險顯著提高

## 六、第二層弱點：/sessions 洩漏有效 session

### 6.1 觀察行為

登入普通帳號後，進一步瀏覽：

/sessions

結果頁面直接列出目前有效的 session 與對應帳號：

1) session:CQyJvXtpkwPJoaxKS7DIOAyH0uWUZK4PANApCqAfti0, {'_permanent': True, 'key': 'admin'}

2) session:hZ08eNf5IPkso5r-pFiS-QI3WlskE875yPioDpN3klY, {'_permanent': True, 'key': '0'}

### 6.2 弱點本質

這表示網站將：

- 有效 session token
- session 對應使用者
- admin 登入狀態

直接暴露給一般使用者。

這是非常嚴重的 session information disclosure。

## 七、攻擊思路

本題利用鏈如下：

- 註冊並登入普通帳號
- → 取得自己的 session cookie
- → 存取 /sessions
- → 找出 key 為 admin 對應的 session
- → 將自己瀏覽器中的 session cookie 改為 admin 的值
- → 重新整理首頁
- → 成功以 admin 身分取得 flag

## 八、利用過程

### 8.1 建立普通帳號

先註冊並登入帳號：

username: 0  
password: 0

登入後取得自己的 session cookie。

### 8.2 觀察 cookie

在瀏覽器 DevTools 的 Application → Cookies 中可見：

session = hZ08eNf5IPkso5r-pFiS-QI3WlskE875yPioDpN3klY

### 8.3 存取 /sessions

瀏覽：

http://dolphin-cove.picoctf.net:59203/sessions

頁面列出：

- admin 對應 session：CQyJvXtpkwPJoaxKS7DIOAyH0uWUZK4PANApCqAfti0
- 使用者 0 對應 session：hZ08eNf5IPkso5r-pFiS-QI3WlskE875yPioDpN3klY

### 8.4 替換 session cookie

將原本 cookie 值：

hZ08eNf5IPkso5r-pFiS-QI3WlskE875yPioDpN3klY

改成 admin 的：

CQyJvXtpkwPJoaxKS7DIOAyH0uWUZK4PANApCqAfti0

### 8.5 重新整理首頁

重新整理網站首頁後，伺服器將該 session 視為 admin，成功顯示 flag。

## 九、利用結果

取得的 flag：

picoCTF{s3t_s3ss10n_3xp1rat10n5_77b6684a}

## 十、完整 exploit 核心

```text
1. Login as normal user
2. Visit /sessions
3. Copy the session value whose key is admin
4. Replace current browser cookie session value
5. Refresh homepage
```

## 十一、成因分析

### 11.1 Session 管理不當

系統將 session 設為 permanent，造成舊 session 長期有效。

### 11.2 敏感資訊外洩

/sessions 頁面不應對一般使用者開放，因其直接暴露 admin 與其他使用者的有效 session token。

### 11.3 缺乏額外綁定條件

伺服器僅根據 session token 即判定身分，未進一步驗證裝置、活動時間或其他風險訊號。

## 十二、防禦建議

### 12.1 不可暴露 session store

任何列出有效 session 的管理路徑都必須嚴格限制權限。

### 12.2 設定合理過期時間

應同時實作：

- idle timeout
- absolute timeout

避免舊 session 長期可被重用。

### 12.3 登出應銷毀 session

使用者登出時應移除伺服器端 session，而非只清除前端狀態。

### 12.4 高敏感帳號需更嚴格保護

管理員 session 應有更短時效、重新驗證與異常偵測機制。

## 使用工具

- browser DevTools
- Cookies
- session hijacking
- manual cookie editing
