---
title: 為 [!DNL Asset Compute Service]
description: 使用 [!DNL Asset Compute Service]。
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '1618'
ht-degree: 0%

---

# 開發自定義應用程式 {#develop}

在開始開發自定義應用程式之前：

* 確保 [先決條件](/help/understand-extensibility.md#prerequisites-and-provisioning) 。
* 安裝 [所需的軟體工具](/help/setup-environment.md#create-dev-environment)。
* 請參閱 [設定環境](setup-environment.md) 以確保您已準備好建立自定義應用程式。

## 建立自定義應用程式 {#create-custom-application}

確保 [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) 本地安裝。

1. 要建立自定義應用程式， [建立App Builder項目](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli)。 為此，請執行 `aio app init <app-name>` 在終端上。

   如果尚未登錄，此命令將提示瀏覽器要求您登錄 [Adobe Developer控制台](https://console.adobe.io/) 你的Adobe ID。 請參閱 [這裡](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli) 有關從cli登錄的詳細資訊。

   Adobe建議您登錄。 如果您有問題，請按照說明操作 [建立應用而不登錄](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user)。

1. 登錄後，按照CLI中的提示選擇 `Organization`。 `Project`, `Workspace` 的子菜單。 選擇在您 [設定環境](setup-environment.md)。 提示時 `Which extension point(s) do you wish to implement ?`，確保選擇 `DX Asset Compute Worker`:

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

1. 出現提示時 `Which Adobe I/O App features do you want to enable for this project?`選中 `Actions`。 確保取消選擇 `Web Assets` 選項，因為web資產使用不同的身份驗證和授權檢查。

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. 提示時 `Which type of sample actions do you want to create?`，確保選擇 `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 按照其餘提示操作，在Visual Studio代碼（或您最喜愛的代碼編輯器）中開啟新應用程式。 它包含定製應用程式的腳手架和示例代碼。

   在此處閱讀有關 [App Builder應用程式的主要元件](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#5-anatomy-of-an-app-builder-application)。

   模板應用程式利用了 [asset computeSDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) 用於上傳、下載和協調應用程式格式副本，因此開發人員只需要實施自定義應用程式邏輯。 在 `actions/<worker-name>` 資料夾 `index.js` 檔案是添加自定義應用程式碼的位置。

請參閱 [示例自定義應用程式](#try-sample) 示例和自定義應用程式的想法。

### 添加憑據 {#add-credentials}

在建立應用程式時登錄時，大多數App Builder憑據都會在ENV檔案中收集。 但是，使用開發人員工具需要其他憑據。

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 開發人員工具儲存憑據 {#developer-tool-credentials}

用於將自定義應用程式與實際應用程式test的開發人員工具 [!DNL Asset Compute service] 需要一個雲儲存容器，用於托管test檔案以及接收和顯示應用程式生成的格式副本。

>[!NOTE]
>
>這與雲儲存 [!DNL Adobe Experience Manager] 作為 [!DNL Cloud Service]。 它只適用於Asset compute開發工具的開發和測試。

確保有權訪問 [支援的雲儲存容器](https://github.com/adobe/asset-compute-devtool#prerequisites)。 此容器可由多個開發人員根據需要跨不同項目共用。

#### 將憑據添加到ENV檔案 {#add-credentials-env-file}

將開發人員工具的以下憑據添加到App Builder項目根目錄中的ENV檔案：

1. 將服務添加到App Builder項目時建立的私鑰檔案的絕對路徑：

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. 從Adobe Developer控制台下載檔案。 轉到項目的根目錄，然後按一下右上角的「Download All（全部下載）」。 檔案下載時 `<namespace>-<workspace>.json` 檔案名。 執行下列任一項作業：

   * 將檔案更名為 `console.json` 然後移到項目的根部。
   * 或者，可以將絕對路徑添加到Adobe Developer控制台整合JSON檔案。 這是相同的 [`console.json`](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) 下載到項目工作區中的檔案。

      ```conf
      ASSET_COMPUTE_INTEGRATION_FILE_PATH=
      ```

1. 添加S3或Azure儲存憑據。 您只需訪問一個雲儲存解決方案。

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
>的 `config.json` 檔案包含憑據。 從項目內，將JSON檔案添加到 `.gitignore` 檔案以阻止其共用。 同樣適用於.env和.aio檔案。

## 執行應用程式 {#run-custom-application}

使用Asset compute開發工具執行應用程式之前，請正確配置 [憑據](#developer-tool-credentials)。

要在開發工具中運行應用程式，請使用 `aio app run` 的子菜單。 它將操作部署到 [!DNL Adobe I/O] 運行時並啟動本地電腦上的開發工具。 此工具用於在開發過程中test應用程式請求。 下面是一個格式副本請求示例：

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
>不要使用 `--local` 帶有 `run` 的子菜單。 這和 [!DNL Asset Compute] 自定義應用程式和Asset compute開發工具。 自定義應用程式由 [!DNL Asset Compute Service] 無法訪問在開發人員的本地電腦上運行的操作。

請參閱 [這裡](test-custom-application.md) 如何test和調試應用程式。 完成自定義應用程式的開發後， [部署自定義應用程式](deploy-custom-application.md)。

## 嘗試通過Adobe提供的示例應用程式 {#try-sample}

以下是示例自定義應用程式：

* [工作基本](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [工人動物圖片](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### 模板自定義應用程式 {#template-custom-application}

的 [工作基本](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) 是模板應用程式。 它只需複製源檔案即可生成格式副本。 此應用程式的內容是選擇時收到的模板 `Adobe Asset Compute` 建立aio應用。

應用程式檔案， [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 使用 [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) 下載源檔案，協調每個格式副本處理，並將生成的格式副本上載回雲儲存。

的 [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) 定義在應用程式碼中，是執行所有應用程式處理邏輯的位置。 格式副本回調 `worker-basic` 只需將源檔案內容複製到格式副本檔案即可。

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## 調用外部API {#call-external-api}

在應用程式碼中，可以進行外部API調用以幫助處理應用程式。 下面是調用外部API的示例應用程式檔案。

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

例如， [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) 使用 [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer) 的下界。

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### 傳遞自定義參數 {#pass-custom-parameters}

可以通過格式副本對象傳遞自定義的參數。 可以在應用程式中引用這些 [`rendition` 說明](https://github.com/adobe/asset-compute-sdk#rendition)。 格式副本對象的示例為：

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

訪問自定義參數的應用程式檔案的示例是：

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

的 `example-worker-animal-pictures` 傳遞自定義參數 [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) 確定從維基媒體獲取的檔案。

## 身份驗證和授權支援 {#authentication-authorization-support}

預設情況下，Asset compute自定義應用程式附帶對App Builder項目的「授權」和「驗證」檢查。 通過設定 `require-adobe-auth` 注釋 `true` 的 `manifest.yml`。

### 訪問其他AdobeAPI {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

將API服務添加到 [!DNL Asset Compute] 在設定中建立的控制台工作區。 這些服務是由 [!DNL Asset Compute Service]。 在應用程式操作中可以訪問令牌和其他憑據 `params` 的雙曲餘切值。

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### 為第三方系統傳遞憑據 {#pass-credentials-for-tp}

要處理其他外部服務的憑據，請將這些憑據作為操作的預設參數傳遞。 這些在傳輸中自動加密。 有關詳細資訊，請參見 [在運行時開發人員指南中建立操作](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md)。 然後在部署過程中使用環境變數設定它們。 這些參數可在 `params` 對象。

在 `inputs` 的 `manifest.yml`:

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

的 `$VAR` 表達式從名為 `VAR`。

在開發過程中，可以在本地ENV檔案中將值設定為 `aio` 除了從調用shell設定的變數之外，還會自動從ENV檔案中讀取環境變數。 在此示例中，ENV檔案如下所示：

```CONF
#...
SECRET_KEY=secret-value
```

對於生產部署，可以在CI系統中設定環境變數，例如在GitHub操作中使用機密。 最後，訪問應用程式內的預設參數，如下：

```javascript
const key = params.secretKey;
```

## 調整應用程式大小 {#sizing-workers}

應用程式在 [!DNL Adobe I/O] 運行時 [限制](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) 可以通過 `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

由於Asset compute應用程式通常執行的處理範圍更廣，因此更有可能必須調整這些限制以獲得最佳效能（足夠大以處理二進位資產）和效率（不會因為未使用的容器記憶體而浪費資源）。

運行時中操作的預設超時為一分鐘，但可以通過設定 `timeout` 限制（毫秒）。 如果希望處理較大的檔案，請增加此次。 考慮下載源、處理檔案和上載格式副本所花的總時間。 如果操作超時，即在指定超時限制之前不返回激活，則運行時將丟棄容器並不重用它。

Asset compute應用程式從本質上來說往往是網路和磁碟輸入或輸出綁定。 必須先下載源檔案，處理通常佔用大量資源，然後重新上載生成的格式副本。

操作容器可用的記憶體由 `memorySize` MB。 當前，它還定義了容器獲得的CPU訪問量，而且最重要的是，它是使用運行時成本的關鍵因素（較大的容器成本更高）。 當處理需要更多記憶體或CPU時，請在此處使用較大的值，但請小心不要浪費資源，因為容器越大，總吞吐量越低。

此外，可利用該方法控制容器內的動作併發 `concurrency` 的子菜單。 這是單個容器（相同操作）獲取的併發激活數。 在此模型中，操作容器類似於接收多個併發請求的Node.js伺服器，直到達此限制。 如果未設定，則運行時中的預設值為200，這對較小的App Builder操作非常有用，但由於應用程式的本地處理和磁碟活動更為密集，因此通常對Asset compute應用程式來說太大。 某些應用程式（視其實施情況而定）在併發活動中可能也不能正常工作。 asset computeSDK通過將檔案寫入不同的唯一資料夾來確保激活分離。

Test應用程式以查找 `concurrency` 和 `memorySize`。 較大的容器=記憶體限制較高可能允許更多併發，但也可能會浪費較低的流量。
