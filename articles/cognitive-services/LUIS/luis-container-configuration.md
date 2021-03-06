---
title: Docker-Containereinstellungen
titleSuffix: Language Understanding - Azure Cognitive Services
description: Die Runtimeumgebung für LUIS-Container wird über die Argumente des Befehls `docker run` konfiguriert. LUIS verfügt über mehrere erforderliche Einstellungen sowie einige optionale Einstellungen.
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 04/16/2019
ms.author: diberry
ms.openlocfilehash: e05998f74223ead6bb4e94b86469e51791e0263f
ms.sourcegitcommit: c3d1aa5a1d922c172654b50a6a5c8b2a6c71aa91
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/17/2019
ms.locfileid: "59678563"
---
# <a name="configure-language-understanding-docker-containers"></a>Konfigurieren von Docker-Containern für Language Understanding 

Die Laufzeitumgebung für LUIS-Container (**Language Understanding**) wird über die Argumente des Befehls `docker run` konfiguriert. LUIS verfügt über mehrere erforderliche Einstellungen sowie einige optionale Einstellungen. Es sind noch viele [Beispiele](#example-docker-run-commands) für den Befehl verfügbar. Die containerspezifischen Einstellungen sind die für [Eingabebereitstellung](#mount-settings) und Abrechnung. 

## <a name="configuration-settings"></a>Konfigurationseinstellungen

Dieser Container hat die folgenden Konfigurationseinstellungen:

|Erforderlich|Einstellung|Zweck|
|--|--|--|
|Ja|[ApiKey](#apikey-setting)|Wird zum Nachverfolgen von Abrechnungsinformationen verwendet.|
|Nein |[ApplicationInsights](#applicationinsights-setting)|Ermöglicht das Hinzufügen von Unterstützung für [Azure Application Insights](https://docs.microsoft.com/azure/application-insights)-Telemetriedaten in Ihrem Container.|
|Ja|[Abrechnung](#billing-setting)|Gibt den Endpunkt-URI der Dienstressource in Azure an.|
|Ja|[Eula](#eula-setting)| Gibt an, dass Sie die Lizenz für den Container akzeptiert haben.|
|Nein |[Fluentd](#fluentd-settings)|Schreibt Protokoll- und optional auch Metrikdaten auf einen Fluentd-Server.|
|Nein |[HTTP-Proxy](#http-proxy-credentials-settings)|Konfigurieren Sie einen HTTP-Proxy für ausgehende Anforderungen.|
|Nein |[Protokollierung](#logging-settings)|Bietet Unterstützung für die ASP.NET Core-Protokollierung für Ihren Container. |
|Ja|[Mounts](#mount-settings)|Liest und schreibt Daten vom Hostcomputer in den Container und umgekehrt.|

> [!IMPORTANT]
> Die Einstellungen [`ApiKey`](#apikey-setting), [`Billing`](#billing-setting) und [`Eula`](#eula-setting) werden gemeinsam verwendet, und Sie müssen gültige Werte für alle drei angeben, da der Container andernfalls nicht startet. Weitere Informationen zum Instanziieren eines Containers mithilfe dieser Konfigurationseinstellungen finden Sie unter [Abrechnung](luis-container-howto.md#billing).

## <a name="apikey-setting"></a>ApiKey-Einstellung

Die `ApiKey`-Einstellung gibt den Schlüssel der Azure-Ressourcen an, mit dem die Abrechnungsinformationen für den Container verfolgt werden. Sie müssen einen Wert für ApiKey angeben. Bei diesem Wert muss es sich um einen gültigen Schlüssel für die Ressource vom Typ _Cognitive Services_ handeln, die für die Konfigurationseinstellung [`Billing`](#billing-setting) angegeben wurde.

Diese Einstellung finden Sie hier:

* Azure-Portal: Ressourcenverwaltung von **Cognitive Services** (unter **Schlüssel**)
* LUIS-Portal: Seite mit den Einstellungen für **Schlüssel und Endpunkt**. 

Verwenden Sie nicht den Starter- oder Erstellungsschlüssel. 

## <a name="applicationinsights-setting"></a>ApplicationInsights-Einstellung

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-setting"></a>Billing-Einstellung

Die `Billing`-Einstellung gibt den Endpunkt-URI der _Cognitive Services_-Ressource in Azure an, der zum Messen der Abrechnungsinformationen für den Container verwendet wird. Sie müssen einen Wert für diese Konfigurationseinstellung angeben, und bei dem Wert muss es sich um einen gültigen URI-Endpunkt für eine _Cognitive Services_-Ressource in Azure handeln. Der Container meldet die Nutzung etwa alle 10 bis 15 Minuten.

Diese Einstellung finden Sie hier:

* Azure-Portal: Übersicht über **Cognitive Services**, mit der Bezeichnung `Endpoint`
* LUIS-Portal: Einstellungsseite für **Schlüssel und Endpunkt** als Teil des Endpunkt-URI.

Denken Sie daran, die `luis/v2.0`-Weiterleitung wie in der folgenden Tabelle dargestellt in die URL einzubeziehen:


|Erforderlich| NAME | Datentyp | BESCHREIBUNG |
|--|------|-----------|-------------|
|Ja| `Billing` | Zeichenfolge | URI des Abrechnungsendpunkts<br><br>Beispiel:<br>`Billing=https://westus.api.cognitive.microsoft.com/luis/v2.0` |

## <a name="eula-setting"></a>Eula-Einstellung

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Fluentd-Einstellungen


[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="http-proxy-credentials-settings"></a>Anmeldeinformationseinstellungen für HTTP-Proxy

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## <a name="logging-settings"></a>Logging-Einstellungen
 
[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]


## <a name="mount-settings"></a>Einbindungseinstellungen

Verwenden Sie Bindungsbereitstellungen zum Lesen und Schreiben von Daten im Container. Sie können eine Eingabe- oder Ausgabebereitstellung über die Option `--mount` im Befehl [docker run](https://docs.docker.com/engine/reference/commandline/run/) angeben. 

Der LUIS-Container verwendet die Eingabe- oder Ausgabebereitstellungen nicht zum Speichern von Trainings- oder Dienstdaten. 

Die genaue Syntax für den Bereitstellungspunkt auf dem Host variiert je nach Betriebssystem des Hosts. Darüber hinaus ist es eventuell nicht möglich, auf den Bereitstellungspunkt auf dem [Hostcomputer](luis-container-howto.md#the-host-computer) zuzugreifen, wenn ein Konflikt zwischen den vom Docker-Dienstkonto verwendeten Berechtigungen und den für den Bereitstellungspunkt auf dem Host verwendeten Berechtigungen besteht. 

In der folgenden Tabelle werden die unterstützten Einstellungen beschrieben.

|Erforderlich| NAME | Datentyp | BESCHREIBUNG |
|-------|------|-----------|-------------|
|Ja| `Input` | Zeichenfolge | Das Ziel der Eingabeeinbindung. Standardwert: `/input`. Dies ist der Speicherort der LUIS-Paketdateien. <br><br>Beispiel:<br>`--mount type=bind,src=c:\input,target=/input`|
|Nein | `Output` | Zeichenfolge | Das Ziel der Ausgabeeinbindung. Standardwert: `/output`. Dies ist der Speicherort der Protokolle. Dazu gehören auch LUIS-Abfrageprotokolle und -Containerprotokolle. <br><br>Beispiel:<br>`--mount type=bind,src=c:\output,target=/output`|

## <a name="example-docker-run-commands"></a>Beispiele für den Befehl „docker run“

Die folgenden Beispiele verwenden die Konfigurationseinstellungen, um zu veranschaulichen, wie `docker run`-Befehle geschrieben und verwendet werden.  Nach dem Ausführen wird der Container so lange ausgeführt, bis Sie ihn [beenden](luis-container-howto.md#stop-the-container).


* **Zeilenfortsetzungszeichen:** In den Docker-Befehlen in den folgenden Abschnitten wird der umgekehrte Schrägstrich (`\`) als Zeilenfortsetzungszeichen verwendet. Ersetzen oder entfernen Sie diesen je nach den Anforderungen des Hostbetriebssystems. 
* **Argumentreihenfolge:** Ändern Sie die Reihenfolge der Argumente nur, wenn Sie mit Docker-Containern sehr gut vertraut sind.

Denken Sie daran, die `luis/v2.0`-Weiterleitung wie in der folgenden Tabelle dargestellt in die URL einzubeziehen.

Ersetzen Sie {_argument_name_} durch Ihre eigenen Werte:

| Platzhalter | Wert | Format oder Beispiel |
|-------------|-------|---|
|{ENDPOINT_KEY} | Der Endpunktschlüssel der trainierten LUIS-Anwendung. |xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx|
|{BILLING_ENDPOINT} | Den Wert des Abrechnungsendpunkts finden Sie auf der Übersichtsseite von Azure `Cognitive Services`. |https://westus.api.cognitive.microsoft.com/luis/v2.0|

> [!IMPORTANT]
> Die Optionen `Eula`, `Billing` und `ApiKey` müssen angegeben werden, um den Container auszuführen, andernfalls wird der Container nicht gestartet.  Weitere Informationen finden Sie unter [Abrechnung](luis-container-howto.md#billing).
> Der Wert ApiKey ist der **Schlüssel** von der Seite „Schlüssel und Endpunkte“ im LUIS-Portal. Er ist auch auf der Seite mit den Schlüsseln der Azure `Cognitive Services`-Ressourcen verfügbar. 

### <a name="basic-example"></a>Einfaches Beispiel

Das folgende Beispiel enthält die wenigsten Argumente, die zum Ausführen des Containers erforderlich sind:

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 2 \
--mount type=bind,src=c:\input,target=/input \
--mount type=bind,src=c:\output,target=/output \
mcr.microsoft.com/azure-cognitive-services/luis:latest \
Eula=accept \
Billing={BILLING_ENDPOINT} \
ApiKey={ENDPOINT_KEY}
```

> [!Note] 
> Der gezeigte Befehl verwendet das Verzeichnis auf dem Laufwerk `c:`, um Berechtigungskonflikte in Windows zu vermeiden. Wenn Sie ein bestimmtes Verzeichnis als Eingabeverzeichnis verwenden möchten, müssen Sie dem Docker-Dienst möglicherweise die erforderliche Berechtigung gewähren. Im obigen Docker-Befehl wird der umgekehrte Schrägstrich (`\`) als Zeilenfortsetzungszeichen verwendet. Ersetzen oder entfernen Sie diesen je nach den Anforderungen des Betriebssystems Ihres [Hostcomputers](luis-container-howto.md#the-host-computer). Ändern Sie die Reihenfolge der Argumente nur, wenn Sie mit Docker-Containern sehr gut vertraut sind.


### <a name="applicationinsights-example"></a>Beispiel für ApplicationInsights

Im folgenden Beispiel wird das ApplicationInsights-Argument so festgelegt, dass während der Ausführung des Containers Telemetriedaten an Application Insights gesendet werden:

```bash
docker run --rm -it -p 5000:5000 --memory 6g --cpus 2 \
--mount type=bind,src=c:\input,target=/input \
--mount type=bind,src=c:\output,target=/output \
mcr.microsoft.com/azure-cognitive-services/luis:latest \
Eula=accept \
Billing={BILLING_ENDPOINT} \
ApiKey={ENDPOINT_KEY}
InstrumentationKey={INSTRUMENTATION_KEY}
```

### <a name="logging-example"></a>Beispiel für die Protokollierung 

Der folgende Befehl legt in `Logging:Console:LogLevel` die Protokollierungsstufe auf [`Information`](https://msdn.microsoft.com) fest. 

```bash
docker run --rm -it -p 5000:5000 --memory 6g --cpus 2 \
--mount type=bind,src=c:\input,target=/input \
--mount type=bind,src=c:\output,target=/output \
mcr.microsoft.com/azure-cognitive-services/luis:latest \
Eula=accept \
Billing={BILLING_ENDPOINT} \
ApiKey={ENDPOINT_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="next-steps"></a>Nächste Schritte

* Weitere Informationen finden Sie unter [Installieren und Ausführen von Containern](luis-container-howto.md).
* Unter [Problembehandlung ](troubleshooting.md) finden Sie Informationen zum Beheben von Problemen im Zusammenhang mit LUIS-Funktionen.
* Verwenden weiterer [Cognitive Services-Container](../cognitive-services-container-support.md)
