---
title: Abrufen eines Signaturschlüssels aus einer Richtlinie – .NET
description: In diesem Thema wird gezeigt, wie Sie mit Media Services V3 .NET SDK einen Signaturschlüssel aus der vorhandenen Richtlinie abrufen.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18
ms.openlocfilehash: 71d4b00d3ebb56d72cbb16cd42394c720f29849f
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/03/2021
ms.locfileid: "106277639"
---
# <a name="get-a-signing-key-from-the-existing-policy"></a>Abrufen eines Signaturschlüssels aus der vorhandenen Richtlinie

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Eines der wichtigsten Entwurfsprinzipien der v3-API ist es, die API sicherer zu machen. v3-APIs geben bei einem **Get**- oder **List**-Vorgang keine geheimen Schlüssel oder Anmeldeinformationen zurück. Die ausführliche Erläuterung finden Sie hier: Weitere Informationen finden Sie unter [Azure RBAC und Media Services-Konten](security-rbac-concept.md).

Anhand des Beispiels in diesem Artikel wird veranschaulicht, wie .NET verwendet wird, um einen Signaturschlüssel aus der vorhandenen Richtlinie abzurufen. 
 
## <a name="download"></a>Download 

Klonen Sie ein GitHub-Repository auf Ihren Computer, das das vollständige .NET-Beispiel enthält, indem Sie den folgenden Befehl verwenden:  

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git
 ```
 
Die Richtlinie „ContentKeyPolicy“ mit Beispiel für Geheimnisse befindet sich im Ordner [EncryptWithDRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/EncryptWithDRM).

## <a name="get-contentkeypolicy-with-secrets"></a>Abrufen von „ContentKeyPolicy“ mit Geheimnissen 

Um den Schlüssel abzurufen, verwenden Sie **GetPolicyPropertiesWithSecretsAsync**, wie im folgenden Beispiel gezeigt.

[!code-csharp[Main](../../../media-services-v3-dotnet-tutorials/AMSV3Tutorials/EncryptWithDRM/Program.cs#GetOrCreateContentKeyPolicy)]

## <a name="next-steps"></a>Nächste Schritte

[Entwurf eines Multi-DRM-Inhaltsschutzsystems mit Zugriffssteuerung](architecture-design-multi-drm-system.md) 
