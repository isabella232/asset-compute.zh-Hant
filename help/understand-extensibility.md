---
title: 了解如何擴展 [!DNL Asset Compute Service]
description: 何時以及如何擴充 [!DNL Asset Compute Service] 功能以進行自訂資產處理。
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '254'
ht-degree: 0%

---

# 擴充性簡介 {#introduction-to-extensibilty}

 [!DNL Experience Manager] 中的「處理設定檔」(a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html))解決了許多轉譯需求，例如轉換為格式和調整影像大小。 [更複雜的業務需求可能需要定製的解決方案來滿足組織的需求。 [!DNL Asset Compute Service] 可借由建立自訂應用程式來擴充，這些應用程式是從中的處理設定檔中 [!DNL Experience Manager]呼叫。這些自訂應用程式適用於[支援的使用案例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

>[!NOTE]
>
>[!DNL Asset Compute Service] 只能與as  [!DNL Experience Manager] a搭配使 [!DNL Cloud Service]用

自訂應用程式為無頭[Project Firefly](https://github.com/AdobeDocs/project-firefly)應用程式。 透過[Asset computeSDK](https://github.com/adobe/asset-compute-sdk)和Project Firefly開發人員工具，使用自訂應用程式擴充[!DNL Asset Compute Service]變得簡單。 這可讓開發人員專注於業務邏輯。 建立自定義應用程式與建立無伺服器的普通[!DNL Adobe I/O]運行時操作一樣簡單。 它是單一Node.js JavaScript函式。 [基本自定義應用程式示例](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js)對此進行了說明。

## 先決條件和布建需求 {#prerequisites-and-provisioning}

確定您符合下列必要條件：

* 您的電腦上安裝了Project Firefly工具。
* [!DNL Experience Cloud]組織。 更多資訊[此處](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials)。
* 體驗組織必須已啟用[!DNL Experience Manager]作為[!DNL Cloud Service]。
* [!DNL Adobe Experience Cloud] 組織是開發人員預 [!DNL Project Firefly] 覽計畫的一部分。請參閱[如何應用訪問](https://www.adobe.io/project-firefly/docs/overview/getting_access/)。
* 確保開發人員在組織中的開發人員角色或管理員權限。
* 確保本地安裝[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli)。

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
