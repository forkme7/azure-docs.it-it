---
title: Come ottenere un tenant di Azure AD | Microsoft Docs
description: Come ottenere un tenant di Azure Active Directory per la registrazione e la creazione di applicazioni.
services: active-directory
documentationcenter: ''
author: mtillman
manager: mtillman
editor: ''
ms.assetid: 1f4b24eb-ab4d-4baa-a717-2a0e5b8d27cd
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: hero-article
ms.date: 03/23/2018
ms.author: mtillman
ms.custom: aaddev
ms.openlocfilehash: ab7db49fa07f260de6ebbe4b2cee943b64cab7fe
ms.sourcegitcommit: c3d53d8901622f93efcd13a31863161019325216
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 03/29/2018
---
# <a name="how-to-get-an-azure-active-directory-tenant"></a>Come ottenere un tenant di Azure Active Directory
In Azure Active Directory (Azure AD), un [tenant](https://msdn.microsoft.com/library/azure/jj573650.aspx#Anchor_0) rappresenta un'organizzazione.  Si tratta di un'istanza dedicata del servizio Azure AD che l'organizzazione riceve al momento della creazione di una relazione con Microsoft, ad esempio tramite l'iscrizione a un servizio cloud Microsoft come Azure, Microsoft Intune o Office 365, e che diventa di sua proprietà.  Ogni tenant di Azure AD è distinto e separato dagli altri tenant di Azure AD.  

Un tenant ospita gli utenti di un'azienda e le informazioni su di essi, ad esempio password dati del profilo utente, autorizzazioni e così via.  Contiene anche gruppi, applicazioni e altre informazioni relative all'azienda e alla sicurezza.

Per consentire agli utenti di Azure AD di accedere a un'applicazione, è necessario registrarla nel proprio tenant.  La creazione di un tenant di Azure AD e la pubblicazione di un'applicazione al suo interno sono operazioni **assolutamente gratuite** (benché sia possibile scegliere un piano a pagamento per ottenere funzionalità Premium per il tenant).  Molti sviluppatori creano infatti più tenant e applicazioni a fini di sperimentazione, sviluppo, staging e test.

## <a name="use-an-existing-azure-ad-tenant"></a>Usare un tenant Azure AD esistente

Molti sviluppatori dispongono già di tenant tramite servizi o sottoscrizioni associate ai tenant Azure AD (ad esempio, sottoscrizioni di Office 365 o Azure).  Per verificare se si dispone già di un tenant, accedere al [portale di Azure](https://portal.azure.com) con l'account che si vuole usare per gestire l'applicazione e controllare nell'angolo in alto a destra dove sono visualizzate le informazioni sull'account.  Se si dispone già di un tenant, verrà effettuata automaticamente la connessione e si vedrà il nome del tenant direttamente sotto il nome dell'account.  Se l'account è associato a più tenant, è possibile fare clic sul nome dell'account per aprire un menu in cui è possibile spostarsi tra i tenant.

Se non si dispone di un tenant esistente associato all'account, si noterà un GUID sotto il nome dell'account e non sarà possibile eseguire operazioni come la registrazione di app finché non si [crea un nuovo tenant](#create-a-new-azure-ad-tenant).

## <a name="create-a-new-azure-ad-tenant"></a>Creare un nuovo tenant Azure AD

Se non si dispone ancora di un tenant Azure AD o si vuole crearne uno nuovo, è possibile farlo tramite l'[esperienza di creazione della directory](https://portal.azure.com/#create/Microsoft.AzureActiveDirectory) nel [portale di Azure](https://portal.azure.com).  Il processo richiede circa un minuto e alla fine verrà chiesto di passare al tenant appena creato.