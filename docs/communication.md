## RPC (Remote Procedure Call)

RPCs (Remote Procedure Calls) are a standard software industry concept that allows methods to be called on objects that are not in the same executable. They enable communication between different processes or systems over a network.

With RPCs, a server can invoke functions on a client, and similarly, a client can invoke functions on a server. This bi-directional communication allows for flexible and dynamic interactions between clients and servers, facilitating various operations such as requesting data, executing commands, and synchronizing states across different parts of a distributed system. RPCs provide a powerful mechanism for enabling remote interactions and enhancing the functionality of networked applications.

---

!!! info "RPC Basic Structure"
    <center>
    ``` mermaid
    graph LR
      A[Ruan<br>___Local Player___] ---> | User Input Rpc | B{Server}
      B ---> | Move Rpc | A
      B ---> | Move Rpc | C[Junior<br>___Remote Player___]
      B ---> | Move Rpc | D[Mike<br>___Remote Player___]
    ```
    </center>

     The diagram illustrates the basic flow of an RPC (Remote Procedure Call) in a multiplayer environment.  

    - The `local player` (Ruan) sends an RPC to the `server` (e.g., a movement or input).  
    - The `server` processes the request and broadcasts the action to all relevant clients.  
    - The remote players (e.g., Junior and Mike), the server sends the update, which they process and direct the action to the identity of the player who initiated the action (Ruan).  
    - This ensures that all players, including the initiating player (Ruan), receive the update, maintaining a consistent and authoritative game state across the network.
---                   

!!! warning
    RPC's are also supported in base classes. If you are using a base class for network functionality, ensure that the base class name includes the `Base` prefix.  
    
    ???+ example
        ```csharp
        public class PlayerBase : NetworkBehaviour // Note the "Base" prefix
        {
        }
        
        public class Player : PlayerBase
        {
        }
        ```
   
    This naming convention is required for the RPCs to function correctly in inherited classes.

!!! note
    Before proceeding, refer to the [Communication Structure](./index.md#communication-structure) and [Service Locator Pattern](./index.md#service-locator-pattern) pages for essential background information.

---

### Method Signature

RPC method signatures can accept up to three parameters, allowing flexibility in handling data, network information, and communication channels. The available parameters include:

- **DataBuffer message**: Used for receiving data sent by the client or server.
- **NetworkPeer peer (***Server-Only***)**: Provides information about the client that invoked the RPC, useful for identifying the sender.
- **int seqChannel**: Specifies the sequence channel for the RPC, allowing control over the order and priority of messages.

Each RPC method must be marked with the `[Server]` or `[Client]` attribute. The method name, parameters, and their order determine the specific signature of the RPC.

---

Below are examples of valid RPC method signatures on the server side:

#### Server-Side

=== "Signature 1"
    ???+ example "Signature 1"
        ```csharp
        public class Player : NetworkBehaviour
        {
            [Server(1)]
            void Example()
            {
               // This function is called on the server when a client invokes it
            }
        }
        ```
=== "Signature 2"
    ???+ example "Signature 2"
        ```csharp
        public class Player : NetworkBehaviour
        {
            [Server(1)]
            void Example(DataBuffer message)
            {
               // This function is called on the server when a client invokes it
            }
        }
        ```
=== "Signature 3"
    ???+ example "Signature 3"
        ```csharp
        public class Player : NetworkBehaviour
        {
            [Server(1)]
            void Example(DataBuffer message, NetworkPeer peer)
            {
               // This function is called on the server when a client invokes it
            }
        }
        ```
=== "Signature 4"
    ???+ example "Signature 4"
        ```csharp
        public class Player : NetworkBehaviour
        {
            [Server(1)]
            void Example(DataBuffer message, NetworkPeer peer, int seqChannel)
            {
               // This function is called on the server when a client invokes it
            }
        }
        ```

---

Below are examples of valid RPC method signatures on the client side:

#### Client-Side

=== "Signature 1"
    ???+ example "Signature 1"
        ```csharp
        public class Player : NetworkBehaviour
        {
            [Client(1)]
            void Example()
            {
                // This function is called on the client when a server invokes it
            }
        }
        ```
=== "Signature 2"
    ???+ example "Signature 2"
        ```csharp
        public class Player : NetworkBehaviour
        {
            [Client(1)]
            void Example(DataBuffer message)
            {
                // This function is called on the client when a server invokes it
            }
        }
        ```
=== "Signature 3"
    ???+ example "Signature 3"
        ```csharp
        public class Player : NetworkBehaviour
        {
            [Client(1)]
            void Example(DataBuffer message, int seqChannel)
            {
                // This function is called on the client when a server invokes it
            }
        }
        ```

!!! note
    Each RPC is assigned a unique **ID** within its class to ensure routing. This ID differentiates RPC methods within the same class for client-server communication.

!!! warning
    RPC ID's must be between ***1*** and ***230***. An exception will be thrown if the ID is out of range. The id's are unique only within the script itself, other script instances can use the same id.

---

Examples

=== "Example 1 (NetworkBehaviour)"
    ???+ example "Example 1 (NetworkBehaviour)"
        ```csharp
        public class Player : NetworkBehaviour
        {
            [Server(1)]
            void ExampleOnServer() // Signature 1
            {
               print("Wow! This works on the server!");
            }
    
            [Client(1)]
            void ExampleOnClient() // Signature 1
            {
               print("Wow! This works on the client!");
            }
        }
        ```

=== "Example 2 (ServerBehaviour & ClientBehaviour)"
    ???+ example "Example 2 (ServerBehaviour & ClientBehaviour)"
        ```csharp
        public class LoginServer : ServerBehaviour
        {
            [Server(1)]
            void Example() // Signature 1
            {
               print("Wow! This works on the server!");
            }
        }
    
        public class LoginClient : ClientBehaviour
        {
            [Client(1)]
            void Example() // Signature 1
            {
               print("Wow! This works on the client!");
            }
        }
        ```

=== "Example 3 (DualBehaviour)"
    ???+ example "Example 3 (DualBehaviour)"
        ```csharp
        public class Player : DualBehaviour
        {
            [Server(1)]
            void ExampleOnServer() // Signature 1
            {
               print("Wow! This works on the server!");
            }
    
            [Client(1)]
            void ExampleOnClient() // Signature 1
            {
               print("Wow! This works on the client!");
            }
        }
        ```

---

### How to Invoke a RPC

The `Local` and `Remote` properties are part of the public API inherited from `NetworkBehaviour`, `ClientBehaviour`, `ServerBehaviour`, and `DualBehaviour`. They are designed to facilitate communication between client and server in the network environment. Each property enforces specific usage restrictions to ensure proper client-server interactions.

**Local**

  - **Purpose**: Allows the client to invoke messages on the server.
  - **Restriction**: This property is intended for client-side use only. Attempting to access it on the server side will throw an exception, preventing unintended server-side access.
  - **Usage**: Useful for client-initiated communications directed at the server.
  - **Overloads**: Invoke() has 8 overloads. For details on the available overloads, please refer to the [API Reference](./index.md).

**Remote**

  - **Purpose**: Allows the server to invoke messages on the client.
  - **Restriction**: This property is intended for server-side use only. Attempting to access it on the client side will throw an exception, ensuring that only the server can utilize this property.
  - **Usage**: Useful for server-initiated communications directed at the client.
  - **Overloads**: Invoke() has 8 overloads. For details on the available overloads, please refer to the [API Reference](./index.md).

---

=== "Example 1 (NetworkBehaviour) - Send an RPC from the client to the server"
    ???+ example "Example 1 (NetworkBehaviour) - Send an RPC from the client to the server"
        ```csharp
        public class Player : NetworkBehaviour
        {
            void Update()
            {
               // Send an RPC from the client to the server
               if (Input.GetKeyUp(KeyCode.S) && IsLocalPlayer)
               {
                   Local.Invoke(1);
               }
            }
    
            [Server(1)]
            void Example() // Signature 1
            {
               print("Wow! This works on the server!");
            }
        }
        ```

=== "Example 2 (NetworkBehaviour) - Send an RPC from the server to the client"
    ???+ example "Example 2 (NetworkBehaviour) - Send an RPC from the server to the client"
        ```csharp
        public class Player : NetworkBehaviour
        {
            void Update()
            {
               // Send an RPC from the server to the client
               if (Input.GetKeyUp(KeyCode.A) && IsServer)
               {
                   Remote.Invoke(1);
               }
            }
    
            [Client(1)]
            void Example() // Signature 1
            {
               print("Wow! This works on the client!");
            }
        }
        ```

`Invoke` with ***arguments***:

!!! tip
    The `Local.Invoke()` and `Remote.Invoke()` methods has 8 overloads and optional arguments. However, the overloads available can vary depending on the network base class used(i.e. `NetworkBehaviour`, `ClientBehaviour`, `ServerBehaviour`, and `DualBehaviour`). For details on the available overloads, please refer to the [API Reference](./index.md).

=== "Client-Side"
    ???+ example "Client-Side"
        ```csharp
        public class Player : NetworkBehaviour
        {
            void Update()
            {
               // Send an RPC from the client to the server
               if (Input.GetKeyUp(KeyCode.S) && IsLocalPlayer)
               {
                   using DataBuffer message = Rent();
                   message.WriteString("Hello World!");
                   message.Write(123f);
                   Local.Invoke(1, message, DeliveryMode.ReliableOrdered);
               }
            }
    
            [Server(1)]
            void Example(DataBuffer message) // Signature 2
            {
               string str = message.ReadString();
               float num = message.Read<float>();
            }
        }
        ```

=== "Server-Side"
    ???+ example "Server-Side"
        ```csharp
        public class Player : NetworkBehaviour
        {
            void Update()
            {
               // Send an RPC from the server to the client
               if (Input.GetKeyUp(KeyCode.A) && IsServer)
               {
                   using DataBuffer message = Rent();
                   message.WriteString("Hello World!");
                   message.Write(123f);
                   Remote.Invoke(1, message, Target.All, DeliveryMode.ReliableOrdered);
               }
            }
    
            [Client(1)]
            void Example(DataBuffer message) // Signature 2
            {
               string str = message.ReadString();
               float num = message.Read<float>();
            }
        }
        ```

!!! tip
    `Invoke()` can send primitive values without the need a explicit `DataBuffer` object. For details on the available overloads, please refer to the [API Reference](./index.md).

    ???+ example
        ```csharp
           // Send an RPC from the client to the server without a explicit `DataBuffer`
           Local.Invoke(1, transform.position, transform.rotation); // Has optional arguments
    
           // Send an RPC from the server to the client without a explicit `DataBuffer`
           Remote.Invoke(1, transform.position, transform.rotation, new SyncOptions()
           {
           	  DeliveryMode = DeliveryMode.Unreliable
           });
        ```

    !!! warning
        Using this overload only unmanaged types is allowed. Like `Vector3`, `Quaternion`, `int`, `float`, etc.

---

## Network Variables

At a high level, a `[NetworkVariable]` attribute is a automatic way to synchronize a property ("variable") between a server and client(s) without having to use custom messages or RPCs.

---

!!! info "Network Variable Structure"
    <center>
    ``` mermaid
    graph LR
      Ref{Game Object<br>___Server Side___} --> | Health Change | A{RPC}
      A ---> | Health Update | B[Mike<br>___Client Side___]
      A ---> | Health Update | C[Ruan<br>___Client Side___]
    ```
    </center>

    The diagram illustrates how a **Network Variable** operates in a multiplayer environment:

    - A **server-side game object** modifies a variable (e.g., `Health`).
    - This change is processed by the **server**, which acts as the authoritative source.
    - The **server** then sends updates to all connected clients (e.g., Mike and Ruan), ensuring that each client reflects the latest value of the variable.
    - These updates allow all players to have a synchronized and consistent view of the variable's state, regardless of who initiated the change or their connection latency.

    This structure highlights the server's role in maintaining authority and consistency across the network.
---          

!!! warning
    Network Variables are also supported in base classes. If you are using a base class for network functionality, ensure that the base class name includes the `Base` prefix.  
    
    ???+ example
        ```csharp
        public class PlayerBase : NetworkBehaviour // Note the "Base" prefix
        {
        }
        
        public class Player : PlayerBase
        {
        }
        ```
   
    This naming convention is required for the Network Variables to function correctly in inherited classes.

!!! note
    Before proceeding, refer to the [Communication Structure](./index.md#communication-structure) and [Service Locator Pattern](./index.md#service-locator-pattern) pages for essential background information.

### How to Use

!!! warning
    **Naming Convention**: The field name must be prefixed with `m_` and the first letter after the prefix must be capitalized.

    - For example, `m_Health`
    - For example, `m_Mana`

    ???+ example
        ```csharp
        public partial class Player : NetworkBehaviour
        {
            [NetworkVariable] 
            private float m_Health = 100f;

            [NetworkVariable] 
            private float m_Mana = 100f;
        }
        ```
    The class must be `partial` to use the `[NetworkVariable]` attribute.

---

!!! note
     The `Omni Source Generator` will generate properties, hooks, options, and methods for each `[NetworkVariable]`.

#### Generated Properties

Generated properties in Omni are designed to automatically synchronize their values across the server and all connected clients each time the property is modified. This ensures that all instances of the property remain consistent throughout the networked environment, maintaining real-time accuracy.

!!! warning
    Omni does not perform checks to determine if the new value is different from the current value. Each time the property’s `setter` is invoked, the value is synchronized across the network, regardless of whether it has changed. This can lead to unnecessary network updates if the property is set to the same value repeatedly, so it is recommended to manage calls to the setter carefully to optimize performance.

???+ example "Automatically Synchronized"
    ```csharp
    public partial class Player : NetworkBehaviour
    {
        [NetworkVariable] 
        private float m_Health = 100f;
        [NetworkVariable] 
        private float m_Mana = 100f;

        void Update()
        {
           if(IsServer && Input.GetKeyUp(KeyCode.N))
           {
              // Automatically synchronized
              Health -= 10f;
              Mana += 10f;
           }
        }
    }
    ```

!!! tip
    You can modify the underlying **field** directly instead of the property if you don’t want automatic synchronization. To manually synchronize the modified field, simply call:
    
    - `SyncHealth(DefaultNetworkVariableOptions)`
    - `SyncMana(DefaultNetworkVariableOptions)`

    for immediate network updates.

    ???+ example "Manually Synchronized"
        ```csharp
        public partial class Player : NetworkBehaviour
        {
            [NetworkVariable] 
            private float m_Health = 100f;

            void Update()
            {
                if(IsServer && Input.GetKeyUp(KeyCode.N))
                {
                    // Manually synchronized
                    m_Health -= 10f;
                    SyncHealth();
                }
            }
        }
        ```

!!! warning
    If you modify a **field** immediately after instantiating a networked object or within `Awake()` or `Start()`, the variable will synchronize correctly. This is because, during object initialization, the server automatically sends updates for network variables to clients. However, if you modify the **property** instead of the **field** at these early stages, synchronization may fail. Property changes trigger an update message, but if the object has not yet been instantiated on the client side, the update will not be applied.

!!! bug
    Occasionally, generated code may not be recognized by the IDE’s IntelliSense (e.g., in Visual Studio). If this occurs, a simple restart of the IDE should resolve the issue.

##### Default Behaviour

!!! tip
    Use `DefaultNetworkVariableSettings` to adjust how network variables are transmitted across the network. This allows for configuring default behaviors for all network variables. For more specific control, you can use individual settings like `HealthOptions` and `ManaOptions` to customize the transmission behavior of specific variables.

    ???+ example
        ```csharp
        public partial class Player : NetworkBehaviour
        {
            [NetworkVariable] 
            private float m_Health = 100f;

            [NetworkVariable] 
            private float m_Mana = 100f;

            protected override void OnAwake()
            {
                // Change the default settings for all network variables
                DefaultNetworkVariableOptions = new()
                {
                    DeliveryMode = DeliveryMode.ReliableOrdered,
                    Target = Target.AllExceptSelf
                };

                // Change specific settings for specific network variables
                HealthOptions = new()
                {
                    DeliveryMode = DeliveryMode.ReliableOrdered,
                    Target = Target.All
                };

                ManaOptions = new()
                {
                    DeliveryMode = DeliveryMode.ReliableOrdered,
                    Target = Target.All
                };
            }
        }
        ```

---

#### Generated Methods

The `[NetworkVariable]` attribute will generate methods for each network variable, such as:

=== "Health Hook"
    ???+ example "Health Hook"
        ```csharp
        // Hook in the same script.
        partial void OnHealthChanged(float prevHealth, float nextHealth, bool isWriting)
        {
            // The isWriting parameter indicates whether the operation is writing the value to the network or reading it from the network.
        }
        
        // Hook in the derived class.
        protected override void OnBaseHealthChanged(float prevHealth, float nextHealth, bool isWriting)
        {
            // The isWriting parameter indicates whether the operation is writing the value to the network or reading it from the network.
        }
        ```

=== "Mana Hook"
    ???+ example "Mana Hook"
        ```csharp
        // Hook in the same script.
        partial void OnManaChanged(float prevMana, float nextMana, bool isWriting)
        {
            // The isWriting parameter indicates whether the operation is writing the value to the network or reading it from the network.
        }    

        // Hook in the derived class.
        protected override void OnBaseManaChanged(float prevMana, float nextMana, bool isWriting)
        {
            // The isWriting parameter indicates whether the operation is writing the value to the network or reading it from the network.
        }
        ```

- **`void SyncHealth(NetworkVariableOptions options);`**

Manually synchronizes the `m_Health` field, allowing control over when and how this field is updated across the network.

- **`void SyncMana(NetworkVariableOptions options);`** 

Manually synchronizes the `m_Mana` field.

---

## RouteX

**RouteX** is a simple simulation of `Express.js` and is one of the most useful features of the API. It can be easily used to request a route and receive a response from the server. Routes can also send responses to multiple clients beyond the one that originally requested the route.

### Registering Routes

1. Import the `RouteX` module with `using static Omni.Core.RouteX;` and `Omni` with `using Omni.Core;`
2. Register the routes on the `Awake` method or on the `Start` method, eg:

!!! Note
    `RouteX` supports both asynchronous and synchronous operations, providing flexibility for various use cases. All functions include asynchronous versions workflows. For additional overloads, detailed explanations, and further information on synchronous and asynchronous versions, consult the [`API Reference`]().

=== "Registering a Get Route"
    ???+ example
        ```csharp
        public class LoginControllerInServer : ServerBehaviour
        {
           protected override void OnAwake()
           {
              XServer.GetAsync("/login", (res) =>
              {
                  res.WriteString("Wow! You are logged in!");
                  res.Send();
              });

              XServer.GetAsync("/register", (res, peer) => // Peer argument is optional
              {
              	  res.WriteString("Ok! You are registered!");
              	  res.Send();
              });
           }
        }
        ```

=== "Registering a Post Route"
    ???+ example
        ```csharp
        public class LoginControllerInServer : ServerBehaviour
        {
           protected override void OnAwake()
           {
              XServer.PostAsync("/login", (req, res) =>
              {
                  // Read the username sent in the request
                  string username = req.ReadString();

                  // Send a response
                  res.WriteString("Wow! You are logged in as " + username);
                  res.Send();
              });

              XServer.PostAsync("/register", (req, res, peer) => // Peer argument is optional
              {
                  // Read the username sent in the request
                  string username = req.ReadString();

                  // Send a response
                  res.WriteString("Ok! You are registered!");
                  res.Send();
              });
           }
        }
        ```

### Requesting Routes

=== "Requesting a Get Route"
    ???+ example
        ```csharp
        public class LoginControllerInClient : ClientBehaviour
        {
           async void Update()
           {
              if (Input.GetKeyDown(KeyCode.R))
              {
                  using DataBuffer res = await XClient.GetAsync("/login");
                  string message = res.ReadString();
                  print(message);
              }
           }
        }
        ```  

=== "Requesting a Post Route"
    ???+ example
        ```csharp
        public class LoginControllerInClient : ClientBehaviour
        {
           async void Update()
           {
              if (Input.GetKeyDown(KeyCode.R))
              {
                  using DataBuffer res = await XClient.PostAsync("/login", req =>
                  {
                      req.WriteString("John Doe");
                  });

                  string message = res.ReadString();
                  print(message);
              }
           }
        }
        ```     

!!! info
    Omni provides the `HttpResponse` and `HttpResponse<T>` objects to streamline the process of sending responses. These objects allow you to include a status code, a message, and optionally, a payload (via the generic version). This approach offers a more organized and structured way to handle and send responses in your application.

=== "Registering a Post Route with HttpResponse"
    ???+ example
        ```csharp
        public class LoginControllerInServer : ServerBehaviour
        {
           protected override void OnAwake()
           {
              XServer.PostAsync("/login", (req, res) =>
              {
                  // Read the username sent in the request
                  string username = req.ReadString();

                  // Send a response with HttpResponse
                  res.WriteHttpResponse(new HttpResponse()
                  {
                  	 StatusCode = StatusCode.Success,
                  	 StatusMessage = $"Login successful, Hello {username}!",
                  });

                  res.Send();
              });

              XServer.PostAsync("/getinfo", (req, res) =>
              {
                  // Read the username sent in the request
                  string username = req.ReadString();

                  // Send a response with HttpResponse and payload
                  res.WriteHttpResponse(new HttpResponse<Player>()
                  {
                  	 StatusCode = StatusCode.Success,
                  	 StatusMessage = $"Login successful, Hello {username}!",
                     Result = new Player()
                  });

                  res.Send();
              });
           }
        }
        ```

=== "Requesting a Post Route with HttpResponse"
    ???+ example
        ```csharp
        public class LoginControllerInClient : ClientBehaviour
        {
           async void Update()
           {
              if (Input.GetKeyDown(KeyCode.R))
              {
                  using DataBuffer res = await XClient.PostAsync("/login", req =>
                  {
                      req.WriteString("John Doe");
                  });

                  var response = res.ReadHttpResponse();
                  print(response.StatusMessage);
              }

              if (Input.GetKeyDown(KeyCode.G))
              {
                  using DataBuffer res = await XClient.PostAsync("/getinfo", req =>
                  {
                      req.WriteString("John Doe");
                  });

                  var response = res.ReadHttpResponse<Player>();
	              print("Player name: " + response.Result.Name);
              }
           }
        }
        ``` 

For more details, refer to the [API Reference](#api-reference).

---

## Serialization and Deserialization

Omni supports serialization of a wide range of data types, including primitives, complex classes, structs, dictionaries, and more, providing unmatched flexibility for networked data structures. Omni offers two serialization methods: JSON-based serialization for readability and compatibility, and binary-based serialization for optimized performance and minimized data size.

With Omni, **everything is serializable**. All network operations utilize the `DataBuffer` object, a dedicated data buffer that efficiently handles data preparation and transmission across the network, ensuring seamless and effective communication.

!!! info
    The `DataBuffer` is the core of all Omni operations. It is used universally across RPCs, RouteX, custom messages, and other network features. Understanding how to manage and utilize `DataBuffer` is essential for working effectively with Omni.

!!! danger
    As a binary serializer, `DataBuffer` requires that the order of reading matches the order of writing precisely. Any discrepancy in the read/write sequence can lead to corrupted or unexpected data. Developers should ensure consistency and adherence to the defined structure when serializing and deserializing data with `DataBuffer`.

!!! info
    The `DataBuffer` functions similarly to a combination of [`MemoryStream`](https://learn.microsoft.com/en-us/dotnet/api/system.io.memorystream?view=net-8.0) and [`BinaryWriter`](https://learn.microsoft.com/en-us/dotnet/api/system.io.binarywriter?view=net-8.0) and [`BinaryReader`](https://learn.microsoft.com/en-us/dotnet/api/system.io.binaryreader?view=net-8.0). It includes comparable properties and features, such as `Position`, enabling developers to efficiently manage and navigate the buffer while performing read and write operations.

---

### Primitives

Omni’s `DataBuffer` provides efficient support for primitive types, allowing direct serialization of commonly used data types such as integers, floats, and booleans. This simplifies network data handling by enabling fast read and write operations for foundational data types.

Using these primitives, Omni ensures minimal overhead in data serialization, making it suitable for high-performance networking where lightweight data handling is essential. Primitive types can be written to or read from the `DataBuffer` in a straightforward manner, supporting rapid data transmission across client-server boundaries.

=== "Writing Primitives"
    ???+ example "Writing Primitives"
        ```csharp
        void Example()
        {
           DataBuffer message = new DataBuffer();
           message.Write(10); // Writes an integer value to the buffer
           message.Write(3.14f); // Writes a floating-point value to the buffer
           message.Write(true); // Writes a boolean value to the buffer
        }
        ```

=== "Reading Primitives"
    ???+ example "Reading Primitives"
        ```csharp
        void Example()
        {
           DataBuffer message = GetHypotheticalValidDataBuffer();
           int num = message.Read<int>(); // Reads an integer value from the buffer
           float f = message.Read<float>(); // Reads a floating-point value from the buffer
           bool b = message.Read<bool>(); // Reads a boolean value from the buffer
        }
        ```

---

### Complex Types

Omni supports the serialization of complex types using [`Newtonsoft.JSON`](https://docs.unity3d.com/Packages/com.unity.nuget.newtonsoft-json@3.2/manual/index.html) or [`MemoryPack`](https://github.com/Cysharp/MemoryPack). For objects and data structures that go beyond primitive types, JSON serialization provides a readable, flexible format ideal for compatibility with third-party systems, while `MemoryPack` enables efficient binary serialization for high-performance network transfers.

Using these serialization methods, Omni can seamlessly handle complex data types, such as custom `structs`, `classes`, `dictionaries`, and `nested structures`, ensuring that all necessary data is transmitted effectively and accurately across the network.

=== "JSON Serialization"
    ???+ example "JSON Serialization"
        ```csharp
        public class Player
        {
            public string name;
            public int score;
            public Dictionary<string, int> inventory;
        }

        void Example()
        {
           Player player = new Player();
   
           // Serialize the player object
           DataBuffer message = new DataBuffer();
           message.WriteAsJson(player);
        }
        ```

=== "MemoryPack Serialization"
    ???+ example "MemoryPack Serialization"
        ```csharp
        [MemoryPackable]
        public partial class Player
        {
            public string name;
            public int score;
            public Dictionary<string, int> inventory;
        }

        void Example()
        {
           Player player = new Player();
   
           // Serialize the player object
           DataBuffer message = new DataBuffer();
           message.WriteAsBinary(player);
        }
        ```

=== "JSON Deserialization"
    ???+ example "JSON Deserialization"
        ```csharp
        public class Player
        {
            public string name;
            public int score;
            public Dictionary<string, int> inventory;
        }

        void Example()
        {
           DataBuffer message = GetHypotheticalValidDataBuffer();
           Player player = message.ReadAsJson<Player>(); // Deserializes to a Player object
        }
        ```

=== "MemoryPack Deserialization"
    ???+ example "MemoryPack Deserialization"
        ```csharp
        [MemoryPackable]
        public partial class Player
        {
            public string name;
            public int score;
            public Dictionary<string, int> inventory;
        }

        void Example()
        {
           DataBuffer message = GetHypotheticalValidDataBuffer();
           Player player = message.ReadAsBinary<Player>(); // Deserializes to a Player object
        }
        ```

!!! note
    See the [`Newtonsoft.JSON`](https://docs.unity3d.com/Packages/com.unity.nuget.newtonsoft-json@3.2/manual/index.html) or [`MemoryPack`](https://github.com/Cysharp/MemoryPack) documentation for more information about using **annotation attributes** to customize serialization and deserialization.

!!! info
    When sending a `DataBuffer`, you will always receive a `DataBuffer` in response; it is not possible to send and receive data in any other way without using a `DataBuffer`, as all operations utilize it internally. **You must also ensure that the reading and writing occur in the same order.**

---

### Compression

The `DataBuffer` object in Omni supports efficient data compression, utilizing the `Brotli` and `LZ4` algorithms. These algorithms are designed to optimize network performance by reducing data size without significant overhead, ensuring faster transmission and lower bandwidth usage.

- **Brotli**: A highly efficient compression algorithm ideal for scenarios where maximum compression is needed, offering significant size reduction for complex or large datasets.
- **LZ4**: Focused on speed, `LZ4` provides fast compression and decompression, making it suitable for real-time applications that prioritize performance over compression ratio.

With these options, Omni allows developers to tailor data compression to their specific needs, balancing speed and efficiency for various network scenarios.

=== "Compression"
    ???+ example "Compression"
        ```csharp
        void Example()
        {
           DataBuffer message = new DataBuffer();
           message.WriteAsJson(GetHypotheticalLargePlayerObject());

           // Compress the data
           message.CompressRaw(); // Compress the current buffer
        }
        ```

=== "Decompression"
    ???+ example "Decompression"
        ```csharp
        void Example()
        {
           DataBuffer message = GetHypotheticalCompressedDataBuffer();

           // Decompress the data
           message.DecompressRaw(); // Decompress the current buffer

           // Read the decompressed data
           Player player = message.ReadAsJson<Player>();
        }
        ```

---

### Cryptography

Omni employs `AES` encryption to secure data buffers, ensuring that sensitive information remains protected during network transmission. The cryptographic system is designed with flexibility and security in mind, offering both peer-specific and global encryption keys to handle various scenarios.

Peer-Specific Encryption

Each `peer` in the network is assigned its own unique encryption key. When a client (e.g., Client A) sends a message using its key, only that client can decrypt the message. This ensures a high level of security, as no other client can access the encrypted data. Peer-specific encryption is ideal for situations where private communication or data integrity is paramount.

!!! info
    Encryption keys are exchanged between the client and server using `RSA`, a robust public-key cryptography algorithm. This ensures that the `AES` keys used for data encryption remain secure during transmission, as only the intended recipient can decrypt the exchanged keys. By combining `RSA` for key exchange with `AES` for data encryption, Omni provides a highly secure and efficient cryptographic system for multiplayer environments.

Global Server Key

In addition to peer-specific keys, Omni provides a global server encryption key. Unlike peer-specific keys, the global key can be used to encrypt and decrypt any data, including messages originating from other clients. This global key is managed by the server and allows for seamless handling of shared data, such as broadcasted messages or server-wide updates. It provides a flexible option for scenarios where universal decryption is required without compromising security.

Key Features

- **AES Encryption**: Omni uses the `Advanced Encryption Standard (AES)` to ensure robust protection against unauthorized access.
- **Peer-Specific Keys**: Restrict decryption to the originating peer, enhancing data privacy.
- **Global Server Key**: Enable decryption of any data within the network, facilitating shared communication and server-driven operations.
- **Flexibility**: The dual-key system allows developers to tailor encryption strategies to the needs of their application, balancing security and convenience.

Omni's cryptography framework ensures that all data transmitted across the network is secure, whether it's private peer-to-peer communication or broadcasted messages. By combining peer-specific encryption with a global server key, Omni provides a powerful and flexible system for managing encrypted data in multiplayer environments.

=== "Encryption"
    ???+ example "Encryption"
        ```csharp
        void Example()
        {
           DataBuffer message = new DataBuffer();
           message.WriteAsJson(GetHypotheticalPlayerObject());

           // Encrypt the data
           message.EncryptRaw(NetworkManager.SharedPeer); // Use `ServerPeer` - Global Encryption Key

           // e.g. Encrypt the data with a peer-specific key in client side...
           // message.EncryptRaw(NetworkManager.LocalPeer);
        }
        ```
 
=== "Decryption"
    ???+ example "Decryption"
        ```csharp
        void Example()
        {
           DataBuffer message = GetHypotheticalEncryptedDataBuffer();

           // Decrypt the data
           message.DecryptRaw(NetworkManager.SharedPeer); // Use `ServerPeer` - Global Encryption Key

           // e.g. Decrypt the data with a peer-specific key in server side...
           // message.DecryptRaw(peer);

           // Read the decrypted data
           Player player = message.ReadAsJson<Player>();
        }
        ```

See the [`API Reference`]() for more information about the `DataBuffer` and its usage.

---

## IMessage Interface

This interface, `IMessage`, can be implemented to customize the serialization and deserialization of a type when used within an RPC or `NetworkVariable`. By implementing `IMessage`, you define how data is written to and read from a `DataBuffer`, enabling greater control over data structure and format.

The `IMessageWithPeer` interface extends `IMessage` to include additional properties, such as `SharedPeer` and `IsServer`, which are useful for managing encryption and authentication in networked communications. This extension provides enhanced flexibility for handling secure and authenticated messaging between server and client.

???+ example
    ```csharp
       public class PlayerStruct : IMessage
       {
       	  private string m_Name;
       	  private Vector3 m_Position;
       	  private int m_Health;
       	  private int m_Mana;

       	  public void Serialize(DataBuffer writer)
       	  {
       	  	writer.WriteString(m_Name);
       	  	writer.Write(m_Position);
       	  	writer.Write(m_Health);
       	  	writer.Write(m_Mana);
       	  }
         
       	  public void Deserialize(DataBuffer reader)
       	  {
       	  	m_Name = reader.ReadString();
       	  	m_Position = reader.Read<Vector3>();
       	  	m_Health = reader.Read<int>();
       	  	m_Mana = reader.Read<int>();
       	  }
       }
    ```

=== "Network Variable"
    ???+ example "Network Variable"
        ```csharp
           public partial class Player : NetworkBehaviour
           {
           	  [NetworkVariable] 
           	  private PlayerStruct m_PlayerStruct = new PlayerStruct();
           }
        ```

=== "RPC"
    ???+ example "RPC"
        ```csharp
           public class Player : NetworkBehaviour
           {
               private PlayerStruct m_PlayerStruct = new PlayerStruct();
    
               void Update()
               {
                  if(IsServer)
                  {
                    // Send an RPC from the server to the client
                    Remote.Invoke(1, m_PlayerStruct, new()
                    {
                    	DeliveryMode = DeliveryMode.Unreliable
                    });
                  }
               }
    
               [Client(1)]
               void Example(DataBuffer message) // Signature 2
               {
               	  m_PlayerStruct = message.Deserialize<PlayerStruct>();
    
                   // Alternative:
                  // Populate an existing object
                  // message.Populate(m_PlayerStruct);
               }
           }
        ```
