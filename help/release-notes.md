---
title: 發行說明 [!DNL Asset Compute Service]。
description: 中的新功能、增強功能和已知問題 [!DNL Asset Compute Service]。
translation-type: tm+mt
source-git-commit: 68d910cd092fccb599c361f24daff80460129e1c
workflow-type: tm+mt
source-wordcount: '193'
ht-degree: 0%

---


# 發行說明 [!DNL Asset Compute Service] {#release-notes}

最新版本 [!DNL Asset Compute Service] 將於2020年7月30日發行。

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## What is new {#what-is-new}

這是第一版 [!DNL Asset Compute Service]。 它是處理數位資產的可擴充 [!DNL Adobe Experience Cloud] 且可擴充的服務。 它可將影像、視訊、檔案和其他檔案格式轉換為不同的轉譯，包括縮圖、擷取的文字和中繼資料，以及封存。

目前， [!DNL Asset Compute Service] 只能用作雲 [!DNL Experience Manager] 端服務。

## 限制和已知問題 {#known-limitations}

若要使用開發人員工具來測試您 [的自訂應用程式](https://github.com/adobe/asset-compute-devtool)，您需要存取雲 [端儲存容器](https://github.com/adobe/asset-compute-devtool#prerequisites)。

* 只有開發人員工具才需要雲端 [!DNL Experience Manager] 儲存空間（不同於點滴儲存空間）存取權。 您仍然可以建立、測試和部署自訂應用程式，毋需使用開發人員工具。
* 它可以是多個開發人員在不同專案間使用的共用容器。

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] 擴充性是在github.com/adobe的開放開發模型下開 [發](https://github.com/adobe) ，歡迎擴充開發人員的貢獻。 所有與開發、建立、測試和部署自訂應用程式相關的元件都是開放原始碼。 瞭解 [如何和何處為計算服務做出貢獻](contribute-to-compute-service.md)。

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
