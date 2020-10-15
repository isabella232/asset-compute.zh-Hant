---
title: '[!DNL Asset Compute Service] HTTP API。'
description: '[!DNL Asset Compute Service] 建立自訂應用程式的HTTP API。'
translation-type: tm+mt
source-git-commit: 18e97e544014933e9910a12bc40246daa445bf4f
workflow-type: tm+mt
source-wordcount: '2931'
ht-degree: 2%

---


# [!DNL Asset Compute Service] HTTP API {#asset-compute-http-api}

API的使用僅限於開發用途。 開發自訂應用程式時，API會以內容形式提供。 [!DNL Adobe Experience Manager] 因為雲端服務會使用API將處理資訊傳遞至自訂應用程式。 如需詳細資訊，請參 [閱使用資產微服務和處理設定檔](https://docs.adobe.com/content/help/zh-Hant/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

>[!NOTE]
>
>[!DNL Asset Compute Service] 只能與雲端服務 [!DNL Experience Manager] 搭配使用。

任何 [!DNL Asset Compute Service] HTTP API用戶端都必須遵循此高階流程：

1. 在IMS組織中，用戶 [!DNL Adobe Developer Console] 端被布建為專案。 每個獨立的客戶端（系統或環境）都需要自己的獨立項目，以分離事件資料流。

1. 客戶端使用 [JWT（服務帳戶）驗證為技術帳戶生成訪問令牌](https://www.adobe.io/authentication/auth-methods.html)。

1. 客戶端僅調 [`/register`](#register) 用一次以檢索日誌URL。

1. 用戶端會呼 [`/process`](#process-request) 叫每個要產生轉譯的資產。 呼叫是非同步的。

1. 客戶會定期輪詢日誌以 [接收事件](#asynchronous-events)。 當轉譯成功處理（事件類型）或發生錯誤（事件類型）時，`rendition_created` 它會接收每個請求轉譯的`rendition_failed` 事件。

@ [](https://github.com/adobe/asset-compute-client) adobe/asset-compute-client模組可讓您在Node.js程式碼中輕鬆使用API。

## 驗證與授權 {#authentication-and-authorization}

所有API都需要存取Token驗證。 這些請求必須設定下列標題：

1. `Authorization` 標題（即技術帳戶Token），是透過 [JWT交換從Adobe Developer Console專案接收](https://www.adobe.io/authentication/auth-methods.html) 。 范 [圍](#scopes) :

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

這要求 [!DNL Adobe Developer Console] 專案必須訂閱 `Asset Compute`、 `I/O Events`和服 `I/O Management API` 務。 個別範圍的劃分如下：

* 基本
   * scopes: `openid,AdobeID`

* 資產計算
   * metascope: `asset_compute_meta`
   * scopes: `asset_compute,read_organizations`

* Adobe I/O活動
   * metascope: `event_receiver_api`
   * scopes: `event_receiver,event_receiver_api`

* Adobe I/O管理API
   * metascope: `ent_adobeio_sdk`
   * scopes: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## 註冊 {#register}

每個客戶端( [!DNL Asset Compute service] 訂閱服務的 [!DNL Adobe Developer Console] 唯一項目)必須先註冊， [才能發出處理請求](#register-request) 。 註冊步驟返回唯一事件日誌，該日誌是從格式副本處理中檢索非同步事件所必需的。

在客戶端生命週期結束時，可以取消注 [冊](#unregister-request)。

### 註冊要求 {#register-request}

此API呼叫會設定用戶 [!DNL Asset Compute] 端並提供事件日誌URL。 這是一個冪等運算，每個用戶端只需呼叫一次。 可以再次調用它以檢索日記賬URL。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/register` |
| 頁首 `Authorization` | 所有與 [授權相關的標題](#authentication-and-authorization)。 |
| 頁首 `x-request-id` | 可選，可由客戶端為系統間處理請求的唯一端到端標識符進行設定。 |
| 請求正文 | 必須是空的。 |

### 註冊回應 {#register-response}

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME類型 | `application/json` |
| 頁首 `X-Request-Id` | 與請求標題相 `X-Request-Id` 同或產生唯一的標題。 用於識別跨系統的請求和／或支援請求。 |
| 響應體 | 包含、 `journal`和/ `ok` 或欄位的JSON `requestId` 物件。 |

HTTP狀態代碼為：

* **200次成功**:請求成功時。 它包含 `journal` URL，會收到任何透過觸發的非同步處理結果的通知( `/process` 成功時或失敗時 `rendition_created` 為事件類型 `rendition_failed` )。

   ```json
   {
       "ok": true,
       "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
       "requestId": "1234567890"
   }
   ```

* **401未授權**:當請求沒有有效驗證時 [發生](#authentication-and-authorization)。 範例可能是無效的存取Token或無效的API金鑰。

* **403禁止**:當請求沒有有效授權時 [發生](#authentication-and-authorization)。 範例可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）並未訂閱所有必要服務。

* **429請求太多**:當系統由此客戶端過載或其它情況下發生。 客戶端應以指數式回退 [方式重試](https://en.wikipedia.org/wiki/Exponential_backoff)。 屍體是空的。
* **4xx錯誤**:發生其他客戶端錯誤時，註冊失敗。 通常會傳回JSON回應，例如此回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**:發生於發生任何其他伺服器端錯誤且註冊失敗時。 通常會傳回JSON回應，例如此回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

### 取消註冊請求 {#unregister-request}

此API呼叫會取消註冊 [!DNL Asset Compute] 用戶端。 之後就無法再呼叫 `/process`。 使用未註冊用戶端或尚未註冊用戶端的API呼叫會傳回錯 `404` 誤。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/unregister` |
| 頁首 `Authorization` | 所有與 [授權相關的標題](#authentication-and-authorization)。 |
| 頁首 `x-request-id` | 可選，可由客戶端為系統間處理請求的唯一端到端標識符進行設定。 |
| 請求正文 | 空白. |

### 註銷響應 {#unregister-response}

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME類型 | `application/json` |
| 頁首 `X-Request-Id` | 與請求標題相 `X-Request-Id` 同或產生唯一的標題。 用於識別跨系統的請求和／或支援請求。 |
| 響應體 | 包含和欄位的 `ok` JSON `requestId` 物件。 |

狀態代碼為：

* **200次成功**:發生於找到並刪除註冊和日誌時。

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **401未授權**:當請求沒有有效驗證時 [發生](#authentication-and-authorization)。 範例可能是無效的存取Token或無效的API金鑰。

* **403禁止**:當請求沒有有效授權時 [發生](#authentication-and-authorization)。 範例可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）並未訂閱所有必要服務。

* **404找不到**:當給定憑據沒有當前註冊時發生。

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **429請求太多**:當系統過載時發生。 客戶端應以指數式回退 [方式重試](https://en.wikipedia.org/wiki/Exponential_backoff)。 屍體是空的。

* **4xx錯誤**:發生在發生任何其他客戶端錯誤並取消註冊失敗時。 通常會傳回JSON回應，例如此回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**:發生於發生任何其他伺服器端錯誤且註冊失敗時。 通常會傳回JSON回應，例如此回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

## 處理請求 {#process-request}

此操 `process` 作會根據請求中的說明，提交將來源資產轉換為多個轉譯的任務。 有關成功完成(事件類型 `rendition_created`)或任何錯誤(事件類型 `rendition_failed`)的通知會傳送至事件日誌，在發出任意數量的請求前，必須使用 [/註冊一次來檢索該日誌](#register)`/process` 。 錯誤格式的請求會立即失敗，並出現400錯誤碼。

使用URL（如Amazon AWS S3預簽名的URL或Azure Blob Storage SAS URL）來參考二進位檔案，以讀取資產( `source``GET` URL)和寫入轉譯(`PUT` URL)。 用戶端負責產生這些預先簽署的URL。

| 參數 | 值 |
|--------------------------|------------------------------------------------------|
| 方法 | `POST` |
| 路徑 | `/process` |
| MIME類型 | `application/json` |
| 頁首 `Authorization` | 所有與 [授權相關的標題](#authentication-and-authorization)。 |
| 頁首 `x-request-id` | 可選，可由客戶端為系統間處理請求的唯一端到端標識符進行設定。 |
| 請求正文 | 必須採用程式請求JSON格式，如下所述。 它提供處理哪些資產以及產生哪些轉譯的指示。 |

### 處理請求JSON {#process-request-json}

請求主體 `/process` 是具有此高階結構描述的JSON物件：

```json
{
    "source": "",
    "renditions" : []
}
```

可用欄位包括：

| 名稱 | 類型 | 說明 | 範例 |
|--------------|----------|-------------|---------|
| `source` | `string` | 要處理的來源資產URL。 根據要求的轉譯格式(例如， `fmt=zip`)。 | `"http://example.com/image.jpg"` |
| `source` | `object` | 說明要處理的來源資產。 請參閱下面的「 [源對象」欄位](#source-object-fields) 的說明。 根據要求的轉譯格式(例如， `fmt=zip`)。 | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | 從來源檔案產生的轉譯。 每個轉譯物件都支援 [轉譯指令](#rendition-instructions)。 必要. | `[{ "target": "https://....", "fmt": "png" }]` |

您 `source` 可以是視 `<string>` 為URL的URL，也可以是具 `<object>` 有其他欄位的。 下列變體類似：

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
| `name` | `string` | 來源資產檔案名稱。 如果未檢測到MIME類型，則可使用名稱中的檔案副檔名。 優先於URL路徑中的檔案名或二進位資 `content-disposition` 源標題中的檔案名。 預設為「檔案」。 | `"image.jpg"` |
| `size` | `number` | 來源資產檔案大小（以位元組為單位）。 優先於二進位 `content-length` 資源的標頭。 | `10234` |
| `mimetype` | `string` | 來源資產檔案MIME類型。 優先於二進位 `content-type` 資源的標頭。 | `"image/jpeg"` |

### 完整的請 `process` 求範例 {#complete-process-request-example}

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

## 流程響應 {#process-response}

請 `/process` 求會根據基本請求驗證立即傳回成功或失敗。 實際資產處理會非同步進行。

| 參數 | 值 |
|-----------------------|------------------------------------------------------|
| MIME類型 | `application/json` |
| 頁首 `X-Request-Id` | 與請求標題相 `X-Request-Id` 同或產生唯一的標題。 用於識別跨系統的請求和／或支援請求。 |
| 響應體 | 包含和欄位的 `ok` JSON `requestId` 物件。 |

狀態代碼：

* **200次成功**:如果請求已成功提交。 回應JSON包 `"ok": true`括：

   ```json
   {
       "ok": true,
       "requestId": "1234567890"
   }
   ```

* **400無效請求**:如果請求的格式不正確，例如請求JSON中遺失的必要欄位。 回應JSON包 `"ok": false`括：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **401未授權**:當請求沒有有效驗 [證](#authentication-and-authorization)。 範例可能是無效的存取Token或無效的API金鑰。
* **403禁止**:當請求沒有有效的授 [權](#authentication-and-authorization)。 範例可能是有效的存取Token，但Adobe Developer Console專案（技術帳戶）並未訂閱所有必要服務。
* **429請求太多**:當系統由此客戶端或一般情況過載時。 客戶端可以在指數型回退時 [重試](https://en.wikipedia.org/wiki/Exponential_backoff)。 屍體是空的。
* **4xx錯誤**:發生其他客戶機錯誤時。 通常會傳回JSON回應，例如此回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

* **5xx錯誤**:發生其他伺服器端錯誤時。 通常會傳回JSON回應，例如此回應，但並非所有錯誤都能保證：

   ```json
   {
       "ok": false,
       "requestId": "1234567890",
       "message": "error message"
   }
   ```

除了401或403等設定問題，或400等無效請求外，大部分的客戶都可能會傾向於 [重試與任何錯誤完全相](https://en.wikipedia.org/wiki/Exponential_backoff) 同的請求，但出現指數式退 *避* 。 除了透過429個回應的一般速率限制外，暫時服務中斷或限制可能會導致5xx錯誤。 然後建議在一段時間後重試。

所有JSON回應（如果有）都 `requestId` 包含與標題相同的 `X-Request-Id` 值。 建議從標題中讀取，因為它始終存在。 在與 `requestId` 處理請求相關的所有事件中，也會傳回該錯誤 `requestId`。 用戶端不得對此字串的格式作任何假設，它是不透明字串識別碼。

## 選擇加入後處理 {#opt-in-to-post-processing}

資 [產計算SDK](https://github.com/adobe/asset-compute-sdk) 支援一組基本影像後處理選項。 自訂工作者可將轉譯物件上的欄位設為，以明確 `postProcess` 選擇加入後處理 `true`。

支援的使用案例包括：

* 將轉譯裁切為限制由crop.w、crop.h、crop.x和crop.y定義的矩形。它由格式副本 `instructions.crop` 對象中定義。
* 使用寬度、高度或兩者調整影像大小。 它由格式副本對 `instructions.width` 像和 `instructions.height` 在格式副本對象中定義。 若要僅使用寬度或高度來調整大小，請只設定一個值。 計算服務控制長寬比。
* 設定JPEG影像的品質。 它由格式副本 `instructions.quality` 對象中定義。 最佳質量由值表示，而值 `100` 較小表示質量降低。
* 建立隔行掃描影像。 它由格式副本 `instructions.interlace` 對象中定義。
* 設定DPI，以調整套用至像素的比例，以調整轉譯大小以用於案頭出版。 它由轉譯物 `instructions.dpi` 件中定義，以變更dpi解析度。 不過，若要調整影像大小，使其在不同解析度下大小相同，請使用說 `convertToDpi` 明。
* 調整影像大小，使其轉譯寬度或高度在指定目標解析度(DPI)時仍與原始影像相同。 它由格式副本 `instructions.convertToDpi` 對象中定義。

## 浮水印資產 {#add-watermark}

「資 [產計算SDK](https://github.com/adobe/asset-compute-sdk) 」支援在PNG、JPEG、TIFF和GIF影像檔案中新增浮水印。 水印會依照轉譯中物件中的轉譯指 `watermark` 示來新增。

在轉譯後處理期間會執行浮水印。 若要浮水印資產，自訂工 [作者會將轉譯物件上的欄位設](#opt-in-to-post-processing) 定為 `postProcess` ，以 `true`選擇後處理方式。 如果工作者不選擇加入，則不會套用浮水印，即使在請求的轉譯物件上設定了浮水印物件。

## 轉譯指示 {#rendition-instructions}

這些是處理／處理中 `renditions` 陣列的 [可用選項](#process-request)。

### 常用欄位 {#common-fields}

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | 轉譯目標格式也可用於文 `text` 字擷取和XMP中 `xmp` 繼資料擷取為xml。 檢視支 [援的格式](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/file-format-support.html) | `png` |
| `worker` | `string` | 自訂應用程 [式的URL](develop-custom-application.md)。 必須是 `https://` URL。 如果此欄位存在，則由自訂應用程式建立轉譯。 然後，任何其他設定的轉譯欄位都會用於自訂應用程式。 | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | 使用HTTP PUT上傳產生轉譯的URL。 | `http://w.com/img.jpg` |
| `target` | `object` | 產生轉譯的多部分預先簽署的URL上傳資訊。 這適用於 [AEM/Oak Direct Binary Upload](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) with this [multipart upload behavior](http://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>欄位:<ul><li>`urls`:字串陣列，每個預先簽署的部件URL各一個</li><li>`minPartSize`:一個部件的最小大小= url</li><li>`maxPartSize`:用於一個部件的最大大小= url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | 由用戶端控制並依原樣傳遞至轉譯事件的選擇性保留空間。 可讓用戶端新增自訂資訊，以識別轉譯事件。 自訂應用程式中不得修改或依賴用戶端，因為用戶端隨時都可自由變更。 | `{ ... }` |

### 轉譯特定欄位 {#rendition-specific-fields}

如需目前支援的檔案格式清單，請參閱支 [援的檔案格式](https://docs.adobe.com/content/help/en/experience-manager-cloud-service/assets/file-format-support.html)。

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `*` | `*` | 可新增自訂應用程式瞭解的進階自 [訂欄位](develop-custom-application.md) 。 |  |
| `embedBinaryLimit` | `number` 位元組 | 如果已設定此值，且轉譯的檔案大小小於此值，則轉譯會內嵌在轉譯產生完成後傳送的事件中。 允許嵌入的最大大小為32 KB（32 x 1024位元組）。 如果轉譯的大小超過限 `embedBinaryLimit` 制，則會將其放在雲端儲存空間的某個位置，且不會嵌入事件。 | `3276` |
| `width` | `number` | 寬度（像素）。 僅限影像轉譯。 | `200` |
| `height` | `number` | 高度（像素）。 僅限影像轉譯。 | `200` |
|  |  | 如果： <ul> <li> 同時 `width` 指定 `height` 和，然後影像會配合尺寸，同時維持長寬比 </li><li> 只指 `width` 定或 `height` 僅指定，產生的影像會使用對應的尺寸，同時保留長寬比</li><li> 如果未指 `width` 定或 `height` 未指定，則會使用原始影像像素大小。 這取決於源類型。 對於某些格式（例如PDF檔案），會使用預設大小。 可以有最大大小限制。</li></ul> |  |
| `quality` | `number` | 在至範圍中指定jpeg `1` 品 `100`質。 僅適用於影像轉譯。 | `90` |
| `xmp` | `string` | 只有XMP中繼資料回寫使用，才會使用base64編碼的XMP來回寫回指定的轉譯。 |  |
| `interlace` | `bool` | 將隔行掃描的PNG或GIF或漸進式JPEG設為 `true`。 它對其他檔案格式沒有影響。 |  |
| `jpegSize` | `number` | JPEG檔案的大小（以位元組為單位）。 它會覆寫任何 `quality` 設定。 對其他格式沒有影響。 |  |
| `dpi` | `number` 或 `object` | 設定x和y DPI。 為簡單起見，它也可以設為單一數字，用於x和y。它對影像本身沒有影響。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` 或 `object` | x和y DPI會重新取樣值，同時維持實體大小。 為簡單起見，它也可以設為單一數字，用於x和y。 | `96` 或 `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | 要包含在ZIP檔案中的檔案清單(`fmt=zip`)。 每個項目可以是URL字串或具有下列欄位的物件：<ul><li>`url`:下載檔案的URL</li><li>`path`:將檔案儲存在ZIP中此路徑下</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | ZIP封存檔案的重複處`fmt=zip`理。 依預設，儲存在ZIP中相同路徑下的多個檔案會產生錯誤。 設 `duplicate` 定 `ignore` 為只產生要儲存的第一個資產，其餘的則忽略。 | `ignore` |
| `watermark` | `object` | 包含有關浮水印的 [說明](#watermark-specific-fields)。 |  |

### 水印特定欄位 {#watermark-specific-fields}

PNG格式會用作浮水印。

| 名稱 | 類型 | 說明 | 範例 |
|-------------------|----------|-------------|---------|
| `scale` | `number` | 水印的縮放，介於 `0.0` 和 `1.0`。 `1.0` 意指水印有其原始縮放比例(1:1)，而較低的值會降低水印大小。 | 值表示 `0.5` 原始大小的一半。 |
| `image` | `url` | 用於浮水印的PNG檔案的URL。 |  |

## 非同步事件 {#asynchronous-events}

轉譯處理完成或發生錯誤時，會傳送事件至 [Adobe I/O Events Journal](https://www.adobe.io/apis/experienceplatform/events/documentation.html#!adobedocs/adobeio-events/master/intro/journaling_api.md)。 客戶必須監聽透過 [/註冊提供的日](#register)志URL。 日誌響應包括 `event` 由每個事件的一個對象組成的陣列，該欄位 `event` 包括實際事件有效載荷。

Adobe I/O事件類型是所有事件的 [!DNL Asset Compute Service] 類型 `asset_compute`。 日記帳只會自動訂閱此事件類型，而且不需要再根據Adobe I/O事件類型進行篩選。 服務特定事件類型可在事件的屬 `type` 性中使用。

### Event types {#event-types}

| 事件 | 說明 |
|---------------------|-------------|
| `rendition_created` | 針對每個成功處理和上傳的轉譯傳送。 |
| `rendition_failed` | 針對每個無法處理或上傳的轉譯傳送。 |

### 事件屬性 {#event-attributes}

| 屬性 | 類型 | 事件 | 說明 |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | 事件以簡化的延伸 [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) (如JavaScript [Date.toISOString()定義)格式傳送的時間戳記](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString)。 |
| `requestId` | `string` | `*` | 原始請求的請求ID `/process`，與標題 `X-Request-Id` 相同。 |
| `source` | `object` | `*` | 請 `source` 求 `/process` 的。 |
| `userData` | `object` | `*` | 請求 `userData` 的轉譯(如果已設 `/process` 定)。 |
| `rendition` | `object` | `rendition_*` | 傳遞的對應轉譯物件 `/process`。 |
| `metadata` | `object` | `rendition_created` | 轉譯 [的中繼資](#metadata) 料屬性。 |
| `errorReason` | `string` | `rendition_failed` | 轉譯失敗 [原因](#error-reasons) （如果有）。 |
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

### 錯誤原因 {#error-reasons}

| 原因 | 說明 |
|---------|-------------|
| `RenditionFormatUnsupported` | 指定的來源不支援請求的轉譯格式。 |
| `SourceUnsupported` | 即使支援類型，也不支援特定來源。 |
| `SourceCorrupt` | 源資料已損壞。 包含空白檔案。 |
| `RenditionTooLarge` | 無法使用中提供的預先簽署URL上傳轉譯 `target`。 實際的轉譯大小可做為中繼資料 `repo:size` 使用，而且用戶端可以使用適當數目的預先簽署URL來重新處理此轉譯。 |
| `GenericError` | 其他任何非預期錯誤。 |
