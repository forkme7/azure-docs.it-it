---
title: Rendere persistenti i file in PowerShell in Azure Cloud Shell (anteprima) | Microsoft Docs
description: Procedura dettagliata su come Azure Cloud Shell rende persistenti i file.
services: azure
documentationcenter: 
author: maertendmsft
manager: timlt
tags: azure-resource-manager
ms.assetid: 
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 01/30/2018
ms.author: damaerte
ms.openlocfilehash: 74488b85ec524e4ad4c06a639a16ddbfd54b3154
ms.sourcegitcommit: 9d317dabf4a5cca13308c50a10349af0e72e1b7e
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 02/01/2018
---
[!INCLUDE [features-introblock](../../includes/cloud-shell-persisting-shell-storage-introblock.md)]

## <a name="how-powershell-in-azure-cloud-shell-preview-works"></a>Come funziona PowerShell in Azure Cloud Shell (anteprima)
PowerShell in Cloud Shell (anteprima) rende persistenti i file tramite il metodo seguente: 
* Montaggio della condivisione file di Azure specificata come `clouddrive` nella directory `$Home` per l'interazione diretta con la condivisione file.

## <a name="list-cloud-drive-azure-file-shares"></a>Elencare le condivisioni file di Azure nell'unità cloud
Il comando `Get-CloudDrive` recupera le informazioni sulle condivisioni file di Azure attualmente montate dall'unità cloud in Cloud Shell. <br>
![Esecuzione di Get-CloudDrive](media/persisting-shell-storage-powershell/Get-Clouddrive.png)

## <a name="unmount-cloud-drive"></a>Smontare l'unità Cloud
È possibile smontare in qualsiasi momento una condivisione file di Azure montata in Cloud Shell. Se la condivisione file di Azure è stata rimossa, alla sessione successiva verrà chiesto di crearne e montarne una nuova.

Il comando `Dismount-CloudDrive` smonta una condivisione file di Azure dall'account di archiviazione corrente. Se si smonta l'unità Cloud, viene terminata la sessione corrente. All'utente viene chiesto di creare e montare una nuova condivisione file di Azure durante la sessione successiva.
![Esecuzione di Dismount-CloudDrive](media/persisting-shell-storage-powershell/Dismount-Clouddrive.png)

[!INCLUDE [features-endblock](../../includes/cloud-shell-persisting-shell-storage-endblock.md)]

## <a name="next-steps"></a>Passaggi successivi
[Avvio rapido di PowerShell](quickstart-powershell.md) <br>
[Informazioni su File di Azure](https://docs.microsoft.com/azure/storage/storage-introduction#file-storage) <br>
[Informazioni sui tag di archiviazione](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-using-tags) <br>