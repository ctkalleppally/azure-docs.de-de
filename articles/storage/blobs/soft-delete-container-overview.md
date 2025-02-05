---
title: Vorläufiges Löschen für Container (Vorschau)
titleSuffix: Azure Storage
description: Vorläufiges Löschen für Container (Vorschau) schützt Ihre Daten, damit Sie sie leichter wiederherstellen können, wenn sie irrtümlich von einer Anwendung oder einem anderen Speicherkontobenutzer geändert oder gelöscht wurden.
services: storage
author: tamram
ms.service: storage
ms.topic: conceptual
ms.date: 03/05/2021
ms.author: tamram
ms.subservice: blobs
ms.custom: references_regions
ms.openlocfilehash: 2efd673d26231e83d820f7971a740d06e9b2a1d2
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/20/2021
ms.locfileid: "107785413"
---
# <a name="soft-delete-for-containers-preview"></a>Vorläufiges Löschen für Container (Vorschau)

Vorläufiges Löschen für Container (Vorschau) schützt Ihre Daten vor versehentlichem oder böswilligen Löschen. Wenn das vorläufige Löschen von Containern für ein Speicherkonto aktiviert ist, wird ein gelöschter Container und dessen Inhalte in Azure Storage für den von Ihnen festgelegten Zeitraum aufbewahrt. Während dieses Aufbewahrungszeitraums können Sie zuvor gelöschte Container wiederherstellen. Beim Wiederherstellen eines Containers werden alle Blobs wiederhergestellt, die sich in diesem Container befanden, als er gelöscht wurde.

Microsoft empfiehlt die Aktivierung der folgenden Datenschutzfeatures, um den vollständigen Schutz für Ihre Blobdaten zu gewährleisten:

- Vorläufiges Löschen für Container, um einen gelöschten Container wiederherzustellen. Informationen zum Aktivieren des vorläufigen Löschens für Container finden Sie unter [Aktivieren und Verwalten des vorläufigen Löschens für Container (Vorschau)](soft-delete-container-enable.md).
- Blobversionsverwaltung zum automatischen Verwalten früherer Versionen eines Blobs. Wenn Blobversionsverwaltung aktiviert ist, können Sie eine frühere Version eines Blobs wiederherstellen, um Daten wiederherzustellen, wenn diese irrtümlich geändert oder gelöscht wurden. Informationen zum Aktivieren der Blobversionsverwaltung finden Sie unter [Aktivieren und Verwalten der Blobversionsverwaltung (Vorschau)](versioning-enable.md).
- Vorläufiges Löschen von Blobs, um einen Blob oder eine Version wiederherzustellen, das/die gelöscht wurde. Informationen zum Aktivieren des vorläufigen Löschens von Blobs finden Sie unter [Aktivieren und Verwalten des vorläufigen Löschens von Blobs](soft-delete-blob-enable.md).

> [!IMPORTANT]
> Das vorläufige Löschen von Containern befindet sich zurzeit in der **VORSCHAU**. Die [zusätzlichen Nutzungsbestimmungen für Microsoft Azure-Vorschauen](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) enthalten rechtliche Bedingungen. Sie gelten für diejenigen Azure-Features, die sich in der Beta- oder Vorschauversion befinden oder aber anderweitig noch nicht zur allgemeinen Verfügbarkeit freigegeben sind.

## <a name="how-container-soft-delete-works"></a>Funktionsweise des vorläufigen Löschens von Containern

Wenn Sie das vorläufige Löschen von Containern aktivieren, können Sie einen Aufbewahrungszeitraum für gelöschte Container zwischen 1 und 365 Tagen angeben. Der Standardaufbewahrungszeitraum beträgt sieben Tage. Während des Aufbewahrungszeitraums können Sie einen gelöschten Container wiederherstellen, indem Sie den Vorgang **Restore Container** (Container wiederherstellen) aufrufen.

Wenn Sie einen Container wiederherstellen, werden auch die Blobs im Container und alle Blobversionen wiederhergestellt. Sie können das vorläufige Löschen von Containern jedoch nur dann zur Wiederherstellung von Blobs verwenden, wenn der Container selbst gelöscht wurde. Um einen gelöschten Blob wiederherzustellen, wenn sein übergeordneter Container nicht gelöscht wurde, müssen Sie mit dem vorläufigen Löschen oder der Versionsverwaltung von Blobs arbeiten.

> [!WARNING]
> Beim vorläufigen Löschen von Containern können nur vollständige Container und deren Inhalte wiederhergestellt werden, die zum Zeitpunkt der Löschung in diesen enthalten waren. Es ist nicht möglich, mithilfe des vorläufigen Löschens von Containern ein gelöschtes Blob in einem Container wiederherzustellen. Microsoft empfiehlt außerdem das Aktivieren von vorläufigem Löschen von Blobs und Blobversionierung zum Schutz einzelner Blobs in einem Container.

Das folgende Diagramm zeigt, wie ein gelöschter Container wiederhergestellt werden kann, wenn das vorläufige Löschen von Containern aktiviert ist:

:::image type="content" source="media/soft-delete-container-overview/container-soft-delete-diagram.png" alt-text="Diagramm der Wiederherstellung eines vorläufig gelöschten Containers":::

Bei der Wiederherstellung eines Containers können Sie diesen mit dem ursprünglichen Namen wiederherstellen, wenn dieser Name nicht wiederverwendet wurde. Wenn der ursprüngliche Containername bereits verwendet wird, können Sie den Container mit einem neuen Namen wiederherstellen.

Nach Ablauf des Aufbewahrungszeitraums wird der Container endgültig aus Azure Storage gelöscht und kann nicht wiederhergestellt werden. Der Aufbewahrungszeitraum beginnt zum Zeitpunkt der Löschung des Containers. Sie können den Aufbewahrungszeitraum jederzeit ändern. Denken Sie jedoch daran, dass ein aktualisierter Aufbewahrungszeitraum nur für neu gelöschte Container gilt. Zuvor gelöschte Container werden basierend auf dem Aufbewahrungszeitraum endgültig gelöscht, der zum Zeitpunkt der Löschung dieser Container festgelegt war.

Das Deaktivieren des vorläufigen Löschens von Containern führt nicht zur endgültigen Löschung der Container, die zuvor vorläufig gelöscht wurden. Vorläufig gelöschte Container werden nach Ablauf des Aufbewahrungszeitraums endgültig gelöscht, der zum Zeitpunkt der Löschung dieser Container festgelegt war.

> [!IMPORTANT]
> Vorläufiges Löschen eines Containers schützt nicht vor dem Löschen eines Speicherkontos. Es schützt nur vor dem Löschen von Containern in diesem Konto. Um ein Speicherkonto vor dem Löschen zu schützen, konfigurieren Sie eine Sperre für die Speicherkontoressource. Weitere Informationen zum Sperren eines Speicherkontos finden Sie unter [Anwenden einer Azure Resource Manager-Sperre auf einem Speicherkonto](../common/lock-account-resource.md).

## <a name="about-the-preview"></a>Informationen zur Vorschau

Das vorläufige Löschen von Containern ist in allen Azure-Regionen in der Vorschauversion verfügbar.

Version 2019-12-12 oder höher der Azure Storage-REST-API unterstützt das vorläufige Löschen von Containern.

### <a name="storage-account-support"></a>Speicherkontounterstützung

Das vorläufige Löschen von Containern ist für die folgenden Typen von Speicherkonten verfügbar:

- Speicherkonten des Typs „Universell v2“ und „Universell v1“
- Blockblob-Speicherkonten
- Blob-Speicherkonten

Speicherkonten mit einem hierarchischen Namespace, die für die Verwendung mit Azure Data Lake Storage Gen2 aktiviert sind, werden ebenfalls unterstützt.

### <a name="register-for-the-preview"></a>Registrieren für die Vorschau

Registrieren Sie sich für die Vorschauversion des vorläufigen Löschens von Containern, indem Sie über PowerShell oder die Azure CLI eine Anforderung zum Registrieren des Features für Ihr Abonnement senden. Wenn die Anforderung genehmigt wurde, können Sie das vorläufige Löschen von Containern mit allen neuen oder bereits vorhandenen Speicherkonten vom Typ „Universell v2“ sowie mit Blob Storage-Konten oder Blockblob-Speicherkonten mit dem Tarif „Premium“ aktivieren.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Rufen Sie für die Registrierung mit PowerShell den Befehl [Register-AzProviderFeature](/powershell/module/az.resources/register-azproviderfeature) auf.

```powershell
# Register for container soft delete (preview)
Register-AzProviderFeature -ProviderNamespace Microsoft.Storage `
    -FeatureName ContainerSoftDelete

# Refresh the Azure Storage provider namespace
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```

# <a name="azure-cli"></a>[Azure-Befehlszeilenschnittstelle](#tab/azure-cli)

Zum Registrieren bei Azure CLI rufen Sie den Befehl „[az feature register](/cli/azure/feature#az_feature_register)“ auf.

```azurecli
az feature register --namespace Microsoft.Storage --name ContainerSoftDelete
az provider register --namespace 'Microsoft.Storage'
```

---

### <a name="check-the-status-of-your-registration"></a>Überprüfen des Registrierungsstatus

Verwenden Sie PowerShell oder die Azure CLI, um den Status Ihrer Registrierung zu überprüfen.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Um den Status Ihrer Registrierung mit PowerShell zu überprüfen, rufen Sie den Befehl [Get-AzProviderFeature](/powershell/module/az.resources/get-azproviderfeature) auf.

```powershell
Get-AzProviderFeature -ProviderNamespace Microsoft.Storage -FeatureName ContainerSoftDelete
```

# <a name="azure-cli"></a>[Azure-Befehlszeilenschnittstelle](#tab/azure-cli)

Um den Status Ihrer Registrierung über die Azure-Befehlszeilenschnittstelle zu überprüfen, rufen Sie den Befehl [az feature](/cli/azure/feature#az_feature_show) auf.

```azurecli
az feature show --namespace Microsoft.Storage --name ContainerSoftDelete
```

---

## <a name="pricing-and-billing"></a>Preise und Abrechnung

Für das Aktivieren des vorläufigen Löschens von Containern fallen keine zusätzlichen Kosten an. Daten in vorläufig gelöschten Containern werden zum selben Tarif wie aktive Daten in Rechnung gestellt.

## <a name="next-steps"></a>Nächste Schritte

- [Aktivieren und Verwalten des vorläufigen Löschens für Container (Vorschau)](soft-delete-container-enable.md)
- [Vorläufiges Löschen für Blobs](soft-delete-blob-overview.md)
- [Blobversionsverwaltung](versioning-overview.md)
