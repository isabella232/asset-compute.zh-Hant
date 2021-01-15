---
title: ' [!DNL Asset Compute Service]簡介。'
description: '[!DNL Asset Compute Service] 是雲端原生資產處理服務，可降低複雜性並改善可擴充性。'
translation-type: tm+mt
source-git-commit: d26ae470507e187249a472ececf5f08d803a636c
workflow-type: tm+mt
source-wordcount: '314'
ht-degree: 0%

---


# [!DNL Asset Compute Service] {#overview}概觀

[!DNL Asset Compute Service] 是處理數位資產的可擴充 [!DNL Adobe Experience Cloud] 且可擴充的服務。它可將影像、視訊、檔案和其他檔案格式轉換為不同的轉譯，包括縮圖、擷取的文字和中繼資料，以及封存。

開發人員可以外掛自訂資產應用程式（也稱為自訂工作者），以處理自訂使用案例。 服務在[!DNL Adobe I/O]運行時運行。 它可透過以Node.js編寫的[!DNL Project Firefly]無頭應用程式延伸。 這些作業可以執行自訂作業，例如呼叫外部API以執行影像作業或運用[!DNL Adobe Sensei]支援。

[!DNL Project Firefly] 是在執行時期上建立和部署自訂Web應用程式 [!DNL Adobe I/O] 以擴充Adobe Experience Cloud解決方案的架構。若要建立自訂應用程式，開發人員可運用[!DNL React Spectrum]（Adobe的UI工具套件）、建立微型服務、建立自訂事件和協調API。 請參閱[Project Firefly](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)的檔案。

>[!NOTE]
>
>目前，[!DNL Asset Compute Service]只能透過[!DNL Experience Manager]使用為[!DNL Cloud Service]。 管理員會建立可呼叫[!DNL Asset Compute Service]的處理設定檔，以傳遞資產以供處理。 請參閱[使用資產微服務和處理配置檔案](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

## [!DNL Asset Compute Service] {#possible-use-cases-benefits}支援的使用案例

[!DNL Asset Compute Service] 支援一些常見的業務使用案例，如基本影像處理；Adobe應用程式特定轉換；以及自訂應用程式的建立，以協調複雜的業務需求。

您可以使用[!DNL Asset Compute] web service來產生不同檔案類型的縮圖，並針對[支援的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)產生高品質的影像轉譯。 請參閱透過自訂設定](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)支援的使用案例。[

>[!NOTE]
>
>服務不提供資產儲存空間。 使用者會提供它，並提供雲端儲存空間中來源和轉譯檔案位置的參考。

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
>* [資產微服務資產處理概 [!DNL Adobe Experience Manager] 述a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html).
>* [Project Firefly的檔案](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html)。
>* [支援處理的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)。
>* [資產計算服務發行說明](release-notes.md)


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
