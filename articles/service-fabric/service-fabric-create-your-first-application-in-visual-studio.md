---
title: Creare un servizio Reliable Services di Azure Service Fabric con C#
description: Creare e distribuire un'applicazione Reliable Services basata su Azure Service Fabric ed eseguirne il debug con Visual Studio.
services: service-fabric
documentationcenter: .net
author: rwike77
manager: timlt
editor: vturecek
ms.assetid: c3655b7b-de78-4eac-99eb-012f8e042109
ms.service: service-fabric
ms.devlang: dotNet
ms.topic: hero-article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 03/14/2018
ms.author: ryanwi
ms.openlocfilehash: 858e322fd7e516f756aa209be92745efa6cf75f7
ms.sourcegitcommit: 8aab1aab0135fad24987a311b42a1c25a839e9f3
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 03/16/2018
---
# <a name="create-your-first-c-service-fabric-stateful-reliable-services-application"></a>Creare la prima applicazione Reliable Services con stato C# di Service Fabric

Questo articolo illustra come distribuire la prima applicazione di Azure Service Fabric per .NET in Windows in pochi minuti. Al termine, si avrà un cluster locale in esecuzione con un'applicazione Reliable Services.

## <a name="prerequisites"></a>prerequisiti

Prima di iniziare, assicurarsi di aver [configurato l'ambiente di sviluppo](service-fabric-get-started.md). Questo processo include l'installazione di Service Fabric SDK e Visual Studio 2017 o 2015.

## <a name="create-the-application"></a>Creazione dell'applicazione

1. Avviare Visual Studio come amministratore.

2. Creare un progetto premendo CTRL+MAIUSC+N.

3. Nella finestra di dialogo **Nuovo progetto** selezionare **Cloud** > **Applicazione di Service Fabric**.

4. Assegnare all'applicazione il nome **MyApplication**. Selezionare **OK**.

   ![Finestra di dialogo Nuovo progetto in Visual Studio][1]

5. Nella finestra di dialogo successiva è possibile creare qualsiasi tipo di applicazione di Service Fabric. Per questa guida introduttiva scegliere **.Net Core 2.0** > **Servizio con stato**.

6. Assegnare al servizio il nome **MyStatefulService**. Selezionare **OK**.

    ![Finestra di dialogo per il nuovo servizio in Visual Studio][2]

    Visual Studio crea il progetto di applicazione e il progetto di servizio con stato e quindi li visualizza in Esplora soluzioni.

    ![Esplora soluzioni dopo la creazione di un'applicazione con un servizio con stato][3]

    Il progetto di applicazione (**MyApplication**) non contiene codice. Fa invece riferimento a un set di progetti di servizio. Include anche altri tre tipi di contenuto:

    * **Profili di pubblicazione**  
    Profili per la distribuzione in diversi ambienti.

    * **Script**  
    Script di PowerShell per la distribuzione o l'aggiornamento dell'applicazione.

    * **Definizione di applicazione**  
Include il file ApplicationManifest.xml in *ApplicationPackageRoot*, che descrive la composizione dell'applicazione. I file di parametri dell'applicazione associati sono disponibili in *ApplicationParameters* e possono essere usati per fornire parametri specifici dell'ambiente. Visual Studio seleziona un file di parametri dell'applicazione specificato nel profilo di pubblicazione associato.
    
Per una panoramica del contenuto del progetto di servizio, vedere la [Guida introduttiva a Reliable Services](service-fabric-reliable-services-quick-start.md).

## <a name="deploy-and-debug-the-application"></a>Distribuire ed eseguire il debug dell'applicazione

Ora che è disponibile un'applicazione, eseguirla, distribuirla ed effettuarne il debug seguendo questa procedura.

1. In Visual Studio premere F5 per distribuire l'applicazione per il debug.

    >[!NOTE]
    >La prima volta che si esegue e si distribuisce l'applicazione in locale, Visual Studio crea un cluster locale per il debug. Questa operazione potrebbe richiedere tempo. Lo stato della creazione del cluster verrà visualizzato nella finestra di output di Visual Studio.

    Quando il cluster è pronto, si riceverà una notifica dall'applicazione dell'area di notifica per la gestione del cluster locale che è inclusa nell'SDK.
    
    ![Notifica della barra delle applicazioni per il cluster locale][4]

    Dopo l'avvio dell'applicazione, Visual Studio apre automaticamente il visualizzatore eventi di diagnostica, in cui viene visualizzato l'output di traccia dei servizi.
    
    ![Visualizzatore eventi di diagnostica][5]

    >[!NOTE]
    >Il rilevamento degli eventi nel visualizzatore eventi di diagnostica dovrebbe essere avviato automaticamente. Se è necessario configurare manualmente il visualizzatore eventi di diagnostica, per prima cosa aprire il file `ServiceEventSource.cs` che si trova nel progetto **MyStatefulService**. Copiare il valore dell'attributo `EventSource` sopra la classe `ServiceEventSource`. Nell'esempio seguente l'origine eventi è denominata `"MyCompany-MyApplication-MyStatefulService"`, ma il nome potrebbe essere diverso nella situazione specifica.
>
    >![Individuazione del nome dell'origine eventi del servizio][service-event-source-name]


2. Successivamente aprire la finestra di dialogo **Provider ETW**, quindi selezionare l'icona a forma di ingranaggio nella scheda **Eventi di diagnostica**. Incollare il nome dell'origine eventi copiato nella casella di input di **Provider ETW**. Selezionare quindi il pulsante **Applica**. Verrà automaticamente avviata la traccia degli eventi.

    ![Impostazione del nome dell'origine eventi di diagnostica][setting-event-source-name]

    Nella finestra Eventi di diagnostica dovrebbero ora essere visualizzati gli eventi.

    Il modello di servizio con stato mostra l'incremento del valore del contatore nel metodo `RunAsync` di **MyStatefulService.cs**.

3. Espandere uno degli eventi per visualizzare altri dettagli, incluso il nodo in cui viene eseguito il codice. In questo caso si tratta di **\_Node\_0**, ma nel computer in uso potrebbe essere diverso.
   
    ![Dettaglio del visualizzatore eventi di diagnostica][6]

4. Il cluster locale contiene cinque nodi ospitati in un singolo computer. In un ambiente di produzione, ogni nodo è ospitato in una macchina virtuale o un computer fisico distinto. Per simulare la perdita di un computer mentre si prova a eseguire il debugger di Visual Studio, disattivare uno dei nodi nel cluster locale.

5. Nella finestra **Esplora soluzioni** aprire **MyStatefulService.cs**. 

6. Trovare il metodo `RunAsync` e quindi impostare un punto di interruzione nella prima riga del metodo.

    ![Punto di interruzione nel metodo RunAsync del servizio con stato][7]

7. Avviare lo strumento Service Fabric Explorer facendo clic con il pulsante destro del mouse sull'applicazione dell'area di notifica **Local Cluster Manager** (Gestione cluster locale) e quindi scegliendo **Manage Local Cluster** (Gestisci cluster locale).

    ![Avvio di Service Fabric Explorer da Local Cluster Manager (Gestione cluster locale)][systray-launch-sfx]

    [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) offre una rappresentazione visiva del cluster, che include il set di applicazioni distribuito al suo interno e il set di nodi fisici da cui è costituito.

8. Nel riquadro sinistro espandere **Cluster** > **Nodi** e trovare il nodo in cui è in esecuzione il codice. Per simulare un riavvio del computer, selezionare **Azioni** > **Disattiva (riavvio)**.

    ![Arresto di un nodo in Service Fabric Explorer][sfx-stop-node]

    Verrà visualizzato il raggiungimento del punto di interruzione in Visual Studio mentre viene effettuato il failover del calcolo eseguito su un nodo in un altro nodo.

9. Tornare quindi al visualizzatore eventi di diagnostica e osservare i messaggi. Il contatore ha continuato ad aumentare anche se gli eventi provengono in effetti da un nodo diverso.

    ![Visualizzatore eventi di diagnostica dopo il failover][diagnostic-events-viewer-detail-post-failover]

## <a name="clean-up-the-local-cluster-optional"></a>Eseguire la pulizia del cluster locale (facoltativo)

Tenere presente che il cluster locale è reale. L'arresto del debugger rimuove l'istanza dell'applicazione e annulla la registrazione del tipo di applicazione. L'esecuzione del cluster, tuttavia, continua in background. Quando si è pronti per arrestare il cluster locale, è possibile procedere in due modi.

### <a name="keep-application-and-trace-data"></a>Mantenere i dati applicazione e di traccia

Arrestare il cluster facendo clic con il pulsante destro del mouse sull'applicazione dell'area di notifica **Local Cluster Manager** (Gestione cluster locale) e quindi scegliendo **Stop Local Cluster** (Arresta cluster locale).

### <a name="delete-the-cluster-and-all-data"></a>Eliminare il cluster e tutti i dati

Rimuovere il cluster facendo clic con il pulsante destro del mouse sull'applicazione dell'area di notifica **Local Cluster Manager** (Gestione cluster locale). Scegliere quindi **Remove Local Cluster** (Rimuovi cluster locale). 

Se si sceglie questa opzione, Visual Studio ridistribuirà il cluster alla successiva esecuzione dell'applicazione. Scegliere questa opzione se non si prevede di usare il cluster locale per un certo periodo o se è necessario recuperare risorse.

## <a name="next-steps"></a>Passaggi successivi
Altre informazioni su [Reliable Services](service-fabric-reliable-services-introduction.md).
<!-- Image References -->

[1]: ./media/service-fabric-create-your-first-application-in-visual-studio/new-project-dialog.png
[2]: ./media/service-fabric-create-your-first-application-in-visual-studio/new-project-dialog-2.png
[3]: ./media/service-fabric-create-your-first-application-in-visual-studio/solution-explorer-stateful-service-template.png
[4]: ./media/service-fabric-create-your-first-application-in-visual-studio/local-cluster-manager-notification.png
[5]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer.png
[6]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer-detail.png
[7]: ./media/service-fabric-create-your-first-application-in-visual-studio/runasync-breakpoint.png
[sfx-stop-node]: ./media/service-fabric-create-your-first-application-in-visual-studio/sfe-deactivate-node.png
[systray-launch-sfx]: ./media/service-fabric-create-your-first-application-in-visual-studio/launch-sfx.png
[diagnostic-events-viewer-detail-post-failover]: ./media/service-fabric-create-your-first-application-in-visual-studio/diagnostic-events-viewer-detail-post-failover.png
[sfe-delete-application]: ./media/service-fabric-create-your-first-application-in-visual-studio/sfe-delete-application.png
[switch-cluster-mode]: ./media/service-fabric-create-your-first-application-in-visual-studio/switch-cluster-mode.png
[cluster-setup-success-1-node]: ./media/service-fabric-get-started-with-a-local-cluster/cluster-setup-success-1-node.png
[service-event-source-name]: ./media/service-fabric-create-your-first-application-in-visual-studio/event-source-attribute-value.png
[setting-event-source-name]: ./media/service-fabric-create-your-first-application-in-visual-studio/setting-event-source-name.png
[switch-cluster-mode]: ./media/service-fabric-create-your-first-application-in-visual-studio/switch-cluster-mode.png
[cluster-setup-success-1-node]: ./media/service-fabric-get-started-with-a-local-cluster/cluster-setup-success-1-node.png
[service-event-source-name]: ./media/service-fabric-create-your-first-application-in-visual-studio/event-source-attribute-value.png
[setting-event-source-name]: ./media/service-fabric-create-your-first-application-in-visual-studio/setting-event-source-name.png
