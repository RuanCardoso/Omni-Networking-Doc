## Overview

!!! tip
    If you plan to translate this documentation, it is recommended to use `Google Chrome` to avoid translation bugs and ensure accurate rendering of the content.

**Omni Networking** is a comprehensive solution designed to support high-performance multiplayer game development in `Unity`. With a wide array of features, `Omni` provides robust tools to handle various networking needs essential for modern games. Below is an `overview` of Omni's core functionalities:

---

**Core Functionalities**

| Feature             | Description                                                                                               |
|---------------------|-----------------------------------------------------------------------------------------------------------|
| **Database Management** | Supports relational databases with a built-in `ORM`, enabling scalable, efficient, and secure data operations tailored for multiplayer environments. |
| **RouteX**          | A flexible routing system inspired by Express.js, allowing developers to define and manage network routes efficiently. It enables organized handling of network requests and responses, making it ideal for complex multiplayer interactions.               |
| **RPC (Remote Procedure Call)** | Allows methods to be called across clients and servers, enabling synchronized actions and data sharing in multiplayer environments. Making it easy to call functions remotely.    |
| **gRPC (Global Remote Procedure Call)** | Allows methods to be called across clients and servers, enabling synchronized actions and data sharing in multiplayer environments. Making it easy to call functions remotely. |
| **Port Forwarding** |Facilitates NAT traversal and opens network ports using protocols like `PMP` and `UPnP`, ensuring seamless connectivity for multiplayer sessions.         |
| **Network Variables** | Enables automatic synchronization of properties between server and clients without the need for custom messages or RPCs, streamlining data consistency across networked players.     |
| **Binary Serialization** | Utilizes the `MemoryPack` library for efficient binary data serialization, minimizing data size for faster transmission and optimized network performance.         |
| **JSON Serialization**  | Uses the `Json.NET (Newtonsoft)` library for flexible and reliable JSON data serialization, ensuring compatibility with third-party services and smooth data interchange. |
| **Compression**     | Leverages `Brotli` and `LZ4` algorithms to reduce data size, optimizing bandwidth usage and improving transmission speed for smoother network performance.      |
| **Cryptography**    | Utilizes `AES` and `RSA` encryption algorithms to secure data transmission, ensuring protection of sensitive information across the network.        |

---

`Omni` supports only versions `2021.3 and above`, as it utilizes the latest APIs available in .NET Standard 2.1 and beyond to achieve the highest possible performance.

Compatibility Table:

| Unity Version           | Compatibilidade        |
|-------------------------|------------------------|
| Unity 2021.3 *(LTS)*    | ✅                     |
| Unity 2022.3 *(LTS)*    | ✅                     |
| Unity 6 *(LTS)*         | ✅                     |

!!! warning
    Versions not listed in the compatibility table may not be fully supported and could exhibit unexpected behavior. For the most reliable experience, please refer to the [Releases](https://github.com/RuanCardoso/Omni-Networking-for-Unity/releases) page to verify compatibility with the latest available versions.

---

## Installation

!!! warning

    **Before getting started, it's important to be aware that Omni includes some dependencies by default:**

    - Newtonsoft Json
    - Memory Pack
    - UniTask
    - DOTween
    - Database modules, among others.

    Make sure to remove these dependencies from your project if they already exist. For this reason, I recommend always starting with a new project to avoid any conflicts.

1. Download the latest Omni package from the [Releases](https://github.com/RuanCardoso/Omni-Networking-for-Unity/releases).
2. Import the package into your project.

---

## Setup

!!! note

    **After importing the package and waiting for the recompilation, some macros are automatically defined:**

    - `OMNI_RELEASE`
    - `OMNI_DEBUG`
    - `OMNI_SERVER` <- is defined when the target platform is set to `DEDICATED SERVER` in the `Build Settings`.

    You can use these macros to switch between release and debug builds, by default the `OMNI_DEBUG` macro is defined.

    ```csharp
    #if OMNI_RELEASE
    print("You're in release mode");
    #endif

    #if OMNI_DEBUG
    print("You're in debug mode");
    #endif

    #if OMNI_SERVER
    print("You're in server mode");
    #endif
    ```

    You can use these macros to strip the code from the build process for security reasons.

    ```csharp
    #if OMNI_SERVER
    // sensitive server code
    #else
    // Client code
    #endif
    ```

    Omni using's:

    ```csharp
    using Omni.Core;
    using static Omni.Core.RouteX;
    ```

1. Go to the Unity Navigation Bar and select `Omni Networking`.
2. Click the `Setup` menu item.

A GameObject named `Network Manager` will be created in the scene. This object is responsible for the connection and contains all the network settings.

!!! warning
    The `Network Manager` object must be in the scene for Omni to function properly and not should be destroyed or renamed.

!!! tip
    The `Network Manager` object has two child objects that can **optionally** host client and server scripts. It’s **optional** to place scripts on these objects. When scripts are assigned to these child objects, Unity automatically separates them during the build process. In `release` mode, server scripts won’t run in the client build, and vice versa.

    Note: This approach doesn’t strip the code; it merely prevents execution by removing the objects from the scene at build time. To strip the code, consider using the `OMNI_RELEASE`, `OMNI_DEBUG`, and `OMNI_SERVER` macros.

3. Select the `Network Manager` object in the scene to view the network settings.

| Option        | Description                           |
| ------------- | ------------------------------------- |
| `Public IPv4` | The public IPv4 address of the client |
| `Public IPv6` | The public IPv6 address of the client |

---

### Modules

| Modules          | Description                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------------- |
| `Console Module` | Allows sending and receiving commands in the console when the build is for a server.        |
| `Match Module`   | Allows matchmaking, creating, joining, and deleting groups, among other features.           |
| `Tick Module`    | Allows the use of a tick-based system for sending messages and other tick-based operations. |
| `Sntp Module`    | Provides a high-precision synchronized clock between all clients and the server.            |

---

### Connection Settings

| Option        | Description                                                           |
| ------------- | --------------------------------------------------------------------- |
| `Server Port` | The port number on which the server listens for incoming connections. |
| `Client Port` | The port number on which the client listens for incoming connections. |

| Option         | Description                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------- |
| `Host Address` | The IP address or hostname of the server to which the client will connect.                  |
| `Port`         | The port number on which the server is listening and which will be used for the connection. |

---

### Miscellaneous

| Option                     | Description                                                                                      |
| -------------------------- | ------------------------------------------------------------------------------------------------ |
| `Tick Rate`                | The rate at which the game loop processes updates, measured in ticks per second.                 |
| `Pool Capacity`            | Represents, in bytes, the size of the buffer `(byte[])` internally allocated for each `DataBuffer`.  |
| `Pool Size`                | Specifies the number of `DataBuffer` instances pre-allocated in the pool, ready for use.           |
| `Max Fps On Client`        | The maximum frame rate that the client will attempt to render, measured in frames per second.    |
| `Auto Start Server`        | Whether the server should automatically start when the application launches.                     |
| `Auto Start Client`        | Whether the client should automatically start when the application launches.                     |
| `Use UTF-8 Encoding`       | Whether to use UTF-8 encoding for text data in the application.                                  |
| `Use Binary Serialization` | Coming soon..... DON'T USE IT YET! Experimental feature.                                         |
| `Use Unaligned Memory`     | Whether to allow unaligned memory access, which may affect performance and compatibility.        |
| `Run In Background`        | Whether the application should continue running while in the background, even when not in focus. |

---

### Permissions

| Option                        | Description                                                                                      |
| ----------------------------- | ------------------------------------------------------------------------------------------------ |
| `Allow NetVar's From Client`  | Determines whether client-side changes to network variables are permitted, allowing clients to modify networked variables directly. |
| `Allow Across Group Message`  | Allows messages to be sent across different network groups, enabling communication between distinct groups in the network. |

---

### Registered Prefabs

This list is used to automatically instantiate network objects. When a network object is instantiated by name or indexer, the object will be looked up in this list and instantiated automatically. Remember, manual instantiation is also available, and using this list is not required.

---

### Transporter Settings

The **Transporter Settings** section allows you to configure various network transport parameters, including disconnection timeout, network event processing per frame, lag simulation, channel setup, IPv6 support, max connections, ping intervals, and more. Available options may vary based on the selected transporter.

Currently, two transporters are supported: **Lite Transporter** and **KCP Transporter**, with additional options planned for future releases.

| Transporter            | ReliableOrdered | Unreliable | ReliableUnordered | Sequenced | ReliableSequenced | Browser Compatibility |
|------------------------|-----------------|------------|-------------------|-----------|-------------------|------------------------|
| Lite Transporter       | ✅              | ✅         | ✅                | ✅        | ✅                | ❌                     |
| Kcp Transporter        | ✅              | ✅         | ❌                | ❌        | ❌                | ❌                     |
| Web Socket Transporter | ✅              | ❌         | ❌                | ❌        | ❌                | ⏳ Available Soon      |

!!! danger
    The **KCP Transporter** is currently ***experimental*** and may contain unresolved issues. Use it with caution and consider thoroughly testing for stability in your specific use case.

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

---

## Known Issues

!!! bug
    **Public IP Configuration Required**: Omni relies on the correct IP configuration in the `Public IP` field of the `Network Manager`. When sharing your project, this field may display an incorrect IP address from another user. To update it, open the context menu of the `Network Manager` script, press **Get External IP**, and Omni will automatically update the field to reflect your current IP address.

!!! bug
    **Omni Package Installation Issues**: If you experience problems during the Omni package installation, such as Unity freezing or crashing, and the necessary macros are not defined, you may need to manually reset the macros. To do this, go to the root folder of the Omni package and delete the hidden file that begins with `omni_macros`. Ensure hidden files are visible in your file explorer to locate this file.

---

## Communication Structure

The Omni framework is structured around four foundational classes designed for general communication. These classes implement methods and properties that simplify and expedite multiplayer functionalities, making the process both efficient and straightforward. Additionally, a "low-level" class is available for advanced communication, which should be utilized in contexts where restrictions or limitations apply, offering finer control for specialized cases.

The Omni framework currently utilizes four base classes, each designed for different networking roles within the multiplayer structure. These classes include:

- `NetworkBehaviour`
- `ServerBehaviour`
- `ClientBehaviour`
- `DualBehaviour`

Each class serves a unique purpose in managing networked objects and handling server or client logic.

---

### `NetworkBehaviour`

- Designed for **networked objects with a `NetworkIdentity`**.
- Primarily used for objects that require a unique identity within the network, such as player avatars or specific interactable objects in the game world.
- This class provides essential methods and properties to facilitate identity-based communication and synchronization across the network.

!!! warning
    Network objects that have an `NetworkIdentity` must be instantiated to be registered with the network. Direct creation without instantiation will not register the object, and the `NetworkBehaviour` don't work.

---

### `ServerBehaviour`

- Suited for **networked objects that do not require a `NetworkIdentity`**.
- Manages logic exclusively on the **server side**.
- Ideal for general server tasks, such as managing game state or processing server-side events.
- Simplifies server-side programming by abstracting client-specific requirements, allowing a focus on server logic.

---

### `ClientBehaviour`

- Suited for **networked objects that do not require a `NetworkIdentity`**.
- Used for managing **client-specific logic** and interactions.
- Ideal for general client tasks, such as login, character selection, or UI management, etc.
- Simplifies client-side programming by abstracting server-specific requirements, allowing a focus on client logic.

---

### `DualBehaviour`

- **Combines `ClientBehaviour` and `ServerBehaviour`** to handle both client and server logic within a single script.
- Best suited for cases where a unified approach to client and server operations is necessary, reducing redundancy.
- Enables shared logic across client and server while maintaining the separation of client-exclusive and server-exclusive tasks.

---

**Summary Table**

| Class              | Identity Requirement       | Logic Scope          | Use Case                                                   |
|--------------------|----------------------------|----------------------|-------------------------------------------------------------|
| `NetworkBehaviour` | Requires a `NetworkIdentity`   | General network      | For networked objects with unique identifiers(`NetworkIdentity`)              |
| `ServerBehaviour`  | No `NetworkIdentity`         | Server-side only     | For server-exclusive logic without identity dependencies.   |
| `ClientBehaviour`  | No `NetworkIdentity`      | Client-side only     | For client-specific logic, optimized for local processing.  |
| `DualBehaviour`    | No `NetworkIdentity`       | Client and Server    | Combines client and server logic within a single script.    |

This structured approach with these base classes simplifies multiplayer development, enabling clear separation of client and server responsibilities while providing flexibility for objects with and without identities.

!!! warning
    Objects with `ClientBehaviour`, `ServerBehaviour`, or `DualBehaviour` scripts attached cannot be spawned and not support `NetworkIdentity`, as they are intended for use with scene objects only. These classes should be used exclusively for objects that exist in the scene from the start, rather than dynamically spawned instances. For this case use the `NetworkBehaviour` class with a `NetworkIdentity` attached.

!!! warning
    All network operations, including RPC's, events, and network variables, require scripts to inherit from one of these four classes (`NetworkBehaviour`, `ServerBehaviour`, `ClientBehaviour`, or `DualBehaviour`). Without this inheritance, network functionality will not be available in the script.

---

### Network Identity

The `NetworkIdentity` component is at the heart of the Omni networking high-level API. It controls a game object's unique identity on the network, and it uses that identity to make the networking system aware of the game object.

!!! warning
    It is important to note that Omni does not support `NetworkIdentity` on nested GameObjects. Otherwise, Omni will emit an error. To avoid this, ensure your parent `GameObject` is the only `GameObject` in the stack with a `NetworkIdentity` component. Child GameObjects can access the parents' `NetworkIdentity` component via `Identity` property inhirited from `NetworkBehaviour`.

Properties:

| Property           | Type             | Description                                                                                                   |
|--------------------|------------------|---------------------------------------------------------------------------------------------------------------|
| `IdentityId`       | `int`            | A unique identifier for the networked object. This ID is assigned by the server to track the object across all clients. |
| `IsServer`         | `bool`           | Indicates if the object is running on the server side. `false` if the object is running on the client side. |
| `IsClient`         | `bool`           | Indicates if the object is running on the client side. `true` if the object is running on the server side. |
| `IsLocalPlayer`    | `bool`           | Specifies if this object represents the local player. `true` if the object is owned by the local client. |
| `IsServerOwner`    | `bool`           | Indicates if the server has ownership of this object. `true` if the server is the primary controller of this object. |
| `LocalPlayer`      | `NetworkIdentity`| A static reference to the local player instance, set on the client only. Assigned when the local player is spawned. |
| `Owner`            | `NetworkPeer`      | The owner of this object. Available only on the server, and returns `NetworkManager.LocalPeer` on the client. |
| `IsRegistered`   | `bool`             | Indicates whether this `NetworkIdentity` is registered on the network. |

!!! warning
    `NetworkIdentity.LocalPlayer` is only available on the client side. Assigned when the prefab have the tag `Player`, contain `Player` in the tag, or include `Player` in the prefab name, otherwise it will be `null`.

    *Example of valid prefab name*: 

    - `RuanCharacterPlayer`
    - `JuniorPlayer`

    *Example of valid prefab tag*: 

    - `Player`
    - `RuanTagPlayer`
    - `JuniorTagPlayer`

---

# Service Locator Pattern

The **Service Locator** pattern is a design pattern that provides a centralized registry or "locator" for retrieving instances of services or dependencies at runtime. This pattern allows for flexible dependency management and reduces the coupling between objects, making it ideal for complex applications like multiplayer games.

Key Benefits of the Service Locator Pattern

- **Centralized Access to Services**: By acting as a central registry, the Service Locator allows different parts of the application to access services without tightly coupling dependencies.
- **Flexible and Scalable**: Services can be registered, replaced, or removed dynamically, providing flexibility for handling different networked components and systems in a multiplayer environment.
- **Reduced Dependency on Singleton**: Unlike the Singleton pattern, which can introduce issues in multiplayer setups (such as unwanted global state persistence), the Service Locator keeps services manageable and avoids potential conflicts.
- **Improved Testing and Maintainability**: The pattern facilitates testing and maintainability by allowing services to be swapped or mocked, which is crucial in a large multiplayer codebase.

## Service Locator vs. Singleton

While the **Singleton pattern** is commonly used to ensure only one instance of a class exists, it can lead to issues in multiplayer environments. Singletons often hold global state, which can interfere with networked instances and create unpredictable behavior, especially when managing player-specific data.

!!! tip
    Omni recommends using the **Service Locator** pattern instead of Singletons for multiplayer development. The Service Locator provides better control over dependencies and avoids the pitfalls of global state inherent in Singletons, making it a more stable choice for complex networked systems.

---

## Usage Guide

By default, any script that inherits from a network class(`NetworkBehaviour`, `ServerBehaviour`, `ClientBehaviour`, `DualBehaviour` or `ServiceBehaviour`) is automatically registered in the **Service Locator**. This registration simplifies access to network services across your game's architecture. 

Service Naming and Customization

- **Automatic Naming**: Each service name is assigned automatically upon registration, based on the script's name.
- **Customizable Names**: You can modify the default service name directly in the Unity Inspector, allowing flexibility in organizing and identifying services as needed.

Types of Service Locators in Omni

Omni provides two types of Service Locators to manage service instances effectively within different scopes:

1. **Global Service Locator**: A shared Service Locator accessible across all networked instances. This global registry is ideal for managing universal services that need to be accessed by multiple objects or systems throughout the game.

2. **Local Service Locator**: Each `NetworkIdentity` has its own local Service Locator, unique to that networked identity. allows you to retrieve specific services within the same identity, providing fine control over dependencies and enabling isolated management of services per networked object.

---

With this dual Service Locator approach, Omni offers a flexible, scalable structure that enhances dependency management in multiplayer environments, ensuring services are easily accessible while maintaining clear separation between global and local contexts.

=== "Global Service Locator"
     ```csharp
     // Example usage of the Global Service Locator
     public class UIManager : ServiceBehaviour
     { 

     }

     public class LoginManager : ClientBehaviour 
     { 

     }
    
     public class Player : NetworkBehaviour
     {
        void Example()
        {
           // Accessing the `LoginManager` from the Global Service Locator
           LoginManager loginManager = NetworkService.Get<LoginManager>();
    
           // Accessing the `UIManager` from the Global Service Locator with a custom name
           UIManager uiManager = NetworkService.Get<UIManager>("MyUIManager");
        }
     }
     ```

=== "Local Service Locator"
     ```csharp
     // Example usage of the Local Service Locator
     // This script must be attached to the same networked object(`NetworkIdentity`).
     public class WeaponManager : NetworkBehaviour
     { 

     }
    
     public class PlayerManager : NetworkBehaviour
     {
        void Example()
        {
           // Accessing the `WeaponManager` from the Local Service Locator using my own identity
           WeaponManager weaponManager = Identity.Get<WeaponManager>();
        }
     }
     ```

!!! note
    The service locator can find services regardless of their depth within the hierarchy, ensuring accessibility even in deeply nested objects.

!!! warning
    If you have multiple instances of the same script type across different objects in the hierarchy (e.g., multiple `WeaponManager` scripts, one for each weapon), you should rename the service in the Inspector for each instance. Use a named lookup to retrieve the specific instance you need, such as: 

    - `Identity.Get<WeaponManager>("GlockManager");`
    - `Identity.Get<WeaponManager>("MeeleManager");`

    An exception will be thrown if the service is not found or if multiple instances of the same type have the same name.

!!! tip
    The Service Locator is designed to be fast and efficient, with minimal performance cost, unlike `GetComponent`, making it suitable for frequent use.

*See the API Reference for more information about the Service Locator and its usage.*