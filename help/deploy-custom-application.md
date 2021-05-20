---
title: 部署 [!DNL Asset Compute Service] 自定義應用程式
description: 部署 [!DNL Asset Compute Service] 自定義應用程式。
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 187a788d036f33b361a0fd1ca34a854daeb4a101
workflow-type: tm+mt
source-wordcount: '183'
ht-degree: 0%

---

# 部署自定義應用程式{#deploy-custom-application}

要部署應用程式，請使用[aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy)命令。 在終端機中，命令會顯示存取自訂應用程式的URL。 URL的格式為`https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`。

若要在不重新部署應用程式的情況下取得相同的URL，請使用[`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action)命令。

在 [!DNL Experience Manager] 中以 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)形式使用[處理配置檔案中的URL，將應用程式與[!DNL Experience Manager]作為[!DNL Cloud Service]形式整合。

請確定您的Firefly專案和工作區對應至您要使用動作的[!DNL Experience Manager]環境。 [!DNL Cloud Service]其開發、測試和生產環境不同。 您可以檢查`AIO_runtime_*`認證，這些認證定義於Firefly應用程式根目錄的ENV檔案中。 例如，要部署到`Stage`工作區，`AIO_runtime_namespace`的格式為`xxxxxx_xxxxxxxxx_stage`。 若要與[!DNL Experience Manager]整合為[!DNL Cloud Service]生產環境，請使用Firefly `Production`工作區的應用程式URL。

>[!CAUTION]
>
>請勿在關鍵[!DNL Experience Manager]環境中使用個人工作區。

>[!MORELIKETHIS]
>
>* [了解和管理 [!DNL Experience Manager] 環境 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html)。

