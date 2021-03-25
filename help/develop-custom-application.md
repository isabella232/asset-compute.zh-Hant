---
title: 開發 [!DNL Asset Compute Service]
description: 使用 [!DNL Asset Compute Service]建立自訂應用程式。
translation-type: tm+mt
source-git-commit: 7ae47fdb7ff91e1388d2037d90abe35fe5218216
workflow-type: tm+mt
source-wordcount: '1615'
ht-degree: 0%

---


# 開發自訂應用程式{#develop}

開始開發自訂應用程式之前：

* 確保滿足所有[先決條件](/help/understand-extensibility.md#prerequisites-and-provisioning)。
* 安裝[所需的軟體工具](/help/setup-environment.md#create-dev-environment)。
* 請參閱[設定您的環境](setup-environment.md)，以確定您已準備好建立自訂應用程式。

## 建立自定義應用程式{#create-custom-application}

請確保在本地安裝[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli)。

1. 若要建立自訂應用程式，請[建立Firefly應用程式](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#4-bootstrapping-new-app-using-the-cli)。 若要這麼做，請在您的終端中執行`aio app init <app-name>`。

   如果您尚未登入，此命令會提示瀏覽器要求您使用您的Adobe ID登入[Adobe開發人員主控台](https://console.adobe.io/)。 有關從cli登入的詳細資訊，請參閱[此處](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli)。

   Adobe建議您登入。 如果您有問題，請依照[指示建立應用程式，而不登入](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user)。

1. 登錄後，請按照CLI中的提示操作，並選擇`Organization`、`Project`和`Workspace`以用於應用程式。 選擇在[設定環境時建立的項目和工作區。](setup-environment.md)

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Select Workspace myworkspace
   create console.json
   ```

1. 出現提示時，選擇`Which Adobe I/O App features do you want to enable for this project?`。 `Actions`請確定取消選取`Web Assets`選項，因為Web資產會使用不同的驗證和授權檢查。

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. 當提示`Which type of sample actions do you want to create?`時，請務必選擇`Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. 依照其他提示，在Visual Studio代碼（或您最愛的代碼編輯器）中開啟新應用程式。 它包含自訂應用程式的架構和范常式式碼。

   請在這裡閱讀有關Firefly應用程式[主要元件的資訊。](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application)

   範本應用程式運用我們的[Asset computeSDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk)來上傳、下載和協調應用程式轉譯，因此開發人員只需建置自訂應用程式邏輯。 在`actions/<worker-name>`資料夾中，`index.js`檔案是新增自訂應用程式碼的位置。

如需自訂應用程式的範例和構想，請參閱[範例自訂應用程式](#try-sample)。

### 添加憑據{#add-credentials}

當您在建立應用程式時登錄時，大多數Firefly憑據都會收集到ENV檔案中。 不過，使用開發人員工具需要額外的認證。

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### 開發人員工具儲存憑證{#developer-tool-credentials}

用於測試實際[!DNL Asset Compute service]自訂應用程式的開發人員工具，需要有雲端儲存容器來代管測試檔案，以及接收和顯示應用程式產生的轉譯。

>[!NOTE]
>
>這與[!DNL Adobe Experience Manager]的雲端儲存區不同，是[!DNL Cloud Service]。 它僅適用於使用Asset compute開發人員工具進行開發與測試。

請務必存取[支援的雲端儲存容器](https://github.com/adobe/asset-compute-devtool#prerequisites)。 此容器可視需要由多位開發人員跨不同專案共用。

#### 將憑據添加到ENV檔案{#add-credentials-env-file}

將開發人員工具的下列認證新增至Firefly專案根目錄的ENV檔案：

1. 將服務新增至Firefly專案時建立的私密金鑰檔案的絕對路徑：

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. 從Adobe開發人員主控台下載檔案。 前往專案的根目錄，然後按一下右上角的「全部下載」。 檔案以`<namespace>-<workspace>.json`作為檔案名下載。 執行下列任一項作業：

   * 將檔案重新命名為`console.json`，並將它移至專案的根目錄中。
   * 或者，您也可以將絕對路徑新增至「Adobe開發人員主控台」整合JSON檔案。 這是您專案工作區中下載的相同[`console.json`](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#42-developer-is-not-logged-in-as-enterprise-organization-user)檔案。

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
>`config.json`檔案包含憑據。 從您的專案中，將JSON檔案新增至您的`.gitignore`檔案，以防止共用檔案。 同樣的情況也適用於。env和。aio檔案。

## 執行應用程式{#run-custom-application}

在使用Asset compute開發人員工具執行應用程式之前，請正確設定[認證](#developer-tool-credentials)。

若要在開發人員工具中執行應用程式，請使用`aio app run`命令。 它會將動作部署至「[!DNL Adobe I/O]執行階段」，並在您的本機電腦上啟動開發工具。 此工具用於在開發期間測試應用程式要求。 以下是範例轉譯請求：

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
>請勿將`--local`標幟與`run`命令搭配使用。 它無法用於[!DNL Asset Compute]自訂應用程式和Asset compute開發人員工具。 自訂應用程式由[!DNL Asset Compute Service]呼叫，無法存取開發人員本機電腦上執行的動作。

請參閱[此處](test-custom-application.md)如何測試和除錯應用程式。 當您完成自訂應用程式的開發後，[會部署自訂應用程式](deploy-custom-application.md)。

## 試用Adobe{#try-sample}提供的範例應用程式

以下是自訂應用程式的範例：

* [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [工人——動物——圖片](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### 範本自訂應用程式{#template-custom-application}

[worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)是模板應用程式。 它只要複製來源檔案，就會產生轉譯。 此應用程式的內容是在建立aio應用程式時選擇`Adobe Asset Compute`時收到的範本。

應用程式檔案[`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js)使用[`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview)來下載來源檔案、協調每個轉譯處理，並將產生的轉譯上傳回雲端儲存空間。

應用程式碼內定義的[`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required)是執行所有應用程式處理邏輯的位置。 `worker-basic`中的轉譯回呼只會將來源檔案內容複製到轉譯檔案。

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## 呼叫外部API {#call-external-api}

在應用程式碼中，您可以進行外部API呼叫，以協助應用程式處理。 下面是調用外部API的示例應用程式檔案。

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

例如，[`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46)使用[`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer)程式庫，從維基媒體對靜態URL提出擷取要求。

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### 傳遞自訂參數{#pass-custom-parameters}

您可以透過轉譯物件傳遞自訂定義的參數。 在[`rendition`指令](https://github.com/adobe/asset-compute-sdk#rendition)中，可在應用程式內參考這些指令。 轉譯物件的範例是：

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

`example-worker-animal-pictures`會傳遞自訂參數[`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39)，以決定要從維基媒體擷取的檔案。

## 驗證和授權支援{#authentication-authorization-support}

依預設，Asset compute自訂應用程式會隨附Firefly應用程式的授權和驗證檢查。 通過將`require-adobe-auth`注釋設定為`manifest.yml`中的`true`來啟用此功能。

### 存取其他AdobeAPI {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

將API服務添加到在設定中建立的[!DNL Asset Compute]控制台工作區。 這些服務是由[!DNL Asset Compute Service]生成的JWT訪問令牌的一部分。 Token和其他憑證可在應用程式動作`params`物件記憶體取。

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### 傳遞第三方系統{#pass-credentials-for-tp}的認證

要處理其他外部服務的憑據，請將這些作為操作的預設參數傳遞。 在傳輸中會自動加密這些檔案。 如需詳細資訊，請參閱「執行階段開發人員指南」中的「建立動作」。 [](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md)然後在部署期間使用環境變數來設定這些變數。 這些參數可在動作內的`params`物件中存取。

在`manifest.yml`的`inputs`中設定預設參數：

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

`$VAR`表達式從名為`VAR`的環境變數中讀取值。

在開發過程中，除了從調用的殼中設定的變數外，該值還可以在本地ENV檔案中設定為`aio`自動從ENV檔案讀取環境變數。 在此示例中，ENV檔案的外觀如下：

```CONF
#...
SECRET_KEY=secret-value
```

對於生產部署，您可以在CI系統中設定環境變數，例如使用GitHub動作中的機密。 最後，存取應用程式內的預設參數，如下：

```javascript
const key = params.secretKey;
```

## 調整應用程式大小{#sizing-workers}

應用程式會在[!DNL Adobe I/O]執行階段的容器中執行，其中[limits](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md)可透過`manifest.yml`進行設定：

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

由於Asset compute應用程式通常會進行更廣泛的處理，因此更可能必須調整這些限制，以獲得最佳效能（大到足以處理二進位資產）和效率（而不會因為未使用的容器記憶體而浪費資源）。

Runtime中動作的預設逾時為一分鐘，但可設定`timeout`限制（以毫秒為單位）來增加。 如果您希望處理較大的檔案，請增加此次。 請考慮下載來源、處理檔案及上傳轉譯所需的總時間。 如果動作逾時，例如在指定逾時限制之前未傳回啟動，則執行階段會放棄容器，而不會重複使用它。

Asset compute應用程式從本質上看往往是網路和磁碟輸入或輸出綁定。 必須先下載來源檔案，處理通常需要耗費大量資源，然後會重新上傳產生的轉譯。

動作容器可用的記憶體由`memorySize`指定，單位為MB。 目前這也定義了容器可存取的CPU數量，最重要的是，它是使用Runtime成本的關鍵元素（較大容器的成本較高）。 當您的處理需要更多記憶體或CPU時，請在此處使用較大的值，但請小心不要浪費資源，因為容器越大，總體吞吐量越低。

此外，使用`concurrency`設定可控制容器內動作並行性。 這是單個容器（相同動作）所獲取的並行啟動次數。 在此模型中，動作容器就像Node.js伺服器，可接收多個並行請求，但最高限制為此。 如果未設定，則執行階段中的預設值為200，這對較小的Firefly動作非常有用，但由於Asset compute應用程式的本機處理和磁碟活動較為密集，因此通常會過大。 某些應用程式（視其實作而定）也可能無法與並行活動搭配運作。 asset computeSDK可將檔案寫入不同的唯一資料夾，以確保個別啟動。

測試應用程式，以找出`concurrency`和`memorySize`的最佳數目。 較大的容器=記憶體限制較高可能允許更多併發，但流量較低也可能會浪費。
