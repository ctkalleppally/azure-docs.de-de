---
title: '.NET-Tutorial: Übersicht über die Integration von Suchfunktionen'
titleSuffix: Azure Cognitive Search
description: Technische Übersicht und Setup zum Hinzufügen von Suchfunktionen zu einer Website und zum Bereitstellen in einer statischen Azure-Web-App mit .NET
manager: nitinme
author: diberry
ms.author: diberry
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 04/23/2021
ms.custom: devx-track-csharp
ms.devlang: dotnet
ms.openlocfilehash: 8a73ab04ea782c248a8fab61fb375c01a2339d1c
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/25/2021
ms.locfileid: "107981952"
---
# <a name="1---overview-of-adding-search-to-a-website-with-net"></a>1: Übersicht über das Hinzufügen von Suchfunktionen zu einer Website mit .NET

In diesem Tutorial wird eine Website zum Durchsuchen eines Katalogs mit Büchern erstellt. Anschließend wird die Website in einer statischen Azure-Web-App bereitgestellt. 

Die Anwendung ist hier verfügbar: 
* [Beispiel](https://github.com/azure-samples/azure-search-dotnet-samples/tree/master/search-website)
* [Demowebsite: aka.ms/azs-good-books](https://aka.ms/azs-good-books)

## <a name="what-does-the-sample-do"></a>Wozu dient das Beispiel? 

Diese Beispielwebsite bietet Zugriff auf einen Katalog mit 10.000 Büchern. Ein Benutzer kann den Katalog durchsuchen, indem er Text in die Suchleiste eingibt. Während der Texteingabe durch den Benutzer verwendet die Website das Vorschlagfeature des Suchindexes, um den Text zu vervollständigen. Sobald die Abfrage abgeschlossen ist, wird die Liste der Bücher mit einem Teil der Details angezeigt. Ein Benutzer kann ein Buch auswählen, um alle im Suchindex gespeicherten Details des Buchs anzuzeigen. 

:::image type="content" source="./media/tutorial-javascript-overview/cognitive-search-enabled-book-website.png" alt-text="Diese Beispielwebsite bietet Zugriff auf einen Katalog mit 10.000 Büchern. Ein Benutzer kann den Katalog durchsuchen, indem er Text in die Suchleiste eingibt. Während der Texteingabe durch den Benutzer verwendet die Website das Vorschlagfeature des Suchindexes, um den Text zu vervollständigen. Sobald die Suche abgeschlossen ist, wird die Liste der Bücher mit einem Teil der Details angezeigt. Ein Benutzer kann ein Buch auswählen, um alle im Suchindex gespeicherten Details des Buchs anzuzeigen.":::

Die Suchfunktion umfasst Folgendes: 

* Suche: Stellt Suchfunktionen für die Anwendung bereit.
* Vorschlag: Gibt während der Benutzereingabe über die Suchleiste Vorschläge an.
* Dokumentsuche: Sucht ein Dokument anhand der ID, um den gesamten Inhalt für die Detailseite abzurufen.

## <a name="how-is-the-sample-organized"></a>Wie ist das Beispiel organisiert?

Das [Beispiel](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/search-website) enthält Folgendes:

|App|Zweck|GitHub<br>Repository<br>Standort|
|--|--|--|
|Client|React-App (Darstellungsebene) zum Anzeigen von Büchern mit der Suche. Sie ruft die Azure-Funktions-App auf. |[/search-website/src](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/search-website/src)|
|Server|Azure-.NET-Funktions-App (Geschäftsebene): Ruft die Azure Cognitive Search-API mithilfe des .NET SDK auf. |[/search-website/api](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/search-website/api)|
|Masseneinfügung|.NET-Datei zum Erstellen des Indexes und zum Hinzufügen von Dokumenten zu diesem Index|[/search-website/bulk-insert](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/search-website/bulk-insert)|

## <a name="set-up-your-development-environment"></a>Einrichten der Entwicklungsumgebung

Installieren Sie Folgendes für die lokale Entwicklungsumgebung: 

- [.NET 3](https://dotnet.microsoft.com/download/dotnet/5.0)  
- [Git](https://git-scm.com/downloads)
- [Visual Studio Code](https://code.visualstudio.com/) und die folgenden Erweiterungen:
    - [Azure-Ressourcen](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureresourcegroups)
    - [Azure Cognitive Search 0.2.0 oder höher](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurecognitivesearch)
    - [Statische Azure-Web-App](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestaticwebapps) 
- Optional:
    - In diesem Tutorial wird die Azure Functions-API nicht lokal ausgeführt. Wenn Sie sie jedoch lokal ausführen möchten, müssen Sie [azure-functions-core-tools](/azure/azure-functions/functions-run-local?tabs=linux%2Ccsharp%2Cbash#install-the-azure-functions-core-tools) installieren.

## <a name="fork-and-clone-the-search-sample-with-git"></a>Forken und Klonen des Suchbeispiels mit Git

Das Forken des Beispielrepositorys ist wichtig, um die statische Web-App bereitstellen zu können. Die Web-Apps bestimmen die Buildaktionen und Bereitstellungsinhalte basierend auf Ihrem eigenen GitHub-Fork-Speicherort. Die Codeausführung in der statischen Web-App erfolgt remote. Dabei liest Azure Static Web Apps aus dem Code in Ihrem geforkten Beispiel.

1. Forken Sie auf GitHub das [Beispielrepository](https://github.com/Azure-Samples/azure-search-dotnet-samples). 

    Schließen Sie den Fork-Prozess in Ihrem Webbrowser mit Ihrem GitHub-Konto ab. In diesem Tutorial wird der Fork als Teil der Bereitstellung in einer statischen Azure-Web-App verwendet. 

1. Laden Sie an einem Bash-Terminal die Beispielanwendung auf Ihren lokalen Computer herunter. 

    Ersetzen Sie `YOUR-GITHUB-ALIAS` durch Ihren GitHub-Alias. 

    ```bash
    git clone https://github.com/YOUR-GITHUB-ALIAS/azure-search-dotnet-samples
    ```

1. Öffnen Sie in Visual Studio Code den lokalen Ordner des geklonten Repositorys. Die übrigen Aufgaben werden über Visual Studio Code ausgeführt, sofern nicht anders angegeben.

## <a name="create-a-resource-group-for-your-azure-resources"></a>Erstellen einer Ressourcengruppe für Ihre Azure-Ressourcen

1. Öffnen Sie in Visual Studio Code die [Aktivitätsleiste](https://code.visualstudio.com/docs/getstarted/userinterface), und wählen Sie das Azure-Symbol aus. 
1. Klicken Sie auf der Seitenleiste im Bereich `Resource Groups` **mit der rechten Maustaste auf Ihr Azure-Abonnement**, und wählen Sie **Ressourcengruppe erstellen** aus.

    :::image type="content" source="./media/tutorial-javascript-overview/visual-studio-code-create-resource-group.png" alt-text="**Rechtsklicken auf das Azure-Abonnement** auf der Seitenleiste im Bereich „Ressourcengruppen“ und Auswählen von **Ressourcengruppe erstellen**":::
1. Geben Sie einen Ressourcengruppennamen ein, etwa `cognitive-search-website-tutorial`. 
1. Wählen Sie einen Standort in Ihrer Nähe aus.
1. Verwenden Sie diese Ressourcengruppe später in diesem Tutorial beim Erstellen der Cognitive Search-Ressourcen und der Ressourcen der statischen Web-App. 

    Durch die Erstellung einer Ressourcengruppe erhalten Sie eine logische Einheit zum Verwalten der Ressourcen. Sie können sie u. a. löschen, wenn Sie sie nicht mehr benötigen.

## <a name="next-steps"></a>Nächste Schritte

* [Erstellen eines Suchindexes und Laden mit Dokumenten](tutorial-csharp-create-load-index.md)
* [Bereitstellen Ihrer statischen Web-App](tutorial-csharp-deploy-static-web-app.md)
