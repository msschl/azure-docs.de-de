---
title: Verwenden der dynamischen DRM-Verschlüsselung und des Lizenzbereitstellungsdiensts mit Azure Media Services | Microsoft-Dokumentation
description: Mithilfe von Azure Media Services können Sie Streams mit Microsoft PlayReady-, Google Widevine- und Apple FairPlay-Lizenzen verschlüsselt übermitteln.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: juliako
ms.custom: seodec18
ms.openlocfilehash: 24ea6b2b44518b4cf75389585caf42ff6bc6722f
ms.sourcegitcommit: 0568c7aefd67185fd8e1400aed84c5af4f1597f9
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/06/2019
ms.locfileid: "65191068"
---
# <a name="tutorial-use-drm-dynamic-encryption-and-license-delivery-service"></a>Tutorial: Verwenden der dynamischen DRM-Verschlüsselung und des Lizenzbereitstellungsdiensts

Mithilfe von Azure Media Services können Sie Streams mit Microsoft PlayReady-, Google Widevine- und Apple FairPlay-Lizenzen verschlüsselt übermitteln. Ausführliche Erläuterungen finden Sie unter [Inhaltsschutz mit dynamischer Verschlüsselung](content-protection-overview.md).

Darüber hinaus bietet Media Services einen Dienst zum Übermitteln von PlayReady-, Widevine- und FairPlay-DMR-Lizenzen. Wenn ein Benutzer die mit DRM geschützten Inhalte anfordert, fordert die Playeranwendung eine Lizenz vom Media Services-Lizenzdienst an. Wenn die Playeranwendung autorisiert wurde, gibt der Media Services-Lizenzdienst eine Lizenz für den Player aus. Eine Lizenz enthält den Entschlüsselungsschlüssel, der vom Clientplayer zum Entschlüsseln und Streamen des Inhalts verwendet werden kann.

Dieser Artikel basiert auf dem Beispiel für [Verschlüsseln mit DRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM). 

Das in diesem Artikel beschriebene Beispiel führt zu folgendem Ergebnis:

![AMS mit DRM-Schutz (Video)](./media/protect-with-drm/ams_player.png)

Dieses Tutorial veranschaulicht folgende Vorgehensweisen:    

> [!div class="checklist"]
> * Erstellen einer Codierungstransformation
> * Festlegen des Signaturschlüssels, der für die Überprüfung des Tokens verwendet wird
> * Festlegen der Anforderungen für die Richtlinie für Inhaltsschlüssel
> * Erstellen eines Streaminglocators mit der angegebenen Streamingrichtlinie
> * Erstellen einer URL zum Wiedergeben der Datei

## <a name="prerequisites"></a>Voraussetzungen

Zum Abschließen dieses Lernprogramms müssen folgende Voraussetzungen erfüllt sein:

* Lesen Sie den Artikel [Übersicht über den Inhaltsschutz](content-protection-overview.md).
* Lesen Sie die Informationen unter [Entwurf eines Multi-DRM-Inhaltsschutzsystems mit Zugriffssteuerung](design-multi-drm-system-with-access-control.md).
* Installation von Visual Studio Code oder Visual Studio
* Erstellen Sie ein Azure Media Services-Konto, wie in [dieser Schnellstartanleitung](create-account-cli-quickstart.md) beschrieben.
* Rufen Sie die zur Verwendung von Media Services-APIs erforderlichen Anmeldeinformationen ab (siehe [Zugreifen auf APIs](access-api-cli-how-to.md)).
* Legen Sie die entsprechenden Werte in der Anwendungskonfigurationsdatei (appsettings.json) fest.

## <a name="download-code"></a>Code herunterladen

Klonen Sie ein GitHub-Repository, das das vollständige in diesem Artikel beschriebene .NET-Beispiel enthält, mit dem folgenden Befehl auf Ihrem Computer:

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```
 
Das Beispiel „Encrypt with DRM“ befindet sich im Ordner [EncryptWithDRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM).

> [!NOTE]
> Mit dem Beispiel werden bei jeder Ausführung der Anwendung eindeutige Ressourcen erstellt. Normalerweise verwenden Sie vorhandene Ressourcen wie Transformationen und Richtlinien erneut (wenn vorhandene Ressourcen über erforderliche Konfigurationen verfügen). 

## <a name="start-using-media-services-apis-with-net-sdk"></a>Starten der Verwendung von Media Services-APIs mit dem .NET SDK

Um mit der Verwendung von Media Services-APIs in .NET zu beginnen, müssen Sie ein **AzureMediaServicesClient**-Objekt erstellen. Zum Erstellen des Objekts müssen Sie Anmeldeinformationen bereitstellen, die für den Client zum Herstellen einer Verbindung mit Azure mithilfe von Azure AD erforderlich sind. In dem Code, den Sie zu Beginn des Artikels geklont haben, erstellt die Funktion **GetCredentialsAsync** das ServiceClientCredentials-Objekt basierend auf den in der lokalen Konfigurationsdatei angegebenen Anmeldeinformationen. 

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CreateMediaServicesClient)]

## <a name="create-an-output-asset"></a>Erstellen eines Ausgabemedienobjekts  

Das [Ausgabeobjekt](https://docs.microsoft.com/rest/api/media/assets) speichert das Ergebnis Ihres Codierungsauftrags.  

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CreateOutputAsset)]
 
## <a name="get-or-create-an-encoding-transform"></a>Abrufen oder Erstellen einer Codierungstransformation

Beim Erstellen einer neuen [Transformations](https://docs.microsoft.com/rest/api/media/transforms)instanz müssen Sie angeben, was als Ausgabe generiert werden soll. Der erforderliche Parameter ist ein **TransformOutput**-Objekt, wie im folgenden Code gezeigt. Jedes **TransformOutput**-Objekt enthält eine **Voreinstellung**. Die **Voreinstellung** beschreibt die schrittweisen Anweisungen von Video- und/oder Audioverarbeitungsvorgängen, die verwendet werden sollen, um das gewünschte **TransformOutput**-Objekt zu generieren. Das in diesem Artikel beschriebene Beispiel verwendet eine integrierte Voreinstellung namens **AdaptiveStreaming**. Die Voreinstellung codiert das Eingabevideo in eine automatisch generierte Bitratenkette (Paare aus Bitrate und Auflösung) auf Basis der Eingabeauflösung und -bitrate und generiert ISO MP4-Dateien mit H.264-Video und AAC-Audio entsprechend jedem Paar aus Bitrate und Auflösung. 

Vor dem Erstellen einer neuen [Transformation](https://docs.microsoft.com/rest/api/media/transforms) sollten Sie zunächst mit der **Get**-Methode überprüfen, ob eine solche bereits vorhanden ist. Der folgende Code veranschaulicht dies.  In Media Services v3 geben **Get**-Methoden für Entitäten **null** zurück, wenn die Entität nicht vorhanden ist (eine Überprüfung des Namens ohne Unterscheidung zwischen Groß-/Kleinschreibung ).

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#EnsureTransformExists)]

## <a name="submit-job"></a>Auftrag übermitteln

Wie bereits erwähnt, ist das [Transformations](https://docs.microsoft.com/rest/api/media/transforms)objekt die Anleitung und ein [Auftrag](https://docs.microsoft.com/rest/api/media/jobs) die tatsächliche Anforderung an Media Services, diese **Transformation** auf ein bestimmtes Eingabevideo oder einen Audioinhalt anzuwenden. Der **Auftrag** gibt Informationen wie den Speicherort des Eingabevideos und den Speicherort für die Ausgabe an.

In diesem Tutorial wird die Auftragseingabe basierend auf einer Datei erstellt, die direkt aus einer [HTTPS-Quell-URL](job-input-from-http-how-to.md) erfasst wird.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#SubmitJob)]

## <a name="wait-for-the-job-to-complete"></a>Warten auf den Abschluss des Auftrags

Der Abschluss des Auftrags nimmt einige Zeit in Anspruch. Wenn er erfolgt ist, möchten Sie benachrichtigt werden. Das Codebeispiel unten zeigt, wie der Status des [Auftrags](https://docs.microsoft.com/rest/api/media/jobs) vom Dienst abgerufen wird. Der Abruf ist aufgrund potenzieller Latenzzeiten keine empfohlene bewährte Methode für Produktionsanwendungen. Der Abruf kann gedrosselt werden, wenn er für ein Konto im Übermaß verwendet wird. Entwickler sollten stattdessen Event Grid verwenden. Weitere Informationen finden Sie unter [Routing von Ereignissen an einen benutzerdefinierten Webendpunkt](job-state-events-cli-how-to.md).

Der **Auftrag** durchläuft in der Regel die folgenden Zustände: **Geplant**, **In Warteschlange**, **Wird verarbeitet**, **Abgeschlossen** (Endzustand). Wenn für den Auftrag ein Fehler aufgetreten ist, erhalten Sie den Zustand **Fehler**. Wenn der Auftrag aktuell abgebrochen wird, erhalten Sie **Abbrechen** und **Abgebrochen**, wenn dies geschehen ist.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#WaitForJobToFinish)]

## <a name="create-a-contentkeypolicy"></a>Erstellen einer ContentKeyPolicy

Ein Inhaltsschlüssel ermöglicht den sicheren Zugriff auf Ihre Medienobjekte. Sie müssen eine Richtlinie für den Inhaltsschlüssel erstellen, mit der konfiguriert wird, wie der Inhaltsschlüssel an Endclients übermittelt wird. Der Inhaltsschlüssel wird StreamingLocator zugeordnet. Media Services bietet außerdem den Schlüsselbereitstellungsdienst, der Verschlüsselungsschlüssel und Lizenzen an autorisierte Benutzer übermittelt. 

Sie müssen die Anforderungen (Einschränkungen) für die Richtlinie für den Inhaltsschlüssel festlegen, die erfüllt sein müssen, um Schlüssel mit der angegebenen Konfiguration bereitzustellen. In diesem Beispiel werden die folgenden Konfigurationen und Anforderungen festgelegt:

* Konfiguration 

    Die [PlayReady](playready-license-template-overview.md)- und [Widevine](widevine-license-template-overview.md)-Lizenzen werden so konfiguriert, dass sie vom Media Services-Lizenzbereitstellungsdienst bereitgestellt werden können. Obwohl in dieser Beispiel-App die [FairPlay](fairplay-license-overview.md)-Lizenz nicht konfiguriert wird, enthält sie eine Methode, die Sie zum Konfigurieren von FairPlay verwenden können. Sie können die FairPlay-Konfiguration als weitere Option hinzufügen.

* Einschränkung

    Die App legt eine Einschränkung des JWT-Tokentyps in der Richtlinie fest.

Wenn ein Stream von einem Player angefordert wird, verwendet Media Services den angegebenen Schlüssel, um Ihren Inhalt dynamisch zu verschlüsseln. Um den Stream zu entschlüsseln, fordert der Player den Schlüssel vom Schlüsselübermittlungsdienst an. Um zu ermitteln, ob der Benutzer berechtigt ist, den Schlüssel zu erhalten, wertet der Dienst die Richtlinie für Inhaltsschlüssel aus, die Sie für den Schlüssel angegeben haben.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetOrCreateContentKeyPolicy)]

## <a name="create-a-streaminglocator"></a>Erstellen eines Streaminglocators

Nachdem die Codierung abgeschlossen ist und die Richtlinie für den Inhaltsschlüssel festgelegt wurde, besteht der nächste Schritt darin, das Video im Ausgabeobjekt Clients für die Wiedergabe zur Verfügung zu stellen. Sie erreichen dies in zwei Schritten: 

1. Erstellen eines [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators)
2. Erstellen der Streaming-URLs, die von Clients verwendet werden können 

Der Vorgang des Erstellens eines **StreamingLocator** wird als „Veröffentlichen“ bezeichnet. Standardmäßig ist der **StreamingLocator** sofort nach dem Vornehmen der API-Aufrufe gültig und bleibt es auch, bis er gelöscht wird (es sei denn, Sie konfigurieren die optionalen Start- und Endzeiten). 

Beim Erstellen eines [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators) müssen Sie den gewünschten **StreamingPolicyName** angeben. In diesem Tutorial verwenden wir eine der vordefinierten Streamingrichtlinien, die Azure Media Services mitteilt, wie der Inhalt für das Streaming veröffentlicht werden soll. In diesem Beispiel wird „StreamingLocator.StreamingPolicyName“ auf die Richtlinie „Predefined_MultiDrmCencStreaming“ festgelegt. Diese Richtlinie gibt an, dass zwei Inhaltsschlüssel (Umschlag und CENC) generiert und für den Locator festgelegt werden sollen. Daher werden die Umschlag-, PlayReady- und Widevine-Verschlüsselungen angewendet (der Schlüssel wird dem Client basierend auf den konfigurierten DRM-Lizenzen bereitgestellt). Wenn Sie den Stream auch mit CBCS (FairPlay) verschlüsseln möchten, verwenden Sie „Predefined_MultiDrmStreaming“. 

> [!IMPORTANT]
> Wenn Sie eine benutzerdefinierte [Streamingrichtlinie](https://docs.microsoft.com/rest/api/media/streamingpolicies) verwenden, sollten Sie eine begrenzte Sammlung solcher Richtlinien für Ihr Media Services-Konto erstellen und diese für Ihre StreamingLocators wiederverwenden, wenn dieselben Verschlüsselungsoptionen und Protokolle benötigt werden. Ihr Media Services-Konto weist ein Kontingent für die Anzahl von StreamingPolicy-Einträgen auf. Sie sollten nicht für jeden StreamingLocator eine neue StreamingPolicy erstellen.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CreateStreamingLocator)]

## <a name="get-a-test-token"></a>Abrufen eines Testtokens
        
In diesem Tutorial wird für die Richtlinie für den Inhaltsschlüssel eine Tokeneinschränkung angegeben. Eine durch Token eingeschränkte Richtlinie gilt nur zusammen mit einem Token, das von einem Sicherheitstokendienst (Security Token Service, STS) ausgestellt wurde. Media Services unterstützt Token im [JSON Web Token](https://msdn.microsoft.com/library/gg185950.aspx#BKMK_3)-Format (JWT). Dieses Format wird auch im Beispiel konfiguriert.

In ContentKeyPolicy wird ContentKeyIdentifierClaim verwendet, d.h., das an den Schlüsselbereitstellungsdienst übergebene Token muss den Bezeichner von ContentKey enthalten. Im Beispiel geben wir beim Erstellen von StreamingLocator keinen Inhaltsschlüssel an, er wird im System zufällig erstellt. Um das Testtoken zu generieren, muss die ContentKeyId abgerufen werden, die in den Anspruch ContentKeyIdentifierClaim eingefügt werden muss.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetToken)]

## <a name="build-a-streaming-url"></a>Erstellen einer Streaming-URL

Nachdem der [StreamingLocator](https://docs.microsoft.com/rest/api/media/streaminglocators) erstellt wurde, können Sie die Streaming-URLs abrufen. Um eine URL zu erstellen, müssen Sie den [StreamingEndpoint](https://docs.microsoft.com/rest/api/media/streamingendpoints)-Hostnamen und den **StreamingLocator**-Pfad miteinander verketten. In diesem Beispiel wird der *standardmäßige* **StreamingEndpoint** verwendet. Bei der erstmaligen Erstellung eines Media Services-Kontos befindet sich dieser *standardmäßige* **StreamingEndpoint** im Zustand „Beendet“. Sie müssen daher **Start** aufrufen.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetMPEGStreamingUrl)]

Wenn Sie die App ausführen, wird Folgendes angezeigt:

![Schützen mit DRM](./media/protect-with-drm/playready_encrypted_url.png)

Sie können einen Browser öffnen und die resultierende URL einfügen, um die Azure Media Player-Demoseite mit der URL und dem bereits ausgefüllten Token zu starten. 
 
## <a name="clean-up-resources-in-your-media-services-account"></a>Bereinigen von Ressourcen in Ihrem Media Services-Konto

Im Allgemeinen sollten Sie alles mit Ausnahme der Objekte bereinigen, die Sie wiederverwenden möchten (in der Regel werden Sie Transformationen wiederverwenden, und Sie werden StreamingLocators usw. persistent speichern). Wenn Sie Ihr Konto nach dem Experimentieren bereinigen möchten, sollten Sie die Ressourcen löschen, bei denen Sie nicht beabsichtigen, sie wiederzuverwenden.  Der folgende Code löscht beispielsweise Aufträge.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#CleanUp)]

## <a name="clean-up-resources"></a>Bereinigen von Ressourcen

Wenn Sie keine Ressourcen in Ihrer Ressourcengruppe mehr benötigen, einschließlich der Media Services und Speicherkonten, die Sie für dieses Tutorial erstellt haben, löschen Sie die zuvor erstellte Ressourcengruppe. 

Führen Sie den folgenden CLI-Befehl aus:

```azurecli
az group delete --name amsResourceGroup
```

## <a name="ask-questions-give-feedback-get-updates"></a>Fragen stellen, Feedback geben, Updates abrufen

Im Artikel [Azure Media Services-Community](media-services-community.md) finden Sie verschiedene Möglichkeiten, Fragen zu stellen, Feedback zu geben und Updates zu Media Services zu bekommen.

## <a name="next-steps"></a>Nächste Schritte

Auschecken

> [!div class="nextstepaction"]
> [Verwenden der dynamischen AES-128-Verschlüsselung und des Schlüsselübermittlungsdiensts](protect-with-aes128.md)

