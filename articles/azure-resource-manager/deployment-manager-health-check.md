---
title: Rollout der Integritätsintegration in Azure-Bereitstellungs-Manager
description: Beschreibt die zahlreiche Regionen umfassende Bereitstellung eines Diensts mit dem Azure-Bereitstellungs-Manager. Zeigt sichere Bereitstellungsmethoden, mit denen Sie die Stabilität Ihrer Bereitstellung vor dem Rollout in alle Regionen überprüfen können.
services: azure-resource-manager
documentationcenter: na
author: mumian
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/08/19
ms.author: jgao
ms.openlocfilehash: 1006cc902cf4b6f763d86165a039a1fec5dc97a1
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467082"
---
# <a name="introduce-health-integration-rollout-to-azure-deployment-manager-public-preview"></a>Rollout der Integritätsintegration in Azure-Bereitstellungs-Manager (öffentliche Vorschau)

Mit dem [Azure-Bereitstellungs-Manager](./deployment-manager-overview.md) können Sie gestaffelte Rollouts von Azure Resource Manager-Ressourcen durchführen. Die Ressourcen werden in einer bestimmten Reihenfolge Region für Region bereitgestellt. Die integrierte Integritätsprüfung des Azure-Bereitstellungs-Managers kann Rollouts überwachen und problematische Rollouts automatisch beenden, sodass Sie Probleme beheben und das Ausmaß der Auswirkungen reduzieren können. Durch dieses Feature kann die Nichtverfügbarkeit von Diensten aufgrund von Regressionen in Updates reduziert werden.

## <a name="health-monitoring-providers"></a>Anbieter für Systemüberwachung

Um die Integritätsintegration so einfach wie möglich zu gestalten, hat Microsoft mit einigen der führenden Unternehmen für Dienstintegritätsüberwachung zusammengearbeitet, um Ihnen eine einfache Copy&Paste-Lösung zu bieten, mit der Sie Integritätsprüfungen in Ihre Bereitstellungen integrieren können. Wenn Sie noch keinen Integritätsmonitor verwenden, eignen sich die folgenden Lösungen optimal für den Einstieg:

| ![Azure-Bereitstellungs-Manager, Systemüberwachungsanbieter Datadog](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-datadog.svg) | ![Azure-Bereitstellungs-Manager, Systemüberwachungsanbieter Site24x7](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-site24x7.svg) | ![Azure-Bereitstellungs-Manager, Systemüberwachungsanbieter Wavefront](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-wavefront.svg) |
|-----|------|------|
|Datadog ist die führende Überwachungs- und Analyseplattform für moderne Cloudumgebungen. Weitere Informationen finden Sie unter [Integration von Datadog in Azure-Bereitstellungs-Manager](https://www.datadoghq.com/azure-deployment-manager/).|Site24x7 ist eine umfassende Überwachungslösung für private und öffentliche Clouddienste. Weitere Informationen finden Sie unter [Integration von Site24x7 in Azure-Bereitstellungs-Manager](https://www.site24x7.com/azure/adm.html).| Wavefront ist eine Überwachungs- und Analyseplattform für Multi-Cloud-Anwendungsumgebungen. Weitere Informationen finden Sie unter [Integration von Wavefront in Azure-Bereitstellungs-Manager](https://go.wavefront.com/wavefront-adm/).|

## <a name="how-service-health-is-determined"></a>Ermittlung der Dienstintegrität

[Anbieter für Systemüberwachung](#health-monitoring-providers) bieten mehrere Mechanismen für die Überwachung von Diensten und zur Benachrichtigung über Probleme mit der Dienstintegrität. [Azure Monitor](/services/monitor/) ist ein Beispiel für ein solches Angebot. Azure Monitor kann zum Erstellen von Warnungen verwendet werden, die Sie erhalten, wenn bestimmte Schwellenwerte überschritten werden. Beispielsweise übersteigt die Arbeitsspeicher- und CPU-Auslastung die erwarteten Werte, wenn Sie ein neues Update für Ihren Dienst bereitstellen. Wenn Sie benachrichtigt werden, können Sie Korrekturmaßnahmen ergreifen.

Diese Systemüberwachungsanbieter bieten in der Regel REST-APIs, damit der Status der Dienstmonitore programmgesteuert überprüft werden kann. Die REST-APIs können entweder mit einem einfachen Signal für den fehlerfreien/fehlerhaften Status (bestimmt durch den HTTP-Antwortcode) und/oder mit detaillierten Informationen über die empfangenen Signale antworten.

Mit dem neuen *healthCheck*-Schritt im Azure-Bereitstellungs-Manager können Sie HTTP-Codes deklarieren, die einen fehlerfreien Dienst angeben. Und wenn Sie komplexere REST-Ergebnisse erzielen möchten, können Sie auch reguläre Ausdrücke angeben, mit denen bei entsprechender Übereinstimmung eine fehlerfreie Antwort angegeben wird.

Der Flow zum Einrichten von Integritätsprüfungen im Azure-Bereitstellungs-Manager sieht wie folgt aus:

1. Erstellen Sie Ihre Integritätsmonitore über einen Systemüberwachungsanbieter Ihrer Wahl.
1. Erstellen Sie einen oder mehrere healthCheck-Schritte im Rahmen Ihres Azure-Bereitstellungs-Manager-Rollouts. Geben Sie in den healthCheck-Schritten die folgenden Informationen an:

    1. Den URI für die REST-API für Ihre Integritätsmonitore (wie von Ihrem Systemüberwachungsanbieter angegeben).
    1. Authentifizierungsinformationen. Derzeit wird nur der Authentifizierungstyp „API-Schlüssel“ unterstützt.
    1. [HTTP-Statuscodes](https://www.wikipedia.org/wiki/List_of_HTTP_status_codes) oder reguläre Ausdrücke, die eine fehlerfreie Antwort definieren. Beachten Sie, dass Sie reguläre Ausdrücke angeben können, die ALLE übereinstimmen müssen, damit die Antwort als fehlerfrei angesehen wird. Oder Sie können Ausdrücke angeben, von denen EIN Ausdruck übereinstimmen muss, damit die Antwort als fehlerfrei angesehen wird. Beide Methoden werden unterstützt.

    Nachfolgend finden Sie ein JSON-Beispiel:

    ```json
    {
      "type": "Microsoft.DeploymentManager/steps",
      "apiVersion": "2018-09-01-preview",
      "name": "healthCheckStep",
      "location": "[parameters('azureResourceLocation')]",
      "properties": {
        "stepType": "healthCheck",
        "attributes": {
          "waitDuration": "PT0M",
          "maxElasticDuration": "PT0M",
          "healthyStateDuration": "PT1M",
          "type": "REST",
          "properties": {
            "healthChecks": [
              {
                "name": "appHealth",
                "request": {
                  "method": "GET",
                  "uri": "[parameters('healthCheckUrl')]",
                  "authentication": {
                    "type": "ApiKey",
                    "name": "code",
                    "in": "Query",
                    "value": "[parameters('healthCheckAuthAPIKey')]"
                  }
                },
                "response": {
                  "successStatusCodes": [
                    "200"
                  ],
                  "regex": {
                    "matches": [
                      "Status: healthy",
                      "Status: warning"
                    ],
                    "matchQuantifier": "Any"
                  }
                }
              }
            ]
          }
        }
      }
    },
    ```

1. Rufen Sie die healthCheck-Schritte zum richtigen Zeitpunkt in Ihrem Azure-Bereitstellungs-Manager-Rollout auf. Im folgenden Beispiel wird ein Schritt zur Integritätsprüfung (healthCheck-Schritt) in **postDeploymentSteps** von **stepGroup2** aufgerufen.

    ```json
    "stepGroups": [
        {
            "name": "stepGroup1",
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit2.name)]",
            "postDeploymentSteps": []
        },
        {
            "name": "stepGroup2",
            "dependsOnStepGroups": ["stepGroup1"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit1.name)]",
            "postDeploymentSteps": [
                {
                    "stepId": "[resourceId('Microsoft.DeploymentManager/steps/', 'healthCheckStep')]"
                }
            ]
        },
        {
            "name": "stepGroup3",
            "dependsOnStepGroups": ["stepGroup2"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit2.name)]",
            "postDeploymentSteps": []
        },
        {
            "name": "stepGroup4",
            "dependsOnStepGroups": ["stepGroup3"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit1.name)]",
            "postDeploymentSteps": []
        }
    ]
    ```

Unter [Tutorial: Verwenden der Integritätsprüfung im Azure-Bereitstellungs-Manager](./deployment-manager-health-check.md) werden Sie durch ein Beispiel geführt.

## <a name="phases-of-a-health-check"></a>Phasen einer Integritätsprüfung

Zu diesem Zeitpunkt weiß der Azure-Bereitstellungs-Manager, wie die Integrität Ihres Diensts abgefragt wird und in welchen Phasen des Rollouts dies erfolgen muss. Der Azure-Bereitstellungs-Manager lässt jedoch auch eine umfassende Konfiguration der zeitlichen Abfolge dieser Überprüfungen zu. Ein healthCheck-Schritt wird in 3 aufeinander folgenden Phasen ausgeführt, deren Dauer jeweils konfiguriert werden kann: 

1. Warten

    1. Nach dem Abschluss eines Bereitstellungsvorgangs können VMs neu gestartet, basierend auf neuen Daten neu konfiguriert oder sogar zum ersten Mal gestartet werden. Es dauert auch einige Zeit, bis die Dienste mit dem Ausgeben von Integritätssignalen beginnen, die vom Systemüberwachungsanbieter in nützlichen Informationen zusammengefasst werden. Während dieses turbulenten Vorgangs ist es möglicherweise nicht sinnvoll, die Dienstintegrität zu überprüfen, weil das Update noch keinen stabilen Zustand erreicht hat. Tatsächlich kann der Dienst zwischen fehlerfreien und fehlerhaften Zuständen oszillieren, während sich die Ressourcen stabilisieren. 
    1. Während der Wartephase wird die Dienstintegrität nicht überwacht. In dieser Phase wird den bereitgestellten Ressourcen Zeit für die Integration eingeräumt, bevor die Integritätsprüfung beginnt. 
1. Elastic

    1. Da man unmöglich in allen Fällen wissen kann, wie lange die Integration der Ressourcen dauert, bis sie stabil sind, ermöglicht die Phase „Elastic“ einen flexiblen Zeitraum zwischen dem Zeitpunkt, an dem die Ressourcen potenziell instabil sind, und dem Zeitpunkt, an dem sie einen fehlerfreien, stabilen Zustand aufrechterhalten müssen.
    1. Mit Beginn der Phase „Elastic“ beginnt der Azure-Bereitstellungs-Manager damit, den bereitgestellten REST-Endpunkt regelmäßig nach der Dienstintegrität abzufragen. Das Abrufintervall ist konfigurierbar. 
    1. Wenn der Integritätsmonitor mit Signalen antwortet, die einen fehlerhaften Dienststatus angeben, werden diese Signale ignoriert, die Phase „Elastic“ wird fortgesetzt, und das Abrufen wird fortgesetzt. 
    1. Sobald der Integritätsmonitor mit Signalen antwortet, die angeben, dass der Dienst fehlerfrei ist, endet die Phase „Elastic“, und die Phase „HealthyState“ beginnt. 
    1. Daher ist die für die Phase „Elastic“ angegebene Dauer die maximale Zeit, die zum Abrufen der Dienstintegrität aufgewendet werden kann, bevor eine fehlerfreie Antwort als obligatorisch angesehen wird. 
1. HealthyState

    1. Während der Phase „HealthyState“ wird die Dienstintegrität im gleichen Intervall wie in der Phase „Elastic“ kontinuierlich abgerufen. 
    1. Für die gesamte angegebene Dauer wird erwartet, dass der Dienst Signale für den fehlerfreien Status vom Systemüberwachungsanbieter beibehält. 
    1. Wenn zu irgendeinem Zeitpunkt eine fehlerhafte Antwort erkannt wird, beendet der Azure-Bereitstellungs-Manager den gesamten Rollout und gibt die REST-Antwort mit den Signalen für einen fehlerhaften Dienst zurück.
    1. Nach Abschluss der Phase „HealthyState“ ist die Integritätsprüfung (der healthCheck-Schritt) abgeschlossen, und die Bereitstellung wird mit dem nächsten Schritt fortgesetzt.

## <a name="next-steps"></a>Nächste Schritte

In diesem Artikel haben Sie erfahren, wie Sie eine Systemüberwachung in den Azure-Bereitstellungs-Manager integrieren. Im nächsten Artikel erfahren Sie, wie Sie mit dem Bereitstellungs-Manager Bereitstellungen ausführen.

> [!div class="nextstepaction"]
> [Tutorial: Verwenden der Integritätsprüfung im Azure-Bereitstellungs-Manager](./deployment-manager-tutorial-health-check.md)