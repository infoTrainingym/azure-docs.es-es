---
title: Introducción a Azure Data Lake Analytics mediante Azure PowerShell | Microsoft Docs
description: 'Use Azure Portal para crear una cuenta de Data Lake Analytics, crear un trabajo de Data Lake Analytics mediante U-SQL y enviar el trabajo. '
services: data-lake-analytics
documentationcenter: ''
author: saveenr
manager: saveenr
editor: cgronlun
ms.assetid: 8a4e901e-9656-4a60-90d0-d78ff2f00656
ms.service: data-lake-analytics
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 05/04/2017
ms.author: saveenr
ms.openlocfilehash: f37a4563a758d442760f4a6be3c11bb9a9ddfc28
ms.sourcegitcommit: 8aab1aab0135fad24987a311b42a1c25a839e9f3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/16/2018
---
# <a name="get-started-with-azure-data-lake-analytics-using-azure-powershell"></a>Introducción a Azure Data Lake Analytics mediante Azure PowerShell
[!INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]

Aprenda a usar Azure PowerShell para crear cuentas de Azure Data Lake Analytics y luego enviar y ejecutar trabajos de U-SQL. Para obtener más información acerca de Análisis de Data Lake, consulte [Información general sobre Análisis de Azure Data Lake](data-lake-analytics-overview.md).

## <a name="prerequisites"></a>requisitos previos

Antes de empezar este tutorial, debe tener la siguiente información:

* **Una cuenta de Análisis de Azure Data Lake**. Consulte [Introducción a Data Lake Analytics](https://docs.microsoft.com/azure/data-lake-analytics/data-lake-analytics-get-started-portal).
* **Una estación de trabajo con Azure PowerShell**. Consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/overview).

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

En este tutorial se asume que sabe usar Azure PowerShell. En concreto, debe saber iniciar sesión en Azure. Si necesita ayuda, consulte [Get started with Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps) (Introducción a Azure PowerShell).

Para iniciar sesión con un nombre de suscripción:

```
Login-AzureRmAccount -SubscriptionName "ContosoSubscription"
```

En lugar del nombre de la suscripción, también puede utilizar un identificador de suscripción para iniciar sesión:

```
Login-AzureRmAccount -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

Si se realiza correctamente, la salida de este comando es similar a la siguiente:

```
Environment           : AzureCloud
Account               : joe@contoso.com
TenantId              : "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
SubscriptionId        : "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
SubscriptionName      : ContosoSubscription
CurrentStorageAccount :
```

## <a name="preparing-for-the-tutorial"></a>Preparativos para el tutorial

Los fragmentos de código de PowerShell de este tutorial usan estas variables para almacenar esta información:

```
$rg = "<ResourceGroupName>"
$adls = "<DataLakeStoreAccountName>"
$adla = "<DataLakeAnalyticsAccountName>"
$location = "East US 2"
```

## <a name="get-information-about-a-data-lake-analytics-account"></a>Información acerca de una cuenta de Data Lake Analytics

```
Get-AdlAnalyticsAccount -ResourceGroupName $rg -Name $adla  
```

## <a name="submit-a-u-sql-job"></a>Envío de un trabajo de U-SQL

Cree una variable de PowerShell para almacenar el script U-SQL.

```
$script = @"
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();

"@
```

Envíe el script.

```
$job = Submit-AdlJob -Account $adla -Name "My Job" –Script $script
```

Como alternativa, puede guardar el script como archivo y enviarlo con el siguiente comando:

```
$filename = "d:\test.usql"
$script | out-File $filename
$job = Submit-AdlJob -Account $adla -Name "My Job" –ScriptPath $filename
```


Obtenga el estado de un trabajo específico. Diga usando este cmdlet hasta que vea que el trabajo se ha realizado.

```
$job = Get-AdlJob -Account $adla -JobId $job.JobId
```

En lugar de llamar a Get-AdlAnalyticsJob una y otra vez hasta que finaliza un trabajo, puede usar el cmdlet Wait-AdlJob.

```
Wait-AdlJob -Account $adla -JobId $job.JobId
```

Descargue el archivo de salida.

```
Export-AdlStoreItem -Account $adls -Path "/data.csv" -Destination "C:\data.csv"
```

## <a name="see-also"></a>Otras referencias
* Para ver el mismo tutorial con otras herramientas, haga clic en los selectores de pestañas en la parte superior de la página.
* Para obtener más información sobre U-SQL, consulte [Introducción al lenguaje U-SQL de Análisis de Azure Data Lake](data-lake-analytics-u-sql-get-started.md).
* Para conocer las tareas de administración, consulte [Administración de Azure Data Lake Analytics mediante el Azure Portal](data-lake-analytics-manage-use-portal.md).
