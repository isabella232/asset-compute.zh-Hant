---
title: 為 [!DNL Asset Compute Service]開發
description: 使用 [!DNL Asset Compute Service]建立自訂應用程式。
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '1605'
ht-degree: 0%

---

# 開發自訂應用程式 {#develop}

開始開發自訂應用程式之前：

* 請確定所有[必要條件](/help/understand-extensibility.md#prerequisites-and-provisioning)均已滿足。
* 安裝所需的軟體工具](/help/setup-environment.md#create-dev-environment)。[
* 請參閱[設定您的環境](setup-environment.md) ，以確定您已準備好建立自訂應用程式。

## 建立自訂應用程式 {#create-custom-application}

請確保本地安裝[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli)。

1. 若要建立自訂應用程式，請[建立Firefly應用程式](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli)。 要執行此操作，請在您的終端中執行`aio app init <app-name>`。

   如果您尚未登入，此命令會提示瀏覽器，要求您使用您的Adobe ID登入[Adobe開發人員控制台](https://console.adobe.io/)。 有關從cli登錄的詳細資訊，請參閱[此處](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#3-signing-in-from-cli)。

   Adobe建議您登入。 如果您有問題，請依照指示[建立應用程式，而不登入](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user)。

1. 登錄後，按照CLI中的提示操作，選擇用於應用程式的`Organization`、`Project`和`Workspace`。 選擇您在[設定環境時建立的專案和工作區](setup-environment.md)。

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Select Workspace myworkspace
   create console.json
   ```

1. 提示`Which Adobe I/O App features do you want to enable for this project?`時，選擇`Actions`。 請務必取消選取`Web Assets`選項，因為Web資產使用不同的驗證和授權檢查。

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. 提示`Which type of sample actions do you want to create?`時，請務必選擇`Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 按照其餘提示操作，在Visual Studio代碼（或您最喜愛的代碼編輯器）中開啟新應用程式。 其中包含自訂應用程式的架構和范常式式碼。

   請在這裡閱讀[ Firefly應用程式的主要元件](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#5-anatomy-of-a-project-firefly-application)。

   範本應用程式會運用我們的[Asset computeSDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk)來上傳、下載和協調應用程式轉譯，因此開發人員只需實作自訂應用程式邏輯。 在`actions/<worker-name>`資料夾內，`index.js`檔案是新增自訂應用程式程式碼的位置。

如需自訂應用程式的範例和構想，請參閱[自訂應用程式範例](#try-sample)。

### 添加憑據 {#add-credentials}

當您在建立應用程式時登入時，ENV檔案中會收集大部分的Firefly憑證。 不過，使用開發人員工具需要其他憑證。

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 開發人員工具儲存憑證 {#developer-tool-credentials}

用於測試具有實際[!DNL Asset Compute service]的自定義應用程式的開發者工具要求一個雲儲存容器來承載測試檔案，以及接收和顯示應用程式生成的格式副本。

>[!NOTE]
>
>這與[!DNL Adobe Experience Manager]的雲端儲存區分開，作為[!DNL Cloud Service]。 僅適用於使用Asset compute開發人員工具進行開發和測試。

請務必存取[支援的雲端儲存容器](https://github.com/adobe/asset-compute-devtool#prerequisites)。 此容器可視需要由多個開發人員跨不同專案共用。

#### 將憑據添加到ENV檔案 {#add-credentials-env-file}

將開發人員工具的下列認證新增至Firefly專案根目錄的ENV檔案：

1. 將絕對路徑新增至將服務新增至Firefly專案時建立的私密金鑰檔案：

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. 從Adobe開發人員控制台下載檔案。 前往專案的根目錄，然後按一下右上角的「全部下載」。 檔案的檔案名下載為`<namespace>-<workspace>.json`。 執行下列任一操作：

   * 將檔案重新命名為`console.json`，並將其移至專案的根目錄中。
   * 您可以選擇將絕對路徑新增至Adobe開發人員控制台整合JSON檔案。 這與在專案工作區中下載的[`console.json`](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user)檔案相同。

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
>`config.json`檔案包含憑據。 從專案內，將JSON檔案新增至`.gitignore`檔案，以防止共用。 .env和.aio檔案也是如此。

## 執行應用程式 {#run-custom-application}

在使用「Asset compute開發人員工具」執行應用程式之前，請正確配置[憑據](#developer-tool-credentials)。

要在開發人員工具中運行應用程式，請使用`aio app run`命令。 它會將動作部署至[!DNL Adobe I/O]執行階段，然後在您的本機電腦上啟動開發工具。 此工具用於在開發期間測試應用程式請求。 以下是範例轉譯請求：

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
>請勿將`--local`標幟與`run`命令搭配使用。 它無法用於[!DNL Asset Compute]自訂應用程式和Asset compute開發工具。 自訂應用程式由[!DNL Asset Compute Service]呼叫，該無法存取開發人員本機電腦上執行的動作。

請參閱[這裡](test-custom-application.md)如何測試應用程式並除錯。 完成自定義應用程式的開發後，[部署自定義應用程式](deploy-custom-application.md)。

## 嘗試由Adobe提供的示例應用程式 {#try-sample}

以下是範例自訂應用程式：

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [工人 — 動物圖片](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### 範本自訂應用程式 {#template-custom-application}

[worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)是模板應用程式。 它只需複製源檔案即可生成格式副本。 此應用程式的內容是在建立aio應用程式時選擇`Adobe Asset Compute`時收到的範本。

應用程式檔案[`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js)使用[`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview)來下載源檔案、協調每個轉譯處理，並將產生的轉譯上傳回雲端儲存空間。

應用程式代碼內定義的[`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required)是執行所有應用程式處理邏輯的位置。 `worker-basic`中的格式副本回調只會將源檔案內容複製到格式副本檔案中。

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

例如， [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46)使用[`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer)庫從Wikimedia對靜態URL提出提取請求。

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### 傳遞自訂參數 {#pass-custom-parameters}

您可以透過轉譯物件傳遞自訂定義的參數。 可在[`rendition`指示](https://github.com/adobe/asset-compute-sdk#rendition)的應用程式內引用它們。 格式副本物件的範例為：

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

`example-worker-animal-pictures`傳遞一個自定義參數[`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39)，以確定要從維基媒體獲取的檔案。

## 驗證和授權支援 {#authentication-authorization-support}

依預設，Asset compute自訂應用程式會隨附Firefly應用程式的授權和驗證檢查。 在`manifest.yml`中將`require-adobe-auth`附註設定為`true`即可啟用此功能。

### 存取其他AdobeAPI {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

將API服務新增至在設定中建立的[!DNL Asset Compute]控制台工作區。 這些服務是[!DNL Asset Compute Service]所產生JWT存取權杖的一部分。 可在應用程式動作`params`物件記憶體取代號和其他憑證。

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### 傳遞第三方系統的憑證 {#pass-credentials-for-tp}

若要處理其他外部服務的憑證，請在動作上將這些傳遞為預設參數。 在傳輸過程中會自動加密。 如需詳細資訊，請參閱執行階段開發人員指南](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md)中的[建立動作。 然後在部署期間使用環境變數來設定。 這些參數可在動作內的`params`物件中存取。

在`manifest.yml`的`inputs`內設定預設參數：

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

`$VAR`運算式會從名為`VAR`的環境變數讀取值。

在開發期間，該值可以在本地ENV檔案中設定為`aio`，除了調用shell中設定的變數外，還自動從ENV檔案讀取環境變數。 在此範例中，ENV檔案看起來類似：

```CONF
#...
SECRET_KEY=secret-value
```

若是生產部署，您可以在CI系統中設定環境變數，例如在GitHub動作中使用機密。 最後，請依下列方式存取應用程式內的預設參數：

```javascript
const key = params.secretKey;
```

## 調整應用程式大小 {#sizing-workers}

應用程式在[!DNL Adobe I/O]運行時中的容器中執行，該容器具有[limits](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md)，可通過`manifest.yml`配置：

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

執行階段中動作的預設逾時為一分鐘，但可透過設定`timeout`限制（以毫秒為單位）來增加。 如果您希望處理較大的檔案，請增加此次。 請考慮下載來源、處理檔案及上傳轉譯所花的總時間。 如果動作逾時，即在指定的逾時限制前未傳回啟動，則執行階段會捨棄容器，而不會重複使用。

Asset compute應用程式從本質上來說往往是網路和磁碟輸入或輸出綁定。 必須先下載來源檔案，處理通常需要大量資源，然後再上傳產生的轉譯。

`memorySize`為MB指定了可用於操作容器的記憶體。 目前，這也定義了容器可存取的CPU數量，最重要的是，這是使用Runtime成本的關鍵要素（較大的容器成本更高）。 當處理需要更多記憶體或CPU時，請在此處使用較大的值，但請小心不要浪費資源，因為容器越大，總吞吐量就越低。

此外，還可使用`concurrency`設定控制容器內的動作並行性。 這是單一容器（相同動作）取得的同時啟用次數。 在此模型中，動作容器就像Node.js伺服器，接收多個同時請求，但以上限為準。 若未設定，執行階段中的預設值為200，這對較小的Firefly動作很適合，但由於Asset compute應用程式的本機處理和磁碟活動較為密集，因此通常太大。 某些應用程式（視其實作而定）在同時執行活動時可能也無法正常運作。 asset computeSDK可借由將檔案寫入不同的唯一資料夾，來確保啟動分開。

測試應用程式以找到`concurrency`和`memorySize`的最佳數。 較大的容器=記憶體限制較高可能允許更多併發，但也可能浪費較低的流量。
