---
title: Diagnoseressourcenprotokollierung für eine Netzwerksicherheitsgruppe
titlesuffix: Azure Virtual Network
description: In diesem Artikel erfahren Sie, wie Sie Diagnoseressourcenprotokolle für Ereignis- und Regelzähler für eine Azure-Netzwerksicherheitsgruppe aktivieren.
services: virtual-network
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 06/04/2018
ms.author: kumud
ms.openlocfilehash: 0d171dee87a391c5e1d66db10363e6823ef387c1
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/20/2021
ms.locfileid: "107774159"
---
# <a name="resource-logging-for-a-network-security-group"></a>Ressourcenprotokollierung für eine Netzwerksicherheitsgruppe

Eine Netzwerksicherheitsgruppe (NSG) umfasst Regeln, die Datenverkehr an ein Subnetz oder eine Netzwerkschnittstelle eines virtuellen Netzwerks oder beides zulassen oder ablehnen. 

Wenn Sie die Protokollierung für eine NSG aktivieren, können Sie die folgenden Typen von Ressourcenprotokollinformationen erfassen:

* **Ereignis:** Es werden Einträge protokolliert, für die auf Grundlage der MAC-Adresse NSG-Regeln auf VMs angewendet werden.
* **Regelzähler:** Enthält Einträge darüber, wie oft jede NSG-Regel angewendet wurde, um Datenverkehr zuzulassen oder zu verweigern. Der Status für diese Regeln wird alle 300 Sekunden erfasst.

Ressourcenprotokolle sind nur für NSGs verfügbar, die über das Azure Resource Manager-Bereitstellungsmodell bereitgestellt wurden. Sie können die Ressourcenprotokollierung nicht für NSGs aktivieren, die über das klassische Bereitstellungsmodell bereitgestellt wurden. Zum besseren Verständnis der beiden Modelle lesen Sie den Artikel [Azure Resource Manager-Bereitstellung im Vergleich zur klassischen Bereitstellung: Grundlegendes zu Bereitstellungsmodellen und zum Status von Ressourcen](../azure-resource-manager/management/deployment-models.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

Die Ressourcenprotokollierung wird für *jede* NSG separat aktiviert, für die Sie Diagnosedaten sammeln möchten. Wenn Sie stattdessen an Aktivitätsprotokollen (Betriebsprotokollen) interessiert sind, lesen Sie die Informationen zur [Azure-Aktivitätsprotokollierung](../azure-monitor/essentials/platform-logs-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Informationen zu IP-Datenverkehr, der durch NSGs fließt, finden Sie unter Azure Network Watcher [NSG-Flussprotokolle](../network-watcher/network-watcher-nsg-flow-logging-overview.md) 

## <a name="enable-logging"></a>Aktivieren der Protokollierung

Die Ressourcenprotokollierung kann über das [Azure-Portal](#azure-portal), [PowerShell](#powershell) oder die [Azure-Befehlszeilenschnittstelle](#azure-cli) aktiviert werden.

### <a name="azure-portal"></a>Azure-Portal

1. Melden Sie sich beim [Portal](https://portal.azure.com) an.
2. Klicken Sie auf **Alle Dienste**, und geben Sie dann *Netzwerksicherheitsgruppen* ein. Wenn **Netzwerksicherheitsgruppen** in den Suchergebnissen angezeigt werden, wählen Sie diese aus.
3. Wählen Sie die NSG aus, für die Sie die Protokollierung aktivieren möchten.
4. Klicken Sie unter **ÜBERWACHUNG** auf die Option **Diagnoseprotokolle** und dann auf **Diagnose aktivieren**, wie in der folgenden Abbildung gezeigt wird:

   ![Aktivieren der Diagnose](./media/virtual-network-nsg-manage-log/turn-on-diagnostics.png)

5. Geben Sie unter **Diagnoseeinstellungen** die folgenden Informationen ein, oder wählen Sie sie aus, und klicken Sie dann auf **Speichern**:

    | Einstellung                                                                                     | Wert                                                          |
    | ---------                                                                                   |---------                                                       |
    | Name                                                                                        | Ein Name Ihrer Wahl.  Beispiel: *myNsgDiagnostics*      |
    | **In einem Speicherkonto archivieren**, **An einen Event Hub streamen** und **An Log Analytics senden** | Sie können beliebig viele Ziele auswählen. Weitere Informationen zu den einzelnen Zielen finden Sie unter [Protokollziele](#log-destinations).                                                                                                                                           |
    | PROTOKOLL                                                                                         | Wählen Sie eine oder beide Protokollkategorien aus. Weitere Informationen zu den Daten, die zu den einzelnen Kategorien protokolliert werden, finden Sie unter [Protokollkategorien](#log-categories).                                                                                                                                             |
6. Zeigen Sie Protokolle an, und analysieren Sie sie. Weitere Informationen finden Sie unter [Anzeigen und Analysieren von Protokollen](#view-and-analyze-logs).

### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Sie können die nachfolgenden Befehle in [Azure Cloud Shell](https://shell.azure.com/powershell) oder über PowerShell auf Ihrem Computer ausführen. Azure Cloud Shell ist eine kostenlose interaktive Shell. Sie verfügt über allgemeine vorinstallierte Tools und ist für die Verwendung mit Ihrem Konto konfiguriert. Wenn Sie PowerShell auf Ihrem Computer ausführen, müssen Sie das Azure PowerShell-Modul Version 1.0.0 oder höher ausführen. Führen Sie `Get-Module -ListAvailable Az` auf Ihrem Computer aus, um nach der installierten Version zu suchen. Wenn Sie ein Upgrade ausführen müssen, finden Sie unter [Installieren des Azure PowerShell-Moduls](/powershell/azure/install-az-ps) Informationen dazu. Wenn Sie PowerShell lokal ausführen, müssen Sie auch `Connect-AzAccount` ausführen, um sich bei Azure mit einem Konto anzumelden, das über die [erforderlichen Berechtigungen](virtual-network-network-interface.md#permissions) verfügt.

Um die Ressourcenprotokollierung zu aktivieren, benötigen Sie die ID einer vorhandenen NSG. Wenn noch keine NSG vorhanden ist, können Sie eine mit [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) erstellen.

Rufen Sie mit [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) die Netzwerksicherheitsgruppe ab, für die die Ressourcenprotokollierung aktiviert werden soll. Um z.B. eine NSG namens *myNsg* abzurufen, die sich in einer Ressourcengruppe mit dem Namen *myResourceGroup* befindet, geben Sie den folgenden Befehl ein:

```azurepowershell-interactive
$Nsg=Get-AzNetworkSecurityGroup `
  -Name myNsg `
  -ResourceGroupName myResourceGroup
```

Sie können Ressourcenprotokolle in drei Zieltypen schreiben. Weitere Informationen finden Sie unter [Protokollziele](#log-destinations). In diesem Artikel werden Protokolle beispielhaft an das Ziel *Log Analytics* gesendet. Rufen Sie mit [Get-AzOperationalInsightsWorkspace](/powershell/module/az.operationalinsights/get-azoperationalinsightsworkspace) einen vorhandenen Log Analytics-Arbeitsbereich ab. Geben Sie beispielsweise den folgenden Befehl ein, wenn Sie einen vorhandenen Arbeitsbereich mit dem Namen *myWorkspace* in einer Ressourcengruppe mit dem Namen *myWorkspaces* abrufen möchten:

```azurepowershell-interactive
$Oms=Get-AzOperationalInsightsWorkspace `
  -ResourceGroupName myWorkspaces `
  -Name myWorkspace
```

Falls kein Arbeitsbereich vorhanden ist, können Sie einen mit [New-AzOperationalInsightsWorkspace](/powershell/module/az.operationalinsights/new-azoperationalinsightsworkspace) erstellen.

Es gibt zwei Protokollierungskategorien, für die Sie Protokolle aktivieren können. Weitere Informationen finden Sie unter [Protokollkategorien](#log-categories). Aktivieren Sie mit [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) die Ressourcenprotokollierung für die NSG. Im folgenden Beispiel werden mithilfe der IDs für die NSG und des Arbeitsbereichs, den Sie zuvor abgerufen haben, Ereignis- und Zählerkategoriedaten für den Arbeitsbereich einer NSG protokolliert:

```azurepowershell-interactive
Set-AzDiagnosticSetting `
  -ResourceId $Nsg.Id `
  -WorkspaceId $Oms.ResourceId `
  -Enabled $true
```

Wenn Sie Daten nur für eine der Kategorien protokollieren möchten anstatt für beide Kategorien, fügen Sie die Option `-Categories` zum vorherigen Befehl hinzu, gefolgt von *NetworkSecurityGroupEvent* oder *NetworkSecurityGroupRuleCounter*. Wenn Sie Daten in einem anderen [Ziel](#log-destinations) als einem Log Analytics-Arbeitsbereich protokollieren möchten, verwenden Sie die entsprechenden Parameter für ein [Azure Storage-Konto](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-storage) oder für [Event Hub](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-event-hubs).

Zeigen Sie Protokolle an, und analysieren Sie sie. Weitere Informationen finden Sie unter [Anzeigen und Analysieren von Protokollen](#view-and-analyze-logs).

### <a name="azure-cli"></a>Azure CLI

Sie können die nachfolgenden Befehle in [Azure Cloud Shell](https://shell.azure.com/bash) oder über die Azure-Befehlszeilenschnittstelle auf Ihrem Computer ausführen. Azure Cloud Shell ist eine kostenlose interaktive Shell. Sie verfügt über allgemeine vorinstallierte Tools und ist für die Verwendung mit Ihrem Konto konfiguriert. Wenn Sie die Befehlszeilenschnittstelle über Ihren Computer ausführen, ist mindestens Version 2.0.38 erforderlich. Führen Sie `az --version` auf Ihrem Computer aus, um nach der installierten Version zu suchen. Wenn Sie ein Upgrade ausführen müssen, finden Sie unter [Installieren der Azure-Befehlszeilenschnittstelle](/cli/azure/install-azure-cli) weitere Informationen. Wenn Sie die Befehlszeilenschnittstelle lokal ausführen, müssen Sie auch `az login` ausführen, um sich bei Azure mit einem Konto anzumelden, das über die [erforderlichen Berechtigungen](virtual-network-network-interface.md#permissions) verfügt.

Um die Ressourcenprotokollierung zu aktivieren, benötigen Sie die ID einer vorhandenen NSG. Wenn noch keine NSG vorhanden ist, können Sie mit dem Befehl [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create) eine erstellen.

Rufen Sie mit dem Befehl [az network nsg show](/cli/azure/network/nsg#az_network_nsg_show) die Netzwerksicherheitsgruppe ab, für die die Ressourcenprotokollierung aktiviert werden soll. Um z.B. eine NSG namens *myNsg* abzurufen, die sich in einer Ressourcengruppe mit dem Namen *myResourceGroup* befindet, geben Sie den folgenden Befehl ein:

```azurecli-interactive
nsgId=$(az network nsg show \
  --name myNsg \
  --resource-group myResourceGroup \
  --query id \
  --output tsv)
```

Sie können Ressourcenprotokolle in drei Zieltypen schreiben. Weitere Informationen finden Sie unter [Protokollziele](#log-destinations). In diesem Artikel werden Protokolle beispielhaft an das Ziel *Log Analytics* gesendet. Weitere Informationen finden Sie unter [Protokollkategorien](#log-categories).

Aktivieren Sie mit dem Befehl [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az_monitor_diagnostic_settings_create) die Ressourcenprotokollierung für die NSG. Im folgenden Beispiel werden Daten für die Ereignis- und die Indikatorkategorie in einem vorhandenen Arbeitsbereich namens *myWorkspace* protokolliert. Dieser ist in einer Ressourcengruppe namens *myWorkspaces* vorhanden, und die zuvor abgerufene ID der NSG lautet:

```azurecli-interactive
az monitor diagnostic-settings create \
  --name myNsgDiagnostics \
  --resource $nsgId \
  --logs '[ { "category": "NetworkSecurityGroupEvent", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } }, { "category": "NetworkSecurityGroupRuleCounter", "enabled": true, "retentionPolicy": { "days": 30, "enabled": true } } ]' \
  --workspace myWorkspace \
  --resource-group myWorkspaces
```

Wenn Sie über keinen vorhandenen Arbeitsbereich verfügen, können Sie über das [Azure-Portal](../azure-monitor/logs/quick-create-workspace.md?toc=%2fazure%2fvirtual-network%2ftoc.json) oder über [PowerShell](/powershell/module/az.operationalinsights/new-azoperationalinsightsworkspace) einen erstellen. Es gibt zwei Protokollierungskategorien, für die Sie Protokolle aktivieren können.

Wenn Sie Daten nur für die eine oder die andere Kategorie protokollieren möchten, entfernen Sie im vorherigen Befehl die Kategorie, für die Sie keine Daten protokollieren möchten. Wenn Sie Daten in einem anderen [Ziel](#log-destinations) als einem Log Analytics-Arbeitsbereich protokollieren möchten, verwenden Sie die entsprechenden Parameter für ein [Azure Storage-Konto](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-storage) oder für [Event Hub](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-event-hubs).

Zeigen Sie Protokolle an, und analysieren Sie sie. Weitere Informationen finden Sie unter [Anzeigen und Analysieren von Protokollen](#view-and-analyze-logs).

## <a name="log-destinations"></a>Protokollziele

Mit Diagnosedaten können Sie folgende Vorgänge durchführen:
- [Schreiben von Diagnosedaten in ein Azure Storage-Konto](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-storage) zur Überwachung oder manuellen Überprüfung. Mithilfe der Diagnoseeinstellungen für Ressourcen können Sie eine Aufbewahrungsdauer (in Tagen) angeben.
- [Streamen von Diagnosedaten an einen Event Hub](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-event-hubs) zur Erfassung durch einen Drittanbieterdienst oder durch eine benutzerdefinierte Analyselösung wie Power BI
- [Geschrieben in Azure Monitor-Protokolle](../azure-monitor/essentials/resource-logs.md?toc=%2fazure%2fvirtual-network%2ftoc.json#send-to-azure-storage)

## <a name="log-categories"></a>Protokollkategorien

JSON-formatierte Daten werden für die folgenden Protokollkategorien geschrieben:

### <a name="event"></a>Ereignis

Dieses Ereignisprotokoll enthält Informationen darüber, welche NSG-Regeln basierend auf der MAC-Adresse für VMs gelten. Die folgenden Daten werden für jedes Ereignis protokolliert. Im folgenden Beispiel werden die Daten für einen virtuellen Computer mit der IP-Adresse „192.168.1.4“ und der MAC-Adresse „00-0D-3A-92-6A-7C“ protokolliert:

```json
{
    "time": "[DATE-TIME]",
    "systemId": "[ID]",
    "category": "NetworkSecurityGroupEvent",
    "resourceId": "/SUBSCRIPTIONS/[SUBSCRIPTION-ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG-NAME]",
    "operationName": "NetworkSecurityGroupEvents",
    "properties": {
        "vnetResourceGuid":"[ID]",
        "subnetPrefix":"192.168.1.0/24",
        "macAddress":"00-0D-3A-92-6A-7C",
        "primaryIPv4Address":"192.168.1.4",
        "ruleName":"[SECURITY-RULE-NAME]",
        "direction":"[DIRECTION-SPECIFIED-IN-RULE]",
        "priority":"[PRIORITY-SPECIFIED-IN-RULE]",
        "type":"[ALLOW-OR-DENY-AS-SPECIFIED-IN-RULE]",
        "conditions":{
            "protocols":"[PROTOCOLS-SPECIFIED-IN-RULE]",
            "destinationPortRange":"[PORT-RANGE-SPECIFIED-IN-RULE]",
            "sourcePortRange":"[PORT-RANGE-SPECIFIED-IN-RULE]",
            "sourceIP":"[SOURCE-IP-OR-RANGE-SPECIFIED-IN-RULE]",
            "destinationIP":"[DESTINATION-IP-OR-RANGE-SPECIFIED-IN-RULE]"
            }
        }
}
```

### <a name="rule-counter"></a>Regelzähler

Das Regelzählerprotokoll enthält Informationen über jede Regel, die auf Ressourcen angewendet wird. Die folgenden Beispieldaten werden jedes Mal protokolliert, wenn eine Regel angewendet wird. Im folgenden Beispiel werden die Daten für einen virtuellen Computer mit der IP-Adresse „192.168.1.4“ und der MAC-Adresse „00-0D-3A-92-6A-7C“ protokolliert:

```json
{
    "time": "[DATE-TIME]",
    "systemId": "[ID]",
    "category": "NetworkSecurityGroupRuleCounter",
    "resourceId": "/SUBSCRIPTIONS/[SUBSCRIPTION ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG-NAME]",
    "operationName": "NetworkSecurityGroupCounters",
    "properties": {
        "vnetResourceGuid":"[ID]",
        "subnetPrefix":"192.168.1.0/24",
        "macAddress":"00-0D-3A-92-6A-7C",
        "primaryIPv4Address":"192.168.1.4",
        "ruleName":"[SECURITY-RULE-NAME]",
        "direction":"[DIRECTION-SPECIFIED-IN-RULE]",
        "type":"[ALLOW-OR-DENY-AS-SPECIFIED-IN-RULE]",
        "matchedConnections":125
        }
}
```

> [!NOTE]
> Die Quell-IP-Adresse für die Kommunikation wird nicht protokolliert. Sie können die [NSG-Flussprotokollierung](../network-watcher/network-watcher-nsg-flow-logging-portal.md) für eine NSG aktivieren, wobei jedoch alle Regelzählerinformationen sowie die Quell-IP-Adresse, die die Kommunikation initiiert, protokolliert werden. NSG-Flussprotokolldaten werden in einem Azure Storage-Konto gespeichert. Sie können die Daten mit der Funktion [Traffic Analytics](../network-watcher/traffic-analytics.md) von Azure Network Watcher analysieren.

## <a name="view-and-analyze-logs"></a>Anzeigen und Analysieren von Protokollen

Informationen zum Anzeigen von Ressourcenprotokolldaten finden Sie unter [Übersicht über die Azure-Plattformprotokolle](../azure-monitor/essentials/platform-logs-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Beim Senden von Diagnosedaten in folgenden Lösungen sollten Sie Folgendes wissen:
- **Azure Monitor-Protokolle:** Mit der [Analyselösung für Netzwerksicherheitsgruppen](../azure-monitor/insights/azure-networking-analytics.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-network-security-group-analytics-solution-in-azure-monitor
) können Sie bessere Erkenntnisse erzielen. Die Lösung bietet Visualisierungen für NSG-Regeln, die Datenverkehr mittels der MAC-Adresse der Netzwerkschnittstelle auf einem virtuellen Computer zulassen oder ablehnen.
- **Azure Storage-Konto**: Daten werden in eine Datei namens „PT1H.json“ geschrieben. Folgende Informationen finden Sie in den angegebenen Speicherorten:
  - Ereignisprotokoll unter folgendem Pfad: `insights-logs-networksecuritygroupevent/resourceId=/SUBSCRIPTIONS/[ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME-FOR-NSG]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG NAME]/y=[YEAR]/m=[MONTH/d=[DAY]/h=[HOUR]/m=[MINUTE]`
  - Regelzählerprotokoll unter folgendem Pfad: `insights-logs-networksecuritygrouprulecounter/resourceId=/SUBSCRIPTIONS/[ID]/RESOURCEGROUPS/[RESOURCE-GROUP-NAME-FOR-NSG]/PROVIDERS/MICROSOFT.NETWORK/NETWORKSECURITYGROUPS/[NSG NAME]/y=[YEAR]/m=[MONTH/d=[DAY]/h=[HOUR]/m=[MINUTE]`

## <a name="next-steps"></a>Nächste Schritte

- Erfahren Sie mehr über die [Aktivitätsprotokollierung](../azure-monitor/essentials/platform-logs-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Die Aktivitätsprotokollierung ist für alle NSGs standardmäßig aktiviert, unabhängig davon, in welchem Azure-Bereitstellungsmodell sie erstellt wurden. Um zu bestimmen, welche Vorgänge für Netzwerksicherheitsgruppen im Aktivitätsprotokoll abgeschlossen wurden, suchen Sie nach Einträgen, die die folgenden Ressourcentypen enthalten:
  - Microsoft.ClassicNetwork/networkSecurityGroups
  - Microsoft.ClassicNetwork/networkSecurityGroups/securityRules
  - Microsoft.Network/networkSecurityGroups
  - Microsoft.Network/networkSecurityGroups/securityRules
- Informationen zum Protokollieren von Diagnoseinformationen zur Berücksichtigung der Quell-IP-Adresse für jeden Datenfluss finden Sie unter [NSG-Flussprotokollierung](../network-watcher/network-watcher-nsg-flow-logging-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
