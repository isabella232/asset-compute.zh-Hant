---
title: 設定所需的開發環境 [!DNL Asset Compute Service]。
description: 開發人員環境 [!DNL Asset Compute Service] 設定，以開始建立和測試自訂程式碼。
translation-type: tm+mt
source-git-commit: 1c2a1dc41296bf26c432c51b5afa20cb07a4c5c5
workflow-type: tm+mt
source-wordcount: '376'
ht-degree: 0%

---


# 設定開發人員環境 {#create-dev-environment}

若要建立可供您開發的設定，請依 [!DNL Asset Compute Service]照下列需求和指示進行。

1. [取得Project](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials) Firefly的存取權和認證。

1. [設定本機環境](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#local-environment-set-up) ，以及必要的工具。

1. 有些工具可協助您順利開始開發：

   * [蠢貨](https://git-scm.com/)。
   * [Docker Desktop](https://www.docker.com/get-started)。
   * [NodeJS](https://nodejs.org) （v10到v12 LTS，不建議使用奇數版本）和 [NPM](https://www.npmjs.com)。 OSX HomeBrew的使用者可 `brew install node` 以安裝兩者。 否則，請從 [NodeJS下載頁面下載](https://nodejs.org/en/)。
   * NodeJS適用的IDE，我們建議使用 [Visual Studio代碼（VS代碼）](https://code.visualstudio.com) ，因為它是除錯程式的支援IDE。 您可以將任何其他IDE用作代碼編輯器，但尚未支援高級用法（例如調試程式）。
   * [AIO CLI](https://github.com/adobe/aio-cli) (`aio`)-使用安裝 `npm install -g @adobe/aio-cli`。

1. 請務必符合必要 [條件](/help/understand-extensibility.md#prerequisites-and-provisioning)。

## 設定Firefly專案 {#create-firefly-project}

1. 在「體驗組織」中獲得系統管理員或開發人員角色存取權。 這可由系統管理員在「管理控制台」中 [設定](https://adminconsole.adobe.com/overview)。

1. 登入 [Adobe Developer Console](https://console.adobe.io/)。 確保您與AEM是雲端服務整合的相同Adobe Experience Cloud組織。 如需Adobe Developer Console的詳細資訊，請參閱 [Console檔案](https://www.adobe.io/apis/experienceplatform/console/docs.html)。

1. [建立Firefly專案](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)。 按一 **[!UICONTROL 下「建立新專案]** >從范 **[!UICONTROL 本建立專案」]**。 選取「Firefly」（螢火蟲）。 它會建立新的Firefly專案，其中包含兩個工作區： `Production` 和 `Stage`。 例如，視需要新增其 `Development`他工作區。

1. 在Firefly專案中，選取工作區並訂閱資產計算所需的服務。 按一 **下「新增至專案** > **API** 」, `Asset Compute`並新增 `IO Events`、和 `IO Events Management` 服務。 新增第一個API時，會提示建立私密金鑰。 當您需要使用此金鑰來使用開發人員工具測試您的自訂應用程式時，請將此資訊儲存在您的機器上。

## Next step {#next-step}

現在您的環境已經設定好，您就可以建立自 [訂應用程式了](develop-custom-application.md)。

<!-- TBD items for later:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)
-->
