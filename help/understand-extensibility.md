---
title: 瞭解擴展 [!DNL Asset Compute Service]
description: 何時及如何擴充 [!DNL Asset Compute Service] 功能以執行自訂資產處理。
translation-type: tm+mt
source-git-commit: 95e384d2a298b3237d4f93673161272744e7f44a
workflow-type: tm+mt
source-wordcount: '259'
ht-degree: 0%

---


# 擴充性簡介{#introduction-to-extensibilty}

許多轉譯需求（例如轉換為格式和調整影像大小）都由 [!DNL Experience Manager] 中的「處理描述檔」(Processing Profiles)作為a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html)來解決。 [更複雜的業務需求可能需要定製的解決方案，以滿足組織的需求。 [!DNL Asset Compute Service] 可以通過在中建立從處理配置檔案調用的自定義應用程式來擴展 [!DNL Experience Manager]。這些自訂應用程式符合[支援的使用案例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

>[!NOTE]
>
>[!DNL Asset Compute Service] 只能與 [!DNL Experience Manager] as  [!DNL Cloud Service]

自訂應用程式是無頭[專案Firefly](https://github.com/AdobeDocs/project-firefly)應用程式。 透過[資產計算SDK](https://github.com/adobe/asset-compute-sdk)和Project Firefly開發人員工具，輕鬆擴充使用自訂應用程式的[!DNL Asset Compute Service]。 這可讓開發人員專注於商業邏輯。 建立自訂應用程式就像建立簡單的無伺服器[!DNL Adobe I/O]執行階段動作一樣簡單。 它是單一Node.js JavaScript函式。 [基本自定義應用程式示例](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js)說明了它。

## 先決條件和設定要求{#prerequisites-and-provisioning}

請確定您符合下列必要條件：

* Project Firefly工具會安裝在您的電腦上。
* [!DNL Experience Cloud]組織。 詳細資訊[此處](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials)。
* 體驗組織必須啟用[!DNL Experience Manager]作為[!DNL Cloud Service]。
* [!DNL Adobe Experience Cloud] 組織是開發人員預 [!DNL Project Firefly] 覽計畫的一部分。請參閱[如何應用訪問](https://github.com/AdobeDocs/project-firefly/blob/master/overview/getting_access.md)。
* 確保開發人員在組織中擁有開發人員角色或管理員權限。
* 確保[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli)已安裝在本地。

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
