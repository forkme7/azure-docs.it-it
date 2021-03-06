---
title: Introduzione all'autenticazione in Automazione di Azure
description: Questo articolo offre una panoramica della sicurezza in Automazione e dei diversi metodi di autenticazione disponibili per gli account di Automazione in Automazione di Azure.
keywords: sicurezza in Automazione, proteggere Automazione; autenticazione in Automazione
services: automation
ms.service: automation
author: georgewallace
ms.author: gwallace
ms.date: 03/19/2018
ms.topic: article
manager: carmonm
ROBOTS: NOINDEX
ms.openlocfilehash: 4280c0de3f5c38aa8a72bad391544b03116e26c9
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 03/23/2018
---
# <a name="introduction-to-authentication-in-azure-automation"></a>Introduzione all'autenticazione in Automazione di Azure  
Automazione di Azure consente di automatizzare le attività sulle risorse in Azure, in locale e con altri provider di servizi cloud, ad esempio Amazon Web Services (AWS).  Un runbook, per eseguire le azioni obbligatorie, deve avere le autorizzazioni per accedere in modo sicuro alle risorse con i diritti minimi necessari nella sottoscrizione.

Questo articolo illustra i diversi scenari di autenticazione supportati da Automazione di Azure e spiega come iniziare partendo da uno o più ambienti che è necessario gestire.  

## <a name="automation-account-overview"></a>Panoramica dell'account di Automazione
Al primo avvio di Automazione di Azure sarà necessario creare almeno un account di Automazione. Gli account di Automazione consentono di isolare le risorse di Automazione (runbook, asset, configurazioni) dalle risorse contenute in altri account di Automazione. È possibile usare gli account di Automazione per separare le risorse in ambienti logici distinti. Ad esempio, è possibile usare un account per lo sviluppo, uno per la produzione e un altro per l'ambiente locale.  Un account di Automazione di Azure è diverso dagli account Microsoft creati nella sottoscrizione di Azure.

Le risorse di Automazione per ogni account di Automazione sono associate a una singola area di Azure, ma gli account di Automazione possono gestire tutte le risorse nella sottoscrizione. Il motivo principale per cui creare gli account di Automazione in aree diverse è la presenza di criteri che richiedono dati e risorse per essere isolati in un'area specifica.

Tutte le attività eseguite sulle risorse con Azure Resource Manager e i cmdlet di Azure in Automazione di Azure devono eseguire l'autenticazione in Azure con l'autenticazione basata su credenziali dell'identità dell'organizzazione di Azure Active Directory.  L'autenticazione basata su certificati è il metodo di autenticazione originale con le risorse classiche di Azure, ma è complessa da configurare.  L'autenticazione in Azure con un utente di Azure AD è stata reintrodotta nel 2014 non solo per semplificare il processo di configurazione di un account di autenticazione, ma anche per offrire la possibilità di eseguire l'autenticazione in Azure in modo non interattivo con un solo account utente che funzionasse con risorse sia classiche che di Azure Resource Manager.   

Attualmente, quando si crea un nuovo account di Automazione nel portale di Azure viene creato automaticamente quanto segue:

* Account RunAs che crea una nuova entità servizio in Azure Active Directory e un certificato e assegna il ruolo Collaboratore per il controllo degli accessi in base al ruolo, usato per gestire le risorse di Resource Manager con i runbook.
* Account RunAs classico, caricando un certificato di gestione che viene usato per gestire le risorse classiche di Azure con i runbook.  

Il controllo degli accessi in base al ruolo è disponibile con Azure Resource Manager per l'esecuzione di azioni consentite con un account utente di Azure AD e per l'autenticazione di tale entità servizio.  Per altre informazioni utili per sviluppare il modello per la gestione delle autorizzazioni di Automazione, vedere [Controllo degli accessi in base al ruolo in Automazione di Azure](automation-role-based-access-control.md).  

I runbook eseguiti in un ruolo di lavoro ibrido per runbook nel data center o in servizi di calcolo in AWS non possono usare lo stesso metodo usato per i runbook che si autenticano per le risorse di Azure.  Il motivo è che queste risorse vengono eseguite all'esterno di Azure e di conseguenza devono usare le proprie credenziali di sicurezza definite in Automazione per autenticare le risorse cui accedono in locale.  

## <a name="authentication-methods"></a>Metodi di autenticazione
La tabella seguente riepiloga i diversi metodi di autenticazione per ogni ambiente supportato da Automazione di Azure e l'articolo che descrive come configurare l'autenticazione per i runbook.

| Metodo | Environment | Articolo |
| --- | --- | --- |
| Account utente di Azure AD |Azure Resource Manager e ambiente Azure classico |[Autenticare runbook con un account utente di Azure AD](automation-create-aduser-account.md) |
| Account RunAs di Azure |Gestione risorse di Azure |[Autenticare runbook con account RunAs di Azure](automation-sec-configure-azure-runas-account.md) |
| Account RunAs classico di Azure |Ambiente Azure classico |[Autenticare runbook con account RunAs di Azure](automation-sec-configure-azure-runas-account.md) |
| Autenticazione di Windows |Data Center locale |[Autenticare runbook per ruoli di lavoro ibridi per runbook](automation-hybrid-runbook-worker.md) |
| Credenziali AWS |Amazon Web Services |[Autenticare runbook con Amazon Web Services (AWS)](automation-config-aws-account.md) |
