---
title: Quitar Hybrid Runbook Workers de Azure Automation
description: En este artículo encontrará información sobre cómo administrar los Hybrid Runbook Workers de Azure Automation implementados que le permiten ejecutar runbooks en equipos en su centro de datos local o entorno de nube.
services: automation
ms.service: automation
author: georgewallace
ms.author: gwallace
ms.date: 03/19/2018
ms.topic: article
manager: carmonm
ms.openlocfilehash: 0b8f951bbd9712e3d20c3286ef3571e287499c68
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/23/2018
---
# <a name="remove-azure-automation-hybrid-runbook-workers"></a>Quitar Hybrid Runbook Workers de Azure Automation

La característica Hybrid Runbook Worker de Azure Automation permite ejecutar runbooks directamente en el equipo que hospeda el rol y en los recursos del entorno para administrar dichos recursos locales. Este artículo le guía por los pasos para quitar una instancia de Hybrid Worker de un equipo local.

## <a name="removing-hybrid-runbook-worker"></a>Eliminación de Hybrid Runbook Worker

Puede quitar uno o más Hybrid Runbook Worker de un grupo o puede quitar el grupo, dependiendo de sus requisitos.  Para quitar un Hybrid Runbook Worker de un equipo local, realice los pasos siguientes.

1. En Azure Portal, vaya a su cuenta de Automation.  
2. En la hoja **Configuración**, seleccione **Claves** y anote los valores de **URL** y **Primary Access Key** (Clave de acceso principal).  Esta información la necesita para el siguiente paso.
3. Abra una sesión de PowerShell en modo Administrador y ejecute el comando siguiente: `Remove-HybridRunbookWorker -url <URL> -key <PrimaryAccessKey>`.  Use el modificador **-Verbose** para ver un registro detallado del proceso de eliminación.

> [!NOTE]
> Con esto no se quita Microsoft Monitoring Agent del equipo, sino solo la funcionalidad y la configuración del rol Hybrid Runbook Worker.  

## <a name="remove-hybrid-worker-groups"></a>Eliminación de grupos de Hybrid Worker

Para quitar un grupo, primero debe quitar el Hybrid Runbook Worker de todos los equipos que sean miembros del grupo mediante el procedimiento mostrado anteriormente y luego realizar los pasos siguientes para quitar el grupo.  

1. Abra la cuenta de Automation en Azure Portal.
1. En **Automatización de procesos**, seleccione **Grupos de Hybrid Worker**. Seleccione el grupo que quiere eliminar.  Tras seleccionar el grupo específico, se muestra la hoja de propiedades de **Grupo de Hybrid Worker**.<br> ![Hoja Grupo de Hybrid Runbook Worker](media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-group-properties.png)   
2. En la hoja de propiedades del grupo seleccionado, haga clic en **Eliminar**.  Aparecerá un mensaje solicitándole que confirme esta acción. Si está seguro de que quiere continuar, seleccione **Sí**.<br> ![Cuadro de diálogo de confirmación de eliminación de grupo](media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-confirm-delete.png)<br> Este proceso puede tardar varios segundos en completarse y puede realizar el seguimiento de su progreso en **Notificaciones** en el menú. 

## <a name="next-steps"></a>Pasos siguientes

* Revise la [ejecución de runbooks en Hybrid Runbook Worker](automation-hrw-run-runbooks.md) para más información sobre cómo configurar los runbooks para automatizar los procesos del centro de datos local o en otro entorno de nube.
* Para más información sobre cómo instalar Hybrid Runbook Workers de Windows, vea [Hybrid Runbook Worker de Azure Automation en Windows](automation-windows-hrw-install.md).
* Para más información sobre cómo instalar Hybrid Runbook Workers de Linux, vea [Hybrid Runbook Worker de Azure Automation en Linux](automation-linux-hrw-install.md).