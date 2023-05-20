---
title: 瞭解擴展 [!DNL Asset Compute Service]
description: 何時以及如何擴展 [!DNL Asset Compute Service] 功能執行自定義資產處理。
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '260'
ht-degree: 5%

---

# 擴展性簡介 {#introduction-to-extensibilty}

許多格式副本要求（如轉換為格式和調整影像大小）由 [處理配置檔案 [!DNL Experience Manager] 作為 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/asset-microservices-overview.html)。 更複雜的業務需求可能需要定製的解決方案來滿足組織的需求。 [!DNL Asset Compute Service] 可通過建立從中的處理配置檔案調用的自定義應用程式來擴展 [!DNL Experience Manager]。 這些定制應用程式 [支援的使用案例](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)。

>[!NOTE]
>
>[!DNL Asset Compute Service] 僅供使用 [!DNL Experience Manager] 作為 [!DNL Cloud Service]。

定制應用程式是無頭的 [Adobe Developer應用程式生成器](https://github.com/AdobeDocs/app-builder) 。 擴展 [!DNL Asset Compute Service] 使自定義應用程式通過 [asset computeSDK](https://github.com/adobe/asset-compute-sdk) 和Adobe DeveloperApp Builder開發工具。 這使開發人員能夠專注於業務邏輯。 建立自定義應用程式與建立純無伺服器一樣簡單 [!DNL Adobe I/O] 運行時操作。 它是單個Node.js JavaScript函式。 的 [基本自定義應用程式示例](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) 說明。

## 先決條件和設定要求 {#prerequisites-and-provisioning}

確保滿足以下先決條件：

* Adobe Developer應用程式生成器工具已安裝在您的電腦上。
* 安 [!DNL Experience Cloud] 組織。 詳細資訊 [這裡](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials)。
* 體驗組織必須 [!DNL Experience Manager] 作為 [!DNL Cloud Service] 啟用。
* [!DNL Adobe Experience Cloud] 組織是 [!DNL Adobe Developer App Builder] 開發人員預覽程式。 請參閱 [如何申請訪問](https://developer.adobe.com/app-builder/docs/overview/getting_access)。
* 確保開發人員在組織中具有開發人員角色或管理員權限。
* 確保 [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) 本地安裝。

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
