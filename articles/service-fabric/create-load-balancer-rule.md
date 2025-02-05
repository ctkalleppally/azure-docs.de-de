---
title: Erstellen einer Azure Load Balancer-Regel für einen Cluster
description: Konfigurieren Sie eine Azure Load Balancer-Instanz zum Öffnen von Ports für Ihren Azure Service Fabric-Cluster.
ms.topic: conceptual
ms.date: 12/06/2017
ms.openlocfilehash: 7e09c7b0b3e2bfa5a5ff834e243f5098cbbd947b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/29/2021
ms.locfileid: "92319902"
---
# <a name="open-ports-for-a-service-fabric-cluster"></a>Öffnen von Port für einen Service Fabric-Cluster

Der mit Ihren Azure Service Fabric-Cluster bereitgestellte Lastenausgleich leitet Datenverkehr an Ihre auf einem Knoten ausgeführte App weiter. Wenn Sie Ihre Anwendung für die Verwendung eines anderen Ports ändern, müssen Sie diesen Port in Azure Load Balancer verfügbar machen (oder einen anderen Port weiterleiten).

Als Sie Ihren Service Fabric-Cluster in Azure bereitgestellt haben, wurde automatisch ein Lastenausgleich erstellt. Wenn Sie über keinen Lastenausgleich verfügen, finden Sie weitere Informationen unter [Konfigurieren eines internetseitigen Lastenausgleichs](../load-balancer/quickstart-load-balancer-standard-public-portal.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="configure-service-fabric"></a>Konfigurieren von Service Fabric

Die Konfigurationsdatei für die Service Fabric-Anwendung **ServiceManifest.xml** definiert die Endpunkte, die Ihre Anwendung erwartet. Nachdem die Konfigurationsdatei mit der Definition eines Endpunkts aktualisiert wurde, muss der Lastenausgleich aktualisiert werden, um diesen Port (oder einen anderen) verfügbar zu machen. Weitere Informationen zum Erstellen des Service Fabric-Endpunkts finden Sie unter [Einrichten eines Endpunkts](service-fabric-service-manifest-resources.md).

## <a name="create-a-load-balancer-rule"></a>Erstellen einer Load Balancer-Regel

Eine Lastenausgleichsregel öffnet einen internetseitigen Port und leitet Datenverkehr an den von Ihrer Anwendung verwendeten Port des internen Knotens weiter. Wenn Sie über keinen Lastenausgleich verfügen, finden Sie weitere Informationen unter [Konfigurieren eines internetseitigen Lastenausgleichs](../load-balancer/quickstart-load-balancer-standard-public-portal.md).

Um eine Lastenausgleichsregel zu erstellen, müssen Sie die folgenden Informationen sammeln:

- Name des Lastenausgleichs
- Ressourcengruppe des Lastenausgleichs und Service Fabric-Cluster
- Externer Port
- Interner Port

## <a name="azure-cli"></a>Azure-Befehlszeilenschnittstelle
Es ist nur ein einziger Befehl erforderlich, um mit der **Azure-Befehlszeilenschnittstelle** eine Lastenausgleichsregel zu erstellen. Sie müssen lediglich den Namen des Lastenausgleichs und die Ressourcengruppe kennen, um eine neue Regel zu erstellen.

>[!NOTE]
>Wenn Sie den Namen des Lastenausgleichs ermitteln müssen, verwenden Sie diesen Befehl, um schnell eine Liste aller Lastenausgleichsmodule und der zugehörigen Ressourcengruppen abzurufen.
>
>`az network lb list --query "[].{ResourceGroup: resourceGroup, Name: name}"`
>


```azurecli
az network lb rule create --backend-port 40000 --frontend-port 39999 --protocol Tcp --lb-name LB-svcfab3 -g svcfab_cli -n my-app-rule
```

Der Azure CLI-Befehl verfügt über einige Parameter, die in der folgenden Tabelle beschrieben werden:

| Parameter | BESCHREIBUNG |
| --------- | ----------- |
| `--backend-port`  | Port, an dem die Service Fabric-Anwendung lauscht |
| `--frontend-port` | Port, den der Lastenausgleich für externe Verbindungen verfügbar macht |
| `-lb-name` | Name des Lastenausgleichs, der geändert werden soll |
| `-g`       | Ressourcengruppe, in der sich der Lastenausgleich und der Service Fabric-Cluster befinden |
| `-n`       | Gewünschter Name der Regel |


>[!NOTE]
>Weitere Informationen zum Erstellen eines Lastenausgleichs mit der Azure-Befehlszeilenschnittstelle finden Sie unter [Erstellen eines Lastenausgleichs mit der Azure-Befehlszeilenschnittstelle](../load-balancer/quickstart-load-balancer-standard-internal-cli.md).

## <a name="powershell"></a>PowerShell

PowerShell ist etwas komplizierter als die Azure-Befehlszeilenschnittstelle. Führen Sie diese grundlegenden Schritte zum Erstellen einer Regel aus:

1. Rufen Sie den Lastenausgleich von Azure ab.
2. Erstellen Sie eine Regel.
3. Fügen Sie dem Lastenausgleich eine neue Regel hinzu.
4. Aktualisieren Sie den Lastenausgleich.

>[!NOTE]
>Wenn Sie den Namen des Lastenausgleichs ermitteln müssen, verwenden Sie diesen Befehl, um schnell eine Liste aller Lastenausgleichsmodule und der zugehörigen Ressourcengruppen abzurufen.
>
>`Get-AzLoadBalancer | Select Name, ResourceGroupName`

```powershell
# Get the load balancer
$lb = Get-AzLoadBalancer -Name LB-svcfab3 -ResourceGroupName svcfab_cli

# Create the rule based on information from the load balancer.
$lbrule = New-AzLoadBalancerRuleConfig -Name my-app-rule7 -Protocol Tcp -FrontendPort 39990 -BackendPort 40009 `
                                            -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
                                            -BackendAddressPool  $lb.BackendAddressPools[0] `
                                            -Probe $lb.Probes[0]

# Add the rule to the load balancer
$lb.LoadBalancingRules.Add($lbrule)

# Update the load balancer on Azure
$lb | Set-AzLoadBalancer
```

Im `New-AzLoadBalancerRuleConfig`-Befehl gibt `-FrontendPort` den Port an, den der Lastenausgleich für externe Verbindungen verfügbar macht, und `-BackendPort` gibt den Port an, an dem die Service Fabric-App lauscht.

>[!NOTE]
>Weitere Informationen zum Erstellen eines Lastenausgleichs mit PowerShell finden Sie unter [Erstellen eines Lastenausgleichs mit PowerShell](../load-balancer/quickstart-load-balancer-standard-internal-powershell.md).

## <a name="next-steps"></a>Nächste Schritte

Erfahren Sie mehr über [Netzwerke in Service Fabric](service-fabric-patterns-networking.md).