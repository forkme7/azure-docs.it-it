---
title: Domande frequenti e problemi noti di Identità del servizio gestito (MSI) per Azure Active Directory
description: Problemi noti di Identità del servizio gestito per Azure Active Directory.
services: active-directory
documentationcenter: ''
author: daveba
manager: mtillman
editor: ''
ms.assetid: 2097381a-a7ec-4e3b-b4ff-5d2fb17403b6
ms.service: active-directory
ms.devlang: ''
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 12/12/2017
ms.author: daveba
ms.openlocfilehash: a50854b2e12db9a202d769f9e5feebee8e5f9395
ms.sourcegitcommit: 1362e3d6961bdeaebed7fb342c7b0b34f6f6417a
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/18/2018
---
# <a name="faqs-and-known-issues-with-managed-service-identity-msi-for-azure-active-directory"></a>Domande frequenti e problemi noti di Identità del servizio gestito (MSI) per Azure Active Directory

[!INCLUDE[preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

## <a name="frequently-asked-questions-faqs"></a>Domande frequenti (FAQ)

### <a name="is-there-a-private-preview-program-available-for-upcoming-msi-features-and-integrations"></a>È disponibile un programma di anteprima privata per le integrazioni e le funzionalità future di Identità del servizio gestito (MSI)?

Sì. Se si vuole sottoporre la propria candidatura per la registrazione al programma di anteprima privata, [vedere la pagina di iscrizione](https://aka.ms/azuremsiprivatepreview).

### <a name="does-msi-work-with-azure-cloud-services"></a>Identità del servizio gestito funziona con Servizi cloud di Azure?

No. Non è previsto il supporto di Identità del servizio gestito in Servizi cloud di Azure.

### <a name="does-msi-work-with-the-active-directory-authentication-library-adal-or-the-microsoft-authentication-library-msal"></a>L'Identità del servizio gestito funziona con Active Directory Authentication Library (ADAL) o con Microsoft Authentication Library (MSAL)?

No, l'Identità del servizio gestito non è ancora integrata con ADAL o MSAL. Per informazioni su come acquisire il token dell'identità del servizio gestito usando l'endpoint REST dell'identità del servizio gestito, vedere [Come usare un'identità del servizio gestito di una macchina virtuale di Azure per l'acquisizione di token](how-to-use-vm-token.md).

### <a name="what-is-the-security-boundary-of-a-managed-service-identity"></a>Che cos'è il limite di sicurezza di un'identità del servizio gestito?

Il limite di sicurezza dell'identità è la risorsa a cui è collegata. Il limite di sicurezza per un file MSI di macchina virtuale, ad esempio, è la macchina virtuale. Qualsiasi codice in esecuzione su tale macchina virtuale è in grado di chiamare l'endpoint MSI e richiedere i token. L'esperienza è simile a quella con altre risorse che supportano MSI.

### <a name="should-i-use-the-msi-vm-imds-endpoint-or-the-msi-vm-extension-endpoint"></a>È necessario usare l'endpoint del Servizio metadati dell'istanza (IMDS) della macchina virtuale dell'identità del servizio gestito (MSI) o l'endpoint dell'estensione della macchina virtuale MSI?

Quando si usa MSI con le macchine virtuali, è consigliabile usare l'endpoint IMDS MSI. Il Servizio metadati dell'istanza di Azure è un endpoint REST accessibile a tutte le macchine virtuali IaaS create tramite Azure Resource Manager. Alcuni dei vantaggi dell'uso di MSI rispetto a IMDS sono i seguenti:

1. Tutti i sistemi operativi supportati IaaS di Azure possono usare MSI su IMDS. 
2. Non è più necessario installare un'estensione nella macchina virtuale per abilitare MSI. 
3. I certificati usati da MSI non sono più presenti nella macchina virtuale. 
4. L'endpoint IMDS è un indirizzo IP non instradabile noto disponibile solo dalla macchina virtuale. 

L'estensione della macchina virtuale MSI è ancora disponibile per l'uso, ma per il futuro verrà usato per impostazione predefinita l'endpoint IMDS. Il piano di deprecazione per l'estensione della macchina virtuale MSI avrà presto inizio. 

Per altre informazioni sul Servizio metadati dell'istanza di Azure, vedere la [documentazione di IMDS](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/instance-metadata-service)

### <a name="what-are-the-supported-linux-distributions"></a>Quali sono le distribuzioni di Linux supportate?

Tutte le distribuzioni Linux supportate da IaaS di Azure possono essere usate con MSI tramite l'endpoint IMDS. 

Nota: l'estensione della macchina virtuale MSI supporta solo le distribuzioni Linux seguenti:
- CoreOS Stable
- CentOS 7.1
- RedHat 7.2
- Ubuntu 15.04
- Ubuntu 16.04

Le altre distribuzioni di Linux attualmente non sono supportate e l'estensione potrebbe non riuscire nelle distribuzioni non supportate.

L'estensione funziona in CentOS 6.9. Tuttavia, a causa della mancanza del supporto di sistema nella versione 6.9, l'estensione non si riavvia automaticamente in caso di arresto anomalo o di arresto. Viene riavviata al riavvio della VM. Per riavviare manualmente l'estensione, vedere [Come si riavvia l'estensione MSI?](#how-do-you-restart-the-msi-extension)

### <a name="how-do-you-restart-the-msi-extension"></a>Come si riavvia l'estensione MSI?
In Windows e alcune versioni di Linux, se si arresta l'estensione, è possibile usare il cmdlet seguente per riavviarla manualmente:

```powershell
Set-AzureRmVMExtension -Name <extension name>  -Type <extension Type>  -Location <location> -Publisher Microsoft.ManagedIdentity -VMName <vm name> -ResourceGroupName <resource group name> -ForceRerun <Any string different from any last value used>
```

Dove: 
- Il nome e il tipo di estensione per Windows è: ManagedIdentityExtensionForWindows
- Il nome e il tipo di estensione per Linux è: ManagedIdentityExtensionForLinux

## <a name="known-issues"></a>Problemi noti

### <a name="automation-script-fails-when-attempting-schema-export-for-msi-extension"></a>Si verifica un errore di "Script di automazione" durante il tentativo di esportazione dello schema per l'estensione dell'identità del servizio gestito

Se l'identità del servizio gestito è abilitata in una macchina virtuale, viene visualizzato l'errore seguente quando si tenta di usare la funzionalità "Script di automazione" per la macchina virtuale o il relativo gruppo di risorse:

![Errore di esportazione dello script di automazione per l'identità del servizio gestito](../media/msi-known-issues/automation-script-export-error.png)

L'estensione della macchina virtuale dell'identità del servizio gestito non supporta attualmente la possibilità di esportare lo schema in un modello di gruppo di risorse. Di conseguenza, il modello generato non include i parametri di configurazione per abilitare l'identità del servizio gestito per la risorsa. È possibile aggiungere manualmente queste sezioni seguendo gli esempi in [Configurare un'Identità del servizio gestito della macchina virtuale tramite un modello](qs-configure-template-windows-vm.md).

Quando è disponibile per l'estensione della macchina virtuale dell'identità del servizio gestito, la funzionalità di esportazione dello schema sarà elencata in [Esportazione di gruppi di risorse contenenti estensioni macchina virtuale](../../virtual-machines/windows/extensions-export-templates.md#supported-virtual-machine-extensions).

### <a name="configuration-blade-does-not-appear-in-the-azure-portal"></a>Il pannello di configurazione non viene visualizzato nel portale di Azure

Se il pannello di configurazione della macchina virtuale non viene visualizzato nella macchina virtuale vuol dire che l'Identità del servizio gestito non è stata ancora abilitata nel portale della propria area geografica.  Ricontrollare in seguito.  È anche possibile abilitare l'Identità del servizio gestito per la macchina virtuale tramite [PowerShell](qs-configure-powershell-windows-vm.md) o l'[interfaccia della riga di comando di Azure](qs-configure-cli-windows-vm.md).

### <a name="cannot-assign-access-to-virtual-machines-in-the-access-control-iam-blade"></a>Impossibile assegnare l'accesso alle macchine virtuali nel pannello Controllo di accesso (IAM)

Se la **macchina virtuale** non viene visualizzata nel portale di Azure come opzione per **Assegna accesso a** in **Controllo di accesso (IAM)** > **Aggiungi autorizzazioni**, vuol dire che l'Identità del servizio gestito non è stata ancora abilitata nel portale della propria area geografica. Ricontrollare in seguito.  È comunque possibile selezionare l'Identità del servizio gestito per l'assegnazione di ruolo eseguendo una ricerca dell'entità servizio dell'Identità del servizio gestito.  Immettere il nome della macchina virtuale nel campo **Seleziona** per visualizzare l'entità servizio nei risultati della ricerca.

### <a name="vm-fails-to-start-after-being-moved-from-resource-group-or-subscription"></a>La macchina virtuale non viene avviata dopo essere stata spostata dal gruppo di risorse o dalla sottoscrizione

Se si sposta una macchina virtuale in esecuzione, questa continuerà a eseguire durante lo spostamento. Tuttavia, dopo lo spostamento, se la macchina virtuale viene arrestata e riavviata, non si avvierà. Questo problema si verifica perché la macchina virtuale non aggiorna il riferimento all'Identità del servizio gestito e continua a puntare a questa nel gruppo di risorse precedente.

**Soluzione alternativa** 
 
Attivare un aggiornamento nella macchina virtuale così da poter ottenere i valori corretti per l'Identità del servizio gestito. È possibile apportare una modifica alla proprietà della macchina virtuale per aggiornare il riferimento all'identità del servizio gestito. Ad esempio, è possibile impostare un nuovo valore di tag nella macchina virtuale con il comando seguente:

```azurecli-interactive
 az  vm update -n <VM Name> -g <Resource Group> --set tags.fixVM=1
```
 
Questo comando imposta un nuovo tag "fixVM" con valore 1 nella macchina virtuale. 
 
Impostando questa proprietà, la macchina virtuale si aggiorna inserendo l'URI della risorsa dell'Identità del servizio gestito corretto. La macchina virtuale dovrebbe a questo punto avviarsi. 
 
Dopo aver avviato la macchina virtuale, il tag può essere rimosso tramite il seguente comando:

```azurecli-interactive
az vm update -n <VM Name> -g <Resource Group> --remove tags.fixVM
```
