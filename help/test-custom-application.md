---
title: 測試和除錯 [!DNL Asset Compute Service] 自訂應用程式
description: 測試和除錯 [!DNL Asset Compute Service] 自訂應用程式。
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: ebc0d717b3f6fc4518f4a79cd44ebe8fdcf9ec6a
workflow-type: tm+mt
source-wordcount: '811'
ht-degree: 0%

---

# 測試自訂應用程式{#test-debug-custom-worker}並除錯

## 為自定義應用程式{#test-custom-worker}執行單元測試

在您的電腦上安裝[Docker Desktop](https://www.docker.com/get-started)。 要測試自定義工作器，請在應用程式的根目錄中執行以下命令：

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

如下所述，此框架會為項目中的Asset compute應用程式操作運行自定義單元測試框架。 它通過`package.json`檔案中的配置進行連接。 您也可以進行JavaScript單元測試，例如Jest。 `aio app test` 執行兩者。

[aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency)外掛程式已內嵌為自訂應用程式應用程式中的開發相依性，因此不需要將它安裝在組建／測試系統上。

### 應用單元測試框架{#unit-test-framework}

asset compute應用單元測試框架允許無需編寫任何代碼即可測試應用程式。 它依賴於應用程式的源到轉譯檔案原則。 必須設定特定的檔案和檔案夾結構，以定義測試來源檔案、選用參數、預期的轉譯和自訂驗證指令碼的測試案例。 依預設，會比較轉譯的位元組等式。 此外，使用簡單的JSON檔案，可輕鬆模擬外部HTTP服務。

### 新增測試{#add-tests}

測試需要在[!DNL Adobe I/O]專案根層級的`test`資料夾中進行。 每個應用程式的測試案例應位於路徑`test/asset-compute/<worker-name>`中，每個測試案例都有一個資料夾：

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

請參閱[範例自訂應用程式](https://github.com/adobe/asset-compute-example-workers/)，以取得一些範例。 以下是詳細的參考。

### 測試輸出{#test-output}

包括自訂應用程式記錄檔的詳細測試輸出可在Firefly應用程式根目錄的`build`資料夾中取用，如`aio app test`輸出所示。

### 模擬外部服務{#mock-external-services}

在測試案例中定義`mock-<HOST_NAME>.json`檔案，其中HOST_NAME是您要模擬的主機，以模擬動作中的外部服務呼叫。 範例使用案例是對S3進行個別呼叫的應用程式。 新的測試結構如下所示：

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

模擬檔案是JSON格式的http回應。 如需詳細資訊，請參閱本檔案](https://www.mock-server.com/mock_server/creating_expectations.html)。 [如果要模擬多個主機名，請定義多個`mock-<mocked-host>.json`檔案。 以下是名為`mock-google.com.json`的`google.com`範例模擬檔案：

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

示例`worker-animal-pictures`包含一個[模擬檔案](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json)，用於Wikimedia服務。

#### 跨測試案例共用檔案{#share-files-across-test-cases}

如果您在多個測試間共用`file.*`、`params.json`或`validate`指令碼，建議使用相對符號連結。 它們受到git支援。 請務必為您的共用檔案提供唯一的名稱，因為您可能有不同的名稱。 在下列範例中，測試是混合併比對幾個共用檔案，以及它們自己的檔案：

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

### 測試預期錯誤{#test-unexpected-errors}

錯誤測試案例不應包含預期的`rendition.*`檔案，而且應在`params.json`檔案中定義預期的`errorReason`。

>[!NOTE]
>
>如果測試案例不包含預期的`rendition.*`檔案，且未在`params.json`檔案中定義預期的`errorReason`，則假設它是任何`errorReason`的錯誤案例。

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

請參閱[Asset compute錯誤原因](https://github.com/adobe/asset-compute-commons#error-reasons)的完整清單和說明。

## 對自定義應用程式{#debug-custom-worker}進行調試

下列步驟顯示如何使用Visual Studio代碼調試自定義應用程式。 它可讓您檢視即時記錄、點擊中斷點、逐步執行程式碼，以及在每次啟動時即時重新載入本機程式碼變更。

其中許多步驟通常是由出廠時的`aio`自動執行的，請參閱[Firefly文檔](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)中的「調試應用程式」一節。 目前，下列步驟包含因應措施。

1. 從GitHub安裝最新的[wskdebug](https://github.com/apache/openwhisk-wskdebug)和可選的[ngrok](https://www.npmjs.com/package/ngrok)。

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. 新增至您的使用者設定JSON檔案。 它會持續使用舊的VS程式碼除錯程式，而新除錯程式的wskdebug有[某些問題](https://github.com/apache/openwhisk-wskdebug/issues/74):`"debug.javascript.usePreview": false`。
1. 關閉透過`aio app run`開啟的任何應用程式例項。
1. 使用`aio app deploy`部署最新代碼。
1. 僅使用`aio asset-compute devtool`執行Asset computeDevtool。 保持開放。
1. 在VS代碼編輯器中，將以下調試配置添加到`launch.json`中：

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

   從`aio app deploy`的輸出中讀取`ACTION NAME`。

1. 從運行／調試配置中選擇`wskdebug worker` ，然後按播放表徵圖。 等待啟動，直到在&#x200B;**[!UICONTROL 調試控制台]**&#x200B;窗口中顯示&#x200B;**[!UICONTROL 準備激活]**。

1. 在Devtool中按一下&#x200B;**[!UICONTROL run]**。 您可以在VS代碼編輯器中看到執行的操作，並開始顯示日誌。

1. 在程式碼中設定中斷點，再執行一次，就會點到。

任何程式碼變更都會即時載入，並在下次啟動時立即生效。

>[!NOTE]
>
>自訂應用程式中的每個要求都會有兩個啟動。 第一個要求是在SDK程式碼中以非同步方式呼叫自身的Web動作。 第二次啟動是點擊您程式碼的啟動。
