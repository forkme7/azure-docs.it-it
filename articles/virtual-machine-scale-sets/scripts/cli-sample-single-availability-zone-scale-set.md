---
title: Esempi dell'interfaccia della riga di comando di Azure 2.0 - Set di scalabilità a singola zona | Microsoft Docs
description: Esempi dell'interfaccia della riga di comando di Azure 2.0
services: virtual-machine-scale-sets
documentationcenter: ''
author: iainfoulds
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machine-scale-sets
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/27/2018
ms.author: iainfou
ms.custom: mvc
ms.openlocfilehash: f881d90628f5e4b5c4fc3fe56ecb1b3377d8ba42
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 03/28/2018
---
# <a name="create-a-single-zone-virtual-machine-scale-set-with-powershell"></a>Creare un set di scalabilità di macchine virtuali a singola zona con PowerShell
Questo script crea un set di scalabilità di macchine virtuali che esegue Ubuntu in una singola zona di disponibilità. Dopo aver eseguito lo script, è possibile accedere alla macchina virtuale tramite RDP.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script di esempio
[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine-scale-sets/create-single-availability-zone/create-single-availability-zone.sh "Create single-zone scale set")]

## <a name="clean-up-deployment"></a>Pulire la distribuzione
Eseguire questo comando per rimuovere il gruppo di risorse, il set di scalabilità e tutte le risorse correlate.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Spiegazione dello script
Questo script usa i comandi seguenti per creare un gruppo di risorse, il set di scalabilità di macchine virtuali e tutte le risorse correlate. Ogni comando della tabella include collegamenti alla documentazione specifica del comando.

| Comando | Note |
|---|---|
| [az group create](/cli/azure/ad/group#az_ad_group_create) | Consente di creare un gruppo di risorse in cui sono archiviate tutte le risorse. |
| [az vmss create](/cli/azure/vmss#az_vmss_create) | Crea il set di scalabilità di macchine virtuali e lo connette alla rete virtuale, alla subnet e al gruppo di sicurezza di rete. Per distribuire il traffico a più istanze di macchina virtuale, viene anche creato un servizio di bilanciamento del carico. Questo comando specifica anche l'immagine della macchina virtuale da usare e le credenziali di amministrazione.  |
| [az group delete](/cli/azure/ad/group#delete) | Consente di eliminare un gruppo di risorse incluse tutte le risorse annidate. |

## <a name="next-steps"></a>Passaggi successivi
Per altre informazioni sull'interfaccia della riga di comando di Azure 2.0, vedere la [documentazione dell'interfaccia della riga di comando di Azure 2.0](https://docs.microsoft.com/cli/azure/overview).

Altri esempi di script dell'interfaccia della riga di comando di Azure 2.0 per i set di scalabilità di macchine virtuali sono disponibili nella [documentazione dei set di scalabilità di macchine virtuali di Azure](../cli-samples.md).