---
title: Archiviare i log di diagnostica di Azure | Microsoft Docs
description: Informazioni su come archiviare il log di diagnostica di Azure per la conservazione a lungo termine in un account di archiviazione.
author: johnkemnetz
manager: orenr
editor: ''
services: monitoring-and-diagnostics
documentationcenter: monitoring-and-diagnostics
ms.assetid: 3a55c73f-2ef3-45f3-8956-bcf9c0cb7e05
ms.service: monitoring-and-diagnostics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/04/2018
ms.author: johnkem
ms.openlocfilehash: bf776ba8aaeca361250f39fb2c62233ee1dfbd5b
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/06/2018
---
# <a name="archive-azure-diagnostic-logs"></a>Archiviare i log di diagnostica di Azure

Questo articolo illustra come è possibile usare il portale di Azure, i cmdlet di PowerShell, l'interfaccia della riga di comando o l'API REST per archiviare i [log di diagnostica di Azure](monitoring-overview-of-diagnostic-logs.md) in un account di archiviazione. Questa opzione è utile per conservare i log di diagnostica con criteri di conservazione facoltativi per il controllo, l'analisi statica o il backup. L'account di archiviazione non deve trovarsi nella stessa sottoscrizione della risorsa che emette log, purché l'utente che configura l'impostazione abbia un accesso RBAC appropriato a entrambe le sottoscrizioni.

## <a name="prerequisites"></a>prerequisiti

Prima di iniziare, è necessario [creare un account di archiviazione](../storage/storage-create-storage-account.md) in cui archiviare i log di diagnostica. È consigliabile non usare un account di archiviazione esistente in cui sono archiviati altri dati non di monitoraggio, per poter controllare meglio l'accesso ai dati di monitoraggio. Se tuttavia in un account di archiviazione si archiviano anche il log attività e le metriche di diagnostica, può avere senso usare tale account di archiviazione anche per i log di diagnostica per tenere tutti i dati di monitoraggio in una posizione centrale. L'account di archiviazione usato deve essere un account di archiviazione per utilizzo generico, non un account di archiviazione BLOB.

## <a name="diagnostic-settings"></a>Impostazioni di diagnostica

Per archiviare i log di diagnostica usando uno dei metodi indicati di seguito, definire una **impostazione di diagnostica** per una risorsa specifica. Un'impostazione di diagnostica per una risorsa definisce le categorie dei log e di dati di metrica inviati a una destinazione (account di archiviazione, spazio dei nomi di Hub eventi o Log Analytics). Definisce anche i criteri di conservazione (numero di giorni di conservazione) per gli eventi di ogni categoria di log e dati di metrica archiviati in un account di archiviazione. Se un criterio di conservazione è impostato su zero, gli eventi per tale categoria di log vengono archiviati per un periodo illimitato (per sempre). Diversamente, un criterio di conservazione può essere qualsiasi numero di giorni compreso tra 1 e 2147483647. [Altre informazioni sulle impostazioni di diagnostica sono disponibili qui](monitoring-overview-of-diagnostic-logs.md#resource-diagnostic-settings). I criteri di conservazione vengono applicati su base giornaliera. Al termine della giornata, i log relativi a tale giornata non rientrano quindi più nei criteri di conservazione e verranno eliminati. Se, ad esempio, è presente un criterio di conservazione di un giorno, all'inizio della giornata attuale vengono eliminati i log relativi alla giornata precedente a ieri.

> [!NOTE]
> L'invio delle metriche multidimensionali tramite impostazioni di diagnostica non è attualmente supportato. Le metriche con dimensioni sono esportate come metriche a singola dimensione di tipo flat e aggregate a livello di valori di dimensione.
>
> *Ad esempio*: la metrica 'Messaggi in ingresso' su un hub eventi può essere esplorata e rappresentata in un grafico a livello di singola coda. Tuttavia, in caso di esportazione tramite impostazione di diagnostica, la metrica verrà rappresentata come tutti i messaggi in ingresso in tutte le code nell'hub eventi.
>
>

## <a name="archive-diagnostic-logs-using-the-portal"></a>Archiviare i log di diagnostica tramite il portale

1. Nel portale passare a Monitoraggio di Azure e fare clic su **Impostazioni di diagnostica**

    ![Sezione relativa al monitoraggio di Monitoraggio di Azure](media/monitoring-archive-diagnostic-logs/diagnostic-settings-blade.png)

2. Facoltativamente filtrare l'elenco in base al tipo di risorsa o al gruppo di risorse, quindi fare clic sulla risorsa per cui si vuole specificare un'impostazione di diagnostica.

3. Se non esiste un'impostazione sulla risorsa selezionata, viene chiesto di creare un'impostazione. Fare clic su "Attiva diagnostica".

   ![Aggiungi impostazione di diagnostica - Nessuna impostazione esistente](media/monitoring-archive-diagnostic-logs/diagnostic-settings-none.png)

   Se esistono già impostazioni sulla risorsa, verrò visualizzato un elenco di impostazioni già configurate per questa risorsa. Fare clic su "Add diagnostic setting" (Aggiungi impostazione di diagnostica).

   !["Add diagnostic setting" (Aggiungi impostazione di diagnostica) - impostazioni esistenti](media/monitoring-archive-diagnostic-logs/diagnostic-settings-multiple.png)

3. Assegnare un nome all'impostazione, selezionare la casella per **Esporta in Account di archiviazione**, quindi selezionare un account di archiviazione. Facoltativamente, impostare un numero di giorni per la conservazione di questi log usando i dispositivi di scorrimento **Conservazione (giorni)** . Se il valore di conservazione è zero giorni, i log vengono conservati all'infinito.

   !["Add diagnostic setting" (Aggiungi impostazione di diagnostica) - impostazioni esistenti](media/monitoring-archive-diagnostic-logs/diagnostic-settings-configure.png)

4. Fare clic su **Save**.

Dopo qualche istante, la nuova impostazione viene visualizzata nell'elenco delle impostazioni per questa risorsa e i log di diagnostica vengono archiviati in tale account di archiviazione non appena vengono generati nuovi dati di eventi.

## <a name="archive-diagnostic-logs-via-azure-powershell"></a>Archiviare i log di diagnostica tramite Azure PowerShell

```
Set-AzureRmDiagnosticSetting -ResourceId /subscriptions/s1id1234-5679-0123-4567-890123456789/resourceGroups/testresourcegroup/providers/Microsoft.Network/networkSecurityGroups/testnsg -StorageAccountId /subscriptions/s1id1234-5679-0123-4567-890123456789/resourceGroups/myrg1/providers/Microsoft.Storage/storageAccounts/my_storage -Categories networksecuritygroupevent,networksecuritygrouprulecounter -Enabled $true -RetentionEnabled $true -RetentionInDays 90
```

| Proprietà | Obbligatoria | DESCRIZIONE |
| --- | --- | --- |
| ResourceId |Sì |ID risorsa della risorsa in cui si vuole definire un'impostazione di diagnostica. |
| StorageAccountId |No  |ID risorsa dell'account di archiviazione in cui devono essere salvati i log di diagnostica. |
| Categorie |No  |Elenco delimitato da virgole di categorie di log da abilitare. |
| Attivato |Sì |Valore booleano che indica se la diagnostica viene abilitata o disabilitata per questa risorsa. |
| RetentionEnabled |No  |Valore booleano che indica se un criterio di conservazione è abilitato per questa risorsa. |
| RetentionInDays |No  |Numero di giorni per cui gli eventi devono essere conservati, compreso tra 1 e 2147483647. Se il valore è zero, i log vengono conservati all'infinito. |

## <a name="archive-diagnostic-logs-via-the-azure-cli-20"></a>Archiviare i log di diagnostica tramite l'interfaccia della riga di comando di Azure 2.0

```azurecli
az monitor diagnostic-settings create --name <diagnostic name> \
    --storage-account <name or ID of storage account> \
    --resource <target resource object ID> \
    --resource-group <storage account resource group> \
    --logs '[
    {
        "category": <category name>,
        "enabled": true,
        "retentionPolicy": {
            "days": <# days to retain>,
            "enabled": true
        }
    }]'
```

È possibile aggiungere altre categorie al log di diagnostica aggiungendo dizionari alla matrice JSON passata come parametro `--logs`.

L'argomento `--resource-group` è obbligatorio solo se `--storage-account` non è un ID oggetto. Per la documentazione completa relativa all'archiviazione dei log di diagnostica nella risorsa di archiviazione, vedere le informazioni di [riferimento sui comandi dell'interfaccia della riga di comando](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create).

## <a name="archive-diagnostic-logs-via-the-rest-api"></a>Archiviare i log di diagnostica tramite l'API REST

[Vedere questo documento](https://docs.microsoft.com/rest/api/monitor/servicediagnosticsettings) per informazioni su come è possibile configurare un'impostazione di diagnostica usando l'API REST di Monitoraggio di Azure.

## <a name="schema-of-diagnostic-logs-in-the-storage-account"></a>Schema dei log di diagnostica nell'account di archiviazione

Dopo aver configurato l'archiviazione, viene creato un contenitore di archiviazione nell'account di archiviazione non appena si verifica un evento in una delle categorie di log abilitate. I BLOB nel contenitore seguono lo stesso formato nei log di diagnostica e nel log attività. La struttura di questi BLOB è:

> insights-logs-{log category name}/resourceId=/SUBSCRIPTIONS/{subscription ID}/RESOURCEGROUPS/{nome gruppo risorse}/PROVIDERS/{nome provider risorse}/{tipo risorsa}/{nome risorsa}/y={anno numerico a quattro cifre}/m={mese numerico a due cifre}/d={giorno numerico a due cifre}/h={ora in formato 24 ore a due cifre}/m=00/PT1H.json

O più semplicemente,

> insights-logs-{nome categoria log}/resourceId=/{ID risorsa}/y={anno numerico a quattro cifre}/m={mese numerico a due cifre}/d={giorno numerico a due cifre}/h={ora in formato 24 ore a due cifre}/m=00/PT1H.json

Ad esempio, un nome BLOB potrebbe essere:

> insights-logs-networksecuritygrouprulecounter/resourceId=/SUBSCRIPTIONS/s1id1234-5679-0123-4567-890123456789/RESOURCEGROUPS/TESTRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUP/TESTNSG/y=2016/m=08/d=22/h=18/m=00/PT1H.json

Ogni BLOB PT1H.json contiene un BLOB JSON di eventi che si sono verificati nell'ora specificata nell'URL BLOB (ad esempio, h=12). Durante l'ora attuale, gli eventi vengono aggiunti al file PT1H.json man mano che si verificano. Il valore dei minuti (m=00) è sempre 00, perché gli eventi del log di diagnostica vengono sempre suddivisi in singoli BLOB per ogni ora.

Nel file PT1H.json ogni evento viene archiviato nella matrice "records", con questo formato:

```
{
    "records": [
        {
            "time": "2016-07-01T00:00:37.2040000Z",
            "systemId": "46cdbb41-cb9c-4f3d-a5b4-1d458d827ff1",
            "category": "NetworkSecurityGroupRuleCounter",
            "resourceId": "/SUBSCRIPTIONS/s1id1234-5679-0123-4567-890123456789/RESOURCEGROUPS/TESTRESOURCEGROUP/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/TESTNSG",
            "operationName": "NetworkSecurityGroupCounters",
            "properties": {
                "vnetResourceGuid": "{12345678-9012-3456-7890-123456789012}",
                "subnetPrefix": "10.3.0.0/24",
                "macAddress": "000123456789",
                "ruleName": "/subscriptions/ s1id1234-5679-0123-4567-890123456789/resourceGroups/testresourcegroup/providers/Microsoft.Network/networkSecurityGroups/testnsg/securityRules/default-allow-rdp",
                "direction": "In",
                "type": "allow",
                "matchedConnections": 1988
            }
        }
    ]
}
```

| Nome dell'elemento | DESCRIZIONE |
| --- | --- |
| time |Timestamp del momento in cui l'evento è stato generato dal servizio di Azure che ha elaborato la richiesta corrispondente all'evento. |
| ResourceId |ID risorsa della risorsa interessata. |
| operationName |Nome dell'operazione. |
| category |Categoria di log dell'evento. |
| properties |Set di coppie `<Key, Value>` ad esempio Dictionary, che descrivono i dettagli dell'evento. |

> [!NOTE]
> Le proprietà e l'utilizzo di tali proprietà possono variare a seconda della risorsa.

## <a name="next-steps"></a>Passaggi successivi

* [Introduzione all'archivio BLOB di Azure con .NET](../storage/storage-dotnet-how-to-use-blobs.md)
* [Trasmettere log di diagnostica di Azure a uno spazio di nomi di Hub eventi](monitoring-stream-diagnostic-logs-to-event-hubs.md)
* [Altre informazioni sui log di diagnostica](monitoring-overview-of-diagnostic-logs.md)
