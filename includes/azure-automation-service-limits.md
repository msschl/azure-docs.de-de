---
title: Includedatei
description: Includedatei
services: automation
author: georgewallace
ms.service: automation
ms.topic: include
ms.date: 12/13/2018
ms.author: gwallace
ms.custom: include file
ms.openlocfilehash: b701183fa270b1aad9a622a21c5ce0795fc8057f
ms.sourcegitcommit: bf509e05e4b1dc5553b4483dfcc2221055fa80f2
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/22/2019
ms.locfileid: "60012500"
---
#### <a name="process-automation"></a>Prozessautomatisierung

| Ressource | Maximales Limit |Notizen|
| --- | --- |---|
| Maximale Anzahl von neuen Aufträgen, die alle 30 Sekunden pro Azure Automation-Konto (nicht geplante Aufträge) übermittelt werden können |100 |Wenn dieser Grenzwert erreicht wird, treten bei nachfolgenden Anforderungen zum Erstellen eines Auftrags Fehler auf. Der Client empfängt eine Fehlerantwort.|
| Maximale Anzahl gleichzeitig ausgeführter Aufträge pro Automation-Konto (nicht geplante Aufträge) |200 |Wenn dieser Grenzwert erreicht wird, treten bei nachfolgenden Anforderungen zum Erstellen eines Auftrags Fehler auf. Der Client empfängt eine Fehlerantwort.|
| Maximale Speichergröße der Auftragsmetadaten für einen fortlaufenden Zeitraum von 30 Tagen | 10 GB (etwa 4 Millionen Aufträge)|Wenn dieser Grenzwert erreicht wird, treten bei nachfolgenden Anforderungen zum Erstellen eines Auftrags Fehler auf. |
| Maximale Größe von Auftragsdatenströmen|1 MB|Ein einzelner Datenstrom darf nicht größer als 1 MB sein.|
| Maximale Anzahl von Modulen, die alle 30 Sekunden pro Automation-Konto importiert werden können |5 ||
| Maximale Größe eines Moduls |100 MB ||
| Auftragsausführungszeit – Free-Tarif |500 Minuten pro Abonnement pro Kalendermonat ||
| Maximal zulässiger Speicherplatz pro Sandbox<sup>1</sup> |1 GB |Betrifft nur Azure-Sandboxen.|
| Maximaler Arbeitsspeicher pro Sandbox<sup>1</sup> |400 MB |Betrifft nur Azure-Sandboxen.|
| Maximal zulässige Anzahl von Netzwerksockets pro Sandbox<sup>1</sup> |1.000 |Betrifft nur Azure-Sandboxen.|
| Maximal zulässige Laufzeit pro Runbook <sup>1</sup> |3 Stunden |Betrifft nur Azure-Sandboxen.|
| Maximale Anzahl von Automation-Konten in einem Abonnement |Keine Begrenzung ||
|Maximale Anzahl gleichzeitiger Aufträge, die auf einem einzelnen Hybrid Runbook Worker ausgeführt werden können|50 ||
| Maximale Größe der Runbookauftragsparameter   | 512 KBit||
| Maximale Anzahl von Runbookparametern   | 50|Sie können einen JSON- oder XML-Zeichenfolge an einen Parameter übergeben und ihn mit dem Runbook analysieren, wenn Sie das Limit von 50 Parametern erreichen.|
| Maximale Größe für Webhooknutzlast |  512 KBit|
| Maximale Anzahl von Tagen, die Auftragsdaten aufbewahrt werden|30 Tage|
| Maximale Größe des PowerShell-Workflowstatus |5 MB| Gilt für PowerShell-Workflow-Runbooks, wenn Prüfpunkte für Workflows eingerichtet werden.|

<sup>1</sup> Eine Sandbox ist eine freigegebene Umgebung, die von mehreren Aufträgen verwendet werden kann. Aufträge, die die gleiche Sandbox verwenden, sind an die Ressourceneinschränkungen der Sandbox gebunden.

#### <a name="change-tracking-and-inventory"></a>Änderungsnachverfolgung und Bestand

Die folgende Tabelle zeigt die Grenzwerte der nachverfolgten Elemente pro Computer für die Änderungsnachverfolgung.

| **Ressource** | **Begrenzung**| **Hinweise** |
|---|---|---|
|Datei|500||
|Registrierung|250||
|Windows-Software|250|Softwareupdates nicht inbegriffen|
|Linux-Pakete|1.250||
|Dienste|250||
|Daemon|250||
