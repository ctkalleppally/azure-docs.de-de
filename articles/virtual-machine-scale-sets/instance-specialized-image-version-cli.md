---
title: Erstellen einer Skalierungsgruppe aus einer spezialisierten Imageversion mithilfe der Azure CLI
description: Erstellen Sie eine Skalierungsgruppe mithilfe einer spezialisierten Imageversion in einer Shared Image Gallery mithilfe der Azure CLI.
author: cynthn
ms.service: virtual-machine-scale-sets
ms.subservice: shared-image-gallery
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/01/2020
ms.author: cynthn
ms.reviewer: mimckitt
ms.custom: devx-track-azurecli
ms.openlocfilehash: c65f8c3ffcf8e48153701c8d4d2b316c2b41f375
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/06/2021
ms.locfileid: "108768461"
---
# <a name="create-a-scale-set-using-a-specialized-image-version-with-the-azure-cli"></a>Erstellen einer Skalierungsgruppe mithilfe einer spezialisierten Imageversion mithilfe der Azure CLI

Erstellen Sie eine Skalierungsgruppe aus einer [spezialisierten Imageversion](../virtual-machines/shared-image-galleries.md#generalized-and-specialized-images), die in einer Shared Image Gallery gespeichert wird. Wenn Sie eine Skalierungsgruppe mit einer generalisierten Imageversion erstellen möchten, finden Sie weitere Informationen unter [Erstellen einer Skalierungsgruppe aus einem generalisierten Image](instance-generalized-image-version-cli.md).

Wenn Sie die CLI lokal installieren und verwenden möchten, müssen Sie für dieses Tutorial Version 2.4.0 oder höher der Azure CLI ausführen. Führen Sie `az --version` aus, um die Version zu ermitteln. Informationen zum Durchführen einer Installation oder eines Upgrades finden Sie bei Bedarf unter [Installieren der Azure CLI]( /cli/azure/install-azure-cli).

Ersetzen Sie bei Bedarf die Ressourcennamen in diesem Beispiel. 

Listen Sie die Imagedefinitionen in einem Katalog mithilfe von [az sig image-definition list](/cli/azure/sig/image-definition#az_sig_image_definition_list) auf, um den Namen und die ID der Definitionen anzuzeigen.

```azurecli-interactive 
resourceGroup=myGalleryRG
gallery=myGallery
az sig image-definition list \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --query "[].[name, id]" \
   --output tsv
```

Erstellen Sie eine Skalierungsgruppe mit [`az vmss create`](/cli/azure/vmss#az_vmss_create) mithilfe des `--specialized`-Parameters, um anzugeben, dass das Image ein spezialisiertes Image ist.

Verwenden Sie die Imagedefinitions-ID für `--image`, um die Skalierungsgruppeninstanzen auf Grundlage der neuesten verfügbaren Imageversion zu erstellen. Sie können die Skalierungsgruppeninstanzen auch auf Grundlage einer bestimmten Version erstellen, indem Sie die Imageversions-ID für `--image` verwenden. Beachten Sie, dass die Verwendung einer bestimmten Imageversion bedeutet, dass die Automatisierung fehlschlagen kann, wenn diese bestimmte Imageversion nicht verfügbar ist, weil sie gelöscht oder aus der Region entfernt wurde. Es wird empfohlen, die Imagedefinitions-ID zum Erstellen der neuen VM zu verwenden, es sei denn, eine bestimmte Imageversion ist erforderlich.

In diesem Beispiel erstellen Sie Instanzen auf Grundlage der neuesten Version des *myImageDefinition*-Images.

```azurecli
az group create --name myResourceGroup --location eastus
az vmss create \
   --resource-group myResourceGroup \
   --name myScaleSet \
   --image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition" \
   --specialized
```


## <a name="next-steps"></a>Nächste Schritte
[Azure Image Builder (Vorschauversion)](../virtual-machines/image-builder-overview.md) hilft beim Automatisieren der Erstellung von Imageversionen. Sie können den Dienst sogar zum Aktualisieren und [Erstellen einer neuen Imageversion aus einer vorhandenen](../virtual-machines/linux/image-builder-gallery-update-image-version.md) verwenden. 

Sie können auch mithilfe von Vorlagen eine Ressource im Katalog für freigegebene Images erstellen. Es stehen mehrere Azure-Schnellstartvorlagen zur Verfügung: 

- [Erstellen eines Katalogs mit freigegebenen Images](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [Erstellen einer Imagedefinition in einem Katalog mit freigegebenen Images](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [Erstellen einer Imageversion in einem Katalog mit freigegebenen Images](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)