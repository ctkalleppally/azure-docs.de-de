---
title: 'Virtual WAN: Erstellen einer Routingtabelle für den virtuellen Hub in NVA: Azure PowerShell'
description: Informationen zu einer Routingtabelle für einen virtuellen Hub für Virtual WAN zum Steuern des Datenverkehrs zu einem virtuellen Netzwerkgerät.
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: cherylmc
ms.openlocfilehash: 6a6e701377956e39696567eff9a6a0abca927b88
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 03/31/2021
ms.locfileid: "106055075"
---
# <a name="create-a-virtual-hub-route-table-to-steer-traffic-to-a-network-virtual-appliance"></a>Erstellen einer Routingtabelle für einen virtuellen Hub für Virtual WAN zum Steuern des Datenverkehrs zu einem virtuellen Netzwerkgerät

In diesem Artikel wird veranschaulicht, wie Sie Datenverkehr von einem virtuellen Hub zu einem virtuellen Netzwerkgerät steuern. 

![Virtual WAN-Diagramm](./media/virtual-wan-route-table-nva/vwanroute.png)

In diesem Artikel wird Folgendes behandelt:

* Erstellen eines WAN
* Erstellen eines Hubs
* Erstellen von Netzwerkverbindungen mit einem virtuellen Hub
* Erstellen einer Hubroute
* Erstellen einer Routingtabelle
* Anwenden der Routingtabelle

## <a name="before-you-begin"></a>Voraussetzungen

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Vergewissern Sie sich, dass die folgenden Kriterien erfüllt sind:

* Sie verfügen über ein virtuelles Netzwerkgerät. Dies ist eine Drittanbietersoftware Ihrer Wahl, die in der Regel über Azure Marketplace in einem virtuellen Netzwerk bereitgestellt wird.
* Sie haben der Netzwerkschnittstelle des virtuellen Netzwerkgeräts eine private IP-Adresse zugewiesen. 
* Das virtuelle Netzwerkgerät kann nicht im virtuellen Hub bereitgestellt werden. Es muss in einem gesonderten VNET bereitgestellt werden. In diesem Artikel wird das NVA-VNET als „DMZ VNET“ bezeichnet.
* Mit „DMZ VNET“ kann mindestens ein virtuelles Netzwerk verbunden sein. In diesem Artikel wird dieses VNET als „Indirektes Spoke-VNET“ bezeichnet. Diese VNETs können mithilfe von VNET-Peering mit DMZ VNET verbunden werden.
* Bestätigen Sie, dass Sie bereits 2 VNETs erstellt haben. Diese werden als Spoke-VNETs verwendet. In diesem Artikel sind die VNET-Spoke-Adressräume 10.0.2.0/24 und 10.0.3.0/24. Informationen zum Erstellen eine VNET finden Sie unter [Erstellen eines virtuellen Netzwerks mit PowerShell](../virtual-network/quick-create-powershell.md).
* Stellen Sie sicher, dass in den VNETs keine Gateways für virtuelle Netzwerke vorhanden sind.

## <a name="1-sign-in"></a><a name="signin"></a>1. Anmelden

Installieren Sie die neueste Version der PowerShell-Cmdlets für Resource Manager. Weitere Informationen zum Installieren von PowerShell-Cmdlets finden Sie unter [Overview of Azure PowerShell](/powershell/azure/install-az-ps) (Übersicht über Azure PowerShell). Dies ist wichtig, da frühere Versionen der Cmdlets nicht die aktuellen Werte enthalten, die Sie für diese Übung benötigen.

1. Öffnen Sie die PowerShell-Konsole mit erhöhten Rechten, und melden Sie sich bei Ihrem Azure-Konto an. Das Cmdlet fordert Sie zur Eingabe der Anmeldeinformationen auf. Nach der Anmeldung werden Ihre Kontoeinstellungen heruntergeladen, damit sie Azure PowerShell zur Verfügung stehen.

   ```powershell
   Connect-AzAccount
   ```
2. Rufen Sie eine Liste Ihrer Azure-Abonnements ab.

   ```powershell
   Get-AzSubscription
   ```
3. Geben Sie das Abonnement an, das Sie verwenden möchten.

   ```powershell
   Select-AzSubscription -SubscriptionName "Name of subscription"
   ```

## <a name="2-create-resources"></a><a name="rg"></a>2. Erstellen von Ressourcen

1. Erstellen Sie eine Ressourcengruppe.

   ```powershell
   New-AzResourceGroup -Location "West US" -Name "testRG"
   ```
2. Erstellen Sie ein virtuelles WAN.

   ```powershell
   $virtualWan = New-AzVirtualWan -ResourceGroupName "testRG" -Name "myVirtualWAN" -Location "West US"
   ```
3. Erstellen Sie einen virtuellen Hub.

   ```powershell
   New-AzVirtualHub -VirtualWan $virtualWan -ResourceGroupName "testRG" -Name "westushub" -AddressPrefix "10.0.1.0/24" -Location "West US"
   ```

## <a name="3-create-connections"></a><a name="connections"></a>3. Erstellen von Verbindungen

Erstellen von Netzwerkverbindungen mit einem virtuellen Hub aus „Indirektes Spoke-VNET“ und „DMZ VNET“.

  ```powershell
  $remoteVirtualNetwork1= Get-AzVirtualNetwork -Name "indirectspoke1" -ResourceGroupName "testRG"
  $remoteVirtualNetwork2= Get-AzVirtualNetwork -Name "indirectspoke2" -ResourceGroupName "testRG"
  $remoteVirtualNetwork3= Get-AzVirtualNetwork -Name "dmzvnet" -ResourceGroupName "testRG"

  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "westushub" -Name  "testvnetconnection1" -RemoteVirtualNetwork $remoteVirtualNetwork1
  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "westushub" -Name  "testvnetconnection2" -RemoteVirtualNetwork $remoteVirtualNetwork2
  New-AzVirtualHubVnetConnection -ResourceGroupName "testRG" -VirtualHubName "westushub" -Name  "testvnetconnection3" -RemoteVirtualNetwork $remoteVirtualNetwork3
  ```

## <a name="4-create-a-virtual-hub-route"></a><a name="route"></a>4. Erstellen einer virtuellen Hubroute

In diesem Artikel sind die Adressräume für „Indirektes Spoke-VNET“ 10.0.2.0/24 und 10.0.3.0/24. Die private IP-Adresse der Netzwerkschnittstelle von „DMZ VNET“ ist 10.0.4.5.

```powershell
$route1 = New-AzVirtualHubRoute -AddressPrefix @("10.0.2.0/24", "10.0.3.0/24") -NextHopIpAddress "10.0.4.5"
```

## <a name="5-create-a-virtual-hub-route-table"></a><a name="applyroute"></a>5. Erstellen einer Routingtabelle für den virtuellen Hub

Erstellen Sie eine Routingtabelle für den virtuellen Hub, und wenden Sie dann die erstellte Route darauf an.
 
```powershell
$routeTable = New-AzVirtualHubRouteTable -Route @($route1)
```

## <a name="6-commit-the-changes"></a><a name="commit"></a>6. Übergeben der Änderungen

Führen Sie für die Änderungen am virtuellen Hub einen Commit aus.

```powershell
Update-AzVirtualHub -ResourceGroupName "testRG" -Name "westushub" -RouteTable $routeTable
```

## <a name="next-steps"></a>Nächste Schritte

Weitere Informationen zu Virtual WAN finden Sie auf der Seite mit der [Übersicht über Virtual WAN](virtual-wan-about.md).
