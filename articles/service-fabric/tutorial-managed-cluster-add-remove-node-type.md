---
title: Hinzufügen und Entfernen von Knotentypen eines verwalteten Service Fabric-Clusters
description: In diesem Tutorial erfahren Sie, wie Sie Knotentypen eines verwalteten Service Fabric-Clusters hinzufügen und entfernen.
ms.topic: tutorial
ms.date: 05/10/2021
ms.openlocfilehash: 73e6f7676f398287b607deb80f490267a7665733
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/10/2021
ms.locfileid: "109685799"
---
# <a name="tutorial-add-and-remove-node-types-from-a-service-fabric-managed-cluster"></a>Tutorial: Hinzufügen und Entfernen von Knotentypen in einem verwalteten Service Fabric-Cluster

In dieser Tutorialreihe wird Folgendes behandelt:

> [!div class="checklist"]
> * [Bereitstellen eines verwalteten Service Fabric-Clusters](tutorial-managed-cluster-deploy.md)
> * [Aufskalieren eines verwalteten Service Fabric-Clusters](tutorial-managed-cluster-scale.md)
> * Hinzufügen und Entfernen von Knoten in einem verwalteten Service Fabric-Cluster
> * [Bereitstellen einer Anwendung in einem verwalteten Service Fabric-Cluster](tutorial-managed-cluster-deploy-app.md)

In diesem Teil der Reihe wird Folgendes behandelt:

> [!div class="checklist"]
> * Hinzufügen eines Knotentyps zu einem verwalteten Service Fabric-Cluster
> * Löschen eines Knotentyps aus einem verwalteten Service Fabric-Cluster

## <a name="prerequisites"></a>Voraussetzungen

* Ein verwalteter Service Fabric-Cluster (siehe [*Bereitstellen eines verwalteten Clusters*](tutorial-managed-cluster-deploy.md)).
* [Azure PowerShell 4.7.0](/powershell/azure/release-notes-azureps#azservicefabric) oder höher (siehe [*Installieren von Azure PowerShell*](/powershell/azure/install-az-ps)).

## <a name="add-a-node-type-to-a-service-fabric-managed-cluster"></a>Hinzufügen eines Knotentyps zu einem verwalteten Service Fabric-Cluster

Sie können einem verwalteten Service Fabric-Cluster mit einer Azure Resource Manager-Vorlage, mit PowerShell oder mit der CLI einen Knotentyp hinzufügen. In diesem Tutorial fügen wir mit Azure PowerShell einen Knotentyp hinzu.

Um einen neuen Knotentyp zu erstellen, müssen Sie drei Eigenschaften definieren:

* **Name des Knotentyps**: Der Name, der für alle vorhandenen Knotentypen im Cluster eindeutig ist.
* **Instanzenanzahl**: Die anfängliche Anzahl von Knoten des neuen Knotentyps.
* **VM-Größe**: Die VM-SKU für die Knoten. Wenn Sie hier nichts angeben, wird der Standardwert *Standard_D2* verwendet.

> [!NOTE]
> Wenn der hinzugefügte Knotentyp der erste oder einzige Knotentyp im Cluster ist, muss die Eigenschaft „Primary“ verwendet werden.

```powershell
$resourceGroup = "myResourceGroup"
$clusterName = "mysfcluster"
$nodeTypeName = "nt2"
$vmSize = "Standard_D2_v2"

New-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName -Name $nodeTypeName -InstanceCount 3 -vmSize $vmSize
```

## <a name="remove-a-node-type-from-a-service-fabric-managed-cluster"></a>Entfernen eines Knotentyps aus einem verwalteten Service Fabric-Cluster

Wenn Sie einen Knotentyp aus einem verwalteten Service Fabric-Cluster entfernen möchten, müssen Sie PowerShell oder die CLI verwenden. In diesem Tutorial entfernen wir mit Azure PowerShell einen Knotentyp.

> [!NOTE]
> Es ist nicht möglich, einen primären Knotentyp zu entfernen, wenn er der einzige primäre Knotentyp im Cluster ist.  

So entfernen Sie einen Knotentyp:

```powershell
$resourceGroup = "myResourceGroup"
$clusterName = "myCluster"
$nodeTypeName = "nt2"

Remove-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName  -Name $nodeTypeName
```

## <a name="next-steps"></a>Nächste Schritte

 In diesem Abschnitt haben wir Knotentypen hinzugefügt und gelöscht. Informationen zum Bereitstellen einer Anwendung in einem verwalteten Service Fabric-Cluster finden Sie im folgenden Artikel:

> [!div class="nextstepaction"]
> [Bereitstellen einer App in einem verwalteten Service Fabric-Cluster](tutorial-managed-cluster-deploy-app.md)
