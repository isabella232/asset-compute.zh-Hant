---
title: 建築 [!DNL Asset Compute Service]
description: 如何 [!DNL Asset Compute Service] API、應用程式和SDK協同工作，提供雲本地資產處理服務。
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: 0c5ab8ab230e3f033b804849a2c005351542155f
workflow-type: tm+mt
source-wordcount: '486'
ht-degree: 0%

---

# 建築 [!DNL Asset Compute Service] {#overview}

的 [!DNL Asset Compute Service] 建在無伺服器上 [!DNL Adobe I/O] 運行時平台。 它為資產提供Adobe Sensei內容服務支援。 調用客戶端(僅 [!DNL Experience Manager] 作為 [!DNL Cloud Service] （支援）獲得其為資產所尋求的Adobe Sensei生成的資訊。 返回的資訊為JSON格式。

[!DNL Asset Compute Service] 可擴展，方法是： [!DNL Project Adobe Developer App Builder]。 這些自定義應用程式 [!DNL Project Adobe Developer App Builder] 無頭應用和執行任務，如添加自定義轉換工具或調用外部API來執行映像操作。

[!DNL Project Adobe Developer App Builder] 是一個框架，用於在 [!DNL Adobe I/O] 運行時。 要建立自定義應用程式，開發人員可以 [!DNL React Spectrum] (Adobe的UI工具包)、建立微服務、建立自定義事件和協調API。 請參閱 [Adobe DeveloperApp Builder文檔](https://developer.adobe.com/app-builder/docs/overview)。

該體系結構的基礎包括：

* 應用程式的模組化 — 僅包含給定任務所需的內容 — 允許將應用程式彼此分離，並保持輕量級。

* 無伺服器概念 [!DNL Adobe I/O] 運行時可帶來許多好處：非同步、高度可擴展、隔離、基於作業的處理，非常適合於資產處理。

* 二進位雲儲存提供了儲存和單獨訪問資產檔案和格式副本所必需的功能，而無需使用預簽名的URL引用對儲存進行完全訪問權限。 傳輸加速、CDN快取以及將計算應用程式與雲儲存共址，可實現最佳的低延遲內容訪問。 支援AWS和Azure雲。

![asset compute服務體系結構](assets/architecture-diagram.png)

*圖：建築 [!DNL Asset Compute Service] 以及它與 [!DNL Experience Manager]、儲存和處理應用程式。*

該體系結構由以下部分組成：

* **API和業務流程層** 接收指示服務將源資產轉換為多個格式副本的請求（JSON格式）。 請求是非同步的，並返回激活ID，即作業ID。 說明純是聲明性的，對於所有標準處理工作（如縮略圖生成、文本提取），使用者只指定所需的結果，而不指定處理某些格式副本的應用程式。 通用API功能(如Adobe、分析、速率限制)在服務前使用API網關處理，並管理所有將發往 [!DNL Adobe I/O] 運行時。 應用程式路由由業務流程層動態完成。 客戶端可以為特定格式副本指定自定義應用程式，並包括自定義參數。 應用程式執行可以完全並行化，因為它們是中獨立的無伺服器函式 [!DNL Adobe I/O] 運行時。

* **處理資產的應用程式** 專門針對某些類型的檔案格式或目標格式副本的格式副本。 從概念上講，應用程式與Unix管道概念類似：將輸入檔案轉換為一個或多個輸出檔案。

* **A [通用應用程式庫](https://github.com/adobe/asset-compute-sdk)** 處理常見任務，如下載源檔案、上載格式副本、錯誤報告、事件發送和監視。 這樣設計，開發應用程式就會盡可能簡單，遵循無伺服器的思想，並且可以限制到本地檔案系統交互。

<!-- TBD:

* About the YAML file?
* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
