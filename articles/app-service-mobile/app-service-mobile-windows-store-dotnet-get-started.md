---
title: Erstellen einer App für die universelle Windows-Plattform (UWP) mit Azure Mobile Apps | Microsoft-Dokumentation
description: Dieses Tutorial enthält eine Einführung in die Verwendung von Azure Mobile App-Back-Ends zum Entwickeln von Apps für die universelle Windows-Plattform (UWP) in C#, Visual Basic oder JavaScript.
services: app-service\mobile
documentationcenter: windows
author: conceptdev
manager: crdun
editor: ''
ms.assetid: 47124296-2908-4d92-85e0-05c4aa6db916
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-windows
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 08/17/2018
ms.author: crdun
ms.openlocfilehash: 959c1ff8b199320105f650a7eb62a04bedb03b3b
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/08/2019
ms.locfileid: "65412788"
---
# <a name="create-a-windows-app-with-an-azure-backend"></a>Erstellen einer Windows-App mit einem Azure-Back-End

[!INCLUDE [app-service-mobile-selector-get-started](../../includes/app-service-mobile-selector-get-started.md)]

## <a name="overview"></a>Übersicht

In diesem Tutorial erfahren Sie, wie Sie einer App für die universelle Windows-Plattform (UWP) einen cloudbasierten Back-End-Dienst hinzufügen. Weitere Informationen finden Sie unter [Was sind Mobile Apps?](app-service-mobile-value-prop.md) Im Folgenden sehen Sie Screenshots aus der fertigen App:

![Fertige Desktop-App](./media/app-service-mobile-windows-store-dotnet-get-started/mobile-quickstart-completed-desktop.png)

Die Absolvierung dieses Tutorials wird für alle anderen Mobile App-Tutorials für UWP-Apps vorausgesetzt.

## <a name="prerequisites"></a>Voraussetzungen

Für dieses Tutorial benötigen Sie Folgendes:

* Ein aktives Azure-Konto. Falls Sie kein Konto besitzen, können Sie sich für eine Azure-Testversion registrieren. So erhalten Sie bis zu 10 kostenlose mobile Apps, die Sie auch nach Ablauf der Testversion weiter nutzen können. Ausführliche Informationen finden Sie unter [Kostenlose Azure-Testversion](https://azure.microsoft.com/pricing/free-trial/).
* Windows 10
* [Visual Studio Community].
* Kenntnisse in der Entwicklung von UWP-Apps. In der [UWP-Dokumentation](https://docs.microsoft.com/windows/uwp/) finden Sie Informationen zu den [ersten Schritten](https://docs.microsoft.com/windows/uwp/get-started/get-set-up) beim Erstellen von UWP-Apps.

## <a name="create-a-new-azure-mobile-app-backend"></a>Erstellen eines neuen Azure Mobile App-Back-Ends

Führen Sie die folgenden Schritte aus, um ein neues mobiles App-Back-End zu erstellen.

[!INCLUDE [app-service-mobile-dotnet-backend-create-new-service](../../includes/app-service-mobile-dotnet-backend-create-new-service.md)]

Sie haben nun ein Azure-Back-End für mobile Apps bereitgestellt, das von Ihren mobilen Clientanwendungen verwendet werden kann. Als Nächstes laden Sie ein Serverprojekt für ein einfaches "Aufgabenlisten"-Back-End herunter und veröffentlichen es in Azure.

## <a name="configure-the-server-project"></a>Konfigurieren des Serverprojekts

[!INCLUDE [app-service-mobile-configure-new-backend.md](../../includes/app-service-mobile-configure-new-backend.md)]

## <a name="download-and-run-the-client-project"></a>Herunterladen und Ausführen des Clientprojekts

Sobald Sie Ihr Back-End für mobile Apps konfiguriert haben, können Sie eine neue Client-App erstellen oder eine vorhandene App so ändern, dass eine Verbindung mit Azure hergestellt wird. In diesem Abschnitt laden Sie ein Projekt für eine UWP-Beispiel-App herunter, das zum Herstellen einer Verbindung mit dem Mobile App-Back-End angepasst wird.

1. Klicken Sie im Blatt **Schnellstart** für Ihr Back-End für mobile Apps auf **Neue App erstellen** > **Herunterladen**, und entpacken Sie die komprimierten Projektdateien auf Ihrem lokalen Computer.

    ![Windows-Schnellstartprojekt herunterladen](./media/app-service-mobile-windows-store-dotnet-get-started/mobile-app-windows-quickstart.png)

2. Öffnen Sie das UWP-Projekt, und drücken Sie F5, um die App bereitzustellen und auszuführen.
3. Geben Sie in der App unter **TodoItem** einfügen einen beschreibenden Text ein, zum Beispiel *Abschließen des Tutorials*. Klicken Sie dann auf **Speichern**.

    ![Windows-Schnellstart – Vollständig für Desktop](./media/app-service-mobile-windows-store-dotnet-get-started/mobile-quickstart-startup.png)

    Dadurch wird eine POST-Anforderung an das neue in Azure gehostete mobile App-Back-End gesendet.

> [!TIP]
> Sie können das UWP-App-Projekt der gleichen Projektmappe hinzufügen wie das Serverprojekt, falls Sie das .NET-Back-End nutzen. Dies erleichtert das Debuggen und Testen der App und des Back-Ends in der gleichen Visual Studio-Projektmappe. Wenn Sie der Back-End-Projektmappe ein UWP-App-Projekt hinzufügen möchten, müssen Sie Visual Studio 2017 oder höher verwenden.

## <a name="next-steps"></a>Nächste Schritte

* [Hinzufügen von Authentifizierung zur App](app-service-mobile-windows-store-dotnet-get-started-users.md)  
   Enthält Informationen über die Authentifizierung von Benutzern der App mit einem Identitätsanbieter.
* [Hinzufügen von Pushbenachrichtigungen zur App](app-service-mobile-windows-store-dotnet-get-started-push.md)  
   Hier erfahren Sie, wie Sie Ihrer App die Unterstützung von Pushbenachrichtigungen hinzufügen und Ihr Mobile App-Back-End für die Verwendung von Azure Notification Hubs zum Senden von Pushbenachrichtigungen konfigurieren.
* [Aktivieren der Offlinesynchronisierung für Ihre App](app-service-mobile-windows-store-dotnet-get-started-offline-data.md)  
  Erfahren Sie, wie Sie mithilfe eines Mobile App-Back-Ends Ihrer App Offlineunterstützung hinzufügen. Die Offlinesynchronisierung ermöglicht Endbenutzern die Interaktion mit einer mobilen App (also das Anzeigen, Hinzufügen oder Ändern von Daten), auch wenn keine Netzwerkverbindung besteht.

<!-- Anchors. -->
<!-- Images. -->
<!-- URLs. -->
[Mobile App SDK]: https://go.microsoft.com/fwlink/?LinkId=257545
[Azure portal]: https://portal.azure.com/
[Visual Studio Community]: https://go.microsoft.com/fwLink/p/?LinkID=534203
