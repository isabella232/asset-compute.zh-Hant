---
title: Test和調試 [!DNL Asset Compute Service] 自定義應用程式
description: Test和調試 [!DNL Asset Compute Service] 自定義應用程式。
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '812'
ht-degree: 0%

---

# Test和調試自定義應用程式 {#test-debug-custom-worker}

## 為自定義應用程式執行單元test {#test-custom-worker}

安裝 [Docker案頭](https://www.docker.com/get-started) 在你的機器上。 要test自定義工作人員，請在應用程式的根目錄下執行以下命令：

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

此操作將運行自定義設備test框架，用於Asset compute項目中的應用程式操作，如下所述。 它通過 `package.json` 的子菜單。 還可以有JavaScript單元test，如Jest。 `aio app test` 同時執行。

的 [aio-cli插件 — 資產計算](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) 插件作為開發依賴項嵌入到自定義應用程式應用中，因此不需要在生成/test系統上安裝它。

### 應用單元test框架 {#unit-test-framework}

asset compute應用單元test框架允許test應用而不寫入任何代碼。 它依賴於應用程式的源檔案到格式副本檔案原則。 必須設定某個檔案和資料夾結構，以定義具有test源檔案、可選參數、預期格式副本和自定義驗證指令碼的test案例。 預設情況下，格式副本會以位元組等式進行比較。 此外，使用簡單的JSON檔案可以輕鬆地模擬外部HTTP服務。

### 添加test {#add-tests}

Test預計在 `test` 資料夾的根級別 [!DNL Adobe I/O] 項目。 每個應用程式的test案例應位於路徑中 `test/asset-compute/<worker-name>`，每個test案例都有一個資料夾：

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

看看 [示例自定義應用程式](https://github.com/adobe/asset-compute-example-workers/) 例子。 下面是詳細參考。

### Test輸出 {#test-output}

包括自定義應用程式日誌在內的詳細test輸出可在 `build` 資料夾，如 `aio app test` 。

### 模擬外部服務 {#mock-external-services}

通過定義 `mock-<HOST_NAME>.json` test中的檔案，其中HOST_NAME是要模擬的主機。 一個示例用例是對S3進行單獨調用的應用程式。 新的test結構將是這樣的：

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

模擬檔案是JSON格式的http響應。 有關詳細資訊，請參見 [本文檔](https://www.mock-server.com/mock_server/creating_expectations.html)。 如果有多個要模擬的主機名，請定義多個 `mock-<mocked-host>.json` 的子菜單。 下面是示例模擬檔案 `google.com` 命名 `mock-google.com.json`:

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

示例 `worker-animal-pictures` 包含 [模擬檔案](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) 維基媒體服務。

#### 跨test共用檔案 {#share-files-across-test-cases}

如果共用，建議使用相對符號連結 `file.*`。 `params.json` 或 `validate` 指令碼跨多個test。 Git支援這些。 請確保為共用檔案指定唯一的名稱，因為您可能有不同的名稱。 在下面的示例中，test正在混合和匹配幾個共用檔案，並且它們自己：

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

### Test預期錯誤 {#test-unexpected-errors}

錯誤test案例不應包含預期 `rendition.*` 檔案，並應定義 `errorReason` 內 `params.json` 的子菜單。

>[!NOTE]
>
>如果test案例不包含預期 `rendition.*` 檔案，但不定義預期 `errorReason` 內 `params.json` 檔案，假定它是任何 `errorReason`。

錯誤Test案例結構：

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

請參閱的完整清單和說明 [asset compute錯誤原因](https://github.com/adobe/asset-compute-commons#error-reasons)。

## 調試自定義應用程式 {#debug-custom-worker}

以下步驟顯示如何使用Visual Studio代碼調試自定義應用程式。 它允許查看即時日誌、點擊斷點和逐步執行代碼，以及在每次激活時即時重新載入本地代碼更改。

這些步驟中的許多步驟通常由 `aio` 開箱後，請參見中的「調試應用程式」一節 [Adobe DeveloperApp Builder文檔](https://developer.adobe.com/app-builder/docs/getting_started/first_app)。 目前，以下步驟包括一種解決方法。

1. 安裝最新 [wskdebug](https://github.com/apache/openwhisk-wskdebug) 從GitHub和 [恩格羅](https://www.npmjs.com/package/ngrok)。

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. 添加到用戶設定JSON檔案。 它一直使用舊的VS代碼調試器，新的調試器 [一些問題](https://github.com/apache/openwhisk-wskdebug/issues/74) 使用wskdebug: `"debug.javascript.usePreview": false`。
1. 關閉通過 `aio app run`。
1. 使用 `aio app deploy`。
1. 僅使用執行Asset computeDevtool `aio asset-compute devtool`。 保持開啟。
1. 在VS代碼編輯器中，將以下調試配置添加到 `launch.json`:

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

   獲取 `ACTION NAME` 從 `aio app deploy`。

1. 選擇 `wskdebug worker` 從運行/調試配置中，然後按播放表徵圖。 等待它開始，直到它顯示 **[!UICONTROL 準備激活]** 的 **[!UICONTROL 調試控制台]** 的子菜單。

1. 按一下 **[!UICONTROL 運行]** 在Devtool上。 您可以看到在VS代碼編輯器中執行的操作，並且日誌開始顯示。

1. 在代碼中設定斷點，再次運行，應命中。

任何代碼更改都是即時載入的，並且在下次激活時立即生效。

>[!NOTE]
>
>自定義應用程式中的每個請求都有兩個激活。 第一個請求是在SDK代碼中非同步調用自身的Web操作。 第二個激活是命中代碼的激活。
