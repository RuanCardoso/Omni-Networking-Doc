# Network Identity

The `NetworkIdentity` component is at the heart of the Omni networking high-level API. It controls a game object's unique identity on the network, and it uses that identity to make the networking system aware of the game object.

The `NetworkIdentity` component is essential for network-aware GameObjects in Omni. It:

- Assigns unique identifiers to objects
- Enables network synchronization
- Manages object ownership
- Handles object spawning/despawning

!!! warning "NetworkIdentity Nesting Rules"
    **Hierarchy Rules**

    - Only parent objects can have `NetworkIdentity`
    - Child objects cannot have `NetworkIdentity`
    - Children access parent's `NetworkIdentity` via `Identity` property
    
    **Example Structure**

    ```
    Player (NetworkIdentity ✅)
    ├── Weapon (NetworkIdentity ❌)
    └── Inventory (NetworkIdentity ❌)
    ```

    **Code Example**

    ```csharp
    // Parent object
    public class Player : NetworkBehaviour {
        // Has NetworkIdentity automatically
    }

    // Child object
    public class Weapon : NetworkBehaviour {
        void Start() {
            // Access parent's NetworkIdentity
            var parentIdentity = Identity;
        }
    }
    ```

    **Common Error**

    If you see: "Multiple NetworkIdentity components in hierarchy", check for duplicate components in child objects.

??? info "🔍 NetworkIdentity Properties"

    Properties:
    
    | Property           | Type             | Description                                                                                                   |
    |--------------------|------------------|---------------------------------------------------------------------------------------------------------------|
    | `IdentityId`       | `int`            | Unique identifier used for network synchronization and object tracking |
    | `IsServer`         | `bool`           | Indicates if this instance is running on the server side |
    | `IsClient`         | `bool`           | Indicates if this instance is running on a client machine |
    | `IsLocalPlayer`    | `bool`           | Indicates if this object represents the local player on the local machine |
    | `IsServerOwner`    | `bool`           | Indicates if the server has authority over this objec |
    | `LocalPlayer`      | `NetworkIdentity`| Reference to the local player's NetworkIdentity component |
    | `Owner`            | `NetworkPeer`      | Reference to the client that has authority over this object |
    | `IsRegistered`   | `bool`             | Indicates if this object is registered with the network system |

!!! warning "LocalPlayer Assignment Rules"
    **Overview**

    The `NetworkIdentity.LocalPlayer` property:

    - Only available on client-side
    - Requires specific naming conventions
    - Auto-assigns based on prefab name or tag
    
    **Valid Naming Patterns**

    Prefab name must include "Player":

    ```csharp
    MyPlayer           // ✅ Valid
    PlayerCharacter    // ✅ Valid
    Character          // ❌ Invalid
    ```

    **Valid Tag Patterns**

    GameObject tag must include "Player":

    ```csharp
    "Player"          // ✅ Valid
    "BluePlayer"      // ✅ Valid
    "Character"       // ❌ Invalid
    ```

    **Troubleshooting**

    If `LocalPlayer` is null, check:

    - Prefab name contains "Player"
    - GameObject tag contains "Player"
    - Script is running on client-side

## Network Behaviour

The `NetworkBehaviour` component is a specialized MonoBehaviour that provides network-aware functionality to GameObjects. It extends the base MonoBehaviour class with additional networking features, such as:

- Remote procedure calls (RPCs)
- Network variable management
- Network event handling

### Events and Callbacks

`NetworkBehaviour` provides a range of events and callbacks to handle network-related actions. These include:

- `OnAwake`

    Called when the object is created on the network, like `Awake()` for networked objects.

    !!! example "Example"
        ```csharp
        public partial class MyBehaviour : NetworkBehaviour
        {
            [NetworkVariable]
            private int m_Ammo;

            protected override void OnAwake()
            {
                // Use field instead of property to avoid unnecessary updates
                // Server will sync initial state after spawn
                m_Ammo = 30;
            }
        }
        ```

    !!! warning "Override Considerations"
        **Base Method Calls**

        - Always call the base method when overriding unity events to ensure proper
        - When overriding `Awake()`, make sure to call `base.Awake()`

        **Best Practice**

        - Prefer using `OnAwake()` over `Awake()` for network-aware initialization
        - `OnAwake()` guarantees proper network setup order and not require manual base method calls

        ```csharp
        // Not recommended
        protected override void Awake() {
            base.Awake(); // Required!
        }

        // Recommended
        protected override void OnAwake() {
        
        }
        ```

- `OnStart`

    Called when the object is initialized on the network, like `Start()` for networked objects.

    !!! example "Example"
        ```csharp
        public partial class MyBehaviour : NetworkBehaviour
        {
            [NetworkVariable]
            private foat m_Health;

            [LocalService]
            private PlayerManager m_PlayerManager;
            
            protected override void OnStart()
            {
                // Use field instead of property to avoid unnecessary updates
                // Server will sync initial state after spawn
                m_Health = 100f;

                // Get local service reference
                Debug.Log("Player manager reference: " + m_PlayerManager.Nickname);
            }
        }
        ```
    
    !!! warning "Override Considerations"
        **Base Method Calls**

        - Always call the base method when overriding unity events to ensure proper execution
        - When overriding `Start()`, make sure to call `base.Start()`
        
        **Best Practice**

        - Prefer using `OnStart()` over `Start()` for network-aware initialization
        - `OnStart()` guarantees proper network setup order and not require manual base method calls

        ```csharp
        // Not recommended
        protected override void Start() {
            base.Start(); // Required!
        }

        // Recommended
        protected override void OnStart() {

        }
        ```

- `OnStartLocalPlayer`

    Called when the object is initialized as the local player on the network.

    !!! example "Example"
        ```csharp
        public class MyBehaviour : NetworkBehaviour
        {
            [SerializeField]
            private Camera m_PlayerCamera;

            protected override void OnStartLocalPlayer()
            {
                m_PlayerCamera.enabled = true;
                Debug.Log("Local player initialized");
            }
        }
        ```

- `OnStartRemotePlayer`

    Called when the object is initialized as a remote player on the network.

    !!! example "Example"
        ```csharp
        public class MyBehaviour : NetworkBehaviour
        {
            [SerializeField]
            private Camera m_PlayerCamera;

            protected override void OnStartRemotePlayer()
            {
                m_PlayerCamera.enabled = false;
                Debug.Log("Remote player initialized");
            }
        }
        ```

!!! tip "Best Practices"
    - Use `OnAwake` for network variable initialization
    - Use `OnStart` for component references and general setup
    - Use `OnStartLocalPlayer` for local player-specific setup
    - Use `OnStartRemotePlayer` for remote player visualization
    
    Always check `IsServer` or `IsClient` when needed to ensure code runs in the correct context.

- `OnServerClientSpawned`

    Called on the server when a client-side object is fully spawned and registered. This method ensures that the client object is ready and fully initialized on the network before the server performs any post-spawn operations.

    The `peer` parameter represents the client that instantiated this NetworkIdentity, providing access to client-specific information like connection ID and status.

    !!! example "Example"
        ```csharp
        public partial class PlayerManager : NetworkBehaviour
        {
            private const int SPAWNED_MESSAGE_ID = 10;

            protected override void OnServerClientSpawned(NetworkPeer peer)
            {
                Debug.Log($"Client {peer.Id} has been fully spawned and registered. Ready for server-side operations");
                using var message = Rent();
                message.WriteString("Wow! You have spawned successfully!");
                Server.RpcToPeer(SPAWNED_MESSAGE_ID, peer, message, Target.SelfOnly, DeliveryMode.ReliableOrdered);           
            }

            [Client(SPAWNED_MESSAGE_ID)]
            void OnSpawnedMessage(DataBuffer message)
            {
                string text = message.ReadString();
                Debug.Log($"Server message: {text}");
            }
        }
        ```

    !!! tip "Usage Tip: Leveraging OnServerClientSpawned for Client Initialization"
        When you need to execute server-side logic that depends on the client being completely initialized, overriding the     `OnServerClientSpawned` method is ideal. This approach can be used to:
    
        - **Synchronize Data**: Send initial game state or identity-specific data to the client that instantiates it's identity.
        - **Log Client Activity**: Confirm the client has been successfully registered and log the event.
        - **Initialize Resources**: Set up server-side components or trigger game mechanics that rely on the client’s active     participation that instantiates your identity.

- `OnTick`

    Called on each network update tick, useful for synchronized updates.

    !!! example "Example"
        ```csharp
        public partial class PlayerMovement : NetworkBehaviour
        {
            [NetworkVariable]
            private Vector3 m_Position;
            
            public override void OnTick(ITickInfo data)
            {
                if (IsServer)
                {
                    // Manually sync network variable if needed!!
                    SyncPosition(DefaultNetworkVariableOptions);
                }
            }
        }
        ```

    !!! warning
        It is necessary to enable the module in the Network Manager object in your scene for this code to work properly.

- `OnNetworkDestroy`

    Called when the object is unregistered from the network and destroyed.

    !!! example "Example"
        ```csharp
        public class ResourceManager : NetworkBehaviour
        {
            protected override void OnNetworkDestroy()
            {
                Debug.Log($"Cleaning up network resources for {IdentityId}");
            }
        }
        ```

- `OnRequestedAction`

    The `OnRequestedAction` method is a server-side callback designed to handle remote actions initiated by client-side entities. When a client requests an operation—such as activating an item, modifying configuration settings, or triggering an event—this method is invoked on the server to perform the corresponding action, identically to a remote procedure call (RPC), but embedded within the `NetworkBehaviour` class.

    !!! example "Example"
        ```csharp
        public class ItemInteraction : NetworkBehaviour
        {
            protected override void OnRequestedAction(DataBuffer data, NetworkPeer peer)
            {
                if (IsServer)
                {
                    var itemId = data.ReadInt32();
                    var action = data.ReadString();
                    
                    Debug.Log($"Player {peer.Id} requested action {action} on item {itemId}");
                }
            }
        }
        ```

    !!! tip "Usage Tip: Handling Client-Triggered Actions"
        After a client has spawned and is fully initialized, it can call a method like `RequestAction()` to send a remote request to the server. Common use cases include:
        
        - **Item Interactions**: Activate or deactivate in-game items.
        - **Configuration Updates**: Change settings or preferences stored on the server.
        - **Event Triggers**: Initiate specific game events or mechanics.
        - **State Synchronization**: Update or synchronize the client’s state with the server.

- `Scene-Related Events`

    A set of events for handling scene loading and unloading:

    !!! example "Example"
        ```csharp
        public class LevelManager : NetworkBehaviour
        {
            protected override void OnBeforeSceneLoad(Scene scene, SceneOperationMode op)
            {
                // Prepare for scene load
                Debug.Log($"Preparing to load scene: {scene.name}");
            }

            protected override void OnSceneLoaded(Scene scene, LoadSceneMode mode)
            {
                // Initialize scene-specific content
                if (IsServer)
                {
                    SpawnSceneEntities(scene);
                }
            }

            protected override void OnSceneUnloaded(Scene scene)
            {
                // Cleanup scene resources
                Debug.Log($"Cleaning up scene: {scene.name}");
            }
        }
        ```

!!! tip "Event Execution Order"
    The typical execution order of network events is:
    
    1. `OnAwake`
    2. `OnStart`
    3. `OnStartLocalPlayer` or `OnStartRemotePlayer`
    4. `OnServerClientSpawned` (server-side)
    5. `OnTick` (repeated)
    6. Scene events (as needed)
    7. `OnNetworkDestroy`


