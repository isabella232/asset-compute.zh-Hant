---
title: 設定 [!DNL Asset Compute Service]所需的開發環境。
description: ' [!DNL Asset Compute Service] 的開發人員環境設定，以開始建立和測試自訂代碼。'
translation-type: tm+mt
source-git-commit: 7e520921ebb459c963d61d70c66497b8e62521cf
workflow-type: tm+mt
source-wordcount: '372'
ht-degree: 0%

---


# 設定開發人員環境{#create-dev-environment}

要建立允許您為[!DNL Asset Compute Service]開發的設定，請遵循以下要求和說明。

1. [取得Project Firefly的](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#acquire-access-and-credentials) 存取權和認證。

1. [設定本機環](https://github.com/AdobeDocs/project-firefly/blob/master/getting_started/setup.md#local-environment-set-up) 境和必要工具。

1. 有些工具可協助您順利開始開發：

   * [蠢貨](https://git-scm.com/)。
   * [Docker Desktop](https://www.docker.com/get-started)。
   * [NodeJS](https://nodejs.org) （v10到v12 LTS，不建議使用奇數版本）和 [NPM](https://www.npmjs.com)。OSX HomeBrew的使用者可執行`brew install node`兩者安裝。 否則，請從[NodeJS下載頁面](https://nodejs.org/en/)下載。
   * IDE適合NodeJS，建議使用[Visual Studio代碼（VS代碼）](https://code.visualstudio.com)，因為它是調試器的支援IDE。 您可以將任何其他IDE用作代碼編輯器，但尚未支援高級用法（例如調試程式）。
   * [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`)-使用安裝 `npm install -g @adobe/aio-cli`。

1. 請務必符合[先決條件](/help/understand-extensibility.md#prerequisites-and-provisioning)。

## 設定Firefly專案{#create-firefly-project}

1. 在「體驗組織」中獲得系統管理員或開發人員角色存取權。 這可由系統管理員在[管理控制台](https://adminconsole.adobe.com/overview)中設定。

1. 登入[Adobe Developer Console](https://console.adobe.io/)。 請確定您是[!DNL Experience Manager]與[!DNL Cloud Service]整合的相同Adobe Experience Cloud組織的一員。 如需Adobe Developer Console的詳細資訊，請參閱[ Console檔案](https://www.adobe.io/apis/experienceplatform/console/docs.html)。

1. [建立Firefly專案](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)。按一下「建立新專案&#x200B;]**>**[!UICONTROL &#x200B;從範本&#x200B;]**建立專案」。**[!UICONTROL &#x200B;選取「Firefly」（螢火蟲）。 它會建立新的Firefly專案，其中包含兩個工作區：`Production`和`Stage`。 視需要新增其他工作區，例如`Development`。

1. 在Firefly專案中，選取工作區並訂閱資產計算所需的服務。 按一下「新增至專案&#x200B;**>** API **」，並新增`Asset Compute`、`IO Events`和`IO Events Management`服務。**&#x200B;新增第一個API時，會提示建立私密金鑰。 當您需要使用此金鑰來使用開發人員工具測試您的自訂應用程式時，請將此資訊儲存在您的機器上。

## 下一步{#next-step}

現在您的環境已經設定好，您可以[建立自定義應用程式](develop-custom-application.md)了。

<!-- TBD items for later:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)
-->
