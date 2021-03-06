---
title: Descarga y extracción del Kit de desarrollo de Azure Stack (ASDK) | Microsoft Docs
description: Describe cómo descargar y extraer el Kit de desarrollo de Azure Stack (ASDK).
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/16/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: 0cf389c9443a9cff477b884c277d72de27769afc
ms.sourcegitcommit: a36a1ae91968de3fd68ff2f0c1697effbb210ba8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/17/2018
---
# <a name="download-and-extract-the-azure-stack-development-kit-asdk"></a>Descarga y extracción de Kit de desarrollo de Azure Stack (ASDK)
Después de asegurarse de que el equipo host del kit de desarrollo cumple los requisitos básicos para instalar el Kit de desarrollo de Azure Stack, el siguiente paso es descargar y extraer el paquete de implementación del kit para obtener Cloudbuilder.vhdx.

## <a name="download-the-asdk"></a>Descarga del ASDK
1. Antes de iniciar la descarga, asegúrese de que el equipo cumple con los requisitos previos siguientes:

  - El equipo debe tener al menos 60 GB de espacio en disco disponible en cuatro unidades de disco duro idénticas e independientes además del disco de sistema operativo.
  - [.NET framework 4.6 (o una versión posterior)](https://aka.ms/r6mkiy) debe estar instalado.

2. [Vaya a la página de introducción](https://azure.microsoft.com/overview/azure-stack/try/?v=try) donde puede descargar el Kit de desarrollo de Azure Stack, proporcione sus datos y, a continuación, haga clic en **Enviar**.
3. Descargue y ejecute el [comprobador de implementaciones del Kit de desarrollo de Azure Stack](https://go.microsoft.com/fwlink/?LinkId=828735&clcid=0x409) que es el script de comprobación que constituye un requisito previo. Este script independiente revisa las comprobaciones previas realizadas por la configuración del Kit de desarrollo de Azure Stack. Ofrece una manera de confirmar que se están cumpliendo los requisitos de hardware y software, antes de descargar el paquete más grande del Kit de desarrollo de Azure Stack.
4. En **Descargar el software**, haga clic en **Kit de desarrollo de Azure Stack**.

  > [!NOTE]
  > La descarga de ASDK (AzureStackDevelopmentKit.exe) ocupa aproximadamente 10 GB.

## <a name="extract-the-asdk"></a>Extracción del ASDK
1. Una vez finalizada la descarga, haga clic en **Ejecutar** para iniciar el autoextractor de ASDK (AzureStackDevelopmentKit.exe).
2. Revise y acepte el contrato de licencia que aparece en la página **Contrato de licencia** del asistente del autoextractor y, a continuación, haga clic en **Siguiente**.
3. Revise la información sobre la declaración de privacidad que aparece en la página **Aviso importante** del asistente del autoextractor y, a continuación, haga clic en **Siguiente**.
4. Seleccione la ubicación de los archivos de instalación de Azure Stack que se van a extraer en la página **Seleccionar ubicación de destino** del asistente del autoextractor y, a continuación, haga clic en **Siguiente**. La ubicación predeterminada es *carpeta actual*\Azure Stack Development Kit. 
5. Revise el resumen de información de la ubicación de destino en la página **Ready to Extract** (Listo para extraer) del asistente del autoextractor y, a continuación, haga clic en **Extraer** para extraer los archivos CloudBuilder.vhdx (de unos 25 GB) y ThirdPartyLicenses.rtf. Este proceso tarda un tiempo en completarse.
6. Copie o mueva el archivo CloudBuilder.vhdx a la raíz de la unidad C:\ (C:\CloudBuilder.vhdx) del equipo host de ASDK.

> [!NOTE]
> Después de extraer los archivos, puede eliminar los archivos EXE y .BIN para recuperar espacio en el disco duro. O bien, puede hacer una copia de seguridad de estos archivos, con el fin de que si necesita implementar ASDK, no tenga que volver a descargarlos.


## <a name="next-steps"></a>Pasos siguientes
[Preparación del equipo host del Kit de desarrollo de Azure Stack](asdk-prepare-host.md)