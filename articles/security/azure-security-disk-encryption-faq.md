---
title: Domande frequenti su Crittografia dischi di Azure | Microsoft Docs
description: Questo articolo fornisce le risposte alle domande frequenti su Crittografia dischi di Microsoft Azure per le macchine virtuali IaaS Windows e Linux.
services: security
documentationcenter: na
author: DevTiw
manager: avibm
editor: barclayn
ms.assetid: 7188da52-5540-421d-bf45-d124dee74979
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/18/2018
ms.author: devtiw
ms.openlocfilehash: cb523b4fbf6e8abdf5c5158ab041d3485add9b23
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/16/2018
---
# <a name="azure-disk-encryption-faq"></a>Domande frequenti su Crittografia dischi di Azure

Questo articolo fornisce le risposte alle domande frequenti (FAQ) su Crittografia dischi di Azure per le macchine virtuali IaaS Windows e Linux. Per altre informazioni su questo servizio, vedere [Crittografia dischi di Azure per macchine virtuali IaaS Windows e Linux](https://docs.microsoft.com/azure/security/azure-security-disk-encryption).

## <a name="where-is-azure-disk-encryption-in-general-availability-ga"></a>Dove è presente Crittografia dischi di Azure con disponibilità generale (GA)?

Crittografia dischi di Azure per VM IaaS Windows e Linux è presente con disponibilità generale in tutte le aree pubbliche di Azure.

## <a name="what-user-experiences-are-available-with-azure-disk-encryption"></a>Quali esperienze utente sono disponibili con Crittografia dischi di Azure?

La versione GA di Crittografia dischi di Azure supporta i modelli di Azure Resource Manager, Azure PowerShell e l'interfaccia della riga di comando di Azure. Ciò consente una notevole flessibilità. Sono disponibili tre diverse opzioni per l'abilitazione della crittografia del disco per le macchine virtuali IaaS. Per altre informazioni sull'esperienza utente e indicazioni dettagliate per Crittografia dischi di Azure, vedere gli [scenari e le esperienze di distribuzione di Crittografia dischi di Azure](azure-security-disk-encryption.md#disk-encryption-deployment-scenarios-and-user-experiences).

## <a name="how-much-does-azure-disk-encryption-cost"></a>Quanto costa Crittografia dischi di Azure?

Non è previsto alcun addebito per la crittografia dei dischi delle macchine virtuali con Crittografia dischi di Azure.

## <a name="which-virtual-machine-tiers-does-azure-disk-encryption-support"></a>Quali livelli di macchine virtuali sono supportati da Crittografia dischi di Azure?

Crittografia dischi di Azure è disponibile nelle VM di livello standard tra cui le VM IaaS delle serie [A, D, DS, G, GS e F](https://azure.microsoft.com/pricing/details/virtual-machines/). È anche disponibile per le VM con archiviazione premium. Non è disponibile per le VM del livello Basic.

## <a name="what-linux-distributions-does-azure-disk-encryption-support"></a>Quali distribuzioni di Linux sono supportate da Crittografia dischi di Azure?

Crittografia dischi di Azure è supportata nelle distribuzioni e versioni del server Linux seguenti:

| Distribuzione Linux | Version | Tipo di volume supportato per la crittografia|
| --- | --- |--- |
| Ubuntu | 16.04-DAILY-LTS | Disco del sistema operativo e dati |
| Ubuntu | 14.04.5-DAILY-LTS | Disco del sistema operativo e dati |
| RHEL | 7.4 | Disco dati* |
| RHEL | 7.3 | Disco dati* |
| RHEL | 7,2 | Disco dati* |
| RHEL | 6.8 | Disco dati* |
| RHEL | 6.7 | Disco dati* |
| CentOS | 7.3 | Disco del sistema operativo e dati |
| CentOS | 7.2n | Disco del sistema operativo e dati |
| CentOS | 6.8 | Disco del sistema operativo e dati |
| CentOS | 7.1 | Disco dati |
| CentOS | 7.0 | Disco dati |
| CentOS | 6.7 | Disco dati |
| CentOS | 6.6 | Disco dati |
| CentOS | 6,5 | Disco dati |
| openSUSE | 13.2 | Disco dati |
| SLES | 12 SP1 | Disco dati |
| SLES | Priority:12-SP1 | Disco dati |
| SLES | HPC 12 | Disco dati |
| SLES | Priority:11-SP4 | Disco dati |
| SLES | 11 SP4 | Disco dati |

*__ADE è supportato per RHEL per il disco dati. L'implementazione ADE corrente funziona per il disco del sistema operativo, ma non è attualmente supportata congiuntamente. Microsoft e Red Hat stanno collaborato per sviluppare una soluzione congiunta supportata. Nel frattempo è possibile fare riferimento al white paper ADE per la crittografia del disco del sistema operativo Linux [qui](https://docs.microsoft.com/en-us/azure/security/azure-security-disk-encryption).__

## <a name="how-can-i-start-using-azure-disk-encryption"></a>Come si può iniziare a usare Crittografia dischi di Azure?

Per iniziare, leggere il white paper [Crittografia dischi di Azure per l'anteprima di macchine virtuali IaaS Windows e Linux](https://docs.microsoft.com/azure/security/azure-security-disk-encryption).

## <a name="can-i-encrypt-both-boot-and-data-volumes-with-azure-disk-encryption"></a>È possibile crittografare sia i volumi di avvio che i volumi di dati con Crittografia dischi di Azure?

Sì, è possibile crittografare volumi di avvio e volumi di dati per le macchine virtuali IaaS Windows e Linux. Per le macchine virtuali Windows, non è possibile crittografare i dati senza prima crittografare il volume del sistema operativo. Per le macchine virtuali Linux, è possibile crittografare il volume dei dati senza dover prima crittografare il volume del sistema operativo. Dopo aver crittografato il volume del sistema operativo per Linux, la disabilitazione della crittografia in un volume del sistema operativo per le macchine virtuali IaaS Linux non è supportata.

## <a name="does-azure-disk-encryption-allow-you-to-bring-your-own-key-byok-capability"></a>Con Crittografia dischi di Azure sono disponibili funzionalità BYOK (Bring Your Own Key)?

Sì, è possibile fornire le proprie chiavi di crittografia della chiave. Queste chiavi sono protette in Azure Key Vault, l'archivio delle chiavi per Crittografia dischi di Azure. Per altre informazioni sugli scenari di supporto delle chiavi di crittografia della chiave, vedere gli [scenari e le esperienze di distribuzione di Crittografia dischi di Azure](azure-security-disk-encryption.md#disk-encryption-deployment-scenarios-and-user-experiences).

## <a name="can-i-use-an-azure-created-key-encryption-key"></a>È possibile usare una chiave di crittografia della chiave creata in Azure?

Sì, è possibile usare Azure Key Vault per generare una chiave di crittografia della chiave per l'uso con Crittografia dischi di Azure. Queste chiavi sono protette in Azure Key Vault, l'archivio delle chiavi per Crittografia dischi di Azure. Per altre informazioni sugli scenari di supporto delle chiavi di crittografia della chiave, vedere gli [scenari e le esperienze di distribuzione di Crittografia dischi di Azure](azure-security-disk-encryption.md#disk-encryption-deployment-scenarios-and-user-experiences).

## <a name="can-i-use-an-on-premises-key-management-service-or-hsm-to-safeguard-the-encryption-keys"></a>È possibile usare un servizio di gestione delle chiavi/modulo di protezione hardware locale per proteggere le chiavi di crittografia?

Non è possibile usare il servizio di gestione delle chiavi o modulo di protezione hardware locale per proteggere le chiavi di crittografia con Crittografia dischi di Azure. Per proteggere le chiavi di crittografia, è possibile usare solo il servizio Azure Key Vault. Per altre informazioni sugli scenari di supporto delle chiavi di crittografia della chiave, vedere gli [scenari e le esperienze di distribuzione di Crittografia dischi di Azure](azure-security-disk-encryption.md#disk-encryption-deployment-scenarios-and-user-experiences).

## <a name="what-are-the-prerequisites-to-configure-azure-disk-encryption"></a>Quali sono i prerequisiti per la configurazione di Crittografia dischi di Azure?

Come prerequisito è richiesto uno script di PowerShell. Con questo script è possibile creare un'applicazione Azure Active Directory, creare un nuovo insieme di credenziali delle chiavi o configurare l'insieme di credenziali delle chiavi esistente per l'accesso alla crittografia del disco, per abilitare la crittografia e proteggere i segreti e le chiavi. Per altre informazioni sugli scenari di supporto delle chiavi di crittografia della chiave, vedere i [prerequisiti, gli scenari e le esperienze di distribuzione di Crittografia dischi di Azure](azure-security-disk-encryption.md#prerequisites).

## <a name="where-can-i-get-more-information-on-how-to-use-powershell-for-configuring-azure-disk-encryption"></a>Dove si possono ottenere altre informazioni su come usare PowerShell per la configurazione di Crittografia dischi di Azure?

Sono disponibili alcuni interessanti articoli su come eseguire le attività di base in Crittografia dischi di Azure, oltre a scenari più avanzati. Per le attività di base, vedere [Esplorare Crittografia dischi di Azure con Azure PowerShell - Parte 1](https://blogs.msdn.microsoft.com/azuresecurity/2015/11/16/explore-azure-disk-encryption-with-azure-powershell/). Per gli scenari più avanzati, vedere [Esplorare Crittografia dischi di Azure con Azure PowerShell - Parte 2](https://blogs.msdn.microsoft.com/azuresecurity/2015/11/21/explore-azure-disk-encryption-with-azure-powershell-part-2/).

## <a name="what-version-of-azure-powershell-does-azure-disk-encryption-support"></a>Quale versione di Azure PowerShell è supportata da Crittografia dischi di Azure?

Usare la versione più recente di Azure PowerShell SDK per configurare Crittografia dischi di Azure. Scaricare la versione più recente di [Azure PowerShell](https://github.com/Azure/azure-powershell/releases). Crittografia dischi di Azure *non* è supportato da Azure SDK versione 1.1.0.

> [!NOTE]
> L'estensione di anteprima di Crittografia dischi di Azure per Linux è deprecata. Per informazioni dettagliate, vedere [Deprecating Azure disk encryption preview extension for Linux IaaS VMs](https://blogs.msdn.microsoft.com/azuresecurity/2017/07/12/deprecating-azure-disk-encryption-preview-extension-for-linux-iaas-vms/) (Deprecazione dell'estensione dell'anteprima di Crittografia dischi di Azure per le VM IaaS Linux).

## <a name="can-i-apply-azure-disk-encryption-on-my-custom-linux-image"></a>È possibile applicare Crittografia dischi di Azure a un'immagine Linux personalizzata?

Non è possibile applicare Crittografia dischi di Azure a un'immagine Linux personalizzata. Sono supportate solo le immagini Linux per le distribuzioni supportate indicate sopra. Attualmente le immagini Linux personalizzate non sono supportate.

## <a name="can-i-apply-updates-to-a-linux-red-hat-vm-that-uses-the-yum-update"></a>È possibile applicare gli aggiornamenti a una VM Red Hat Linux che usa l'aggiornamento Yum?

Sì, è possibile eseguire un aggiornamento o applicare patch a una VM Red Hat Linux. Per altre informazioni, vedere [Applying updates to an encrypted Azure IaaS Red Hat VM by using the yum update](https://blogs.msdn.microsoft.com/azuresecurity/2017/07/13/applying-updates-to-a-encrypted-azure-iaas-red-hat-vm-using-yum-update/) (Applicazione di aggiornamenti a una VM Red Hat IaaS di Azure mediante l'aggiornamento Yum).

## <a name="what-is-the-recommended-azure-disk-encryption-workflow-for-linux"></a>Qual è il flusso di lavoro di crittografia dischi di Azure consigliato per Linux?

Di seguito è riportato il flusso di lavoro consigliato per ottenere risultati ottimali in Linux:
* Iniziare dall'immagine della raccolta non modificata corrispondente alla distribuzione e alla versione del sistema operativo desiderate
* Eseguire il backup di tutte le unità montate che verranno crittografate.  Questo permetterà il ripristino in caso di errore, ad esempio se la macchina virtuale viene riavviata prima del completamento della crittografia.
* Eseguire la crittografia (può richiedere più ore o addirittura giorni a seconda delle caratteristiche della macchina virtuale e delle dimensioni di qualsiasi disco dati collegato)
* Personalizzare e aggiungere il software all'immagine in base alle esigenze.

Se questo flusso di lavoro non è possibile, l'uso della [crittografia del servizio di archiviazione](https://docs.microsoft.com/azure/storage/common/storage-service-encryption) a livello di account di archiviazione della piattaforma può essere un'alternativa alla crittografia completa del disco tramite dm-crypt.

## <a name="what-is-the-disk-bek-volume-or-mntazurebekdisk"></a>Che cos'è il disco "volume BEK" o "/mnt/azure_bek_disk"?

Il disco "volume BEK" per Windows o "/mnt/azure_bek_disk" per Linux è un volume dati locale in cui vengono archiviate in modo sicuro le chiavi di crittografia per le VM IaaS di Azure crittografate.
> [!NOTE]
> Non eliminare né modificare il contenuto del disco. Non smontare il disco, perché la presenza delle chiavi di crittografia è necessaria per qualsiasi operazione di crittografia sulla VM IaaS.

## <a name="where-can-i-go-to-ask-questions-or-provide-feedback"></a>Quali risorse sono disponibili per porre domande o inviare commenti?

È possibile porre domande o inviare commenti e suggerimenti sul [forum di Crittografia dischi di Azure disponibile](https://social.msdn.microsoft.com/Forums/en-US/home?forum=AzureDiskEncryption).

## <a name="next-steps"></a>Passaggi successivi
In questo documento sono state fornite le risposte alle domande più frequenti relative a Crittografia dischi di Azure. Per altre informazioni su questo servizio e sulle relative funzionalità, vedere gli articoli seguenti:

- [Applicare la crittografia dei dischi nel Centro sicurezza di Azure](https://docs.microsoft.com/azure/security-center/security-center-apply-disk-encryption)
- [Crittografare una macchina virtuale di Azure](https://docs.microsoft.com/azure/security-center/security-center-disk-encryption)
- [Crittografia dei dati inattivi in Azure](https://docs.microsoft.com/azure/security/azure-security-encryption-atrest)