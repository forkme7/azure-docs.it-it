---
title: Personalizzare cluster HDInsight tramite l'azione script - Azure | Microsoft Docs
description: Aggiungere componenti personalizzati nei cluster HDInsight basati su Linux tramite azioni script. Le azioni script sono script Bash utilizzabili per personalizzare la configurazione del cluster o aggiungere servizi e utilità come Hue, Solr o R.
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal
ms.assetid: 48e85f53-87c1-474f-b767-ca772238cc13
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: na
ms.topic: conceptual
ms.date: 01/29/2018
ms.author: larryfr
ms.openlocfilehash: ff8b8ffc66ea4c90ca2f11edc36aac670f24eac2
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/28/2018
---
# <a name="customize-linux-based-hdinsight-clusters-using-script-actions"></a>Personalizzare i cluster HDInsight basati su Linux tramite azioni script

HDInsight offre un'opzione di configurazione denominata **Azione script** che richiama script personalizzati per la personalizzazione del cluster. Questi script vengono utilizzati per installare i componenti aggiuntivi e modificare le impostazioni di configurazione. Le azioni script possono essere utilizzate durante o dopo la creazione del cluster.

> [!IMPORTANT]
> La possibilità di usare le azioni script in un cluster già in esecuzione è disponibile solo per i cluster HDInsight basati su Linux.
>
> Linux è l'unico sistema operativo usato in HDInsight versione 3.4 o successiva. Per altre informazioni, vedere la sezione relativa al [ritiro di HDInsight in Windows](hdinsight-component-versioning.md#hdinsight-windows-retirement).

Le azioni di script possono anche essere pubblicate in Azure Marketplace come applicazione HDInsight. Alcuni esempi in questo documento mostrano come è possibile installare un'applicazione HDInsight usando i comandi di azione script di PowerShell e .NET SDK. Per altre informazioni sulle applicazioni HDInsight, vedere [Pubblicare applicazioni HDInsight in Azure Marketplace](hdinsight-apps-publish-applications.md).

## <a name="permissions"></a>Autorizzazioni

Se si usa un cluster HDInsight aggiunto a un dominio, sono necessarie due autorizzazioni Ambari per l'uso di azioni script con il cluster:

* **AMBARI.RUN\_CUSTOM\_COMMAND**: il ruolo di amministratore Ambari ha questa autorizzazione per impostazione predefinita.
* **CLUSTER.RUN\_CUSTOM\_COMMAND**: sia l'amministratore cluster HDInsight che l'amministratore Ambari hanno questa autorizzazione per impostazione predefinita.

Per altre informazioni sull'uso delle autorizzazioni con HDInsight aggiunto a un dominio, vedere [Manage domain-joined HDInsight clusters](./domain-joined/apache-domain-joined-manage.md) (Gestire cluster HDInsight aggiunti al dominio).

## <a name="access-control"></a>Controllo di accesso

Se non si è l'amministratore o il proprietario della sottoscrizione di Azure, l'account deve disporre almeno dell'accesso **Collaboratore** al gruppo di risorse contenente il cluster HDInsight.

Se si crea un cluster HDInsight, è necessario che un utente con almeno l'accesso **Collaboratore** alla sottoscrizione di Azure abbia registrato in precedenza il provider per HDInsight. La registrazione del provider viene eseguita quando un utente con accesso Collaboratore alla sottoscrizione crea una risorsa per la prima volta nella sottoscrizione stessa. Può essere eseguita anche senza creare una risorsa [registrando un provider tramite REST](https://msdn.microsoft.com/library/azure/dn790548.aspx).

Per altre informazioni sull'uso della gestione degli accessi, vedere i documenti seguenti:

* [Introduzione alla gestione degli accessi nel portale di Azure](../role-based-access-control/overview.md)
* [Usare le assegnazioni di ruolo per gestire l'accesso alle risorse della sottoscrizione di Azure](../role-based-access-control/role-assignments-portal.md)

## <a name="understanding-script-actions"></a>Informazioni sulle azioni script

Un'azione di script è uno script Bash eseguito sui nodi di un cluster HDInsight. Di seguito sono riportate le caratteristiche e funzionalità delle azioni script.

* Devono essere archiviate in un URI accessibile dal cluster HDInsight. Di seguito vengono indicate alcune tra le posizioni di archiviazione possibili:

    * Account **Azure Data Lake Store** accessibile dal cluster HDInsight. Per informazioni sull'uso di Azure Data Lake Store con HDInsight, vedere [Creare un cluster HDInsight con Data Lake Store usando il portale](../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md).

        Quando si usa uno script archiviato in Data Lake Store, il formato dell'URI è `adl://DATALAKESTOREACCOUNTNAME.azuredatalakestore.net/path_to_file`.

        > [!NOTE]
        > L'entità servizio usata da HDInsight per accedere a Data Lake Store deve avere accesso in lettura allo script.

    * Un BLOB in un **account di archiviazione di Azure** che rappresenta l'account di archiviazione primario o aggiuntivo per il cluster HDInsight. HDInsight può accedere a entrambi i tipi di account di archiviazione durante la creazione del cluster.

    * Un servizio di condivisione file pubblico, ad esempio BLOB di Azure, GitHub, OneDrive, Dropbox e così via.

        Per gli URI di esempio, vedere la sezione [Script di Azione script di esempio](#example-script-action-scripts).

        > [!WARNING]
        > HDInsight supporta solo account di archiviazione di Azure __per uso generico__. Non supporta attualmente il tipo di account di __archiviazione BLOB__.

* Possono essere limitate all'**esecuzione solo in alcuni tipi di nodi**, ad esempio nodi head o del ruolo di lavoro.

* Possono essere **persistenti** o **ad hoc**.

    Gli script **persistenti** vengono usati per personalizzare nuovi nodi del ruolo di lavoro nel cluster tramite operazioni di ridimensionamento. Uno script persistente può anche applicare modifiche apportate a un altro tipo di nodo, ad esempio un nodo head, durante operazioni di ridimensionamento.

  > [!IMPORTANT]
  > Le azioni script persistenti devono avere un nome univoco.

    Gli script **ad hoc** non sono persistenti. Non vengono infatti applicati ai nodi del ruolo di lavoro aggiunti al cluster dopo l'esecuzione dello script. È possibile alzare di livello uno script ad hoc in un secondo momento per renderlo persistente o abbassare di livello uno script persistente per renderlo ad hoc.

  > [!IMPORTANT]
  > Le azioni script usate durante la creazione di un cluster vengono automaticamente rese persistenti.
  >
  > Gli script che hanno esito negativo non vengono resi persistenti, anche in presenza di indicazioni specifiche in tal senso.

* Possono accettare **parametri** usati dallo script durante l'esecuzione.

* Eseguire lo script con **privilegi a livello radice** nei nodi del cluster.

* Possono essere usate con il **portale di Azure**, **Azure PowerShell**, l'**interfaccia della riga di comando di Azure 1.0** o **HDInsight .NET SDK**.

Il cluster mantiene una cronologia di tutti gli script eseguiti. La cronologia è utile quando è necessario trovare l'ID di uno script per le operazioni di innalzamento o abbassamento di livello.

> [!IMPORTANT]
> Non esiste un metodo automatico per annullare le modifiche apportate da un'azione script. Annullare manualmente le modifiche o fornire uno script che le inverta.

### <a name="script-action-in-the-cluster-creation-process"></a>Azione script nel processo di creazione di cluster

Le azioni script usate durante la creazione del cluster sono leggermente diverse da quelle eseguite in un cluster esistente:

* Lo script viene **salvato automaticamente in modo permanente**.

* Un **errore** nello script può causare l'esito negativo del processo di creazione del cluster.

Il diagramma seguente illustra quando viene eseguita l'azione script durante il processo di creazione:

![Personalizzazione di cluster HDInsight e fasi durante la creazione di un cluster][img-hdi-cluster-states]

Lo script viene eseguito durante la configurazione di HDInsight. Lo script viene eseguito in parallelo in tutti i nodi specificati nel cluster e con privilegi a livello radice sui nodi.

> [!NOTE]
> È possibile eseguire operazioni come l'arresto e l'avvio di servizi, inclusi quelli correlati ad Hadoop. Se si arrestano i servizi, è necessario assicurarsi che i servizi di Ambari e altri servizi correlati ad Hadoop siano attivi prima che sia completato lo script. Questi servizi sono necessari per determinare correttamente l'integrità e lo stato del cluster durante la creazione.


Durante la creazione del cluster, è possibile usare più azioni di script alla volta. Questi script vengono richiamati nell'ordine in cui sono stati specificati.

> [!IMPORTANT]
> Le azioni di script devono essere completate entro 60 minuti; in caso contrario si verifica un timeout. Durante il provisioning dei cluster, lo script viene eseguito contemporaneamente ad altri processi di installazione e configurazione. In caso di concorrenza per risorse come il tempo di CPU o la larghezza di banda di rete, lo script può richiedere più tempo per completare l'operazione rispetto al tempo che impiegherebbe in un ambiente di sviluppo.
>
> Per ridurre al minimo il tempo necessario per eseguire lo script, evitare attività come il download e la compilazione di applicazioni dall'origine. Precompilare le applicazioni e archiviare il file binario in Archiviazione di Azure.


### <a name="script-action-on-a-running-cluster"></a>Azione script in un cluster in esecuzione

Un errore in uno script eseguito in un cluster già in esecuzione non determina automaticamente uno stato di errore del cluster. Al termine di uno script, il cluster deve restituire uno stato In esecuzione.

> [!IMPORTANT]
> Anche se lo stato del cluster è "in esecuzione", lo script non riuscito potrebbe includere attività interrotte. Ad esempio, uno script potrebbe eliminare file richiesti dal cluster.
>
> Le azioni script vengono eseguite con privilegi a livello radice. Occorre quindi conoscere il comportamento di uno script prima di applicarlo al cluster.

Quando si applica uno script a un cluster, lo stato del cluster passa da **In esecuzione** ad **Accettato**, quindi a **Configurazione di HDInsight** e infine di nuovo a **In esecuzione** per gli script con esito positivo. Lo stato dello script viene registrato nella cronologia dell'azione script, che può essere usata per determinare l'esito positivo o negativo dello script. Ad esempio, il cmdlet di PowerShell `Get-AzureRmHDInsightScriptActionHistory` può essere usato per visualizzare lo stato di uno script. Verranno restituite informazioni simili al testo seguente:

    ScriptExecutionId : 635918532516474303
    StartTime         : 8/14/2017 7:40:55 PM
    EndTime           : 8/14/2017 7:41:05 PM
    Status            : Succeeded

> [!IMPORTANT]
> Se è stata cambiata la password dell'utente del cluster (admin) dopo la creazione del cluster stesso, questo potrebbe causare l'esito negativo delle azioni script eseguite su questo cluster. Nel caso in cui siano presenti azioni script persistenti che hanno come destinazione nodi del ruolo di lavoro diversi, questi script potrebbero avere esito negativo se si ridimensiona il cluster.

## <a name="example-script-action-scripts"></a>Script di esempio di azione script

Gli script di azione script possono essere usati tramite le utilità seguenti:

* Portale di Azure
* Azure PowerShell
* Interfaccia della riga di comando di Azure 1.0
* HDInsight .NET SDK

HDInsight fornisce script di esempio per installare i componenti seguenti nei cluster HDInsight:

| NOME | Script |
| --- | --- |
| **Aggiungere un account di archiviazione di Azure** |https://hdiconfigactions.blob.core.windows.net/linuxaddstorageaccountv01/add-storage-account-v01.sh. Vedere [Add additional storage to an HDInsight cluster](hdinsight-hadoop-add-storage.md) (Aggiungere altra memoria a un cluster HDInsight). |
| **Installare Hue.** |https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv02/install-hue-uber-v02.sh. Vedere [Installare e usare Hue in cluster HDInsight](hdinsight-hadoop-hue-linux.md). |
| **Installare Presto** |https://raw.githubusercontent.com/hdinsight/presto-hdinsight/master/installpresto.sh. Vedere [Installare e usare Presto nei cluster HDInsight Hadoop](hdinsight-hadoop-install-presto.md). |
| **Installare Solr** |https://hdiconfigactions.blob.core.windows.net/linuxsolrconfigactionv01/solr-installer-v01.sh. Vedere [Installare e usare Solr nei cluster Hadoop di HDInsight](hdinsight-hadoop-solr-install-linux.md). |
| **Installare Giraph** |https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh. Vedere [Installare Giraph nei cluster HDInsight Hadoop](hdinsight-hadoop-giraph-install-linux.md). |
| **Precaricare le librerie Hive** |https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh. Vedere l'articolo relativo all' [aggiunta di librerie Hive in cluster HDInsight](hdinsight-hadoop-add-hive-libraries.md). |
| **Installare o aggiornare Mono** | https://hdiconfigactions.blob.core.windows.net/install-mono/install-mono.bash. Vedere [Installare o aggiornare Mono in HDInsight](hdinsight-hadoop-install-mono.md). |

## <a name="use-a-script-action-during-cluster-creation"></a>Usare un'azione script durante la creazione di un cluster

In questa sezione vengono forniti esempi sulle diverse modalità di utilizzo delle azioni script durante la creazione di un cluster HDInsight.

### <a name="use-a-script-action-during-cluster-creation-from-the-azure-portal"></a>Usare un'azione script durante la creazione di un cluster dal portale di Azure

1. Avviare la creazione di un cluster come descritto in [Creare cluster Hadoop in HDInsight](hdinsight-hadoop-provision-linux-clusters.md). Fermarsi quando si raggiunge la sezione __Riepilogo del cluster__.

2. Nella sezione __Riepilogo del cluster__ selezionare il collegamento __Modifica__ per __Impostazioni avanzate__.

    ![Collegamento impostazioni avanzate](./media/hdinsight-hadoop-customize-cluster-linux/advanced-settings-link.png)

3. Nella sezione __Impostazioni avanzate__ selezionare __Azioni script__. Nella sezione __Azioni script__ selezionare __+ Invia nuova__

    ![Inviare una nuova azione script](./media/hdinsight-hadoop-customize-cluster-linux/add-script-action.png)

4. Usare la voce __Seleziona uno script__ per selezionare uno script pronto. Per usare uno script personalizzato, selezionare __Personalizzato__ e indicare __Nome__ e __URI script Bash__ per lo script.

    ![Aggiungere uno script nel modulo di selezione script](./media/hdinsight-hadoop-customize-cluster-linux/select-script.png)

    Nella tabella seguente vengono illustrati gli elementi nel modulo:

    | Proprietà | Valore |
    | --- | --- |
    | Selezionare uno script | Per usare uno script personalizzato, selezionare __Personalizzato__. In caso contrario, selezionare uno degli script disponibili. |
    | NOME |Specificare un nome per l'azione script. |
    | URI script Bash |Specificare l'URI dello script. |
    | Head/Worker/Zookeeper |Specificare i nodi **head**, **ruolo di lavoro** o **Zookeeper** in cui viene eseguito lo script. |
    | Parametri |Specificare i parametri, se richiesti dallo script. |

    Usare la voce __Salvare questa azione script in modo permanente__ per assicurarsi che lo script venga applicato durante le operazioni di ridimensionamento.

5. Selezionare __Crea__ per salvare lo script. È quindi possibile usare __+ Invia nuovo__ per aggiungere un altro script.

    ![Azioni script multiple](./media/hdinsight-hadoop-customize-cluster-linux/multiple-scripts.png)

    Dopo aver completato l'aggiunta degli script, usare il pulsante __Seleziona__ e quindi il pulsante __Avanti__ per tornare alla sezione __Riepilogo del cluster__.

3. Per creare il cluster, selezionare __Crea__ nella sezione __Riepilogo del cluster__.

### <a name="use-a-script-action-from-azure-resource-manager-templates"></a>Usare un'azione script dai modelli di Azure Resource Manager

Le azioni script possono essere usate con i modelli di Azure Resource Manager. Per un esempio, vedere [https://azure.microsoft.com/resources/templates/hdinsight-linux-run-script-action/](https://azure.microsoft.com/resources/templates/hdinsight-linux-run-script-action/).

In questo esempio, l'azione script viene aggiunta tramite il codice seguente:

```json
"scriptActions": [
    {
        "name": "setenvironmentvariable",
        "uri": "[parameters('scriptActionUri')]",
        "parameters": "headnode"
    }
]
```

Per informazioni su come distribuire un modello, vedere i documenti seguenti:

* [Distribuire le risorse con i modelli di Azure Resource Manager e Azure PowerShell](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy)

* [Distribuire le risorse con i modelli di Azure Resource Manager e l'interfaccia della riga di comando di Azure](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-template-deploy-cli)

### <a name="use-a-script-action-during-cluster-creation-from-azure-powershell"></a>Usare un'azione script durante la creazione di un cluster da Azure PowerShell

In questa sezione si userà il cmdlet [Add-AzureRmHDInsightScriptAction](https://msdn.microsoft.com/library/mt603527.aspx) per richiamare script per personalizzare un cluster. Prima di procedere, assicurarsi di aver installato e configurato Azure PowerShell. Per informazioni sulla configurazione di una workstation per l'esecuzione di cmdlet PowerShell per HDInsight, vedere [Come installare e configurare Azure PowerShell](/powershell/azure/overview).

Lo script seguente mostra come applicare un'azione script durante la creazione di un cluster con PowerShell:

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=5-90)]

La creazione del cluster può richiedere alcuni minuti.

### <a name="use-a-script-action-during-cluster-creation-from-the-hdinsight-net-sdk"></a>Usare un'azione script durante la creazione di un cluster da HDInsight .NET SDK

HDInsight .NET SDK fornisce librerie client che semplificano l'uso di HDInsight da un'applicazione .NET. Per un esempio di codice, vedere [Creare cluster basati su Linux in HDInsight tramite .NET SDK](hdinsight-hadoop-create-linux-clusters-dotnet-sdk.md#use-script-action).

## <a name="apply-a-script-action-to-a-running-cluster"></a>Applicare un'azione script a un cluster in esecuzione

Questa sezione offre informazioni su come applicare azioni script a un cluster in esecuzione.

### <a name="apply-a-script-action-to-a-running-cluster-from-the-azure-portal"></a>Applicare un'azione script a un cluster in esecuzione dal portale di Azure

1. Nel [portale di Azure](https://portal.azure.com)selezionare il cluster HDInsight.

2. Nella panoramica del cluster HDInsight selezionare il riquadro **Azioni script**.

    ![Riquadro Azioni script](./media/hdinsight-hadoop-customize-cluster-linux/scriptactionstile.png)

   > [!NOTE]
   > È anche possibile selezionare **Tutte le impostazioni** e quindi **Azioni script** dalla sezione Impostazioni.

3. Nella parte superiore della sezione Azioni script selezionare **Invia nuova**.

    ![Aggiungere uno script a un cluster in esecuzione](./media/hdinsight-hadoop-customize-cluster-linux/add-script-running-cluster.png)

4. Usare la voce __Seleziona uno script__ per selezionare uno script pronto. Per usare uno script personalizzato, selezionare __Personalizzato__ e indicare __Nome__ e __URI script Bash__ per lo script.

    ![Aggiungere uno script nel modulo di selezione script](./media/hdinsight-hadoop-customize-cluster-linux/select-script.png)

    Nella tabella seguente vengono illustrati gli elementi nel modulo:

    | Proprietà | Valore |
    | --- | --- |
    | Selezionare uno script | Per usare uno script personalizzato, selezionare __Personalizzato__. In caso contrario, selezionare uno degli script disponibili. |
    | NOME |Specificare un nome per l'azione script. |
    | URI script Bash |Specificare l'URI dello script. |
    | Head/Worker/Zookeeper |Specificare i nodi **head**, **ruolo di lavoro** o **Zookeeper** in cui viene eseguito lo script. |
    | Parametri |Specificare i parametri, se richiesti dallo script. |

    Usare la voce __Salvare questa azione script in modo permanente__ per assicurarsi che lo script venga applicato durante le operazioni di ridimensionamento.

5. Infine, usare il pulsante **Crea** per applicare lo script al cluster.

### <a name="apply-a-script-action-to-a-running-cluster-from-azure-powershell"></a>Applicare un'azione script a un cluster in esecuzione da Azure PowerShell

Prima di procedere, assicurarsi di aver installato e configurato Azure PowerShell. Per informazioni sulla configurazione di una workstation per l'esecuzione di cmdlet PowerShell per HDInsight, vedere [Come installare e configurare Azure PowerShell](/powershell/azure/overview).

L'esempio seguente mostra come applicare un'azione script a un cluster in esecuzione:

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=105-117)]

Al termine dell'operazione, vengono visualizzate informazioni simili alle seguenti:

    OperationState  : Succeeded
    ErrorMessage    :
    Name            : Giraph
    Uri             : https://hdiconfigactions.blob.core.windows.net/linuxgiraphconfigactionv01/giraph-installer-v01.sh
    Parameters      :
    NodeTypes       : {HeadNode, WorkerNode}

### <a name="apply-a-script-action-to-a-running-cluster-from-the-azure-cli"></a>Applicare un'azione script a un cluster in esecuzione dall'interfaccia della riga di comando di Azure

Prima di procedere, assicurarsi di aver installato e configurato l'interfaccia della riga di comando di Azure. Per altre informazioni, vedere [Installare l'interfaccia della riga di comando di Azure 1.0](../cli-install-nodejs.md).

> [!IMPORTANT]
> HDInsight richiede l'interfaccia della riga di comando di Azure 1.0. L'interfaccia della riga di comando di Azure 2.0 non offre attualmente comandi per l'interazione con HDInsight.

1. Per passare alla modalità Azure Resource Manager, usare il comando seguente nella riga di comando:

    ```bash
    azure config mode arm
    ```

2. Usare il comando seguente per eseguire l'autenticazione nella sottoscrizione di Azure.

    ```bash
    azure login
    ```

3. Usare il comando seguente per applicare un'azione script a un cluster in esecuzione

    ```bash
    azure hdinsight script-action create <clustername> -g <resourcegroupname> -n <scriptname> -u <scriptURI> -t <nodetypes>
    ```

    Se non vengono specificati alcuni parametri per il comando, verrà richiesto di specificarli. Se lo script specificato con `-u` accetta parametri, è possibile specificarli usando il parametro `-p`.

    I tipi di nodo validi sono `headnode`, `workernode`, e `zookeeper`. Se lo script deve essere applicato a più tipi di nodo, specificare i tipi separati da ';'. Ad esempio: `-n headnode;workernode`.

    Per salvare lo script in modo permanente, aggiungere `--persistOnSuccess`. È anche possibile salvare lo script in modo permanente in un secondo momento usando `azure hdinsight script-action persisted set`.

    Al termine del processo, viene visualizzato un output simile al testo seguente:

        info:    Executing command hdinsight script-action create
        + Executing Script Action on HDInsight cluster
        data:    Operation Info
        data:    ---------------
        data:    Operation status:
        data:    Operation ID:  b707b10e-e633-45c0-baa9-8aed3d348c13
        info:    hdinsight script-action create command OK

### <a name="apply-a-script-action-to-a-running-cluster-using-rest-api"></a>Applicare un'azione script a un cluster in esecuzione usando le API REST

Vedere l'articolo su come [eseguire azioni script in un cluster in esecuzione](https://msdn.microsoft.com/library/azure/mt668441.aspx).

### <a name="apply-a-script-action-to-a-running-cluster-from-the-hdinsight-net-sdk"></a>Applicare un'azione script a un cluster in esecuzione da HDInsight .NET SDK

Per un esempio relativo all'uso di .NET SDK per applicare script a un cluster, vedere [https://github.com/Azure-Samples/hdinsight-dotnet-script-action](https://github.com/Azure-Samples/hdinsight-dotnet-script-action).

## <a name="view-history-promote-and-demote-script-actions"></a>Visualizzare la cronologia, alzare e abbassare di livello le azioni script

### <a name="using-the-azure-portal"></a>Uso del portale di Azure

1. Nel [portale di Azure](https://portal.azure.com)selezionare il cluster HDInsight.

2. Nella panoramica del cluster HDInsight selezionare il riquadro **Azioni script**.

    ![Riquadro Azioni script](./media/hdinsight-hadoop-customize-cluster-linux/scriptactionstile.png)

   > [!NOTE]
   > È anche possibile selezionare **Tutte le impostazioni** e quindi **Azioni script** dalla sezione Impostazioni.

4. Una cronologia degli script per il cluster viene visualizzata nella sezione Azioni script. Queste informazioni includono un elenco degli script persistenti. Nella schermata seguente si noterà che lo script Solr è stato eseguito nel cluster, la schermata tuttavia non mostra script persistenti.

    ![Sezione Azioni script](./media/hdinsight-hadoop-customize-cluster-linux/script-action-history.png)

5. Selezionando uno script nella cronologia, viene visualizzata la sezione Proprietà per lo script. Nella parte superiore della schermata è possibile eseguire di nuovo lo script o alzarlo di livello.

    ![Proprietà delle azioni script](./media/hdinsight-hadoop-customize-cluster-linux/promote-script-actions.png)

6. È anche possibile usare i puntini di sospensione **...** a destra delle voci nella sezione Azioni script per eseguire alcune operazioni.

    ![Uso di ... nelle azioni script](./media/hdinsight-hadoop-customize-cluster-linux/deletepromoted.png)

### <a name="using-azure-powershell"></a>Uso di Azure PowerShell

| Usare | Per |
| --- | --- |
| Get-AzureRmHDInsightPersistedScriptAction |Recuperare informazioni sulle azioni script persistenti |
| Get-AzureRmHDInsightScriptActionHistory |Recuperare una cronologia delle azioni script applicate al cluster o i dettagli di uno script specifico |
| Set-AzureRmHDInsightPersistedScriptAction |Alzare di livello un'azione script ad hoc per renderla un'azione script persistente |
| Remove-AzureRmHDInsightPersistedScriptAction |Abbassare di livello un'azione script persistente per renderla un'azione script ad hoc |

> [!IMPORTANT]
> Usando `Remove-AzureRmHDInsightPersistedScriptAction` non vengono annullate le azioni eseguite da uno script. Questo cmdlet rimuove solo il flag persistente.

Lo script di esempio seguente mostra come usare i cmdlet per alzare di livello e poi abbassare di livello uno script.

[!code-powershell[main](../../powershell_scripts/hdinsight/use-script-action/use-script-action.ps1?range=123-140)]

### <a name="using-the-azure-cli"></a>Uso dell'interfaccia della riga di comando di Azure

| Usare | Per |
| --- | --- |
| `azure hdinsight script-action persisted list <clustername>` |Recuperare un elenco di azioni script con salvataggio permanente |
| `azure hdinsight script-action persisted show <clustername> <scriptname>` |Recuperare informazioni su una specifica azione script con salvataggio permanente |
| `azure hdinsight script-action history list <clustername>` |Recuperare una cronologia delle azioni script applicate al cluster |
| `azure hdinsight script-action history show <clustername> <scriptname>` |Recuperare informazioni su un'azione script specifica |
| `azure hdinsight script action persisted set <clustername> <scriptexecutionid>` |Alzare di livello un'azione script ad hoc per renderla un'azione script persistente |
| `azure hdinsight script-action persisted delete <clustername> <scriptname>` |Abbassare di livello un'azione script persistente per renderla un'azione script ad hoc |

> [!IMPORTANT]
> Usando `azure hdinsight script-action persisted delete` non vengono annullate le azioni eseguite da uno script. Questo cmdlet rimuove solo il flag persistente.

### <a name="using-the-hdinsight-net-sdk"></a>Uso di HDInsight .NET SDK

Per un esempio relativo all'uso di .NET SDK per recuperare la cronologia degli script da un cluster e alzare o abbassare di livello gli script, vedere [https://github.com/Azure-Samples/hdinsight-dotnet-script-action](https://github.com/Azure-Samples/hdinsight-dotnet-script-action).

> [!NOTE]
> Questo esempio mostra anche come installare un'applicazione HDInsight mediante .NET SDK.

## <a name="support-for-open-source-software-used-on-hdinsight-clusters"></a>Supporto per software open source usato nei cluster HDInsight

Il servizio Microsoft Azure HDInsight usa un ecosistema di tecnologie open source ispirate ad Hadoop. Microsoft Azure offre un livello di supporto generale per le tecnologie open source. Per altre informazioni, vedere la sezione **Ambito del supporto** nel [sito Web delle domande frequenti sul supporto tecnico di Azure](https://azure.microsoft.com/support/faq/). Il servizio HDInsight offre un livello di supporto aggiuntivo per i componenti predefiniti.

Nel servizio HDInsight sono disponibili due tipi di componenti open source:

* **Componenti predefiniti** - Questi componenti sono preinstallati nei cluster HDInsight e forniscono la funzionalità di base del cluster. Questa categoria include ad esempio il gestore risorse YARN, il linguaggio di query Hive (HiveQL) e la libreria Mahout. L'elenco completo dei componenti del cluster è disponibile in [Novità delle versioni cluster di Hadoop incluse in HDInsight](hdinsight-component-versioning.md).
* **Componenti personalizzati** - Un utente del cluster può installare o usare nel carico di lavoro qualsiasi componente disponibile nella community o creato da lui stesso.

> [!WARNING]
> I componenti forniti con il cluster HDInsight sono completamente supportati. Il supporto tecnico Microsoft aiuta a isolare e risolvere i problemi legati a tali componenti.
>
> I componenti personalizzati ricevono supporto commercialmente ragionevole per semplificare la risoluzione dei problemi. Il supporto tecnico Microsoft potrebbe essere in grado di risolvere il problema OPPURE richiedere di usare i canali disponibili per le tecnologie open source, in cui è possibile ottenere supporto estremamente competente per la tecnologia specifica. È ad esempio possibile ricorrere a molti siti di community, come il [forum MSDN per HDInsight](https://social.msdn.microsoft.com/Forums/azure/home?forum=hdinsight) o [http://stackoverflow.com](http://stackoverflow.com). Anche per i progetti Apache sono disponibili siti specifici in [http://apache.org](http://apache.org), ad esempio [Hadoop](http://hadoop.apache.org/).

Il servizio HDInsight permette di usare i componenti personalizzati in molti modi. Indipendentemente dal modo in cui un componente viene usato o installato nel cluster, verrà applicato lo stesso livello di supporto. L'elenco seguente illustra i modi più comuni per usare i componenti personalizzati nei cluster HDInsight:

1. Invio di processi - È possibile inviare al cluster processi Hadoop o di altro tipo che eseguono o usano componenti personalizzati.

2. Personalizzazione del cluster - Durante la creazione di un cluster, è possibile specificare impostazioni aggiuntive e componenti personalizzati, che verranno installati nei nodi del cluster.

3. Esempi - Microsoft e altri utenti possono fornire esempi relativi all'uso dei componenti personalizzati più diffusi nei cluster HDInsight. Questi esempi vengono forniti senza supporto.

## <a name="troubleshooting"></a>risoluzione dei problemi

È possibile usare l'interfaccia utente Web Ambari per visualizzare le informazioni registrate dalle azioni script. Se lo script ha esito negativo durante la creazione del cluster, i log sono disponibili anche nell'account di archiviazione predefinito associato al cluster. Questa sezione fornisce informazioni su come recuperare i registri usando entrambe le opzioni.

### <a name="using-the-ambari-web-ui"></a>Utilizzo dell'interfaccia utente Web Ambari

1. Nel browser passare a https://CLUSTERNAME.azurehdinsight.net. Sostituire CLUSTERNAME con il nome del cluster HDInsight.

    Quando richiesto, immettere il nome dell'account amministratore (admin) e la password per il cluster. Potrebbe essere necessario immettere di nuovo le credenziali di amministratore in un Web Form.

2. Nella barra nella parte superiore della pagina fare clic sulla voce **ops**. Verrà visualizzato un elenco delle operazioni correnti e precedenti eseguite nel cluster tramite Ambari.

    ![Barra nell'interfaccia utente di Ambari con selezionato ops](./media/hdinsight-hadoop-customize-cluster-linux/ambari-nav.png)

3. Trovare le voci con **run\_customscriptaction** nella colonna **Operations**. Queste voci vengono create quando si eseguono le azioni script.

    ![Schermata delle operazioni](./media/hdinsight-hadoop-customize-cluster-linux/ambariscriptaction.png)

    Selezionare la voce run\customscriptaction ed eseguire il drill-down dei collegamenti per visualizzare l'output di STDOUT e STDERR. Questo output viene generato all'esecuzione dello script e può contenere informazioni utili.

### <a name="access-logs-from-the-default-storage-account"></a>Accesso ai registri dall'account di archiviazione predefinito

Se la creazione del cluster non riesce a causa di un errore nello script, i log vengono conservati nell'account di archiviazione cluster.

* I registri di archiviazione sono disponibili in `\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\CLUSTER_NAME\DATE`.

    ![Schermata delle operazioni](./media/hdinsight-hadoop-customize-cluster-linux/script_action_logs_in_storage.png)

    In questa directory i registri sono organizzati per nodi head, nodi del ruolo di lavoro e nodi zookeeper. Di seguito sono riportati alcuni esempi:

    * **Nodo head** - `<uniqueidentifier>AmbariDb-hn0-<generated_value>.cloudapp.net`

    * **Nodo del ruolo di lavoro** - `<uniqueidentifier>AmbariDb-wn0-<generated_value>.cloudapp.net`

    * **Nodo Zookeeper** - `<uniqueidentifier>AmbariDb-zk0-<generated_value>.cloudapp.net`

* Tutti i file stdout e stderr dell'host corrispondente vengono caricati nell'account di archiviazione. Per ogni azione script esiste un file **output-\*.txt** e un file **errors-\*.txt**. Il file output-*.txt contiene informazioni relative all'URI dello script che è stato eseguito nell'host. Il testo seguente è un esempio di queste informazioni:

        'Start downloading script locally: ', u'https://hdiconfigactions.blob.core.windows.net/linuxrconfigactionv01/r-installer-v01.sh'

* È possibile creare più volte un cluster dell'azione di script con lo stesso nome. In questo caso, è possibile distinguere i registri corrispondenti in base al nome della cartella della data. Ad esempio, la struttura di cartelle per un cluster (mycluster) creato in diverse date sarà simile alle seguenti voci di registro:

    `\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\mycluster\2015-10-04``\STORAGE_ACCOUNT_NAME\DEFAULT_CONTAINER_NAME\custom-scriptaction-logs\mycluster\2015-10-05`

* Se in uno stesso giorno si creano più cluster dell'azione di script con lo stesso nome, è possibile usare il prefisso univoco per identificare i file di registro corrispondenti.

* Se si crea un cluster verso le 00.00 (mezzanotte), è possibile che i file di log si estendano su due giorni. In questi casi, per lo stesso cluster vengono visualizzate due diverse cartelle della data.

* Il caricamento dei file di registro nel contenitore predefinito può richiedere fino a 5 minuti, soprattutto per i cluster di grandi dimensioni. Se si desidera accedere ai file di registro, quindi, è opportuno non eliminare immediatamente il cluster in caso di esito negativo di un'azione di script.

### <a name="ambari-watchdog"></a>Watchdog Ambari

> [!WARNING]
> Non modificare la password del watchdog Ambari (hdinsightwatchdog) nel cluster HDInsight basato su Linux. La modifica della password per questo account rende impossibile eseguire nuove azioni script nel cluster HDInsight.

### <a name="cant-import-name-blobservice"></a>Non è possibile importare il nome BlobService

__Sintomi__: l'azione script non riesce. Quando si visualizza l'operazione in Ambari, viene restituito un errore simile al seguente:

```
Traceback (most recent call list):
  File "/var/lib/ambari-agent/cache/custom_actions/scripts/run_customscriptaction.py", line 21, in <module>
    from azure.storage.blob import BlobService
ImportError: cannot import name BlobService
```

__Causa__: questo errore si verifica se si aggiorna il client di archiviazione di Azure Python incluso con il cluster HDInsight. HDInsight prevede l'uso della versione 0.20.0 del client di archiviazione di Azure.

__Risoluzione__: per risolvere questo problema, connettersi manualmente a ciascun nodo del cluster usando `ssh` e usare il comando seguente per reinstallare la versione corretta del client di archiviazione:

```bash
sudo pip install azure-storage==0.20.0
```

Per informazioni sulla connessione al cluster tramite SSH, vedere [Usare SSH con HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

### <a name="history-doesnt-show-scripts-used-during-cluster-creation"></a>La cronologia non mostra gli script usati durante la creazione di un cluster

Se il cluster è stato creato prima del 15 marzo 2016, potrebbe non essere visualizzata una voce nella cronologia delle azioni script. Ridimensionando il cluster, gli script verranno visualizzati nella cronologia delle azioni script.

Sussistono due eccezioni:

* Se il cluster è stato creato prima del 1° settembre 2015. Le azioni script sono state introdotte in questa data. Per i cluster creati prima di tale data non possono quindi essere state usate le azioni script.

* Se sono state usate più azioni script durante la creazione del cluster ed è stato usato lo stesso nome per più script oppure lo stesso nome e lo stesso URI ma parametri diversi per più script. In questi casi si riceve un errore simile al seguente:

    Non è possibile eseguire nuove azioni script nel cluster a causa di nomi di script in conflitto negli script esistenti. I nomi di script forniti durante la creazione del cluster devono essere tutti univoci. Gli script esistenti vengono eseguiti durante il ridimensionamento.

## <a name="next-steps"></a>Passaggi successivi

* [Sviluppare script di azioni script per HDInsight](hdinsight-hadoop-script-actions-linux.md)
* [Installare e usare Solr nei cluster HDInsight](hdinsight-hadoop-solr-install-linux.md)
* [Installare e usare Giraph nei cluster HDInsight](hdinsight-hadoop-giraph-install-linux.md)
* [Add additional storage to an HDInsight cluster](hdinsight-hadoop-add-storage.md) (Aggiungere altra memoria a un cluster HDInsight)

[img-hdi-cluster-states]: ./media/hdinsight-hadoop-customize-cluster-linux/HDI-Cluster-state.png "Fasi durante la creazione di un cluster"
