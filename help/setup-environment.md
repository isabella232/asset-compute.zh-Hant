---
title: 設定所需的開發環境 [!DNL Asset Compute Service]
description: 開發人員環境設定 [!DNL Asset Compute Service] 開始建立和測試自定義代碼。
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '357'
ht-degree: 3%

---

# 設定開發人員環境 {#create-dev-environment}

建立允許您開發的設定 [!DNL Asset Compute Service]，請遵循這些要求和說明。

1. [獲取訪問權和憑據](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) 為 [!DNL Adobe Developer App Builder]。

1. [設定本地環境](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) 以及所需的工具。

1. 還有一些工具可幫助您開始順利開發：

   * [Git](https://git-scm.com/)
   * [Docker案頭](https://www.docker.com/get-started)
   * [節點JS](https://nodejs.org) （v14 LTS，不建議使用奇數版本）和 [NPM](https://www.npmjs.com)。 OSX HomeBrew的用戶可以 `brew install node` 安裝兩者。 否則，從 [NodeJS下載頁](https://nodejs.org/en/)
   * 對於NodeJS，我們建議使用一個IDE [Visual Studio代碼（VS代碼）](https://code.visualstudio.com) 因為它是調試器支援的IDE。 可以將任何其它IDE用作代碼編輯器，但尚不支援高級用法（例如調試器）
   * 安裝最新[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)

   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. 確保與 [先決條件](/help/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## 設定App Builder項目 {#create-App-Builder-project}

1. 確保系統管理員或開發人員角色 [!DNL Experience Cloud] 組織。 這由系統管理員在 [Admin Console](https://adminconsole.adobe.com/overview)。

1. 登錄 [Adobe Developer控制台](https://console.adobe.io/)。 確保您屬於同一組 [!DNL Experience Cloud] 組織 [!DNL Experience Manager] 作為 [!DNL Cloud Service] 整合。 有關Adobe Developer控制台的詳細資訊，請參見 [控制台文檔](https://www.adobe.io/apis/experienceplatform/console/docs.html)。

1. [建立應用生成器項目](https://developer.adobe.com/app-builder/docs/getting_started/first_app/)。 按一下 **[!UICONTROL 建立新項目]** > **[!UICONTROL 來自模板的項目]**。 選擇應用生成器。 它將建立一個包含兩個工作區的新App Builder項目： `Production` 和 `Stage`。 添加其他工作區，例如 `Development`，如需要。

1. 在App Builder項目中，選擇一個工作區並訂閱Asset compute所需的服務。 按一下 **添加到項目** > **API** 添加 `Asset Compute`。 `IO Events`, `IO Events Management` 服務。 添加第一個API時，它會提示建立私鑰。 在需要此密鑰時將此資訊保存在電腦上，以便使用開發人員工具test自定義應用程式。

## 下一步 {#next-step}

現在您的環境已設定，您已準備好 [建立自定義應用程式](develop-custom-application.md)。

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
