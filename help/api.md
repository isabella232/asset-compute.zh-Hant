---
title: '[!DNL Asset Compute Service] HTTP API'
description: '[!DNL Asset Compute Service] 建立自訂應用程式的HTTP API。'
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: 780ddb7e119a28a1f8cc555ed2f1d3cee543b73f
workflow-type: tm+mt
source-wordcount: '2906'
ht-degree: 2%

---

# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

API的使用僅限於開發用途。 開發自訂應用程式時，API會以內容形式提供。 [!DNL Adobe Experience Manager] as a使 [!DNL Cloud Service] 用API將處理資訊傳遞至自訂應用程式。如需詳細資訊，請參閱[使用資產微服務和處理設定檔](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

>[!NOTE]
>
>[!DNL Asset Compute Service] 只能與as  [!DNL Experience Manager] a搭配使 [!DNL Cloud Service]用

[!DNL Asset Compute Service] HTTP API的任何用戶端都必須遵循此高階流程：

1. 在IMS組織中，用戶端已布建為[!DNL Adobe Developer Console]專案。 每個獨立的客戶端（系統或環境）都需要各自的獨立項目，以便分隔事件資料流。

1. 客戶端使用[JWT（服務帳戶）Authentication](https://www.adobe.io/authentication/auth-methods.html)為技術帳戶生成訪問令牌。

1. 客戶端僅調用一次[`/register`](#register)以檢索日誌URL。

1. 用戶端會針對每個要產生轉譯的資產呼叫[`/process`](#process-request)。 呼叫為非同步。

1. 客戶端定期輪詢日誌以[接收事件](#asynchronous-events)。 成功處理轉譯時（`rendition_created`事件類型）或發生錯誤時（`rendition_failed`事件類型），它會接收每個請求轉譯的事件。

[@adobe/asset-compute-client](https://github.com/adobe/asset-compute-client)模組可讓您在Node.js程式碼中輕鬆使用API。

## 驗證和授權 {#authentication-and-authorization}

所有API都需要存取權杖驗證。 請求必須設定下列標題：

1. `Authorization` 具有記號代號的標題，即技術帳戶代號，會透過Adobe開發 [人員](https://www.adobe.io/authentication/auth-methods.html) 控制台專案的JWT exchange接收。下面記錄了[範圍](#scopes)。

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` 標題為IMS組織ID。

1. `x-api-key` 使用專案的用戶端 [!DNL Adobe Developers Console] ID。

### 範圍 {#scopes}

請確定存取權杖的下列範圍：

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

這些要求[!DNL Adobe Developer Console]專案必須訂閱`Asset Compute`、`I/O Events`和`I/O Management API`服務。 個別範圍的劃分如下：

* 基本
   * 範圍：`openid,AdobeID`

* asset compute
   * metascope:`asset_compute_meta`
   * 範圍：`asset_compute,read_organizations`

* [!DNL Adobe I/O] 事件
   * metascope:`event_receiver_api`
   * 範圍：`event_receiver,event_receiver_api`

* [!DNL Adobe I/O] 管理API
   * metascope:`ent_adobeio_sdk`
   * 範圍：`adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 註冊 {#register}

[!DNL Asset Compute service] — 訂閱服務的唯一[!DNL Adobe Developer Console]專案的每個用戶端 — 必須先[註冊](#register-request)，才能提出處理請求。 註冊步驟返回唯一事件日記帳，該日記帳是從格式副本處理中檢索非同步事件所必需的。

在其生命週期結束時，客戶端可以[取消註冊](#unregister-request)。

### 註冊請求 {#register-request}

此API呼叫會設定[!DNL Asset Compute]用戶端並提供事件日誌URL。 這是等冪操作，每個用戶端只需呼叫一次。 可以再次調用它以檢索日誌URL。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/register` |
| 頁首 `Authorization` | 所有[授權相關標頭](#authentication-and-authorization)。 |
| 頁首 `x-request-id` | 可選，可由用戶端為系統間處理請求的唯一端對端識別碼進行設定。 |
| 要求內文 | 必須為空。 |

### 註冊響應 {#register-response}

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME類型 | `application/json` |
| 頁首 `X-Request-Id` | 與`X-Request-Id`請求標題相同，或是唯一產生的標題。 用於識別跨系統的請求和/或支援請求。 |
| 回應內文 | 具有`journal`、`ok`及/或`requestId`欄位的JSON物件。 |

HTTP狀態代碼為：

* **200次成功**:請求成功時。它包含`journal` URL，系統會針對透過`/process`觸發的非同步處理的任何結果（如果成功，則為事件類型`rendition_created`，若失敗，則為`rendition_failed`）提供通知。

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401未授權**:當要求沒有有效驗證時 [發生](#authentication-and-authorization)。例如，存取權杖或API金鑰無效。

* **403禁止**:當請求沒有有效授權時 [發生](#authentication-and-authorization)。例如可能是有效的存取權杖，但Adobe開發人員控制台專案（技術帳戶）並未訂閱所有必要服務。

* **429請求太多**:當此客戶端或其他方式使系統過載時發生。用戶端應以[指數回退](https://en.wikipedia.org/wiki/Exponential_backoff)重試。 屍體是空的。
* **4xx錯誤**:發生任何其他客戶端錯誤時，註冊失敗。通常會傳回這類JSON回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**:發生在有任何其他伺服器端錯誤且註冊失敗時。通常會傳回這類JSON回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### 取消註冊請求 {#unregister-request}

此API呼叫會取消註冊[!DNL Asset Compute]用戶端。 之後，將無法再呼叫`/process`。 對未註冊的客戶端或尚未註冊的客戶端使用API調用將返回`404`錯誤。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/unregister` |
| 頁首 `Authorization` | 所有[授權相關標頭](#authentication-and-authorization)。 |
| 頁首 `x-request-id` | 可選，可由用戶端為系統間處理請求的唯一端對端識別碼進行設定。 |
| 要求內文 | 空白. |

### 註銷響應 {#unregister-response}

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME類型 | `application/json` |
| 頁首 `X-Request-Id` | 與`X-Request-Id`請求標題相同，或是唯一產生的標題。 用於識別跨系統的請求和/或支援請求。 |
| 回應內文 | 具有`ok`和`requestId`欄位的JSON物件。 |

狀態代碼為：

* **200次成功**:在找到並刪除註冊和日誌時發生。

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401未授權**:當要求沒有有效驗證時 [發生](#authentication-and-authorization)。例如，存取權杖或API金鑰無效。

* **403禁止**:當請求沒有有效授權時 [發生](#authentication-and-authorization)。例如可能是有效的存取權杖，但Adobe開發人員控制台專案（技術帳戶）並未訂閱所有必要服務。

* **404未找到**:當給定憑據沒有當前註冊時發生。

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429請求太多**:在系統過載時發生。用戶端應以[指數回退](https://en.wikipedia.org/wiki/Exponential_backoff)重試。 屍體是空的。

* **4xx錯誤**:發生任何其他客戶端錯誤並註銷失敗時。通常會傳回這類JSON回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**:發生在有任何其他伺服器端錯誤且註冊失敗時。通常會傳回這類JSON回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## 處理請求 {#process-request}

`process`操作會根據請求中的指示提交將源資產轉換為多個格式副本的作業。 有關成功完成（事件類型`rendition_created`）或任何錯誤（事件類型`rendition_failed`）的通知將發送到事件日誌，在發出任何數量的`/process`請求之前，必須使用[/register](#register)檢索該日誌一次。 錯誤格式的請求會立即失敗，並顯示400錯誤代碼。

使用URL來參考二進位檔，例如Amazon AWS S3預先簽名的URL或Azure Blob儲存SAS URL，以讀取`source`資產(`GET` URL)和寫入轉譯(`PUT` URL)。 用戶端負責產生這些預先簽署的URL。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/process` |
| MIME類型 | `application/json` |
| 頁首 `Authorization` | 所有[授權相關標頭](#authentication-and-authorization)。 |
| 頁首 `x-request-id` | 可選，可由用戶端為系統間處理請求的唯一端對端識別碼進行設定。 |
| 要求內文 | 必須為程式要求JSON格式，如下所述。 它提供處理哪些資產以及要產生哪些轉譯的指示。 |

### 程式要求JSON {#process-request-json}

`/process`的要求內文是具有此高階結構的JSON物件：

```json
{
    "source": "",
    "renditions" : []
}
```

可用欄位包括：

| 名稱 | 類型 | 說明 | 範例 |
|--------------|----------|-------------|---------|
| `source` | `string` | 要處理的來源資產URL。 根據要求的轉譯格式選用(例如`fmt=zip`)。 | `"http://example.com/image.jpg"` |
| `source` | `object` | 說明要處理的來源資產。 請參閱下面[源對象欄位](#source-object-fields)的說明。 根據要求的轉譯格式選用(例如`fmt=zip`)。 | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | 要從源檔案生成的格式副本。 每個轉譯對象支援[轉譯指令](#rendition-instructions)。 必要. | `[{ "target": "https://....", "fmt": "png" }]` |

`source`可以是被視為URL的`<string>`，也可以是包含其他欄位的`<object>`。 下列變體類似：

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
| `url` | `string` | 要處理的來源資產URL。 必要. | `"http://example.com/image.jpg"` |
| `name` | `string` | 來源資產檔案名稱。 如果未偵測到MIME類型，則名稱中的副檔名可能會使用。 在二進位資源的`content-disposition`標題中，優先於URL路徑中的檔案名或檔案名。 預設為「檔案」。 | `"image.jpg"` |
| `size` | `number` | 來源資產檔案大小（以位元組為單位）。 優先於二進位資源的`content-length`標頭。 | `10234` |
| `mimetype` | `string` | 來源資產檔案MIME類型。 優先於二進位資源的`content-type`標頭。 | `"image/jpeg"` |

### 完整的`process`要求範例 {#complete-process-request-example}

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

根據基本請求驗證，`/process`請求會立即傳回成功或失敗。 實際的資產處理會以非同步方式進行。

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME類型 | `application/json` |
| 頁首 `X-Request-Id` | 與`X-Request-Id`請求標題相同，或是唯一產生的標題。 用於識別跨系統的請求和/或支援請求。 |
| 回應內文 | 具有`ok`和`requestId`欄位的JSON物件。 |

狀態代碼：

* **200次成功**:如果已成功提交請求。回應JSON包含`"ok": true`:

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400無效請求**:如果要求格式不正確，例如要求JSON中缺少必要欄位。回應JSON包含`"ok": false`:

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401未授權**:當要求沒有有效的驗 [證](#authentication-and-authorization)。例如，存取權杖或API金鑰無效。
* **403禁止**:當請求沒有有效的授 [權](#authentication-and-authorization)。例如可能是有效的存取權杖，但Adobe開發人員控制台專案（技術帳戶）並未訂閱所有必要服務。
* **429請求太多**:當此客戶端或一般情況下系統超載時。客戶端可以使用[指數回退](https://en.wikipedia.org/wiki/Exponential_backoff)重試。 屍體是空的。
* **4xx錯誤**:發生任何其他客戶端錯誤時。通常會傳回這類JSON回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**:發生任何其他伺服器端錯誤時。通常會傳回這類JSON回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

大多數客戶端可能傾向於對除&#x200B;*配置問題（如401或403）或無效請求（如400）之外的任何錯誤*，使用[指數回退](https://en.wikipedia.org/wiki/Exponential_backoff)重試完全相同的請求。 除了透過429個回應來限制正常速率外，暫時服務中斷或限制可能會導致5xx錯誤。 之後建議您在一段時間後重試。

所有JSON回應（如果存在）都包含`requestId`，其值與`X-Request-Id`標題相同。 建議從標題中讀取，因為它一律存在。 在與處理請求相關的所有事件中，也會將`requestId`傳回為`requestId`。 用戶端不得對此字串的格式作出任何假設，該字串是不透明的字串標識符。

## 選擇加入後續處理 {#opt-in-to-post-processing}

[Asset computeSDK](https://github.com/adobe/asset-compute-sdk)支援一組基本影像後處理選項。 將轉譯物件上的欄位`postProcess`設為`true`，自訂背景工作即可明確選擇加入後續處理。

支援的使用案例包括：

* 將轉譯裁切為由crop.w、crop.h、crop.x和crop.y定義限制的矩形。它由`instructions.crop`在轉譯物件中定義。
* 使用寬度、高度或兩者調整影像大小。 它由`instructions.width`和`instructions.height`在轉譯物件中定義。 要僅使用寬度或高度調整大小，請僅設定一個值。 計算服務會控制外觀比例。
* 設定JPEG影像的質量。 它由`instructions.quality`在轉譯物件中定義。 最佳質量由`100`表示，較小的值表示質量降低。
* 建立隔行掃描影像。 它由`instructions.interlace`在轉譯物件中定義。
* 設定DPI以調整套用至像素的比例，以調整用於案頭發佈用途的轉譯大小。 它由`instructions.dpi`在轉譯物件中定義，以變更dpi解析度。 但是，要調整影像大小，使其以不同解析度呈現相同大小，請使用`convertToDpi`指示。
* 調整影像大小，使其渲染的寬度或高度在指定目標解析度(DPI)下保持與原始影像相同。 它由`instructions.convertToDpi`在轉譯物件中定義。

## 浮水印資產 {#add-watermark}

[Asset computeSDK](https://github.com/adobe/asset-compute-sdk)支援為PNG、JPEG、TIFF和GIF影像檔案添加水印。 會根據轉譯的`watermark`物件中的轉譯指示來新增浮水印。

在轉譯後處理期間會執行浮水印。 若要為資產加上浮水印，自訂背景工作[會將轉譯物件上的欄位`postProcess`設為`true`，以選擇進行後續處理](#opt-in-to-post-processing)。 如果工作人員不選擇加入，則不會應用水印，即使請求中的格式副本對象上設定了水印對象也是如此。

## 轉譯指示 {#rendition-instructions}

這些是[/process](#process-request)中`renditions`陣列的可用選項。

### 公用欄位 {#common-fields}

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | 轉譯目標格式也可以是`text`用於文字擷取，以及`xmp`用於以xml擷取XMP中繼資料。 請參閱[支援的格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | [自訂應用程式](develop-custom-application.md)的URL。 必須是`https://` URL。 如果此欄位存在，則由自訂應用程式建立轉譯。 然後，在自訂應用程式中使用任何其他設定的轉譯欄位。 | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | 應使用HTTPPUT上傳產生轉譯的URL。 | `http://w.com/img.jpg` |
| `target` | `object` | 產生轉譯的多部分預先簽署的URL上傳資訊。 這適用於[AEM/Oak直接二進位上傳](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html)，具有此[多部分上傳行為](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html)。<br>欄位:<ul><li>`urls`:字串陣列，每個預簽名的部件URL各一個</li><li>`minPartSize`:用於一部分的最小大小= url</li><li>`maxPartSize`:用於一部分的最大大小= url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | 由用戶端控制並依原樣傳遞至轉譯事件的選用保留空間。 允許用戶端新增自訂資訊以識別轉譯事件。 不得在自定義應用程式中修改或依賴客戶端，因為客戶端隨時可以自由更改。 | `{ ... }` |

### 轉譯特定欄位 {#rendition-specific-fields}

有關當前支援的檔案格式的清單，請參見[支援的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)。

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `*` | `*` | 可新增[自訂應用程式](develop-custom-application.md)可理解的進階自訂欄位。 |  |
| `embedBinaryLimit` | `number` 位元組 | 如果已設定此值，且轉譯的檔案大小小於此值，則轉譯會嵌入在轉譯產生完成後所傳送的事件中。 內嵌允許的最大大小為32 KB（32 x 1024位元組）。 如果轉譯的大小大於`embedBinaryLimit`限制，則會將其放在雲端儲存空間中的某個位置，且不會嵌入事件中。 | `3276` |
| `width` | `number` | 寬度（像素）。 僅用於影像轉譯。 | `200` |
| `height` | `number` | 高度（像素）。 僅用於影像轉譯。 | `200` |
|  |  | 在下列情況下，一律會維持外觀比例： <ul> <li> 指定`width`和`height`，然後影像適合大小，同時保持長寬比 </li><li> 僅指定`width`或僅指定`height`，所生成的影像將使用相應的維，同時保持長寬比</li><li> 如果未指定`width`或`height`，則使用原始影像像素大小。 取決於來源類型。 對於某些格式（如PDF檔案），會使用預設大小。 可以有最大大小限制。</li></ul> |  |
| `quality` | `number` | 在`1`到`100`的範圍內指定jpeg質量。 僅適用於影像轉譯。 | `90` |
| `xmp` | `string` | 僅供XMP中繼資料回寫使用，以base64編碼的XMP回寫至指定的轉譯。 |  |
| `interlace` | `bool` | 將隔行掃描PNG或GIF或逐行JPEG設定為`true`，以建立隔行掃描PNG或GIF或逐行JPEG。 對其他檔案格式沒有影響。 |  |
| `jpegSize` | `number` | JPEG檔案的大小約（以位元組為單位）。 它會覆寫任何`quality`設定。 對其他格式沒有影響。 |  |
| `dpi` | `number` 或 `object` | 設定x和y DPI。 為了簡單起見，也可將其設為單一數字，用於x和y。它對影像本身沒有影響。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` 或 `object` | x和y DPI會重新取樣值，同時維持物理大小。 為了簡單起見，也可將其設為單一數字，用於x和y。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | 要包含在ZIP存檔中的檔案清單(`fmt=zip`)。 每個項目都可以是URL字串或具有下列欄位的物件：<ul><li>`url`:下載檔案的URL</li><li>`path`:將檔案儲存在ZIP中的此路徑下</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | ZIP封存檔的重複處理(`fmt=zip`)。 依預設，儲存在ZIP中相同路徑下的多個檔案會產生錯誤。 將`duplicate`設為`ignore`只會產生要儲存的第一個資產，並忽略其餘資產。 | `ignore` |
| `watermark` | `object` | 包含有關[watermark](#watermark-specific-fields)的說明。 |  |

### 水印特定欄位 {#watermark-specific-fields}

PNG格式會作為浮水印使用。

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 在`0.0`和`1.0`之間的水印的縮放。 `1.0` 意指水印具有其原始縮放(1:1)，且較低的值會減小水印的大小。 | 值`0.5`表示原始大小的一半。 |
| `image` | `url` | 用於浮水印的PNG檔案的URL。 |  |

## 非同步事件 {#asynchronous-events}

完成格式副本的處理或發生錯誤後，事件將發送到[[!DNL Adobe I/O] Events Journal](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md)。 用戶端必須監聽透過[/register](#register)提供的日誌URL。 日誌響應包括`event`陣列，該陣列由每個事件的一個對象組成，其中`event`欄位包含實際事件有效負載。

[!DNL Asset Compute Service]的所有事件的[!DNL Adobe I/O]事件類型為`asset_compute`。 日記帳僅自動訂閱此事件類型，並且不再需要根據[!DNL Adobe I/O]事件類型進行篩選。 在事件的`type`屬性中可使用服務特定事件類型。

### 事件類型 {#event-types}

| 事件 | 說明 |
|---------------------|-------------|
| `rendition_created` | 為每個成功處理和上傳的轉譯傳送。 |
| `rendition_failed` | 針對無法處理或上傳的每個轉譯而傳送。 |

### 事件屬性 {#event-attributes}

| 屬性 | 類型 | 事件 | 說明 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | 以簡化的延伸[ISO-8601](https://en.wikipedia.org/wiki/ISO_8601)格式傳送事件的時間戳記，如JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString)所定義。 |
| `requestId` | `string` | `*` | 原始請求對`/process`的請求ID，與`X-Request-Id`標頭相同。 |
| `source` | `object` | `*` | `/process`請求的`source`。 |
| `userData` | `object` | `*` | 若已設定，來自`/process`請求的轉譯的`userData`。 |
| `rendition` | `object` | `rendition_*` | 傳入`/process`的對應轉譯物件。 |
| `metadata` | `object` | `rendition_created` | 轉譯的[metadata](#metadata)屬性。 |
| `errorReason` | `string` | `rendition_failed` | 格式副本失敗[reason](#error-reasons)（如果有）。 |
| `errorMessage` | `string` | `rendition_failed` | 提供更多轉譯失敗詳細資訊（如果有）的文字。 |

### 中繼資料 {#metadata}

| 屬性 | 說明 |
|--------|-------------|
| `repo:size` | 格式副本的大小（以位元組為單位）。 |
| `repo:sha1` | 轉譯的sha1摘要。 |
| `dc:format` | 轉譯的MIME類型。 |
| `repo:encoding` | 轉譯的charset編碼（若為文字格式）。 |
| `tiff:ImageWidth` | 轉譯的寬度（像素）。 僅用於影像轉譯。 |
| `tiff:ImageLength` | 轉譯的長度（像素）。 僅用於影像轉譯。 |

### 錯誤原因 {#error-reasons}

| 原因 | 說明 |
|---------|-------------|
| `RenditionFormatUnsupported` | 給定源不支援請求的格式副本格式。 |
| `SourceUnsupported` | 即使支援類型，也不支援特定源。 |
| `SourceCorrupt` | 源資料已損壞。 包含空檔案。 |
| `RenditionTooLarge` | 無法使用`target`中提供的預簽名URL上載格式副本。 實際的轉譯大小在`repo:size`中以元資料的形式提供，並且可供用戶端使用適當數量的預簽名URL來重新處理此轉譯。 |
| `GenericError` | 任何其他意外錯誤。 |
