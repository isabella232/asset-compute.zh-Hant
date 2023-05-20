---
title: 疑難排解 [!DNL Asset Compute Service]
description: 使用 [!DNL Asset Compute Service]。
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '291'
ht-degree: 1%

---

# 疑難排解 {#troubleshoot}

一些可能有助於您使用Asset compute服務進行故障排除的一般故障排除提示包括：

* 確保JavaScript應用程式在啟動時不會崩潰。 此類崩潰通常與缺少的庫或依賴項相關。
* 確保應用程式中引用了要安裝的所有依賴關係 `package.json` 的子菜單。
* 確保在故障時清除可能產生的任何錯誤不會生成隱藏原始問題的錯誤。

* 首次使用新工具啟動開發人員工具時 [!DNL Asset Compute Service] 整合時，如果Asset compute事件日記帳未完全設定，則可能會失敗第一個處理請求。 請等待一段時間以設定日記帳，然後再發送另一個請求。
* 如果在發送Asset compute時遇到錯誤 `/register` 或 `/process` 請求，確保將所有必需的API添加到 [!DNL Adobe I/O] 項目和工作區 — 即Asset compute, [!DNL Adobe I/O] 事件， [!DNL Adobe I/O] 事件管理和 [!DNL Adobe I/O] 運行時。

## 通過 [!DNL Adobe I/O] CLI {#login-via-aio-cli}

如果登錄時遇到問題 [!DNL Adobe Developer Console] [通過 [!DNL Adobe I/O] CLI](https://developer.adobe.com/app-builder/docs/getting_started/first_app/#3-signing-in-from-cli)，然後手動添加開發、測試和部署自定義應用程式所需的憑據：

1. 導航至上的Adobe DeveloperApp Builder項目和工作區 [Adobe Developer控制台](https://console.adobe.io/) 按 **[!UICONTROL 下載]** 從右上角。 開啟此檔案，並將此JSON保存到電腦上的安全位置。

1. 導航到您的Adobe Developer應用程式生成器應用程式中的ENV檔案。

1. 添加 [!DNL Adobe I/O] 運行時憑據。 獲取 [!DNL Adobe I/O] 從下載的JSON獲取的運行時憑據。 憑據在 `project.workspace.services.runtime`。 添加 [!DNL Adobe I/O] 運行時憑據 `AIO_runtime_XXX` 變數：

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. 在步驟1中將絕對路徑添加到下載的JSON:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. 設定其餘 [必需的憑據](develop-custom-application.md) 開發工具所需的。

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
