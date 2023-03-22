---
title: 了解自訂應用程式的運作方式
description: 內部工作 [!DNL Asset Compute Service] 自訂應用程式，協助了解運作方式。
exl-id: a3ee6549-9411-4839-9eff-62947d8f0e42
source-git-commit: 2af710443cdc2e5e25e105eca6e779eb58631ae9
workflow-type: tm+mt
source-wordcount: '751'
ht-degree: 0%

---

# 自訂應用程式的內部 {#how-custom-application-works}

請使用下圖來了解當用戶端使用自訂應用程式處理數位資產時的端對端工作流程。

![自訂應用程式工作流程](assets/customworker.svg)

*圖：使用 [!DNL Asset Compute Service].*

## 註冊 {#registration}

用戶端必須呼叫 [`/register`](api.md#register) 一次 [`/process`](api.md#process-request) 以便設定和檢索用於接收的日記帳URL [!DNL Adobe I/O] AdobeAsset compute的事件。

```sh
curl -X POST \
  https://asset-compute.adobe.io/register \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY"
```

此 [`@adobe/asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) JavaScript程式庫可用於NodeJS應用程式，以處理從註冊、處理到非同步事件處理等所有必要步驟。 如需所需標題的詳細資訊，請參閱 [驗證和授權](api.md).

## 處理 {#processing}

用戶端傳送 [處理](api.md#process-request) 請求。

```sh
curl -X POST \
  https://asset-compute.adobe.io/process \
  -H "x-ims-org-id: $ORG_ID" \
  -H "x-gw-ims-org-id: $ORG_ID" \
  -H "Authorization: Bearer $JWT_TOKEN" \
  -H "x-api-key: $API_KEY" \
  -d "<RENDITION_JSON>
```

用戶端負責使用預先簽署的URL正確格式化轉譯。 此 [`@adobe/node-cloud-blobstore-wrapper`](https://github.com/adobe/node-cloud-blobstore-wrapper#presigned-urls) JavaScript程式庫可用於NodeJS應用程式中，以預先簽署URL。 目前，程式庫僅支援Azure Blob儲存和AWS S3容器。

處理請求會傳回 `requestId` 可用於輪詢 [!DNL Adobe I/O] 事件。

以下是自訂應用程式處理請求的範例。

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

此 [!DNL Asset Compute Service] 將自訂應用程式轉譯請求傳送至自訂應用程式。 它會對提供的應用程式URL使用HTTPPOST，即來自App Builder的安全Web動作URL。 所有要求都使用HTTPS通訊協定，以最大化資料安全性。

此 [asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) 由自訂應用程式使用，可處理HTTPPOST要求。 它也可處理來源的下載、上傳轉譯、傳送 [!DNL Adobe I/O] 事件和錯誤處理。

<!-- TBD: Add the application diagram. -->

### 應用程式代碼 {#application-code}

自訂程式碼只需提供回呼，並取用本機可用的原始碼檔案(`source.path`)。 此 `rendition.path` 是資產處理請求最終結果的放置位置。 自訂應用程式使用回呼，以使用傳入的名稱(`rendition.path`)。 自定義應用程式必須寫入 `rendition.path` 若要建立轉譯：

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

自訂應用程式只處理本機檔案。 下載源檔案由 [asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk).

### 轉譯建立 {#rendition-creation}

SDK會呼叫非同步 [轉譯回撥函式](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 對應每個轉譯。

回呼函式可存取 [來源](https://github.com/adobe/asset-compute-sdk#source) 和 [轉譯](https://github.com/adobe/asset-compute-sdk#rendition) 對象。 此 `source.path` 已存在，是源檔案的本地副本的路徑。 此 `rendition.path` 是必須儲存已處理轉譯的路徑。 除非 [disableSourceDownload標幟](https://github.com/adobe/asset-compute-sdk#worker-options-optional) 已設定，應用程式必須完全使用 `rendition.path`. 否則，SDK無法找到或識別轉譯檔案，且會失敗。

本示例的過度簡化旨在說明並重點介紹自定義應用程式的結構。 應用程式只會將源檔案複製到格式副本目的地。

如需轉譯回呼參數的詳細資訊，請參閱 [asset computeSDK API](https://github.com/adobe/asset-compute-sdk#api-details).

### 上傳轉譯 {#upload-rendition}

建立每個轉譯並將其儲存在檔案中，且其路徑由提供 `rendition.path`, [asset computeSDK](https://github.com/adobe/asset-compute-sdk#adobe-asset-compute-worker-sdk) 將每個轉譯上傳至雲端儲存空間(AWS或Azure)。 如果且唯有當傳入的請求有多個轉譯指向相同應用程式URL時，自訂應用程式才會同時取得多個轉譯。 上傳至雲端儲存空間會在每次轉譯後，以及在下次轉譯時執行回呼前完成。

此 `batchWorker()` 有不同的行為，因為它實際上會處理所有轉譯，而且只有在所有轉譯都已處理完畢後才會上傳這些轉譯。

## [!DNL Adobe I/O] 事件 {#aio-events}

SDK會傳送 [!DNL Adobe I/O] 每個轉譯的事件。 這些事件為 `rendition_created` 或 `rendition_failed` 取決於結果。 請參閱 [asset compute非同步事件](api.md#asynchronous-events) 以取得事件詳細資訊。

## 接收 [!DNL Adobe I/O] 事件 {#receive-aio-events}

客戶端輪詢 [[!DNL Adobe I/O] 事件日誌](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#/Journaling) 根據其消費邏輯。 初始日記帳URL是 `/register` API回應。 可使用 `requestId` 存在於事件中，且與中傳回的相同 `/process`. 每個轉譯都有個別的事件，會在轉譯上傳（或失敗）後立即傳送。 一旦收到相符的事件，用戶端就可以顯示或處理產生的轉譯。

JavaScript程式庫 [`asset-compute-client`](https://github.com/adobe/asset-compute-client#usage) 使用 `waitActivation()` 方法來取得所有事件。

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

有關如何獲取日誌事件的詳細資訊，請參閱 [[!DNL Adobe I/O] 事件API](https://www.adobe.io/apis/experienceplatform/events/ioeventsapi.html#!adobedocs/adobeio-events/master/events-api-reference.yaml).

<!-- TBD:
* Illustration of the controls/data flow.
* Basic overview, in text and not code, of how an application works.
-->
