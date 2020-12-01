---
title: 部署 [!DNL Asset Compute Service] 自訂應用程式。
description: 部署 [!DNL Asset Compute Service] 自訂應用程式。
translation-type: tm+mt
source-git-commit: 78c1246f5fc42006013701a6cf4d375a1d8c9fd8
workflow-type: tm+mt
source-wordcount: '183'
ht-degree: 0%

---


# 部署自訂應用程式{#deploy-custom-application}

若要部署您的應用程式，請使用[aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy)命令。 在終端機中，命令會顯示存取自訂應用程式的URL。 URL的格式為`https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`。

若要取得相同的URL而不重新部署應用程式，請使用[`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action)命令。

使用 [!DNL Experience Manager] 中[處理描述檔中的URL作為 [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html)，將您的應用程式與[!DNL Experience Manager]作為[!DNL Cloud Service]整合。

請確定您的Firefly專案和工作區對應至您要使用動作的[!DNL Experience Manager]環境。 [!DNL Cloud Service]它有不同的開發、測試和生產環境。 您可以通過檢查`AIO_runtime_*`憑據來驗證環境，這些憑據是在Firefly應用程式根目錄的ENV檔案中定義的。 例如，要部署到`Stage`工作區，`AIO_runtime_namespace`的格式為`xxxxxx_xxxxxxxxx_stage`。 若要將[!DNL Experience Manager]整合為[!DNL Cloud Service]生產環境，請使用Firefly `Production`工作區中的應用程式URL。

>[!CAUTION]
>
>請勿在關鍵[!DNL Experience Manager]環境中使用個人工作區。

>[!MORELIKETHIS]
>
>* [瞭解並管理環境 [!DNL Experience Manager] a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html)。

