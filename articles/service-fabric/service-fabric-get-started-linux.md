---
title: Configurare l'ambiente di sviluppo in Linux | Microsoft Docs
description: Installare il runtime e l'SDK e creare un cluster di sviluppo locale in Linux. Al termine della configurazione, sarà possibile iniziare a sviluppare applicazioni.
services: service-fabric
documentationcenter: .net
author: mani-ramaswamy
manager: timlt
editor: ''
ms.assetid: d552c8cd-67d1-45e8-91dc-871853f44fc6
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: get-started-article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 2/23/2018
ms.author: subramar
ms.openlocfilehash: bf88e4c702321a7810ec6a3e50eb6cd47a788734
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/23/2018
---
# <a name="prepare-your-development-environment-on-linux"></a>Preparare l'ambiente di sviluppo in Linux
> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started.md)
> * [Linux](service-fabric-get-started-linux.md)
> * [OSX](service-fabric-get-started-mac.md)
>
>  

Per distribuire ed eseguire [applicazioni di Azure Service Fabric](service-fabric-application-model.md) in un computer di sviluppo Linux, installare il runtime e l'SDK comune. È anche possibile installare SDK facoltativi per lo sviluppo Java e .NET Core. 

I passaggi illustrati in questo articolo presuppongono che venga eseguita l'installazione in modalità nativa in Linux o che venga usata un'immagine del contenitore Service Fabric OneBox, `microsoft/service-fabric-onebox`. 

L'installazione del runtime di Service Fabric e dell'SDK nel sottosistema di Windows per Linux non è supportata. È invece supportata l'interfaccia della riga di comando di Azure Service Fabric, che consente di gestire le entità di Service Fabric ospitate in altre posizioni nel cloud o in locale. Per informazioni su come installare l'interfaccia della riga di comando, vedere [Configurare l'interfaccia della riga di comando di Service Fabric](./service-fabric-cli.md).


## <a name="prerequisites"></a>prerequisiti

* Per lo sviluppo, sono supportati i sistemi operativi seguenti:

    * Ubuntu 16.04 (`Xenial Xerus`)

      * Assicurarsi che sia installato il pacchetto `apt-transport-https`:
         
         ```bash
         sudo apt-get install apt-transport-https
         ```
    * Red Hat Enterprise Linux 7.4 (supporto per l'anteprima di Service Fabric)


## <a name="installation-methods"></a>Metodi di installazione

### <a name="1-script-installation-ubuntu"></a>1. Installazione con script (Ubuntu)

Per praticità viene fornito uno script per l'installazione del runtime di Service Fabric e dell'SDK comune di Service Fabric insieme all'interfaccia della riga di comando **sfctl**. Eseguire la procedura di installazione manuale della sezione successiva per determinare gli elementi installati e le licenze accettate. L'esecuzione dello script presuppone l'accettazione delle licenze di tutto il software installato. 

Al termine dell'esecuzione dello script, è possibile passare direttamente a [Configurare un cluster locale](#set-up-a-local-cluster).

```bash
sudo curl -s https://raw.githubusercontent.com/Azure/service-fabric-scripts-and-templates/master/scripts/SetupServiceFabric/SetupServiceFabric.sh | sudo bash
```

### <a name="2-manual-installation"></a>2. Installazione manuale
Per l'installazione manuale del runtime e dell'SDK comune di Service Fabric, seguire il resto di questa guida.

## <a name="update-your-apt-sourcesyum-repositories"></a>Aggiornare le origini APT e i repository Yum
Per installare l'SDK e il pacchetto di runtime associato tramite lo strumento da riga di comando apt-get, è prima necessario aggiornare le origini APT (Advanced Packaging Tool).

### <a name="ubuntu"></a>Ubuntu

1. Aprire un terminale.
2. Aggiungere il repository di Service Fabric all'elenco di origini.

    ```bash
    sudo sh -c 'echo "deb [arch=amd64] http://apt-mo.trafficmanager.net/repos/servicefabric/ xenial main" > /etc/apt/sources.list.d/servicefabric.list'
    ```

3. Aggiungere il repository `dotnet` all'elenco di origini.

    ```bash
    sudo sh -c 'echo "deb [arch=amd64] https://apt-mo.trafficmanager.net/repos/dotnet-release/ xenial main" > /etc/apt/sources.list.d/dotnetdev.list'
    ```

4. Aggiungere la nuova chiave Gnu Privacy Guard (GnuPG o GPG) al keyring di APT.

    ```bash
    sudo apt-key adv --keyserver apt-mo.trafficmanager.net --recv-keys 417A0893
    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 417A0893
    ```

5. Aggiungere la chiave GPG Docker ufficiale al keyring di APT.

    ```bash
    sudo apt-get install curl
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

6. Configurare il repository di Docker.

    ```bash
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    ```

7. Aggiornare l'elenco dei pacchetti in base ai repository appena aggiunti.

    ```bash
    sudo apt-get update
    ```


### <a name="red-hat-enterprise-linux-74-service-fabric-preview-support"></a>Red Hat Enterprise Linux 7.4 (supporto per l'anteprima di Service Fabric)

1. Aprire un terminale.
2. Scaricare e installare Extra Packages for Enterprise Linux (EPEL).

    ```bash
    wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    sudo yum install epel-release-latest-7.noarch.rpm
    ```
3. Aggiungere il repository di pacchetti EfficiOS RHEL7 nel sistema.

    ```bash
    sudo wget -P /etc/yum.repos.d/ https://packages.efficios.com/repo.files/EfficiOS-RHEL7-x86-64.repo
    ```

4. Importare la chiave di firma del pacchetto EfficiOS nel keyring GPG locale.

    ```bash
    sudo rpmkeys --import https://packages.efficios.com/rhel/repo.key
    ```

5. Aggiungere il repository Microsoft RHEL nel sistema.

    ```bash
    curl https://packages.microsoft.com/config/rhel/7.4/prod.repo > ./microsoft-prod.repo
    sudo cp ./microsoft-prod.repo /etc/yum.repos.d/
    ```

6. Installare dotnet sdk.

    ```bash
    yum install rh-dotnet20 -y
    ```

## <a name="install-and-set-up-the-service-fabric-sdk-for-local-cluster-setup"></a>Installare e configurare Service Fabric SDK per la configurazione del cluster locale

Dopo aver aggiornato le origini, è possibile installare l'SDK. Installare il pacchetto Service Fabric SDK, confermare l'installazione e accettare il contratto di licenza.

### <a name="ubuntu"></a>Ubuntu

```bash
sudo apt-get install servicefabricsdkcommon
```

>   [!TIP]
>   I comandi seguenti permettono di automatizzare l'accettazione della licenza per i pacchetti di Service Fabric:
>   ```bash
>   echo "servicefabric servicefabric/accepted-eula-ga select true" | sudo debconf-set-selections
>   echo "servicefabricsdkcommon servicefabricsdkcommon/accepted-eula-ga select true" | sudo debconf-set-selections
>   ```

### <a name="red-hat-enterprise-linux-74-service-fabric-preview-support"></a>Red Hat Enterprise Linux 7.4 (supporto per l'anteprima di Service Fabric)

```bash
sudo yum install servicefabricsdkcommon
```

Il runtime di Service Fabric fornito con l'installazione precedente include i pacchetti elencati nella tabella seguente. 

 | | DotNetCore | Java | Python | NodeJS | 
--- | --- | --- | --- |---
Ubuntu | 2.0.0 | OpenJDK 1.8 | Implicito da npm | più recenti |
RHEL | - | OpenJDK 1.8 | Implicito da npm | più recenti |

## <a name="set-up-a-local-cluster"></a>Configurare un cluster locale
  Al termine dell'installazione, sarà possibile avviare un cluster locale.

  1. Eseguire lo script di configurazione del cluster.

      ```bash
      sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh
      ```

  2. Aprire un Web browser e passare a [Service Fabric Explorer](http://localhost:19080/Explorer) (`http://localhost:19080/Explorer`). Se il cluster è stato avviato, verrà visualizzato il dashboard di Service Fabric Explorer. La configurazione del cluster può richiedere alcuni minuti. Se il browser non riesce ad aprire l'URL o Service Fabric Explorer non visualizza il sistema come pronto, attendere qualche minuto e riprovare.

      ![Service Fabric Explorer in Linux][sfx-linux]

  A questo punto, è possibile distribuire pacchetti di applicazione di Service Fabric predefiniti o nuovi pacchetti basati su contenitori o eseguibili guest. Per creare nuovi servizi usando gli SDK per Java o .NET Core, seguire le procedure di configurazione facoltative riportate nelle sezioni successive.


  > [!NOTE]
  > I cluster autonomi non sono supportati in Linux.
  >


>   [!TIP]
    Se è disponibile un disco SSD, per ottenere prestazioni ottimali è consigliabile passare un percorso di cartella SSD usando `--clusterdataroot` con devclustersetup.sh.

## <a name="set-up-the-service-fabric-cli"></a>Configurare l'interfaccia della riga di comando di Service Fabric

L'[interfaccia della riga di comando di Service Fabric](service-fabric-cli.md) include comandi per l'interazione con entità di Service Fabric, come cluster e applicazioni.
Seguire le istruzioni contenute in [Interfaccia della riga di comando di Service Fabric](service-fabric-cli.md) per installare l'interfaccia della riga di comando.


## <a name="set-up-yeoman-generators-for-containers-and-guest-executables"></a>Configurare i generatori Yeoman per contenitori ed eseguibili guest
Service Fabric offre gli strumenti di scaffolding che consentono di creare applicazioni Service Fabric dal terminale tramite i generatori di modelli Yeoman. Per configurare i generatori di modelli Yeoman di Service Fabric, seguire questa procedura:

1. Installare nodejs e NPM nella macchina virtuale

Ubuntu
  ```bash
  sudo apt-get install npm
  sudo apt install nodejs-legacy
  ```

Red Hat Enterprise Linux 7.4 (supporto per l'anteprima di Service Fabric)
  ```bash
  sudo yum install nodejs
  sudo yum install npm
  ```
2. Installare il generatore di modelli [Yeoman](http://yeoman.io/) nella macchina virtuale da NPM

  ```bash
  sudo npm install -g yo
  ```
3. Installare il generatore di contenitori Yeo per Service Fabric e il generatore di eseguibili guest da NPM

  ```bash
  sudo npm install -g generator-azuresfcontainer  # for Service Fabric container application
  sudo npm install -g generator-azuresfguest      # for Service Fabric guest executable application
  ```

Al termine dell'installazione dei generatori, sarà possibile creare servizi contenitore o eseguibili guest eseguendo, rispettivamente, `yo azuresfcontainer` o `yo azuresfguest`.

## <a name="set-up-net-core-20-development"></a>Configurare lo sviluppo .NET Core 2.0

Installare [.NET Core 2.0 SDK per Ubuntu](https://www.microsoft.com/net/core#linuxubuntu) per avviare la [creazione di applicazioni C# di Service Fabric](service-fabric-create-your-first-linux-application-with-csharp.md). I pacchetti per le applicazioni .NET Core 2.0 di Service Fabric sono ospitati in NuGet.org, attualmente in anteprima.

## <a name="set-up-java-development"></a>Configurare lo sviluppo Java

Per compilare servizi di Service Fabric con Java, installare JDK 1.8 e Gradle per eseguire le attività di compilazione. Il frammento di codice seguente installa Open JDK 1.8 insieme a Gradle. Il pull delle librerie Java per Service Fabric viene eseguito da Maven.


Ubuntu 
 ```bash
  sudo apt-get install openjdk-8-jdk-headless
  sudo apt-get install gradle
  ```

Red Hat Enterprise Linux 7.4 (supporto per l'anteprima di Service Fabric)
  ```bash
  sudo yum install java-1.8.0-openjdk-devel
  curl -s https://get.sdkman.io | bash
  sdk install gradle
  ```
 
## <a name="install-the-eclipse-plug-in-optional"></a>Installare il plug-in Eclipse (facoltativo)

È possibile installare il plug-in Eclipse per Service Fabric dall'IDE di Eclipse per sviluppatori Java o per sviluppatori Java EE. È possibile usare Eclipse per creare applicazioni contenitore e applicazioni eseguibili guest di Service Fabric in aggiunta alle applicazioni Java di Service Fabric.

> [!IMPORTANT]
> Il plug-in Service Fabric richiede Eclipse Neon o versione successiva. Vedere le istruzioni che seguono questa nota per verificare la versione di Eclipse. Se è installata una versione precedente di Eclipse, è possibile scaricare versioni più recenti dal [sito di Eclipse](https://www.eclipse.org). Non è consigliabile eseguire l'installazione sovrascrivendo un'installazione esistente di Eclipse. È possibile rimuoverla prima di eseguire il programma di installazione o installare la versione più recente in una directory diversa. 
> 
> In Ubuntu si consiglia di eseguire l'installazione direttamente dal sito di Eclipse invece di usare un programma di installazione di pacchetti (`apt` o `apt-get`). In questo modo si è certi di ottenere la versione più aggiornata di Eclipse. È possibile installare l'IDE di Eclipse per sviluppatori Java o per sviluppatori Java EE.

1. In Eclipse verificare che siano installati Eclipse Neon o versione successiva e Buildship versione 2.2.1 o successiva. Per controllare le versioni dei componenti installati, selezionare **Help (?)** > **About Eclipse (Informazioni su Eclipse)** > **Installation Details (Dettagli installazione)**. È possibile aggiornare Buildship seguendo le istruzioni riportate in [Eclipse Buildship: Eclipse Plug-ins for Gradle][buildship-update] (Eclipse Buildship: plug-in Eclipse per Gradle).

2. Per installare il plug-in Service Fabric, selezionare **Help** > **Install New Software** (? > Installa nuovo software).

3. Nella casella **Work with** (Lavora con) digitare **http://dl.microsoft.com/eclipse**.

4. Fare clic su **Aggiungi**.

    ![Pagina relativa al software disponibile][sf-eclipse-plugin]

5. Selezionare il plug-in **ServiceFabric** e quindi fare clic su **Next** (Avanti).

6. Completare la procedura di installazione e quindi accettare il contratto di licenza con l'utente finale.

Se il plug-in Eclipse per Service Fabric è già installato, verificare che la versione sia la più recente. Per controllare, selezionare **Help (?)** > **About Eclipse (Informazioni su Eclipse)** > **Installation Details (Dettagli installazione)** e quindi cercare Service Fabric nell'elenco dei plug-in installati. Se è disponibile una versione più recente, selezionare **Update** (Aggiorna).

Per altre informazioni, vedere [Plug-in Service Fabric per lo sviluppo di applicazioni Java in Eclipse](service-fabric-get-started-eclipse.md).

## <a name="update-the-sdk-and-runtime"></a>Aggiornare SDK e runtime

Per aggiornare l'SDK e il runtime all'ultima versione, eseguire questi comandi:

```bash
sudo apt-get update
sudo apt-get install servicefabric servicefabricsdkcommon
```
Per aggiornare i file binari Java SDK da Maven, è necessario aggiornare i dettagli della versione del file binario corrispondente nel file ``build.gradle`` in modo che facciano riferimento alla versione più recente. Per individuare esattamente il punto in cui è necessario aggiornare la versione, è possibile vedere qualsiasi file ``build.gradle`` negli esempi introduttivi di Service Fabric , disponibili [qui](https://github.com/Azure-Samples/service-fabric-java-getting-started).

> [!NOTE]
> L'aggiornamento dei pacchetti può causare l'arresto dell'esecuzione del cluster di sviluppo locale. Riavviare il cluster locale dopo un aggiornamento seguendo le istruzioni disponibili in questa pagina.

## <a name="remove-the-sdk"></a>Rimuovere l'SDK
Per rimuovere le istanze di Service Fabric SDK, eseguire questi comandi:

### <a name="ubuntu"></a>Ubuntu

```bash
sudo apt-get remove servicefabric servicefabicsdkcommon
sudo npm uninstall generator-azuresfcontainer
sudo npm uninstall generator-azuresfguest
sudo apt-get install -f
```


### <a name="red-hat-enterprise-linux-74-service-fabric-preview-support"></a>Red Hat Enterprise Linux 7.4 (supporto per l'anteprima di Service Fabric)

```bash
sudo yum remote servicefabric servicefabicsdkcommon
sudo npm uninstall generator-azuresfcontainer
sudo npm uninstall generator-azuresfguest
```

## <a name="next-steps"></a>Passaggi successivi

* [Creare e distribuire la prima applicazione Java di Service Fabric in Linux usando Yeoman](service-fabric-create-your-first-linux-application-with-java.md)
* [Creare e distribuire la prima applicazione Java di Service Fabric in Linux usando il plug-in Service Fabric per Eclipse](service-fabric-get-started-eclipse.md)
* [Creare la prima applicazione CSharp in Linux](service-fabric-create-your-first-linux-application-with-csharp.md)
* [Preparare l'ambiente di sviluppo in OSX](service-fabric-get-started-mac.md)
* [Prepare a Linux development environment on Windows (Preparare un ambiente di sviluppo Linux in Windows)](service-fabric-local-linux-cluster-windows.md)
* [Usare l'interfaccia della riga di comando di Service Fabric per gestire le applicazioni](service-fabric-application-lifecycle-sfctl.md)
* [Differenze in Service Fabric tra Windows e Linux](service-fabric-linux-windows-differences.md)
* [Automatizzare l'applicazione di patch al sistema operativo nel cluster Linux](service-fabric-patch-orchestration-application-linux.md)
* [Introduzione all'interfaccia della riga di comando di Service Fabric](service-fabric-cli.md)

<!-- Links -->

[azure-xplat-cli-github]: https://github.com/Azure/azure-xplat-cli
[install-node]: https://nodejs.org/en/download/package-manager/#installing-node-js-via-package-manager
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship

<!--Images -->

[sf-eclipse-plugin]: ./media/service-fabric-get-started-linux/service-fabric-eclipse-plugin.png
[sfx-linux]: ./media/service-fabric-get-started-linux/sfx-linux.png
