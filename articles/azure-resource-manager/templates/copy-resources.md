---
title: Bereitstellen mehrerer Instanzen von Ressourcen
description: Verwenden des copy-Vorgangs und von Arrays in einer Azure Resource Manager-Vorlage (ARM), um einen Ressourcentyp mehrere Male bereitzustellen.
ms.topic: conceptual
ms.date: 04/01/2021
ms.openlocfilehash: 3af676cce544c125e441857f06556b9ff7eee697
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/05/2021
ms.locfileid: "106385706"
---
# <a name="resource-iteration-in-arm-templates"></a>Ressourceniteration in ARM-Vorlagen

In diesem Artikel erfahren Sie, wie Sie in Ihrer Azure Resource Manager-Vorlage (ARM) mehrere Instanzen einer Ressource erstellen. Durch das Hinzufügen der copy-Schleife zum Ressourcenabschnitt Ihrer Vorlage können Sie die Anzahl der bereitzustellenden Ressourcen dynamisch festlegen. Außerdem vermeiden Sie so die Wiederholung von Vorlagensyntax.

Die copy-Schleife kann auch mit [Eigenschaften](copy-properties.md), [Variablen](copy-variables.md) und [Ausgaben](copy-outputs.md) verwendet werden.

Wenn Sie angeben müssen, ob eine Ressource überhaupt bereitgestellt wird, finden Sie die erforderlichen Informationen unter [Element „condition“](conditional-resource-deployment.md).

## <a name="syntax"></a>Syntax

# <a name="json"></a>[JSON](#tab/json)

Fügen Sie das `copy`-Element dem Ressourcenabschnitt ihrer Vorlage hinzu, um mehrere Instanzen der Ressource bereitzustellen. Das `copy`-Element hat das folgende allgemeine Format:

```json
"copy": {
  "name": "<name-of-loop>",
  "count": <number-of-iterations>,
  "mode": "serial" <or> "parallel",
  "batchSize": <number-to-deploy-serially>
}
```

Die Eigenschaft `name` ist ein beliebiger Wert, der die Schleife identifiziert. Die Eigenschaft `count` gibt die für den Ressourcentyp gewünschte Anzahl von Iterationen an.

Verwenden Sie die Eigenschaften `mode` und `batchSize`, um anzugeben, ob die Ressourcen parallel oder nacheinander bereitgestellt werden. Diese Eigenschaften werden unter [Seriell oder parallel](#serial-or-parallel) beschrieben.

# <a name="bicep"></a>[Bicep](#tab/bicep)

Mit Schleifen können mehrere Ressourcen deklariert werden durch:

- Durchlaufen eines Arrays:

  ```bicep
  @batchSize(<number>)
  resource <resource-symbolic-name> '<resource-type>@<api-version>' = [for <item> in <collection>: {
    <resource-properties>
  }
  ```

- Durchlaufen der Elemente eines Arrays

  ```bicep
  @batchSize(<number>)
  resource <resource-symbolic-name> '<resource-type>@<api-version>' = [for (<item>, <index>) in <collection>: {
    <resource-properties>
  }
  ```

- Verwenden des Schleifenindexes

  ```bicep
  @batchSize(<number>)
  resource <resource-symbolic-name> '<resource-type>@<api-version>' = [for <index> in range(<start>, <stop>): {
    <resource-properties>
  }
  ```

---

## <a name="copy-limits"></a>Einschränkungen für „copy“

Der Wert von „count“ darf 800 nicht überschreiten.

Der Wert von „count“ darf nicht negativ sein. Er kann Null sein, wenn Sie die Vorlage mit einer neueren Version von Azure CLI, PowerShell oder der REST-API bereitstellen. Insbesondere müssen Sie Folgendes verwenden:

- Azure PowerShell **2.6** oder höher
- Azure CLI **2.0.74** oder höher
- REST-API-Version **2019-05-10** oder höher
- [Verknüpfte Bereitstellungen](linked-templates.md) müssen API-Version **2019-05-10** oder höher für den Bereitstellungsressourcentyp verwenden.

Frühere Versionen von PowerShell, CLI und der REST-API unterstützen den Wert „0“ (null) für „count“ nicht.

Wenden Sie die [Bereitstellung im vollständigen Modus](deployment-modes.md) mit der copy-Schleife mit Vorsicht an. Wenn Sie mit dem vollständigen Modus erneut in einer Ressourcengruppe bereitstellen, werden alle Ressourcen, die nicht in der Vorlage angegeben sind, nach dem Auflösen der Kopierschleife gelöscht.

## <a name="resource-iteration"></a>Ressourceniteration

Im folgenden Beispiel wird die Anzahl von Speicherkonten erstellt, die im Parameter `storageCount` angegeben ist.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageCount": {
      "type": "int",
      "defaultValue": 2
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {},
      "copy": {
        "name": "storagecopy",
        "count": "[parameters('storageCount')]"
      }
    }
  ]
}
```

Beachten Sie, dass der Name der einzelnen Ressourcen die `copyIndex()`-Funktion enthält, die die aktuelle Iteration in der Schleife zurückgibt. `copyIndex()` ist nullbasiert. Im folgenden Beispiel werden die unten aufgeführten Namen erstellt:

```json
"name": "[concat('storage', copyIndex())]",
```

Namen:

- storage0
- storage1
- storage2

Zum Versetzen des Indexwerts können Sie einen Wert in der `copyIndex()`-Funktion übergeben. Die Anzahl von Iterationen wird weiterhin im copy-Element angegeben, aber der Wert von `copyIndex` wird um den angegebenen Wert versetzt. Im folgenden Beispiel werden die unten aufgeführten Namen erstellt:

```json
"name": "[concat('storage', copyIndex(1))]",
```

Namen:

- storage1
- storage2
- storage3

Der „copy“-Vorgang ist besonders bei Verwendung von Arrays hilfreich, weil Sie jedes Element im Array durchlaufen können. Verwenden Sie die Funktion `length` für das Array, um die Anzahl von Iterationen anzugeben, und `copyIndex`, um den aktuellen Index im Array abzurufen.

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param storageCount int = 2

resource storage_id 'Microsoft.Storage/storageAccounts@2019-04-01' = [for i in range(0, storageCount): {
  name: '${i}storage${uniqueString(resourceGroup().id)}'
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
  properties: {}
}]
```

Beachten Sie, dass der Index `i` zum Erstellen des Ressourcennamens für das Speicherkonto verwendet wird.

---

Im folgenden Beispiel wird ein Speicherkonto für jeden im Parameter angegebenen Namen erstellt.

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "storageNames": {
          "type": "array",
          "defaultValue": [
            "contoso",
            "fabrikam",
            "coho"
          ]
      }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(parameters('storageNames')[copyIndex()], uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "properties": {},
      "copy": {
        "name": "storagecopy",
        "count": "[length(parameters('storageNames'))]"
      }
    }
  ],
  "outputs": {}
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
param storageNames array = [
  'contoso'
  'fabrikam'
  'coho'
]

resource storageNames_id 'Microsoft.Storage/storageAccounts@2019-04-01' = [for name in storageNames: {
  name: concat(name, uniqueString(resourceGroup().id))
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
  properties: {}
}]
```

---

Wenn Sie Werte von den bereitgestellten Ressourcen zurückgeben möchten, können Sie [„copy“ im Ausgabenabschnitt](copy-outputs.md) verwenden.

## <a name="serial-or-parallel"></a>Seriell oder parallel

Resource Manager erstellt die Ressourcen standardmäßig gleichzeitig. Es gilt keine Beschränkung für die Anzahl der parallel bereitgestellten Ressourcen, mit Ausnahme der Begrenzung der Gesamtanzahl auf 800 Ressourcen in der Vorlage. Die Reihenfolge, in der sie erstellt werden, ist nicht garantiert.

Es ist aber möglicherweise sinnvoll, anzugeben, dass die Ressource sequenziell bereitgestellt werden. Wenn Sie z.B. eine Produktionsumgebung aktualisieren, möchten Sie die Updates möglicherweise staffeln, sodass nur eine bestimmte Anzahl von Ressourcen gleichzeitig aktualisiert wird.

Zum seriellen Bereitstellen von zwei Speicherkonten gleichzeitig verwenden Sie beispielsweise:

# <a name="json"></a>[JSON](#tab/json)

Legen Sie zum seriellen Bereitstellen mehrerer Instanzen einer Ressource `mode` auf **serial** und `batchSize` auf die Anzahl der Instanzen fest, die zu einem Zeitpunkt bereitgestellt werden sollen. Im seriellen Modus erstellt Resource Manager eine Abhängigkeit von früheren Instanzen in der Schleife, sodass ein Batch erst dann gestartet wird, wenn der vorherige Batch abgeschlossen wurde.

Der Wert für `batchSize` kann den Wert für `count` im copy-Element nicht überschreiten.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2019-04-01",
      "name": "[concat(copyIndex(),'storage', uniqueString(resourceGroup().id))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "Storage",
      "copy": {
        "name": "storagecopy",
        "count": 4,
        "mode": "serial",
        "batchSize": 2
      },
      "properties": {}
    }
  ],
  "outputs": {}
}
```

Die `mode`-Eigenschaft akzeptiert auch **parallel**, wobei es sich um den Standardwert handelt.

# <a name="bicep"></a>[Bicep](#tab/bicep)

Legen Sie zum seriellen Bereitstellen mehrerer Instanzen einer Ressource den [Decorator](./bicep-file.md#resource-and-module-decorators) `batchSize` auf die Anzahl der Instanzen fest, die zu einem Zeitpunkt bereitgestellt werden sollen. Im seriellen Modus erstellt Resource Manager eine Abhängigkeit von früheren Instanzen in der Schleife, sodass ein Batch erst dann gestartet wird, wenn der vorherige Batch abgeschlossen wurde.

```bicep
@batchSize(2)
resource storage_id 'Microsoft.Storage/storageAccounts@2019-04-01' = [for i in range(0, 4): {
  name: '${i}storage${uniqueString(resourceGroup().id)}'
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'Storage'
  properties: {}
}]
```

---

## <a name="iteration-for-a-child-resource"></a>Iteration für eine untergeordnete Ressource

Für eine untergeordnete Ressource kann keine Kopierschleife verwendet werden. Um mehrere Instanzen einer Ressource zu erstellen, die Sie in der Regel als innerhalb einer anderen Ressource geschachtelt definieren, müssen Sie diese Ressource stattdessen als Ressource oberster Ebene erstellen. Sie definieren die Beziehung zur übergeordneten Ressource mithilfe der Eigenschaften „type“ und „name“.

Angenommen, Sie definieren ein Dataset als untergeordnete Ressource innerhalb einer Data Factory.

```json
"resources": [
{
  "type": "Microsoft.DataFactory/factories",
  "name": "exampleDataFactory",
  ...
  "resources": [
    {
      "type": "datasets",
      "name": "exampleDataSet",
      "dependsOn": [
        "exampleDataFactory"
      ],
      ...
    }
  ]
```

Um mehrere Datasets zu erstellen, verschieben Sie sie außerhalb der Data Factory. Das Dataset muss sich auf der gleichen Ebene wie die Data Factory befinden, ist aber immer noch eine untergeordnete Ressource der Data Factory. Sie erhalten die Beziehung zwischen Dataset und Data Factory mithilfe der Eigenschaften „type“ und „name“ bei. Da „type“ nicht mehr von seiner Position in der Vorlage abgeleitet werden kann, müssen Sie den vollqualifizierten Typ im folgenden Format angeben: `{resource-provider-namespace}/{parent-resource-type}/{child-resource-type}`.

Um eine Über-/Unterordnungsbeziehung mit einer Instanz der Data Factory herzustellen, geben Sie einen Namen für das Dataset an, das den Namen der übergeordneten Ressource enthält. Verwenden Sie das folgende Format: `{parent-resource-name}/{child-resource-name}`.

Das folgende Beispiel zeigt die Implementierung:

# <a name="json"></a>[JSON](#tab/json)

```json
"resources": [
{
  "type": "Microsoft.DataFactory/factories",
  "name": "exampleDataFactory",
  ...
},
{
  "type": "Microsoft.DataFactory/factories/datasets",
  "name": "[concat('exampleDataFactory', '/', 'exampleDataSet', copyIndex())]",
  "dependsOn": [
    "exampleDataFactory"
  ],
  "copy": {
    "name": "datasetcopy",
    "count": "3"
  },
  ...
}]
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
resource dataFactoryName_resource 'Microsoft.DataFactory/factories@2018-06-01' = {
  name: "exampleDataFactory"
  ...
}

resource dataFactoryName_ArmtemplateTestDatasetIn 'Microsoft.DataFactory/factories/datasets@2018-06-01' = [for i in range(0, 3): {
  name: 'exampleDataFactory/exampleDataset${i}'
  ...
}
```

---

## <a name="example-templates"></a>Beispielvorlagen

Die folgenden Beispiele zeigen allgemeine Szenarien für das Erstellen mehrerer Ressourcen oder Eigenschaften.

|Vorlage  |Beschreibung  |
|---------|---------|
|[Speicher kopieren](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copystorage.json) |Stellt mehrere Speicherkonten mit einer Indexnummer im Namen bereit. |
|[Speicher seriell kopieren](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/serialcopystorage.json) |Stellt mehrere Speicherkonten nacheinander bereit. Der Name enthält die Indexnummer. |
|[Speicher mit Array kopieren](https://github.com/Azure/azure-docs-json-samples/blob/master/azure-resource-manager/multipleinstance/copystoragewitharray.json) |Stellt mehrere Speicherkonten bereit. Der Name enthält einen Wert aus einem Array. |

## <a name="next-steps"></a>Nächste Schritte

- Unter [Definieren der Reihenfolge für die Bereitstellung von Ressourcen in ARM-Vorlagen](define-resource-dependency.md) finden Sie weitere Informationen zum Festlegen von Abhängigkeiten für Ressourcen, die in einer Kopierschleife erstellt werden.
- Ein entsprechendes Tutorial finden Sie unter [Tutorial: Erstellen mehrerer Ressourceninstanzen mit ARM-Vorlagen](template-tutorial-create-multiple-instances.md).
- Ein Microsoft Learn-Modul, das eine Ressourcenkopie behandelt, finden Sie unter [Verwalten komplexer Cloudbereitstellungen mithilfe erweiterter Funktionen von ARM-Vorlagen](/learn/modules/manage-deployments-advanced-arm-template-features/).
- Informationen zu anderen Verwendungsmöglichkeiten der copy-Schleife finden Sie unter:
  - [Eigenschafteniteration in ARM-Vorlagen](copy-properties.md)
  - [Variableniteration in ARM-Vorlagen](copy-variables.md)
  - [Ausgabeiteration in ARM-Vorlagen](copy-outputs.md)
- Informationen zur Verwendung des „copy“-Elementes mit geschachtelten Vorlagen finden Sie unter [Verwenden des „copy“-Elements](linked-templates.md#using-copy).
