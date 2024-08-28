# Overview

RPCs (Remote Procedure Calls) are a standard software industry concept that allows methods to be called on objects that are not in the same executable. They enable communication between different processes or systems over a network.

With RPCs, a server can invoke functions on a client, and similarly, a client can invoke functions on a server. This bi-directional communication allows for flexible and dynamic interactions between clients and servers, facilitating various operations such as requesting data, executing commands, and synchronizing states across different parts of a distributed system. RPCs provide a powerful mechanism for enabling remote interactions and enhancing the functionality of networked applications.

# Basic Usage

In Omni, RPCs work with any scripts that inherit from NetworkBehaviour, `ClientBehaviour`, `DualBehaviour`, or `ServerBehaviour`. The differences between them are as follows:

- `NetworkBehaviour`: Used for objects instantiated with an identity.
- `DualBehaviour`: Allows you to program both client and server logic in the same script. It cannot be instantiated and does not have an identity.
- `ClientBehaviour` and `ServerBehaviour`: Used to program behaviors in separate scripts for client and server respectively. These cannot be instantiated and do not have an identity.

### Usage...

An example of RPCs in the same script, handling both client and server logic:

!!! note

    The method signatures can have up to three arguments for server methods and two arguments for client methods, with all arguments being optional.

    The full signature for server methods is:

    ```csharp
    [Server(id)]
    void name_of_method(DataBuffer buffer, NetworkPeer peer, int seqChannel) {}
    ```

    The full signature for client methods is:

    ```csharp
    [Client(id)]
    void name_of_method(DataBuffer buffer, int seqChannel) {}
    ```

    All arguments are optional, but they must be in the correct order and be of the correct type.

    `without arguments:`

    ```csharp
    [Server(id)]
    void name_of_method() {}
    ```

    The signature for client methods is:

    ```csharp
    [Client(id)]
    void name_of_method() {}
    ```

    `one argument:`

    ```csharp
    [Server(id)]
    void name_of_method(DataBuffer buffer) {}
    ```

    The signature for client methods is:

    ```csharp
    [Client(id)]
    void name_of_method(DataBuffer buffer) {}
    ```

    `two arguments:`

    ```csharp
    [Server(id)]
    void name_of_method(DataBuffer buffer, NetworkPeer peer) {}
    ```

    The signature for client methods is:

    ```csharp
    [Client(id)]
    void name_of_method(DataBuffer buffer, int seqChannel) {}
    ```

    Only the server method can have a `NetworkPeer` parameter.

- Let's send an RPC containing a simple message:

```csharp
[Server(1)]
void OnMsg_Server(DataBuffer buffer)
{
    print("received message: " + buffer.ReadString() + " from client");
}

[Client(1)]
void OnMsg_Client(DataBuffer buffer)
{
    print("received message: " + buffer.ReadString() + " from server");
}
```

In this example, we are invoking the RPC on the client that is connected to the server.

```csharp
protected override void OnServerPeerConnected(NetworkPeer peer, Phase phase)
{
    if (phase == Phase.End)
    {
        DataBuffer buffer = new DataBuffer();
        buffer.WriteString("Hello! (:)");
        Remote.Invoke(1, peer, buffer);
    }
}
```

!!! note

    - **The `Remote.Invoke` method**: This method is used by the server to     call RPCs on a connected client. It allows the server to request that the     client execute a specific method or operation remotely.

    - **The `Local.Invoke` method**: This method is used by the client to     call RPCs on the server. It allows the client to request that the server     execute a specific method or operation remotely.

    The `id` parameter is the ID of the RPC, used to identify the RPC on the server and client on the same object.

!!! note

    The method `Invoke` has overloads and optional arguments, allowing you to specify the recipient, delivery method, group, channel, cache, and more.

Here is the complete `example` if you want to test it. Attach this code to a script that is in the scene.

```csharp
[Server(1)]
void OnMsg_Server(DataBuffer buffer)
{
    print("received message: " + buffer.ReadString() + " from client");
}

[Client(1)]
void OnMsg_Client(DataBuffer buffer)
{
    print("received message: " + buffer.ReadString() + " from server");
}

protected override void OnServerPeerConnected(NetworkPeer peer, Phasphase)
{
    if (phase == Phase.End)
    {
        DataBuffer buffer = new DataBuffer();
        buffer.WriteString("Hello! (:)");
        Remote.Invoke(1, peer, buffer);
    }
}
```

- Send an RPC from the client to the server:

```csharp
private void Update()
{
    if (Input.GetKeyDown(KeyCode.G))
    {
        DataBuffer buffer = new DataBuffer();
        buffer.WriteString("Hello! (:)");
        Local.Invoke(1, buffer);
    }
}
```

!!! note

    Remember that we are inheriting from `DualBehaviour`, which implements the logic for both in the same script. `Local.Invoke` is not available when inheriting from `ServerBehaviour`, as it only contains server logic, and any client RPC signatures will be ignored. The same applies to `ClientBehaviour`, where `Remote.Invoke` is not available because it only handles client logic, and any server RPC signatures will be ignored;

    `NetworkBehaviour` should be used for objects that are instantiated on the network and have an identity.

- Let's create a messaging chat to clearly demonstrate the use of RPCs. This time, we will keep the logic in separate scripts.

Create a `ChatServer` script that inherits from `ServerBehaviour` and create a `ChatClient` script that inherits from `ClientBehaviour`.

`Server-Side`:

```csharp
public class ChatServer : ServerBehaviour
{
    // eg 1(example): Message without validation
    [Server(1)]
    void OnChat_Server(DataBuffer buffer, NetworkPeer peer)
    {
        // Server resend the message to all clients
        Remote.Invoke(1, peer, buffer, Target.All, DeliveryMode.ReliableOrdered);
    }

    // eg(example): Message with simple validation on server
    [Server(1)]
    void OnChat_Server(DataBuffer buffer, NetworkPeer peer)
    {
        var nBuffer = new DataBuffer();
        // Re-send the message to all clients
        string message = buffer.ReadString();
        if (message.Contains("Hello"))
        {
            message = "*****";
        }

        nBuffer.WriteString(message);
        Remote.Invoke(1, peer, nBuffer, Target.All, DeliveryMode.ReliableOrdered);
    }
}
```

!!! warning

    You cannot have multiple RPC's with the same identification ID in the same script.

`Client-Side`:

```csharp
public class ChatClient : ClientBehaviour
{
    [Client(1)]
    void OnChat_Client(DataBuffer buffer)
    {
        print(buffer.ReadString());
    }

    private void Update()
    {
        if (Input.GetKeyUp(KeyCode.Space))
        {
            DataBuffer buffer = new DataBuffer();
            buffer.WriteString("Hello World!");
            Local.Invoke(1, buffer);
        }
    }
}
```

### Network Behaviour

When using an instantiated object with an identity, the scripts must inherit from `NetworkBehaviour`. The RPCs function exactly the same way, except thatthey do not take the `Peer` argument.

eg:
`Remote.Invoke(1, nBuffer, Target.All, DeliveryMode.ReliableOrdered);`
