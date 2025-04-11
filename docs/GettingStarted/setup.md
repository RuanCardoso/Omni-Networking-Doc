## Setup

!!! note "🚀 Introduction"

    **Build Defines**

    Omni automatically configures the following build defines after package import:
    
    | Define | Description |
    |--------|-------------|
    | `#if OMNI_RELEASE` | Optimizes code for production. Disables logging, debugging features. Used when building the final game or server. |
    | `#if OMNI_DEBUG` | Enables development detailed error information and runtime checks. Default for Unity Editor and Build. |
    | `#if OMNI_SERVER` | Indicates a dedicated server build. Removes client-specific code, UI elements, and graphics rendering. |

    **Common Imports**

    Add these using directives to your scripts:
    ```csharp
    using Omni.Core;      // Core functionality
    using Omni.Inspector; // (Optional) - Unity inspector extensions
    using Omni.Core.Web;  // (Optional) - Web and networking features
    ```

    **Conditional Compilation**
    ```csharp
    // Debug vs Release mode
    #if OMNI_DEBUG
        Debug.Log("Development build");
    #else
        Debug.Log("Release build");
    #endif
    
    // Client vs Server code
    #if OMNI_SERVER
        // Server-only code
        Debug.Log("Running on dedicated server");
    #else
        // Client-only code
        Debug.Log("Running on client");
    #endif
    ```

    **Conditional Compilation with [Conditional Attribute]**
    ```csharp
    [Conditional("OMNI_DEBUG")]
    void DebugMethod() 
    {
        Debug.Log("Running in debug mode");
    }

    [Conditional("OMNI_SERVER")]
    void ServerMethod() 
    {
        Debug.Log("Running on dedicated server");
    }
    ```

    !!! danger "Conditional Attribute Limitation"
        The `[Conditional]` attribute only removes the **method calls**, not the method implementation. 
        The method code still exists in your build. For complete code stripping, use `#if` directives:

1. Go to the Unity Navigation Bar and select `Omni Networking`.
2. Click the `Setup` menu item.

A game object named `Network Manager` will be created in the scene. This object is responsible for the connection and contains all the network settings.

!!! warning "Network Manager Requirements"
    **Required Configuration**

    - The `Network Manager` object **must** exist in your scene
    - Do not destroy this object during runtime
    - Do not rename this object
    - Keep the default name: `Network Manager`

    **Common Issues**
    
    If the `Network Manager` is missing or renamed:

    - Network connections will fail
    - Multiplayer features won't work
    - Runtime errors will occur

!!! tip "Network Manager Structure(***Optional***)"
    **Object Hierarchy**

    ```
    Network Manager (Main Object)
    ├── Client (Child Object)
    └── Server (Child Object)
    ```

    **How It Works**

    - The `Network Manager` comes with two child objects: `Client` and `Server`
    - You can attach your networking scripts to these objects(***Optional***)
    - During build:
        - Client build: Server scripts are removed
        - Server build: Client scripts are removed

    !!! warning
        This system only **removes objects** from the scene. The code still exists in the build.
        
        For complete code removal, use conditional compilation:
        ```csharp
        #if OMNI_SERVER
            // This code will be completely removed from client builds
            void ServerOnlyMethod() {
                // Server-specific code here
            }
        #endif
        ```

3. Select the `Network Manager` object in the scene to view the network settings.

| Option        | Description                           |
| ------------- | ------------------------------------- |
| `Current Version` | Displays the installed Omni version. Important for compatibility checks and troubleshooting. |
| `Public IPv4` | Your device's public IPv4 address. Updates automatically but can be refreshed manually |
| `Public IPv6` | Your device's public IPv6 address (if available). Used for modern network configurations. |
| `GUID` | Unique 128-bit identifier used for network authentication. Can be regenerated through context menu |

!!! bug
    If the `Public IP` field displays an incorrect IP address, click the context menu of the `Network Manager` script and select **Force Get Public Ip** to update the field with the correct IP address. The correct IP address is essential for server identification and connection.

!!! warning
    If the `GUID` between the client and server does not match, the connection will be refused. Ensure the `GUID` is correctly set in the `Network Manager` object to establish a successful connection. To update the `GUID`, click the context menu of the `Network Manager` script and select **Generate GUID**.

### Modules

| Modules          | Description                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------------- |
| `Tick Module`    | Allows the use of a tick-based system for sending messages and other tick-based operations. |
| `Sntp Module`    | Provides a high-precision synchronized clock between all clients and the server.            |

### Connection Settings

| Option         | Description                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------- |
| `Host Address` | A list of IP addresses that the client can connect to, the first address is used to connect. |
| `Port`         | The server port number used for both listening and client connections. |

### Configuration Options

=== "Basic"
    | Option | Description |
    |--------|-------------|       
    | `Auto Start Client` | When enabled, client automatically connects to server on scene load (Default: true) | 
    | `Auto Start Server` | When enabled, server automatically starts hosting on scene load (Default: true) |
    | `Tick Rate` | Server update frequency in Hz. Higher values increase precision but consume more CPU (Default: 15) |
    | `Use UTF-8 Encoding` | Uses UTF8 for string encoding. Enable for non-ASCII text support (Default: false) |
    | `Lock Client Fps` | Limits client frame rate to reduce CPU/GPU load. Set to 0 for unlimited (Default: 60) |
    | `Lock Server Fps` | Limits server frame rate to reduce CPU load. Set to 0 for unlimited (Default: 240) |	

=== "Advanced"
    | Option | Description |
    |--------|-------------|
    | `Pool Capacity` | Maximum size in bytes for each network message buffer. Larger values consume more memory (Default: 32768) |
    | `Pool Size` | Number of pre-allocated network buffers. Increase for high-traffic scenarios (Default: 32) |    
    | `Use Unaligned Memory` | Enables faster memory access on supported platforms. May cause issues on mobile (Default: false) |
    | `Enable Bandwidth Optimization` | Enable bandwidth optimization for data transmission |
    | `Run In Background` | Keep game running when window loses focus. Essential for server hosting (Default: true) |

=== "HTTP Server"
    | Option | Description |
    |--------|-------------|
    | `Enable Http Server` | Activates REST API endpoint for external service integration (Default: false) |
    | `Enable Http Ssl` | Enables HTTPS for secure API communication. Requires valid SSL certificate (Default: false) |
    | `Http Server Port` | Port number for HTTP/HTTPS server. Common values: 80 (HTTP), 443 (HTTPS) (Default: 80) |

### Permissions

| Option                        | Description                                                                                      |
| ----------------------------- | ------------------------------------------------------------------------------------------------ |
| `Allow NetVar's From Client`  | Determines whether client-side changes to network variables are permitted, allowing clients to modify networked variables directly. |
| `Allow Across Group Message`  | Allows messages to be sent across different network groups, enabling communication between distinct groups in the network. |

---

### Registered Prefabs

The Registered Prefabs list provides a centralized way to manage network-spawnable objects. When objects are registered, they can be instantiated across the network using either their name or index.

!!! tip "Registration Methods"
    **1. Using the Inspector**

    - Drag and drop prefabs directly into the Registered Prefabs list
    - Objects are automatically assigned a unique network ID
    
    **2. Runtime Registration**
    ```csharp
    [SerializeField]
    private NetworkIdentity playerPrefab;
    
    void Start()
    {
        NetworkManager.AddPrefab(playerPrefab);
    }
    ```

!!! warning "Important Considerations"
    - Prefab list must be identical on both server and clients
    - Index-based spawning is more efficient than name-based
    - Changes to the list require a rebuild

---

### Transporter Settings

The **Transporter Settings** section allows you to configure various network transport parameters, including disconnection timeout, network event processing per frame, lag simulation, channel setup, IPv6 support, max connections, ping intervals, and more. Available options may vary based on the selected transporter.

Currently, three transporters are supported: **Lite Transporter**, **KCP Transporter**, and **Web Socket Transporter**. Each transporter offers distinct features and capabilities, catering to different network requirements and scenarios.

| Transporter  | Rel. Ordered | Unreliable | Rel. Unordered | Sequence | Rel. Sequence | Browser |
|--------------|--------------|------------|----------------|-----------|---------------|---------|
| Lite         | ✅           | ✅         | ✅             | ✅        | ✅            | ❌      |
| Kcp          | ✅           | ✅         | ❌             | ❌        | ❌            | ❌      |
| Web Socket   | ✅           | ❌         | ❌             | ❌        | ❌            | ✅      |

Caption:

- Reliable: Guarantees packet delivery and correct order.  
- Unreliable: Fast transmission with no guarantee of delivery or order.  
- Reliable Unordered: Guarantees packet delivery but does not enforce order.  
- Sequenced: Ensures newer packets are always processed in order, but older packets are discarded if they arrive out of sequence; does not guarantee delivery.  
- Reliable Sequenced: Guarantees that only the latest packet in a sequence is delivered reliably, discarding older packets in transit.  
- Browser: Supported in web browsers. 

#### Message Delivery Methods

Each delivery method serves a specific purpose in networked applications.

=== "Reliable Ordered"

    - Guarantees both **delivery** and **order**.
    - Higher overhead due to **acknowledgments** and **retransmissions**.
    - Best for: **Critical game data**.
    
    **Examples:**

    - ❤️ Player health updates (ensures damage is applied correctly)  
    - 🎒 Inventory changes (prevents item duplication/loss)  
    - 🏆 Quest progress updates (avoids desynchronization between client and server)  
    - 🎮 Match results (ensures accurate win/loss reporting)  
    - 💰 Economy transactions (prevents currency exploits or rollback issues)  

=== "Unreliable"

    - No **delivery** or **order** guarantees.
    - Lowest **latency**, highest **performance**.
    - Best for: **Frequent, non-critical updates**.

    **Examples:**

    - 🚀 Player position updates (real-time accuracy is more important than every packet arriving)    
    - 🎭 Character animations (minor frame drops won’t break gameplay)  
    - ✨ Particle effects (purely visual; occasional data loss is unnoticeable)  
    - 🔊 Sound effects triggers (footsteps, background noise, minor impact if dropped)
    - 🔥 Bullet tracers (cosmetic projectiles that don’t affect game logic)  

=== "Reliable Unordered"

    - Guarantees **delivery**, but not **order**.
    - Less overhead compared to **Reliable** since it does not enforce sequence.
    - Best for: **Independent events**.

    **Examples:**

    - 💬 Chat messages (messages must be received but don’t have to be perfectly sequential)  
    - 🎁 Item pickups (ensures the item is received even if network lag occurs) 
    - 🤪 Player emotes (taunts, dance moves – must be delivered, but order isn’t crucial)
    - 🔔 Notification triggers (level-up alerts, achievement unlocks, ability ready status)  

=== "Sequenced"

    - Only processes the **latest data**.
    - **Drops older packets** if newer ones arrive first.
    - Best for: **State updates** that should not be processed out of order.

    **Examples:**

    - 🏃 Player movement updates (keeps the latest position; outdated positions are ignored)  
    - 📡 NPC AI states (ensures AI actions are always based on the latest conditions)  
    - 📊 UI updates (keeps HUD elements like health bars, ammo count, or minimaps in sync)  
    - 🌦️ Weather conditions (only the current weather state matters, past updates are irrelevant)  
    - 🔵 Targeting indicators (ensures the player sees the latest enemy highlight)

=== "Reliable Sequenced"

    - Guarantees **delivery** of the **latest state only**.
    - Older packets are **discarded** if a newer one is received first.
    - Best for: **Critical state updates** where only the most recent value matters.

    **Examples:**

    - 🕹️ Game state synchronization (ensures all players receive the latest match state)  
    - 🏆 Score updates (prevents outdated scores from being displayed)  
    - ⏳ Round timers (ensures correct remaining time is always shown)  
    - 🔄 Ability cooldowns (players always see the current cooldown state without outdated info)  
    - 📦 Active objective status (ensures the current mission or capture point is correct)  

!!! tip "Choosing the Right Method"

    - 🛡️ **Reliable Ordered** → When data loss and security exploits is **unacceptable**  
    - ⚡ **Unreliable** → For **frequent, disposable** updates  
    - 📦 **Reliable Unordered** → For **independent events**  
    - 🔄 **Sequenced** → When **only the latest state matters**  
    - 🎯 **Reliable Sequenced** → When **only the latest state matters** and for **critical state synchronization**  

!!! warning "Sequenced & Reliable Sequenced: Lite Transporter Only"
    **Note:** These delivery methods are only available in the **Lite Transporter**.

    **Sequenced** and **Reliable Sequenced** packets must be sent over **separate channels** for different data types.  
    If multiple data types share the same channel, one type of data may effectively **overwrite** the other.

    **Why can packets be discarded?**  
    In **Sequenced** or **Reliable Sequenced** modes, each packet is assigned a **sequence number**.  
    When a packet with a higher (newer) sequence number arrives, any **older** packet (lower sequence number) that arrives afterward is automatically discarded.  

    **Example Issue:**  

    - Sending both *health updates* and *ammo count* over the **same sequenced channel**:
      ```plaintext
      Channel 1: Health (100 → 90) [Older packet, seq. 10]
      Channel 1: Ammo   (50 → 45)  [Newer packet, seq. 11]
      ```
      - The ammo update (`seq. 11`) is processed first.
      - When the health update (`seq. 10`) arrives, it is considered outdated compared to `seq. 11`, so it is discarded.

    **Result:**  

    - The **health update** is never applied because it arrived after a **newer** (ammo) packet.

    **Solution:**  

    - Use separate channels for different data types:
      ```plaintext
      Channel 1: Health (100 → 90)  ✅
      Channel 2: Ammo   (50 → 45)   ✅
      ```
    - This ensures each type of data maintains its **own** sequence ordering and avoids overwriting or discarding unrelated updates.

    !!! tip "Learn More About Delivery Methods"
        While Omni uses its own implementation, you can learn more about how delivery methods and sequence channels workin general networking by reading resources like the [LidGren Documentation](https://www.dotnetcodegeeks.com/2015/05/lidgren-network-explaining-netdeliverymethod-and-sequence-channels.html). The concepts are similar across networking solutions.

!!! info "Performance Considerations"
    - **Reliable methods** increase **bandwidth usage** due to acknowledgments.  
    - **Unreliable methods** provide the best **latency** and performance.  
    - **Sequenced methods** help maintain order but **sacrifice older data** to ensure real-time accuracy.  
    - **Reliable Sequenced** is useful when only the **latest state** is important, reducing unnecessary retransmissions.  

!!! danger
    The **KCP Transporter** and **Web Socket Transporter** is currently ***experimental*** and may contain unresolved issues. Use it with caution and consider thoroughly testing for stability in your specific use case.

!!! note
    By default, **Omni** utilizes the **Lite Transporter** for network operations. To switch to a different transporter, follow these steps:
 
    1. **Remove the Lite Transporter**: In your scene, locate the `Network Manager` object. Select it, and remove the `Lite Transporter` component from this object.
    2. **Add the Desired Transporter**: Once the Lite Transporter is removed, add the component of your preferred transporter to the `Network Manager` object.
    
    This configuration enables you to tailor network transport settings to suit the specific requirements of your project, ensuring optimal compatibility and performance.

!!! warning
    Some properties or functions may be unavailable for certain transporters. If an incompatible option is used, an error message will appear to inform you of the mismatch.

For detailed information on each transporter and their specific features, consult the respective documentation:

- [LiteNetLib Documentation](https://github.com/RevenantX/LiteNetLib)
- [KCP Transporter (kcp2k) Documentation](https://github.com/MirrorNetworking/kcp2k)