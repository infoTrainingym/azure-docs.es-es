---
title: Configuración de Key Vault para máquinas virtuales Linux con la CLI de Azure 1.0 | Microsoft Docs
description: Configuración de Key Vault para usarlo con una máquina virtual de Azure Resource Manager con la CLI 1.0 de Azure.
services: virtual-machines-linux
documentationcenter: ''
author: singhkays
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: bccdd5ab-5ccf-4760-9039-92c6eafb15bd
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 02/24/2017
ms.author: singhkay
ms.openlocfilehash: a9225429e878415334b0c8a66777902395606d63
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
# <a name="set-up-key-vault-for-virtual-machines-in-azure-resource-manager-with-the-azure-cli-10"></a>Configuración de Key Vault para máquinas virtuales en Azure Resource Manager con la CLI de Azure 1.0
En la pila de Azure Resource Manager, los certificados o secretos se modelan como recursos que se proporcionan mediante el proveedor de recursos de Key Vault. Para más información sobre Azure Key Vault, consulte [¿Qué es Azure Key Vault?](../../key-vault/key-vault-whatis.md) Para poder usar Key Vault con máquinas virtuales de Azure Resource Manager, la propiedad *EnabledForDeployment* de Key Vault se debe establecer en true. Puede hacer esto en varios clientes. En este artículo se muestra cómo configurar Key Vault para su uso con Azure Virtual Machines.

## <a name="cli-versions-to-complete-the-task"></a>Versiones de la CLI para completar la tarea
Puede completar la tarea mediante una de las siguientes versiones de la CLI

- [CLI de Azure 1.0](#quick-commands): la CLI para los modelos de implementación clásico y de Resource Manager (este artículo)
- [CLI de Azure 2.0 ](../windows/key-vault-setup.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json): CLI de última generación para el modelo de implementación de administración de recursos

## <a name="use-cli-10-to-set-up-key-vault"></a>Uso de la CLI 1.0 para configurar Key Vault
Para crear un almacén de claves mediante la interfaz de la línea de comandos (CLI), consulte [Administración de Key Vault mediante la CLI](../../key-vault/key-vault-manage-with-cli2.md#create-a-key-vault).

Para la CLI 1.0, primero debe crear el almacén de claves y luego asignar la directiva de implementación. A continuación, puede asignar la directiva mediante el comando siguiente:

    azure keyvault set-policy ContosoKeyVault –enabled-for-deployment true

## <a name="use-templates-to-set-up-key-vault"></a>Uso de plantillas para configurar Key Vault
Al utilizar plantillas, debe configurar la propiedad `enabledForDeployment` como `true` para el recurso de Key Vault.

    {
      "type": "Microsoft.KeyVault/vaults",
      "name": "ContosoKeyVault",
      "apiVersion": "2015-06-01",
      "location": "<location-of-key-vault>",
      "properties": {
        "enabledForDeployment": "true",
        ....
        ....
      }
    }

Para otras opciones que puede configurar al crear un almacén de claves mediante plantillas, consulte [Create a key vault](https://azure.microsoft.com/documentation/templates/101-key-vault-create/)(Creación de un almacén de claves).
