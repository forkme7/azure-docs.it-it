---
title: Trigger timer per Funzioni di Azure
description: Informazioni su come usare trigger timer in Funzioni di Azure.
services: functions
documentationcenter: na
author: tdykstra
manager: cfowler
editor: ''
tags: ''
keywords: Funzioni di Azure, Funzioni, elaborazione eventi, calcolo dinamico, architettura senza server
ms.assetid: d2f013d1-f458-42ae-baf8-1810138118ac
ms.service: functions
ms.devlang: multiple
ms.topic: reference
ms.tgt_pltfrm: multiple
ms.workload: na
ms.date: 02/27/2017
ms.author: tdykstra
ms.custom: ''
ms.openlocfilehash: 2bc2559dc1cf737e018895ffae61d0da0e56fc85
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/05/2018
---
# <a name="timer-trigger-for-azure-functions"></a>Trigger timer per Funzioni di Azure 

Questo articolo illustra come usare trigger timer in Funzioni di Azure. Un trigger timer consente di eseguire una funzione in base a una pianificazione. 

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

## <a name="packages"></a>Pacchetti

Il trigger timer è disponibile nel pacchetto NuGet [Microsoft.Azure.WebJobs.Extensions](http://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions). Il codice sorgente del pacchetto si trova nel repository GitHub [azure-webjobs-sdk-extensions](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/).

[!INCLUDE [functions-package-auto](../../includes/functions-package-auto.md)]

## <a name="example"></a>Esempio

Vedere l'esempio specifico per ciascun linguaggio:

* [C#](#trigger---c-example)
* [Script C# (file con estensione csx)](#trigger---c-script-example)
* [F#](#trigger---f-example)
* [JavaScript](#trigger---javascript-example)

### <a name="c-example"></a>Esempio in C#

L'esempio seguente mostra una [funzione C#](functions-dotnet-class-library.md) che viene eseguita ogni cinque minuti:

```cs
[FunctionName("TimerTriggerCSharp")]
public static void Run([TimerTrigger("0 */5 * * * *")]TimerInfo myTimer, TraceWriter log)
{
    if(myTimer.IsPastDue)
    {
        log.Info("Timer is running late!");
    }
    log.Info($"C# Timer trigger function executed at: {DateTime.Now}");
}
```

### <a name="c-script-example"></a>Esempio di script C#

L'esempio seguente mostra un'associazione di trigger timer in un file *function.json* e una [funzione script C#](functions-reference-csharp.md) che usa l'associazione. La funzione scrive un log che indica se la chiamata di funzione è dovuta un'occorrenza di pianificazione mancante.

Ecco i dati di associazione nel file *function.json*:

```json
{
    "schedule": "0 */5 * * * *",
    "name": "myTimer",
    "type": "timerTrigger",
    "direction": "in"
}
```

Ecco il codice script C#:

```csharp
public static void Run(TimerInfo myTimer, TraceWriter log)
{
    if(myTimer.IsPastDue)
    {
        log.Info("Timer is running late!");
    }
    log.Info($"C# Timer trigger function executed at: {DateTime.Now}" );  
}
```

### <a name="f-example"></a>Esempio F#

L'esempio seguente mostra un'associazione di trigger timer in un file *function.json* e una [funzione script F#](functions-reference-fsharp.md) che usa l'associazione. La funzione scrive un log che indica se la chiamata di funzione è dovuta un'occorrenza di pianificazione mancante.

Ecco i dati di associazione nel file *function.json*:

```json
{
    "schedule": "0 */5 * * * *",
    "name": "myTimer",
    "type": "timerTrigger",
    "direction": "in"
}
```

Ecco il codice script F#:

```fsharp
let Run(myTimer: TimerInfo, log: TraceWriter ) =
    if (myTimer.IsPastDue) then
        log.Info("F# function is running late.")
    let now = DateTime.Now.ToLongTimeString()
    log.Info(sprintf "F# function executed at %s!" now)
```

### <a name="javascript-example"></a>Esempio JavaScript

L'esempio seguente mostra un'associazione di trigger timer in un file *function.json* e una [funzione JavaScript](functions-reference-node.md) che usa l'associazione. La funzione scrive un log che indica se la chiamata di funzione è dovuta un'occorrenza di pianificazione mancante.

Ecco i dati di associazione nel file *function.json*:

```json
{
    "schedule": "0 */5 * * * *",
    "name": "myTimer",
    "type": "timerTrigger",
    "direction": "in"
}
```

Ecco il codice script JavaScript:

```JavaScript
module.exports = function (context, myTimer) {
    var timeStamp = new Date().toISOString();

    if(myTimer.isPastDue)
    {
        context.log('Node.js is running late!');
    }
    context.log('Node.js timer trigger function ran!', timeStamp);   

    context.done();
};
```

## <a name="attributes"></a>Attributi

Nelle [librerie di classi C#](functions-dotnet-class-library.md) usare l'attributo [TimerTriggerAttribute](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/TimerTriggerAttribute.cs).

Il costruttore dell'attributo accetta un'espressione CRON o un valore `TimeSpan`. È possibile usare `TimeSpan` solo se l'app per le funzioni viene eseguita in un piano di servizio app. L'esempio seguente mostra un'espressione CRON:

```csharp
[FunctionName("TimerTriggerCSharp")]
public static void Run([TimerTrigger("0 */5 * * * *")]TimerInfo myTimer, TraceWriter log)
{
    if (myTimer.IsPastDue)
    {
        log.Info("Timer is running late!");
    }
    log.Info($"C# Timer trigger function executed at: {DateTime.Now}");
}
 ```

## <a name="configuration"></a>Configurazione

Nella tabella seguente sono illustrate le proprietà di configurazione dell'associazione impostate nel file *function.json* e nell'attributo `TimerTrigger`.

|Proprietà di function.json | Proprietà dell'attributo |DESCRIZIONE|
|---------|---------|----------------------|
|**type** | n/d | Il valore deve essere impostato su "timerTrigger". Questa proprietà viene impostata automaticamente quando si crea il trigger nel portale di Azure.|
|**direction** | n/d | Il valore deve essere impostato su "in". Questa proprietà viene impostata automaticamente quando si crea il trigger nel portale di Azure. |
|**name** | n/d | Nome della variabile che rappresenta l'oggetto timer nel codice della funzione. | 
|**schedule**|**ScheduleExpression**|[Espressione CRON](#cron-expressions) o valore [TimeSpan](#timespan). `TimeSpan` può essere usato solo per un'app per le funzioni in esecuzione in un piano di servizio app. È possibile inserire l'espressione schedule in un'impostazione dell'app e definire per questa proprietà il nome dell'impostazione dell'app racchiuso tra simboli **%**, come in questo esempio: "%ImpostazioneAppSchedule%". |
|**runOnStartup**|**RunOnStartup**|Se `true`, la funzione viene richiamata all'avvio del runtime. Ad esempio, il runtime viene avviato quando l'app per le funzioni si riattiva dopo un periodo di inattività, quando l'app per le funzioni viene riavviata a causa di modifiche alla funzione e quando l'app per le funzioni viene scalata orizzontalmente. Di conseguenza, **runOnStartup** deve essere impostato raramente, o addirittura mai, su `true`, in quando causa l'esecuzione del codice in momenti estremamente imprevedibili.|
|**useMonitor**|**UseMonitor**|Impostata su `true` o `false` per indicare se monitorare la pianificazione. Il monitoraggio della pianificazione rende persistenti le occorrenze della pianificazione per garantire che la pianificazione venga gestita correttamente anche quando le istanze dell'app per le funzioni vengono riavviate. Se la proprietà non è impostata in modo esplicito, il valore predefinito è `true` per le pianificazioni che hanno un intervallo di ricorrenza maggiore di 1 minuto. Per le pianificazioni attivate più di una volta al minuto, il valore predefinito è `false`.

[!INCLUDE [app settings to local.settings.json](../../includes/functions-app-settings-local.md)]

## <a name="usage"></a>Uso

Quando viene richiamata una funzione di trigger timer, l'[oggetto timer](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/WebJobs.Extensions/Extensions/Timers/TimerInfo.cs) viene passato alla funzione. Il codice JSON seguente è una rappresentazione di esempio dell'oggetto timer. 

```json
{
    "Schedule":{
    },
    "ScheduleStatus": {
        "Last":"2016-10-04T10:15:00+00:00",
        "LastUpdated":"2016-10-04T10:16:00+00:00",
        "Next":"2016-10-04T10:20:00+00:00"
    },
    "IsPastDue":false
}
```

La proprietà `IsPastDue` è `true` quando la chiamata della funzione corrente avviene successivamente al momento pianificato. Ad esempio, un riavvio dell'app per le funzioni può causare la mancata riuscita di una chiamata.

## <a name="cron-expressions"></a>Espressioni CRON 

Un'espressione CRON per il trigger timer di Funzioni di Azure include sei campi: 

`{second} {minute} {hour} {day} {month} {day-of-week}`

Ogni campo può avere uno dei tipi di valori seguenti:

|Tipo  |Esempio  |Quando viene attivato  |
|---------|---------|---------|
|Valore specifico |<nobr>"0 5 * * * *"</nobr>|Alle hh.05.00, dove hh corrisponde a ogni ora (una volta all'ora)|
|Tutti i valori (`*`)|<nobr>"0 * 5 * * *"</nobr>|Alle 5.mm.00 ogni giorno, dove mm è ogni minuto dell'ora (60 volte al giorno)|
|Intervallo (operatore `-`)|<nobr>"5-7 * * * * *"</nobr>|Alle hh.mm.05, hh.mm.06 e hh.mm.07, dove hh.mm è ogni minuto di ogni ora (3 volte al minuto)|  
|Set di valori (operatore `,`)|<nobr>"5,8,10 * * * * *"</nobr>|Alle hh.mm.05, hh.mm.08 e hh.mm.10, dove hh.mm è ogni minuto di ogni ora (3 volte al minuto)|
|Valore di intervallo (operatore `/`)|<nobr>"0 */5 * * * *"</nobr>|Alle hh.05.00, hh.10.00, hh.15.00 e così via fino alle hh.55.00, dove hh è ogni ora (12 volte all'ora)|

### <a name="cron-examples"></a>Esempi CRON

Ecco alcuni esempi di espressioni CRON che è possibile usare per il trigger timer in Funzioni di Azure.

|Esempio|Quando viene attivato  |
|---------|---------|
|"0 */5 * * * *"|Una volta ogni cinque minuti|
|"0 0 * * * *"|Una volta all'inizio di ogni ora|
|"0 0 */2 * * *"|Una volta ogni due ore|
|"0 0 9-17 * * *"|Una volta ogni ora dalle 9 alle 17|
|"0 30 9 * * *"|Alle 9.30 di ogni giorno|
|"0 30 9 * * 1-5"|Alle 9.30 di ogni giorno feriale|

>[!NOTE]   
>Alcuni esempi di espressioni CRON sono disponibili online, ma molti omettono il campo `{second}`. Se si copia una di queste espressioni, aggiungere il campo `{second}` mancante. In genere è preferibile impostare uno zero in questo campo, anziché un asterisco.

### <a name="cron-time-zones"></a>Fusi orari CRON

I numeri in un'espressione CRON fanno riferimento a una data e a un'ora e non a un intervallo di tempo. Ad esempio, il valore 5 nel campo `hour` fa riferimento alle 5.00 e non a ogni 5 ore.

Il fuso orario predefinito usato con le espressioni CRON è Coordinated Universal Time (UTC). Per fare in modo che l'espressione CRON sia basata su un altro fuso orario, creare un'impostazione per l'app per le funzioni denominata `WEBSITE_TIME_ZONE`. Impostare il valore sul nome del fuso orario prescelto come illustrato nell'[indice dei fusi orari di Microsoft](https://technet.microsoft.com/library/cc749073). 

Ad esempio, *Ora solare fuso orientale* (EST) è UTC-05:00. Per attivare il timer trigger ogni giorno alle 10:00 EST, usare la seguente espressione CRON che rappresenta il fuso orario UTC:

```json
"schedule": "0 0 15 * * *",
``` 

In alternativa, è possibile creare un'impostazione per l'app per le funzioni denominata `WEBSITE_TIME_ZONE` e impostare il valore su **Ora solare fuso orientale**.  Usare quindi l'espressione CRON seguente: 

```json
"schedule": "0 0 10 * * *",
``` 

## <a name="timespan"></a>Intervallo di tempo

 `TimeSpan` può essere usato solo per un'app per le funzioni in esecuzione in un piano di servizio app.

Diversamente da un'espressione CRON, un valore `TimeSpan` specifica l'intervallo di tempo tra ogni chiamata di funzione. Quando una funzione viene completata dopo essere stata eseguita più a lungo dell'intervallo specificato, il timer richiama immediatamente di nuovo la funzione.

Espresso come stringa, il formato di `TimeSpan` è `hh:mm:ss`, dove `hh` è minore di 24. Quando le prime due cifre sono un numero uguale o maggiore di 24, il formato è `dd:hh:mm`. Di seguito sono riportati alcuni esempi:

|Esempio |Quando viene attivato  |
|---------|---------|
|"01:00:00" | Ogni ora        |
|"00:01:00"|Ogni minuto         |
|"24:00:00" | Ogni 24 giorni        |

## <a name="scale-out"></a>Scalabilità orizzontale

Se un'app per le funzioni viene scalata orizzontalmente a più istanze, viene eseguita un'unica istanza di una funzione attivata dal timer in tutte le istanze.

## <a name="function-apps-sharing-storage"></a>App per le funzioni che condividono un account di archiviazione

Se si condivide un account di archiviazione tra più app per le funzioni, assicurarsi che ogni app abbia un valore diverso per `id` in *host.json*. È possibile omettere la proprietà `id` o impostare manualmente la proprietà `id` di ogni app per le funzioni su un valore diverso. Il trigger timer usa un blocco dell'archiviazione per garantire che vi sia una sola istanza del timer quando un'app per le funzioni viene scalata orizzontalmente a più istanze. Se due app per le funzioni condividono la stessa proprietà `id` e ognuna usa un trigger timer, verrà eseguito solo un timer.

## <a name="retry-behavior"></a>Comportamento in caso di nuovo tentativo

Diversamente dal trigger di coda, il trigger timer non viene ripetuto se una funzione non riesce. Quando una funzione non riesce, viene chiamata di nuovo fino alla volta successiva nella pianificazione.

## <a name="next-steps"></a>Passaggi successivi

> [!div class="nextstepaction"]
> [Passare a una guida introduttiva che usa un trigger timer](functions-create-scheduled-function.md)

> [!div class="nextstepaction"]
> [Altre informazioni su trigger e associazioni di Funzioni di Azure](functions-triggers-bindings.md)
