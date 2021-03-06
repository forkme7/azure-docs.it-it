---
title: Azure Multi-Factor Authentication - Come funziona
description: Azure multi-Factor Authentication contribuisce a salvaguardare l'accesso a dati e applicazioni rispondendo alla richiesta degli utenti di poter usare un processo di accesso semplice. Offre ulteriore sicurezza richiedendo una seconda forma di autenticazione nonché un'autenticazione avanzata tramite una gamma di opzioni di verifica semplice.
services: multi-factor-authentication
documentationcenter: ''
author: MicrosoftGuyJFlo
manager: mtillman
ms.assetid: d14db902-9afe-4fca-b3a5-4bd54b3d8ec5
ms.service: multi-factor-authentication
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 06/20/2017
ms.author: joflore
ms.reviewer: richagi
ms.custom: it-pro
ms.openlocfilehash: 3df8d4af21fe37a44983da1aae330785b62323db
ms.sourcegitcommit: fa493b66552af11260db48d89e3ddfcdcb5e3152
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/23/2018
---
# <a name="how-azure-multi-factor-authentication-works"></a>Come funziona Azure Multi-Factor Authentication
La sicurezza della verifica in due passaggi sta nel suo approccio a livelli. La manomissione di più fattori rappresenta una sfida significativa per gli autori di attacchi. Tuttavia, anche se un autore di attacco riesce a ottenere la password dell'utente, questa risulta inutile se non è in possesso del dispositivo attendibile. 

![Verifica](./media/concept-mfa-howitworks/howitworks.png)

Azure multi-Factor Authentication contribuisce a salvaguardare l'accesso a dati e applicazioni rispondendo alla richiesta degli utenti di poter usare un processo di accesso semplice.  Offre ulteriore sicurezza richiedendo una seconda forma di autenticazione nonché un'autenticazione avanzata tramite una gamma di opzioni di verifica semplice.


## <a name="methods-available-for-two-step-verification"></a>Metodi disponibili per la verifica in due passaggi
Quando un utente accede, una verifica aggiuntiva viene inviata all'utente.  Di seguito è riportato un elenco di metodi che possono essere utilizzati per la seconda verifica.

| Metodo di verifica | DESCRIZIONE |
| --- | --- |
| Chiamata telefonica |Viene eseguita una chiamata al telefono registrato di un utente. Se è necessario l'utente immette un PIN, quindi preme il tasto #. |
| SMS |Viene inviato un SMS al cellulare dell'utente con un codice di sei cifre. L'utente immette il codice nella pagina di accesso. |
| Notifica dell'app per dispositivi mobili |Viene inviata una richiesta di verifica allo smartphone dell'utente. L'utente immette il PIN, se necessario, quindi seleziona **Verifica** nell'app per dispositivi mobili. |
| Codice di verifica dell'app per dispositivi mobili |L'app per dispositivi mobili, che è in esecuzione sullo smartphone dell'utente, mostra un codice di verifica che cambia ogni 30 secondi. L'utente individua il codice più recente e lo inserisce nella pagina di accesso. |
| Token OATH di terze parti | Il server Azure Multi-Factor Authentication può essere configurato per accettare metodi di verifica di terze parti. |

Azure multi-Factor Authentication fornisce metodi di verifica selezionabili per cloud e server. È possibile scegliere i metodi disponibili per gli utenti: telefonata, SMS, notifica dell'app o codici dell'app. Per altre informazioni, vedere i [metodi di verifica selezionabili](howto-mfa-mfasettings.md#selectable-verification-methods).

## <a name="next-steps"></a>Passaggi successivi

- Informazioni sulle diverse [versioni e i metodi di utilizzo per Multi-Factor Authentication](concept-mfa-licensing.md)

- Scegliere se distribuire Azure MFA [nel cloud o in locale](concept-mfa-whichversion.md)

- Risposte alle [Domande frequenti](multi-factor-authentication-faq.md)