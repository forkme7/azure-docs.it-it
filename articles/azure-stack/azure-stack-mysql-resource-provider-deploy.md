---
title: Utilizzare i database MySQL come PaaS nello Stack di Azure | Documenti Microsoft
description: Informazioni su come distribuire il Provider di risorse MySQL e fornire i database MySQL come servizio nello Stack di Azure.
services: azure-stack
documentationCenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/27/2018
ms.author: mabrigg
ms.reviewer: jeffgo
ms.openlocfilehash: 66e1d5691b431be0c3d040570b13e8d16b1669ef
ms.sourcegitcommit: 20d103fb8658b29b48115782fe01f76239b240aa
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 04/03/2018
---
# <a name="use-mysql-databases-on-microsoft-azure-stack"></a>Utilizzare i database MySQL in Microsoft Azure Stack

*Si applica a: Azure Stack integrate di sistemi Azure Stack Development Kit*

È possibile distribuire un provider di risorse MySQL nello Stack di Azure. Dopo aver distribuito il provider di risorse, è possibile creare MySQL Server e database tramite i modelli di distribuzione Azure Resource Manager. È anche possibile fornire i database MySQL come servizio. 

Database MySQL, che sono comuni nei siti web, supportano molte piattaforme di sito Web. Ad esempio, dopo aver distribuito il provider di risorse, è possibile creare siti Web WordPress dalla piattaforma Web App come un componente aggiuntivo del servizio (PaaS) per lo Stack di Azure.

Per distribuire il provider di MySQL in un sistema che non dispone dell'accesso a Internet, copiare il file [mysql-connector-net-6.10.5.msi](https://dev.mysql.com/get/Download/sConnector-Net/mysql-connector-net-6.10.5.msi) in una condivisione locale. Quando richiesto, quindi specificare il nome della condivisione. È necessario installare i moduli di Azure e Azure PowerShell dello Stack.


## <a name="mysql-server-resource-provider-adapter-architecture"></a>Architettura dell'adapter provider risorse MySQL Server

Il provider di risorse è costituito da tre componenti:

- **Adattatore di provider di risorse MySQL VM**, che è una macchina virtuale di Windows che esegue il provider di servizi.

- **Il provider di risorse stesso**, che elabora le richieste di provisioning ed espone le risorse di database.

- **I server che ospitano il MySQL Server**, che forniscono capacità per i database che vengono chiamati i server di hosting.

Questa versione non crea più istanze di MySQL. Ciò significa che è necessario crearle manualmente e/o fornire l'accesso a istanze di SQL esterne. Visitare il [raccolta Guida introduttiva di Azure Stack](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/mysql-standalone-server-windows) per un modello di esempio che è possibile:
- Creare un server MySQL.
- Scaricare e distribuire un MySQL Server da Azure Marketplace.

> [!NOTE]
> I server che vengono installati nello Stack di Azure di hosting sistemi integrati devono essere creati da una sottoscrizione tenant. Non possono essere creati dalla sottoscrizione provider predefinito. Essi devono essere creati dal portale tenant o da una sessione di PowerShell con un accesso appropriato. Tutti i server di hosting sono addebitabile macchine virtuali e devono disporre delle licenze appropriate. L'amministratore del servizio può essere il proprietario della sottoscrizione tenant.

### <a name="required-privileges"></a>Privilegi necessari
L'account di sistema deve disporre dei privilegi seguenti:

1.  Database: Creare, eliminare
2.  Account di accesso: Creazione, impostare, eliminare, concedere, revocare

## <a name="deploy-the-resource-provider"></a>Distribuire il provider di risorse

1. Se non è già stato fatto, registrare il kit di sviluppo e scaricare l'immagine di Windows Server 2016 Data Center Core scaricabile tramite la gestione del Marketplace. È necessario utilizzare un'immagine di Windows Server 2016 Core. È inoltre possibile utilizzare uno script per creare un [immagine di Windows Server 2016](https://docs.microsoft.com/azure/azure-stack/azure-stack-add-default-image). (Assicurarsi di selezionare l'opzione di core.)


2. Accedere a un host che possa accedere all'endpoint con privilegi di macchina virtuale.

    - Nelle installazioni di Azure SDK, accedere all'host fisico. 
    - Nei sistemi a più nodi, l'host deve essere un sistema che possa accedere all'endpoint con privilegi.
    
    >[!NOTE]
    > Il sistema in cui è in esecuzione lo script *deve* un sistema di Windows 10 o Windows Server 2016 con la versione più recente del runtime di .NET installata. Installazione non riesce in caso contrario. L'host di Azure SDK Stack soddisfa questo criterio.
    

3. Scaricare il provider di risorse MySQL binario. Quindi eseguire il programma di autoestrazione per estrarre il contenuto in una directory temporanea.

    >[!NOTE] 
    > Il provider di risorse dispone di uno Stack di Azure corrispondente minimo di compilazione. Assicurarsi di scaricare il file binario corretto per la versione dello Stack di Azure che è in esecuzione.

    | Compilazione di Azure Stack | Programma di installazione di MySQL RP |
    | --- | --- |
    | 1802: 1.0.180302.1 | [RP MySQL versione 1.1.18.0](https://aka.ms/azurestackmysqlrp1802) |
    | 1712: 1.0.180102.3 o 1.0.180106.1 (a più nodi) | [RP MySQL versione 1.1.14.0](https://aka.ms/azurestackmysqlrp1712) |
    | 1711: 1.0.171122.1 | [RP MySQL versione 1.1.12.0](https://aka.ms/azurestackmysqlrp1711) |
    | 1710: 1.0.171028.1 | [RP MySQL versione 1.1.8.0](https://aka.ms/azurestackmysqlrp1710) |

4.  Per Azure SDK, viene creato un certificato autofirmato come parte di questo processo. Per più nodi, è necessario fornire un certificato appropriato.

    Se è necessario fornire il proprio certificato, inserire un file con estensione pfx nel **DependencyFilesLocalPath** che soddisfa i criteri seguenti:

    - Un certificato con caratteri jolly per \*.dbadapter.\< area\>.\< fqdn esterno\> o un certificato del singolo sito con un nome comune mysqladapter.dbadapter.\< area\>.\< fqdn esterno\>.

    - Questo certificato deve essere considerato attendibile. Vale a dire la catena di certificati deve esistere senza richiedere i certificati intermedi.

    - Solo un singolo file di certificato esiste nel DependencyFilesLocalPath.
    
    - Il nome del file non deve contenere spazi o caratteri speciali.


5. Aprire un **nuova** console di PowerShell con privilegi elevati (amministratore). Quindi passare alla directory in cui sono stati estratti i file. Per evitare problemi che potrebbero verificarsi dei moduli di PowerShell non corretti che sono già caricati nel sistema, utilizzare una nuova finestra.

6. [Installare Azure PowerShell versione 1.2.11](azure-stack-powershell-install.md).

7. Eseguire lo script `DeployMySqlProvider.ps1`.

Lo script esegue le operazioni:

* Scarica il file binario connettore MySQL (ciò può essere fornito non in linea).
* Carica i certificati e altri elementi di un account di archiviazione nello Stack di Azure.
* Pubblica i pacchetti di raccolta in modo che è possibile distribuire il database SQL tramite la raccolta.
* Pubblica un pacchetto di raccolta per la distribuzione di server di hosting.
* Consente di distribuire una macchina virtuale usando l'immagine di Windows Server 2016 che è stato creato nel passaggio 1. Inoltre, viene installato il provider di risorse.
* Registra un record DNS locale che esegue il mapping al provider di risorse macchina virtuale.
* Registra il provider di risorse con locale Gestione risorse di Azure (admin e tenant).


È possibile:
- Specificare i parametri obbligatori nella riga di comando.
- Eseguire senza parametri e quindi immetterli quando richiesto.

Di seguito è riportato un esempio che è possibile eseguire dal prompt di PowerShell. Assicurarsi di modificare le informazioni sull'account e password in base alle esigenze:


```
# Install the AzureRM.Bootstrapper module, set the profile, and install the AzureRM and AzureStack modules.
Install-Module -Name AzureRm.BootStrapper -Force
Use-AzureRmProfile -Profile 2017-03-09-profile
Install-Module -Name AzureStack -RequiredVersion 1.2.11 -Force

# Use the NetBIOS name for the Azure Stack domain. On the Azure Stack SDK, the default is AzureStack but could have been changed at install time.
$domain = "AzureStack"

# For integrated systems, use the IP address of one of the ERCS virtual machines
$privilegedEndpoint = "AzS-ERCS01"

# Point to the directory where the resource provider installation files were extracted.
$tempDir = 'C:\TEMP\MYSQLRP'

# The service admin account (can be Azure Active Directory or Active Directory Federation Services).
$serviceAdmin = "admin@mydomain.onmicrosoft.com"
$AdminPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force
$AdminCreds = New-Object System.Management.Automation.PSCredential ($serviceAdmin, $AdminPass)

# Set the credentials for the new resource provider VM local administrator account
$vmLocalAdminPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force
$vmLocalAdminCreds = New-Object System.Management.Automation.PSCredential ("mysqlrpadmin", $vmLocalAdminPass)

# And the cloudadmin credential required for privileged endpoint access.
$CloudAdminPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force
$CloudAdminCreds = New-Object System.Management.Automation.PSCredential ("$domain\cloudadmin", $CloudAdminPass)

# Change the following as appropriate.
$PfxPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force

# Run the installation script from the folder where you extracted the installation files.
# Find the ERCS01 IP address first, and make sure the certificate
# file is in the specified directory.
. $tempDir\DeployMySQLProvider.ps1 -AzCredential $AdminCreds `
  -VMLocalCredential $vmLocalAdminCreds `
  -CloudAdminCredential $cloudAdminCreds `
  -PrivilegedEndpoint $privilegedEndpoint `
  -DefaultSSLCertificatePassword $PfxPass `
  -DependencyFilesLocalPath $tempDir\cert `
  -AcceptLicense

 ```


### <a name="deploymysqlproviderps1-parameters"></a>Parametri DeployMySqlProvider.ps1
È possibile specificare questi parametri nella riga di comando. In caso contrario, o qualsiasi parametro di convalida non riesce, viene chiesto di fornire i parametri richiesti.

| Nome parametro | DESCRIZIONE | Commento o il valore predefinito |
| --- | --- | --- |
| **CloudAdminCredential** | Le credenziali per l'amministratore del cloud, necessaria per l'accesso all'endpoint con privilegi. | _Obbligatorio_ |
| **AzCredential** | Le credenziali per l'account amministratore del servizio Azure Stack. Utilizzare le stesse credenziali utilizzate per la distribuzione dello Stack di Azure. | _Obbligatorio_ |
| **VMLocalCredential** | Le credenziali per l'account amministratore locale del provider di risorse MySQL macchina virtuale. | _Obbligatorio_ |
| **PrivilegedEndpoint** | L'indirizzo IP o nome DNS dell'endpoint con privilegi. |  _Obbligatorio_ |
| **DependencyFilesLocalPath** | Percorso di una condivisione locale contenente [mysql-connector-net-6.10.5.msi](https://dev.mysql.com/get/Downloads/Connector-Net/mysql-connector-net-6.10.5.msi). Se si specifica uno di questi percorsi, il file del certificato deve trovarsi in questa directory. | _Parametro facoltativo_ (_obbligatorio_ a nodi multipli) |
| **DefaultSSLCertificatePassword** | La password per il certificato con estensione pfx. | _Obbligatorio_ |
| **MaxRetryCount** | Il numero di volte in cui che si desidera ripetere ogni operazione se si è verificato un errore.| 2 |
| **RetryDuration** | L'intervallo di timeout tra due tentativi, in secondi. | 120 |
| **Disinstallare** | Rimuove il provider di risorse e tutte le risorse associate (vedere le note seguenti). | No  |
| **DebugMode** | Impedisce la pulizia automatica in caso di errore. | No  |
| **AcceptLicense** | Ignora la richiesta di accettare le condizioni di licenza GPL.  (http://www.gnu.org/licenses/old-licenses/gpl-2.0.html) | |


## <a name="verify-the-deployment-by-using-the-azure-stack-portal"></a>Verificare la distribuzione tramite il portale di Azure Stack

> [!NOTE]
>  Al termine dell'esecuzione dello script di installazione, è necessario aggiornare il portale per visualizzare il pannello di amministrazione.


1. Accedere al portale di amministrazione con l'amministratore del servizio.

2. Verificare che la distribuzione ha avuto esito positivo. Passare a **gruppi di risorse**, quindi selezionare il **system.\< percorso\>.mysqladapter** gruppo di risorse. Verificare che tutte le distribuzioni di quattro ha avuto esito positivo.

      ![Verificare la distribuzione di RP di MySQL](./media/azure-stack-mysql-rp-deploy/mysqlrp-verify.png)

## <a name="provide-capacity-by-connecting-to-a-mysql-hosting-server"></a>Offrono capacità connettendosi a un server di hosting MySQL

1. Accedere al portale di Azure Stack come un amministratore del servizio.

2. Selezionare **risorse amministrative** > **i server di Hosting MySQL** > **+ Aggiungi**.

    Nel **server di Hosting MySQL** pannello, è possibile connettersi al provider di risorse di MySQL Server effettive istanze di MySQL Server che servono come back-end del provider di risorse.

    ![Server di hosting](./media/azure-stack-mysql-rp-deploy/mysql-add-hosting-server-2.png)

3. Fornire i dettagli di connessione dell'istanza di MySQL Server. Assicurarsi di fornire il nome di dominio completo (FQDN) o un indirizzo IPv4 valido e non il nome breve della macchina virtuale. Questa installazione non include un'istanza di MySQL predefinita. La dimensione che viene fornita consente il provider di risorse di gestire la capacità del database. Dovrebbe essere vicino la capacità fisica del server di database.

    > [!NOTE]
    >Se l'istanza di MySQL è possibile accedere dai tenant e amministrazione di gestione risorse di Azure, può essere inserita nel controllo del codice del provider di risorse. Istanza di MySQL *deve* essere allocata in modo esclusivo al provider di risorse.

4. Quando si aggiungono server, è necessario assegnare a uno SKU di nuovo o esistente per consentire la differenziazione delle offerte di servizio.
  Ad esempio, è possibile avere un'istanza enterprise fornisce:
    - capacità di database
    - Backup automatico
    - server ad alte prestazioni per i singoli reparti di riserva
 

Il nome SKU deve riflettere le proprietà in modo che i tenant è possono posizionare i database in modo appropriato. Tutti i server di hosting in un'unità SKU devono avere le stesse funzionalità.

![Creare uno SKU di MySQL](./media/azure-stack-mysql-rp-deploy/mysql-new-sku.png)


>[!NOTE]
> SKU può richiedere a un'ora per essere visibile nel portale. È possibile creare un database fino a quando non viene creato lo SKU.


## <a name="test-your-deployment-by-creating-your-first-mysql-database"></a>Testare la distribuzione tramite la creazione di un database MySQL


1. Accedere al portale di Azure Stack come un amministratore del servizio.

2. Selezionare **+ nuova** > **dati e archiviazione** > **Database MySQL**.

3. Fornire i dettagli del database.

    ![Creare un test di database MySQL](./media/azure-stack-mysql-rp-deploy/mysql-create-db.png)

4. Selezionare uno SKU.

    ![Selezionare un'unità SKU](./media/azure-stack-mysql-rp-deploy/mysql-select-a-sku.png)

5. Creare un'impostazione di account di accesso. È possibile riutilizzare un'impostazione di account di accesso esistente o crearne uno nuovo. Questa impostazione contiene il nome utente e la password per il database.

    ![Creare un nuovo account di database](./media/azure-stack-mysql-rp-deploy/create-new-login.png)

    La stringa di connessione include il nome del server di database reali. Copiarlo dal portale.

    ![Ottenere la stringa di connessione per il database MySQL](./media/azure-stack-mysql-rp-deploy/mysql-db-created.png)

> [!NOTE]
> La lunghezza dei nomi utente non può superare i 32 caratteri 5.7 MySQL. Nelle versioni precedenti, non può superare i 16 caratteri.


## <a name="add-capacity"></a>Aggiungere capacità

Aggiungere la capacità aggiungendo altri server di MySQL nel portale di Azure Stack. A uno SKU di nuovo o esistente, è possibile aggiungere ulteriori server. Verificare che le caratteristiche server sono uguali.


## <a name="make-mysql-databases-available-to-tenants"></a>Rendere disponibili ai tenant i database MySQL
Creare i piani e le offerte per rendere disponibili i database di MySQL per tenant. Ad esempio, aggiungere il servizio Microsoft.MySqlAdapter, aggiungere una quota e così via.

![Creare i piani e le offerte per includere i database](./media/azure-stack-mysql-rp-deploy/mysql-new-plan.png)

## <a name="update-the-administrative-password"></a>Aggiornare la password amministrativa
È possibile modificare la password può essere modificato nell'istanza del server MySQL. Selezionare **risorse amministrative** > **i server di Hosting MySQL**. Selezionare quindi il server di hosting. Nel **impostazioni** Pannello di seleziona **Password**.

![Aggiornare la password dell'amministratore](./media/azure-stack-mysql-rp-deploy/mysql-update-password.png)

## <a name="update-the-mysql-resource-provider-adapter-multi-node-only-builds-1710-and-later"></a>Aggiornare l'adapter di provider di risorse MySQL (a più nodi solo, le compilazioni 1710 e versioni successive)
Un nuovo adattatore di provider di risorse SQL potrebbe essere rilasciato quando vengono aggiornate le compilazioni di Stack di Azure. Mentre l'adapter esistente continua a funzionare, si consiglia di aggiornare appena possibile per la build più recente. 

Per l'aggiornamento del provider di risorse a cui si utilizza il *UpdateMySQLProvider.ps1* script. Il processo è simile a quello utilizzato per installare un provider di risorse, come descritto nel [distribuire il provider di risorse](#deploy-the-resource-provider) sezione di questo articolo. Lo script è incluso con il download del provider di risorse.

Il *UpdateMySQLProvider.ps1* script crea una nuova macchina virtuale con il codice di provider di risorse più recente e consente di migrare le impostazioni dalla macchina virtuale precedente per la nuova macchina virtuale. Le impostazioni di cui eseguire la migrazione includono database e le informazioni sul server di hosting e registra il DNS necessario.

Lo script richiede l'utilizzo degli stessi argomenti descritti per lo script DeployMySqlProvider.ps1. Specificare anche il certificato. 

Di seguito è riportato un esempio del *UpdateMySQLProvider.ps1* script che è possibile eseguire dal prompt di PowerShell. Assicurarsi di modificare le informazioni sull'account e password in base alle esigenze: 

> [!NOTE]
> Il processo di aggiornamento si applica solo ai sistemi integrati.

```
# Install the AzureRM.Bootstrapper module, set the profile, and install AzureRM and AzureStack modules.
Install-Module -Name AzureRm.BootStrapper -Force
Use-AzureRmProfile -Profile 2017-03-09-profile
Install-Module -Name AzureStack -RequiredVersion 1.2.11 -Force

# Use the NetBIOS name for the Azure Stack domain. On the Azure Stack SDK, the default is AzureStack but could have been changed at install time.
$domain = "AzureStack"

# For integrated systems, use the IP address of one of the ERCS virtual machines
$privilegedEndpoint = "AzS-ERCS01"

# Point to the directory where the resource provider installation files were extracted.
$tempDir = 'C:\TEMP\MYSQLRP'

# The service admin account (can be Azure Active Directory or Active Directory Federation Services).
$serviceAdmin = "admin@mydomain.onmicrosoft.com"
$AdminPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force
$AdminCreds = New-Object System.Management.Automation.PSCredential ($serviceAdmin, $AdminPass)

# Set credentials for the new resource provider VM.
$vmLocalAdminPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force
$vmLocalAdminCreds = New-Object System.Management.Automation.PSCredential ("sqlrpadmin", $vmLocalAdminPass)

# And the cloudadmin credential required for privileged endpoint access.
$CloudAdminPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force
$CloudAdminCreds = New-Object System.Management.Automation.PSCredential ("$domain\cloudadmin", $CloudAdminPass)

# Change the following as appropriate.
$PfxPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force

# Change directory to the folder where you extracted the installation files.
# Then adjust the endpoints.
. $tempDir\UpdateMySQLProvider.ps1 -AzCredential $AdminCreds `
  -VMLocalCredential $vmLocalAdminCreds `
  -CloudAdminCredential $cloudAdminCreds `
  -PrivilegedEndpoint $privilegedEndpoint `
  -DefaultSSLCertificatePassword $PfxPass `
  -DependencyFilesLocalPath $tempDir\cert `
  -AcceptLicense
 ```

### <a name="updatemysqlproviderps1-parameters"></a>Parametri UpdateMySQLProvider.ps1
È possibile specificare questi parametri nella riga di comando. In caso contrario, o qualsiasi parametro di convalida non riesce, viene chiesto di fornire i parametri richiesti.

| Nome parametro | DESCRIZIONE | Commento o il valore predefinito |
| --- | --- | --- |
| **CloudAdminCredential** | Le credenziali per l'amministratore del cloud, necessaria per l'accesso all'endpoint con privilegi. | _Obbligatorio_ |
| **AzCredential** | Le credenziali per l'account amministratore del servizio Azure Stack. Utilizzare le stesse credenziali utilizzate per la distribuzione dello Stack di Azure. | _Obbligatorio_ |
| **VMLocalCredential** |Le credenziali per l'account amministratore locale del provider di risorse SQL macchina virtuale. | _Obbligatorio_ |
| **PrivilegedEndpoint** | L'indirizzo IP o nome DNS dell'endpoint con privilegi. |  _Obbligatorio_ |
| **DependencyFilesLocalPath** | Il file pfx del certificato deve trovarsi in questa directory. | _Parametro facoltativo_ (_obbligatorio_ a nodi multipli) |
| **DefaultSSLCertificatePassword** | La password per il certificato con estensione pfx | _Obbligatorio_ |
| **MaxRetryCount** | Il numero di volte in cui che si desidera ripetere ogni operazione se si è verificato un errore.| 2 |
| **RetryDuration** | L'intervallo di timeout tra due tentativi, in secondi. | 120 |
| **Disinstallare** | Rimuovere il provider di risorse e tutte le risorse associate (vedere le note seguenti). | No  |
| **DebugMode** | Impedisce la pulizia automatica in caso di errore. | No  |
| **AcceptLicense** | Ignora la richiesta di accettare le condizioni di licenza GPL.  (http://www.gnu.org/licenses/old-licenses/gpl-2.0.html) | |


## <a name="collect-diagnostic-logs"></a>Raccogliere i log di diagnostica
Il provider di risorse MySQL è un controllo bloccato inattivo macchina virtuale. Se è necessario raccogliere i log dalla macchina virtuale, un endpoint di PowerShell Just Enough Administration (JEA) _DBAdapterDiagnostics_ viene fornito a tale scopo. Sono disponibili due comandi tramite questo endpoint:

* Get-AzsDBAdapterLog - prepara un pacchetto zip contenente i registri di diagnostica RP e lo inserisce nell'unità sessione utente. Il comando può essere chiamato senza parametri e raccoglierà le ultime quattro ore dei log.
* Remove-AzsDBAdapterLog - pulisce i pacchetti di log esistenti nel provider di risorse macchina virtuale

Un account utente denominato _dbadapterdiag_ viene creato durante la distribuzione di relying Party o aggiornamento per la connessione all'endpoint di diagnostica per l'estrazione dei registri di relying Party. La password dell'account è uguale a quella fornita per l'account amministratore locale durante l'installazione/aggiornamento.

Per utilizzare questi comandi, è necessario creare una sessione remota di PowerShell per la macchina virtuale del provider di risorse e richiamare il comando. È anche possibile specificare parametri FromDate e ToDate. Se non si specifica una o entrambe queste opzioni, FromDate sarà quattro ore prima dell'ora corrente e il ToDate sarà l'ora corrente.

Questo script di esempio viene illustrato l'utilizzo di questi comandi:

```
# Create a new diagnostics endpoint session.
$databaseRPMachineIP = '<RP VM IP>'
$diagnosticsUserName = 'dbadapterdiag'
$diagnosticsUserPassword = '<see above>'

$diagCreds = New-Object System.Management.Automation.PSCredential `
        ($diagnosticsUserName, $diagnosticsUserPassword)
$session = New-PSSession -ComputerName $databaseRPMachineIP -Credential $diagCreds `
        -ConfigurationName DBAdapterDiagnostics

# Sample captures logs from the previous one hour
$fromDate = (Get-Date).AddHours(-1)
$dateNow = Get-Date
$sb = {param($d1,$d2) Get-AzSDBAdapterLog -FromDate $d1 -ToDate $d2}
$logs = Invoke-Command -Session $session -ScriptBlock $sb -ArgumentList $fromDate,$dateNow

# Copy the logs
$sourcePath = "User:\{0}" -f $logs
$destinationPackage = Join-Path -Path (Convert-Path '.') -ChildPath $logs
Copy-Item -FromSession $session -Path $sourcePath -Destination $destinationPackage

# Cleanup logs
$cleanup = Invoke-Command -Session $session -ScriptBlock {Remove- AzsDBAdapterLog }
# Close the session
$session | Remove-PSSession
```

## <a name="maintenance-operations-integrated-systems"></a>Operazioni di manutenzione (sistemi integrati)
Il provider di risorse MySQL è un controllo bloccato inattivo macchina virtuale. Aggiornamento della protezione della macchina virtuale la risorsa del provider può essere eseguita tramite l'endpoint di PowerShell Just Enough Administration (JEA) _DBAdapterMaintenance_.

Uno script viene fornito con il pacchetto di installazione del componente per semplificare queste operazioni.


### <a name="update-the-virtual-machine-operating-system"></a>Aggiornare il sistema operativo della macchina virtuale
Esistono diversi modi per aggiornare la macchina virtuale di Windows Server:
* Installare il pacchetto di provider di risorse più recente usando un'immagine di Windows Server 2016 Core attualmente con patch
* Installa un pacchetto di Windows Update durante l'installazione o aggiornamento del componente


### <a name="update-the-virtual-machine-windows-defender-definitions"></a>Aggiornare le definizioni di Windows Defender di macchina virtuale

Seguire questi passaggi per aggiornare le definizioni di Defender:

1. Download di aggiornare le definizioni di Windows Defender [Windows Defender definizione](https://www.microsoft.com/en-us/wdsi/definitions)

    In questa pagina in "Scaricare e installare manualmente le definizioni" Scarica "Antivirus di Windows Defender per Windows 10 e Windows 8.1" il file a 64 bit. 
    
    Collegamento diretto: https://go.microsoft.com/fwlink/?LinkID=121721&arch=x64

2. Creare una sessione di PowerShell all'endpoint di manutenzione della macchina virtuale di MySQL RP adapter
3. Copiare il file di aggiornamento delle definizioni al computer di adapter DB utilizzando la sessione di manutenzione endpoint
4. Nella pagina Manutenzione PowerShell sessione richiamare il _aggiornamento DBAdapterWindowsDefenderDefinitions_ comando
5. Dopo l'installazione, è consigliabile rimuovere il file di aggiornamento tra le definizioni. Può essere rimosso della sessione di manutenzione utilizzando la _Remove-ItemOnUserDrive)_ comando.


Di seguito è riportato un esempio di script per aggiornare le definizioni di Defender (sostituire l'indirizzo o il nome della macchina virtuale con il valore effettivo):

```
# Set credentials for the diagnostic user
$diagPass = ConvertTo-SecureString "P@ssw0rd1" -AsPlainText -Force
$diagCreds = New-Object System.Management.Automation.PSCredential `
    ("dbadapterdiag", $vmLocalAdminPass)$diagCreds = Get-Credential

# Public IP Address of the DB adapter machine
$databaseRPMachine  = "XX.XX.XX.XX"
$localPathToDefenderUpdate = "C:\DefenderUpdates\mpam-fe.exe"
 
# Download Windows Defender update definitions file from https://www.microsoft.com/en-us/wdsi/definitions. 
Invoke-WebRequest -Uri https://go.microsoft.com/fwlink/?LinkID=121721&arch=x64 `
    -Outfile $localPathToDefenderUpdate 

# Create session to the maintenance endpoint
$session = New-PSSession -ComputerName $databaseRPMachine `
    -Credential $diagCreds -ConfigurationName DBAdapterMaintenance
# Copy defender update file to the db adapter machine
Copy-Item -ToSession $session -Path $localPathToDefenderUpdate `
     -Destination "User:\mpam-fe.exe"
# Install the update file
Invoke-Command -Session $session -ScriptBlock `
    {Update-AzSDBAdapterWindowsDefenderDefinitions -DefinitionsUpdatePackageFile "User:\mpam-fe.exe"}
# Cleanup the definitions package file and session
Invoke-Command -Session $session -ScriptBlock `
    {Remove-AzSItemOnUserDrive -ItemPath "User:\mpam-fe.exe"}
$session | Remove-PSSession
```


## <a name="remove-the-mysql-resource-provider-adapter"></a>Rimuovere la scheda provider di risorse MySQL

Per rimuovere il provider di risorse, è necessario innanzitutto rimuovere tutte le dipendenze.

1. Assicurarsi di disporre il pacchetto di distribuzione originale scaricati per questa versione del provider di risorse.

2. È necessario eliminare tutti i database tenant dal provider di risorse. (Eliminare i database tenant non eliminare i dati). Questa attività deve essere eseguita per i tenant stessi.

3. È necessario annullare la registrazione di tenant dallo spazio dei nomi.

4. L'amministratore deve eliminare il server di hosting dalla scheda di MySQL.

5. L'amministratore deve eliminare i piani che fanno riferimento l'Adapter di MySQL.

6. L'amministratore deve eliminare le quote che sono associate con l'Adapter di MySQL.

7. Eseguire nuovamente lo script di distribuzione con gli elementi seguenti:
    - -Disinstallare parametro
    - Gli endpoint di gestione risorse di Azure
    - The DirectoryTenantID
    - Le credenziali per l'account amministratore del servizio
