---
title: 部署 [!DNL Asset Compute Service] 自訂應用程式
description: 部署 [!DNL Asset Compute Service] 自訂應用程式。
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 50f69e16772cee7f79a812f2b86f0ef0221db369
workflow-type: tm+mt
source-wordcount: '190'
ht-degree: 3%

---

# 部署自定義應用程式 {#deploy-custom-application}

要部署應用程式，請使用 [aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy) 命令。 在終端機中，命令會顯示存取自訂應用程式的URL。 URL為格式 `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

若要在不重新部署應用程式的情況下取得相同的URL，請使用 [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) 命令。

在 [處理設定檔(於 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html) 將應用程式與 [!DNL Experience Manager] as a [!DNL Cloud Service].

確認您的App Builder專案和工作區對應至 [!DNL Experience Manager] as a [!DNL Cloud Service] 您要使用動作的環境。 其開發、測試和生產環境不同。 您可以檢查 `AIO_runtime_*` 在ENV檔案中定義、位於Adobe Developer App Builder應用程式根目錄的憑證。 例如，若要部署至 `Stage` 工作區、 `AIO_runtime_namespace` 為格式 `xxxxxx_xxxxxxxxx_stage`. 若要整合 [!DNL Experience Manager] as a [!DNL Cloud Service] 生產環境，使用Adobe Developer App Builder中的應用程式URL `Production` 工作區。

>[!CAUTION]
>
>請勿在重要環境上使用個人工作區 [!DNL Experience Manager] 環境。

>[!MORELIKETHIS]
>
>* [了解和管理 [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).

