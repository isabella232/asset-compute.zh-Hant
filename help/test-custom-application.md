---
title: 測試和除錯自 [!DNL Asset Compute Service] 訂應用程式。
description: 測試和除錯自 [!DNL Asset Compute Service] 訂應用程式。
translation-type: tm+mt
source-git-commit: 54afa44d8d662ee1499a385f504fca073ab6c347
workflow-type: tm+mt
source-wordcount: '788'
ht-degree: 0%

---


# 測試和除錯自訂應用程式 {#test-debug-custom-worker}

## 自訂應用程式的執行單元測試 {#test-custom-worker}

在您 [的電腦上安裝Docker Desktop](https://www.docker.com/get-started) 。 要測試自定義工作器，請在應用程式的根目錄中執行以下命令：

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `adobe-asset-compute test-worker` command in the root of the custom application application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

如下所述，此框架會為項目中的「資產計算」應用程式操作運行自定義單元測試框架。 它通過檔案中的配置 `package.json` 連接。 您也可以進行JavaScript單元測試，例如Jest。 `aio app test` 執行兩者。

在自 [訂應用程式應用程式中，aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) plugin會內嵌為開發相依性，因此不需要將它安裝在建置／測試系統上。

### 應用單元測試框架 {#unit-test-framework}

Asset Compute應用程式單元測試架構可讓您測試應用程式，毋需編寫任何程式碼。 它依賴於應用程式的源到轉譯檔案原則。 必須設定特定的檔案和檔案夾結構，以定義測試來源檔案、選用參數、預期的轉譯和自訂驗證指令碼的測試案例。 依預設，會比較轉譯的位元組等式。 此外，使用簡單的JSON檔案，可輕鬆模擬外部HTTP服務。

### 新增測試 {#add-tests}

測試需要在AIO項 `test` 目根級別的資料夾內進行。 每個應用程式的測試案例應位於路徑中， `test/asset-compute/<worker-name>`每個測試案例應包含一個資料夾：

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

請檢視自訂應 [用程式的範例](https://github.com/adobe/asset-compute-example-workers/) ，以取得一些範例。 以下是詳細的參考。

### Test output {#test-output}

Firefly應用程式根目錄的資料夾中提供詳細的測試輸出，包括自訂應用程 `build` 式的記錄檔，如輸出中所 `aio app test` 示。

### 模擬外部服務 {#mock-external-services}

在測試案例中定義檔案(其中 `mock-<HOST_NAME>.json` HOST_NAME是您要模擬的主機)，以模擬動作中的外部服務呼叫。 範例使用案例是對S3進行個別呼叫的應用程式。 新的測試結構如下所示：

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

模擬檔案是JSON格式的http回應。 如需詳細資訊，請參 [閱本檔案](https://www.mock-server.com/mock_server/creating_expectations.html)。 如果要模擬多個主機名，請定義多個 `mock-<mocked-host>.json` 檔案。 以下是名為的範例模 `google.com` 擬檔 `mock-google.com.json`:

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

此範例 `worker-animal-pictures` 包含 [Wikimedia服務](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) （它與之互動）的模擬檔案。

#### 跨測試案例共用檔案 {#share-files-across-test-cases}

如果您在多個測試間共用或指令碼，建議 `file.*`使 `params.json` 用相 `validate` 對symlinks。 它們受到git支援。 請務必為您的共用檔案提供唯一的名稱，因為您可能有不同的名稱。 在下列範例中，測試是混合併比對幾個共用檔案，以及它們自己的檔案：

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### 測試預期錯誤 {#test-unexpected-errors}

錯誤測試案例不應包含預期的 `rendition.*` 檔案，而應定義檔 `errorReason` 案中的 `params.json` 預期。

錯誤測試案例結構：

```json
<error_test_case>/
    file.jpg
    params.json
```

參數檔案，錯誤原因：

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

請參閱「資產計算」錯誤原 [因的完整清單和說明](https://github.com/adobe/asset-compute-commons#error-reasons)。

## 對自訂應用程式除錯 {#debug-custom-worker}

下列步驟顯示如何使用Visual Studio代碼調試自定義應用程式。 它可讓您檢視即時記錄、點擊中斷點、逐步執行程式碼，以及在每次啟動時即時重新載入本機程式碼變更。

其中許多步驟通常都會立即自 `aio` 動執行，請參閱 [Firefly檔案中的除錯應用程式一節](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)。 目前，下列步驟包含因應措施。

1. 從GitHub安裝最 [新的wskdebug](https://github.com/apache/openwhisk-wskdebug) ，以及選 [用的ngrok](https://www.npmjs.com/package/ngrok)。

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. 新增至您的使用者設定JSON檔案。 它會持續使用舊的VS程式碼除錯程式，而新除錯程式與wskdebug [有一些問](https://github.com/apache/openwhisk-wskdebug/issues/74) 題： `"debug.javascript.usePreview": false`.
1. 關閉透過開啟的任何應用程式例項 `aio app run`。
1. 使用部署最新的程式碼 `aio app deploy`。
1. 僅使用執行資產計算設備工具 `npx adobe-asset-compute devtool`。 保持開放。
1. 在VS程式碼編輯器中，將下列除錯設定新增至您的 `launch.json`:

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   從的輸出中獲取ACTION NAME `aio app deploy`。 看起來像 `Your deployed actions -> TypicalCoffeeCat-0.0.1/__secured_worker`。

1. 從執 `wskdebug worker` 行／除錯設定中選取，然後按播放圖示。 等待啟動，直到在「除錯控制台 **[!UICONTROL 」視窗中顯示]** 「 **[!UICONTROL Ready for activations]** 」。

1. 在「 **[!UICONTROL Devtool]** 」中按一下「執行」。 您可以在VS代碼編輯器中看到執行的操作，並開始顯示日誌。

1. 在程式碼中設定中斷點，再執行一次，就會到達。

任何程式碼變更都會即時載入，並在下次啟動時立即生效。

>[!NOTE]
>
>自訂應用程式中的每個要求都會有兩個啟動。 第一個要求是在SDK程式碼中以非同步方式呼叫自身的Web動作。 第二次啟動是點擊您程式碼的啟動。
