---
title: 導言 [!DNL Asset Compute Service]
description: '"[!DNL Asset Compute Service] 是一種雲本地資產處理服務，可降低複雜性並提高可擴充性。」'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '309'
ht-degree: 6%

---

# 概述 [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] 是可擴展的 [!DNL Adobe Experience Cloud] 處理數字資產。 它可以將影像、視頻、文檔和其他檔案格式轉換為不同的格式副本，包括縮略圖、提取的文本和元資料，以及存檔。

開發人員可以插入自定義資產應用程式（也稱為自定義工作程式）以解決自定義使用案例。 服務在 [!DNL Adobe I/O] 運行時。 它是可擴展的 [!DNL Adobe Developer App Builder] Node.js中寫入的無頭應用。 這些操作可以執行自定義操作，如調用外部API來執行映像操作或利用 [!DNL Adobe Sensei] 支援。

[!DNL Adobe Developer App Builder] 是一個框架，用於在 [!DNL Adobe I/O] 擴展Adobe Experience Cloud解決方案。 要建立自定義應用程式，開發人員可以 [!DNL React Spectrum] (Adobe的UI工具包)、建立微服務、建立自定義事件和協調API。 請參閱 [Adobe DeveloperApp Builder文檔](https://developer.adobe.com/app-builder/docs/overview/)。

>[!NOTE]
>
>當前， [!DNL Asset Compute Service] 只能通過 [!DNL Experience Manager] 作為 [!DNL Cloud Service]。 管理員建立可調用 [!DNL Asset Compute Service] 以傳遞資產以供處理。 請參閱 [使用資產微服務和處理配置檔案](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

## 支援的使用案例 [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] 支援基本影像處理等常見業務使用案例；Adobe應用程式特定轉換；和定制應用程式建立，以協調複雜的業務要求。

您可以使用 [!DNL Asset Compute] Web服務，為不同檔案類型生成縮略圖，為其提供高質量影像渲染 [支援的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)。 請參閱 [通過自定義配置支援的使用案例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

>[!NOTE]
>
>該服務不提供資產儲存。 用戶提供它，並提供對雲儲存中源檔案和格式副本檔案位置的引用。

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
>* [資產微服務資產處理概述 [!DNL Adobe Experience Manager] 作為 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html)。
>* [Adobe DeveloperApp Builder文檔](https://developer.adobe.com/app-builder/docs/overview)。
>* [支援處理的檔案格式](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/file-format-support.html)。


<!-- **TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
