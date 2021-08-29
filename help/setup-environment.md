---
title: 設定 [!DNL Asset Compute Service]所需的開發環境
description: ' [!DNL Asset Compute Service] 的開發人員環境設定，以開始建立和測試自訂程式碼。'
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: 9404ffcc66a3b6ba206155d1b1a5c16a43e22a39
workflow-type: tm+mt
source-wordcount: '370'
ht-degree: 2%

---

# 設定開發人員環境 {#create-dev-environment}

若要建立可讓您為[!DNL Asset Compute Service]開發的設定，請遵循下列需求和指示。

1. [取得存取權](https://www.adobe.io/project-firefly/docs/getting_started/#acquire-access-and-credentials) 和認證 [!DNL Project Firefly]。

1. [設定本機環](https://www.adobe.io/project-firefly/docs/getting_started/#local-environment-set-up) 境和必要的工具。

1. 其他有助於您順利開發的工具包括：

   * [Git](https://git-scm.com/)。
   * [Docker案頭](https://www.docker.com/get-started)。
   * [NodeJS](https://nodejs.org) （v10至v12 LTS，不建議使用奇數版本）和 [NPM](https://www.npmjs.com)。OSX HomeBrew的用戶可以執行`brew install node`以安裝兩者。 否則，請從[NodeJS下載頁面](https://nodejs.org/en/)下載。
   * 對於NodeJS來說，我們建議使用[Visual Studio代碼（VS代碼）](https://code.visualstudio.com)這是Debugger支援的IDE。 您可以使用任何其他IDE作為代碼編輯器，但尚不支援高級用法（例如調試器）。
   * [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) (`aio`) — 使用安裝 `npm install -g @adobe/aio-cli@7.1.0`。

1. 請務必符合[必要條件](/help/understand-extensibility.md#prerequisites-and-provisioning)。

>[!NOTE]
>
>目前，請使用[!DNL Adobe I/O] CLI v7.1.0，而不要使用[!DNL Adobe I/O] CLI v8。

## 設定Firefly專案 {#create-firefly-project}

1. 確保[!DNL Experience Cloud]組織中的系統管理員或開發人員角色。 這由系統管理員在[Admin Console](https://adminconsole.adobe.com/overview)中設定。

1. 登入[Adobe開發人員控制台](https://console.adobe.io/)。 確認您是[!DNL Experience Manager]與[!DNL Cloud Service]整合相同[!DNL Experience Cloud]組織的一員。 如需Adobe開發人員控制台的詳細資訊，請參閱[主控台檔案](https://www.adobe.io/apis/experienceplatform/console/docs.html)。

1. [建立Firefly專案](https://www.adobe.io/apis/experienceplatform/project-firefly/docs.html#!AdobeDocs/project-firefly/master/getting_started/first_app.md)。按一下「從模板&#x200B;]**建立新項目]** > **[!UICONTROL 項目」。**[!UICONTROL &#x200B;選擇Firefly。 它會建立新的Firefly專案，其中包含兩個工作區：`Production`和`Stage`。 視需要新增其他工作區，例如`Development`。

1. 在Firefly專案中，選取工作區並訂閱Asset compute所需的服務。 按一下「**新增至專案** > **API**」並新增`Asset Compute`、`IO Events`和`IO Events Management`服務。 新增第一個API時，系統會提示建立私密金鑰。 當您需要此金鑰時，將此資訊儲存在電腦上，以便使用開發人員工具測試自訂應用程式。

## 下一步 {#next-step}

現在環境已設定完畢，您就可以[建立自定義應用程式](develop-custom-application.md)了。

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
