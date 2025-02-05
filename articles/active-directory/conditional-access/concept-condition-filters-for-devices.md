---
title: Filter für Geräte als Bedingung in der Richtlinie für bedingten Zugriff – Azure Active Directory
description: Verwenden von Gerätefiltern beim bedingten Zugriff zur Verbesserung des Sicherheitsstatus
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 05/19/2021
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: f0eb008cdf85e6891f7d6e1159314ceccf46b794
ms.sourcegitcommit: 2ab484c6c8aeef8c6b7a1eb56d28ba13010d912c
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 05/20/2021
ms.locfileid: "110208645"
---
# <a name="conditional-access-filters-for-devices-preview"></a>Bedingter Zugriff: Filter für Geräte (Vorschau)

Beim Erstellen von Richtlinien für bedingten Zugriff haben Administratoren nach einer Möglichkeit gesucht, bestimmte Geräte in ihrer Umgebung als Ziel anzugeben oder auszuschließen. Die Bedingung „Filter für Geräte“ (Vorschau) bietet Administratoren diese Möglichkeit. Jetzt können Sie mithilfe von [unterstützten Operatoren und Geräteeigenschaften für Filter](#supported-operators-and-device-properties-for-filters) und den anderen verfügbaren Zuweisungsbedingungen in Ihren Richtlinien für bedingten Zugriff auf bestimmte Geräte abzielen.

:::image type="content" source="media/concept-condition-filters-for-devices/create-filter-for-devices-condition.png" alt-text="Erstellen der Bedingung „Filter für Geräte“ in einer Richtlinie für bedingten Zugriff":::

> [!IMPORTANT]
> Die Bedingung „Filter für Geräte“ befindet sich derzeit in der öffentlichen Vorschau. Diese Vorschauversion wird ohne Vereinbarung zum Servicelevel bereitgestellt und ist nicht für Produktionsworkloads vorgesehen. Manche Features werden möglicherweise nicht unterstützt oder sind nur eingeschränkt verwendbar. Weitere Informationen finden Sie unter [Zusätzliche Nutzungsbestimmungen für Microsoft Azure-Vorschauen](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="common-scenarios"></a>Häufige Szenarien

Es gibt mehrere Szenarien, bei denen Organisationen jetzt die Bedingung „Filter für Geräte“ aktivieren und verwenden können. Im Folgenden finden Sie einige Schlüsselszenarien mit Beispielen für die Verwendung dieser neuen Bedingung.

- Beschränken des Zugriffs auf privilegierte Ressourcen wie Microsoft Azure Management auf privilegierte Benutzer, die von [privilegierten oder sicheren Administratorarbeitsstationen (Secure Admin Workstations, SAW)](/security/compass/privileged-access-devices) aus zugreifen. Für dieses Szenario würden Organisationen zwei Richtlinien für bedingten Zugriff erstellen:
   - Richtlinie 1: Alle Benutzer mit der Verzeichnisrolle „Globaler Administrator“, die auf die Microsoft Azure Management-Cloud-App zugreifen, sowie „Zugriff gewähren“ als Zugriffssteuerung. Erforderlich sind jedoch eine mehrstufige Authentifizierung (MFA) und ein als kompatibel markiertes Gerät.
   - Richtlinie 2: Alle Benutzer mit der Verzeichnisrolle „Globaler Administrator“, die auf die Microsoft Azure Management-Cloud-App zugreifen, mit Ausnahme der mit „Filter für Geräte“ und dem Regelausdruck „device.extensionAttribute1 NotEquals SAW“ gefilterten Geräte, für die „Blockieren“ als Zugriffssteuerung festgelegt wird.
- Blockieren des Zugriffs auf Organisationsressourcen von Geräten, auf denen eine nicht unterstützte Betriebssystemversion wie Windows 7 ausgeführt wird. Für dieses Szenario würden Organisationen die beiden folgenden Richtlinien für bedingten Zugriff erstellen:
   - Richtlinie 1: Alle Benutzer, die auf alle Cloud-Apps zugreifen, sowie „Zugriff gewähren“ als Zugriffssteuerung. Erforderlich ist jedoch ein als kompatibel markiertes Gerät oder ein hybrid in Azure AD eingebundenes Gerät.
   - Richtlinie 2: Alle Benutzer, die auf alle Cloud-Apps zugreifen, einschließlich der mit „Filter für Geräte“ und dem Regelausdruck „device.operatingSystem Equals Windows“ und „device.operatingSystemVersion startsWith 6.1“ gefilterten Geräte, für die „Blockieren“ als Zugriffssteuerung festgelegt wird.
- Für bestimmte Konten (z. B. Dienstkonten) soll bei Verwendung auf bestimmten Geräten (z. B. Teams-Smartphones oder Surface Hub-Geräten) keine mehrstufige Authentifizierung (MFA) erforderlich sein. Für dieses Szenario würden Organisationen die beiden folgenden Richtlinien für bedingten Zugriff erstellen:
   - Richtlinie 1: Alle Benutzer mit Ausnahme von Dienstkonten, die auf alle Cloud-Apps zugreifen, sowie „Zugriff gewähren“ als Zugriffssteuerung. Erforderlich ist jedoch eine mehrstufige Authentifizierung (MFA).
   - Richtlinie 2: Wählen Sie Benutzer und Gruppen aus, und schließen Sie eine Gruppe ein, die nur Dienstkonten enthält, die auf alle Cloud-Apps zugreifen, mit Ausnahme der mit „Filter für Geräte“ und dem Regelausdruck „device.extensionAttribute2 NotEquals TeamsPhoneDevice“ gefilterten Geräte, für die „Blockieren“ als Zugriffssteuerung festgelegt wird.

## <a name="create-a-conditional-access-policy"></a>Erstellen der Richtlinie für bedingten Zugriff

Die Bedingung „Filter für Geräte“ ist eine Option, wenn Sie eine Richtlinie für bedingten Zugriff im Azure-Portal oder mit der Microsoft Graph-API erstellen.

> [!IMPORTANT]
> „Gerätestatus“ und „Filter für Geräte“ können in der Richtlinie für bedingten Zugriff nicht zusammen verwendet werden. „Filter für Geräte“ bietet eine präzisere Zielgruppenadressierung sowie Unterstützung für das Ansteuern von Gerätestatusinformationen über die Eigenschaften `trustType` und `isCompliant`.

Mit den folgenden Schritten können Sie zwei Richtlinien für bedingten Zugriff erstellen, um das erste unter [Häufige Szenarien](#common-scenarios) beschriebene Szenario zu unterstützen. 

Richtlinie 1: Alle Benutzer mit der Verzeichnisrolle „Globaler Administrator“, die auf die Microsoft Azure Management-Cloud-App zugreifen, sowie „Zugriff gewähren“ als Zugriffssteuerung. Erforderlich sind jedoch eine mehrstufige Authentifizierung (MFA) und ein als kompatibel markiertes Gerät.

1. Melden Sie sich beim **Azure-Portal** als globaler Administrator, Sicherheitsadministrator oder Administrator für bedingten Zugriff an.
1. Navigieren Sie zu **Azure Active Directory** > **Sicherheit** > **Bedingter Zugriff**.
1. Wählen Sie **Neue Richtlinie**.
1. Benennen Sie Ihre Richtlinie. Es wird empfohlen, dass Unternehmen einen aussagekräftigen Standard für die Namen ihrer Richtlinien erstellen.
1. Klicken Sie unter **Zuweisungen** auf **Benutzer und Gruppen**.
   1. Wählen Sie unter **Einschließen** die Option **Verzeichnisrollen**  aus, und wählen Sie dann **Globaler Administrator** aus.
   
      > [!WARNING]
      > Richtlinien für bedingten Zugriff unterstützen integrierte Rollen. Richtlinien für bedingten Zugriff werden nicht für andere Rollentypen erzwungen, einschließlich [Rollen einer administrativen Einheit](../roles/admin-units-assign-roles.md) oder [benutzerdefinierten Rollen](../roles/custom-create.md).

   1. Wählen Sie unter **Ausschließen** die Option **Benutzer und Gruppen** und dann die Konten für den Notfallzugriff Ihres Unternehmens aus. 
   1. Wählen Sie **Fertig** aus.
1. Wählen Sie unter **Cloud-Apps oder -Aktionen** die Option  > **Einschließen** aus, wählen Sie **Apps auswählen** aus, und wählen Sie dann **Microsoft Azure Management**  aus.
1. Wählen Sie unter **Zugriffssteuerungen** > **Gewähren** die Optionen **Zugriff gewähren**, **Mehrstufige Authentifizierung erforderlich** und **Markieren des Geräts als kompatibel erforderlich** aus, und wählen Sie dann **Auswählen** aus.
1. Bestätigen Sie die Einstellungen und legen Sie **Richtlinie aktivieren** auf **Ein** fest.
1. Wählen Sie **Erstellen** aus, um die Richtlinie zu erstellen und zu aktivieren.

Richtlinie 2: Alle Benutzer mit der Verzeichnisrolle „Globaler Administrator“, die auf die Microsoft Azure Management-Cloud-App zugreifen, mit Ausnahme der mit „Filter für Geräte“ und dem Regelausdruck „device.extensionAttribute1 NotEquals SAW“ gefilterten Geräte, für die „Blockieren“ als Zugriffssteuerung festgelegt wird.

1. Wählen Sie **Neue Richtlinie**.
1. Benennen Sie Ihre Richtlinie. Es wird empfohlen, dass Unternehmen einen aussagekräftigen Standard für die Namen ihrer Richtlinien erstellen.
1. Klicken Sie unter **Zuweisungen** auf **Benutzer und Gruppen**.
   1. Wählen Sie unter **Einschließen** die Option **Verzeichnisrollen**  aus, und wählen Sie dann **Globaler Administrator** aus.
   
      > [!WARNING]
      > Richtlinien für bedingten Zugriff unterstützen integrierte Rollen. Richtlinien für bedingten Zugriff werden nicht für andere Rollentypen erzwungen, einschließlich [Rollen einer administrativen Einheit](../roles/admin-units-assign-roles.md) oder [benutzerdefinierten Rollen](../roles/custom-create.md).

   1. Wählen Sie unter **Ausschließen** die Option **Benutzer und Gruppen** und dann die Konten für den Notfallzugriff Ihres Unternehmens aus. 
   1. Wählen Sie **Fertig** aus.
1. Wählen Sie unter **Cloud-Apps oder -Aktionen** die Option  > **Einschließen** aus, wählen Sie **Apps auswählen** aus, und wählen Sie dann **Microsoft Azure Management**  aus.
1. Wechseln Sie unter **Bedingungen** zu **Filter für Geräte (Vorschau)** .
   1. Legen Sie **Konfigurieren** auf **Ja** fest.
   1. Legen Sie **Geräte, die der Regel entsprechen** auf **Gefilterte Geräte von der Richtlinie ausschließen** fest.
   1. Legen Sie die Eigenschaft auf `ExtensionAttribute1`, den Operator auf `Equals` und den Wert auf `SAW` fest.
   1. Wählen Sie **Fertig** aus.
1. Wählen Sie unter **Zugriffssteuerung** > **Erteilen** die Option **Zugriff blockieren** und dann **Auswählen** aus.
1. Bestätigen Sie die Einstellungen und legen Sie **Richtlinie aktivieren** auf **Ein** fest.
1. Wählen Sie **Erstellen** aus, um die Richtlinie zu erstellen und zu aktivieren.

### <a name="filters-for-devices-graph-api"></a>Graph-API „Filter für Geräte“

Die API „Filter für Geräte“ ist derzeit im Beta-Endpunkt von Microsoft Graph verfügbar und kann über https://graph.microsoft.com/beta/identity/conditionalaccess/policies/ aufgerufen werden. Sie können Filter für Geräte konfigurieren, wenn Sie eine neue Richtlinie für bedingten Zugriff erstellen. Sie können aber auch eine vorhandene Richtlinie aktualisieren, um Filter für Geräte als Bedingung zu konfigurieren. Um eine vorhandene Richtlinie zu aktualisieren, können Sie einen Patchaufruf an den oben erwähnten Beta-Endpunkt von Microsoft Graph senden, indem Sie die Richtlinien-ID einer vorhandenen Richtlinie anfügen und den folgenden Anforderungstext ausführen. Im hier gezeigten Beispiel wird das Konfigurieren einer Bedingung „Filter für Geräte“ veranschaulicht, durch die Geräte ausgeschlossen werden, die nicht als SAW-Geräte markiert sind. Die Regelsyntax kann aus mehreren Ausdrücken bestehen. Weitere Informationen zur Syntax finden Sie unter den Regeln mit mehreren Ausdrücken. 

```json
{
    "conditions": {
        "devices": {
            "deviceFilter": {
                "mode": "exclude",
                "rule": "device.extensionAttribute1 -ne \"SAW\""
            }
        }
    }
}
```

## <a name="supported-operators-and-device-properties-for-filters"></a>Unterstützte Operatoren und Geräteeigenschaften für Filter

Die folgenden Geräteattribute können mit der Bedingung „Filter für Geräte“ für den bedingten Zugriff verwendet werden.

| Unterstützte Geräteattribute | Unterstützte Operatoren | Unterstützte Werte | Beispiel |
| --- | --- | --- | --- |
| deviceId | Equals, NotEquals, In, NotIn | Gültige „deviceId“, die eine GUID ist | (device.deviceid -eq “498c4de7-1aee-4ded-8d5d-000000000000”) |
| displayName | Equals, NotEquals, StartsWith, NotStartsWith, EndsWith, NotEndsWith, Contains, NotContains, In, NotIn | Beliebige Zeichenfolge | (device.displayName -contains „ABC“) |
| Hersteller | Equals, NotEquals, StartsWith, NotStartsWith, EndsWith, NotEndsWith, Contains, NotContains, In, NotIn | Beliebige Zeichenfolge | (device.manufacturer -startsWith „Microsoft“) |
| mdmAppId | Equals, NotEquals, In, NotIn | Gültige MDM-Anwendungs-ID | (device.mdmAppId -in [“0000000a-0000-0000-c000-000000000000“] |
| model | Equals, NotEquals, StartsWith, NotStartsWith, EndsWith, NotEndsWith, Contains, NotContains, In, NotIn | Beliebige Zeichenfolge | (device.model -notContains „Surface“) |
| operatingSystem | Equals, NotEquals, StartsWith, NotStartsWith, EndsWith, NotEndsWith, Contains, NotContains, In, NotIn | Gültiges Betriebssystem (z. B. Windows, iOS oder Android) | (device.operatingSystem -eq „Windows“) |
| operatingSystemVersion | Equals, NotEquals, StartsWith, NotStartsWith, EndsWith, NotEndsWith, Contains, NotContains, In, NotIn | Gültige Betriebssystemversion (z. B. 6.1 für Windows 7, 6.2 für Windows 8 oder 10.0 für Windows 10) | (device.operatingSystemVersion -in [„10.0.18363“, „10.0.19041”, „10.0.19042”]) |
| pyhsicalIds | Contains, NotContains | Beispiel: Alle Windows Autopilot-Geräte speichern ZTDId (eindeutiger Wert, der allen importierten Windows Autopilot-Geräten zugewiesen ist) in der PhysicalIds-Eigenschaft des Geräts. | (device.devicePhysicalIDs -contains „[ZTDId]“) |
| profileType | Equals, NotEquals | Gültiger Profiltyp, der für ein Gerät festgelegt ist. Unterstützte Werte: RegisteredDevice (Standard), SecureVM (für virtuelle Windows-Computer, die in Azure mit Azure AD-Anmeldung aktiviert sind), Printer (für Drucker), Shared (für freigegebene Geräte), IoT (für IoT-Geräte) | (device.profileType -notIn [„Printer“, „Shared“, „IoT“] |
| systemLabels | Contains, NotContains | Liste der Bezeichnungen, die vom System auf das Gerät angewendet werden. Einige unterstützte Werte: AzureResource (für virtuelle Windows-Computer, die in Azure mit Azure AD-Anmeldung aktiviert sind), M365Managed (für mit Microsoft Managed Desktop verwaltete Geräte), MultiUser (für freigegebene Geräte) | (device.systemLabels -contains "M365Managed") |
| trustType | Equals, NotEquals | Gültiger registrierter Status für Geräte. Unterstützte Werte: AzureAD (für in Azure AD eingebundene Geräte), ServerAD (für hybrid in Azure AD eingebundene Geräte), Workplace (für in Azure AD registrierte Geräte) | (device.trustType -notIn „ServerAD, Workplace“) |
| extensionAttribute1-15 | Equals, NotEquals, StartsWith, NotStartsWith, EndsWith, NotEndsWith, Contains, NotContains, In, NotIn | extensionAttributes1-15 sind Attribute, die Kunden für Geräteobjekte verwenden können. Kunden können alle Attribute des Typs „extensionAttributes1-15“ mit benutzerdefinierten Werten aktualisieren und sie in der Bedingung „Filter für Geräte“ für den bedingten Zugriff verwenden. Es kann ein beliebiger Zeichenfolgewert verwendet werden. | (device.extensionAttribute1 -eq „SAW“) |

## <a name="policy-behavior-with-filters-for-devices"></a>Verhalten von Richtlinien mit „Filter für Geräte“

Die Bedingung „Filter für Geräte“ (Vorschau) für den bedingten Zugriff wertet richtlinienbasiert die Geräteattribute eines in Azure AD registrierten Geräts aus. Daher ist es wichtig zu verstehen, unter welchen Umständen die Richtlinie angewendet bzw. nicht angewendet wird. In der folgenden Tabelle wird das Verhalten veranschaulicht, wenn die Bedingung „Filter für Geräte“ konfiguriert ist. 

| Bedingung „Filter für Geräte“ | Geräteregistrierungsstatus | Angewendeter Gerätefilter 
| --- | --- | --- |
| Include-/Exclude-Modus mit positiven Operatoren (Equals, StartsWith, EndsWith, Contains, In) und Verwendung beliebiger Attribute | Nicht registriertes Gerät | Nein |
| Include-/Exclude-Modus mit positiven Operatoren (Equals, StartsWith, EndsWith, Contains, In) und Verwendung von Attributen mit Ausnahme von extensionAttributes1-15 | Registriertes Gerät | Ja, wenn die Kriterien erfüllt sind |
| Include-/Exclude-Modus mit positiven Operatoren (Equals, StartsWith, EndsWith, Contains, In) und Verwendung von Attributen einschließlich extensionAttributes1-15 | Registriertes Gerät, das von Intune verwaltet wird | Ja, wenn die Kriterien erfüllt sind |
| Include-/Exclude-Modus mit positiven Operatoren (Equals, StartsWith, EndsWith, Contains, In) und Verwendung von Attributen einschließlich extensionAttributes1-15 | Registriertes Gerät, das nicht von Intune verwaltet wird | Ja, wenn die Kriterien erfüllt sind und das Gerät kompatibel oder hybrid in Azure AD eingebunden ist |
| Include-/Exclude-Modus mit negativen Operatoren (NotEquals, NotStartsWith, NotEndsWith, NotContains, NotIn) und Verwendung beliebiger Attribute | Nicht registriertes Gerät | Ja |
| Include-/Exclude-Modus mit negativen Operatoren (NotEquals, NotStartsWith, NotEndsWith, NotContains, NotIn) und Verwendung beliebiger Attribute mit Ausnahme von extensionAttributes1-15 | Registriertes Gerät | Ja, wenn die Kriterien erfüllt sind |
| Include-/Exclude-Modus mit negativen Operatoren (NotEquals, NotStartsWith, NotEndsWith, NotContains, NotIn) und Verwendung beliebiger Attribute einschließlich extensionAttributes1-15 | Registriertes Gerät, das von Intune verwaltet wird | Ja, wenn die Kriterien erfüllt sind |
| Include-/Exclude-Modus mit negativen Operatoren (NotEquals, NotStartsWith, NotEndsWith, NotContains, NotIn) und Verwendung beliebiger Attribute einschließlich extensionAttributes1-15 | Registriertes Gerät, das nicht von Intune verwaltet wird | Ja, wenn die Kriterien erfüllt sind und das Gerät kompatibel oder hybrid in Azure AD eingebunden ist |

## <a name="next-steps"></a>Nächste Schritte

- [Bedingter Zugriff: Bedingungen](concept-conditional-access-conditions.md)
- [Allgemeine Richtlinien für bedingten Zugriff](concept-conditional-access-policy-common.md)
- [Schützen von Geräten im Rahmen der Geschichte des privilegierten Zugriffs](/security/compass/privileged-access-devices)
