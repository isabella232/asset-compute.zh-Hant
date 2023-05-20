---
title: 部署 [!DNL Asset Compute Service] 自定義應用程式
description: 部署 [!DNL Asset Compute Service] 自定義應用程式。
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 50f69e16772cee7f79a812f2b86f0ef0221db369
workflow-type: tm+mt
source-wordcount: '190'
ht-degree: 3%

---

# 部署自定義應用程式 {#deploy-custom-application}

要部署應用程式，請使用 [aio應用程式部署](https://github.com/adobe/aio-cli#aio-appdeploy) 的子菜單。 在終端中，該命令顯示訪問自定義應用程式的URL。 URL的格式為 `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`。

要在不重新部署應用程式的情況下獲取相同的URL，請使用 [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) 的子菜單。

在 [處理配置檔案 [!DNL Experience Manager] 作為 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) 將應用程式與 [!DNL Experience Manager] 作為 [!DNL Cloud Service]。

確保您的App Builder項目和工作區與 [!DNL Experience Manager] 作為 [!DNL Cloud Service] 要使用操作的環境。 它有不同的開發、準備和生產環境。 可以通過檢查 `AIO_runtime_*` 在您的Adobe DeveloperApp Builder應用程式根中的ENV檔案中定義的憑據。 例如，要部署到 `Stage` 工作區， `AIO_runtime_namespace` 是 `xxxxxx_xxxxxxxxx_stage`。 與 [!DNL Experience Manager] 作為 [!DNL Cloud Service] 生產環境，使用您的Adobe Developer應用程式生成器的應用程式URL `Production` 工作區。

>[!CAUTION]
>
>不在關鍵位置使用個人工作區 [!DNL Experience Manager] 環境。

>[!MORELIKETHIS]
>
>* [瞭解和管理 [!DNL Experience Manager] 作為 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html)。

