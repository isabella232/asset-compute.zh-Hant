---
title: 測試和調試 [!DNL Asset Compute Service] 自定義應用程式
description: 測試和調試 [!DNL Asset Compute Service] 自定義應用程式。
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: ebc0d717b3f6fc4518f4a79cd44ebe8fdcf9ec6a
workflow-type: tm+mt
source-wordcount: '811'
ht-degree: 0%

---

# 測試和調試自定義應用程式{#test-debug-custom-worker}

## 對自定義應用程式{#test-custom-worker}執行單元測試

在電腦上安裝[Docker Desktop](https://www.docker.com/get-started)。 要測試自定義工作器，請在應用程式的根目錄中執行以下命令：

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

如下所述，此程式會為專案中的Asset compute應用程式動作執行自訂單元測試架構。 可透過`package.json`檔案中的設定來連接。 也可以進行JavaScript單元測試，例如Jest。 `aio app test` 同時執行。

[aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency)外掛程式在自訂應用程式應用程式中以開發相依性形式內嵌，因此不需要安裝在組建/測試系統上。

### 應用單元測試框架{#unit-test-framework}

asset compute應用單元測試框架允許測試應用而不寫任何代碼。 它依賴應用程式的源到轉譯檔案原則。 必須設定特定檔案和資料夾結構，以使用測試來源檔案、選用參數、預期的轉譯和自訂驗證指令碼來定義測試案例。 依預設，會比較轉譯以取得位元組相等。 此外，使用簡單的JSON檔案，即可輕鬆模擬外部HTTP服務。

### 添加測試{#add-tests}

應在[!DNL Adobe I/O]項目的根級`test`資料夾內進行測試。 每個應用程式的測試案例應位於路徑`test/asset-compute/<worker-name>`中，每個測試案例應包含一個資料夾：

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

查看[示例自定義應用程式](https://github.com/adobe/asset-compute-example-workers/)以了解一些示例。 下面是詳細的參考。

### 測試輸出{#test-output}

包括自訂應用程式記錄在內的詳細測試輸出可在位於Firefly應用程式根目錄的`build`資料夾中使用，如`aio app test`輸出所示。

### 模擬外部服務{#mock-external-services}

您可以在測試案例中定義`mock-<HOST_NAME>.json`檔案，其中HOST_NAME是您要模擬的主機，借此模擬動作中的外部服務呼叫。 對S3進行個別呼叫的應用程式就是範例使用案例。 新的測試結構如下所示：

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

模擬檔案為JSON格式的http回應。 如需詳細資訊，請參閱本檔案[](https://www.mock-server.com/mock_server/creating_expectations.html)。 如果有多個要模擬的主機名，請定義多個`mock-<mocked-host>.json`檔案。 以下是`google.com`的範例模擬檔案，名為`mock-google.com.json`:

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

示例`worker-animal-pictures`包含與其交互的Wiki媒體服務的[模擬檔案](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json)。

#### 跨測試案例共用檔案{#share-files-across-test-cases}

如果您在多個測試間共用`file.*`、`params.json`或`validate`指令碼，建議使用相對符號連結。 Git支援這些功能。 請務必為共用檔案指定唯一的名稱，因為您可能有不同的名稱。 在下列範例中，測試會混合併比對幾個共用檔案，以及它們自己的檔案：

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

錯誤測試案例不應包含預期的`rendition.*`檔案，並應在`params.json`檔案內定義預期的`errorReason`。

>[!NOTE]
>
>如果測試案例不包含預期的`rendition.*`檔案，且未在`params.json`檔案內定義預期的`errorReason`，則假設該案例是任何`errorReason`的錯誤案例。

錯誤測試用例結構：

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

## 調試自定義應用程式{#debug-custom-worker}

下列步驟顯示如何使用Visual Studio Code調試自定義應用程式。 它可讓您查看即時記錄、點擊中斷點、逐步執行程式碼，以及在每次啟動時即時重新載入本機程式碼變更。

其中許多步驟通常由`aio`自動執行，請參閱[Firefly檔案](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)中的應用程式除錯一節。 目前，下列步驟包含因應措施。

1. 從GitHub安裝最新的[wskdebug](https://github.com/apache/openwhisk-wskdebug)以及選用的[ngrok](https://www.npmjs.com/package/ngrok)。

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. 新增至您的使用者設定JSON檔案。 它持續使用舊版VS程式碼偵錯器，新版偵錯器有[部分問題](https://github.com/apache/openwhisk-wskdebug/issues/74) wskdebug:`"debug.javascript.usePreview": false`。
1. 關閉透過`aio app run`開啟的應用程式任何例項。
1. 使用`aio app deploy`部署最新代碼。
1. 僅使用`aio asset-compute devtool`執行Asset computeDevtool。 保持開啟。
1. 在VS程式碼編輯器中，將下列除錯設定新增至您的`launch.json`:

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

   從`aio app deploy`的輸出擷取`ACTION NAME`。

1. 從運行/調試配置中選擇`wskdebug worker`，然後按播放表徵圖。 等待它啟動，直到它在&#x200B;**[!UICONTROL 調試控制台]**&#x200B;窗口中顯示&#x200B;**[!UICONTROL 準備激活]**。

1. 在Devtool中按一下&#x200B;**[!UICONTROL run]**。 您可以在VS程式碼編輯器中看到執行的動作，且記錄開始顯示。

1. 在代碼中設定斷點，重新運行，該斷點應該已命中。

任何程式碼變更都會即時載入，並在下次啟動時立即生效。

>[!NOTE]
>
>自訂應用程式中，每個請求會有兩個啟用。 第一個要求是Web動作，會在SDK程式碼中以非同步方式呼叫本身。 第二次啟動是點擊您程式碼的啟動。
