---
title: "Esempio JSON di Criteri di Azure: controllare l'assenza di amministratore per Azure Active Directory| Microsoft Docs"
description: Questo esempio di criteri JSON controlla quando non vi è alcun amministratore di Azure Active Directory assegnato a SQL Server.
services: azure-policy
documentationcenter: ''
author: DCtheGeek
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-policy
ms.devlang: ''
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: ''
ms.date: 11/13/2017
ms.author: dacoulte
ms.custom: mvc
ms.openlocfilehash: 576916e9a5702a3c7c21f1dbf2276a81c96a908e
ms.sourcegitcommit: e2adef58c03b0a780173df2d988907b5cb809c82
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/28/2018
---
# <a name="audit-no-azure-active-directory-administrator"></a>Controlla l'assenza di un amministratore per Azure Active Directory

Controlla quando non vi è nessun amministratore di Azure Active Directory assegnato a SQL Server.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-template"></a>Modello di esempio

[!code-json[main](../../../policy-templates/samples/SQL/audit-if-no-sql-active-directory-admin/azurepolicy.json "Audit SQL DB Level Audit Setting")]

È possibile distribuire questo modello usando il [portale di Azure](#deploy-with-the-portal), con [PowerShell](#deploy-with-powershell) o con l'[interfaccia della riga di comando di Azure](#deploy-with-azure-cli).

## <a name="deploy-with-the-portal"></a>Eseguire la distribuzione con il portale

[![Distribuzione in Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/?feature.customportal=false&microsoft_azure_policy=true&microsoft_azure_policy_policyinsights=true&feature.microsoft_azure_security_policy=true&microsoft_azure_marketplace_policy=true#blade/Microsoft_Azure_Policy/CreatePolicyDefinitionBlade/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-policy%2Fmaster%2Fsamples%2FSQL%2Faudit-if-no-sql-active-directory-admin%2Fazurepolicy.json)

## <a name="deploy-with-powershell"></a>Distribuire con PowerShell

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

```powershell
$definition = New-AzureRmPolicyDefinition -Name "audit-if-no-sql-active-directory-admin" -DisplayName "Audit If no AAD Admin" -description "Aduit If there is no AAD Admin assigned to this server" -Policy 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/SQL/audit-if-no-sql-active-directory-admin/azurepolicy.rules.json' -Parameter 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/SQL/audit-if-no-sql-active-directory-admin/azurepolicy.parameters.json' -Mode All
$definition
$assignment = New-AzureRMPolicyAssignment -Name <assignmentname> -Scope <scope>  -PolicyDefinition $definition
$assignment
```

### <a name="clean-up-powershell-deployment"></a>Eliminare la distribuzione di PowerShell

Eseguire questo comando per rimuovere il gruppo di risorse, la macchina virtuale e tutte le risorse correlate.

```powershell
Remove-AzureRmResourceGroup -Name myResourceGroup
```

## <a name="deploy-with-azure-cli"></a>Distribuire con l'interfaccia della riga di comando di Azure

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

```azurecli-interactive
az policy definition create --name 'audit-if-no-sql-active-directory-admin' --display-name 'Audit If no AAD Admin' --description 'Aduit If there is no AAD Admin assigned to this server' --rules 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/SQL/audit-if-no-sql-active-directory-admin/azurepolicy.rules.json' --params 'https://raw.githubusercontent.com/Azure/azure-policy/master/samples/SQL/audit-if-no-sql-active-directory-admin/azurepolicy.parameters.json' --mode All

az policy assignment create --name <assignmentname> --scope <scope> --policy "audit-if-no-sql-active-directory-admin" 
```

### <a name="clean-up-azure-cli-deployment"></a>Eliminare la distribuzione dell'interfaccia della riga di comando di Azure

Eseguire questo comando per rimuovere il gruppo di risorse, la macchina virtuale e tutte le risorse correlate.

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Passaggi successivi

- I modelli di Criteri di Azure di esempio aggiuntivi sono disponibili nella pagina [Templates for Azure Policy](../json-samples.md) (Modelli per Criteri di Azure).
