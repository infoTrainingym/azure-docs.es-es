---
title: Log Analytics para Apache Kafka - Azure HDInsight | Microsoft Docs
description: Obtenga información acerca de cómo usar Log Analytics para analizar los registros de clúster de Apache Kafka en Azure HDInsight.
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: jhubbard
editor: cgronlun
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: ''
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 01/30/2018
ms.author: larryfr
ms.openlocfilehash: a373ef5cc71d5ae69c83555dc71525aa2188233e
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2018
---
# <a name="analyze-logs-for-apache-kafka-on-hdinsight"></a>Análisis de registros para Apache Kafka en HDInsight

Obtenga información acerca de cómo usar Log Analytics para analizar los registros generados por Apache Kafka en Azure HDInsight.

## <a name="enable-log-analytics-for-kafka"></a>Habilitación de Log Analytics para Kafka

Los pasos para habilitar Log Analytics para HDInsight son los mismos para todos los clústeres de HDInsight. Use los vínculos siguientes para aprender a crear y configurar los servicios necesarios:

1. Cree un área de trabajo de Log Analytics. Para obtener más información, consulte el documento [Introducción a un área de trabajo de Log Analytics](https://docs.microsoft.com/azure/log-analytics).

2. Cree Kafka en un clúster de HDInsight. Para obtener más información, consulte el documento [Introducción a Apache Kafka (versión preliminar) en HDInsight](apache-kafka-get-started.md).

3. Configure el clúster de Kafka para usar Log Analytics. Para obtener más información, vea el documento [Use Log Analytics to monitor HDInsight](../hdinsight-hadoop-oms-log-analytics-tutorial.md) (Uso de Log Analytics para supervisar HDInsight).

    > [!NOTE]
    > También puede configurar el clúster para usar Log Analytics mediante el cmdlet `Enable-AzureRmHDInsightOperationsManagementSuite`. Este cmdlet requiere la información siguiente:
    >
    > * El nombre del clúster de HDInsight.
    > * El identificador de área de trabajo de Log Analytics. Puede encontrar el identificador de área de trabajo de su área de trabajo de Log Analytics.
    > * La clave principal para la conexión de Log Analytics. Para buscar la clave principal, seleccione la instancia de Log Analytics y, a continuación, __Portal de OMS__. En el portal de OMS, seleccione __Configuración__, __Orígenes conectados__ y, a continuación, __Servidores de Linux__.


> [!IMPORTANT]
> Pueden pasar unos 20 minutos hasta que los datos estén disponibles para Log Analytics.

## <a name="query-logs"></a>Registros de consultas

1. En [Azure Portal](https://portal.azure.com), seleccione el área de trabajo de Log Analytics.

2. Seleccione __Búsqueda de registros__. Aquí podrá buscar los datos recopilados de Kafka. A continuación se presentan algunos ejemplos de búsqueda:

    * Uso de disco: `Type=Perf ObjectName="Logical Disk" (CounterName="Free Megabytes")  InstanceName="_Total" Computer='hn*-*' or Computer='wn*-*' | measure avg(CounterValue) by   Computer interval 1HOUR`
    * Uso de CPU: `Type:Perf CounterName="% Processor Time" InstanceName="_Total" Computer='hn*-*' or Computer='wn*-*' | measure avg(CounterValue) by Computer interval 1HOUR`
    * Mensajes entrantes por segundo: `Type=metrics_kafka_CL ClusterName_s="your_kafka_cluster_name" InstanceName_s="kafka-BrokerTopicMetrics-MessagesInPerSec-Count" | measure avg(kafka_BrokerTopicMetrics_MessagesInPerSec_Count_value_d) by HostName_s interval 1HOUR`
    * Bytes entrantes por segundo: `Type=metrics_kafka_CL HostName_s="wn0-kafka" InstanceName_s="kafka-BrokerTopicMetrics-BytesInPerSec-Count" | measure avg(kafka_BrokerTopicMetrics_BytesInPerSec_Count_value_d) interval 1HOUR`
    * Bytes salientes por segundo: `Type=metrics_kafka_CL ClusterName_s="your_kafka_cluster_name" InstanceName_s="kafka-BrokerTopicMetrics-BytesOutPerSec-Count" |  measure avg(kafka_BrokerTopicMetrics_BytesOutPerSec_Count_value_d) interval 1HOUR`

    > [!IMPORTANT]
    > Reemplace los valores de consulta por la información específica de su clúster. Por ejemplo, `ClusterName_s` debe establecerse en el nombre del clúster. `HostName_s` debe establecerse en el nombre de dominio de un nodo de trabajo en el clúster.

    También puede escribir `*` para buscar todos los tipos registrados. Actualmente están disponibles los registros siguientes para las consultas:

    | Tipo de registro | DESCRIPCIÓN |
    | ---- | ---- |
    | log\_kafkaserver\_CL | Kafka broker server.log |
    | log\_kafkacontroller\_CL | Kafka broker controller.log |
    | metrics\_kafka\_CL | Kafka JMX metrics |

    ![Imagen de la búsqueda de uso de CPU](./media/apache-kafka-log-analytics-operations-management/kafka-cpu-usage.png)
 
 ## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Log Analytics, consulte el documento [Introducción a un área de trabajo de Log Analytics](../../log-analytics/log-analytics-get-started.md).

Para obtener información sobre cómo trabajar con Kafka, consulte los documentos siguientes:

 * [Mirror Kafka between HDInsight clusters](apache-kafka-mirroring.md) (Reflejar Kafka entre clústeres de HDInsight)
 * [Increase the scalability of Kafka on HDInsight](apache-kafka-scalability.md) (Aumentar la escalabilidad de Kafka en HDInsight)
 * [Ejemplo de streaming de Apache Spark (DStream) con Kafka (versión preliminar) en HDInsight](../hdinsight-apache-spark-with-kafka.md)
 * [Uso del flujo estructurado de Spark con Kafka (versión preliminar) en HDInsight](../hdinsight-apache-kafka-spark-structured-streaming.md)
