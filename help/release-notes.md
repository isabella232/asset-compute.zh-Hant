---
title: ' [!DNL Asset Compute Service]的發行說明'
description: ' [!DNL Asset Compute Service]中的新功能、增強功能和已知問題。'
translation-type: tm+mt
source-git-commit: 95e384d2a298b3237d4f93673161272744e7f44a
workflow-type: tm+mt
source-wordcount: '191'
ht-degree: 0%

---


# 發行說明[!DNL Asset Compute Service] {#release-notes}

[!DNL Asset Compute Service]的最新版本將於2020年7月30日發行。

<!--

To test your custom applications with the [developer tool](https://github.com/adobe/asset-compute-devtool), you need access to a [cloud storage container](https://github.com/adobe/asset-compute-devtool#prerequisites). Currently, Adobe supports Azure Blob Storage and AWS S3.

>[!NOTE]
>
>Cloud storage access is only required for using the developer tool. You can still create, test and deploy custom applications with out using the developer tool.
-->

## {#what-is-new}的新增功能

這是[!DNL Asset Compute Service]的第一個版本。 它是[!DNL Adobe Experience Cloud]的可擴充且可擴充的服務，可處理數位資產。 它可將影像、視訊、檔案和其他檔案格式轉換為不同的轉譯，包括縮圖、擷取的文字和中繼資料，以及封存。

目前，[!DNL Asset Compute Service]只能用在[!DNL Experience Manager]中作為[!DNL Cloud Service]。

## 限制和已知問題{#known-limitations}

若要使用[開發人員工具](https://github.com/adobe/asset-compute-devtool)測試您的自訂應用程式，您需要存取[雲端儲存容器](https://github.com/adobe/asset-compute-devtool#prerequisites)。

* 只有開發人員工具才需要雲端儲存空間（與[!DNL Experience Manager]blob儲存空間不同）存取權。 您仍然可以建立、測試和部署自訂應用程式，毋需使用開發人員工具。
* 它可以是多個開發人員在不同專案間使用的共用容器。

## Contribute {#contribute-open-source}

[!DNL Asset Compute Service] 擴充性是在 [github.com/adobe的開放開發模型下開發，](https://github.com/adobe) 歡迎擴充開發人員的貢獻。所有與開發、建立、測試和部署自訂應用程式相關的元件都是開放原始碼。 請參閱[如何和何處為Compute Service貢獻](contribute-to-compute-service.md)。

<!-- **TBD:**
* Are we versioning the releases?
* Is there any compatibility information to be added? With Project Firefly versions, or AEMaaCS releases, or other offerings/integrations such as InDesign Server?
-->
