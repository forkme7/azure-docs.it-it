---
title: "Azure Cosmos DB: Sviluppare con l'API SQL in .NET | Microsoft Docs"
description: Informazioni su come sviluppare con l'API SQL di Azure Cosmos DB usando .NET
services: cosmos-db
documentationcenter: ''
author: rafats
manager: kfile
editor: ''
tags: ''
ms.assetid: ''
ms.service: cosmos-db
ms.devlang: dotnet
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: ''
ms.date: 05/10/2017
ms.author: rafats
ms.custom: mvc
ms.openlocfilehash: a6ed74de159593003e8a18daefce2eb9a5945481
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/16/2018
---
# <a name="azure-cosmos-db-develop-with-the-sql-api-in-net"></a>Azure Cosmos DB: Sviluppare con l'API SQL in .NET

Azure Cosmos DB è il servizio di database di Microsoft multimodello distribuito a livello globale. È possibile creare ed eseguire rapidamente query su database di documenti, coppie chiave-valore e grafi, sfruttando in ognuno dei casi i vantaggi offerti dalle funzionalità di scalabilità orizzontale e distribuzione globale alla base di Azure Cosmos DB.

Questa esercitazione illustra come creare un account Azure Cosmos DB usando il portale di Azure e come creare una raccolta e un database di documenti con una [chiave di partizione](sql-api-partition-data.md#partition-keys) usando l'[API .NET SQL](sql-api-introduction.md). Definendo una chiave di partizione quando si crea una raccolta, l'applicazione viene preparata ad una facile scalabilità in linea con la crescita dei dati.

Questa esercitazione illustra le attività seguenti usando l'[API .NET SQL](sql-api-sdk-dotnet.md):

> [!div class="checklist"]
> * Creare un account Azure Cosmos DB
> * Creare una raccolta e un database con una chiave di partizione
> * Creare documenti JSON
> * Aggiornare un documento
> * Eseguire query su raccolte partizionate
> * Eseguire stored procedure
> * Eliminare un documento
> * Eliminare un database

## <a name="prerequisites"></a>prerequisiti
Prima di iniziare, assicurarsi che siano disponibili gli elementi seguenti:

* Accesso a un account Azure Cosmos DB

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

  È anche possibile usare la propria sottoscrizione di Azure iscrivendosi per ottenere un [account Azure gratuito](https://azure.microsoft.com/free/). È quindi possibile [Creare un account Azure Cosmos DB](create-sql-api-dotnet.md#create-a-database-account).

* Se Visual Studio 2017 non è ancora installato, è possibile scaricare e usare la versione **gratuita** [Visual Studio 2017 Community Edition](https://www.visualstudio.com/downloads/). Durante l'installazione di Visual Studio abilitare **Sviluppo di Azure**.


> [!TIP]
> * Se si sceglie di usare Azure Cosmos DB Emulator, seguire la procedura illustrata in [Emulatore di Azure Cosmos DB](local-emulator.md) per configurare l'emulatore
>
>


## <a id="SetupVS"></a>Configurare la soluzione di Visual Studio
1. Aprire **Visual Studio** nel computer.
2. Scegliere **Nuovo** dal menu **File** e quindi selezionare **Progetto**.
3. Nella finestra di dialogo **Nuovo progetto** selezionare **Modelli** / **Visual C#** / **App console (.NET Framework)**, assegnare un nome al progetto e quindi fare clic su **OK**.
   ![Screenshot della finestra Nuovo progetto](./media/tutorial-develop-sql-api-dotnet/nosql-tutorial-new-project-2.png)

4. In **Esplora soluzioni** fare clic con il pulsante destro del mouse sulla nuova applicazione console, disponibile nella soluzione di Visual Studio, quindi scegliere **Gestisci pacchetti NuGet**.

    ![Screenshot del menu selezionato con il pulsante destro del mouse per il progetto](./media/tutorial-develop-sql-api-dotnet/nosql-tutorial-manage-nuget-pacakges.png)
5. Nella scheda **NuGet** fare clic su **Sfoglia** e digitare **documentdb** nella casella di ricerca.
<!---stopped here--->
6. Nei risultati trovare **Microsoft.Azure.DocumentDB** e fare clic su **Installa**.
   L'ID pacchetto per la libreria client di Azure Cosmos DB è [Microsoft.Azure.DocumentDB](https://www.nuget.org/packages/Microsoft.Azure.DocumentDB).
   ![Screenshot del menu di NuGet per l'individuazione dell'SDK client di Cosmos DB](./media/tutorial-develop-sql-api-dotnet/nosql-tutorial-manage-nuget-pacakges-2.png)

    Se viene visualizzato un messaggio sulla verifica delle modifiche alla soluzione, fare clic su **OK**. Se viene visualizzato un messaggio sull'accettazione della licenza, fare clic su **Accetto**.

## <a id="Connect"></a>Aggiungere riferimenti al progetto
I passaggi rimanenti di questa esercitazione specificano i frammenti di codice dell'API SQL necessari per creare e aggiornare le risorse di Azure Cosmos DB nel progetto.

Aggiungere questi riferimenti all'applicazione.
<!---These aren't added by default when you install the pkg?--->

```csharp
using System.Net;
using Microsoft.Azure.Documents;
using Microsoft.Azure.Documents.Client;
using Newtonsoft.Json;
```

## <a id="add-references"></a>Connessione dell'app

Aggiungere quindi queste due costanti e la variabile *client* nell'applicazione.

```csharp
private const string EndpointUrl = "<your endpoint URL>";
private const string PrimaryKey = "<your primary key>";
private DocumentClient client;
```

Tornare al [portale di Azure](https://portal.azure.com) per recuperare la chiave primaria e l'URL dell'endpoint. L'URL e la chiave primaria dell'endpoint sono necessari all'applicazione per conoscere la destinazione della connessione e ad Azure Cosmos DB per considerare attendibile la connessione dell'applicazione.

Nel portale di Azure passare all'account Azure Cosmos DB. Nel menu a sinistra selezionare **Chiavi** e quindi selezionare **Chiavi di lettura/scrittura**.

Copiare l'URI dal portale e incollarlo su `<your endpoint URL>` nel file program.cs. Copiare quindi la CHIAVE PRIMARIA dal portale e incollarla su `<your primary key>`. Assicurarsi di rimuovere `<` e `>` dai valori.

![Screenshot del portale di Azure usato nell'esercitazione su NoSQL per creare un'applicazione console C#. Mostra un account Azure Cosmos DB, il pulsante CHIAVI evidenziato nella sezione dell'account Azure Cosmos DB e i valori URI e CHIAVE PRIMARIA evidenziati nella sezione Chiavi](./media/tutorial-develop-sql-api-dotnet/nosql-tutorial-keys.png)

## <a id="instantiate"></a>Creare un'istanza di DocumentClient

Creare ora una nuova istanza di **DocumentClient**.

```csharp
DocumentClient client = new DocumentClient(new Uri(EndpointUrl), PrimaryKey);
```

## <a id="create-database"></a>Creare un database

Creare quindi un [database](sql-api-resources.md#databases) di Azure Cosmos DB usando il metodo [CreateDatabaseAsync](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.createdatabaseasync.aspx) o [CreateDatabaseIfNotExistsAsync](https://msdn.microsoft.com/library/microsoft.azure.documents.client.documentclient.createdatabaseifnotexistsasync.aspx) della classe **DocumentClient** da [SQL .NET SDK](sql-api-sdk-dotnet.md). Un database è il contenitore logico per l'archiviazione di documenti JSON partizionato nelle raccolte.

```csharp
await client.CreateDatabaseAsync(new Database { Id = "db" });
```
## <a name="decide-on-a-partition-key"></a>Scegliere una chiave di partizione

Le raccolte sono contenitori per l'archiviazione di documenti. Le raccolte sono risorse logiche e possono [comprendere una o più partizioni fisiche](partition-data.md). Una [chiave di partizione](sql-api-partition-data.md) è una proprietà (o percorso) all'interno dei documenti che possono essere usati per distribuire i dati tra i server o le partizioni. Tutti i documenti con la stessa chiave di partizione vengono archiviati nella stessa partizione.

L'individuazione di una chiave di partizione è una decisione importante da prendere prima di creare una raccolta. Le chiavi di partizione sono una proprietà o percorso all'interno di documenti che possono essere usate da Azure Cosmos DB per distribuire i dati tra più server o più partizioni. Cosmos DB eseguirà l'hashing del valore della chiave di partizione e userà il risultato con hash per determinare la partizione in cui archiviare il documento. Tutti i documenti con la stessa chiave di partizione vengono archiviati nella stessa partizione e le chiavi di partizione non possono essere modificate dopo la creazione di una raccolta.

In questa esercitazione è necessario impostare la chiave di partizione su `/deviceId` in modo che tutti i dati per un singolo dispositivo vengano archiviati in una singola partizione. È possibile scegliere una chiave di partizione contenente un numero elevato di valori, ognuno dei quali viene usato approssimativamente alla stessa frequenza per garantire che Cosmos DB possa bilanciare il carico in linea con la crescita dei dati e ottenere la velocità effettiva completa della raccolta.

Per altre informazioni sul partizionamento, vedere [Come eseguire il partizionamento e il ridimensionamento in Azure Cosmos DB](partition-data.md).

## <a id="CreateColl"></a>Creare una raccolta

Con la chiave di partizione, `/deviceId`, è possibile creare una [raccolta](sql-api-resources.md#collections) usando il metodo [CreateDocumentCollectionAsync](https://msdn.microsoft.com/library/azure/microsoft.azure.documents.client.documentclient.createdocumentcollectionasync.aspx) o il metodo [CreateDocumentCollectionIfNotExistsAsync](https://msdn.microsoft.com/library/microsoft.azure.documents.client.documentclient.createdocumentcollectionifnotexistsasync.aspx) della classe **DocumentClient**. Una raccolta è un contenitore di documenti JSON e di eventuale logica dell'applicazione JavaScript associata.

> [!WARNING]
> La creazione di una raccolta ha implicazioni a livello di prezzi, poiché si sta riservando velocità effettiva per l'applicazione per comunicare con Azure Cosmos DB. Per altre informazioni, visitare la [pagina dei prezzi](https://azure.microsoft.com/pricing/details/cosmos-db/).
>
>

```csharp
// Collection for device telemetry. Here the JSON property deviceId is used  
// as the partition key to spread across partitions. Configured for 2500 RU/s  
// throughput and an indexing policy that supports sorting against any  
// number or string property. .
DocumentCollection myCollection = new DocumentCollection();
myCollection.Id = "coll";
myCollection.PartitionKey.Paths.Add("/deviceId");

await client.CreateDocumentCollectionAsync(
    UriFactory.CreateDatabaseUri("db"),
    myCollection,
    new RequestOptions { OfferThroughput = 2500 });
```

Questo metodo effettua una chiamata API REST ad Azure Cosmos DB e il servizio esegue il provisioning di una serie di partizioni in base alla velocità effettiva richiesta. È possibile modificare la velocità effettiva di una raccolta se le esigenze in termini di prestazioni cambiano usando l'SDK o il [portale di Azure](set-throughput.md).

## <a id="CreateDoc"></a>Creare documenti JSON
Verranno ora inseriti alcuni documenti JSON in Azure Cosmos DB. È possibile creare un [documento](sql-api-resources.md#documents) usando il metodo [CreateDocumentAsync](https://msdn.microsoft.com/library/microsoft.azure.documents.client.documentclient.createdocumentasync.aspx) della classe **DocumentClient**. I documenti sono contenuto JSON definito dall'utente (arbitrario). Questa classe di esempio contiene una lettura di dispositivo e una chiamata a CreateDocumentAsync per inserire una nuova lettura di dispositivo in una raccolta.

```csharp
public class DeviceReading
{
    [JsonProperty("id")]
    public string Id;

    [JsonProperty("deviceId")]
    public string DeviceId;

    [JsonConverter(typeof(IsoDateTimeConverter))]
    [JsonProperty("readingTime")]
    public DateTime ReadingTime;

    [JsonProperty("metricType")]
    public string MetricType;

    [JsonProperty("unit")]
    public string Unit;

    [JsonProperty("metricValue")]
    public double MetricValue;
  }

// Create a document. Here the partition key is extracted
// as "XMS-0001" based on the collection definition
await client.CreateDocumentAsync(
    UriFactory.CreateDocumentCollectionUri("db", "coll"),
    new DeviceReading
    {
        Id = "XMS-001-FE24C",
        DeviceId = "XMS-0001",
        MetricType = "Temperature",
        MetricValue = 105.00,
        Unit = "Fahrenheit",
        ReadingTime = DateTime.UtcNow
    });
```
## <a name="read-data"></a>Leggere i dati

Leggere il documento in base alla chiave di partizione e l'ID usando il metodo ReadDocumentAsync. Si noti che le letture includono un valore PartitionKey, corrispondente all'intestazione di richiesta `x-ms-documentdb-partitionkey` nell'API REST.

```csharp
// Read document. Needs the partition key and the Id to be specified
Document result = await client.ReadDocumentAsync(
  UriFactory.CreateDocumentUri("db", "coll", "XMS-001-FE24C"),
  new RequestOptions { PartitionKey = new PartitionKey("XMS-0001") });

DeviceReading reading = (DeviceReading)(dynamic)result;
```

## <a name="update-data"></a>Aggiornare i dati

Ora si procederà all'aggiornamento di alcuni dati usando il metodo ReplaceDocumentAsync.

```csharp
// Update the document. Partition key is not required, again extracted from the document
reading.MetricValue = 104;
reading.ReadingTime = DateTime.UtcNow;

await client.ReplaceDocumentAsync(
  UriFactory.CreateDocumentUri("db", "coll", "XMS-001-FE24C"),
  reading);
```

## <a name="delete-data"></a>Eliminare i dati

Eliminare quindi un documento in base a chiave di partizione e ID usando il metodo DeleteDocumentAsync.

```csharp
// Delete a document. The partition key is required.
await client.DeleteDocumentAsync(
  UriFactory.CreateDocumentUri("db", "coll", "XMS-001-FE24C"),
  new RequestOptions { PartitionKey = new PartitionKey("XMS-0001") });
```
## <a name="query-partitioned-collections"></a>Eseguire query su raccolte partizionate

Quando viene eseguita una query sui dati delle raccolte partizionate, Azure Cosmos DB instrada automaticamente la query alle partizioni corrispondenti ai valori della chiave di partizione specificati nel filtro (se presenti). Ad esempio, questa query viene instradata solo alla partizione contenente la chiave di partizione "XMS-0001".

```csharp
// Query using partition key
IQueryable<DeviceReading> query = client.CreateDocumentQuery<DeviceReading>(
    UriFactory.CreateDocumentCollectionUri("db", "coll"))
    .Where(m => m.MetricType == "Temperature" && m.DeviceId == "XMS-0001");
```

La query seguente non dispone di un filtro per la chiave di partizione (DeviceId) e viene effettuato il fan-out a tutte le partizioni in cui viene eseguita a fronte dell'indice della partizione. Si noti che è necessario specificare EnableCrossPartitionQuery (`x-ms-documentdb-query-enablecrosspartition` nell'API REST) affinché l'SDK esegua una query tra le partizioni.

```csharp
// Query across partition keys
IQueryable<DeviceReading> crossPartitionQuery = client.CreateDocumentQuery<DeviceReading>(
    UriFactory.CreateDocumentCollectionUri("db", "coll"),
    new FeedOptions { EnableCrossPartitionQuery = true })
    .Where(m => m.MetricType == "Temperature" && m.MetricValue > 100);
```

## <a name="parallel-query-execution"></a>Esecuzione di query in parallelo
Gli SDK SQL 1.9.0 e versioni successive di Azure Cosmos DB supportano le opzioni di esecuzione di query in parallelo, che consentono di eseguire query a bassa latenza sulle raccolte partizionate, anche quando è necessario toccare un numero elevato di partizioni. Ad esempio, la query seguente è configurata in modo da essere eseguita in parallelo tra le partizioni.

```csharp
// Cross-partition Order By queries
IQueryable<DeviceReading> crossPartitionQuery = client.CreateDocumentQuery<DeviceReading>(
    UriFactory.CreateDocumentCollectionUri("db", "coll"),
    new FeedOptions { EnableCrossPartitionQuery = true, MaxDegreeOfParallelism = 10, MaxBufferedItemCount = 100})
    .Where(m => m.MetricType == "Temperature" && m.MetricValue > 100)
    .OrderBy(m => m.MetricValue);
```

È possibile gestire l'esecuzione di query in parallelo, ottimizzando i parametri seguenti:

* Impostando `MaxDegreeOfParallelism`è possibile controllare il grado di parallelismo, ovvero il numero massimo di connessioni di rete simultanee alle partizioni della raccolta. Se si imposta questo parametro su -1, il grado di parallelismo viene gestito dall'SDK. Se `MaxDegreeOfParallelism` non è specificato o è impostato su 0, ovvero il valore predefinito, esisterà una sola connessione di rete per le partizioni della raccolta.
* Impostando `MaxBufferedItemCount` è possibile raggiungere un compromesso tra latenza della query e uso della memoria dal lato client. Se si omette questo parametro o lo si imposta su -1, il numero di elementi memorizzati nel buffer durante l'esecuzione di query in parallelo viene gestito dall'SDK.

Considerato lo stato della raccolta, una query in parallelo restituirà i risultati nello stesso ordine dell'esecuzione seriale. Quando si esegue una query tra partizioni che include l'ordinamento (ORDER BY e/o TOP), l'SDK di SQL esegue la query in parallelo tra le partizioni e unisce i risultati ordinati parzialmente sul lato client per produrre risultati ordinati a livello globale.

## <a name="execute-stored-procedures"></a>Eseguire stored procedure
Infine è possibile eseguire transazioni atomiche rispetto a documenti con lo stesso ID dispositivo, ad esempio se si gestiscono aggregazioni o lo stato più recente di un dispositivo in un unico documento aggiungendo il codice seguente al progetto.

```csharp
await client.ExecuteStoredProcedureAsync<DeviceReading>(
    UriFactory.CreateStoredProcedureUri("db", "coll", "SetLatestStateAcrossReadings"),
    new RequestOptions { PartitionKey = new PartitionKey("XMS-001") },
    "XMS-001-FE24C");
```

L'attività è terminata. Questi sono i componenti principali di un'applicazione Azure Cosmos DB che usa una chiave di partizione per scalare in modo efficiente la distribuzione dei dati tra partizioni.  

## <a name="clean-up-resources"></a>Pulire le risorse

Se non si intende continuare a usare l'app, eliminare tutte le risorse create tramite l'esercitazione nel portale di Azure eseguendo questi passaggi:

1. Scegliere **Gruppi di risorse** dal menu a sinistra del portale di Azure e quindi fare clic sul nome univoco della risorsa creata.
2. Nella pagina del gruppo di risorse fare clic su **Elimina**, digitare il nome della risorsa da eliminare nella casella di testo e quindi fare clic su **Elimina**.

## <a name="next-steps"></a>Passaggi successivi

In questa esercitazione sono state eseguite le operazioni seguenti:

> [!div class="checklist"]
> * Creazione di un account Azure Cosmos DB
> * Creazione di una raccolta e un database con una chiave di partizione
> * Creazione di documenti JSON
> * Aggiornamento di un documento
> * Esecuzione di query su raccolte partizionate
> * Esecuzione di una stored procedure
> * Eliminazione di un documento
> * Eliminazione di un database

È ora possibile passare all'esercitazione successiva e importare i dati aggiuntivi nell'account Cosmos DB.

> [!div class="nextstepaction"]
> [Importare dati in Azure Cosmos DB](import-data.md)
