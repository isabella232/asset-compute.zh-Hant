---
title: 開發對象 [!DNL Asset Compute Service]
description: 使用 [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '1618'
ht-degree: 0%

---

# 開發自訂應用程式 {#develop}

開始開發自訂應用程式之前：

* 確保 [必要條件](/help/understand-extensibility.md#prerequisites-and-provisioning) 的URL。
* 安裝 [所需軟體工具](/help/setup-environment.md#create-dev-environment).
* 請參閱 [設定環境](setup-environment.md) 以確定您已準備好建立自訂應用程式。

## 建立自訂應用程式 {#create-custom-application}

請務必將 [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) 本機安裝。

1. 若要建立自訂應用程式， [建立App Builder專案](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli). 要執行此操作，請執行 `aio app init <app-name>` 在您的終端機中。

   如果您尚未登入，此命令會提示您登入 [Adobe Developer Console](https://console.adobe.io/) 和你的Adobe ID。 請參閱 [此處](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli) 有關從cli登錄的詳細資訊。

   Adobe建議您登入。 如果您有問題，請依照指示 [若要建立應用程式而不登入](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. 登錄後，按照CLI中的提示操作並選擇 `Organization`, `Project`，和 `Workspace` 以用於應用程式。 選擇您在 [設定環境](setup-environment.md). 提示時 `Which extension point(s) do you wish to implement ?`，請務必選取 `DX Asset Compute Worker`:

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyAdobe Developer App BuilderProject
   ? Which extension point(s) do you wish to implement ? (Press <space> to select, <a>
   to toggle all, <i> to invert selection)
   ❯◯ DX Experience Cloud SPA
   ◯ DX Asset Compute Worker
   ```

1. 提示時為 `Which Adobe I/O App features do you want to enable for this project?`，選取 `Actions`. 請務必取消選取 `Web Assets` 選項，作為網頁資產使用不同的驗證和授權檢查。

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. 提示時 `Which type of sample actions do you want to create?`，請務必選取 `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 按照其餘提示操作，在Visual Studio代碼（或您最喜愛的代碼編輯器）中開啟新應用程式。 其中包含自訂應用程式的架構和范常式式碼。

   請在這裡閱讀 [App Builder應用程式的主要元件](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application).

   範本應用程式會運用我們的 [asset computeSDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) 上傳、下載和協調應用程式轉譯，因此開發人員只需要實作自訂應用程式邏輯。 內 `actions/<worker-name>` 資料夾， `index.js` 檔案是新增自訂應用程式程式碼的位置。

請參閱 [範例自訂應用程式](#try-sample) 以取得自訂應用程式的範例和想法。

### 添加憑據 {#add-credentials}

當您在建立應用程式時登入時，ENV檔案中會收集大部分的App Builder憑證。 不過，使用開發人員工具需要其他憑證。

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 開發人員工具儲存憑證 {#developer-tool-credentials}

用於測試自訂應用程式的開發人員工具，實際為 [!DNL Asset Compute service] 需要雲端儲存容器，用於托管測試檔案，以及接收和顯示應用程式產生的轉譯。

>[!NOTE]
>
>這與 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]. 僅適用於使用Asset compute開發人員工具進行開發和測試。

請務必存取 [支援的雲端儲存容器](https://github.com/adobe/asset-compute-devtool#prerequisites). 此容器可視需要由多個開發人員跨不同專案共用。

#### 將憑據添加到ENV檔案 {#add-credentials-env-file}

將開發人員工具的下列認證新增至App Builder專案根目錄的ENV檔案：

1. 將服務新增至您的App Builder專案時建立的私密金鑰檔案，新增絕對路徑：

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. 從Adobe Developer主控台下載檔案。 前往專案的根目錄，然後按一下右上角的「全部下載」。 檔案下載時使用 `<namespace>-<workspace>.json` 作為檔案名。 執行下列任一項作業：

   * 將檔案重新命名為 `console.json` 並移到項目的根中。
   * 您可以選擇將絕對路徑新增至Adobe Developer Console整合JSON檔案。 這是一樣的 [`console.json`](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) 下載的檔案。

      ```conf
      ASSET_COMPUTE_INTEGRATION_FILE_PATH=
      ```

1. 新增S3或Azure儲存憑證。 您只需要存取一個雲端儲存解決方案。

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

>[!TIP]
>
>此 `config.json` 檔案包含憑據。 從專案內，將JSON檔案新增至 `.gitignore` 檔案來防止共用。 .env和.aio檔案也是如此。

## 執行應用程式 {#run-custom-application}

使用Asset compute開發人員工具執行應用程式之前，請正確設定 [憑據](#developer-tool-credentials).

若要在開發人員工具中執行應用程式，請使用 `aio app run` 命令。 它會將動作部署至 [!DNL Adobe I/O] 在本地電腦上運行並啟動開發工具。 此工具用於在開發期間測試應用程式請求。 以下是範例轉譯請求：

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>請勿使用 `--local` 標幟 `run` 命令。 不適用 [!DNL Asset Compute] 自訂應用程式和Asset compute開發人員工具。 自訂應用程式由 [!DNL Asset Compute Service] 無法存取開發人員本機電腦上執行的動作。

請參閱 [此處](test-custom-application.md) 如何測試應用程式並除錯。 完成自定義應用程式的開發後， [部署自定義應用程式](deploy-custom-application.md).

## 嘗試由Adobe提供的示例應用程式 {#try-sample}

以下是範例自訂應用程式：

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [工人 — 動物圖片](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### 範本自訂應用程式 {#template-custom-application}

此 [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) 是範本應用程式。 它只需複製源檔案即可生成格式副本。 此應用程式的內容是選擇 `Adobe Asset Compute` 在aio應用程式的建立中。

應用程式檔案， [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 使用 [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) 若要下載來源檔案，請協調每個轉譯處理，並將產生的轉譯上傳回雲端儲存空間。

此 [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 定義於應用程式程式碼內，是執行所有應用程式處理邏輯的位置。 中的轉譯回呼 `worker-basic` 只需將源檔案內容複製到格式副本檔案即可。

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## 呼叫外部API {#call-external-api}

在應用程式程式碼中，您可以進行外部API呼叫，以協助處理應用程式。 以下是叫用外部API的範例應用程式檔案。

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

例如， [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) 使用 [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) 程式庫。

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### 傳遞自訂參數 {#pass-custom-parameters}

您可以透過轉譯物件傳遞自訂定義的參數。 可在 [`rendition` 說明](https://github.com/adobe/asset-compute-sdk#rendition). 格式副本物件的範例為：

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

存取自訂參數的應用程式檔案範例為：

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

此 `example-worker-animal-pictures` 傳遞自訂參數 [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) 來確定要從維基媒體上擷取的檔案。

## 驗證和授權支援 {#authentication-authorization-support}

依預設，Asset compute自訂應用程式會隨附App Builder專案的授權和驗證檢查。 這可透過設定 `require-adobe-auth` 註解 `true` 在 `manifest.yml`.

### 存取其他AdobeAPI {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

將API服務新增至 [!DNL Asset Compute] 在設定中建立的主控台工作區。 這些服務是由 [!DNL Asset Compute Service]. 可在應用程式動作記憶體取代號和其他憑證 `params` 物件。

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### 傳遞第三方系統的憑證 {#pass-credentials-for-tp}

若要處理其他外部服務的憑證，請在動作上將這些傳遞為預設參數。 在傳輸過程中會自動加密。 如需詳細資訊，請參閱 [在執行階段開發人員指南中建立動作](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md). 然後在部署期間使用環境變數來設定。 這些參數可在 `params` 動作內的物件。

在 `inputs` 在 `manifest.yml`:

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

此 `$VAR` 運算式會從名為 `VAR`.

在開發期間，此值可在本機ENV檔案中設定為 `aio` 除了從調用的shell中設定的變數外，還會自動從ENV檔案讀取環境變數。 在此範例中，ENV檔案看起來類似：

```CONF
#...
SECRET_KEY=secret-value
```

若是生產部署，您可以在CI系統中設定環境變數，例如在GitHub動作中使用機密。 最後，請依下列方式存取應用程式內的預設參數：

```javascript
const key = params.secretKey;
```

## 調整應用程式大小 {#sizing-workers}

應用程式在 [!DNL Adobe I/O] 使用 [限制](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) 可透過 `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

由於Asset compute應用程式通常執行的處理範圍更廣，因此更有可能必須調整這些限制以獲得最佳效能（足夠大，可處理二進位資產）和效率（而不是由於未使用的容器記憶體而浪費資源）。

執行階段中動作的預設逾時為一分鐘，但可借由設定 `timeout` 限制（以毫秒為單位）。 如果您希望處理較大的檔案，請增加此次。 請考慮下載來源、處理檔案及上傳轉譯所花的總時間。 如果動作逾時，即在指定的逾時限制前未傳回啟動，則執行階段會捨棄容器，而不會重複使用。

Asset compute應用程式從本質上來說往往是網路和磁碟輸入或輸出綁定。 必須先下載來源檔案，處理通常需要大量資源，然後再上傳產生的轉譯。

操作容器可用的記憶體由指定 `memorySize` 以MB表示。 目前，這也定義了容器可存取的CPU數量，最重要的是，這是使用Runtime成本的關鍵要素（較大的容器成本更高）。 當處理需要更多記憶體或CPU時，請在此處使用較大的值，但請小心不要浪費資源，因為容器越大，總吞吐量就越低。

此外，可以使用 `concurrency` 設定。 這是單一容器（相同動作）取得的同時啟用次數。 在此模型中，動作容器就像Node.js伺服器，接收多個同時請求，但以上限為準。 若未設定，執行階段中的預設值為200，這非常適合於較小的App Builder動作，但由於Asset compute應用程式的本機處理和磁碟活動較為密集，因此通常太大。 某些應用程式（視其實作而定）在同時執行的活動中可能也無法正常運作。 asset computeSDK可借由將檔案寫入不同的唯一資料夾，來確保啟動分開。

測試應用程式以找出 `concurrency` 和 `memorySize`. 較大的容器=記憶體限制較高可能允許更多併發，但也可能浪費較低的流量。
