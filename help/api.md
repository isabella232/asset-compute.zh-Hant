---
title: "[!DNL Asset Compute Service] HTTP API"
description: "[!DNL Asset Compute Service] HTTP API建立自定義應用程式。"
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: 93d3b407c8875888f03bec673d0a677a3205cfbb
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 2%

---

# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

API的使用僅限於開發目的。 在開發自定義應用程式時，API被作為上下文提供。 [!DNL Adobe Experience Manager] 作為 [!DNL Cloud Service] 使用API將處理資訊傳遞給自定義應用程式。 有關詳細資訊，請參見 [使用資產微服務和處理配置檔案](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

>[!NOTE]
>
>[!DNL Asset Compute Service] 僅供使用 [!DNL Experience Manager] 作為 [!DNL Cloud Service]。

任何客戶 [!DNL Asset Compute Service] HTTP API必須遵循此高級流：

1. 將客戶端設定為 [!DNL Adobe Developer Console] IMS組織的項目。 每個獨立的客戶端（系統或環境）都需要自己的獨立項目來分隔事件資料流。

1. 客戶端使用 [JWT（服務帳戶）身份驗證](https://www.adobe.io/authentication/auth-methods.html)。

1. 客戶端呼叫 [`/register`](#register) 只檢索一次日誌URL。

1. 客戶端呼叫 [`/process`](#process-request) 要為其生成格式副本的每個資產。 呼叫是非同步的。

1. 客戶定期輪詢日誌 [接收事件](#asynchronous-events)。 在成功處理格式副本時，它會接收每個請求格式副本的事件(`rendition_created` 事件類型)或如果出現錯誤(`rendition_failed` 事件類型)。

的 [@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client) 模組使Node.js代碼中的API易於使用。

## 驗證和授權 {#authentication-and-authorization}

所有API都需要訪問令牌驗證。 請求必須設定以下標頭：

1. `Authorization` 帶有承載令牌的頭，即技術帳戶令牌，通過 [JWT交換](https://www.adobe.io/authentication/auth-methods.html) Adobe Developer控制台項目。 的 [作用域](#scopes) 下面記錄。

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` IMS組織ID的頭。

1. `x-api-key` 從 [!DNL Adobe Developers Console] 項目。

### 範圍 {#scopes}

確保訪問令牌的以下作用域：

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

這些要求 [!DNL Adobe Developer Console] 要訂閱的項目 `Asset Compute`。 `I/O Events`, `I/O Management API` 服務。 單個作用域的細分為：

* 基本
   * 範圍： `openid,AdobeID`

* asset compute
   * 元目錄： `asset_compute_meta`
   * 範圍： `asset_compute,read_organizations`

* [!DNL Adobe I/O] 事件
   * 元目錄： `event_receiver_api`
   * 範圍： `event_receiver,event_receiver_api`

* [!DNL Adobe I/O] 管理API
   * 元目錄： `ent_adobeio_sdk`
   * 範圍： `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 註冊 {#register}

每個客戶端 [!DNL Asset Compute service]  — 獨特 [!DNL Adobe Developer Console] 訂閱服務的項目 — 必須 [註冊](#register-request) 在發出處理請求之前。 註冊步驟返回唯一事件日誌，該日誌是從格式副本處理中檢索非同步事件所必需的。

在其生命週期結束時，客戶機 [註銷](#unregister-request)。

### 註冊請求 {#register-request}

此API調用設定 [!DNL Asset Compute] 提供事件日誌URL。 這是冪等操作，只需為每個客戶端調用一次。 可以再次調用它以檢索日誌URL。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/register` |
| 頁首 `Authorization` | 全部 [授權相關標頭](#authentication-and-authorization)。 |
| 頁首 `x-request-id` | 可選，可由客戶端為系統間處理請求的唯一端到端標識符設定。 |
| 請求正文 | 必須為空。 |

### 註冊響應 {#register-response}

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME類型 | `application/json` |
| 頁首 `X-Request-Id` | 或 `X-Request-Id` 請求標頭或唯一生成的標頭。 用於識別跨系統和/或支援請求的請求。 |
| 反應體 | JSON對象 `journal`。 `ok` 和/或 `requestId` 的子菜單。 |

HTTP狀態代碼為：

* **200次成功**:請求成功時。 它包含 `journal` 通過以下方式觸發的非同步處理結果的通知的URL `/process` （作為事件類型） `rendition_created` 成功時，或 `rendition_failed` 失敗時)。

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401未授權**:請求無效時發生 [認證](#authentication-and-authorization)。 示例可能是無效的訪問令牌或無效的API密鑰。

* **《小行星403》**:請求無效時發生 [授權](#authentication-and-authorization)。 示例可能是有效的訪問令牌，但Adobe Developer控制台項目（技術帳戶）未訂閱所有必需的服務。

* **429請求過多**:當系統由此客戶端或其他客戶端過載時發生。 客戶端應使用 [指數退避](https://en.wikipedia.org/wiki/Exponential_backoff)。 屍體是空的。
* **4xx錯誤**:出現任何其他客戶端錯誤時，註冊失敗。 通常會返回JSON響應，但並不保證所有錯誤都會返回：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**:出現任何其他伺服器端錯誤並註冊失敗時發生。 通常會返回JSON響應，但並不保證所有錯誤都會返回：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### 註銷請求 {#unregister-request}

此API調用取消註冊 [!DNL Asset Compute] 客戶端。 此後，將無法再呼叫 `/process`。 對未註冊的客戶端或尚未註冊的客戶端使用API調用將返回 `404` 錯誤。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/unregister` |
| 頁首 `Authorization` | 全部 [授權相關標頭](#authentication-and-authorization)。 |
| 頁首 `x-request-id` | 可選，可由客戶端為系統間處理請求的唯一端到端標識符設定。 |
| 請求正文 | 空白. |

### 註銷響應 {#unregister-response}

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME類型 | `application/json` |
| 頁首 `X-Request-Id` | 或 `X-Request-Id` 請求標頭或唯一生成的標頭。 用於識別跨系統和/或支援請求的請求。 |
| 反應體 | JSON對象 `ok` 和 `requestId` 的子菜單。 |

狀態代碼為：

* **200次成功**:在找到並刪除註冊和日記帳時發生。

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401未授權**:請求無效時發生 [認證](#authentication-and-authorization)。 示例可能是無效的訪問令牌或無效的API密鑰。

* **《小行星403》**:請求無效時發生 [授權](#authentication-and-authorization)。 示例可能是有效的訪問令牌，但Adobe Developer控制台項目（技術帳戶）未訂閱所有必需的服務。

* **找不到404**:當給定憑據沒有當前註冊時發生。

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429請求過多**:在系統過載時發生。 客戶端應使用 [指數退避](https://en.wikipedia.org/wiki/Exponential_backoff)。 屍體是空的。

* **4xx錯誤**:出現任何其他客戶端錯誤並註銷失敗時發生。 通常會返回JSON響應，但並不保證所有錯誤都會返回：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**:出現任何其他伺服器端錯誤並註冊失敗時發生。 通常會返回JSON響應，但並不保證所有錯誤都會返回：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## 處理請求 {#process-request}

的 `process` 操作根據請求中的說明提交將源資產轉換為多個格式副本的任務。 成功完成的通知（事件類型） `rendition_created`)或任何錯誤（事件類型） `rendition_failed`)被發送到必須使用 [/register](#register) 在進行任意數量 `/process` 請求。 錯誤格式的請求立即失敗，錯誤代碼為400。

使用URL(如AmazonAWSS3預簽名的URL或Azure Blob儲存SAS URL)引用二進位檔案，以便讀取 `source` 資產(A)`GET` URL)和寫入格式副本(`PUT` URL)。 客戶端負責生成這些預簽名的URL。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/process` |
| MIME類型 | `application/json` |
| 頁首 `Authorization` | 全部 [授權相關標頭](#authentication-and-authorization)。 |
| 頁首 `x-request-id` | 可選，可由客戶端為系統間處理請求的唯一端到端標識符設定。 |
| 請求正文 | 必須採用進程請求JSON格式，如下所述。 它提供了有關要處理的資產和要生成的格式副本的說明。 |

### 進程請求JSON {#process-request-json}

請求正文 `/process` 是具有此高級架構的JSON對象：

```json
{
    "source": "",
    "renditions" : []
}
```

可用欄位包括：

| 名稱 | 類型 | 說明 | 範例 |
|--------------|----------|-------------|---------|
| `source` | `string` | 要處理的源資產的URL。 根據請求的格式副本格式(例如， `fmt=zip`)。 | `"http://example.com/image.jpg"` |
| `source` | `object` | 描述要處理的源資產。 請參閱 [源對象欄位](#source-object-fields) 下。 根據請求的格式副本格式(例如， `fmt=zip`)。 | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | 要從源檔案生成的格式副本。 每個格式副本對象都支援 [格式說明](#rendition-instructions)。 必要. | `[{ "target": "https://....", "fmt": "png" }]` |

的 `source` 可以是 `<string>` 可視為URL，或可以是 `<object>` 的子菜單。 以下變型相似：

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### 源對象欄位 {#source-object-fields}

| 名稱 | 類型 | 說明 | 範例 |
|-----------|----------|-------------|---------|
| `url` | `string` | 要處理的源資產的URL。 必要. | `"http://example.com/image.jpg"` |
| `name` | `string` | 源資產檔案名。 如果未檢測到MIME類型，則可能使用名稱中的檔案副檔名。 優先於URL路徑中的檔案名或 `content-disposition` 二進位資源的標頭。 預設為「file」。 | `"image.jpg"` |
| `size` | `number` | 源資產檔案大小（以位元組為單位）。 優先於 `content-length` 二進位資源的標頭。 | `10234` |
| `mimetype` | `string` | 源資產檔案MIME類型。 優先於 `content-type` 二進位資源的標頭。 | `"image/jpeg"` |

### 完整 `process` 請求示例 {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## 進程響應 {#process-response}

的 `/process` 請求立即返回，並基於基本請求驗證成功或失敗。 實際資產處理非同步進行。

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME類型 | `application/json` |
| 頁首 `X-Request-Id` | 或 `X-Request-Id` 請求標頭或唯一生成的標頭。 用於識別跨系統和/或支援請求的請求。 |
| 反應體 | JSON對象 `ok` 和 `requestId` 的子菜單。 |

狀態代碼：

* **200次成功**:如果請求已成功提交。 響應JSON包括 `"ok": true`:

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400無效請求**:如果請求格式不正確，例如請求JSON中缺少必需欄位。 響應JSON包括 `"ok": false`:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401未授權**:當請求無效時 [認證](#authentication-and-authorization)。 示例可能是無效的訪問令牌或無效的API密鑰。
* **《小行星403》**:當請求無效時 [授權](#authentication-and-authorization)。 示例可能是有效的訪問令牌，但Adobe Developer控制台項目（技術帳戶）未訂閱所有必需的服務。
* **429請求過多**:當系統由此客戶端或通常過載時。 客戶端可以使用 [指數退避](https://en.wikipedia.org/wiki/Exponential_backoff)。 屍體是空的。
* **4xx錯誤**:出現任何其他客戶端錯誤時。 通常會返回JSON響應，但並不保證所有錯誤都會返回：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**:當存在任何其他伺服器端錯誤時。 通常會返回JSON響應，但並不保證所有錯誤都會返回：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

大多數客戶端可能傾向於使用 [指數退避](https://en.wikipedia.org/wiki/Exponential_backoff) 在任何錯誤 *除* 配置問題（如401或403）或無效請求（如400）。 除了通過429個響應來限制正常速率外，臨時服務中斷或限制可能導致5xx錯誤。 建議在一段時間後重試。

所有JSON響應（如果存在）包括 `requestId` 與 `X-Request-Id` 標題。 建議從頭讀取，因為它始終存在。 的 `requestId` 在與處理請求相關的所有事件中也返回為 `requestId`。 客戶端不能對此字串的格式作任何假設，它是不透明的字串標識符。

## 選擇加入後處理 {#opt-in-to-post-processing}

的 [asset computeSDK](https://github.com/adobe/asset-compute-sdk) 支援一組基本影像後處理選項。 通過設定欄位，自定義工作人員可以明確選擇後處理 `postProcess` 格式副本對象上 `true`。

支援的使用案例包括：

* 將格式副本裁剪到其限制由crop.w、crop.h、crop.x和crop.y定義的矩形。它由 `instructions.crop` 的子菜單。
* 使用寬度、高度或兩者調整影像大小。 它由 `instructions.width` 和 `instructions.height` 的子菜單。 要僅使用寬度或高度調整大小，請僅設定一個值。 計算服務可降低縱橫比。
* 設定JPEG影像的質量。 它由 `instructions.quality` 的子菜單。 最佳質量由 `100` 而較小的值表示質量下降。
* 建立隔行掃描影像。 它由 `instructions.interlace` 的子菜單。
* 設定DPI以通過調整應用於像素的比例來調整呈現大小以用於案頭發佈。 它由 `instructions.dpi` 的子菜單。 但是，要調整影像大小以使其以不同的解析度大小相同，請使用 `convertToDpi` 的下界。
* 調整影像的大小，使其渲染的寬度或高度在指定目標解析度(DPI)下保持與原始影像相同。 它由 `instructions.convertToDpi` 的子菜單。

## 浮水印資產 {#add-watermark}

的 [asset computeSDK](https://github.com/adobe/asset-compute-sdk) 支援向PNG、JPEG、TIFF和GIF影像檔案添加水印。 水印將按照 `watermark` 格式副本上的對象。

在格式副本後處理期間執行水印。 要對資產進行水印，自定義工作人員 [後處理](#opt-in-to-post-processing) 設定欄位 `postProcess` 格式副本對象上 `true`。 如果工作程式不選擇加入，則不應用水印，即使在請求中的格式副本對象上設定了水印對象。

## 格式副本說明 {#rendition-instructions}

這些是 `renditions` 陣列 [/進程](#process-request)。

### 公用欄位 {#common-fields}

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | 格式副本目標格式，也可以 `text` 用於文本提取和 `xmp` 將元XMP資料提取為xml。 請參閱 [支援的格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | URL [自定義應用程式](develop-custom-application.md)。 必須是 `https://` URL。 如果存在此欄位，則格式副本由自定義應用程式建立。 然後，在自定義應用程式中使用任何其他集格式副本欄位。 | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | 應使用HTTPPUT將生成的格式副本上載到的URL。 | `http://w.com/img.jpg` |
| `target` | `object` | 生成的格式副本的多部分預簽名URL上載資訊。 這是 [AEM/Oak Direct二進位上載](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) 這 [多部件上載行為](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html)。<br>欄位:<ul><li>`urls`:字串陣列，每個預簽名部件URL對應一個</li><li>`minPartSize`:用於一個部件的最小大小= url</li><li>`maxPartSize`:用於一個部件的最大大小= url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | 由客戶端控制並按原樣傳遞到格式副本事件的可選保留空間。 允許客戶端添加自定義資訊以標識格式副本事件。 在自定義應用程式中，不得修改或依賴客戶端，因為客戶端隨時可以隨意更改。 | `{ ... }` |

### 格式副本特定欄位 {#rendition-specific-fields}

有關當前支援的檔案格式的清單，請參見 [支援的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)。

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `*` | `*` | 可以添加高級、自定義欄位 [自定義應用程式](develop-custom-application.md) 明白。 |  |
| `embedBinaryLimit` | `number` 位元組 | 如果設定了此值，並且格式副本的檔案大小小於此值，則格式副本將嵌入在格式副本生成完成後發送的事件中。 允許嵌入的最大大小為32 KB（32 x 1024位元組）。 如果格式副本的大小大於 `embedBinaryLimit` 限制，它被放置在雲儲存中的某個位置，並且不嵌入事件。 | `3276` |
| `width` | `number` | 寬度（像素）。 僅用於影像格式副本。 | `200` |
| `height` | `number` | 高度（以像素為單位）。 僅用於影像格式副本。 | `200` |
|  |  | 如果： <ul> <li> 兩者 `width` 和 `height` 指定，然後影像適合大小，同時保持長寬比 </li><li> 僅 `width` 僅 `height` 指定，生成的影像在保持縱橫比的同時使用相應的尺寸</li><li> 如果兩者都不 `width` 無 `height` 指定，則使用原始影像像素大小。 這取決於源類型。 對於某些格式(如PDF檔案)，使用預設大小。 可以有最大大小限制。</li></ul> |  |
| `quality` | `number` | 在以下範圍中指定jpeg質量 `1` 至 `100`。 僅適用於影像格式副本。 | `90` |
| `xmp` | `string` | 僅由元數XMP據寫回使用，它是base64編XMP碼以寫回指定的格式副本。 |  |
| `interlace` | `bool` | 通過將交錯PNG或GIF或逐行JPEG設定為 `true`。 它對其他檔案格式沒有影響。 |  |
| `jpegSize` | `number` | JPEG檔案的大小（以位元組為單位）。 它覆蓋任何 `quality` 的子菜單。 對其他格式沒有影響。 |  |
| `dpi` | `number` 或 `object` | 設定x和y DPI。 為簡單起見，還可以將它設定為一個單數，該單數用於x和y。它對影像本身沒有影響。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` 或 `object` | x和y DPI在保持物理大小的同時重新採樣值。 為簡單起見，還可以將它設定為一個單數，該單數用於x和y。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | 要包含在ZIP存檔中的檔案清單(`fmt=zip`)。 每個條目可以是URL字串，也可以是具有以下欄位的對象：<ul><li>`url`:下載檔案的URL</li><li>`path`:將檔案儲存在ZIP中此路徑下</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | ZIP存檔的重複處理(`fmt=zip`)。 預設情況下，儲存在ZIP中同一路徑下的多個檔案會生成錯誤。 設定 `duplicate` 至 `ignore` 只導致要儲存的第一個資產和忽略的其餘資產。 | `ignore` |
| `watermark` | `object` | 包含有關 [水印](#watermark-specific-fields)。 |  |

### 特定於水印的欄位 {#watermark-specific-fields}

PNG格式用作水印。

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 水印的比例，介於 `0.0` 和 `1.0`。 `1.0` 表示水印有其原始比例(1:1)，且較低的值會減小水印大小。 | 值 `0.5` 相當於原來大小的一半。 |
| `image` | `url` | 要用於水印的PNG檔案的URL。 |  |

## 非同步事件 {#asynchronous-events}

一旦完成格式副本的處理或出錯，事件將發送到 [[!DNL Adobe I/O] 事件日記帳](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md)。 客戶端必須偵聽通過提供的日誌URL [/register](#register)。 日誌響應包括 `event` 陣列，每個事件包含一個對象，其中 `event` 欄位包含實際事件負載。

的 [!DNL Adobe I/O] 所有事件的事件類型 [!DNL Asset Compute Service] 是 `asset_compute`。 日記帳僅自動訂閱此事件類型，並且不再需要根據 [!DNL Adobe I/O] 事件類型。 服務特定事件類型可在 `type` 事件的屬性。

### 事件類型 {#event-types}

| Event | 說明 |
|---------------------|-------------|
| `rendition_created` | 為每個成功處理和上載的格式副本發送。 |
| `rendition_failed` | 為無法處理或上載的每個格式副本發送。 |

### 事件屬性 {#event-attributes}

| 屬性 | 類型 | Event | 說明 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | 在簡化擴展中發送事件的時間戳 [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 格式，由JavaScript定義 [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString)。 |
| `requestId` | `string` | `*` | 原始請求的請求ID `/process`，與 `X-Request-Id` 標題。 |
| `source` | `object` | `*` | 的 `source` 的 `/process` 請求。 |
| `userData` | `object` | `*` | 的 `userData` 從 `/process` 的子菜單。 |
| `rendition` | `object` | `rendition_*` | 傳入的相應格式副本對象 `/process`。 |
| `metadata` | `object` | `rendition_created` | 的 [元資料](#metadata) 格式副本的屬性。 |
| `errorReason` | `string` | `rendition_failed` | 格式副本失敗 [原因](#error-reasons) 如果有的話。 |
| `errorMessage` | `string` | `rendition_failed` | 文本提供有關格式副本失敗（如果有）的詳細資訊。 |

### 中繼資料 {#metadata}

| 屬性 | 說明 |
|--------|-------------|
| `repo:size` | 格式副本的大小（以位元組為單位）。 |
| `repo:sha1` | 格式副本的sha1摘要。 |
| `dc:format` | 格式副本的MIME類型。 |
| `repo:encoding` | 格式副本的字元集編碼（如果它是基於文本的格式）。 |
| `tiff:ImageWidth` | 格式副本的寬度（以像素為單位）。 僅用於影像格式副本。 |
| `tiff:ImageLength` | 格式副本的長度（以像素為單位）。 僅用於影像格式副本。 |

### 錯誤原因 {#error-reasons}

| 原因 | 說明 |
|---------|-------------|
| `RenditionFormatUnsupported` | 給定源不支援請求的格式副本格式。 |
| `SourceUnsupported` | 即使支援該類型，也不支援特定源。 |
| `SourceCorrupt` | 源資料已損壞。 包括空檔案。 |
| `RenditionTooLarge` | 無法使用中提供的預簽名URL上載格式副本 `target`。 實際格式副本大小可作為元資料在 `repo:size` 並且可由客戶端使用適當數量的預簽名URL重新處理此格式副本。 |
| `GenericError` | 其他任何意外錯誤。 |
