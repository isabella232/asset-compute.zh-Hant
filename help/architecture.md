---
title: ' [!DNL Asset Compute Service]的架構'
description: 如何 [!DNL Asset Compute Service] API、應用程式和SDK搭配運作，以提供雲端原生資產處理服務。
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: 187a788d036f33b361a0fd1ca34a854daeb4a101
workflow-type: tm+mt
source-wordcount: '485'
ht-degree: 0%

---

# [!DNL Asset Compute Service]的體系結構 {#overview}

[!DNL Asset Compute Service]建置在無伺服器[!DNL Adobe I/O] Runtime平台之上。 為資產提供Adobe Sensei內容服務支援。 為調用的客戶端（僅支援[!DNL Experience Manager]作為[!DNL Cloud Service]）提供其為資產所搜索的Adobe Sensei生成的資訊。 傳回的資訊為JSON格式。

[!DNL Asset Compute Service] 可依據建立自訂應用程式來擴充 [!DNL Project Firefly]。這些自訂應用程式為[!DNL Project Firefly]無標題應用程式，執行新增自訂轉換工具或呼叫外部API等工作以執行影像操作。

[!DNL Project Firefly] 是在執行階段建立和部署自訂Web應用程式的 [!DNL Adobe I/O] 架構。若要建立自訂應用程式，開發人員可以運用[!DNL React Spectrum](Adobe的UI工具包)、建立微服務、建立自訂事件和協調API。 請參閱Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)的[檔案。

架構的基礎包括：

* 應用程式的模組化 — 僅包含給定任務所需的內容 — 允許應用程式彼此分離，並保持輕量。

* [!DNL Adobe I/O]運行時的無伺服器概念提供了許多好處：非同步、高度可擴充、隔離、以工作為基礎的處理，非常適合於資產處理。

* 二進位雲端儲存空間提供必要功能，可個別儲存和存取資產檔案與轉譯，而無須透過預先簽署的URL參考取得儲存空間的完整存取權限。 傳輸加速、CDN快取，以及將計算應用程式與雲端儲存空間結合，以提供最佳的低延遲內容存取。 AWS和Azure雲均受支援。

![asset compute服務架構](assets/architecture-diagram.png)

*圖：體系結構 [!DNL Asset Compute Service] 及其與、儲存和處理應 [!DNL Experience Manager]用程式整合的方式。*

此架構包含下列部分：

* **API和協調層** 接收指示服務將來源資產轉換為多個轉譯的請求（JSON格式）。請求為非同步，並以啟用ID（即工作ID）傳回。 說明純粹是宣告性的，對於所有標準處理工作（例如縮圖產生、文字擷取），使用者只需指定所需的結果，而不是處理特定轉譯的應用程式。 一般API功能（例如驗證、分析、速率限制）是使用服務前的AdobeAPI閘道處理，並管理前往[!DNL Adobe I/O]執行階段的所有請求。 應用路由由協調層動態完成。 用戶端可為特定轉譯指定自訂應用程式，並包含自訂參數。 應用程式執行可以完全並行化，因為它們是[!DNL Adobe I/O]運行時中獨立的無伺服器函式。

* **處理專門處** 理特定類型檔案格式或目標轉譯的資產的應用程式。從概念上講，應用程式與Unix管道概念類似：輸入檔案被轉換為一個或多個輸出檔案。

* **常見 [應用程](https://github.com/adobe/asset-compute-sdk)** 式庫可處理常見工作，例如下載來源檔案、上傳轉譯、錯誤報告、事件傳送和監控。這樣，開發應用程式就會盡可能簡單，遵循無伺服器的思想，並且可以限制於本地檔案系統交互。

<!-- TBD:

* About the YAML file?
* See [https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/first_app.md#5-anatomy-of-a-project-firefly-application).

* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
