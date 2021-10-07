---
title: ' [!DNL Asset Compute Service]簡介'
description: '[!DNL Asset Compute Service] 是雲端原生資產處理服務，可降低複雜性並改善可擴充性。'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: a2460a0719f8c585ed72e44c904aa0df33301032
workflow-type: tm+mt
source-wordcount: '307'
ht-degree: 0%

---

# [!DNL Asset Compute Service]概述 {#overview}

[!DNL Asset Compute Service] 是可擴充的數位資產處 [!DNL Adobe Experience Cloud] 理服務。它可以將影像、視訊、檔案和其他檔案格式轉換為不同的轉譯，包括縮圖、擷取的文字和中繼資料，以及封存。

開發人員可外掛自訂資產應用程式（也稱為自訂背景工作），以處理自訂使用案例。 該服務在[!DNL Adobe I/O]運行時運行。 可透過在Node.js中撰寫的[!DNL Project Firefly]無標題應用程式延伸。 這些功能可執行自訂操作，例如呼叫外部API以執行影像操作或運用[!DNL Adobe Sensei]支援。

[!DNL Project Firefly] 是在執行階段上建置和部署自訂Web應用程式的架 [!DNL Adobe I/O] 構，以擴充Adobe Experience Cloud解決方案。若要建立自訂應用程式，開發人員可以運用[!DNL React Spectrum](Adobe的UI工具包)、建立微服務、建立自訂事件和協調API。 請參閱Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)的[檔案。

>[!NOTE]
>
>目前，[!DNL Asset Compute Service]只能透過[!DNL Experience Manager]作為[!DNL Cloud Service]使用。 管理員建立處理設定檔，可呼叫[!DNL Asset Compute Service]以傳遞資產以進行處理。 請參閱[使用資產微服務和處理設定檔](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

## [!DNL Asset Compute Service]支援的使用案例 {#possible-use-cases-benefits}

[!DNL Asset Compute Service] 支援一些常見的業務使用案例，如基本影像處理；Adobe應用程式特定轉換；和定制應用程式的建立，以便協調複雜的業務需求。

您可以使用[!DNL Asset Compute] Web服務為不同檔案類型生成縮略圖，為[支援的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)生成高質量的影像呈現。 請參閱透過自訂設定支援的[使用案例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

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
>* [資產微服務的資產處理概 [!DNL Adobe Experience Manager] 述a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Project Firefly的檔案](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)。
>* [支援處理的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)。


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
