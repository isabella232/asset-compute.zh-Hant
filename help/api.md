---
title: '[!DNL Asset Compute Service] HTTP API。'
description: '[!DNL Asset Compute Service] 建立自訂應用程式的HTTP API。'
translation-type: tm+mt
source-git-commit: d26ae470507e187249a472ececf5f08d803a636c
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 2%

---


# [!DNL Asset Compute Service] HTTP API  {#asset-compute-http-api}

API的使用僅限於開發用途。 開發自訂應用程式時，API會以內容形式提供。 [!DNL Adobe Experience Manager] as  [!DNL Cloud Service] as uses the API to processing information to a custom application.如需詳細資訊，請參閱[使用資產微服務和處理設定檔](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

>[!NOTE]
>
>[!DNL Asset Compute Service] 只能與 [!DNL Experience Manager] as  [!DNL Cloud Service]

[!DNL Asset Compute Service] HTTP API的任何用戶端都必須遵循此高階流程：

1. 在IMS組織中，用戶端被布建為[!DNL Adobe Developer Console]專案。 每個獨立的客戶端（系統或環境）都需要自己的獨立項目，以分離事件資料流。

1. 客戶端使用[JWT（服務帳戶）驗證](https://www.adobe.io/authentication/auth-methods.html)為技術帳戶生成訪問令牌。

1. 客戶端僅調用[`/register`](#register)一次以檢索日誌URL。

1. 用戶端會針對每個要產生轉譯的資產呼叫[`/process`](#process-request)。 呼叫是非同步的。

1. 客戶端定期輪詢日誌以[接收事件](#asynchronous-events)。 當轉譯成功處理（`rendition_created`事件類型）或發生錯誤（`rendition_failed`事件類型）時，它會接收每個請求轉譯的事件。

[@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client)模組可讓您輕鬆在Node.js程式碼中使用API。

## 驗證和授權{#authentication-and-authorization}

所有API都需要存取Token驗證。 這些請求必須設定下列標題：

1. `Authorization` 標題（即技術帳戶Token），是透過 [JWT ](https://www.adobe.io/authentication/auth-methods.html) exchange從Adobe Developer Console專案接收。[範圍](#scopes)說明如下。

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in AIO's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` 標題（含IMS組織ID）。

1. `x-api-key` 使用專案的用戶端 [!DNL Adobe Developers Console] ID。

### 範圍 {#scopes}

請確定存取Token的下列範圍：

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

這要求[!DNL Adobe Developer Console]專案必須訂閱`Asset Compute`、`I/O Events`和`I/O Management API`服務。 個別範圍的劃分如下：

* 基本
   * 範圍：`openid,AdobeID`

* 資產計算
   * metascope:`asset_compute_meta`
   * 範圍：`asset_compute,read_organizations`

* [!DNL Adobe I/O] 事件
   * metascope:`event_receiver_api`
   * 範圍：`event_receiver,event_receiver_api`

* [!DNL Adobe I/O] 管理API
   * metascope:`ent_adobeio_sdk`
   * 範圍：`adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 註冊{#register}

[!DNL Asset Compute service] —— 預訂服務的唯一[!DNL Adobe Developer Console]項目的每個客戶端在發出處理請求前必須[註冊](#register-request)。 註冊步驟返回唯一事件日誌，該日誌是從格式副本處理中檢索非同步事件所必需的。

在其生命週期結束時，客戶端可以[取消註冊](#unregister-request)。

### 註冊請求{#register-request}

此API呼叫會設定[!DNL Asset Compute]用戶端，並提供事件日誌URL。 這是一個冪等運算，每個用戶端只需呼叫一次。 可以再次調用它以檢索日記賬URL。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/register` |
| 頁首 `Authorization` | 所有[授權相關標頭](#authentication-and-authorization)。 |
| 頁首 `x-request-id` | 可選，可由客戶端為系統間處理請求的唯一端到端標識符進行設定。 |
| 請求正文 | 必須是空的。 |

### 註冊響應{#register-response}

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME類型 | `application/json` |
| 頁首 `X-Request-Id` | 與`X-Request-Id`請求標題相同，或是唯一產生的標題。 用於識別跨系統的請求和／或支援請求。 |
| 響應體 | 具有`journal`、`ok`和／或`requestId`欄位的JSON物件。 |

HTTP狀態代碼為：

* **200次成功**:請求成功時。它包含`journal` URL，會收到透過`/process`觸發之非同步處理結果的通知（如果成功，則為事件類型`rendition_created`，或失敗時為`rendition_failed`）。

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401未授權**:當請求沒有有效驗證時 [發生](#authentication-and-authorization)。範例可能是無效的存取Token或無效的API金鑰。

* **403禁止**:當請求沒有有效授權時 [發生](#authentication-and-authorization)。範例可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）並未訂閱所有必要服務。

* **429請求太多**:當系統由此客戶端過載或其它情況下發生。客戶端應使用[指數回退](https://en.wikipedia.org/wiki/Exponential_backoff)重試。 屍體是空的。
* **4xx錯誤**:發生其他客戶端錯誤時，註冊失敗。通常會傳回JSON回應，例如此回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**:發生於發生任何其他伺服器端錯誤且註冊失敗時。通常會傳回JSON回應，例如此回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### 註銷請求{#unregister-request}

此API呼叫會取消註冊[!DNL Asset Compute]用戶端。 在此之後，無法再呼叫`/process`。 使用未註冊客戶機或尚未註冊客戶機的API調用返回`404`錯誤。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/unregister` |
| 頁首 `Authorization` | 所有[授權相關標頭](#authentication-and-authorization)。 |
| 頁首 `x-request-id` | 可選，可由客戶端為系統間處理請求的唯一端到端標識符進行設定。 |
| 請求正文 | 空白. |

### 註銷響應{#unregister-response}

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME類型 | `application/json` |
| 頁首 `X-Request-Id` | 與`X-Request-Id`請求標題相同，或是唯一產生的標題。 用於識別跨系統的請求和／或支援請求。 |
| 響應體 | 具有`ok`和`requestId`欄位的JSON物件。 |

狀態代碼為：

* **200次成功**:發生於找到並刪除註冊和日誌時。

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401未授權**:當請求沒有有效驗證時 [發生](#authentication-and-authorization)。範例可能是無效的存取Token或無效的API金鑰。

* **403禁止**:當請求沒有有效授權時 [發生](#authentication-and-authorization)。範例可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）並未訂閱所有必要服務。

* **404找不到**:當給定憑據沒有當前註冊時發生。

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429請求太多**:當系統過載時發生。客戶端應使用[指數回退](https://en.wikipedia.org/wiki/Exponential_backoff)重試。 屍體是空的。

* **4xx錯誤**:發生在發生任何其他客戶端錯誤並取消註冊失敗時。通常會傳回JSON回應，例如此回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**:發生於發生任何其他伺服器端錯誤且註冊失敗時。通常會傳回JSON回應，例如此回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## 處理請求{#process-request}

`process`操作會根據請求中的指示，提交將來源資產轉換為多個轉譯的作業。 有關成功完成（事件類型`rendition_created`）或任何錯誤（事件類型`rendition_failed`）的通知會傳送至事件日誌，事件日誌必須使用[/register](#register)擷取一次，才能發出任何數量的`/process`請求。 錯誤格式的請求會立即失敗，並出現400錯誤碼。

使用URL（如Amazon AWS S3預簽的URL或Azure Blob Storage SAS URL）來引用二進位檔案，以讀取`source`資產(`GET` URL)和寫入轉譯(`PUT` URL)。 用戶端負責產生這些預先簽署的URL。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/process` |
| MIME類型 | `application/json` |
| 頁首 `Authorization` | 所有[授權相關標頭](#authentication-and-authorization)。 |
| 頁首 `x-request-id` | 可選，可由客戶端為系統間處理請求的唯一端到端標識符進行設定。 |
| 請求正文 | 必須採用程式請求JSON格式，如下所述。 它提供處理哪些資產以及產生哪些轉譯的指示。 |

### 處理請求JSON {#process-request-json}

`/process`的請求主體是具有此高階架構的JSON物件：

```json
{
    "source": "",
    "renditions" : []
}
```

可用欄位包括：

| 名稱 | 類型 | 說明 | 範例 |
|--------------|----------|-------------|---------|
| `source` | `string` | 要處理的來源資產URL。 根據要求的轉譯格式(例如，`fmt=zip`)。 | `"http://example.com/image.jpg"` |
| `source` | `object` | 說明要處理的來源資產。 請參閱下面[源對象欄位](#source-object-fields)的說明。 根據要求的轉譯格式(例如，`fmt=zip`)。 | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | 從來源檔案產生的轉譯。 每個轉譯對象支援[轉譯指令](#rendition-instructions)。 必要. | `[{ "target": "https://....", "fmt": "png" }]` |

`source`可以是被視為URL的`<string>`，也可以是具有附加欄位的`<object>`。 下列變體類似：

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### 源對象欄位{#source-object-fields}

| 名稱 | 類型 | 說明 | 範例 |
|-----------|----------|-------------|---------|
| `url` | `string` | 要處理的來源資產URL。 必要. | `"http://example.com/image.jpg"` |
| `name` | `string` | 來源資產檔案名稱。 如果未檢測到MIME類型，則可使用名稱中的檔案副檔名。 優先於二進位資源`content-disposition`標題中URL路徑或檔案名。 預設為「檔案」。 | `"image.jpg"` |
| `size` | `number` | 來源資產檔案大小（以位元組為單位）。 優先於二進位資源的`content-length`標頭。 | `10234` |
| `mimetype` | `string` | 來源資產檔案MIME類型。 優先於二進位資源的`content-type`標頭。 | `"image/jpeg"` |

### 完整的`process`請求範例{#complete-process-request-example}

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

## 進程響應{#process-response}

`/process`請求會根據基本請求驗證立即傳回成功或失敗。 實際資產處理會非同步進行。

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME類型 | `application/json` |
| 頁首 `X-Request-Id` | 與`X-Request-Id`請求標題相同，或是唯一產生的標題。 用於識別跨系統的請求和／或支援請求。 |
| 響應體 | 具有`ok`和`requestId`欄位的JSON物件。 |

狀態代碼：

* **200次成功**:如果請求已成功提交。回應JSON包含`"ok": true`:

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400無效請求**:如果請求的格式不正確，例如請求JSON中遺失的必要欄位。回應JSON包含`"ok": false`:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401未授權**:當請求沒有有效驗 [證](#authentication-and-authorization)。範例可能是無效的存取Token或無效的API金鑰。
* **403禁止**:當請求沒有有效的授 [權](#authentication-and-authorization)。範例可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）並未訂閱所有必要服務。
* **429請求太多**:當系統由此客戶端或通常由此客戶端過載時。客戶端可以使用[指數回退](https://en.wikipedia.org/wiki/Exponential_backoff)重試。 屍體是空的。
* **4xx錯誤**:發生其他客戶機錯誤時。通常會傳回JSON回應，例如此回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**:發生其他伺服器端錯誤時。通常會傳回JSON回應，例如此回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

多數客戶可能會傾向於針對除&#x200B;*組態問題（例如401或403）或無效請求（如400）以外的任何錯誤*，以[指數式回退](https://en.wikipedia.org/wiki/Exponential_backoff)重試完全相同的請求。 除了透過429個回應的一般速率限制外，暫時服務中斷或限制可能會導致5xx錯誤。 然後建議在一段時間後重試。

所有JSON回應（如果存在）都包含與`X-Request-Id`標題相同的`requestId`值。 建議從標題中讀取，因為它始終存在。 在與處理請求相關的所有事件中，`requestId`也會傳回為`requestId`。 用戶端不得對此字串的格式作任何假設，它是不透明字串識別碼。

## 選擇加入後處理{#opt-in-to-post-processing}

[資產計算SDK](https://github.com/adobe/asset-compute-sdk)支援一組基本影像後處理選項。 自訂工作者可明確選擇加入後處理，方法是將轉譯物件上的欄位`postProcess`設為`true`。

支援的使用案例包括：

* 將轉譯裁切為限制由crop.w、crop.h、crop.x和crop.y定義的矩形。它由`instructions.crop`在轉譯對象中定義。
* 使用寬度、高度或兩者調整影像大小。 它由`instructions.width`和`instructions.height`在轉譯對象中定義。 若要僅使用寬度或高度來調整大小，請只設定一個值。 計算服務控制長寬比。
* 設定JPEG影像的品質。 它由`instructions.quality`在轉譯對象中定義。 最佳質量由`100`表示，較小的值表示質量降低。
* 建立隔行掃描影像。 它由`instructions.interlace`在轉譯對象中定義。
* 設定DPI，以調整套用至像素的比例，以調整轉譯大小以用於案頭出版。 它由`instructions.dpi`在轉譯物件中定義，以變更dpi解析度。 不過，若要調整影像大小，使其以不同解析度顯示相同大小，請使用`convertToDpi`指示。
* 調整影像大小，使其轉譯寬度或高度在指定目標解析度(DPI)時仍與原始影像相同。 它由`instructions.convertToDpi`在轉譯對象中定義。

## 浮水印資產{#add-watermark}

[Asset Compute SDK](https://github.com/adobe/asset-compute-sdk)支援將浮水印新增至PNG、JPEG、TIFF和GIF影像檔案。 水印會依照轉譯中`watermark`物件的轉譯指示來新增。

在轉譯後處理期間會執行浮水印。 若要浮水印資產，自訂工作者[會將轉譯物件上的欄位`postProcess`設為`true`，以選擇後處理](#opt-in-to-post-processing)。 如果工作者不選擇加入，則不會套用浮水印，即使在請求的轉譯物件上設定了浮水印物件。

## 轉譯說明{#rendition-instructions}

這些是[/process](#process-request)中`renditions`陣列的可用選項。

### 常見欄位{#common-fields}

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | 轉譯目標格式也可以是`text`用於文字擷取，`xmp`用於將XMP中繼資料擷取為xml。 請參閱[支援的格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | [自訂應用程式的URL](develop-custom-application.md)。 必須是`https://` URL。 如果此欄位存在，則由自訂應用程式建立轉譯。 然後，任何其他設定的轉譯欄位都會用於自訂應用程式。 | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | 使用HTTP PUT上傳產生轉譯的URL。 | `http://w.com/img.jpg` |
| `target` | `object` | 產生轉譯的多部分預先簽署的URL上傳資訊。 這適用於具有此[多部分上傳行為](http://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html)的[AEM/Oak Direct Binary Upload](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html)。<br>欄位:<ul><li>`urls`:字串陣列，每個預先簽署的部件URL各一個</li><li>`minPartSize`:一個部件的最小大小= url</li><li>`maxPartSize`:用於一個部件的最大大小= url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | 由用戶端控制並依原樣傳遞至轉譯事件的選擇性保留空間。 可讓用戶端新增自訂資訊，以識別轉譯事件。 自訂應用程式中不得修改或依賴用戶端，因為用戶端隨時都可自由變更。 | `{ ... }` |

### 轉譯特定欄位{#rendition-specific-fields}

有關當前支援的檔案格式的清單，請參見[支援的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)。

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `*` | `*` | 可新增[自訂應用程式](develop-custom-application.md)瞭解的進階自訂欄位。 |  |
| `embedBinaryLimit` | `number` 位元組 | 如果已設定此值，且轉譯的檔案大小小於此值，則轉譯會內嵌在轉譯產生完成後傳送的事件中。 允許嵌入的最大大小為32 KB（32 x 1024位元組）。 如果轉譯的大小大於`embedBinaryLimit`限制，則會將其放在雲端儲存空間的某個位置，且不會嵌入事件中。 | `3276` |
| `width` | `number` | 寬度（像素）。 僅限影像轉譯。 | `200` |
| `height` | `number` | 高度（像素）。 僅限影像轉譯。 | `200` |
|  |  | 如果： <ul> <li> 同時指定`width`和`height`，然後影像符合大小並維持長寬比 </li><li> 只指定`width`或僅指定`height`，產生的影像會使用對應的尺寸，同時保持高寬比</li><li> 如果未指定`width`或`height`，則會使用原始影像像素大小。 這取決於源類型。 對於某些格式（例如PDF檔案），會使用預設大小。 可以有最大大小限制。</li></ul> |  |
| `quality` | `number` | 在`1`至`100`的範圍內指定jpeg品質。 僅適用於影像轉譯。 | `90` |
| `xmp` | `string` | 只有XMP中繼資料回寫使用，才會使用base64編碼的XMP來回寫回指定的轉譯。 |  |
| `interlace` | `bool` | 將隔行掃描的PNG或GIF或漸進式JPEG設為`true`。 它對其他檔案格式沒有影響。 |  |
| `jpegSize` | `number` | JPEG檔案的大小（以位元組為單位）。 它會覆寫任何`quality`設定。 對其他格式沒有影響。 |  |
| `dpi` | `number` 或 `object` | 設定x和y DPI。 為簡單起見，它也可以設為單一數字，用於x和y。它對影像本身沒有影響。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` 或 `object` | x和y DPI會重新取樣值，同時維持實體大小。 為簡單起見，它也可以設為單一數字，用於x和y。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | 要包含在ZIP檔案中的檔案清單(`fmt=zip`)。 每個項目可以是URL字串或具有下列欄位的物件：<ul><li>`url`:下載檔案的URL</li><li>`path`:將檔案儲存在ZIP中此路徑下</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | ZIP封存檔的處理重複(`fmt=zip`)。 依預設，儲存在ZIP中相同路徑下的多個檔案會產生錯誤。 將`duplicate`設為`ignore`只會產生要儲存的第一個資產，而忽略其餘的資產。 | `ignore` |
| `watermark` | `object` | 包含有關[watermark](#watermark-specific-fields)的說明。 |  |

### 水印特定欄位{#watermark-specific-fields}

PNG格式會用作浮水印。

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 在`0.0`和`1.0`之間縮放水印。 `1.0` 意指水印有其原始縮放比例(1:1)，而較低的值會降低水印大小。 | 值`0.5`表示原始大小的一半。 |
| `image` | `url` | 用於浮水印的PNG檔案的URL。 |  |

## 非同步事件{#asynchronous-events}

一旦格式副本的處理完成或發生錯誤時，事件便會傳送至[[!DNL Adobe I/O] 事件日誌](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md)。 客戶必須監聽通過[/register](#register)提供的日誌URL。 日誌響應包括一個`event`陣列，該陣列由每個事件的一個對象組成，其中`event`欄位包括實際事件有效負荷。

[!DNL Adobe I/O][!DNL Asset Compute Service]的所有事件的事件類型為`asset_compute`。 日記帳只會自動訂閱此事件類型，並且不再需要根據[!DNL Adobe I/O]事件類型進行篩選。 服務特定事件類型在事件的`type`屬性中可用。

### 事件類型{#event-types}

| 事件 | 說明 |
|---------------------|-------------|
| `rendition_created` | 針對每個成功處理和上傳的轉譯傳送。 |
| `rendition_failed` | 針對每個無法處理或上傳的轉譯傳送。 |

### 事件屬性{#event-attributes}

| 屬性 | 類型 | 事件 | 說明 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | 事件以簡化的延伸[ISO-8601](https://en.wikipedia.org/wiki/ISO_8601)格式傳送的時間戳記，如JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString)所定義。 |
| `requestId` | `string` | `*` | 原始請求至`/process`的請求ID，與`X-Request-Id`標題相同。 |
| `source` | `object` | `*` | `/process`請求的`source`。 |
| `userData` | `object` | `*` | 如果已設定，則`/process`請求的轉譯`userData`。 |
| `rendition` | `object` | `rendition_*` | 傳入`/process`的對應轉譯物件。 |
| `metadata` | `object` | `rendition_created` | 轉譯的[metadata](#metadata)屬性。 |
| `errorReason` | `string` | `rendition_failed` | 轉譯失敗[reason](#error-reasons)（如果有）。 |
| `errorMessage` | `string` | `rendition_failed` | 文字會提供更詳細的轉譯失敗（如果有的話）。 |

### 中繼資料 {#metadata}

| 屬性 | 說明 |
|--------|-------------|
| `repo:size` | 格式副本的大小（以位元組為單位）。 |
| `repo:sha1` | 轉譯的sha1摘要。 |
| `dc:format` | 轉譯的MIME類型。 |
| `repo:encoding` | 轉譯的字元集編碼，以防其為文字格式。 |
| `tiff:ImageWidth` | 轉譯的寬度（以像素為單位）。 僅適用於影像轉譯。 |
| `tiff:ImageLength` | 轉譯長度（以像素為單位）。 僅適用於影像轉譯。 |

### 錯誤原因{#error-reasons}

| 原因 | 說明 |
|---------|-------------|
| `RenditionFormatUnsupported` | 指定的來源不支援請求的轉譯格式。 |
| `SourceUnsupported` | 即使支援類型，也不支援特定來源。 |
| `SourceCorrupt` | 源資料已損壞。 包含空白檔案。 |
| `RenditionTooLarge` | 無法使用`target`中提供的預先簽署URL上傳轉譯。 實際的轉譯大小可在`repo:size`中當做中繼資料使用，而且用戶端可以使用適當數目的預先簽署URL來重新處理此轉譯。 |
| `GenericError` | 其他任何非預期錯誤。 |
