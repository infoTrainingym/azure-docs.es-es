---
title: Solución de problemas con receptores de Event Hubs en Azure Stream Analytics
description: Prácticas recomendadas de consulta para la consideración de grupos de consumidores de Event Hubs en trabajos de Stream Analytics.
services: stream-analytics
author: jseb225
ms.author: jeanb
manager: kfile
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 04/20/2017
ms.openlocfilehash: 20614986fc6c6afa9a92d163bf973a148e0517c0
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
---
# <a name="debug-azure-stream-analytics-with-event-hub-receivers"></a>Depuración de Azure Stream Analytics con receptores de Event Hubs

Puede usar Azure Event Hubs en Azure Stream Analytics para ingerir o extraer datos de un trabajo. Una práctica recomendada para usar Event Hubs es usar varios grupos de consumidor para garantizar la escalabilidad del trabajo. Una razón es que el número de lectores en el trabajo de Stream Analytics para una entrada específica afecta a la cantidad de lectores en un grupo de consumidores único. El número exacto de destinatarios se basa en los detalles de implementación interna para la lógica de la topología de escalado horizontal. El número de destinatarios no se expone externamente. El número de lectores puede cambiar a la hora de inicio del trabajo o durante las actualizaciones del trabajo.

> [!NOTE]
> Cuando el número de lectores cambia durante una actualización de trabajo, se escriben advertencias transitorias en registros de auditoría. Los trabajos de Stream Analytics se recuperan automáticamente de estos problemas transitorios.

## <a name="number-of-readers-per-partition-exceeds-event-hubs-limit-of-five"></a>El número de lectores por partición supera el límite de Event Hubs de cinco

Los escenarios en los que el número de lectores por partición supera el límite de Event Hubs de cinco incluyen los siguientes:

* Varias instrucciones SELECT: si usa varias instrucciones SELECT que hacen referencia a la **misma** entrada de centro de eventos, cada instrucción SELECT provoca que se cree un nuevo destinatario.
* UNION: cuando se utiliza UNION, es posible tener varias entradas que hacen referencia al **mismo** centro de eventos y grupo de consumidores.
* SELF JOIN: cuando se usa una operación SELF JOIN, es posible hacer referencia al **mismo** centro de eventos varias veces.

## <a name="solution"></a>Solución

Las siguientes prácticas recomendadas pueden ayudar a reducir los escenarios en los que el número de lectores por partición supera el límite de Event Hubs de cinco.

### <a name="split-your-query-into-multiple-steps-by-using-a-with-clause"></a>División de la consulta en varios pasos mediante una cláusula WITH

La cláusula WITH especifica un conjunto de resultados con nombre temporal al que se puede hacer referencia mediante una cláusula FROM en la consulta. Defina la cláusula WITH en el ámbito de ejecución de una única instrucción SELECT.

Por ejemplo, en lugar de esta consulta:

```
SELECT foo 
INTO output1
FROM inputEventHub

SELECT bar
INTO output2
FROM inputEventHub 
…
```

Utilice esta consulta:

```
WITH data AS (
   SELECT * FROM inputEventHub
)

SELECT foo
INTO output1
FROM data

SELECT bar
INTO output2
FROM data
…
```

### <a name="ensure-that-inputs-bind-to-different-consumer-groups"></a>Asegúrese de que las entradas se enlazan a grupos de consumidores diferentes

Para las consultas en las que hay conectadas tres o más entradas al mismo grupo de consumidores de Event Hubs, cree grupos de consumidores independientes. Esto requiere la creación de entradas de Stream Analytics adicionales.


## <a name="get-help"></a>Obtención de ayuda
Para obtener más ayuda, pruebe nuestro [foro de Azure Stream Analytics](https://social.msdn.microsoft.com/Forums/azure/home?forum=AzureStreamAnalytics).

## <a name="next-steps"></a>Pasos siguientes
* [¿Qué es Stream Analytics?](stream-analytics-introduction.md)
* [Introducción a Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Escalado de trabajos de Stream Analytics](stream-analytics-scale-jobs.md)
* [Referencia de lenguaje de consulta de Stream Analytics](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Referencia de API de REST de administración de Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)
