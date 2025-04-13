# Network Identity

The `NetworkIdentity` class is a fundamental component of the **Omni Networking** framework for Unity, playing a role similar to that of `NetworkIdentity` in frameworks like **Mirror** and **Photon Unity Networking (PUN)**.

It uniquely identifies a GameObject across the network and links it to the networking system, enabling synchronization between clients and servers. Every networked GameObject must have a `NetworkIdentity` to participate in the networking system.

Key responsibilities of the `NetworkIdentity` include:

- Assigning and maintaining a unique network ID for each object.
- Managing network authority (server-authoritative or client-authoritative).
- Enabling the replication of state and data between clients and the server.
- Supporting remote procedure calls (RPCs) and network variables.
- Handling spawning, destruction, and visibility of objects across the network.

This class acts as the foundation for any GameObject that needs to be synchronized or controlled in a multiplayer environment. Without it, objects cannot be managed by the networking system.

> ⚠️ Note: Only one `NetworkIdentity` should exist per networked GameObject.

---

## Methods

### Get

Retrieves a service instance by its name from the service locator.

- **Signature**: 
  - `public T Get<T>(string serviceName) where T : class`
  - `public T Get<T>() where T : class`

Description

The `Get` method retrieves a service instance from the service locator. It provides two overloads: one that accepts a service name and another that uses the type name as the service name. If the service is not found or cannot be cast to the specified type, an exception is thrown.

Parameters

| Parameter     | Type     | Description                                    |
|--------------|----------|------------------------------------------------|
| `serviceName`| `string` | The name of the service to retrieve (optional). |

Returns

- **`T`**: The service instance cast to the specified type.

Exceptions

- Throws an `Exception` if the service is not found.
- Throws an `Exception` if the service cannot be cast to the specified type.

???+ example
    ```csharp
    // Example of retrieving a service by name
    IMyService service = networkIdentity.Get<IMyService>("MyService");
    
    // Example of retrieving a service using type name
    IMyService service = networkIdentity.Get<IMyService>();
    ```

---

### TryGet

Attempts to retrieve a service instance from the service locator.

- **Signature**: 
  - `public bool TryGet<T>(string serviceName, out T service) where T : class`
  - `public bool TryGet<T>(out T service) where T : class`

Description

The `TryGet` method attempts to retrieve a service instance from the service locator. It provides a safe way to access services without throwing exceptions if the service is not found or cannot be cast to the specified type.

Parameters

| Parameter     | Type     | Description                                    |
|--------------|----------|------------------------------------------------|
| `serviceName`| `string` | The name of the service to retrieve (optional). |
| `service`    | `T`      | The retrieved service instance (output parameter). |

Returns

- **`bool`**: `true` if the service is found and successfully cast to the specified type; otherwise, `false`.

???+ example
    ```csharp
    // Example of trying to retrieve a service by name
    if (networkIdentity.TryGet<IMyService>("MyService", out var service))
    {
        // Use the service
    }
    
    // Example of trying to retrieve a service using type name
    if (networkIdentity.TryGet<IMyService>(out var service))
    {
        // Use the service
    }
    ```

---

### Register

Registers a new service instance in the service locator.

- **Signature**: 
  - `public void Register<T>(T service) where T : class`
  - `public void Register<T>(T service, string serviceName) where T : class`

Description

The `Register` method adds a new service instance to the service locator. It provides two overloads: one that uses the type name as the service name and another that accepts a custom service name.

Parameters

| Parameter     | Type     | Description                                    |
|--------------|----------|------------------------------------------------|
| `service`    | `T`      | The service instance to register.              |
| `serviceName`| `string` | The name to associate with the service (optional). |

Exceptions

- Throws an `Exception` if a service with the same name already exists.

???+ example
    ```csharp
    // Example of registering a service using type name
    MyService service = new MyService();
    networkIdentity.Register(service);
    
    // Example of registering a service with a custom name
    networkIdentity.Register(service, "CustomServiceName");
    ```

---

### TryRegister

Attempts to register a new service instance in the service locator.

- **Signature**: 
  - `public bool TryRegister<T>(T service) where T : class`
  - `public bool TryRegister<T>(T service, string serviceName) where T : class`

Description

The `TryRegister` method attempts to add a new service instance to the service locator. Unlike `Register`, this method returns a boolean indicating success or failure instead of throwing an exception.

Parameters

| Parameter     | Type     | Description                                    |
|--------------|----------|------------------------------------------------|
| `service`    | `T`      | The service instance to register.              |
| `serviceName`| `string` | The name to associate with the service (optional). |

Returns

- **`bool`**: `true` if the service was successfully registered; otherwise, `false`.

???+ example
    ```csharp
    // Example of trying to register a service using type name
    MyService service = new MyService();
    if (networkIdentity.TryRegister(service))
    {
        // Service registered successfully
    }
    
    // Example of trying to register a service with a custom name
    if (networkIdentity.TryRegister(service, "CustomServiceName"))
    {
        // Service registered successfully
    }
    ```

---

### UpdateService

Updates an existing service instance in the service locator.

- **Signature**: 
  - `public void UpdateService<T>(T service) where T : class`
  - `public void UpdateService<T>(T service, string serviceName) where T : class`

Description

The `UpdateService` method updates an existing service instance in the service locator. If the service doesn't exist, an exception is thrown.

Parameters

| Parameter     | Type     | Description                                    |
|--------------|----------|------------------------------------------------|
| `service`    | `T`      | The new service instance.                      |
| `serviceName`| `string` | The name of the service to update (optional).  |

Exceptions

- Throws an `Exception` if the service does not exist.

???+ example
    ```csharp
    // Example of updating a service using type name
    MyService updatedService = new MyService();
    networkIdentity.UpdateService(updatedService);
    
    // Example of updating a service with a custom name
    networkIdentity.UpdateService(updatedService, "CustomServiceName");
    ```

---

### TryUpdateService

Attempts to update an existing service instance in the service locator.

- **Signature**: 
  - `public bool TryUpdateService<T>(T service) where T : class`
  - `public bool TryUpdateService<T>(T service, string serviceName) where T : class`

Description

The `TryUpdateService` method attempts to update an existing service instance in the service locator. Returns a boolean indicating success or failure instead of throwing an exception.

Parameters

| Parameter     | Type     | Description                                    |
|--------------|----------|------------------------------------------------|
| `service`    | `T`      | The new service instance.                      |
| `serviceName`| `string` | The name of the service to update (optional).  |

Returns

- **`bool`**: `true` if the service was successfully updated; otherwise, `false`.

???+ example
    ```csharp
    // Example of trying to update a service using type name
    MyService updatedService = new MyService();
    if (networkIdentity.TryUpdateService(updatedService))
    {
        // Service updated successfully
    }
    
    // Example of trying to update a service with a custom name
    if (networkIdentity.TryUpdateService(updatedService, "CustomServiceName"))
    {
        // Service updated successfully
    }
    ```

---

### Unregister

Removes a service from the service locator.

- **Signature**: 
  - `public bool Unregister<T>() where T : class`
  - `public bool Unregister(string serviceName)`

Description

The `Unregister` method removes a service from the service locator. It can be called using either the type name or a custom service name.

Parameters

| Parameter     | Type     | Description                                    |
|--------------|----------|------------------------------------------------|
| `serviceName`| `string` | The name of the service to remove (optional).  |

Returns

- **`bool`**: `true` if the service was successfully removed; otherwise, `false`.

???+ example
    ```csharp
    // Example of unregistering a service using type name
    networkIdentity.Unregister<IMyService>();
    
    // Example of unregistering a service with a custom name
    networkIdentity.Unregister("CustomServiceName");
    ```

---

### Exists

Checks if a service exists in the service locator.

- **Signature**: 
  - `public bool Exists<T>() where T : class`
  - `public bool Exists(string serviceName)`

Description

The `Exists` method checks whether a service exists in the service locator. It can be called using either the type name or a custom service name.

Parameters

| Parameter     | Type     | Description                                    |
|--------------|----------|------------------------------------------------|
| `serviceName`| `string` | The name of the service to check (optional).   |

Returns

- **`bool`**: `true` if the service exists; otherwise, `false`.

???+ example
    ```csharp
    // Example of checking if a service exists using type name
    if (networkIdentity.Exists<IMyService>())
    {
        // Service exists
    }
    
    // Example of checking if a service exists with a custom name
    if (networkIdentity.Exists("CustomServiceName"))
    {
        // Service exists
    }
    ```

---

### GetAsComponent

Retrieves a Unity MonoBehaviour component that was assigned to a NetworkComponentService in the inspector.

- **Signature**: 
  - `public void GetAsComponent<T>(out T component) where T : class`
  - `public void GetAsComponent<T>(string componentName, out T component) where T : class`
  - `public T GetAsComponent<T>() where T : class`
  - `public T GetAsComponent<T>(string componentName) where T : class`

Description

The `GetAsComponent` method retrieves a Unity MonoBehaviour component that was previously assigned to a NetworkComponentService in the inspector. This allows for network-aware access to Unity components like Rigidbody, Transform, or custom MonoBehaviours.

Parameters

| Parameter       | Type     | Description                                                           |
|----------------|----------|-----------------------------------------------------------------------|
| `componentName`| `string` | The name of the NetworkComponentService (optional, defaults to T name).|
| `component`    | `T`      | The retrieved Unity component (output parameter).                      |

Returns

- **`T`**: The Unity component instance (for non-void overloads).

???+ example
    ```csharp
    // Example of retrieving a Rigidbody component
    networkIdentity.GetAsComponent<Rigidbody>(out var rb);
    
    // Example of retrieving a Transform component with direct return
    var transform = networkIdentity.GetAsComponent<Transform>();
    
    // Example with custom NetworkComponentService name
    var animator = networkIdentity.GetAsComponent<Animator>("PlayerAnimator");
    ```

---

### TryGetAsComponent

Attempts to retrieve a Unity MonoBehaviour component that was assigned to a NetworkComponentService in the inspector.

- **Signature**: 
  - `public bool TryGetAsComponent<T>(out T component) where T : class`
  - `public bool TryGetAsComponent<T>(string componentName, out T component) where T : class`

Description

The `TryGetAsComponent` method attempts to retrieve a Unity MonoBehaviour component that was previously assigned to a NetworkComponentService in the inspector. It provides a safe way to access components without throwing exceptions if the component is not found or not properly assigned.

Parameters

| Parameter       | Type     | Description                                                           |
|----------------|----------|-----------------------------------------------------------------------|
| `componentName`| `string` | The name of the NetworkComponentService (optional, defaults to T name).|
| `component`    | `T`      | The retrieved Unity component (output parameter).                      |

Returns

- **`bool`**: `true` if the component was successfully retrieved; otherwise, `false`.

???+ example
    ```csharp
    // Example of safely retrieving a Rigidbody component
    if (networkIdentity.TryGetAsComponent<Rigidbody>(out var rb))
    {
        rb.AddForce(Vector3.up);
    }
    
    // Example with custom NetworkComponentService name
    if (networkIdentity.TryGetAsComponent<Animator>("PlayerAnimator", out var animator))
    {
        animator.SetTrigger("Jump");
    }
    ```

---

### GetAsGameObject

Retrieves the GameObject that contains a component assigned to a NetworkComponentService.

- **Signature**: 
  - `public GameObject GetAsGameObject<T>()`
  - `public GameObject GetAsGameObject(string gameObjectName)`

Description

The `GetAsGameObject` method retrieves the GameObject that contains a component that was previously assigned to a NetworkComponentService in the inspector. This is useful when you need to access the GameObject that holds a specific networked component.

Parameters

| Parameter        | Type     | Description                                                           |
|-----------------|----------|-----------------------------------------------------------------------|
| `gameObjectName`| `string` | The name of the NetworkComponentService (optional, defaults to T name).|

Returns

- **`GameObject`**: The GameObject containing the specified component.

???+ example
    ```csharp
    // Example of retrieving the GameObject containing a specific component
    GameObject rbObject = networkIdentity.GetAsGameObject<Rigidbody>();
    
    // Example of retrieving the GameObject using NetworkComponentService name
    GameObject animatorObject = networkIdentity.GetAsGameObject("PlayerAnimator");
    ```

---

### TryGetAsGameObject

Attempts to retrieve the GameObject that contains a component assigned to a NetworkComponentService.

- **Signature**: 
  - `public bool TryGetAsGameObject<T>(out GameObject gameObject)`
  - `public bool TryGetAsGameObject(string gameObjectName, out GameObject gameObject)`

Description

The `TryGetAsGameObject` method attempts to retrieve the GameObject that contains a component that was previously assigned to a NetworkComponentService in the inspector. It provides a safe way to access GameObjects without throwing exceptions if the component or GameObject is not found.

Parameters

| Parameter        | Type         | Description                                                           |
|-----------------|--------------|-----------------------------------------------------------------------|
| `gameObjectName`| `string`     | The name of the NetworkComponentService (optional, defaults to T name).|
| `gameObject`    | `GameObject` | The retrieved GameObject (output parameter).                          |

Returns

- **`bool`**: `true` if the GameObject was successfully retrieved; otherwise, `false`.

???+ example
    ```csharp
    // Example of safely retrieving the GameObject containing a Rigidbody
    if (networkIdentity.TryGetAsGameObject<Rigidbody>(out GameObject rbObject))
    {
        rbObject.SetActive(true);
    }
    
    // Example of safely retrieving the GameObject using NetworkComponentService name
    if (networkIdentity.TryGetAsGameObject("PlayerAnimator", out GameObject animatorObject))
    {
        animatorObject.SetActive(true);
    }
    ```

---

### SpawnOnClient

Spawns the network identity on the client.

- **Signature**: 
  - `public void SpawnOnClient(ServerOptions options)`
  - `public void SpawnOnClient(Target target = Target.Auto, DeliveryMode deliveryMode = DeliveryMode.ReliableOrdered, int groupId = 0, DataCache dataCache = default, byte sequenceChannel = 0)`

Description

The `SpawnOnClient` method spawns the network identity on connected clients. It can be called with either `ServerOptions` or individual parameters for customization.

Parameters

| Parameter       | Type           | Description                                                |
|----------------|----------------|------------------------------------------------------------|
| `options`      | `ServerOptions`| Options for spawning the network identity.                 |
| `target`       | `Target`       | The target clients for spawning (default is Auto).         |
| `deliveryMode` | `DeliveryMode` | The delivery mode for the spawn message.                   |
| `groupId`      | `int`          | The group ID for the spawn message.                        |
| `dataCache`    | `DataCache`    | The data cache options for the spawn message.              |
| `sequenceChannel`| `byte`       | The sequence channel for message ordering.                 |

???+ example
    ```csharp
    // Example using ServerOptions
    ServerOptions options = new ServerOptions
    {
        Target = Target.All,
        DeliveryMode = DeliveryMode.ReliableOrdered
    };
    networkIdentity.SpawnOnClient(options);
    
    // Example using individual parameters
    networkIdentity.SpawnOnClient(Target.All, DeliveryMode.ReliableOrdered);
    ```

---

### Despawn

Despawns the network identity from clients.

- **Signature**: 
  - `public void Despawn(ServerOptions options)`
  - `public void Despawn(Target target = Target.Auto, DeliveryMode deliveryMode = DeliveryMode.ReliableOrdered, int groupId = 0, DataCache dataCache = default, byte sequenceChannel = 0)`

Description

The `Despawn` method removes the network identity from connected clients. It can be called with either `ServerOptions` or individual parameters for customization.

Parameters

| Parameter       | Type           | Description                                                |
|----------------|----------------|------------------------------------------------------------|
| `options`      | `ServerOptions`| Options for despawning the network identity.               |
| `target`       | `Target`       | The target clients for despawning (default is Auto).       |
| `deliveryMode` | `DeliveryMode` | The delivery mode for the despawn message.                 |
| `groupId`      | `int`          | The group ID for the despawn message.                      |
| `dataCache`    | `DataCache`    | The data cache options for the despawn message.            |
| `sequenceChannel`| `byte`       | The sequence channel for message ordering.                 |

???+ example
    ```csharp
    // Example using ServerOptions
    ServerOptions options = new ServerOptions
    {
        Target = Target.All,
        DeliveryMode = DeliveryMode.ReliableOrdered
    };
    networkIdentity.Despawn(options);
    
    // Example using individual parameters
    networkIdentity.Despawn(Target.All, DeliveryMode.ReliableOrdered);
    ```

---

### DespawnToPeer

Despawns the network identity for a specific peer.

- **Signature**: 
  - `public void DespawnToPeer(NetworkPeer peer, ServerOptions options)`
  - `public void DespawnToPeer(NetworkPeer peer, DeliveryMode deliveryMode = DeliveryMode.ReliableOrdered, DataCache dataCache = default, byte sequenceChannel = 0)`

Description

The `DespawnToPeer` method removes the network identity for a specific peer. It can be called with either `ServerOptions` or individual parameters for customization.

Parameters

| Parameter       | Type           | Description                                                |
|----------------|----------------|------------------------------------------------------------|
| `peer`         | `NetworkPeer`  | The peer to despawn the network identity for.              |
| `options`      | `ServerOptions`| Options for despawning the network identity.               |
| `deliveryMode` | `DeliveryMode` | The delivery mode for the despawn message.                 |
| `dataCache`    | `DataCache`    | The data cache options for the despawn message.            |
| `sequenceChannel`| `byte`       | The sequence channel for message ordering.                 |

???+ example
    ```csharp
    // Example using ServerOptions
    ServerOptions options = new ServerOptions
    {
        DeliveryMode = DeliveryMode.ReliableOrdered
    };
    networkIdentity.DespawnToPeer(somePeer, options);
    
    // Example using individual parameters
    networkIdentity.DespawnToPeer(somePeer, DeliveryMode.ReliableOrdered);
    ```

---

### DestroyOnClient

Destroys the network identity on the client only.

- **Signature**: `public void DestroyOnClient()`

Description

The `DestroyOnClient` method destroys the network identity locally on the client side only. This action does not affect other clients or the server.

???+ example
    ```csharp
    // Example of destroying a network identity on the client
    networkIdentity.DestroyOnClient();
    ```

---

### DestroyOnServer

Destroys the network identity on the server only.

- **Signature**: `public void DestroyOnServer()`

Description

The `DestroyOnServer` method destroys the network identity locally on the server side only. This action does not affect clients.

???+ example
    ```csharp
    // Example of destroying a network identity on the server
    networkIdentity.DestroyOnServer();
    ```

---

### Destroy

Destroys the network identity based on the current context (client or server).

- **Signature**: `public void Destroy()`

Description

The `Destroy` method destroys the network identity either on the client or server, depending on where it is called. It automatically determines the appropriate destroy method to call based on the current context.

???+ example
    ```csharp
    // Example of destroying a network identity
    networkIdentity.Destroy();
    ```

---

### AddNetworkComponent

Adds a network component to the network identity.

- **Signature**: `public T AddNetworkComponent<T>(string serviceName = null) where T : NetworkBehaviour`

Description

The `AddNetworkComponent` method adds a new network component to the network identity and initializes it with the necessary network settings.

Parameters

| Parameter     | Type     | Description                                                |
|--------------|----------|------------------------------------------------------------|
| `serviceName`| `string` | Optional name for the service (defaults to type name).     |

Returns

- **`T`**: The newly added network component.

???+ example
    ```csharp
    // Example of adding a network component
    MyNetworkComponent component = networkIdentity.AddNetworkComponent<MyNetworkComponent>();
    
    // Example with custom service name
    MyNetworkComponent component = networkIdentity.AddNetworkComponent<MyNetworkComponent>("CustomName");
    ```

---

### SetOwner

Sets the owner of the network identity.

- **Signature**: `public void SetOwner(NetworkPeer peer, Target target = Target.Auto, DeliveryMode deliveryMode = DeliveryMode.ReliableOrdered, int groupId = 0, DataCache dataCache = default, byte sequenceChannel = 0)`

Description

The `SetOwner` method changes the ownership of the network identity to the specified peer.

Parameters

| Parameter       | Type           | Description                                                |
|----------------|----------------|------------------------------------------------------------|
| `peer`         | `NetworkPeer`  | The new owner of the network identity.                     |
| `target`       | `Target`       | The target for the ownership change message.               |
| `deliveryMode` | `DeliveryMode` | The delivery mode for the ownership change message.        |
| `groupId`      | `int`          | The group ID for the ownership change message.             |
| `dataCache`    | `DataCache`    | The data cache options for the ownership change message.   |
| `sequenceChannel`| `byte`       | The sequence channel for message ordering.                 |

???+ example
    ```csharp
    // Example of setting the owner of a network identity
    networkIdentity.SetOwner(newOwnerPeer);
    
    // Example with custom parameters
    networkIdentity.SetOwner(newOwnerPeer, Target.All, DeliveryMode.ReliableOrdered);
    ```

---

### RequestAction

Requests an action to be performed on the server-side entity.

- **Signature**: `public void RequestAction(DataBuffer data = default, DeliveryMode deliveryMode = DeliveryMode.ReliableOrdered, byte sequenceChannel = 0)`

Description

The `RequestAction` method sends a request from the client to perform an action on the server-side entity.

Parameters

| Parameter       | Type           | Description                                                |
|----------------|----------------|------------------------------------------------------------|
| `data`         | `DataBuffer`   | The data buffer containing action parameters.              |
| `deliveryMode` | `DeliveryMode` | The delivery mode for the action request.                  |
| `sequenceChannel`| `byte`       | The sequence channel for message ordering.                 |

???+ example
    ```csharp
    // Example of requesting an action
    using (var data = NetworkManager.Pool.Rent())
    {
        data.Write("action_data");
        networkIdentity.RequestAction(data);
    }
    
    // Example with custom delivery mode
    networkIdentity.RequestAction(null, DeliveryMode.Unreliable);
    ```

---

## Properties

### Id

Gets or sets the unique identifier for this network identity.

- **Signature**: `public int Id { get; internal set; }`

Description

The `Id` property represents a unique identifier assigned to each `NetworkIdentity` instance in the network. This ID is used to track and reference specific network objects across the server and clients.

Returns

- **`int`**: The unique identifier of this network identity.

???+ example
    ```csharp
    // Example of accessing a NetworkIdentity's ID
    NetworkIdentity identity = GetComponent<NetworkIdentity>();
    Debug.Log($"Network Identity ID: {identity.Id}");
    ```

---

### LocalPlayer

Gets or sets the local player instance. This property is only set on the client side.

- **Signature**: `public static NetworkIdentity LocalPlayer { get; internal set; }`

Description

The `LocalPlayer` property provides access to the `NetworkIdentity` instance representing the local player in the game. This static property is only set on the client side and represents the player's own character or entity.

Returns

- **`NetworkIdentity`**: The network identity instance representing the local player.

???+ example
    ```csharp
    // Example of accessing the local player
    NetworkIdentity localPlayer = NetworkIdentity.LocalPlayer;
    if (localPlayer != null)
    {
        Debug.Log($"Local player ID: {localPlayer.Id}");
    }
    ```

---

### Owner

Gets or sets the owner of this object, available on both server and client side.

- **Signature**: `public NetworkPeer Owner { get; internal set; }`

Description

The `Owner` property represents the `NetworkPeer` that owns this network identity. On the client side, only a few properties of the owner are accessible, such as `SharedData`. This property is crucial for determining authority and permissions over the network object.

Returns

- **`NetworkPeer`**: The peer that owns this network identity.

???+ example
    ```csharp
    // Example of checking the owner of a NetworkIdentity
    NetworkIdentity identity = GetComponent<NetworkIdentity>();
    if (identity.Owner != null)
    {
        Debug.Log($"Object owned by peer ID: {identity.Owner.Id}");
    }
    ```

---

### IsServer

Gets or sets whether this object is obtained from the server side or checked on the client side.

- **Signature**: `public bool IsServer { get; internal set; }`

Description

The `IsServer` property indicates whether this network identity is operating on the server side. When `true`, the object has server authority and can perform server-specific operations. When `false`, the object is operating on the client side.

Returns

- **`bool`**: `true` if the object is on the server side; otherwise, `false`.

???+ example
    ```csharp
    // Example of checking if a NetworkIdentity is on the server
    NetworkIdentity identity = GetComponent<NetworkIdentity>();
    if (identity.IsServer)
    {
        Debug.Log("This object is running on the server");
    }
    ```

---

### IsClient

Gets a value indicating whether this instance is on the client side.

- **Signature**: `public bool IsClient => !IsServer`

Description

The `IsClient` property indicates whether this network identity is operating on the client side. This is the inverse of `IsServer` and is used to determine client-side behavior and permissions.

Returns

- **`bool`**: `true` if this instance is on the client side; otherwise, `false`.

???+ example
    ```csharp
    // Example of checking if a NetworkIdentity is on the client
    NetworkIdentity identity = GetComponent<NetworkIdentity>();
    if (identity.IsClient)
    {
        Debug.Log("This object is running on the client");
    }
    ```

---

### IsLocalPlayer

Gets or sets whether this object is owned by the local player.

- **Signature**: `public bool IsLocalPlayer { get; internal set; }`

Description

The `IsLocalPlayer` property indicates whether this network identity represents the local player's object. This is useful for determining if the object should respond to local input or have special rendering/behavior for the local player.

Returns

- **`bool`**: `true` if this object is owned by the local player; otherwise, `false`.

???+ example
    ```csharp
    // Example of checking if a NetworkIdentity is the local player
    NetworkIdentity identity = GetComponent<NetworkIdentity>();
    if (identity.IsLocalPlayer)
    {
        Debug.Log("This object belongs to the local player");
    }
    ```

---

### IsRegistered

Gets whether this NetworkIdentity is registered in the network system.

- **Signature**: `public bool IsRegistered => Owner != null && Id != 0`

Description

The `IsRegistered` property indicates whether this network identity has been properly registered in the network system. An object is considered registered when it has both a valid owner and a non-zero ID assigned to it.

Returns

- **`bool`**: `true` if the object is registered (has an owner and valid ID); otherwise, `false`.

???+ example
    ```csharp
    // Example of checking if a NetworkIdentity is registered
    NetworkIdentity identity = GetComponent<NetworkIdentity>();
    if (identity.IsRegistered)
    {
        Debug.Log("This object is registered in the network");
    }
    ```

---

### IsServerOwner

Gets or sets whether this identity is owned by the server.

- **Signature**: `public bool IsServerOwner { get; internal set; }`

Description

The `IsServerOwner` property indicates whether this network identity is owned by the server itself rather than a client. This is useful for objects that should be controlled exclusively by the server.

Returns

- **`bool`**: `true` if this identity is owned by the server; otherwise, `false`.

???+ example
    ```csharp
    // Example of checking if a NetworkIdentity is owned by the server
    NetworkIdentity identity = GetComponent<NetworkIdentity>();
    if (identity.IsServerOwner)
    {
        Debug.Log("This object is owned by the server");
    }
    ```
