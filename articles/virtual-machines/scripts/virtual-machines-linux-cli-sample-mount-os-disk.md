---
title: Azure CLI-Beispielskript – Einbinden eines Betriebssystem-Datenträgers | Microsoft-Dokumentation
description: Azure CLI-Beispielskript – Einbinden eines Betriebssystem-Datenträgers
services: virtual-machines-linux
documentationcenter: virtual-machines
author: cynthn
manager: jeconnoc
editor: tysonn
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 02/27/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 6f2d4c9a7871e0917b33407605abe1389eb4420e
ms.sourcegitcommit: 039263ff6271f318b471c4bf3dbc4b72659658ec
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 02/06/2019
ms.locfileid: "55753604"
---
# <a name="troubleshoot-a-vms-operating-system-disk"></a>Problembehandlung bei einem Betriebssystem-Datenträger eines virtuellen Computers

Wenn bei einem virtuellen Computer ein Fehler oder Probleme aufgetreten sind, bindet dieses Skript dessen Betriebssystem-Datenträger als Datenträger für Daten in einen zweiten virtuellen Computer ein. Dies kann bei der Behandlung von Datenträgerproblemen oder bei der Wiederherstellung von Daten nützlich sein.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Beispielskript

[!code-azurecli-interactive[main](../../../cli_scripts/virtual-machine/mount-os-disk/mount-os-disk.sh "Quick Create VM")]

## <a name="script-explanation"></a>Erläuterung des Skripts

In diesem Skript werden die folgenden Befehle verwendet, um eine Ressourcengruppe, einen virtuellen Computer und alle zugehörigen Ressourcen zu erstellen. Jeder Befehl in der Tabelle ist mit der zugehörigen Dokumentation verknüpft.

| Get-Help | Notizen |
|---|---|
| [az vm show](https://docs.microsoft.com/cli/azure/vm) | Gibt eine Liste virtueller Computer zurück. In diesem Fall wird die Abfrageoption verwendet, um den Betriebssystem-Datenträger des virtuellen Computers zurückzugeben. Dieser Wert wird dann einer Variablen namens „uri“ hinzugefügt. |
| [az vm delete](https://docs.microsoft.com/cli/azure/vm) | Löscht einen virtuellen Computer. |
| [az vm create](https://docs.microsoft.com/cli/azure/vm) | Erstellt einen virtuellen Computer.  |
| [az vm disk attach](https://docs.microsoft.com/cli/azure/vm/disk) | Fügt einem virtuellen Computer einen Datenträger an. |
| [az vm list-ip-addresses](https://docs.microsoft.com/cli/azure/vm) | Gibt die IP-Adressen eines virtuellen Computers zurück. |

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zur Azure CLI finden Sie in der [Azure CLI-Dokumentation](https://docs.microsoft.com/cli/azure).

Zusätzliche VM-CLI-Skriptbeispiele finden Sie in der [Dokumentation zu Linux-VMs in Azure](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
