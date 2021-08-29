---
title: 疑難排解 [!DNL Asset Compute Service]
description: 使用 [!DNL Asset Compute Service]疑難排解自訂應用程式並除錯。
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '285'
ht-degree: 1%

---

# 疑難排解 {#troubleshoot}

一些可協助您疑難排解Asset compute服務的一般疑難排解提示如下：

* 請確定JavaScript應用程式在啟動時不會當機。 這類當機通常與遺失的程式庫或相依性有關。
* 確保應用程式的`package.json`檔案中引用了要安裝的所有依賴項。
* 確保故障時清除可能產生的任何錯誤不會產生隱藏原始問題的錯誤。

* 首次使用新的[!DNL Asset Compute Service]整合啟動開發人員工具時，如果Asset compute事件日誌未完全設定，它可能會失敗第一個處理請求。 等待日誌設定一段時間，然後再發送另一個請求。
* 如果傳送Asset compute`/register`或`/process`要求時發生錯誤，請確定所有必要的API已新增至[!DNL Adobe I/O]專案和工作區，即Asset compute、[!DNL Adobe I/O]事件、[!DNL Adobe I/O]事件管理和[!DNL Adobe I/O]執行階段。

## 通過[!DNL Adobe I/O] CLI登錄問題 {#login-via-aio-cli}

如果通過 [!DNL Adobe I/O] CLI](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#3-signing-in-from-cli)登錄[!DNL Adobe Developer Console] [時遇到問題，請手動添加開發、測試和部署自定義應用程式所需的憑據：

1. 導覽至您的Firefly專案和工作區，位於[Adobe開發人員控制台](https://console.adobe.io/)，然後從右上角按&#x200B;**[!UICONTROL 下載]**。 開啟此檔案，並將此JSON儲存至電腦上的安全位置。

1. 導覽至Firefly應用程式中的ENV檔案。

1. 添加[!DNL Adobe I/O]運行時憑據。 從下載的JSON取得[!DNL Adobe I/O] A Runtime憑證。 憑據在`project.workspace.services.runtime`下。 在`AIO_runtime_XXX`變數中新增[!DNL Adobe I/O]執行階段憑證：

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 在步驟1中，將絕對路徑新增至下載的JSON:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 設定開發人員工具所需的其餘[必要憑證](develop-custom-application.md)。

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
