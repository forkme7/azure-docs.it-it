---
title: Creare una macchina virtuale Linux usando PowerShell nello Stack di Azure | Documenti Microsoft
description: Creare una macchina virtuale Linux con PowerShell nello Stack di Azure.
services: azure-stack
documentationcenter: ''
author: mattbriggs
manager: femila
editor: ''
ms.assetid: 03EE5929-4F05-47D7-B246-EA93D6FC47CD
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 04/24/2018
ms.author: mabrigg
ms.custom: mvc
ms.openlocfilehash: 86597defad7c76d41065270030a4c77ee901b014
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 04/28/2018
---
# <a name="quickstart-create-a-linux-server-virtual-machine-by-using-powershell-in-azure-stack"></a>Guida introduttiva: creare una macchina virtuale di Linux server utilizzando PowerShell nello Stack di Azure

*Si applica a: Azure Stack integrate di sistemi Azure Stack Development Kit*

È possibile creare una macchina virtuale Ubuntu Server 16.04 LTS tramite Azure PowerShell dello Stack. Seguire i passaggi descritti in questo articolo per creare e usare una macchina virtuale.  In questo articolo offre inoltre la procedura per:

* Connettersi alla macchina virtuale con un client remoto.
* Pulire le risorse inutilizzate.

## <a name="prerequisites"></a>Prerequisiti

* **Un'immagine Linux nel marketplace Azure Stack**

   Per impostazione predefinita, il marketplace Azure Stack non contiene un'immagine Linux. Ottenere l'operatore di Stack di Azure per fornire il **Ubuntu Server 16.04 LTS** immagine necessaria. L'operatore può utilizzare i passaggi descritti nel [scaricare elementi del marketplace da Azure a Azure Stack](../azure-stack-download-azure-marketplace-item.md) articolo.

* Stack di Azure richiede una versione specifica di Azure PowerShell per creare e gestire le risorse. Se non è configurato per lo Stack di Azure PowerShell, accedi al [kit di sviluppo](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-remote-desktop), o un client esterno con codifica basata su Windows in caso di [connessi tramite VPN](azure-stack-connect-azure-stack.md#connect-to-azure-stack-with-vpn) e seguire i passaggi per [ installare](azure-stack-powershell-install.md) e [configurare](azure-stack-powershell-configure-user.md) PowerShell.

* Una chiave SSH pubblica con il nome di id_rsa.pub salvati nella directory .ssh del profilo utente Windows. Per informazioni dettagliate sulla creazione di chiavi SSH, vedere [creazione SSH chiavi in Windows](../../virtual-machines/linux/ssh-from-windows.md).

## <a name="create-a-resource-group"></a>Creare un gruppo di risorse

Un gruppo di risorse è un contenitore logico in cui è possibile distribuire e gestire le risorse di Azure Stack. Il kit di sviluppo o il sistema integrato dello Stack di Azure, eseguire il blocco di codice seguente per creare un gruppo di risorse. Vengono assegnati i valori per tutte le variabili in questo documento, è possibile utilizzare questi valori oppure assegnare loro nuovi valori.

```powershell
# Create variables to store the location and resource group names.
$location = "local"
$ResourceGroupName = "myResourceGroup"

New-AzureRmResourceGroup `
  -Name $ResourceGroupName `
  -Location $location
```

## <a name="create-storage-resources"></a>Creare risorse di archiviazione

Creare un account di archiviazione e quindi creare un contenitore di archiviazione per l'immagine Ubuntu Server 16.04 LTS.

```powershell
# Create variables to store the storage account name and the storage account SKU information
$StorageAccountName = "mystorageaccount"
$SkuName = "Standard_LRS"

# Create a new storage account
$StorageAccount = New-AzureRMStorageAccount `
  -Location $location `
  -ResourceGroupName $ResourceGroupName `
  -Type $SkuName `
  -Name $StorageAccountName

Set-AzureRmCurrentStorageAccount `
  -StorageAccountName $storageAccountName `
  -ResourceGroupName $resourceGroupName

# Create a storage container to store the virtual machine image
$containerName = 'osdisks'
$container = New-AzureStorageContainer `
  -Name $containerName `
  -Permission Blob
```

## <a name="create-networking-resources"></a>Creare risorse di rete

Creare una rete virtuale, una subnet e un indirizzo IP pubblico. Queste risorse sono utilizzate per fornire la connettività di rete alla macchina virtuale.

```powershell
# Create a subnet configuration
$subnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
  -Name mySubnet `
  -AddressPrefix 192.168.1.0/24

# Create a virtual network
$vnet = New-AzureRmVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -Name MyVnet `
  -AddressPrefix 192.168.0.0/16 `
  -Subnet $subnetConfig

# Create a public IP address and specify a DNS name
$pip = New-AzureRmPublicIpAddress `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -AllocationMethod Static `
  -IdleTimeoutInMinutes 4 `
  -Name "mypublicdns$(Get-Random)"

```

### <a name="create-a-network-security-group-and-a-network-security-group-rule"></a>Creare un gruppo di sicurezza di rete e una regola del gruppo di sicurezza di rete

Il gruppo di sicurezza di rete consente di proteggere la macchina virtuale, usando le regole in entrata e in uscita. Creare una regola in ingresso per la porta 3389 per consentire le connessioni Desktop remoto in ingresso e una regola in ingresso per la porta 80 per consentire il traffico web in ingresso.

```powershell
# Create an inbound network security group rule for port 22
$nsgRuleSSH = New-AzureRmNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleSSH  -Protocol Tcp `
-Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
-DestinationPortRange 22 -Access Allow

# Create an inbound network security group rule for port 80
$nsgRuleWeb = New-AzureRmNetworkSecurityRuleConfig -Name myNetworkSecurityGroupRuleWWW  -Protocol Tcp `
-Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
-DestinationPortRange 80 -Access Allow

# Create a network security group
$nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName myResourceGroup -Location local `
-Name myNetworkSecurityGroup -SecurityRules $nsgRuleSSH,$nsgRuleWeb
```

### <a name="create-a-network-card-for-the-virtual-machine"></a>Creare una scheda di rete per la macchina virtuale

La scheda di rete connette la macchina virtuale a una subnet, a un gruppo di sicurezza di rete e a un indirizzo IP pubblico.

```powershell
# Create a virtual network card and associate it with public IP address and NSG
$nic = New-AzureRmNetworkInterface `
  -Name myNic `
  -ResourceGroupName $ResourceGroupName `
  -Location $location `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id `
  -NetworkSecurityGroupId $nsg.Id
```

## <a name="create-a-virtual-machine"></a>Creare una macchina virtuale

Creare una configurazione di macchina virtuale. Questa configurazione include le impostazioni utilizzate quando si distribuisce la macchina virtuale. Ad esempio: le credenziali dell'utente, dimensioni e l'immagine di macchina virtuale.

```powershell
# Define a credential object.
$UserName='demouser'
$securePassword = ConvertTo-SecureString ' ' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ($UserName, $securePassword)

# Create the virtual machine configuration object
$VmName = "VirtualMachinelatest"
$VmSize = "Standard_D1"
$VirtualMachine = New-AzureRmVMConfig `
  -VMName $VmName `
  -VMSize $VmSize

$VirtualMachine = Set-AzureRmVMOperatingSystem `
  -VM $VirtualMachine `
  -Linux `
  -ComputerName "MainComputer" `
  -Credential $cred

$VirtualMachine = Set-AzureRmVMSourceImage `
  -VM $VirtualMachine `
  -PublisherName "Canonical" `
  -Offer "UbuntuServer" `
  -Skus "16.04-LTS" `
  -Version "latest"

$osDiskName = "OsDisk"
$osDiskUri = '{0}vhds/{1}-{2}.vhd' -f `
  $StorageAccount.PrimaryEndpoints.Blob.ToString(),`
  $vmName.ToLower(), `
  $osDiskName

# Sets the operating system disk properties on a virtual machine.
$VirtualMachine = Set-AzureRmVMOSDisk `
  -VM $VirtualMachine `
  -Name $osDiskName `
  -VhdUri $OsDiskUri `
  -CreateOption FromImage | `
  Add-AzureRmVMNetworkInterface -Id $nic.Id

# Configure SSH Keys
$sshPublicKey = Get-Content "$env:USERPROFILE\.ssh\id_rsa.pub"

# Adds the SSH Key to the virtual machine
Add-AzureRmVMSshPublicKey -VM $VirtualMachine `
 -KeyData $sshPublicKey `
 -Path "/home/azureuser/.ssh/authorized_keys"

# Create the virtual machine.
New-AzureRmVM `
  -ResourceGroupName $ResourceGroupName `
 -Location $location `
  -VM $VirtualMachine
```

## <a name="connect-to-the-virtual-machine"></a>Connettersi alla macchina virtuale

Dopo aver distribuita la macchina virtuale, configurare una connessione SSH per la macchina virtuale. Usare il comando [Get-AzureRmPublicIpAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress?view=azurermps-4.3.1) per ottenere l'indirizzo IP pubblico della macchina virtuale.

```powershell
Get-AzureRmPublicIpAddress -ResourceGroupName myResourceGroup | Select IpAddress
```

Da un sistema client con SSH installato, utilizzare il comando seguente per connettersi alla macchina virtuale. Se si lavora in Windows, è possibile utilizzare [Putty](http://www.putty.org/) per creare la connessione.

```
ssh <Public IP Address>
```

Quando richiesto, immettere azureuser come l'utente di accesso. Se è stata utilizzata una passphrase durante la creazione di chiavi SSH, sarà necessario fornire la passphrase.

## <a name="clean-up-resources"></a>Pulire le risorse

Pulire le risorse che non è necessario più. È possibile usare il [Remove-AzureRmResourceGroup](/powershell/module/azurerm.resources/remove-azurermresourcegroup?view=azurermps-4.3.1) comando per rimuovere queste risorse. Per eliminare il gruppo di risorse e tutte le relative risorse, eseguire il comando seguente:

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="next-steps"></a>Passaggi successivi

In questa Guida rapida, è stato distribuito una macchina virtuale di base Linux server. Per ulteriori informazioni sulle macchine virtuali di Azure Stack, passare a [considerazioni per le macchine virtuali in Azure Stack](azure-stack-vm-considerations.md).
