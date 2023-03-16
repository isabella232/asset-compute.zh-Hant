---
title: 了解如何擴充 [!DNL Asset Compute Service]
description: 何時及如何擴充 [!DNL Asset Compute Service] 功能執行自訂資產處理。
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '260'
ht-degree: 5%

---

# 擴充性簡介 {#introduction-to-extensibilty}

許多轉譯需求（例如轉換為格式和調整影像大小）都由以下方面提供： [在中處理設定檔 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html). 更複雜的業務需求可能需要定製的解決方案來滿足組織的需求。 [!DNL Asset Compute Service] 可借由建立自訂應用程式來擴充，這些應用程式是從 [!DNL Experience Manager]. 這些自訂應用程式適用於 [支援的使用案例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html).

>[!NOTE]
>
>[!DNL Asset Compute Service] 僅適用於 [!DNL Experience Manager] as a [!DNL Cloud Service].

自定義應用程式無頭 [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) 應用程式。 延伸 [!DNL Asset Compute Service] 使用自訂應用程式，可透過 [asset computeSDK](https://github.com/adobe/asset-compute-sdk) 和Adobe Developer App Builder開發人員工具。 這可讓開發人員專注於業務邏輯。 建立自定義應用程式與建立無伺服器一樣簡單 [!DNL Adobe I/O] 運行時操作。 它是單一Node.js JavaScript函式。 此 [基本自訂應用程式範例](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 說明。

## 先決條件和布建需求 {#prerequisites-and-provisioning}

確定您符合下列必要條件：

* Adobe Developer App Builder工具會安裝在您的電腦上。
* 安 [!DNL Experience Cloud] 組織。 更多資訊 [此處](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* 體驗組織必須具備 [!DNL Experience Manager] as a [!DNL Cloud Service] 已啟用。
* [!DNL Adobe Experience Cloud] 組織是 [!DNL Adobe Developer App Builder] 開發人員預覽程式。 請參閱 [如何申請訪問](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* 確保開發人員在組織中的開發人員角色或管理員權限。
* 確保 [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) 已安裝在本機。

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
