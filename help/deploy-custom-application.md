---
title: 部署 [!DNL Asset Compute Service] 自訂應用程式。
description: 部署 [!DNL Asset Compute Service] 自訂應用程式。
translation-type: tm+mt
source-git-commit: 1c2a1dc41296bf26c432c51b5afa20cb07a4c5c5
workflow-type: tm+mt
source-wordcount: '201'
ht-degree: 8%

---


# 部署自訂應用程式 {#deploy-custom-application}

若要部署您的應用程式，請使 [用aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy) 命令。 在終端機中，命令會顯示存取自訂應用程式的URL。 URL的格式為 `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`。

若要取得相同的URL而不重新部署應用程式，請使用 [`aio app get-url`](https://github.com/adobe/aio-cli#aio-appget-url-action) 命令。

將Experience Manager中處 [理設定檔的URL當做雲端服務](https://docs.adobe.com/content/help/zh-Hant/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) ，將應用程式與雲 [!DNL Experience Manager] 端服務整合。

請確定您的Firefly專案和工作區對應至您 [!DNL Experience Manager] 要使用動作的雲端服務環境。 它有不同的開發、測試和生產環境。 您可以通過檢查Firefly應 `AIO_runtime_*` 用程式根中ENV檔案中定義的憑據來驗證環境。 例如，要部署到工 `Stage` 作區， `AIO_runtime_namespace` 格式為 `xxxxxx_xxxxxxxxx_stage`。 若要與雲端 [!DNL Experience Manager] 服務生產環境整合，請使用Firefly工作區中的應用程式 `Production` URL。

>[!CAUTION]
>
>切勿在關鍵環境上使用個人工 [!DNL Experience Manager] 作區。

>[!MORELIKETHIS]
>
>* [以雲端服務的形式瞭解並管理Experience Manager中的環境](https://docs.adobe.com/content/help/zh-Hant/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html)。

