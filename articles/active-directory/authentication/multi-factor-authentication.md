---
title: Informazioni sulla verifica in due passaggi in Azure MFA | Documentazione Microsoft
description: "Cos'è Azure Multi-Factor Authentication (MFA) e perché usare questo servizio, informazioni sul client Multi-Factor Authentication e sui diversi metodi e versioni disponibili. "
keywords: Introduzione a MFA, panoramica di mfa, che cos'è mfa
services: multi-factor-authentication
documentationcenter: ''
author: MicrosoftGuyJFlo
manager: mtillman
ms.reviewer: richagi
ms.assetid: c40d7a34-1274-4496-96b0-784850c06e9b
ms.service: multi-factor-authentication
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/03/2017
ms.author: joflore
ms.openlocfilehash: 74441b5053a9c7c25d94406293828fe92f210bee
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/23/2018
---
# <a name="what-is-azure-multi-factor-authentication"></a>Informazioni su Azure Multi-Factor Authentication
La verifica in due passaggi è un metodo di autenticazione che richiede più di un metodo di verifica e con il quale viene aggiunto un secondo livello di sicurezza critico agli accessi e alle transazioni degli utenti. In genere richiede due o più dei metodi di verifica seguenti:

* Un'informazione nota (in genere una password)
* Un oggetto che si possiede (un dispositivo attendibile non facile da duplicare, ad esempio un telefono)
* Una caratteristica fisica dell'utente (biometrica)

<center>![Nome utente e password](./media/multi-factor-authentication/pword.png) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![Certificati](./media/multi-factor-authentication/phone.png) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![Smartphone](./media/multi-factor-authentication/hware.png) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![Smart card](./media/multi-factor-authentication/smart.png) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![Smart card virtuale](./media/multi-factor-authentication/vsmart.png) &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![Nome utente e password](./media/multi-factor-authentication/cert.png)</center>

Azure Multi-Factor Authentication (MFA) è una soluzione di verifica in due passaggi di Microsoft. Azure MFA consente di proteggere l'accesso ai dati e alle applicazioni dell'utente, garantendo al tempo stesso una procedura di accesso semplice. Offre autenticazione avanzata tramite una gamma di metodi di verifica, fra cui una telefonata, un SMS o una verifica dell'app per dispositivi mobili.

## <a name="why-use-azure-multi-factor-authentication"></a>Vantaggi dell'uso di Azure Multi-Factor Authentication
Oggi più che mai le persone sono sempre più connesse. Con Smartphone, tablet, laptop e PC, gli utenti dispongono di più opzioni per accedere ai propri account e alle applicazioni da qualsiasi luogo e di rimanere connessi in qualsiasi momento.

Azure Multi-Factor Authentication è una soluzione semplice da usare, scalabile e affidabile che offre un secondo metodo di autenticazione per proteggere gli utenti.

| ![Facile da usare](./media/multi-factor-authentication/simple.png) | ![Scalabile](./media/multi-factor-authentication/scalable.png) | ![Sempre protetti](./media/multi-factor-authentication/protected.png) | ![Affidabile](./media/multi-factor-authentication/reliable.png) |
|:---:|:---:|:---:|:---:|
| **Facile da usare** |**Scalabile** |**Sempre protetti** |**Affidabile** |

* **Facile da usare** : Azure Multi-Factor Authentication è semplice da configurare e usare. La protezione aggiuntiva offerta da Azure Multi-Factor Authentication consente agli utenti di usare e gestire i propri dispositivi. L'aspetto più importante è che in molti casi può essere configurata con pochi semplici clic.
* **Scalabile** : Azure Multi-Factor Authentication usa le potenzialità del cloud e si integra con Active Directory locale e con le app personalizzate. Questa protezione viene estesa anche agli scenari di importanza strategica con volumi elevati.
* **Sempre protetti** : Azure multi-Factor Authentication fornisce autenticazione avanzata grazie all'uso dei più elevati standard di settore.
* **Affidabile**: Microsoft garantisce il 99,9% della disponibilità di Azure Multi-Factor Authentication. Il servizio viene considerato non disponibile quando non è in grado di ricevere o elaborare le richieste di verifica per la verifica in due passaggi.

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/Windows-Azure-Multi-Factor-Authentication/player]


## <a name="next-steps"></a>Passaggi successivi

- Informazioni su [come funziona Azure Multi-Factor Authentication](concept-mfa-howitworks.md)

- Informazioni sulle diverse [versioni e i metodi di utilizzo per Multi-Factor Authentication](concept-mfa-licensing.md)
