# ASP.NET Core - SignalR

## Table of Contents

- [What Is SignalR?](#what-is-signalr)
- [Core SignalR Concepts](#core-signalr-concepts)
  - [Transports](#transports)
  - [Hubs](#hubs)
- [Creating a SignalR Project](#creating-a-signalr-project)
  - [Server-Side Setup](#server-side-setup)
  - [Client-Side Setup](#client-side-setup)
- [Sending Data from Client to Server and Back](#sending-data-from-client-to-server-and-back)
- [Sending Client-Specific Messages](#sending-client-specific-messages)
- [Automatic Reconnection](#automatic-reconnection)
  - [Configuring Automatic Reconnect](#configuring-automatic-reconnect)

## What Is SignalR?

SignalR simplifies adding real-time web functionality to applications. Real-time web functionality enables server-side code to push content to clients instantly.

SignalR provides an API for creating server-to-client Remote Procedure Calls (RPC). The RPCs invoke functions on clients from server-side ASP.NET Core code.

Features include:

- Automatic connection management
- Sending messages to all connected clients simultaneously
- Sending messages to specific clients or groups of clients
- Scaling to handle increasing traffic
- The SignalR Hub Protocol

[↑ Back to Top](#table-of-contents)

## Core SignalR Concepts

### Transports

SignalR supports the following techniques for handling real-time communication, in order of graceful fallback:

- WebSockets
- Server-Sent Events
- Long polling

SignalR automatically chooses the best transport method supported by the server and client.

### Hubs

A hub is a high-level pipeline that enables communication between a client and a server and allows each side to call methods on the other. A hub acts as the communication foundation between the client and server when using SignalR.

SignalR provides two built-in hub protocols:

- A text protocol based on JSON
- A binary protocol based on MessagePack

MessagePack generally creates smaller messages than JSON. Older browsers must support XHR level 2 to provide MessagePack protocol support.

Hubs call client-side code by sending messages containing the name and parameters of the client-side method. Objects sent as method parameters are deserialized using the configured protocol. The client tries to match the name to a method in the client-side code. When the client finds a match, it calls the method and passes the deserialized parameter data to it.

[↑ Back to Top](#table-of-contents)

## Creating a SignalR Project

### Server-Side Setup

Create a business-specific hub class, such as `ChartHub`, by inheriting from the `Hub` class:

```csharp
public class ChartHub : Hub
{
}
```

Add SignalR to the service collection using `AddSignalR`:

```csharp
builder.Services.AddSignalR();
```

Add SignalR to the request pipeline and configure the hub route:

```csharp
app.MapControllers();
app.MapHub<ChartHub>("/chart");
```

Create a business-specific controller derived from `ControllerBase`. The controller should have an `IHubContext<ChartHub>` dependency. The hub context can access and invoke all hub methods.

```csharp
[Route("api/[controller]")]
[ApiController]
public class ChartController : ControllerBase
{
    private readonly IHubContext<ChartHub> _hub;
    private readonly TimerManager _timer;

    public ChartController(IHubContext<ChartHub> hub, TimerManager timer)
    {
        _hub = hub;
        _timer = timer;
    }

    [HttpGet]
    public IActionResult Get()
    {
        if (!_timer.IsTimerStarted)
        {
            _timer.PrepareTimer(
                () => _hub.Clients.All.SendAsync(
                    "TransferChartData",
                    DataManager.GetData()));
        }

        return Ok(new { Message = "Request Completed" });
    }
}
```

Using the `TransferChartData` event, the server sends data to all subscribed clients. Every client with a listener for `TransferChartData` receives the published data.

For a complete example, see [Using .NET Core SignalR with Angular and real-time charts](https://code-maze.com/netcore-signalr-angular-realtime-charts/).

### Client-Side Setup

Install the SignalR JavaScript client library:

```shell
npm install @microsoft/signalr --save
```

Create a business-specific service class and define a property for the SignalR hub connection. Build and start the connection in a function, and subscribe to the `TransferChartData` event with `addTransferChartDataListener`.

Inside the application component:

- Add `SignalRService` as a dependency in the constructor.
- Call `_service.startConnection()` to establish the SignalR connection.
- Call `_service.addTransferChartDataListener()` to subscribe to the `transferDataEvent` event.
- Make the HTTP `GET` request using the HTTP client and subscribe to its result.

The flow is straightforward: start the connection, add the listener, and send the request to the controller's `GET` action.

[↑ Back to Top](#table-of-contents)

## Sending Data from Client to Server and Back

The earlier example only shows one-way communication: broadcasting data from the server to the client. Client-to-server communication can use a hub method that receives data and then broadcasts it to all connected clients.

Add a method to the business-specific hub class:

```csharp
public class ChartHub : Hub
{
    public async Task BroadcastChartData(List<ChartModel> data)
    {
        await Clients.All.SendAsync("broadcastchartdata", data);
    }
}
```

In the application component, add a method that publishes data and a listener that subscribes to the `broadcastchartdata` event:

```typescript
public broadcastChartData = () => {
    const data = this.data.map(m => {
        const temp = {
            data: m.data,
            label: m.label
        };
        return temp;
    });

    this.hubConnection.invoke('broadcastchartdata', data)
        .catch(err => console.error(err));
};

public addBroadcastChartDataListener = () => {
    this.hubConnection.on('broadcastchartdata', (data) => {
        this.broadcastedData = data;
    });
};
```

[↑ Back to Top](#table-of-contents)

## Sending Client-Specific Messages

SignalR allows messages to be sent to:

- A particular client connection
- All connections associated with a specific user
- A named group of connections

Each client connecting to a hub has a unique connection ID. It can be retrieved from `Context.ConnectionId` in the hub context. This value can be used to send a message only to that client.

For implementation details, see [How to send client-specific messages using SignalR](https://code-maze.com/how-to-send-client-specific-messages-using-signalr/).

[↑ Back to Top](#table-of-contents)

## Automatic Reconnection

SignalR uses WebSockets under the hood where available and gracefully falls back to other techniques when a WebSocket connection cannot be established. WebSocket connections can disconnect intermittently.

If the connection is lost, real-time behavior stops until the connection is restored. SignalR does not reconnect automatically after every disconnection. Refreshing the page is a poor fallback because it creates a bad user experience.

ASP.NET Core 3.0 introduced SignalR's automatic reconnection feature.

### Configuring Automatic Reconnect

Modify the `startConnection()` method of the `SignalRService` class:

```typescript
this.hubConnection = new signalR.HubConnectionBuilder()
    .withUrl('https://localhost:5001/chart')
    .withAutomaticReconnect()
    .configureLogging(signalR.LogLevel.Information)
    .build();
```

Calling `withAutomaticReconnect()` without arguments waits 0, 2, 10, and 30 seconds before each reconnection attempt. It stops after four failed attempts. Custom retry delays can be passed as an argument.

To resume the data flow after reconnection, call `onreconnected` on the hub connection:

```typescript
this.hubConnection.onreconnected(() => {
    this.http.get('https://localhost:5001/api/chart')
        .subscribe(res => {
            console.log(res);
        });
});
```

For additional details, see [SignalR automatic reconnect option](https://code-maze.com/signalr-automatic-reconnect-option/).

[↑ Back to Top](#table-of-contents)
