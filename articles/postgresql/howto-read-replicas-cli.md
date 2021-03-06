---
title: Verwalten von Lesereplikaten für Azure Database for PostgreSQL – Einzelserver über die Azure-Befehlszeilenschnittstelle
description: Erfahren Sie, wie Sie Lesereplikate für Azure Database for PostgreSQL verwalten – Einzelserver über die Azure-Befehlszeilenschnittstelle.
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: 9730faf3191ef2e2bd0b6c3caddefa0492b33fc5
ms.sourcegitcommit: 8fc5f676285020379304e3869f01de0653e39466
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/09/2019
ms.locfileid: "65510235"
---
# <a name="create-and-manage-read-replicas-from-the-azure-cli"></a>Erstellen und Verwalten von Lesereplikaten über die Azure CLI

In diesem Artikel erfahren Sie, wie Sie Lesereplikate in Azure Database for PostgreSQL über die Azure CLI erstellen und verwalten. Weitere Informationen zu Lesereplikaten finden Sie in der [Übersicht](concepts-read-replicas.md).

> [!NOTE]
> Die Azure-Befehlszeilenschnittstelle unterstützt noch nicht das Erstellen von Replikaten in einer anderen Region als der des Masterservers. Verwenden Sie das [Azure-Portal](howto-read-replicas-portal.md), um ein regionsübergreifendes Replikat zu erstellen.

## <a name="prerequisites"></a>Voraussetzungen
- Ein [Azure Database for PostgreSQL-Server](quickstart-create-server-up-azure-cli.md), der als Masterserver verwendet wird

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

Wenn Sie die CLI lokal installieren und verwenden möchten, müssen Sie für diesen Artikel die Azure CLI-Version 2.0 oder höher ausführen. Führen Sie den Befehl `az --version` aus, um die installierte Version anzuzeigen. Informationen zum Durchführen einer Installation oder eines Upgrades finden Sei bei Bedarf unter [Installieren der Azure CLI]( /cli/azure/install-azure-cli). 


## <a name="prepare-the-master-server"></a>Vorbereiten des Masterservers
Diese Schritte müssen durchgeführt werden, um einen Masterserver in den Tarifen „Universell“ und „Arbeitsspeicheroptimiert“ vorzubereiten.

Auf dem Masterserver muss der Parameter `azure.replication_support` auf **REPLICA** festgelegt werden. Bei Änderung dieses statischen Parameters ist ein Neustart des Servers erforderlich, damit die Änderung wirksam wird.

1. Legen Sie `azure.replication_support` auf „REPLICA“ fest.

   ```azurecli-interactive
   az postgres server configuration set --resource-group myresourcegroup --server-name mydemoserver --name azure.replication_support --value REPLICA
   ```

2. Führen Sie einen Neustart durch, um die Änderung auf dem Server zu übernehmen.

   ```azurecli-interactive
   az postgres server restart --name mydemoserver --resource-group myresourcegroup
   ```

## <a name="create-a-read-replica"></a>Erstellen eines Lesereplikats

Der Befehl [az postgres server replica create](/cli/azure/postgres/server/replica?view=azure-cli-latest#az-postgres-server-replica-create) erfordert die folgenden Parameter:

| Einstellung | Beispielwert | BESCHREIBUNG  |
| --- | --- | --- |
| resource-group | myresourcegroup |  Die Ressourcengruppe, in der der Replikatserver erstellt wird.  |
| name | mydemoserver-replica | Der Name des neuen Replikatservers, der erstellt wird. |
| source-server | mydemoserver | Der Name oder die Ressourcen-ID des vorhandenen Masterservers, von dem die Replikation erfolgt. |

```azurecli-interactive
az postgres server replica create --name mydemoserver-replica --source-server mydemoserver --resource-group myresourcegroup
```

Wenn Sie den Parameter `azure.replication_support` auf einem universellen oder arbeitsspeicheroptimierten Masterserver nicht auf **REPLICA** festgelegt und den Server nicht neu gestartet haben, erhalten Sie einen Fehler. Führen Sie diese beiden Schritte aus, bevor Sie ein Replikat erstellen.

Ein Replikat wird mit der gleichen Serverkonfiguration wie der Masterserver erstellt. Nachdem ein Replikat erstellt wurde, können mehrere Einstellungen unabhängig vom Masterserver geändert werden: die Computegeneration, die virtuellen Kerne, der Speicher und der Aufbewahrungszeitraum für Sicherungen. Auch der Tarif kann unabhängig geändert werden, allerdings nicht in den oder aus dem Tarif „Basic“.

> [!IMPORTANT]
> Bevor Sie die Konfiguration eines Masterservers mit neuen Werten aktualisieren, ändern Sie die Replikatkonfiguration in gleiche oder größere Werte. Durch diese Aktion wird sichergestellt, dass das Replikat mit allen Änderungen, die auf dem Masterserver durchgeführt werden, Schritt halten kann.

## <a name="list-replicas"></a>Auflisten von Replikaten
Sie können die Liste der Replikate eines Masterservers mit dem Befehl [az postgres server replica list](/cli/azure/postgres/server/replica?view=azure-cli-latest#az-postgres-server-replica-list) anzeigen.

```azurecli-interactive
az postgres server replica list --server-name mydemoserver --resource-group myresourcegroup 
```

## <a name="stop-replication-to-a-replica-server"></a>Beenden der Replikation auf einem Replikatserver
Sie können die Replikation zwischen einem Masterserver und einem Lesereplikat mit dem Befehl [az postgres server replica stop](/cli/azure/postgres/server/replica?view=azure-cli-latest#az-postgres-server-replica-stop) beenden.

Das Beenden der Replikation zwischen einem Masterserver und einem Lesereplikat kann nicht mehr rückgängig gemacht werden. Das Lesereplikat wird zu einem eigenständigen Server, der sowohl Lese- als auch Schreibvorgänge unterstützt. Der eigenständige Server kann nicht wieder in ein Replikat umgewandelt werden.

```azurecli-interactive
az postgres server replica stop --name mydemoserver-replica --resource-group myresourcegroup 
```

## <a name="delete-a-master-or-replica-server"></a>Löschen eines Master- oder Replikatservers
Wenn Sie einen Master- oder Replikatserver löschen möchten, verwenden Sie den Befehl [az postgres server delete](/cli/azure/postgres/server?view=azure-cli-latest#az-postgres-server-delete).

Wenn Sie einen Masterserver löschen, wird die Replikation auf allen Lesereplikaten beendet. Die Lesereplikate werden zu eigenständigen Servern, die nun Lese- und Schreibvorgänge unterstützen.

```azurecli-interactive
az postgres server delete --name myserver --resource-group myresourcegroup
```

## <a name="next-steps"></a>Nächste Schritte
Erfahren Sie mehr über [Lesereplikate in Azure Database for PostgreSQL](concepts-read-replicas.md).