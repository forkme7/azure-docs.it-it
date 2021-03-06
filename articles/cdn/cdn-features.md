---
title: Funzionalità dei prodotti della rete per la distribuzione di contenuti di Azure (rete CDN) | Microsoft Docs
description: Informazioni sulle funzionalità supportate da ogni prodotto della rete per la distribuzione di contenuti di Azure (rete CDN).
services: cdn
documentationcenter: ''
author: dksimpson
manager: akucer
editor: ''
ms.assetid: ''
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 03/23/2018
ms.author: rli
ms.custom: mvc
ms.openlocfilehash: 40638e2180b63c90fbcbe15cc2c1e944a97e166e
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/05/2018
---
# <a name="azure-cdn-product-features"></a>Funzionalità del prodotto rete CDN di Azure

Per la rete per la distribuzione di contenuti di Azure (rete CDN) sono disponibili tre prodotti: **rete CDN Standard di Azure fornita da Akamai**, **rete CDN Standard di Azure fornita da Verizon** e **rete CDN Premium di Azure fornita da Verizon**. La tabella seguente confronta le funzionalità disponibili con ogni prodotto.

| **Ottimizzazioni e funzionalità per le prestazioni** | Standard Akamai | Standard Verizon | Premium Verizon |
| --- | --- | --- | --- |
| [Accelerazione sito dinamico](https://docs.microsoft.com/azure/cdn/cdn-dynamic-site-acceleration) | **&#x2713;**  | **&#x2713;** | **&#x2713;** |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Accelerazione sito dinamico - Compressione di immagini adattiva](https://docs.microsoft.com/azure/cdn/cdn-dynamic-site-acceleration#adaptive-image-compression-akamai-only) | **&#x2713;**  |  |  |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Accelerazione sito dinamico - Prelettura degli oggetti](https://docs.microsoft.com/azure/cdn/cdn-dynamic-site-acceleration#object-prefetch-akamai-only) | **&#x2713;**  |  |  |
| [Ottimizzazione dello streaming video](https://docs.microsoft.com/azure/cdn/cdn-media-streaming-optimization) | **&#x2713;**  | \* |  \* |
| [Ottimizzazione di file di grandi dimensioni](https://docs.microsoft.com/azure/cdn/cdn-large-file-optimization) | **&#x2713;**  | \* |  \* |
| [Bilanciamento del carico del server globale](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-load-balancing-azure) |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Eliminazione veloce](cdn-purge-endpoint.md) |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Precaricamento Asset](cdn-preload-endpoint.md) | |**&#x2713;** |**&#x2713;** |
| Impostazioni cache/intestazioni (con [regole di memorizzazione nella cache](cdn-caching-rules.md)) |**&#x2713;** |**&#x2713;** | |
| Impostazioni cache/intestazioni (con il [motore regole](cdn-rules-engine.md)) | | |**&#x2713;** |
| [Memorizzazione nella cache della stringa di query](cdn-query-string.md) |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| IPv4/IPv6 dual stack |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Supporto HTTP/2](cdn-http2.md) |**&#x2713;** |**&#x2713;** |**&#x2713;** |
||||
 **Sicurezza** | **Standard Akamai** | **Standard Verizon** | **Premium Verizon** | 
| Supporto per HTTPS con endpoint della rete CDN |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [HTTPS dominio personalizzato](cdn-custom-ssl.md) | |**&#x2713;** |**&#x2713;** |
| [Supporto del nome di dominio personalizzato.](cdn-map-content-to-custom-domain.md) |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Filtro geografico](cdn-restrict-access-by-country.md) |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Autenticazione tramite token](cdn-token-auth.md)|  |  |**&#x2713;**| 
| [Protezione DDoS](https://www.us-cert.gov/ncas/tips/ST04-015) |**&#x2713;** |**&#x2713;** |**&#x2713;** |
||||
| **Analisi e report** | **Standard Akamai** | **Standard Verizon** | **Premium Verizon** | 
| [Log di diagnostica di Azure](cdn-azure-diagnostic-logs.md) | **&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Report principali da Verizon](cdn-analyze-usage-patterns.md) | |**&#x2713;** |**&#x2713;** |
| [Report personalizzati da Verizon](cdn-verizon-custom-reports.md) | |**&#x2713;** |**&#x2713;** |
| [Report HTTP avanzati](cdn-advanced-http-reports.md) | | |**&#x2713;** |
| [Statistiche in tempo reale](cdn-real-time-stats.md) | | |**&#x2713;** |
| [Prestazioni del nodo perimetrale](cdn-edge-performance.md) | | |**&#x2713;** |
| [Avvisi in tempo reale](cdn-real-time-alerts.md) | | |**&#x2713;** |
||||
| **Semplicità d'uso** | **Standard Akamai** | **Standard Verizon** | **Premium Verizon** | 
| Semplice integrazione con i servizi di Azure, ad esempio [Archiviazione](cdn-create-a-storage-account-with-cdn.md), [Servizi cloud](cdn-cloud-service-with-cdn.md), [App Web](../app-service/app-service-web-tutorial-content-delivery-network.md) e [Servizi multimediali](../media-services/media-services-portal-manage-streaming-endpoints.md) |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| Gestione tramite [API REST](https://msdn.microsoft.com/library/mt634456.aspx), [.NET](cdn-app-dev-net.md), [Node.js](cdn-app-dev-node.md) oppure [PowerShell](cdn-manage-powershell.md) |**&#x2713;** |**&#x2713;** |**&#x2713;** |
| [Motore di distribuzione di contenuti personalizzabile, basato su regole](cdn-rules-engine.md) | | |**&#x2713;** |
| Riscrittura/reindirizzamento URL (con il [motore regole](cdn-rules-engine.md)) | | |**&#x2713;** |
| Regole per dispositivi mobili (con il [motore regole](cdn-rules-engine.md)) | | |**&#x2713;** |

\* Verizon supporta la distribuzione di file di grandi dimensioni e di file multimediali direttamente tramite l'ottimizzazione di distribuzione Web generale.



