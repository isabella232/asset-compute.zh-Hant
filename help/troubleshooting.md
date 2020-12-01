---
title: 疑難排解 [!DNL Asset Compute Service].
description: 使用 [!DNL Asset Compute Service]疑難排解自訂應用程式並除錯。
translation-type: tm+mt
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: tm+mt
source-wordcount: '306'
ht-degree: 1%

---


# 疑難排解 {#troubleshoot}

以下是一些有助於您使用資產計算服務進行故障排除的一般故障排除提示：

* 確保JavaScript應用程式在啟動時不會當機。 此類當機通常與遺失的程式庫或相依性有關。
* 確保應用程式的`package.json`檔案中引用了要安裝的所有相關性。
* 確保因故障而清除的任何錯誤不會生成隱藏原始問題的錯誤。

* 當首次使用新的[!DNL Asset Compute Service]整合啟動開發人員工具時，由於可能未完整設定「資產計算事件日誌」，因此第一個處理請求可能會失敗。 請等待一段時間，以便日誌設定，然後再發送另一個請求。
* 如果您在傳送資產計算`/register`或`/process`請求時發生錯誤，請確定所有必要的API都新增至Adobe I/O專案和工作區，即資產計算、IO事件、IO事件管理和執行階段。

## 透過Adobe I/O CLI登入問題{#login-via-aio-cli}

如果您通過Adobe I/O CLI](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#3-signing-in-from-cli)登入[!DNL Adobe Developer Console] [時遇到問題，請手動新增開發、測試和部署自訂應用程式所需的憑證：

1. 導覽至[Adobe Developer Console](https://console.adobe.io/)上的Firefly專案和工作區，然後從右上角按&#x200B;**[!UICONTROL Download]**。 開啟此檔案，並將此JSON儲存至您機器上的安全位置。

1. 導覽至Firefly應用程式中的ENV檔案。

1. 新增Adobe I/O Runtime認證。 從下載的JSON取得Adobe I/O Runtime認證。 憑據位於`project.workspace.services.runtime`下。 在`AIO_runtime_XXX`變數中新增I/O Runtime憑證：

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 在步驟1中新增已下載JSON的絕對路徑：

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 設定開發人員工具所需的[其餘憑證](develop-custom-application.md)。

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
