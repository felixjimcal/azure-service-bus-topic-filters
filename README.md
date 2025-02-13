# azure-service-bus-topic-filters
This project demonstrates how to use Azure Service Bus Topics with Subscription Filters to send and receive targeted messages. 


# **Azure Service Bus Topic with Subscription Filters - README**

## **📌 Overview**
This project demonstrates how to use **Azure Service Bus Topics** with **Subscription Filters** to send and receive targeted messages. The goal is to ensure that only specific subscribers receive messages based on predefined filters.

### **🔹 Scenario**
- A **topic named "ventas"** acts as the central hub for messages.
- There are **multiple subscribers** (Spain, Norway, Denmark).
- Each subscriber only receives messages relevant to them:
  - **Spain** receives messages with `destiny = 'spain'` or `destiny = 'all'`.
  - **Norway** receives messages with `destiny = 'norway'`, `destiny = 'north'`, or `destiny = 'all'`.
  - **Denmark** receives messages with `destiny = 'denmark'`, `destiny = 'north'`, or `destiny = 'all'`.

---

## **⚙️ Prerequisites**
1. **Azure Subscription** with an **Azure Service Bus namespace**.
2. A **Service Bus Topic** named `ventas`.
3. At least **three subscriptions**:
   - **spain** → Filter: `"destiny = 'spain' OR destiny = 'all'"`
   - **norway** → Filter: `"destiny = 'norway' OR destiny = 'north' OR destiny = 'all'"`
   - **denmark** → Filter: `"destiny = 'denmark' OR destiny = 'north' OR destiny = 'all'"`

---

## **🚀 Publisher - Sending Messages**
The publisher sends messages with a **custom property (`destiny`)** that determines the recipients.

### **Code**
```csharp
using Azure.Messaging.ServiceBus;

internal class Publisher
{
    private const string connectionString = "Endpoint=sb:";
    private const string topicName = "ventas";

    private static async Task Main()
    {
        await SendMessage("Notification for everyone", "all");
        await SendMessage("Notification for Norway only", "norway");
        await SendMessage("Notification for Denmark only", "denmark");
        await SendMessage("Notification for Northern entities", "north");
    }

    private static async Task SendMessage(string content, string destination)
    {
        await using var client = new ServiceBusClient(connectionString);
        ServiceBusSender sender = client.CreateSender(topicName);

        ServiceBusMessage message = new ServiceBusMessage(content);
        message.ApplicationProperties.Add("destiny", destination); // 🔥 Important: Must match subscription filters

        await sender.SendMessageAsync(message);
        Console.WriteLine($"📨 Message sent: '{content}' for {destination}");
    }
}
```

---

## **🏢 Spain Subscriber**
The Spain subscriber listens to the `"ventas"` topic and only processes messages where `"destiny = 'spain' OR destiny = 'all'"`.

### **Code**
```csharp
using Azure.Messaging.ServiceBus;

public class Spain
{
    private const string connectionString = "Endpoint=sb:";
    private const string topicName = "ventas";
    private const string subscriptionName = "spain";

    private static async Task Main()
    {
        await using var client = new ServiceBusClient(connectionString);
        ServiceBusReceiver receiver = client.CreateReceiver(topicName, subscriptionName);

        Console.WriteLine("🇪🇸 Spain listening for messages...");
        while (true)
        {
            var message = await receiver.ReceiveMessageAsync();
            if (message != null)
            {
                Console.WriteLine($"📥 Spain received: {message.Body}");
                await receiver.CompleteMessageAsync(message);
            }
        }
    }
}
```

---

## **🏗️ Setting up Subscription Filters**
Subscription filters must be created **before running the subscribers**. You can set them up using **Azure Portal** or **C# Code**.

### **C# Code for Creating Filters**
```csharp
using Azure.Messaging.ServiceBus.Administration;

class SubscriptionSetup
{
    private const string connectionString = "Endpoint=sb:";
    private const string topicName = "ventas";

    static async Task Main()
    {
        await CreateSubscription("spain", "destiny = 'spain' OR destiny = 'all'");
        await CreateSubscription("norway", "destiny = 'norway' OR destiny = 'north' OR destiny = 'all'");
        await CreateSubscription("denmark", "destiny = 'denmark' OR destiny = 'north' OR destiny = 'all'");
    }

    static async Task CreateSubscription(string name, string filter)
    {
        var adminClient = new ServiceBusAdministrationClient(connectionString);

        if (!await adminClient.SubscriptionExistsAsync(topicName, name))
        {
            await adminClient.CreateSubscriptionAsync(topicName, name);
            await adminClient.CreateRuleAsync(topicName, name, new CreateRuleOptions("Filter", new SqlRuleFilter(filter)));
            await adminClient.DeleteRuleAsync(topicName, name, "$Default");
        }

        Console.WriteLine($"✅ Subscription '{name}' created with filter: {filter}");
    }
}
```

---

## **✅ Expected Output**
| **Message**               | **Spain** | **Norway** | **Denmark** |
|---------------------------|----------|------------|------------|
| `"Notification for everyone"` | ✅ | ✅ | ✅ |
| `"Notification for Norway only"` | ❌ | ✅ | ❌ |
| `"Notification for Denmark only"` | ❌ | ❌ | ✅ |
| `"Notification for Northern entities"` | ❌ | ✅ | ✅ |

---

## **📌 Summary**
✅ **Used an Azure Service Bus Topic ("ventas")** as a centralized message broker.  
✅ **Configured multiple subscriptions with filters to receive only relevant messages.**  
✅ **Published messages with a `destiny` property that determines the recipients.**  
✅ **Ensured only specific subscribers received messages based on predefined SQL filters.**  

🚀 **Now your messages reach only the right audience!** 🎯

