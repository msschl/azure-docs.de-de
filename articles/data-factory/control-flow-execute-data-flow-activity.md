---
title: Aktivität „Datenfluss ausführen“ in Azure Data Factory | Microsoft-Dokumentation
description: 'Gewusst wie: Ausführen von Datenflüssen aus einer Data Factory-Pipeline heraus.'
services: data-factory
documentationcenter: ''
author: kromerm
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 02/22/2019
ms.author: makromer
ms.openlocfilehash: e1d4ce355f34014d5099c4b46f4420d032363fce
ms.sourcegitcommit: 2ce4f275bc45ef1fb061932634ac0cf04183f181
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/07/2019
ms.locfileid: "65236672"
---
# <a name="execute-data-flow-activity-in-azure-data-factory"></a>Aktivität „Datenfluss ausführen“ in Azure Data Factory
Verwenden Sie die Aktivität „Datenfluss ausführen“, um Ihren ADF-Datenfluss in (der Sandbox) in Läufen zum Debuggen der Pipeline oder in von der Pipeline ausgelösten Läufen auszuführen.

[!INCLUDE [notes](../../includes/data-factory-data-flow-preview.md)]

## <a name="syntax"></a>Syntax

```json
{
    "name": "MyDataFlowActivity",
    "type": "ExecuteDataFlow",
    "typeProperties": {
      "dataflow": {
         "referenceName": "dataflow1",
         "type": "DataFlowReference"
      },
        "compute": {
          "computeType": "General",
          "coreCount": 8,
      }
}

```

## <a name="type-properties"></a>Typeigenschaften

* ```dataflow``` ist der Name der Datenflussentität, die Sie ausführen möchten.
* ```compute``` beschreibt die Spark-Ausführungsumgebung.
* ```coreCount``` ist die Anzahl der Kerne, die dieser Aktivität zur Ausführung Ihres Datenflusses zugewiesen werden sollen.

![Datenfluss ausführen](media/data-flow/activity-data-flow.png "Datenfluss ausführen")

### <a name="run-on"></a>Run on (Ausführen auf)

Wählen Sie die Compute-Umgebung für die Ausführung Ihres Datenflusses. Standardeinstellung ist die standardmäßige Azure Integration Runtime mit automatischer Auflösung. Bei dieser Wahl wird der Datenfluss in der Spark-Umgebung in der Region Ihrer Data Factory ausgeführt. Der Computetyp ist ein Auftragscluster, d.h. das Starten der Compute-Umgebung benötigt mehrere Minuten.

### <a name="debugging-pipelines-with-data-flows"></a>Debuggen von Pipelines mit Datenflüssen

![Schaltfläche „Debuggen“](media/data-flow/debugbutton.png "Schaltfläche „Debuggen“")

Verwenden Sie das Debuggen des Datenflusses, um einen aufgewärmten Cluster zum interaktiven Testen Ihrer Datenflüsse in einem Lauf zum Debuggen der Pipeline zu nutzen. Verwenden Sie die Option zum Debuggen der Pipeline, um Ihre Datenflüsse innerhalb einer Pipeline zu testen.

### <a name="run-on"></a>Run on (Ausführen auf)

Dies ist ein Pflichtfeld, das definiert, welche Integration Runtime, die für die Ausführung Ihrer Datenflussaktivität verwendet werden soll. Standardmäßig verwendet Data Factory die standardmäßige Azure Integration Runtime mit automatischer Auflösung. Sie können aber auch Ihre eigene Azure Integration Runtime erstellen, die bestimmte Regionen, den Computetyp, die Kernanzahl und die Gültigkeitsdauer (TTL) für die Ausführung Ihrer Datenflussausführung definiert.

Die Standardeinstellung für die Ausführung von Datenflüssen sind 8 Kerne von Compute allgemein mit einer Gültigkeitsdauer (TTL) von 60 Minuten.

### <a name="staging-area"></a>Stagingbereich

Wenn Sie Azure Data Warehouse als Senke verwenden, müssen Sie einen Stagingspeicherort für Ihre Polybase-Batchlast wählen.

## <a name="parameterized-datasets"></a>Parametrisierte Datasets

Wenn Sie parametrisierte Datasets verwenden, müssen Sie die Parameterwerte festlegen.

![Parameter für „Datenfluss ausführen“](media/data-flow/params.png "Parameter")

### <a name="debugging-parameterized-data-flows"></a>Debuggen parametrisierter Datenflüsse

Sie können Datenflüsse mit parametrisierten Datasets im Lauf zum Debuggen der Pipeline nur unter Verwendung der Aktivität „Datenfluss ausführen“ debuggen. Derzeit funktionieren interaktive Debugsitzungen im ADF-Datenfluss mit parametrisierten Datasets nicht. Ausführungen der Pipeline und Debugläufe funktionieren mit Parametern.

Eine bewährte Methode ist es, Ihren Datenfluss mit einem statischen Dataset zu erstellen, damit Sie zur Entwurfszeit über die vollständige Verteilung der Metadatenspalten verfügen. Ersetzen Sie dann das statische Dataset durch ein dynamisches, parametrisiertes Dataset, wenn Sie Ihre Datenfluss-Pipeline operationalisieren.

## <a name="next-steps"></a>Nächste Schritte
Weitere Informationen finden Sie unter anderen Ablaufsteuerungsaktivitäten, die von Data Factory unterstützt werden: 

- [Aktivität „If Condition“](control-flow-if-condition-activity.md)
- [Aktivität „Pipeline ausführen“](control-flow-execute-pipeline-activity.md)
- [ForEach-Aktivität](control-flow-for-each-activity.md)
- [Aktivität „Metadaten abrufen“](control-flow-get-metadata-activity.md)
- [Lookup-Aktivität](control-flow-lookup-activity.md)
- [Webaktivität](control-flow-web-activity.md)
- [Until-Aktivität](control-flow-until-activity.md)
