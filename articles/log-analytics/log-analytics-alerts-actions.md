---
title: Risposte agli avvisi in Log Analytics di Azure | Documentazione Microsoft
description: Gli avvisi in Log Analytics identificano le informazioni importanti nell'area di lavoro di Azure e possono notificare i problemi all'utente in modo proattivo o richiamare le azioni per tentare di correggerle.  Questo articolo descrive come creare una regola di avviso e include i dettagli relativi alle diverse azioni che possono attivare.
services: log-analytics
documentationcenter: ''
author: bwren
manager: jwhit
editor: tysonn
ms.assetid: 6cfd2a46-b6a2-4f79-a67b-08ce488f9a91
ms.service: log-analytics
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/13/2018
ms.author: bwren
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 717adf1b19b9de8542ec507df3a01b187d0df8a5
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/18/2018
---
# <a name="add-actions-to-alert-rules-in-log-analytics"></a>Aggiungere azioni alle regole di avviso in Log Analytics

> [!NOTE]
> Gli avvisi in Log Analytics sono stati [estesi in Azure](../monitoring-and-diagnostics/monitoring-alerts-extend.md).  Per definire le azioni degli avvisi in Azure, vengono usati i [gruppi di azioni](../monitoring-and-diagnostics/monitoring-action-groups.md) anziché le informazioni fornite in questo articolo.


Quando [viene creato un avviso in Log Analytics](log-analytics-alerts.md), è possibile scegliere di [configurare la regola di avviso](log-analytics-alerts.md) per eseguire una o più azioni.  Questo articolo descrive le diverse azioni disponibili e offre informazioni sulla configurazione di ogni tipologia di azione.

| Azione | DESCRIZIONE |
|:--|:--|
| [Indirizzo di posta elettronica](#email-actions) | Inviare un messaggio di posta elettronica con i dettagli dell'avviso a uno o più destinatari. |
| [Webhook](#webhook-actions) | Richiamare un processo esterno tramite una singola richiesta HTTP POST. |
| [Runbook](#runbook-actions) | Avviare un runbook in Automazione di Azure. |


## <a name="email-actions"></a>Azioni di posta elettronica
Le azioni di posta elettronica inviano un messaggio di posta elettronica con i dettagli dell'avviso a uno o più destinatari.  È possibile specificare l'oggetto del messaggio, ma il contenuto è un formato standard creato da Log Analytics.  Include informazioni di riepilogo, come il nome dell'avviso, oltre ai dettagli di un massimo di dieci record restituiti dalla ricerca nei log.  Include anche un collegamento a una ricerca nei log in Log Analytics che restituisce l'intero set di record dalla query.   Il mittente del messaggio è *Team di Microsoft Operations Management Suite &lt;noreply@oms.microsoft.com&gt;*. 

Le azioni di posta elettronica includono le proprietà elencate nella tabella seguente.

| Proprietà | DESCRIZIONE |
|:--- |:--- |
| Oggetto |Oggetto nel messaggio di posta elettronica.  È possibile modificare il corpo del messaggio. |
| Destinatari |Indirizzi di tutti i destinatari di posta elettronica.  Se si specifica più di un indirizzo, separare ognuno con un punto e virgola (;). |


## <a name="webhook-actions"></a>Azioni webhook

Le azioni Webhook consentono di richiamare un processo esterno tramite una singola richiesta HTTP POST.  Il servizio chiamato deve supportare i webhook e determinare come usare gli eventuali payload che riceve.  È anche possibile chiamare un'API REST che non supporta in modo specifico i webhook, purché la richiesta sia in un formato l'API riconosce.  Esempi di utilizzo di un webhook in risposta a un avviso sono l'invio di un messaggio in [Slack](http://slack.com) o la creazione di un evento imprevisto in [PagerDuty](http://pagerduty.com/).  Una procedura dettagliata completa della creazione di una regola di avviso con un webhook per chiamare Slack è disponibile nell'articolo relativo ai [webhook negli avvisi di Log Analytics](log-analytics-alerts-webhooks.md).

Le azioni webhook includono le proprietà elencate nella tabella seguente.

| Proprietà | DESCRIZIONE |
|:--- |:--- |
| URL webhook |URL del webhook. |
| Payload JSON personalizzato |Payload personalizzato da inviare insieme al webhook.  Vedere di seguito per altri dettagli. |


I webhook includono un URL e un payload in fermato JSON che corrisponde ai dati inviati al servizio esterno.  Per impostazione predefinita, il payload include i valori riportati nella tabella seguente.  È possibile scegliere di sostituire questo payload con un payload personalizzato.  In questo caso è possibile usare le variabili nella tabella per ognuno dei parametri per includerne il valore nel payload personalizzato.


| Parametro | Variabile | DESCRIZIONE |
|:--- |:--- |:--- |
| AlertRuleName |#alertrulename |Nome della regola di avviso. |
| AlertThresholdOperator |#thresholdoperator |Operatore di soglia per la regola di avviso.  *Maggiore di* o *Minore di*. |
| AlertThresholdValue |#thresholdvalue |Valore di soglia per la regola di avviso. |
| LinkToSearchResults |#linktosearchresults |Collegamento alla ricerca nei log di Log Analytics che restituisce i record della query che ha creato l'avviso. |
| ResultCount |#searchresultcount |Numero di record nei risultati della ricerca. |
| SearchIntervalEndtimeUtc |#searchintervalendtimeutc |Ora di fine per la query in formato UTC. |
| SearchIntervalInSeconds |#searchinterval |Intervallo di tempo per la regola di avviso. |
| SearchIntervalStartTimeUtc |#searchintervalstarttimeutc |Ora di inizio per la query in formato UTC. |
| SearchQuery |#searchquery |Query di ricerca nei log usata dalla regola di avviso. |
| SearchResults |Vedere di seguito |Record restituiti dalla query in formato JSON.  Limitati ai primi 5.000 record. |
| WorkspaceID |#workspaceid |ID dell'area di lavoro di Log Analytics. |

Ad esempio, è possibile specificare il payload personalizzato seguente che include un singolo parametro denominato *text*.  Il servizio chiamato da questo webhook si aspetta questo parametro.

    {
        "text":"#alertrulename fired with #searchresultcount over threshold of #thresholdvalue."
    }

Il payload di esempio viene risolto in una stringa di simile alla seguente quando viene inviato al webhook.

    {
        "text":"My Alert Rule fired with 18 records over threshold of 10 ."
    }

Per includere i risultati della ricerca in un payload personalizzato, aggiungere la riga seguente come proprietà di primo livello nel payload JSON.  

    "IncludeSearchResults":true

Ad esempio, per creare un payload personalizzato che include solo il nome dell'avviso e i risultati della ricerca, è possibile usare quanto segue. 

    {
       "alertname":"#alertrulename",
       "IncludeSearchResults":true
    }


Per un esempio completo di creazione di una regola di avviso con un webhook per avviare un servizio esterno, vedere l'articolo [Creare un'azione di avviso webhook in Log Analytics di OMS per inviare messaggi a Slack](log-analytics-alerts-webhooks.md).


## <a name="runbook-actions"></a>Azioni runbook
Le azioni runbook avviano un runbook in Automazione di Azure.  Per usare questo tipo di azione, è necessario che la [soluzione di automazione](log-analytics-add-solutions.md) sia installata e configurata nell'area di lavoro di Log Analytics.  È possibile selezionare i runbook nell'account di automazione che è stato configurato nella soluzione di automazione.

Le azioni runbook includono le proprietà elencate nella tabella seguente.

| Proprietà | DESCRIZIONE |
|:--- |:---|
| Runbook | Runbook da avviare quando viene creato un avviso. |
| Esegui in | Specificare **Azure** per eseguire il runbook nel cloud.  Specificare **Ruolo di lavoro ibrido** per eseguire il runbook in un agente in cui è installato un [ruolo di lavoro ibrido per runbook](../automation/automation-hybrid-runbook-worker.md ).  |

Le azioni runbook avviano il runbook tramite un [webhook](../automation/automation-webhooks.md).  Quando si crea la regola di avviso, viene creato automaticamente un nuovo webhook per il runbook con il nome **OMS Alert Remediation** seguito da un GUID.  

Non è possibile popolare direttamente alcun parametro del runbook, ma il [parametro $WebhookData](../automation/automation-webhooks.md) include i dettagli dell'avviso, inclusi i risultati della ricerca nei log che lo ha creato.  Il runbook deve definire **$WebhookData** come parametro per consentire l'accesso alle proprietà dell'avviso.  I dati dell'avviso sono disponibili in formato json in una singola proprietà denominata **SearchResult** (per le azioni runbook e le azioni webhook con payload standard) o **SearchResults** (azioni webhook con payload personalizzato tra cui **IncludeSearchResults":true**) nella proprietà **RequestBody** di **$WebhookData**.  Sono incluse le proprietà riportate nella tabella seguente.

>[!NOTE]
> Se l'area di lavoro è stata aggiornata al [nuovo linguaggio di query di Log Analytics](log-analytics-log-search-upgrade.md), il payload del runbook è stato modificato.  Vedere [API REST di Azure Log Analytics](https://aka.ms/loganalyticsapiresponse) per informazioni dettagliate sul formato.  Nella sezione [Esempi](#sample-payload) viene illustrato un esempio.  

| Nodo | DESCRIZIONE |
|:--- |:--- |
| id |Percorso e GUID della ricerca. |
| __metadata |Informazioni sull'avviso, inclusi il numero di record e lo stato dei risultati della ricerca. |
| value |Voce separata per ogni record nei risultati della ricerca.  I dettagli della voce corrispondono alle proprietà e ai valori del record. |

Ad esempio, i runbook seguenti estraggono i record restituiti dalla ricerca nel log e assegnano proprietà diverse in base al tipo di ogni record.  Si noti che il runbook converte prima di tutto **RequestBody** da JSON, in modo che possa essere usato come oggetto in PowerShell.

>[!NOTE]
> Questo runbook usa **SearchResult**, che è la proprietà che contiene i risultati per le azioni runbook e le azioni webhook con payload standard.  Se il runbook è stato chiamato da una risposta webhook usando un payload personalizzato, è necessario cambiare questa proprietà in **SearchResults**.

    param ( 
        [object]$WebhookData
    )

    $RequestBody = ConvertFrom-JSON -InputObject $WebhookData.RequestBody

    # Get all metadata properties    
    $AlertRuleName = $RequestBody.AlertRuleName
    $AlertThresholdOperator = $RequestBody.AlertThresholdOperator
    $AlertThresholdValue = $RequestBody.AlertThresholdValue
    $AlertDescription = $RequestBody.Description
    $LinktoSearchResults =$RequestBody.LinkToSearchResults
    $ResultCount =$RequestBody.ResultCount
    $Severity = $RequestBody.Severity
    $SearchQuery = $RequestBody.SearchQuery
    $WorkspaceID = $RequestBody.WorkspaceId
    $SearchWindowStartTime = $RequestBody.SearchIntervalStartTimeUtc
    $SearchWindowEndTime = $RequestBody.SearchIntervalEndtimeUtc
    $SearchWindowInterval = $RequestBody.SearchIntervalInSeconds

    # Get detailed search results
    if($RequestBody.SearchResult -ne $null)
    {
        $SearchResultRows    = $RequestBody.SearchResult.tables[0].rows 
        $SearchResultColumns = $RequestBody.SearchResult.tables[0].columns;

        foreach ($SearchResultRow in $SearchResultRows)
        {   
            $Column = 0
            $Record = New-Object –TypeName PSObject 
        
            foreach ($SearchResultColumn in $SearchResultColumns)
            {
                $Name = $SearchResultColumn.name
                $ColumnValue = $SearchResultRow[$Column]
                $Record | Add-Member –MemberType NoteProperty –Name $name –Value $ColumnValue -Force
                        
                $Column++
            }

            # Include code to work with the record. 
            # For example $Record.Computer to get the computer property from the record.
            
        }
    }



## <a name="sample-payload"></a>Esempio di payload
Questa sezione illustra un payload di esempio per le azioni webhook e runbook.

### <a name="webhook-actions"></a>Azioni webhook
Questo esempio usa **SearchResult**, che è la proprietà che contiene i risultati per le azioni webhook con payload standard.  Se il webhook usasse un payload personalizzato che include i risultati della ricerca, questa proprietà sarebbe **SearchResults**.

Di seguito è riportato un esempio di payload per un'azione webhook.

    {
    "WorkspaceId": "workspaceID",
    "AlertRuleName": "WebhookAlert",
    "SearchQuery": "Usage",
    "SearchResult": {
        "tables": [
        {
            "name": "PrimaryResult",
            "columns": [
            {
                "name": "TenantId",
                "type": "string"
            },
            {
                "name": "Computer",
                "type": "string"
            },
            {
                "name": "TimeGenerated",
                "type": "datetime"
            },
            {
                "name": "SourceSystem",
                "type": "string"
            },
            {
                "name": "StartTime",
                "type": "datetime"
            },
            {
                "name": "EndTime",
                "type": "datetime"
            },
            {
                "name": "ResourceUri",
                "type": "string"
            },
            {
                "name": "LinkedResourceUri",
                "type": "string"
            },
            {
                "name": "DataType",
                "type": "string"
            },
            {
                "name": "Solution",
                "type": "string"
            },
            {
                "name": "BatchesWithinSla",
                "type": "long"
            },
            {
                "name": "BatchesOutsideSla",
                "type": "long"
            },
            {
                "name": "BatchesCapped",
                "type": "long"
            },
            {
                "name": "TotalBatches",
                "type": "long"
            },
            {
                "name": "AvgLatencyInSeconds",
                "type": "real"
            },
            {
                "name": "Quantity",
                "type": "real"
            },
            {
                "name": "QuantityUnit",
                "type": "string"
            },
            {
                "name": "IsBillable",
                "type": "bool"
            },
            {
                "name": "MeterId",
                "type": "string"
            },
            {
                "name": "LinkedMeterId",
                "type": "string"
            },
            {
                "name": "Type",
                "type": "string"
            }
            ],
            "rows": [
            [
                "workspaceID",
                "-",
                "2017-09-26T13:59:59Z",
                "OMS",
                "2017-09-26T13:00:00Z",
                "2017-09-26T13:59:59Z",
                "/subscriptions/SubscriptionID/resourcegroups/ResourceGroupName/providers/microsoft.operationalinsights/workspaces/workspace-workspaceID",
                null,
                "Operation",
                "LogManagement",
                8,
                0,
                0,
                8,
                0,
                0.002502,
                "MBytes",
                false,
                "a4e29a95-5b4c-408b-80e3-113f9410566e",
                "00000000-0000-0000-0000-000000000000",
                "Usage"
            ]
            ]
        }
        ]
    },
    "SearchIntervalStartTimeUtc": "2017-09-26T08:10:40Z",
    "SearchIntervalEndtimeUtc": "2017-09-26T09:10:40Z",
    "AlertThresholdOperator": "Greater Than",
    "AlertThresholdValue": 0,
    "ResultCount": 1,
    "SearchIntervalInSeconds": 3600,
    "LinkToSearchResults": "https://workspaceID.portal.mms.microsoft.com/#Workspace/search/index?_timeInterval.intervalEnd=2017-09-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
    "Description": null,
    "Severity": "Low"
    }


### <a name="runbooks"></a>Runbook

Di seguito è riportato un esempio di payload per un'azione runbook.

    {
    "WorkspaceId": "workspaceID",
    "AlertRuleName": "AutomationAlert",
    "SearchQuery": "Usage",
    "SearchResult": {
        "tables": [
        {
            "name": "PrimaryResult",
            "columns": [
            {
                "name": "TenantId",
                "type": "string"
            },
            {
                "name": "Computer",
                "type": "string"
            },
            {
                "name": "TimeGenerated",
                "type": "datetime"
            },
            {
                "name": "SourceSystem",
                "type": "string"
            },
            {
                "name": "StartTime",
                "type": "datetime"
            },
            {
                "name": "EndTime",
                "type": "datetime"
            },
            {
                "name": "ResourceUri",
                "type": "string"
            },
            {
                "name": "LinkedResourceUri",
                "type": "string"
            },
            {
                "name": "DataType",
                "type": "string"
            },
            {
                "name": "Solution",
                "type": "string"
            },
            {
                "name": "BatchesWithinSla",
                "type": "long"
            },
            {
                "name": "BatchesOutsideSla",
                "type": "long"
            },
            {
                "name": "BatchesCapped",
                "type": "long"
            },
            {
                "name": "TotalBatches",
                "type": "long"
            },
            {
                "name": "AvgLatencyInSeconds",
                "type": "real"
            },
            {
                "name": "Quantity",
                "type": "real"
            },
            {
                "name": "QuantityUnit",
                "type": "string"
            },
            {
                "name": "IsBillable",
                "type": "bool"
            },
            {
                "name": "MeterId",
                "type": "string"
            },
            {
                "name": "LinkedMeterId",
                "type": "string"
            },
            {
                "name": "Type",
                "type": "string"
            }
            ],
            "rows": [
            [
                "861bd466-5400-44be-9552-5ba40823c3aa",
                "-",
                "2017-09-26T13:59:59Z",
                "OMS",
                "2017-09-26T13:00:00Z",
                "2017-09-26T13:59:59Z",
            "/subscriptions/SubscriptionID/resourcegroups/ResourceGroupName/providers/microsoft.operationalinsights/workspaces/workspace-861bd466-5400-44be-9552-5ba40823c3aa",
                null,
                "Operation",
                "LogManagement",
                8,
                0,
                0,
                8,
                0,
                0.002502,
                "MBytes",
                false,
                "a4e29a95-5b4c-408b-80e3-113f9410566e",
                "00000000-0000-0000-0000-000000000000",
                "Usage"
            ]
            ]
        }
        ]
    },
    "SearchIntervalStartTimeUtc": "2017-09-26T08:10:40Z",
    "SearchIntervalEndtimeUtc": "2017-09-26T09:10:40Z",
    "AlertThresholdOperator": "Greater Than",
    "AlertThresholdValue": 0,
    "ResultCount": 1,
    "SearchIntervalInSeconds": 3600,
    "LinkToSearchResults": "https://workspaceID.portal.mms.microsoft.com/#Workspace/search/index?_timeInterval.intervalEnd=2017-09-26T09%3a10%3a40.0000000Z&_timeInterval.intervalDuration=3600&q=Usage",
    "Description": null,
    "Severity": "Critical"
    }



## <a name="next-steps"></a>Passaggi successivi
- Completare una procedura dettagliata per la [configurazione di un webhook](log-analytics-alerts-webhooks.md) con una regola di avviso.  
- Informazioni su come scrivere [runbook in Automazione di Azure](https://azure.microsoft.com/documentation/services/automation) per la risoluzione dei problemi identificati dagli avvisi.
