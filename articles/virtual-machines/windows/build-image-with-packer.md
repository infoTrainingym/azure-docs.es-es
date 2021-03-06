---
title: Creación de imágenes de VM de Microsoft Azure con Packer | Microsoft Docs
description: Aprenda a usar Packer para crear imágenes de máquinas virtuales Windows en Azure
services: virtual-machines-windows
documentationcenter: virtual-machines
author: iainfoulds
manager: jeconnoc
editor: tysonn
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 03/29/2018
ms.author: iainfou
ms.openlocfilehash: f174837b8d370ffabdf4148b18d3425d9f3d9f10
ms.sourcegitcommit: 34e0b4a7427f9d2a74164a18c3063c8be967b194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2018
---
# <a name="how-to-use-packer-to-create-windows-virtual-machine-images-in-azure"></a>Uso de Packer para crear imágenes de máquinas virtuales Windows en Azure
Cada máquina virtual (VM) en Azure se crea a partir de una imagen que define la distribución de Windows y la versión del sistema operativo. Las imágenes pueden incluir configuraciones y aplicaciones preinstaladas. Azure Marketplace proporciona muchas imágenes propias y de terceros para los entornos de aplicaciones y sistemas operativos más comunes, pero también puede crear sus propias imágenes personalizadas adaptadas a sus necesidades. En este artículo se detalla cómo utilizar la herramienta de código abierto [Packer](https://www.packer.io/) para definir y crear imágenes personalizadas en Azure.


## <a name="create-azure-resource-group"></a>Creación del grupo de recursos de Azure
Durante el proceso de compilación, Packer crea recursos de Azure temporales mientras genera la máquina virtual de origen. Para capturar dicha máquina virtual para usarla como imagen, debe definir un grupo de recursos. La salida del proceso de compilación de Packer se almacena en este grupo de recursos.

Cree un grupo de recursos con [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```powershell
$rgName = "myResourceGroup"
$location = "East US"
New-AzureRmResourceGroup -Name $rgName -Location $location
```

## <a name="create-azure-credentials"></a>Creación de credenciales de Azure
Packer se autentica con Azure mediante una entidad de servicio. Las entidades de servicio de Azure son identidades de seguridad que pueden usarse con aplicaciones, servicios y herramientas de automatización como Packer. El usuario controla los permisos y los define con respecto a cuáles son las operaciones que la entidad de servicio puede realizar en Azure.

Cree una entidad de servicio con [New-AzureRmADServicePrincipal](/powershell/module/azurerm.resources/new-azurermadserviceprincipal) y asigne permisos para que la entidad de servicio cree y administre recursos con [New-AzureRmRoleAssignment](/powershell/module/azurerm.resources/new-azurermroleassignment):

```powershell
$sp = New-AzureRmADServicePrincipal -DisplayName "Azure Packer" `
    -Password (ConvertTo-SecureString "P@ssw0rd!" -AsPlainText -Force)
Sleep 20
New-AzureRmRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $sp.ApplicationId
```

Para autenticarse en Azure, también tendrá que obtener los identificadores de suscripción e inquilino de Azure con [Get-AzureRmSubscription](/powershell/module/azurerm.profile/get-azurermsubscription):

```powershell
$sub = Get-AzureRmSubscription
$sub.TenantId[0]
$sub.SubscriptionId[0]
```

Estas dos identificadores se usan en el paso siguiente.


## <a name="define-packer-template"></a>Definición de la plantilla de Packer
Para crear imágenes, es preciso crear una plantilla en forma de archivo JSON. En la plantilla, se definen los generadores y aprovisionadores que realizan el proceso de creación real. Packer tiene un [generador para Azure](https://www.packer.io/docs/builders/azure.html) que permite definir los recursos de Azure, como las credenciales de la entidad de servicio creadas en el paso anterior.

Cree un archivo denominado *windows.json* y pegue el siguiente contenido. Escriba sus propios valores para los siguientes elementos:

| .                           | Dónde se obtiene |
|-------------------------------------|----------------------------------------------------|
| *client_id*                         | Vea el identificador de la entidad de servicio con `$sp.applicationId` |
| *client_secret*                     | La contraseña que especificó en `$securePassword` |
| *tenant_id*                         | Salida del comando `$sub.TenantId` |
| *subscription_id*                   | Salida del comando `$sub.SubscriptionId` |
| *object_id*                         | Vea el identificador del objeto de la entidad de servicio con `$sp.Id` |
| *managed_image_resource_group_name* | Nombre del grupo de recursos que creó en el primer paso |
| *managed_image_name*                | Nombre de la imagen de disco administrado que se crea |

```json
{
  "builders": [{
    "type": "azure-arm",

    "client_id": "0831b578-8ab6-40b9-a581-9a880a94aab1",
    "client_secret": "P@ssw0rd!",
    "tenant_id": "72f988bf-86f1-41af-91ab-2d7cd011db47",
    "subscription_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",
    "object_id": "a7dfb070-0d5b-47ac-b9a5-cf214fff0ae2",

    "managed_image_resource_group_name": "myResourceGroup",
    "managed_image_name": "myPackerImage",

    "os_type": "Windows",
    "image_publisher": "MicrosoftWindowsServer",
    "image_offer": "WindowsServer",
    "image_sku": "2016-Datacenter",

    "communicator": "winrm",
    "winrm_use_ssl": "true",
    "winrm_insecure": "true",
    "winrm_timeout": "3m",
    "winrm_username": "packer",

    "azure_tags": {
        "dept": "Engineering",
        "task": "Image deployment"
    },

    "location": "East US",
    "vm_size": "Standard_DS2_v2"
  }],
  "provisioners": [{
    "type": "powershell",
    "inline": [
      "Add-WindowsFeature Web-Server",
      "& $env:SystemRoot\\System32\\Sysprep\\Sysprep.exe /oobe /generalize /quiet /quit",
      "while($true) { $imageState = Get-ItemProperty HKLM:\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Setup\\State | Select ImageState; if($imageState.ImageState -ne 'IMAGE_STATE_GENERALIZE_RESEAL_TO_OOBE') { Write-Output $imageState.ImageState; Start-Sleep -s 10  } else { break } }"
    ]
  }]
}
```

Esta plantilla crea una máquina virtual de Windows Server 2016, instala IIS y generaliza la máquina virtual con Sysprep. La instalación de IIS muestra cómo puede usar el aprovisionador de PowerShell para ejecutar comandos adicionales. La imagen final de Packer incluye así la instalación y configuración del software necesario.


## <a name="build-packer-image"></a>Creación de una imagen de Packer
Si Packer aún no está instalado en el equipo local, de compresor [siga las instrucciones de instalación de Packer](https://www.packer.io/docs/install/index.html).

Para generar la imagen, especifique el archivo de plantilla de Packer como sigue:

```bash
./packer build windows.json
```

A continuación puede ver un ejemplo del resultado de los comandos anteriores:

```bash
azure-arm output will be in this color.

==> azure-arm: Running builder ...
    azure-arm: Creating Azure Resource Manager (ARM) client ...
==> azure-arm: Creating resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> Location          : ‘East US’
==> azure-arm:  -> Tags              :
==> azure-arm:  ->> task : Image deployment
==> azure-arm:  ->> dept : Engineering
==> azure-arm: Validating deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> DeploymentName    : ‘pkrdppq0mthtbtt’
==> azure-arm: Deploying deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> DeploymentName    : ‘pkrdppq0mthtbtt’
==> azure-arm: Getting the certificate’s URL ...
==> azure-arm:  -> Key Vault Name        : ‘pkrkvpq0mthtbtt’
==> azure-arm:  -> Key Vault Secret Name : ‘packerKeyVaultSecret’
==> azure-arm:  -> Certificate URL       : ‘https://pkrkvpq0mthtbtt.vault.azure.net/secrets/packerKeyVaultSecret/8c7bd823e4fa44e1abb747636128adbb'
==> azure-arm: Setting the certificate’s URL ...
==> azure-arm: Validating deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> DeploymentName    : ‘pkrdppq0mthtbtt’
==> azure-arm: Deploying deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> DeploymentName    : ‘pkrdppq0mthtbtt’
==> azure-arm: Getting the VM’s IP address ...
==> azure-arm:  -> ResourceGroupName   : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> PublicIPAddressName : ‘packerPublicIP’
==> azure-arm:  -> NicName             : ‘packerNic’
==> azure-arm:  -> Network Connection  : ‘PublicEndpoint’
==> azure-arm:  -> IP Address          : ‘40.76.55.35’
==> azure-arm: Waiting for WinRM to become available...
==> azure-arm: Connected to WinRM!
==> azure-arm: Provisioning with Powershell...
==> azure-arm: Provisioning with shell script: /var/folders/h1/ymh5bdx15wgdn5hvgj1wc0zh0000gn/T/packer-powershell-provisioner902510110
    azure-arm: #< CLIXML
    azure-arm:
    azure-arm: Success Restart Needed Exit Code      Feature Result
    azure-arm: ------- -------------- ---------      --------------
    azure-arm: True    No             Success        {Common HTTP Features, Default Document, D...
    azure-arm: <Objs Version=“1.1.0.1” xmlns=“http://schemas.microsoft.com/powershell/2004/04"><Obj S=“progress” RefId=“0"><TN RefId=“0”><T>System.Management.Automation.PSCustomObject</T><T>System.Object</T></TN><MS><I64 N=“SourceId”>1</I64><PR N=“Record”><AV>Preparing modules for first use.</AV><AI>0</AI><Nil /><PI>-1</PI><PC>-1</PC><T>Completed</T><SR>-1</SR><SD> </SD></PR></MS></Obj></Objs>
==> azure-arm: Querying the machine’s properties ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> ComputeName       : ‘pkrvmpq0mthtbtt’
==> azure-arm:  -> Managed OS Disk   : ‘/subscriptions/guid/resourceGroups/packer-Resource-Group-pq0mthtbtt/providers/Microsoft.Compute/disks/osdisk’
==> azure-arm: Powering off machine ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> ComputeName       : ‘pkrvmpq0mthtbtt’
==> azure-arm: Capturing image ...
==> azure-arm:  -> Compute ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm:  -> Compute Name              : ‘pkrvmpq0mthtbtt’
==> azure-arm:  -> Compute Location          : ‘East US’
==> azure-arm:  -> Image ResourceGroupName   : ‘myResourceGroup’
==> azure-arm:  -> Image Name                : ‘myPackerImage’
==> azure-arm:  -> Image Location            : ‘eastus’
==> azure-arm: Deleting resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-pq0mthtbtt’
==> azure-arm: Deleting the temporary OS disk ...
==> azure-arm:  -> OS Disk : skipping, managed disk was used...
Build ‘azure-arm’ finished.

==> Builds finished. The artifacts of successful builds are:
--> azure-arm: Azure.ResourceManagement.VMImage:

ManagedImageResourceGroupName: myResourceGroup
ManagedImageName: myPackerImage
ManagedImageLocation: eastus
```

Packer tarda unos minutos en crear la máquina virtual, ejecutar los aprovisionadores y limpiar la implementación.


## <a name="create-a-vm-from-the-packer-image"></a>Creación de una máquina virtual a partir de la imagen de Packer
Ya puede crear una máquina virtual a partir de la imagen con [New-AzureRmVM](/powershell/module/azurerm.compute/new-azurermvm). Si aún no existen, se crean los recursos de red complementarios. Cuando se le solicite, escriba un nombre de usuario administrativo y una contraseña que se crearán en la máquina virtual. En el ejemplo siguiente se crea una máquina virtual denominada *myVM* a partir de *myPackerImage*:

```powershell
New-AzureRmVm `
    -ResourceGroupName $rgName `
    -Name "myVM" `
    -Location $location `
    -VirtualNetworkName "myVnet" `
    -SubnetName "mySubnet" `
    -SecurityGroupName "myNetworkSecurityGroup" `
    -PublicIpAddressName "myPublicIpAddress" `
    -OpenPorts 80 `
    -Image "myPackerImage"
```

La operación de creación de la máquina virtual a partir de la imagen de Packer tarda unos minutos.


## <a name="test-vm-and-webserver"></a>Prueba de la máquina virtual y el servidor web
Obtenga la dirección IP pública de la máquina virtual con [Get-AzureRmPublicIPAddress](/powershell/module/azurerm.network/get-azurermpublicipaddress). En el ejemplo siguiente se obtiene la dirección IP de *myPublicIP* que se ha creado anteriormente:

```powershell
Get-AzureRmPublicIPAddress `
    -ResourceGroupName $rgName `
    -Name "myPublicIPAddress" | select "IpAddress"
```

Para ver en acción la máquina virtual, que incluye la instalación de IIS desde el aprovisionador de Packer, escriba la dirección IP pública en un explorador web.

![Sitio predeterminado de IIS](./media/build-image-with-packer/iis.png) 


## <a name="next-steps"></a>Pasos siguientes
En este ejemplo, ha utilizado Packer para crear una imagen de máquina virtual con IIS instalado. Esta imagen se puede usar junto con los flujos de trabajo de la implementación existentes, como implementar la aplicación en las máquinas virtuales que se crean a partir de la imagen con Team Services, Ansible, Chef o Puppet.

Para ver más plantillas de Packer de ejemplo para otras distribuciones de Windows, consulte [este repositorio de GitHub](https://github.com/hashicorp/packer/tree/master/examples/azure).
