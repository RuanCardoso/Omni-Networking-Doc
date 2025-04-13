# Network Behaviour Methods

## NetworkVariableSync

Synchronizes a network variable's value across the network.

### Client-Side Signatures:
```csharp
void NetworkVariableSync<T>(T property, byte propertyId, NetworkVariableOptions options)
void NetworkVariableSync<T>(T property, byte propertyId, DeliveryMode deliveryMode = DeliveryMode.ReliableOrdered, byte sequenceChannel = 0)
void NetworkVariableSync<T>(NetworkVariableOptions options, [CallerMemberName] string ___ = "")
void NetworkVariableSync<T>(DeliveryMode deliveryMode = DeliveryMode.ReliableOrdered, byte sequenceChannel = 0, [CallerMemberName] string ___ = "")
```

### Server-Side Signatures:
```csharp
void NetworkVariableSync<T>(T property, byte propertyId, NetworkVariableOptions options)
void NetworkVariableSync<T>(T property, byte propertyId, Target target = Target.Auto, DeliveryMode deliveryMode = DeliveryMode.ReliableOrdered, int groupId = 0, DataCache dataCache = default, byte sequenceChannel = 0)
void NetworkVariableSync<T>(NetworkVariableOptions options, [CallerMemberName] string ___ = "")
void NetworkVariableSync<T>(Target target = Target.Auto, DeliveryMode deliveryMode = DeliveryMode.ReliableOrdered, int groupId = 0, DataCache dataCache = default, byte sequenceChannel = 0, [CallerMemberName] string ___ = "")
```

### Description
Sends a network variable synchronization message to update the value across the network. Can be called manually to force synchronization of a network variable.

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `property` | `T` | The value to synchronize |
| `propertyId` | `byte` | The unique identifier of the network variable |
| `options` | `NetworkVariableOptions` | Options for synchronization including delivery mode, target, etc. |
| `target` | `Target` | The target recipients for the synchronization |
| `deliveryMode` | `DeliveryMode` | How the synchronization should be delivered |
| `groupId` | `int` | Optional group ID for targeted synchronization |
| `dataCache` | `DataCache` | Optional caching settings |
| `sequenceChannel` | `byte` | Channel for ordering messages |

### Example

```csharp
public partial class Player : NetworkBehaviour
{
    [NetworkVariable] 
    private float m_Health = 100f;

    void Update()
    {
        if (IsServer)
        {
            // Manual sync with default options
            SyncHealth(DefaultNetworkVariableOptions);
            
            // Manual sync with custom options
            SyncHealth(new NetworkVariableOptions
            {
                DeliveryMode = DeliveryMode.ReliableOrdered,
                Target = Target.AllPlayers
            });
        }
    }
}
```

---

## NetworkVariableSyncToPeer

Synchronizes a network variable's value to a specific peer.

### Server-Side Signature:
```csharp
void NetworkVariableSyncToPeer<T>(T property, byte propertyId, NetworkPeer peer)
```

### Description
Sends a network variable synchronization message to a specific peer. This method allows for targeted synchronization of network variables, which is useful when you need to update the value for just one client instead of broadcasting to all clients.

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `property` | `T` | The value to synchronize |
| `propertyId` | `byte` | The unique identifier of the network variable |
| `peer` | `NetworkPeer` | The specific peer to receive the synchronization |

### Example

```csharp
public partial class PlayerStats : NetworkBehaviour
{
    [NetworkVariable] 
    private int m_Score;
    
    [NetworkVariable]
    private float m_Health;

    public void SyncStatsTo(NetworkPeer player)
    {
        if (IsServer)
        {
            // Sync individual network variables to specific player
            NetworkVariableSyncToPeer(m_Score, ScorePropertyId, player);
            NetworkVariableSyncToPeer(m_Health, HealthPropertyId, player);
        }
    }

    public void OnPlayerJoinedLate(NetworkPeer latecomer)
    {
        if (IsServer)
        {
            // Sync current state to player who joined late
            SyncStatsTo(latecomer);
        }
    }
}
```

### Advanced Example

```csharp
public partial class GameManager : NetworkBehaviour
{
    [NetworkVariable]
    private Dictionary<string, int> m_PlayerScores = new();
    
    [NetworkVariable]
    private GameState m_CurrentState;

    public void SyncGameStateToNewPlayer(NetworkPeer newPlayer)
    {
        if (IsServer)
        {
            // Sync complex data structures to specific player
            NetworkVariableSyncToPeer(m_PlayerScores, PlayerScoresPropertyId, newPlayer);
            NetworkVariableSyncToPeer(m_CurrentState, GameStatePropertyId, newPlayer);
            
            Debug.Log($"Synced game state to player {newPlayer.Id}");
        }
    }

    protected override void OnServerClientSpawned(NetworkPeer peer)
    {
        // When a new client connects, sync the current game state
        SyncGameStateToNewPlayer(peer);
    }
}
```

### Best Practices
- Use this method when you need to synchronize network variables to a specific client
- Useful for sending initial state to late-joining players
- Only callable from the server side
- Automatically uses ReliableOrdered delivery mode to ensure consistency
- The target peer must have authority to receive the network variable updates

### Important Notes
1. This method can only be called from the server side
2. The synchronization is sent with Target.SelfOnly to ensure only the specified peer receives it
3. Uses ReliableOrdered delivery mode by default to ensure consistent state
4. The property must be marked with [NetworkVariable] attribute
5. The propertyId must match the network variable's assigned ID

### Common Use Cases
- Synchronizing initial state to late-joining players
- Sending player-specific network variable updates
- Optimizing network traffic by targeting specific peers instead of broadcasting
- Implementing catch-up mechanics for clients who temporarily lost connection

---

## Rpc

Sends a Remote Procedure Call (RPC) to execute code on remote peers.

### Client-Side Signatures:
```csharp
void Rpc(byte msgId, ClientOptions options)
void Rpc(byte msgId, DataBuffer buffer = null, DeliveryMode deliveryMode = DeliveryMode.ReliableOrdered, byte sequenceChannel = 0)
void Rpc(byte msgId, IMessage message, ClientOptions options = default)
void Rpc<T1>(byte msgId, T1 p1, ClientOptions options = default) where T1 : unmanaged
// Additional generic overloads up to 5 parameters
```

### Server-Side Signatures:
```csharp
void Rpc(byte msgId, ServerOptions options)
void Rpc(byte msgId, DataBuffer buffer = null, Target target = Target.Auto, DeliveryMode deliveryMode = DeliveryMode.ReliableOrdered, int groupId = 0, DataCache dataCache = default, byte sequenceChannel = 0)
void Rpc(byte msgId, IMessage message, ServerOptions options = default)
void Rpc<T1>(byte msgId, T1 p1, ServerOptions options = default) where T1 : unmanaged
// Additional generic overloads up to 5 parameters
```

### Description
Sends an RPC message to execute code on remote peers. Client RPCs are sent to the server, while server RPCs are sent to clients.

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `msgId` | `byte` | Unique identifier for the RPC |
| `options` | `ClientOptions`/`ServerOptions` | Configuration options for the RPC |
| `buffer` | `DataBuffer` | Optional data to send with the RPC |
| `target` | `Target` | (Server only) Target recipients |
| `deliveryMode` | `DeliveryMode` | How the RPC should be delivered |
| `groupId` | `int` | Optional group ID for targeted RPCs |
| `dataCache` | `DataCache` | Optional caching settings |
| `sequenceChannel` | `byte` | Channel for ordering messages |

### Example

```csharp
public class Player : NetworkBehaviour
{
    private const byte MOVE_RPC_ID = 1;
    private const byte DAMAGE_RPC_ID = 2;

    // Client-side RPC
    void Update()
    {
        if (IsLocalPlayer && Input.GetKeyDown(KeyCode.Space))
        {
            // Send RPC to server
            Client.Rpc(MOVE_RPC_ID, transform.position, new ClientOptions
            {
                DeliveryMode = DeliveryMode.Unreliable
            });
        }
    }

    // Server-side RPC handler
    [Server(MOVE_RPC_ID)]
    void OnMoveRequest(Vector3 position)
    {
        // Process movement
        transform.position = position;
        
        // Broadcast to other clients
        Server.Rpc(DAMAGE_RPC_ID, 10f, new ServerOptions
        {
            Target = Target.AllExceptSelf,
            DeliveryMode = DeliveryMode.ReliableOrdered
        });
    }

    // Client-side RPC handler
    [Client(DAMAGE_RPC_ID)]
    void OnDamageReceived(float damage)
    {
        Debug.Log($"Received damage: {damage}");
    }
}
```

## RpcToPeer

Sends a Remote Procedure Call (RPC) to a specific peer.

### Server-Side Signatures:
```csharp
void RpcToPeer(byte msgId, NetworkPeer peer, ServerOptions options)
void RpcToPeer(byte msgId, NetworkPeer peer, DataBuffer buffer = null, Target target = Target.SelfOnly, DeliveryMode deliveryMode = DeliveryMode.ReliableOrdered, DataCache dataCache = default, byte sequenceChannel = 0)
```

### Description
Sends an RPC message to a specific peer, bypassing the standard ownership-based RPC routing. This allows sending RPCs directly to any connected peer regardless of object ownership.

### Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `msgId` | `byte` | Unique identifier for the RPC |
| `peer` | `NetworkPeer` | The specific peer to receive the RPC |
| `options` | `ServerOptions` | Configuration options for the RPC |
| `buffer` | `DataBuffer` | Optional data to send with the RPC |
| `target` | `Target` | Target type (usually SelfOnly for direct peer communication) |
| `deliveryMode` | `DeliveryMode` | How the RPC should be delivered |
| `dataCache` | `DataCache` | Optional caching settings |
| `sequenceChannel` | `byte` | Channel for ordering messages |

### Example

```csharp
public class ChatManager : NetworkBehaviour
{
    private const byte PRIVATE_MESSAGE_RPC = 1;

    public void SendPrivateMessage(NetworkPeer recipient, string message)
    {
        if (IsServer)
        {
            using var buffer = Rent();
            buffer.WriteString(message);
            
            Server.RpcToPeer(PRIVATE_MESSAGE_RPC, recipient, buffer, Target.SelfOnly);
        }
    }

    [Client(PRIVATE_MESSAGE_RPC)]
    void OnPrivateMessageReceived(DataBuffer message)
    {
        string text = message.ReadString();
        Debug.Log($"Private message received: {text}");
    }
}
```

## OnAwake

Called after the object is instantiated and registered, but before it becomes active.

### Signature
```csharp
protected internal virtual void OnAwake()
```

### Description
This method is called during the initialization phase of a networked object, after it has been instantiated and registered with the network system, but before it becomes active. It's the ideal place to perform initial setup that doesn't depend on other networked components or active state.

### Example

```csharp
public partial class MyBehaviour : NetworkBehaviour
{
    [NetworkVariable]
    private int m_Ammo;

    protected override void OnAwake()
    {
        // Initialize network variables
        m_Ammo = 30;
        
        // Setup other initial state
        Debug.Log($"Network object {IdentityId} awakening");
    }
}
```

### Best Practices
- Use for network variable initialization
- Avoid dependencies on other active components
- Prefer `OnAwake()` over Unity's `Awake()` for network-aware initialization
- No need to call base method when overriding

## OnStart

Called after the object is instantiated and has become active.

### Signature
```csharp
protected internal virtual void OnStart()
```

### Description
This method is called when the networked object has been fully initialized and activated. It's the appropriate place to set up references to other components and perform initialization that requires the object to be in an active state.

### Example

```csharp
public partial class MyBehaviour : NetworkBehaviour
{
    [NetworkVariable]
    private float m_Health;

    [LocalService]
    private PlayerManager m_PlayerManager;
    
    protected override void OnStart()
    {
        // Initialize with starting values
        m_Health = 100f;

        // Setup component references
        Debug.Log($"Player manager reference: {m_PlayerManager.Nickname}");
        
        // Additional initialization requiring active state
        if (IsServer)
        {
            InitializeServerComponents();
        }
    }
}
```

## OnStartLocalPlayer

Called after the local player object is instantiated.

### Signature
```csharp
protected internal virtual void OnStartLocalPlayer()
```

### Description
This method is called specifically on the object that represents the local player. It's the ideal place to set up player-specific features that should only be active for the local player instance.

### Example

```csharp
public class PlayerController : NetworkBehaviour
{
    [SerializeField]
    private Camera m_PlayerCamera;
    
    [SerializeField]
    private AudioListener m_AudioListener;

    protected override void OnStartLocalPlayer()
    {
        // Enable local player components
        m_PlayerCamera.enabled = true;
        m_AudioListener.enabled = true;
        
        // Setup input handlers
        SetupInputSystem();
        
        Debug.Log("Local player initialized");
    }

    private void SetupInputSystem()
    {
        // Setup player input system
    }
}
```

## OnStartRemotePlayer

Called after a remote player object is instantiated.

### Signature
```csharp
protected internal virtual void OnStartRemotePlayer()
```

### Description
This method is called on objects that represent other players in the network. It's used to set up the representation of remote players in the local game instance.

### Example

```csharp
public class PlayerController : NetworkBehaviour
{
    [SerializeField]
    private Camera m_PlayerCamera;
    
    [SerializeField]
    private AudioListener m_AudioListener;
    
    [SerializeField]
    private PlayerNameTag m_NameTag;

    protected override void OnStartRemotePlayer()
    {
        // Disable local-only components
        m_PlayerCamera.enabled = false;
        m_AudioListener.enabled = false;
        
        // Setup remote player visualization
        m_NameTag.Show();
        
        Debug.Log("Remote player initialized");
    }
}
```

## OnServerClientSpawned

Called on the server when a client-side object has been fully spawned and registered.

### Signature
```csharp
protected virtual void OnServerClientSpawned(NetworkPeer peer)
```

### Description
This method is called on the server when a client's object is fully spawned and registered in the network. It ensures that the client object is ready and fully initialized before the server performs any post-spawn operations.

### Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `peer` | `NetworkPeer` | The client peer that spawned the object |

### Example

```csharp
public partial class PlayerManager : NetworkBehaviour
{
    private const int SPAWNED_MESSAGE_ID = 10;

    protected override void OnServerClientSpawned(NetworkPeer peer)
    {
        Debug.Log($"Client {peer.Id} has been fully spawned and registered");
        
        // Send initial game state
        using var message = Rent();
        message.WriteString("Welcome to the game!");
        Server.RpcToPeer(SPAWNED_MESSAGE_ID, peer, message, Target.SelfOnly);
        
        // Initialize player state
        SyncInitialState();
    }

    [Client(SPAWNED_MESSAGE_ID)]
    void OnSpawnedMessage(DataBuffer message)
    {
        string text = message.ReadString();
        Debug.Log($"Server message: {text}");
    }
}
```

## OnTick

Called on each network update tick.

### Signature
```csharp
public virtual void OnTick(ITickInfo data)
```

### Description
This method is called at regular intervals defined by the network tick rate. It's used for synchronized updates that need to occur at a fixed rate across the network.

### Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `data` | `ITickInfo` | Information about the current tick |

### Example

```csharp
public partial class PlayerMovement : NetworkBehaviour
{
    [NetworkVariable]
    private Vector3 m_Position;
    
    [NetworkVariable]
    private Vector3 m_Velocity;
    
    public override void OnTick(ITickInfo data)
    {
        if (IsServer)
        {
            // Update position based on velocity
            m_Position += m_Velocity * data.DeltaTime;
            
            // Sync the updated position
            SyncPosition(DefaultNetworkVariableOptions);
        }
    }
}
```

### Important Note
The tick system module must be enabled in the NetworkManager for this method to be called.

## Register

Registers the network behaviour within the network system.

### Signature
```csharp
protected internal void Register()
```

### Description
This method registers the network behaviour with the network system, configuring server or client-specific event handlers and services based on the current network identity state. It also integrates the behaviour with the tick system if enabled.

### Key Operations
- Registers network variables
- Sets up RPC handlers for client or server
- Initializes service locator
- Registers with tick system (if enabled)
- Sets up event handlers for network identity

### Example

```csharp
public class CustomNetworkBehaviour : NetworkBehaviour
{
    // Registration happens automatically
    // You rarely need to call Register() manually
    
    protected override void OnAwake()
    {
        // Registration is already complete at this point
        Debug.Log($"Behaviour registered with ID: {Id}");
    }
}
```

## Unregister

Unregisters the network behaviour from the network system.

### Signature
```csharp
protected internal void Unregister()
```

### Description
Removes the network behaviour from all associated events and systems. This ensures proper cleanup of network resources and event handlers.

### Key Operations
- Removes RPC handlers
- Unregisters from tick system
- Removes event subscriptions
- Cleans up service locator references

### Example

```csharp
public class CustomNetworkBehaviour : NetworkBehaviour
{
    public void CleanupAndRemove()
    {
        // Unregister is called automatically when destroying the object
        // But can be called manually if needed
        Unregister();
        Destroy(gameObject);
    }

    protected override void OnNetworkDestroy()
    {
        Debug.Log("Behaviour unregistered and destroyed");
    }
}
```

## OnNetworkDestroy

Called when the object is unregistered from the network.

### Signature
```csharp
protected virtual void OnNetworkDestroy()
```

### Description
This method is called when the network behaviour is being unregistered from the network system. It's the ideal place to perform cleanup of network-related resources.

### Example

```csharp
public class ResourceManager : NetworkBehaviour
{
    private List<NetworkIdentity> m_ManagedResources = new();

    protected override void OnNetworkDestroy()
    {
        // Clean up managed resources
        foreach (var resource in m_ManagedResources)
        {
            if (resource != null)
            {
                resource.Destroy();
            }
        }
        
        m_ManagedResources.Clear();
        Debug.Log($"Cleaned up network resources for {IdentityId}");
    }
}
```

## OnRequestedAction

Called when a remote action is requested on the server-side entity.

### Signature
```csharp
protected virtual void OnRequestedAction(DataBuffer data, NetworkPeer peer)
```

### Description
This server-side callback handles remote actions initiated by client-side entities. It's used to process and validate client requests for actions.

### Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `data` | `DataBuffer` | The data buffer containing action parameters |
| `peer` | `NetworkPeer` | The client peer requesting the action |

### Example

```csharp
public class ItemInteraction : NetworkBehaviour
{
    protected override void OnRequestedAction(DataBuffer data, NetworkPeer peer)
    {
        if (IsServer)
        {
            // Read action parameters
            var itemId = data.ReadInt32();
            var action = data.ReadString();
            
            // Validate request
            if (CanPerformAction(peer, itemId, action))
            {
                // Process the action
                ProcessItemAction(itemId, action);
                Debug.Log($"Player {peer.Id} performed action {action} on item {itemId}");
            }
            else
            {
                Debug.LogWarning($"Invalid action request from player {peer.Id}");
            }
        }
    }

    private bool CanPerformAction(NetworkPeer peer, int itemId, string action)
    {
        // Implement validation logic
        return true;
    }

    private void ProcessItemAction(int itemId, string action)
    {
        // Implement action processing
    }
}
```

## OnSceneLoaded

Called after a scene has been loaded.

### Signature
```csharp
protected virtual void OnSceneLoaded(Scene scene, LoadSceneMode mode)
```

### Description
This method is called when a scene has finished loading. It provides an opportunity to perform initialization or setup specific to the newly loaded scene.

### Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `scene` | `Scene` | The scene that was loaded |
| `mode` | `LoadSceneMode` | The mode used to load the scene (Single or Additive) |

### Example

```csharp
public class LevelManager : NetworkBehaviour
{
    protected override void OnSceneLoaded(Scene scene, LoadSceneMode mode)
    {
        if (IsServer)
        {
            // Initialize scene-specific content
            SpawnSceneEntities(scene);
            SetupSceneState(scene);
        }

        // Common initialization
        ConfigureSceneLighting(scene);
        Debug.Log($"Scene '{scene.name}' loaded with mode: {mode}");
    }

    private void SpawnSceneEntities(Scene scene)
    {
        // Spawn network objects for this scene
    }

    private void SetupSceneState(Scene scene)
    {
        // Initialize scene-specific game state
    }

    private void ConfigureSceneLighting(Scene scene)
    {
        // Setup scene lighting
    }
}
```

## OnSceneUnloaded

Called after a scene has been unloaded.

### Signature
```csharp
protected virtual void OnSceneUnloaded(Scene scene)
```

### Description
This method is called when a scene has been unloaded. Use it to clean up resources or perform necessary cleanup operations related to the unloaded scene.

### Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `scene` | `Scene` | The scene that was unloaded |

### Example

```csharp
public class LevelManager : NetworkBehaviour
{
    private Dictionary<Scene, List<NetworkIdentity>> m_SceneEntities = new();

    protected override void OnSceneUnloaded(Scene scene)
    {
        // Clean up scene-specific resources
        if (m_SceneEntities.TryGetValue(scene, out var entities))
        {
            foreach (var entity in entities)
            {
                if (entity != null)
                {
                    entity.Destroy();
                }
            }
            m_SceneEntities.Remove(scene);
        }

        // Additional cleanup
        CleanupSceneResources(scene);
        Debug.Log($"Cleaned up scene: {scene.name}");
    }

    private void CleanupSceneResources(Scene scene)
    {
        // Perform additional resource cleanup
    }
}
```

## OnBeforeSceneLoad

Called before a scene begins to load.

### Signature
```csharp
protected virtual void OnBeforeSceneLoad(Scene scene, SceneOperationMode op)
```

### Description
This method is called just before a scene begins loading or unloading. It provides an opportunity to prepare for the scene transition.

### Parameters
| Parameter | Type | Description |
|-----------|------|-------------|
| `scene` | `Scene` | The scene that is about to be loaded/unloaded |
| `op` | `SceneOperationMode` | The type of operation (Load or Unload) |

### Example

```csharp
public class LevelManager : NetworkBehaviour
{
    protected override void OnBeforeSceneLoad(Scene scene, SceneOperationMode op)
    {
        if (op == SceneOperationMode.Load)
        {
            // Prepare for scene load
            Debug.Log($"Preparing to load scene: {scene.name}");
            PreloadSceneAssets(scene);
        }
        else
        {
            // Prepare for scene unload
            Debug.Log($"Preparing to unload scene: {scene.name}");
            PrepareSceneUnload(scene);
        }
    }

    private void PreloadSceneAssets(Scene scene)
    {
        // Preload necessary assets
    }

    private void PrepareSceneUnload(Scene scene)
    {
        // Prepare for scene removal
    }
}
```

## Rent

Rents a DataBuffer from the network manager's buffer pool.

### Signature
```csharp
protected DataBuffer Rent()
```

### Description
This method provides a DataBuffer instance from the network manager's object pool. The buffer must be properly disposed of after use.

### Returns
- `DataBuffer`: A rented buffer instance from the pool

### Example

```csharp
public class MessageHandler : NetworkBehaviour
{
    private const byte MESSAGE_RPC_ID = 1;

    public void SendMessage(string message)
    {
        if (IsServer)
        {
            using (var buffer = Rent()) // Automatically returns to pool when disposed
            {
                buffer.WriteString(message);
                Server.Rpc(MESSAGE_RPC_ID, buffer);
            }
        }
    }

    [Client(MESSAGE_RPC_ID)]
    void OnMessageReceived(DataBuffer buffer)
    {
        string message = buffer.ReadString();
        Debug.Log($"Received message: {message}");
    }
}
```

## Destroy

Removes and destroys this network component.

### Signature
```csharp
public void Destroy()
```

### Description
This method unregisters the network behaviour from the network system and destroys the component. It ensures proper cleanup of network resources.

### Example

```csharp
public class DestructibleObject : NetworkBehaviour
{
    [NetworkVariable]
    private float m_Health = 100f;

    public void TakeDamage(float damage)
    {
        if (IsServer)
        {
            m_Health -= damage;
            
            if (m_Health <= 0)
            {
                // Clean up and destroy
                Destroy();
            }
        }
    }

    protected override void OnNetworkDestroy()
    {
        Debug.Log($"Object {IdentityId} destroyed");
        // Perform any final cleanup
    }
}
```
```

Esta é a documentação completa dos métodos solicitados, incluindo descrições detalhadas, parâmetros, exemplos de uso e considerações importantes para cada método.