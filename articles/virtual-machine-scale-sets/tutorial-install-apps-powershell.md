---
title: Esercitazione - Installare applicazioni in un set di scalabilità con Azure PowerShell | Microsoft Docs
description: Informazioni su come usare Azure PowerShell per installare applicazioni nei set di scalabilità di macchine virtuali con l'estensione Script personalizzato
services: virtual-machine-scale-sets
documentationcenter: ''
author: iainfoulds
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 03/27/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: ab2cc7a0fa86becc00044cd24151822138e23a67
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/19/2018
---
# <a name="tutorial-install-applications-in-virtual-machine-scale-sets-with-azure-powershell"></a>Esercitazione: Installare applicazioni in set di scalabilità di macchine virtuali con Azure PowerShell
Per eseguire applicazioni nelle istanze di macchine virtuali (VM) in un set di scalabilità, è necessario prima installare i componenti dell'applicazione e i file necessari. In un'esercitazione precedente si è appreso come usare un'immagine di macchina virtuale personalizzata per distribuire le istanze di macchina virtuale. Questa immagine personalizzata includeva installazioni e configurazioni manuali di applicazioni. È anche possibile automatizzare l'installazione delle applicazioni in un set di scalabilità dopo la distribuzione di ogni istanza di macchina virtuale oppure aggiornare un'applicazione che è già in esecuzione in un set di scalabilità. In questa esercitazione si apprenderà come:

> [!div class="checklist"]
> * Installare automaticamente le applicazioni nel set di scalabilità
> * Usare l'estensione Script personalizzato di Azure
> * Aggiornare un'applicazione in esecuzione in un set di scalabilità

Se non si ha una sottoscrizione di Azure, creare un [account gratuito](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) prima di iniziare.

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

Se si sceglie di installare e usare PowerShell in locale, per questa esercitazione è necessario il modulo Azure PowerShell versione 5.6.0 o successiva. Eseguire `Get-Module -ListAvailable AzureRM` per trovare la versione. Se è necessario eseguire l'aggiornamento, vedere [Installare e configurare Azure PowerShell](/powershell/azure/install-azurerm-ps). Se si esegue PowerShell in locale, è anche necessario eseguire `Connect-AzureRmAccount` per creare una connessione con Azure. 


## <a name="what-is-the-azure-custom-script-extension"></a>Informazioni sull'estensione Script personalizzato di Azure
L'estensione script personalizzata scarica ed esegue gli script sulle macchine virtuali di Azure. Questa estensione è utile per la configurazione post-distribuzione, l'installazione di software o qualsiasi altra attività di configurazione o gestione. Gli script possono essere scaricati dall'archiviazione di Azure o da GitHub oppure possono essere forniti al portale di Azure durante il runtime dell'estensione.

L'estensione Script personalizzato si integra nei modelli di Azure Resource Manager e può anche essere usata con l'interfaccia della riga di comando di Azure 2.0, Azure PowerShell, il portale di Azure o l'API REST. Per altre informazioni, vedere [Panoramica dell'estensione script personalizzata](../virtual-machines/windows/extensions-customscript.md).

Per vedere l'estensione Script personalizzato in azione, creare un set di scalabilità che installa il server Web IIS e restituisce il nome host dell'istanza di macchina virtuale del set di scalabilità. La definizione dell'estensione Script personalizzato scarica uno script di esempio da GitHub, installa i pacchetti richiesti, quindi scrive il nome host dell'istanza di macchina virtuale in una pagina HTML di base.


## <a name="create-a-scale-set"></a>Creare un set di scalabilità
Impostare prima di tutto nome utente e password dell'amministratore delle istanze di macchine virtuali con il comando [Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential):

```azurepowershell-interactive
$cred = Get-Credential
```

Ora si può creare un set di scalabilità di macchine virtuali con [New-AzureRmVmss](/powershell/module/azurerm.compute/new-azurermvmss). Per distribuire il traffico alle singole istanze di macchine virtuali, viene creato anche un servizio di bilanciamento del carico. Il servizio di bilanciamento del carico include regole per la distribuzione del traffico sulla porta TCP 80, oltre che per consentire il traffico di Desktop remoto sulla porta TCP 3389 e la comunicazione remota di PowerShell sulla porta TCP 5985:

```azurepowershell-interactive
New-AzureRmVmss `
  -ResourceGroupName "myResourceGroup" `
  -VMScaleSetName "myScaleSet" `
  -Location "EastUS" `
  -VirtualNetworkName "myVnet" `
  -SubnetName "mySubnet" `
  -PublicIpAddressName "myPublicIPAddress" `
  -LoadBalancerName "myLoadBalancer" `
  -Credential $cred
```

La creazione e la configurazione di tutte le macchine virtuali e risorse del set di scalabilità richiedono alcuni minuti.


## <a name="create-custom-script-extension-definition"></a>Creare una definizione di estensione Script personalizzato
Azure PowerShell usa una tabella hash per archiviare i file da scaricare e il comando da eseguire. Nell'esempio seguente viene usato script di esempio di GitHub. Creare prima questo oggetto di configurazione nel modo seguente:

```azurepowershell-interactive
$customConfig = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis.ps1"
}
```

Applicare ora l'estensione Script personalizzato con [Add-AzureRmVmssExtension](/powershell/module/AzureRM.Compute/Add-AzureRmVmssExtension). L'oggetto di configurazione definito in precedenza viene passato all'estensione. Aggiornare ed eseguire l'estensione nelle istanze di macchina virtuale con [Update-AzureRmVmss](/powershell/module/azurerm.compute/update-azurermvmss).

```azurepowershell-interactive
# Get information about the scale set
$vmss = Get-AzureRmVmss `
          -ResourceGroupName "myResourceGroup" `
          -VMScaleSetName "myScaleSet"

# Add the Custom Script Extension to install IIS and configure basic website
$vmss = Add-AzureRmVmssExtension `
  -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $customConfig

# Update the scale set and apply the Custom Script Extension to the VM instances
Update-AzureRmVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

Ogni istanza di macchina virtuale nel set di scalabilità scarica ed esegue lo script da GitHub. In un esempio più complesso sarebbe possibile installare più componenti e file di applicazione. Se si aumentano le prestazioni del set di scalabilità, le nuove istanze di macchina virtuale applicano automaticamente la stessa definizione dell'estensione Script personalizzato e installano l'applicazione necessaria.


## <a name="test-your-scale-set"></a>Testare il set di scalabilità
Per vedere il server Web in azione, ottenere l'indirizzo IP pubblico del servizio di bilanciamento del carico con il comando [Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress). L'esempio seguente ottiene l'indirizzo IP creato nel gruppo di risorse *myResourceGroup*:

```azurepowershell-interactive
Get-AzureRmPublicIpAddress -ResourceGroupName "myResourceGroup" | Select IpAddress
```

Immettere l'indirizzo IP pubblico del servizio di bilanciamento del carico in un Web browser. Il servizio di bilanciamento del carico distribuisce il traffico a una delle istanze di macchina virtuale, come illustrato nell'esempio seguente:

![Pagina Web di base in IIS](media/tutorial-install-apps-powershell/running-iis.png)

Lasciare aperto il Web browser per poter visualizzare una versione aggiornata nel passaggio successivo.


## <a name="update-app-deployment"></a>Aggiornare la distribuzione dell'app
Nel ciclo di vita di un set di scalabilità potrebbe essere necessario distribuire una versione aggiornata dell'applicazione. Con l'estensione Script personalizzato è possibile fare riferimento a uno script di distribuzione aggiornato e quindi riapplicare l'estensione al set di scalabilità. Quando è stato creato il set di scalabilità in un passaggio precedente, `-UpgradePolicyMode` è stato impostato su *Automatic*. Questa impostazione consente di aggiornare automaticamente le istanze di macchina virtuale nel set di scalabilità e di applicare automaticamente la versione più recente dell'applicazione.

Creare una nuova definizione di configurazione denominata *customConfigv2*. Questa definizione esegue una versione *v2* aggiornata dello script di installazione dell'applicazione:

```azurepowershell-interactive
$customConfigv2 = @{
  "fileUris" = (,"https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate-iis-v2.ps1");
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File automate-iis-v2.ps1"
}
```

Applicare di nuovo la configurazione dell'estensione Script personalizzato alle istanze di macchina virtuale nel set di scalabilità con [Add-AzureRmVmssExtension](/powershell/module/AzureRM.Compute/Add-AzureRmVmssExtension). La definizione *customConfigv2* viene usata per applicare la versione aggiornata dell'applicazione:

```azurepowershell-interactive
# Reapply the Custom Script Extension to install the updated website
$vmss = Add-AzureRmVmssExtension `
  -VirtualMachineScaleSet $vmss `
  -Name "customScript" `
  -Publisher "Microsoft.Compute" `
  -Type "CustomScriptExtension" `
  -TypeHandlerVersion 1.8 `
  -Setting $customConfigv2

# Update the scale set and reapply the Custom Script Extension to the VM instances
Update-AzureRmVmss `
  -ResourceGroupName "myResourceGroup" `
  -Name "myScaleSet" `
  -VirtualMachineScaleSet $vmss
```

Tutte le istanze di macchina virtuale nel set di scalabilità vengono aggiornate automaticamente con la versione più recente della pagina Web di esempio. Per visualizzare la versione aggiornata, aggiornare il sito Web nel browser:

![Pagina Web aggiornata in IIS](media/tutorial-install-apps-powershell/running-iis-updated.png)


## <a name="clean-up-resources"></a>Pulire le risorse
Per rimuovere il set di scalabilità e le risorse aggiuntive, eliminare il gruppo di risorse e tutte le relative risorse con [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup). Il parametro `-Force` conferma che si desidera eliminare le risorse senza un prompt aggiuntivo a tale scopo. Il parametro `-AsJob` restituisce il controllo al prompt senza attendere il completamento dell'operazione.

```azurepowershell-interactive
Remove-AzureRmResourceGroup -Name "myResourceGroup" -Force -AsJob
```


## <a name="next-steps"></a>Passaggi successivi
In questa esercitazione si è appreso come installare e aggiornare automaticamente applicazioni nel set di scalabilità con Azure PowerShell:

> [!div class="checklist"]
> * Installare automaticamente le applicazioni nel set di scalabilità
> * Usare l'estensione Script personalizzato di Azure
> * Aggiornare un'applicazione in esecuzione in un set di scalabilità

Passare all'esercitazione successiva per informazioni su come ridimensionare automaticamente il set di scalabilità.

> [!div class="nextstepaction"]
> [Ridimensionare automaticamente i set di scalabilità](tutorial-autoscale-powershell.md)
