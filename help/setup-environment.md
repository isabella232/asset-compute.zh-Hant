---
title: 設定 [!DNL Asset Compute Service]
description: 開發人員環境設定 [!DNL Asset Compute Service] 以開始建立和測試自訂程式碼。
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 2dde177933477dc9ac2ff5a55af1fd2366e18359
workflow-type: tm+mt
source-wordcount: '357'
ht-degree: 3%

---

# 設定開發人員環境 {#create-dev-environment}

若要建立可讓您針對 [!DNL Asset Compute Service]，請遵循這些需求和指示。

1. [取得存取權和憑證](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) for [!DNL Adobe Developer App Builder].

1. [設定本機環境](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) 和必要的工具。

1. 其他有助於您順利開發的工具包括：

   * [Git](https://git-scm.com/)
   * [Docker Desktop](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) （v14 LTS，不建議使用奇數版本）和 [NPM](https://www.npmjs.com). OSX HomeBrew的用戶可以 `brew install node` 來安裝兩者。 否則，請從 [NodeJS下載頁面](https://nodejs.org/en/)
   * 對於NodeJS來說，我們建議使用適合的IDE [Visual Studio代碼（VS代碼）](https://code.visualstudio.com) 因為它是調試器支援的IDE。 您可以使用任何其他IDE作為代碼編輯器，但目前尚不支援進階用法（例如debugger）
   * 安裝最新[[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)

   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. 一定要見 [必要條件](/help/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## 設定App Builder專案 {#create-App-Builder-project}

1. 在 [!DNL Experience Cloud] 組織。 這是由系統管理員在 [Admin Console](https://adminconsole.adobe.com/overview).

1. 登入 [Adobe Developer Console](https://console.adobe.io/). 確保您屬於相同 [!DNL Experience Cloud] 組織作為 [!DNL Experience Manager] as a [!DNL Cloud Service] 整合。 如需Adobe Developer Console的詳細資訊，請參閱 [主控台檔案](https://www.adobe.io/apis/experienceplatform/console/docs.html).

1. [建立App Builder專案](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). 按一下 **[!UICONTROL 建立新專案]** > **[!UICONTROL 範本專案]**. 選取「應用程式產生器」。 它會建立含兩個工作區的新App Builder專案： `Production` 和 `Stage`. 例如，新增其他工作區 `Development`，視需要。

1. 在App Builder專案中，選取工作區並訂閱Asset compute所需的服務。 按一下 **新增至專案** > **API** 新增 `Asset Compute`, `IO Events`，和 `IO Events Management` 服務。 新增第一個API時，系統會提示建立私密金鑰。 當您需要此金鑰時，將此資訊儲存在電腦上，以便使用開發人員工具測試自訂應用程式。

## 下一步 {#next-step}

現在環境已設定完畢，您就可以 [建立自訂應用程式](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
