---
title: include file
description: include file
services: event-hubs
author: spelluru
ms.service: event-hubs
ms.topic: include
ms.date: 01/21/2021
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 2970a7271761082a358b50c069da82ed785df8b0
ms.sourcegitcommit: aba63ab15a1a10f6456c16cd382952df4fd7c3ff
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 04/25/2021
ms.locfileid: "107990467"
---
### <a name="what-ports-do-i-need-to-open-on-the-firewall"></a>Welche Ports muss ich in der Firewall öffnen? 
Sie können die folgenden Protokolle mit Azure Event Hubs verwenden, um Ereignisse zu senden und zu empfangen:

- Advanced Message Queuing Protocol 1.0 (AMQP)
- Hypertext Transfer Protocol 1.1 mit TLS (HTTPS)
- Apache Kafka

In der folgenden Tabelle finden Sie die ausgehenden Ports, die Sie öffnen müssen, um diese Protokolle für die Kommunikation mit Azure Event Hubs verwenden zu können. 

| Protocol | Ports | Details | 
| -------- | ----- | ------- | 
| AMQP | 5671 und 5672 | Weitere Informationen finden Sie im [AMQP 1.0 in Azure Service Bus und Event Hubs – Protokollleitfaden](../articles/service-bus-messaging/service-bus-amqp-protocol-guide.md). | 
| HTTPS | 443 | Dieser Port wird für die HTTP/REST-API und für AMQP-over-WebSockets verwendet. |
| Kafka | 9093 | Weitere Informationen finden Sie unter [Verwenden von Azure Event Hubs aus Apache Kafka-Anwendungen](../articles/event-hubs/event-hubs-for-kafka-ecosystem-overview.md).

Der HTTPS-Port ist für die ausgehende Kommunikation auch dann erforderlich, wenn AMQP über Port 5671 verwendet wird, da mehrere von den Client-SDKs ausgeführte Verwaltungsvorgänge sowie der Abruf von Token aus Azure Active Directory (falls verwendet) über HTTPS erfolgen. 

Die offiziellen Azure-SDKs verwenden im Allgemeinen das AMQP-Protokoll zum Senden von Ereignissen an und Empfangen von Ereignissen von Event Hubs. Die Option AMQP-over-WebSockets-Protokoll wird über TCP-Port 443 wie die HTTP-API ausgeführt, ist aber ansonsten funktionell identisch mit einfachem AMQP. Diese Option weist eine höhere anfängliche Verbindungslatenz auf, weil zusätzliche Handshakeroundtrips ausgeführt werden und der Mehraufwand geringfügig höher ist. Dies ist ein Kompromiss, der aufgrund der gemeinsamen Verwendung des HTTPS-Ports eingegangen werden muss. Wenn dieser Modus ausgewählt ist, ist TCP-Port 443 für die Kommunikation ausreichend. Mit den folgenden Optionen können Sie den einfachen AMQP- oder den AMQP-WebSockets-Modus auswählen:

| Sprache | Option   |
| -------- | ----- |
| .NET     | Eigenschaft [EventHubConnectionOptions.TransportType](/dotnet/api/azure.messaging.eventhubs.eventhubconnectionoptions.transporttype) mit [EventHubsTransportType.AmqpTcp](/dotnet/api/azure.messaging.eventhubs.eventhubstransporttype) oder [EventHubsTransportType.AmqpWebSockets](/dotnet/api/azure.messaging.eventhubs.eventhubstransporttype) |
| Java     | [com.microsoft.azure.eventhubs.EventProcessorClientBuilder.transporttype](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/src/main/java/com/azure/messaging/eventhubs/EventProcessorClientBuilder.java) mit [AmqpTransportType.AMQP](/java/api/com.azure.core.amqp.amqptransporttype) oder [AmqpTransportType.AMQP_WEB_SOCKETS](/java/api/com.azure.core.amqp.amqptransporttype) |
| Knoten  | [EventHubConsumerClientOptions](/javascript/api/@azure/event-hubs/eventhubconsumerclientoptions) verfügt über eine `webSocketOptions`-Eigenschaft. |
| Python | [EventHubConsumerClient.transport_type](/python/api/azure-eventhub/azure.eventhub.eventhubconsumerclient) mit [TransportType.Amqp](/python/api/azure-eventhub/azure.eventhub.transporttype) oder [TransportType.AmqpOverWebSocket](/python/api/azure-eventhub/azure.eventhub.transporttype) |

### <a name="what-ip-addresses-do-i-need-to-allow"></a>Welche IP-Adressen muss ich zulassen?
Wenn Sie mit Azure arbeiten, müssen Sie manchmal bestimmten IP-Adressbereichen oder URLs in Ihrer Unternehmensfirewall oder Ihrem Proxy erlauben, auf alle Azure-Dienste zuzugreifen, die Sie verwenden oder zu verwenden versuchen. Überprüfen Sie, ob der Datenverkehr für die von Event Hubs verwendeten IP-Adressen zulässig ist. Weitere Informationen zu den von Azure Event Hubs verwendeten IP-Adressen finden Sie unter [Azure-IP-Bereiche und Diensttags – öffentliche Cloud](https://www.microsoft.com/download/details.aspx?id=56519).

Vergewissern Sie sich außerdem, dass die IP-Adresse für Ihren Namespace zulässig ist. Um die richtigen IP-Adressen zu ermitteln, die für Ihre Verbindungen zulässig sind, führen Sie die folgenden Schritte aus:

1. Führen Sie den folgenden Befehl an einer Eingabeaufforderung aus: 

    ```
    nslookup <YourNamespaceName>.servicebus.windows.net
    ```
2. Notieren Sie sich die IP-Adresse, die in `Non-authoritative answer` zurückgegeben werden. 

Wenn Sie **Zonenredundanz** für Ihren Namespace verwenden, müssen Sie einige zusätzliche Schritte ausführen: 

1. Führen Sie zunächst nslookup für den Namespace aus.

    ```
    nslookup <yournamespace>.servicebus.windows.net
    ```
2. Notieren Sie sich den Namen im Abschnitt **non-authoritative answer** (nicht autorisierende Antwort), der in einem der folgenden Formate vorliegt: 

    ```
    <name>-s1.cloudapp.net
    <name>-s2.cloudapp.net
    <name>-s3.cloudapp.net
    ```
3. Führen Sie den Befehl „nslookup“ für jeden Namen mit den Suffixen s1, s2 und s3 aus, um die IP-Adressen aller drei Instanzen zu erhalten, die in drei Verfügbarkeitszonen ausgeführt werden. 

    > [!NOTE]
    > Die vom `nslookup`-Befehl zurückgegebene IP-Adresse ist keine statische IP-Adresse. Allerdings bleibt sie gleich, bis die zugrunde liegende Bereitstellung gelöscht oder in einen anderen Cluster verschoben wird.

### <a name="what-client-ips-are-sending-events-to-or-receiving-events-from-my-namespace"></a>Welche Client-IP-Adressen senden Ereignisse an meinen Namespace oder empfangen Ereignisse daraus?
Aktivieren Sie zunächst die [IP-Filterung](../articles/event-hubs/event-hubs-ip-filtering.md) für den Namespace. 

Aktivieren Sie dann Diagnoseprotokolle für [Event Hubs-Verbindungsereignisse für virtuelle Netzwerke](../articles/event-hubs/event-hubs-diagnostic-logs.md#event-hubs-virtual-network-connection-event-schema), indem Sie den Anweisungen unter [Aktivieren von Diagnoseprotokollen](../articles/event-hubs/event-hubs-diagnostic-logs.md#enable-diagnostic-logs) folgen. Es wird die IP-Adresse angezeigt, für die die Verbindung verweigert wird.

```json
{
    "SubscriptionId": "0000000-0000-0000-0000-000000000000",
    "NamespaceName": "namespace-name",
    "IPAddress": "1.2.3.4",
    "Action": "Deny Connection",
    "Reason": "IPAddress doesn't belong to a subnet with Service Endpoint enabled.",
    "Count": "65",
    "ResourceId": "/subscriptions/0000000-0000-0000-0000-000000000000/resourcegroups/testrg/providers/microsoft.eventhub/namespaces/namespace-name",
    "Category": "EventHubVNetConnectionEvent"
}
```

> [!IMPORTANT]
> Protokolle virtueller Netzwerke werden nur dann generiert, wenn der Namespace Zugriff über **spezifische IP-Adressen** (IP-Filterregeln) erlaubt. Wenn Sie den Zugriff auf Ihren Namespace mit diesen Features nicht einschränken möchten und dennoch Protokolle virtueller Netzwerke erhalten möchten, um IP-Adressen von Clients zu verfolgen, die sich mit dem Event Hubs-Namespace verbinden, können Sie die folgende Umgehungslösung verwenden: Aktivieren Sie die IP-Filterung, und fügen Sie den gesamten adressierbaren IPv4-Bereich (1.0.0.0/1 bis 255.0.0.0/1) hinzu. Event Hubs unterstützt keine IPv6-Adressbereiche. 

> [!NOTE]
> Derzeit ist es nicht möglich, die IP-Quelladresse einer einzelnen Nachricht oder eines einzelnen Ereignisses zu bestimmen. 
