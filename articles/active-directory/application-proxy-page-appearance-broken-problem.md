---
title: La página de aplicación no se muestra correctamente para una aplicación de proxy de aplicación | Microsoft Docs
description: Instrucciones para cuando la página no se muestre correctamente en una aplicación de proxy de aplicación que se ha integrado con Azure AD
services: active-directory
documentationcenter: ''
author: ajamess
manager: mtillman
ms.assetid: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/23/2018
ms.author: asteen
ms.openlocfilehash: d187b545a486be28fc80e6baf8e58079ff94ec5e
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/28/2018
---
# <a name="application-page-does-not-display-correctly-for-an-application-proxy-application"></a>La página de aplicación no se muestra correctamente para una aplicación de proxy de aplicación

Este artículo lo ayuda a solucionar problemas con aplicaciones de proxy de aplicación de Azure Active Directory cuando va a la página y hay algo en ella que no parece correcto.

## <a name="overview"></a>Información general
Cuando se publica una aplicación de proxy de aplicación, solo son accesibles las páginas en la raíz al acceder a la aplicación. Si la página no se muestra correctamente, puede que falten algunos recursos de página en la dirección URL interna raíz utilizada para la aplicación. Para resolverlo, asegúrese de que ha publicado *todos* los recursos de la página como parte de la aplicación.

Para comprobar si este es el problema, abra la herramienta de seguimiento de red (como Fiddler o las herramientas de F12 en Internet Explorer o Edge), cargue la página y busque errores 404. Eso indica las páginas que actualmente no se encuentran y que es posible que todavía tengan que publicarse.

Como ejemplo de este caso, suponga que ha publicado una aplicación de gastos mediante la dirección URL interna <http://myapps/expenses>, pero la aplicación utiliza la hoja de estilos <http://myapps/style.css>. En este caso, la hoja de estilos no está publicada en la aplicación, por lo que, cuando se carga, la aplicación de gastos genera un error 404 al intentar cargar style.css. En este ejemplo, el problema se podría resolver publicando la aplicación con la dirección URL interna <http://myapp/> en su lugar.

## <a name="problems-with-publishing-as-one-application"></a>Problemas con la publicación como aplicación

Si no es posible publicar todos los recursos en la misma aplicación, debe publicar varias aplicaciones y habilitar vínculos entre ellas.

Para ello, se recomienda utilizar la solución de [dominios personalizados](https://docs.microsoft.com/azure/active-directory/active-directory-application-proxy-custom-domains). Sin embargo, esta solución requiere que posea el certificado de su dominio y que las aplicaciones usen nombres de dominio completos (FQDN). Para ver otras opciones, consulte la [documentación sobre la solución de problemas de vínculos rotos](application-proxy-page-links-broken-problem.md).

## <a name="next-steps"></a>Pasos siguientes
[Publicación de aplicaciones mediante el proxy de aplicación de Azure AD](application-proxy-publish-azure-portal.md)
