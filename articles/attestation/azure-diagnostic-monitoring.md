---
title: Azure-Diagnoseüberwachung für Azure Attestation
description: Azure-Diagnoseüberwachung für Azure Attestation
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: overview
ms.date: 08/31/2020
ms.author: mbaldwin
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 851557c4cdc0f913247d40f2aaea3230a2e8551f
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/28/2021
ms.locfileid: "108204727"
---
# <a name="set-up-diagnostics-with-a-trusted-platform-module-tpm-endpoint-of-azure-attestation"></a>Einrichten der Diagnose mit einem TPM-Endpunkt (Trusted Platform Module) von Azure Attestation

Dieser Artikel unterstützt Sie beim Erstellen und Konfigurieren von Diagnoseeinstellungen, um Plattformmetriken und -protokolle an verschiedene Ziele zu senden. [Plattformprotokolle](../azure-monitor/essentials/platform-logs-overview.md) in Azure, z. B. das Azure-Aktivitätsprotokoll und Ressourcenprotokolle, liefern ausführliche Diagnose- und Überwachungsinformationen für Azure-Ressourcen und die Azure-Plattform, von der sie abhängen. [Plattformmetriken](../azure-monitor/essentials/data-platform-metrics.md) werden standardmäßig gesammelt und in der Azure Monitor-Metrikdatenbank gespeichert.

Stellen Sie zunächst sicher, dass Sie [Azure Attestation mit Azure PowerShell eingerichtet haben](quickstart-powershell.md).

Der TMP-Endpunktdienst (Trusted Platform Module) wird in den Diagnoseeinstellungen aktiviert und kann zum Überwachen der Aktivität verwendet werden. Richten Sie die [Azure-Überwachung](../azure-monitor/overview.md) für den TPM-Dienstendpunkt mithilfe des folgenden Codes ein:

```powershell

 Connect-AzAccount 

 Set-AzContext -Subscription <Subscription id> 

 $attestationProviderName=<Name of the attestation provider> 

 $attestationResourceGroup=<Name of the resource Group> 

 $attestationProvider=Get-AzAttestation -Name $attestationProviderName -ResourceGroupName $attestationResourceGroup 

 $storageAccount=New-AzStorageAccount -ResourceGroupName $attestationProvider.ResourceGroupName -Name <Storage Account Name> -SkuName Standard_LRS -Location <Location> 

 Set-AzDiagnosticSetting -ResourceId $ attestationProvider.Id -StorageAccountId $ storageAccount.Id -Enabled $true 

```

Aktivitätsprotokolle befinden sich im Abschnitt **Container** des Speicherkontos. Weitere Informationen finden Sie unter [Tutorial: Sammeln und Analysieren von Ressourcenprotokollen von einer Azure-Ressource](../azure-monitor/essentials/tutorial-resource-logs.md).