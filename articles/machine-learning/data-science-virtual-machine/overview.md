---
title: Introduzione alla macchina virtuale data science di Azure per Linux e Windows | Microsoft Docs
description: Scenari e componenti chiave dell'analisi per le macchine virtuali data science per Windows e Linux.
keywords: strumenti di analisi scientifica dei dati, macchina virtuale per l'analisi scientifica dei dati, strumenti per l'analisi scientifica dei dati, analisi scientifica dei dati per Linux
services: machine-learning
documentationcenter: ''
author: gopitk
manager: cgronlun
ms.assetid: d4f91270-dbd2-4290-ab2b-b7bfad0b2703
ms.service: machine-learning
ms.component: data-science-vm
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 10/27/2017
ms.author: gokuma
ms.openlocfilehash: 9ef6b216889416ea00786dcd3043d6e0f246b305
ms.sourcegitcommit: 59914a06e1f337399e4db3c6f3bc15c573079832
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 04/19/2018
---
# <a name="introduction-to-azure-data-science-virtual-machine-for-linux-and-windows"></a>Introduzione alla macchina virtuale data science di Azure per Linux e Windows

La macchina virtuale per l'analisi scientifica dei dati (DSVM) è un'immagine VM personalizzata nel cloud di Microsoft Azure creata in modo specifico per l'analisi scientifica dei dati. Include diversi strumenti comuni per l'analisi scientifica dei dati e altri strumenti preinstallati e preconfigurati per implementare rapidamente la creazione di applicazioni intelligenti per l'analisi avanzata. È disponibile in Windows Server e in Linux. L'edizione della DSVM per Windows è disponibile in Windows Server 2016 e Windows Server 2012. Le edizioni della DSVM per Linux sono disponibili in Ubuntu 16.04 LTS e CentOS 7.4.

Questo argomento illustra le operazioni possibili con la VM per l'analisi scientifica dei dati, descrive alcuni degli scenari chiave per l'uso della VM, indica in modo dettagliato le funzionalità principali disponibili nelle versioni per Windows e Linux e include le istruzioni su come iniziare a usarle.


## <a name="what-can-i-do-with-the-data-science-virtual-machine"></a>Usi della macchina virtuale per l'analisi scientifica dei dati
L'obiettivo della macchina virtuale per l'analisi scientifica dei dati è offrire ai professionisti dei dati, in tutti i livelli e i ruoli, un ambiente di analisi scientifica dei dati privo di problemi, preconfigurato e totalmente integrato. Anziché implementare un'area di lavoro analoga, è possibile effettuare il provisioning di una macchina virtuale per l'analisi scientifica e risparmiare così giorni o persino _settimane_ per i processi di installazione, configurazione e gestione dei pacchetti. Dopo aver allocato la macchina virtuale per l'analisi scientifica, è possibile iniziare immediatamente a lavorare al progetto di analisi scientifica dei dati.

La VM data science è progettata e configurata per essere usata in un'ampia gamma di scenari di utilizzo. È possibile aumentare e ridurre le prestazioni dell'ambiente a seconda delle esigenze del progetto. Può essere usato il linguaggio preferito per programmare le attività di analisi scientifica dei dati. È inoltre possibile installare altri strumenti e personalizzare il sistema in base alle esigenze specifiche.

## <a name="key-scenarios"></a>Scenari chiave
Questa sezione suggerisce alcuni scenari chiave per i quali può essere distribuita la VM per l'analisi scientifica dei dati.

### <a name="preconfigured-analytics-desktop-in-the-cloud"></a>Desktop di analisi preconfigurato nel cloud
La VM per l'analisi scientifica dei dati offre una configurazione di base per i team di analisi scientifica dei dati che vogliono sostituire i desktop locali con un desktop cloud gestito. Questa configurazione di base garantisce che tutti gli scienziati dei dati presenti in un team abbiano una configurazione coerente mediante la quale verificare gli esperimenti e promuovere la collaborazione. Anche i costi diminuiscono, grazie alla riduzione del carico lavorativo per gli amministratori di sistema e del tempo richiesto per valutare, installare e gestire i vari pacchetti di software necessari per eseguire analisi avanzate.  

### <a name="data-science-training-and-education"></a>Preparazione e formazione sull'analisi scientifica dei dati
Gli istruttori e i formatori aziendali che insegnano l'analisi scientifica dei dati in genere forniscono un'immagine di macchina virtuale per garantire che gli studenti abbiano una configurazione coerente e che gli esempi abbiano un comportamento prevedibile. La VM per l'analisi scientifica dei dati consente di creare un ambiente su richiesta con una configurazione coerente che semplifica i problemi relativi a incompatibilità e supporto. Esistono vantaggi sostanziali per i casi in cui tali ambienti devono essere compilati di frequente, in particolare per i corsi di formazione più brevi.

### <a name="on-demand-elastic-capacity-for-large-scale-projects"></a>Capacità elastica su richiesta per progetti su larga scala
Gli hackathon e i concorsi di analisi scientifica dei dati o la modellazione e l'esplorazione di dati su larga scala richiedono una maggiore capacità hardware, in genere per brevi periodi di tempo. La VM per l'analisi scientifica dei dati consente di replicare rapidamente e su richiesta l'ambiente di analisi scientifica dei dati su server con maggiore capacità, che consentono di eseguire esperimenti che richiedono l'esecuzione di risorse di calcolo con potenza elevata.

### <a name="short-term-experimentation-and-evaluation"></a>Valutazione e sperimentazione a breve termine
La VM per l'analisi scientifica dei dati può essere usata per valutare o imparare a usare strumenti quali Microsoft ML Server, SQL Server, strumenti di Visual Studio, Jupyter, toolkit di deep learning/ML e i nuovi strumenti popolari nella comunità con il minimo sforzo di installazione. La VM per l'analisi scientifica dei dati può essere configurata rapidamente, pertanto può essere usata anche in altri scenari di utilizzo a breve termine, ad esempio nella replica di esperimenti pubblicati, nell'esecuzione di demo e di procedure dettagliate in sessioni online, nonché in esercitazioni in conferenza.

### <a name="deep-learning"></a>Apprendimento avanzato
La VM di analisi scientifica dei dati può essere usata per il training del modello usando gli algoritmi di apprendimento avanzato sull'hardware basato su GPU (unità di elaborazione grafica). Grazie alle funzioni di scalabilità delle VM del cloud di Azure, la DSVM consente di usare hardware basato su GPU nel cloud in base alle necessità. È possibile passare a una VM basata su GPU durante il training di modelli di grandi dimensioni o quando sono necessari calcoli ad alta velocità, mantenendo lo stesso disco del sistema operativo.  Nell'edizione Windows Server 2016 della DSVM sono preinstallati i driver GPU, i framework e la versione GPU dei framework di apprendimento avanzato. Su Linux, l'apprendimento avanzato su GPU è abilitato sulle DSVM CentOS e Ubuntu. È possibile distribuire l'edizione Ubuntu, CentOS o Windows 2016 della VM data science alla macchina virtuale di Azure non basata su GPU, nel qual caso tutti i framework di apprendimento avanzato eseguiranno il fallback alla modalità CPU. 

## <a name="whats-included-in-the-data-science-vm"></a>Funzionalità incluse nella VM per l'analisi scientifica dei dati
La macchina virtuale per l'analisi scientifica dei dati ha già installati e configurati numerosi strumenti comuni per l'analisi scientifica dei dati e l'apprendimento avanzato. Include inoltre strumenti che semplificano l'uso di vari prodotti di Azure per l'analisi e per i dati. È possibile esplorare e creare modelli predittivi in set di dati su larga scala usando ML Server (R, Python) o SQL Server 2017. Sono inclusi anche una serie di altri strumenti della community open source e di Microsoft, nonché esempi di codice e blocchi appunti. La tabella seguente indica in modo dettagliato e confronta i componenti principali inclusi nelle edizioni per Windows e Linux della macchina virtuale per l'analisi scientifica dei dati.


| **Strumento**                                                           | **Edizione per Windows** | **Edizione per Linux** |
| :------------------------------------------------------------------ |:-------------------:|:------------------:|
| [Microsoft R Open](https://mran.microsoft.com/open/) con i pacchetti più diffusi pre-installati   |S                      | S             |
| [Microsoft ML Server (R, Python)](https://docs.microsoft.com/machine-learning-server/) Developer Edition include <br />  Framework &nbsp;&nbsp;&nbsp;&nbsp;* [RevoScaleR/revoscalepy](https://docs.microsoft.com/machine-learning-server/r/concept-what-is-revoscaler) in parallelo e a prestazioni elevate (R e Python)<br />  &nbsp;&nbsp;&nbsp;&nbsp;* [MicrosoftML](https://docs.microsoft.com/machine-learning-server/r/concept-what-is-the-microsoftml-package): nuovi algoritmi ML all'avanguardia da Microsoft <br />  &nbsp;&nbsp;&nbsp;&nbsp;* [Operazionalizzazione R e Python](https://docs.microsoft.com/machine-learning-server/what-is-operationalization)                                            |S                      | S |
| [Microsoft Office](https://products.office.com/en-us/business/office-365-proplus-business-software) Pro-Plus con attivazione condivisa - Excel, Word e PowerPoint   |S                      |N              |
| [Anaconda Python](https://www.continuum.io/) 2.7, 3.5 con i pacchetti più diffusi pre-installati    |S                      |S              |
| [JuliaPro](https://juliacomputing.com/products/juliapro.html) con i pacchetti più diffusi pre-installati per il linguaggio di programmazione Julia                         |S                      |S              |
| Database relazionali                                                            | [SQL Server 2017](https://www.microsoft.com/sql-server/sql-server-2017) <br/> Developer Edition| [PostgreSQL](https://www.postgresql.org/) (CentOS),<br/>[SQL Server 2017](https://www.microsoft.com/sql-server/sql-server-2017) <br/> Developer Edition (Ubuntu) |
| Strumenti del database                                                       | * SQL Server Management Studio <br/>* SQL Server Integration Services<br/>* [bcp, sqlcmd](https://docs.microsoft.com/sql/tools/command-prompt-utility-reference-database-engine)<br /> * driver di ODBC/JDBC| * [SQuirreL SQL](http://squirrel-sql.sourceforge.net/) (strumento di query), <br /> * bcp, sqlcmd <br /> * driver di ODBC/JDBC|
| Analisi del database scalabile con i servizi ML di SQL Server (R, Python) | S     |N              |
| **[Jupyter Notebook Server](http://jupyter.org/) con i kernel seguenti,**                                  | S     | S |
|     &nbsp;&nbsp;&nbsp;&nbsp;* R | S | S |
|     &nbsp;&nbsp;&nbsp;&nbsp;* Python | S | S |
|     &nbsp;&nbsp;&nbsp;&nbsp;* Julia | S | S |
|     &nbsp;&nbsp;&nbsp;&nbsp;* PySpark | S | S |
|     &nbsp;&nbsp;&nbsp;&nbsp;*   [Sparkmagic](https://github.com/jupyter-incubator/sparkmagic) | N | Y (soltanto Ubuntu) |
|     &nbsp;&nbsp;&nbsp;&nbsp;* SparkR     | N | S |
| JupyterHub (server notebook multiutente)| N | S |
| JupyterLab (server notebook multiutente) | N | Y (soltanto Ubuntu) |
| **Strumenti di sviluppo, editor di codice e IDE**| | |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Visual Studio 2017 (Community Edition)](https://www.visualstudio.com/community/) con plug-in Git, Azure HDInsight (Hadoop), Data Lake, SQL Server Data Tools, [Node.js](https://github.com/Microsoft/nodejstools), [Python](http://aka.ms/ptvs) e [R Tools per Visual Studio (RTVS)](http://microsoft.github.io/RTVS-docs/) | S | N |
| &nbsp;&nbsp;&nbsp;&nbsp;*   [Visual Studio Code](https://code.visualstudio.com/) | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [RStudio Desktop](https://www.rstudio.com/products/rstudio/#Desktop) | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;*   [RStudio Server](https://www.rstudio.com/products/rstudio/#Server) | N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [PyCharm Community Edition](https://www.jetbrains.com/pycharm/) | N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;*   [Atom](https://atom.io/) | N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Juno (Julia IDE)](http://junolab.org/)| S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* Vim ed Emacs | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* Git e GitBash | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* OpenJDK | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* .Net Framework | S | N |
| PowerBI Desktop | S | N |
| SDK per accedere alla suite di servizi di Cortana Intelligence e di Azure | S | S |
| **Strumenti di gestione e spostamento dati** | | |
| &nbsp;&nbsp;&nbsp;&nbsp;* Azure Storage Explorer | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;*   [Interfaccia della riga di comando di Azure](https://docs.microsoft.com/cli/azure) | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* Azure Powershell | S | N |
| &nbsp;&nbsp;&nbsp;&nbsp;*   [Azcopy](https://docs.microsoft.com/azure/storage/storage-use-azcopy) | S | N |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Driver BLOB FUSE](https://github.com/Azure/azure-storage-fuse) | N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Adlcopy(Azure Data Lake Storage)](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-copy-data-azure-storage-blob) | S | N |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Strumento di migrazione dei dati DocDB](https://docs.microsoft.com/azure/documentdb/documentdb-import-data) | S | N |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Gateway di gestione dati di Microsoft](https://msdn.microsoft.com/library/dn879362.aspx): spostare i dati tra posizione locale e cloud | S | N |
| &nbsp;&nbsp;&nbsp;&nbsp;* Utilità della riga di comando Unix/Linux | S | S |
| [Apache Drill](http://drill.apache.org) per l'esplorazione dei dati | S | S |
| **Strumenti di Machine Learning** |||
| &nbsp;&nbsp;&nbsp;&nbsp;*- Integrazione con [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) (R, Python) | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;*   [Xgboost](https://github.com/dmlc/xgboost) | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Vowpal Wabbit](https://github.com/JohnLangford/vowpal_wabbit) | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Weka](http://www.cs.waikato.ac.nz/ml/weka/) | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Rattle](http://rattle.togaware.com/) | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [LightGBM](https://github.com/Microsoft/LightGBM) | N | Y (soltanto Ubuntu) |
| &nbsp;&nbsp;&nbsp;&nbsp;* [H2O](https://www.h2o.ai/h2o/), [Sparkling Water](https://www.h2o.ai/sparkling-water/), [Deep Water](https://www.h2o.ai/deep-water/) | N | Y (soltanto Ubuntu) |
| **Strumenti per Deep Learning** <br>Tutti gli strumenti funzioneranno su GPU o CPU |  |  |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Microsoft Cognitive Toolkit (CNTK)](https://www.microsoft.com/en-us/cognitive-toolkit/) (Windows 2016) | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [TensorFlow](https://www.tensorflow.org/) | S (Windows 2016) | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Horovod](https://github.com/uber/horovod) | N | S (Ubuntu) |
| &nbsp;&nbsp;&nbsp;&nbsp;* [MXNet](http://mxnet.io/) | S (Windows 2016) | S|
| &nbsp;&nbsp;&nbsp;&nbsp;*   [Caffe &amp; Caffe2](https://github.com/caffe2/caffe2) | N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Chainer](https://chainer.org/) | N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Torch](http://torch.ch/) | N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Theano](https://github.com/Theano/Theano) | N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Keras](https://keras.io/)| N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [PyTorch](http://pytorch.org/)| N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [NVidia Digits](https://github.com/NVIDIA/DIGITS) | N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [MXNet Model Server](https://github.com/awslabs/mxnet-model-server) | N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [TensorFlow Serving](https://www.tensorflow.org/serving/) | N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [TensorRT](https://developer.nvidia.com/tensorrt) | N | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [CUDA, cuDNN, NVIDIA Driver](https://developer.nvidia.com/cuda-toolkit) | S | S |
| **Piattaforma Big Data (soltanto Devtest)**|||
| &nbsp;&nbsp;&nbsp;&nbsp;* [Spark](http://spark.apache.org/) locale indipendente | S | S |
| &nbsp;&nbsp;&nbsp;&nbsp;* [Hadoop](http://hadoop.apache.org/) locale (HDFS, YARN) | N | S |

## <a name="get-started"></a>Attività iniziali

### <a name="windows-data-science-vm"></a>Macchina virtuale data science Windows
* Per altre informazioni su come creare una macchina virtuale data science Windows, vedere [Effettuare il provisioning di una macchina virtuale data science Windows](provision-vm.md). Per altre informazioni sull'esecuzione di varie attività necessarie per il progetto di data science sulla macchina virtuale data science Windows, vedere [Dieci cose da fare con la macchina virtuale data science](vm-do-ten-things.md).

### <a name="linux-data-science-vm"></a>Macchina virtuale data science Linux
* Per altre informazioni su come creare una macchina virtuale data science Ubuntu, vedere [Effettuare il provisioning di una macchina virtuale data science per Linux (Ubuntu)](dsvm-ubuntu-intro.md). Per altre informazioni su come creare una macchina virtuale data science CentOS, vedere [Effettuare il provisioning di una macchina virtuale data science CentOS in Azure](linux-dsvm-intro.md).
* Per una procedura dettagliata che illustra come eseguire diverse attività comuni di data science con la macchina virtuale Linux, CentOS e Ubuntu, vedere [Data science in una macchina virtuale data science Linux](linux-dsvm-walkthrough.md).

