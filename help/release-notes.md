---
title: ' [!DNL Asset Compute Service]的發行說明'
description: ' [!DNL Asset Compute Service]中的新功能、增強功能和已知問題。'
exl-id: b348fa8f-0cd6-4ca1-bfe3-f31e8d6583f0
source-git-commit: 187a788d036f33b361a0fd1ca34a854daeb4a101
workflow-type: tm+mt
source-wordcount: '191'
ht-degree: 0%

---

# [!DNL Asset Compute Service] {#release-notes}發行說明

[!DNL Asset Compute Service]的最新版本將於2020年7月30日發行。

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## 新增功能{#what-is-new}

這是[!DNL Asset Compute Service]的第一個版本。 這是[!DNL Adobe Experience Cloud]的可擴充服務，用於處理數位資產。 它可以將影像、視訊、檔案和其他檔案格式轉換為不同的轉譯，包括縮圖、擷取的文字和中繼資料，以及封存。

目前，[!DNL Asset Compute Service]只能在[!DNL Experience Manager]中作為[!DNL Cloud Service]使用。

## 限制和已知問題{#known-limitations}

若要使用[開發人員工具](https://github.com/adobe/asset-compute-devtool)測試您的自訂應用程式，您需要存取[雲端儲存容器](https://github.com/adobe/asset-compute-devtool#prerequisites)。

* 只有開發人員工具才需要雲端儲存空間（與[!DNL Experience Manager] blob存放區不同）存取。 您仍然可以建立、測試和部署自定義應用程式，無需開發人員工具。
* 它可以是共用容器，供不同專案的多位開發人員使用。

## 貢獻{#contribute-open-source}

[!DNL Asset Compute Service] 擴充性是在github.com/adobe上以開放開發模 [式開發，](https://github.com/adobe) 歡迎擴充功能開發人員的貢獻。與開發、建立、測試和部署自定義應用程式相關的所有元件都是開源的。 請參閱[如何及何處貢獻計算服務](contribute-to-compute-service.md)。

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
