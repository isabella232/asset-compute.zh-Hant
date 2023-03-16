---
title: 簡介 [!DNL Asset Compute Service]
description: '"[!DNL Asset Compute Service] 是雲端原生資產處理服務，可降低複雜性並改善可擴充性。」'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '309'
ht-degree: 6%

---

# 概觀 [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] 是可擴充且可延伸的 [!DNL Adobe Experience Cloud] 處理數位資產。 它可以將影像、視訊、檔案和其他檔案格式轉換為不同的轉譯，包括縮圖、擷取的文字和中繼資料，以及封存。

開發人員可外掛自訂資產應用程式（也稱為自訂背景工作），以處理自訂使用案例。 服務適用於 [!DNL Adobe I/O] 執行階段。 它可延伸 [!DNL Adobe Developer App Builder] 在Node.js中寫入的無頭應用。 這些可以執行自訂操作，例如呼叫外部API以執行影像操作或運用 [!DNL Adobe Sensei] 支援。

[!DNL Adobe Developer App Builder] 是在上建立和部署自訂Web應用程式的架構 [!DNL Adobe I/O] 執行階段來擴充Adobe Experience Cloud解決方案。 要建立自定義應用程式，開發人員可以利用 [!DNL React Spectrum] (Adobe的UI工具套件)、建立微服務、建立自訂事件和協調API。 請參閱 [Adobe Developer App Builder檔案](https://developer.adobe.com/app-builder/docs/overview/).

>[!NOTE]
>
>目前， [!DNL Asset Compute Service] 只能透過 [!DNL Experience Manager] as a [!DNL Cloud Service]. 管理員建立可呼叫 [!DNL Asset Compute Service] 傳遞資產以進行處理。 請參閱 [使用資產微服務和處理設定檔](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

## 支援的使用案例 [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] 支援一些常見的業務使用案例，如基本影像處理；Adobe應用程式特定轉換；和定制應用程式的建立，以便協調複雜的業務需求。

您可以使用 [!DNL Asset Compute] 為不同檔案類型生成縮略圖的Web服務，為 [支援的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html). 請參閱 [透過自訂設定支援的使用案例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>服務未提供資產儲存空間。 使用者可提供該檔案，並提供雲端儲存空間中來源和轉譯檔案位置的參考。

<!-- TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [資產微服務的資產處理概觀 [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Adobe Developer App Builder檔案](https://developer.adobe.com/app-builder/docs/overview).
>* [支援處理的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html).


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
