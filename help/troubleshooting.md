---
title: 疑難排解 [!DNL Asset Compute Service]
description: 使用 [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '291'
ht-degree: 1%

---

# 疑難排解 {#troubleshoot}

一些可協助您疑難排解Asset compute服務的一般疑難排解提示如下：

* 請確定JavaScript應用程式在啟動時不會當機。 這類當機通常與遺失的程式庫或相依性有關。
* 確保應用程式的 `package.json` 檔案。
* 確保故障時清除可能產生的任何錯誤不會產生隱藏原始問題的錯誤。

* 首次使用新 [!DNL Asset Compute Service] 整合，如果未完整設定「Asset compute事件日記帳」，則可能會失敗第一個處理請求。 等待日誌設定一段時間，然後再發送另一個請求。
* 如果您遇到傳送Asset compute的錯誤 `/register` 或 `/process` 請求，請確定所有必要的API已新增至 [!DNL Adobe I/O] 專案和工作區 — 即Asset compute、 [!DNL Adobe I/O] 事件、 [!DNL Adobe I/O] 事件管理，以及 [!DNL Adobe I/O] 執行階段。

## 透過 [!DNL Adobe I/O] CLI {#login-via-aio-cli}

如果您在登入 [!DNL Adobe Developer Console] [通過 [!DNL Adobe I/O] CLI](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)，然後手動新增開發、測試和部署自訂應用程式所需的憑證：

1. 導覽至您的Adobe Developer App Builder專案和工作區，位於 [Adobe Developer Console](https://console.adobe.io/) 按 **[!UICONTROL 下載]** 從右上角。 開啟此檔案，並將此JSON儲存至電腦上的安全位置。

1. 導覽至Adobe Developer App Builder應用程式中的ENV檔案。

1. 新增 [!DNL Adobe I/O] 運行時憑據。 取得 [!DNL Adobe I/O] 從下載的JSON取得執行階段憑證。 憑證位於 `project.workspace.services.runtime`. 新增 [!DNL Adobe I/O] 運行時憑據 `AIO_runtime_XXX` 變數：

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 在步驟1中，將絕對路徑新增至下載的JSON:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 設定 [必要憑證](develop-custom-application.md) 開發人員工具所需。

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
