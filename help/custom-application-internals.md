---
title: 瞭解自定義應用程式的工作
description: 內部工作 [!DNL Asset Compute Service] 自定義應用程式，幫助瞭解其工作原理。
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: 2af710443cdc2e5e25e105eca6e779eb58631ae9
workflow-type: tm+mt
source-wordcount: '751'
ht-degree: 0%

---

# 自定義應用程式的內部 {#how-custom-application-works}

使用下圖可瞭解當客戶端使用自定義應用程式處理數字資產時的端到端工作流。

![自定義應用程式工作流](assets/customworker.svg)

*圖：使用 [!DNL Asset Compute Service]。*

## 註冊 {#registration}

客戶端必須調用 [`/register`](api.md#register) 第一次請求之前 [`/process`](api.md#process-request) 以設定和檢索用於接收的日記帳URL [!DNL Adobe I/O] AdobeAsset compute事件。

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

的 [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) JavaScript庫可用於NodeJS應用程式，以處理從註冊、處理到非同步事件處理的所有必要步驟。 有關所需標題的詳細資訊，請參閱 [驗證和授權](api.md)。

## 處理 {#processing}

客戶端發送 [處理](api.md#process-request) 請求。

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

客戶端負責使用預簽名的URL正確格式化格式副本。 的 [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript庫可以在NodeJS應用程式中用於預簽名URL。 當前庫僅支援Azure Blob儲存和AWSS3容器。

處理請求返回 `requestId` 可用於輪詢 [!DNL Adobe I/O] 事件。

下面是自定義應用程式處理請求示例。

```json
{
    "source": "https://www.adobe.com/some-source-file.jpg",
    "renditions" : [
        {
            "worker": "https://my-project-namespace.adobeioruntime.net/api/v1/web/my-namespace-version/my-worker",
            "name": "rendition1.jpg",
            "target": "https://some-presigned-put-url-for-rendition1.jpg",
        }
    ],
    "userData": {
        "my-asset-id": "1234567890"
    }
}
```

的 [!DNL Asset Compute Service] 將自定義應用程式格式副本請求發送到自定義應用程式。 它將HTTPPOST用於提供的應用程式URL，該URL是來自App Builder的安全Web操作URL。 所有請求都使用HTTPS協定來最大化資料安全。

的 [asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) 自定義應用程式使用的HTTPPOST請求。 它還處理源的下載、上傳格式副本、發送 [!DNL Adobe I/O] 事件和錯誤處理。

<!-- TBD: Add the application diagram. -->

### 應用程式碼 {#application-code}

自定義代碼只需要提供使用本地可用源檔案的回調(`source.path`)。 的 `rendition.path` 是放置資產處理請求的最終結果的位置。 自定義應用程式使用回調將本地可用的源檔案轉換為格式副本檔案(`rendition.path`)。 自定義應用程式必須寫入 `rendition.path` 建立格式副本：

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

// worker() is the entry point in the SDK "framework".
// The asynchronous function defined is the rendition callback.
exports.main = worker(async (source, rendition) => {

    // Tip: custom worker parameters are available in rendition.instructions.
    console.log(rendition.instructions.name); // should print out `rendition.jpg`.

    // Simplest example: copy the source file to the rendition file destination so as to transfer the asset as is without processing.
    await fs.copyFile(source.path, rendition.path);
});
```

### 下載源檔案 {#download-source}

自定義應用程式只處理本地檔案。 下載源檔案由 [asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk)。

### 格式副本建立 {#rendition-creation}

SDK調用非同步 [Rendition回調函式](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 格式副本。

回調函式有權訪問 [源](https://github.com/adobe/asset-compute-sdk#source) 和 [格式](https://github.com/adobe/asset-compute-sdk#rendition) 對象。 的 `source.path` 已存在，並且是源檔案的本地副本的路徑。 的 `rendition.path` 是必須儲存已處理格式副本的路徑。 除非 [disableSourceDownload標誌](https://github.com/adobe/asset-compute-sdk#worker-options-optional) 設定，應用程式必須完全使用 `rendition.path`。 否則，SDK無法找到或標識格式副本檔案並失敗。

對示例進行過度簡化，以說明並重點關注定制應用程式的解剖結構。 應用程式只將源檔案複製到格式副本目標。

有關格式副本回調參數的詳細資訊，請參見 [asset computeSDK API](https://github.com/adobe/asset-compute-sdk#api-details)。

### 上載格式副本 {#upload-rendition}

在建立每個格式副本並將其儲存到具有由 `rendition.path`，也請參見Wiki頁。 [asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) 將每個格式副本上載到雲儲存(AWS或Azure)。 若且唯若傳入請求具有多個指向同一應用程式URL的格式副本時，自定義應用程式才會同時獲取多個格式副本。 上載到雲儲存的操作是在每個格式副本之後，並在為下一個格式副本運行回調之前完成的。

的 `batchWorker()` 具有不同的行為，因為它實際上處理所有格式副本，並且只有在所有格式副本都已處理後，才上載這些格式副本。

## [!DNL Adobe I/O] 事件 {#aio-events}

SDK發送 [!DNL Adobe I/O] 每個格式副本的事件。 這些事件或類型 `rendition_created` 或 `rendition_failed` 取決於結果。 請參閱 [asset compute非同步事件](api.md#asynchronous-events) 的子菜單。

## 接收 [!DNL Adobe I/O] 事件 {#receive-aio-events}

客戶端輪詢 [[!DNL Adobe I/O] 事件日記帳](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) 根據它的消費邏輯。 初始日記帳URL是在 `/register` API響應。 可以使用 `requestId` 在事件中存在，與返回的 `/process`。 每個格式副本都有一個單獨的事件，一旦上傳（或失敗）格式副本後即會被發送。 一旦收到匹配的事件，客戶端就可以顯示或處理生成的格式副本。

JavaScript庫 [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) 使用 `waitActivation()` 方法獲取所有事件。

```javascript
const events = await assetCompute.waitActivation(requestId);
await Promise.all(events.map(event => {
    if (event.type === "rendition_created") {
        // get rendition from cloud storage location
    }
    else if (event.type === "rendition_failed") {
        // failed to process
    }
    else {
        // other event types
        // (could be added in the future)
    }
}));
```

有關如何獲取日記帳事件的詳細資訊，請參閱 [[!DNL Adobe I/O] 事件API](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml)。

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
