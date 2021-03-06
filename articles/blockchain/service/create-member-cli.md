---
title: Erstellen des Azure Blockchain-Diensts mit der Azure CLI
description: Verwenden Sie den Azure Blockchain-Dienst, um ein Blockchainmitglied mithilfe der Azure CLI zu erstellen.
services: azure-blockchain
keywords: ''
author: PatAltimore
ms.author: patricka
ms.date: 05/02/2019
ms.topic: quickstart
ms.service: azure-blockchain
ms.reviewer: seal
manager: femila
ms.openlocfilehash: e1b7558ea83c8948a8984215e15040e4d929cb1b
ms.sourcegitcommit: f6ba5c5a4b1ec4e35c41a4e799fb669ad5099522
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/06/2019
ms.locfileid: "65141377"
---
# <a name="quickstart-create-an-azure-blockchain-service-blockchain-member-using-azure-cli"></a>Schnellstart: Erstellen eines Blockchainmitglieds für den Azure Blockchain-Dienst mithilfe der Azure CLI

Bei Azure Blockchain handelt es sich um eine Blockchainplattform, über die Sie Ihre Geschäftslogik in einem Smart Contract ausführen können. In dieser Schnellstartanleitung wird veranschaulicht, wie Sie zum Einstieg ein Blockchainmitglied über die Azure CLI erstellen.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="launch-azure-cloud-shell"></a>Starten von Azure Cloud Shell

Azure Cloud Shell ist eine kostenlose interaktive Shell, mit der Sie die Schritte in diesem Artikel ausführen können. Sie verfügt über allgemeine vorinstallierte Tools und ist für die Verwendung mit Ihrem Konto konfiguriert.

Wählen Sie zum Öffnen von Cloud Shell oben rechts in einem Codeblock einfach die Option **Ausprobieren**. Sie können Cloud Shell auch auf einer separaten Browserregisterkarte starten, indem Sie zu [https://shell.azure.com/bash](https://shell.azure.com/bash) navigieren. Wählen Sie **Kopieren**, um die Blöcke mit dem Code zu kopieren. Fügen Sie ihn anschließend in Cloud Shell ein, und drücken Sie die EINGABETASTE, um ihn auszuführen.

Wenn Sie es vorziehen, die CLI lokal zu installieren und zu verwenden, müssen Sie für diese Schnellstartanleitung mindestens Version 2.0.51 der Azure CLI verwenden. Führen Sie `az --version` aus, um die Version zu finden. Wenn Sie die CLI installieren oder aktualisieren müssen, finden Sie die entsprechende Anleitung unter [Installieren der Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Erstellen einer Ressourcengruppe

Erstellen Sie mit dem Befehl [az group create](https://docs.microsoft.com/cli/azure/group) eine Ressourcengruppe. Eine Azure-Ressourcengruppe ist ein logischer Container, in dem Azure-Ressourcen bereitgestellt und verwaltet werden. Im folgenden Beispiel wird eine Ressourcengruppe mit dem Namen *myResourceGroup* am Standort *eastus* erstellt:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-blockchain-member"></a>Erstellen eines Blockchainmitglieds

Erstellen Sie ein Blockchainmitglied im Azure Blockchain-Dienst, das das Quorum-Ledgerprotokoll in einem neuen Konsortium ausführt.

Es gibt mehrere Parameter und Eigenschaften, die Sie übergeben müssen. Ersetzen Sie die folgenden Parameter durch Ihre Werte.

| Parameter | BESCHREIBUNG |
|---------|-------------|
| **resource-group** | Name der Ressourcengruppe, in der die Ressourcen des Azure Blockchain-Diensts erstellt werden. Verwenden Sie die Ressourcengruppe aus, die Sie im vorherigen Abschnitt erstellt haben.
| **name** | Ein eindeutiger Name, der Ihr Blockchainmitglied in Azure Blockchain identifiziert. Der Name wird für die Adresse eines öffentlichen Endpunkts verwendet. Beispiel: `myblockchainmember.blockchain.azure.com`.
| **Speicherort** | Die Azure-Region, in der das Blockchainmitglied erstellt wird. Beispiel: `eastus`. Wählen Sie den Standort aus, der Ihren Benutzern oder anderen Azure-Anwendungen am nächsten liegt.
| **password** | Das Kennwort des Mitgliedskontos. Das Mitgliedskontokennwort wird verwendet, um den öffentlichen Endpunkt des Blockchainmitglieds zu authentifizieren. Hierzu wird die Standardauthentifizierung verwendet.
| **consortium** | Name des Konsortiums, dem beigetreten oder das erstellt werden soll.
| **consortiumManagementAccountPassword** | Das Verwaltungskennwort des Konsortiums. Hiermit kann einem Konsortium beigetreten werden.
| **skuName** | Tarifart. Verwenden Sie S0 für Standard und B0 für Basic.

```azurecli-interactive
az resource create --resource-group myResourceGroup --name myblockchainmember --resource-type Microsoft.Blockchain/blockchainMembers --is-full-object --properties "{ \"location\": \"eastus\", \"properties\": {\"password\": \"strongMemberAccountPassword@1\", \"protocol\": \"Quorum\", \"consortium\": \"myConsortiumName\", \"consortiumManagementAccountPassword\": \"strongConsortiumManagementPassword@1\" }, \"sku\": { \"name\": \"S0\" } }"
```

Es dauert etwa 10 Minuten, das Blockchainmitglied und die zugehörigen Ressourcen zu erstellen.

Im Folgenden finden Sie die Beispielausgabe, wenn das Erstellen erfolgreich war.

```json
{
  "id": "/subscriptions/<subscriptionId>/resourceGroups/myResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/mymembername",
  "kind": null,
  "location": "eastus",
  "name": "mymembername",
  "properties": {
    "ConsortiumMemberDisplayName": "mymembername",
    "consortium": "myConsortiumName",
    "consortiumManagementAccountAddress": "0xfe5fbb9d1036298abf415282f52397ade5d5beef",
    "consortiumManagementAccountPassword": null,
    "consortiumRole": "ADMIN",
    "dns": "mymembername.blockchain.azure.com",
    "protocol": "Quorum",
    "provisioningState": "Succeeded",
    "userName": "mymembername",
    "validatorNodesSku": {
      "capacity": 2
    }
  },
  "resourceGroup": "myResourceGroup",
  "sku": {
    "name": "S0",
    "tier": "Standard"
  },
  "type": "Microsoft.Blockchain/blockchainMembers"
}
```

## <a name="clean-up-resources"></a>Bereinigen von Ressourcen

Das von Ihnen erstellte Blockchainmitglied können Sie für die nächste Schnellstartanleitung oder das nächste Tutorial verwenden. Wenn Ressourcen nicht mehr benötigt werden, können Sie diese löschen, indem Sie die `myResourceGroup`-Ressourcengruppe löschen, die Sie mit Azure Blockchain erstellt haben.

Führen Sie den folgenden Befehl aus, um die Ressourcengruppe und alle zugehörigen Ressourcen zu entfernen:

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Nächste Schritte

Da Sie nun ein Blockchainmitglied erstellt haben, testen Sie einen der Schnellstarts zum Herstellen einer Verbindung mit einem Transaktionsknoten für [Geth](connect-geth.md), [MetaMask](connect-metamask.md) oder [Truffle](connect-truffle.md).

> [!div class="nextstepaction"]
> [Use Truffle to connect to a an Azure Blockchain Service network (Verwenden von Truffle zum Herstellen einer Verbindung mit einem Azure Blockchain-Netzwerk)](connect-truffle.md)
