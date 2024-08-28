# Overview

**Omni Networking** is a comprehensive networking solution that offers features for high-performance multiplayer game development in Unity. Omni provides functionalities for database management, simulated HTTP, RPC, gRPC, port forwarding, network variables, binary and JSON serialization, among others.

Omni supports only versions 2021.2 and above, as it utilizes the latest APIs available in .NET Standard 2.1 and beyond to achieve the highest possible performance.

# Installation

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

# Setup

!!! note

    **After importing the package and waiting for the recompilation, some macros are automatically defined:**

    - `OMNI_RELEASE`
    - `OMNI_DEBUG`
    - `OMNI_SERVER` is defined when the target platform is set to `DEDICATED SERVER`.

    You can use these macros to switch between release and debug builds.

    ```c#
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

1. Go to the Unity Navigation Bar and select `Omni Networking`.
2. Click the `Setup` menu item.

A GameObject named `Network Manager` will be created in the scene. This object is responsible for the connection and contains all the network settings.

!!! warning

    The `Network Manager` object must be in the scene for Omni to function properly and not should be destroyed or renamed.

!!! note

    The `Network Manager` object, There are two child objects that can be **optionally** used to run client and server scripts. Remember, it is **optional** to place scripts on these objects. When scripts are placed on these two objects, they will be automatically separated during the build process. In **release** mode, server scripts will not be executed in the client build, and vice versa. **Note that this does not strip the code; it merely prevents it from being executed by removing the objects from the scene at build time.**

    **To strip the code, it is recommended to use the macros `OMNI_RELEASE`, `OMNI_DEBUG`, `OMNI_SERVER`.**

3. Select the `Network Manager` object in the scene to view the network settings.

| Option        | Description                           |
| ------------- | ------------------------------------- |
| `Public IPv4` | The public IPv4 address of the client |
| `Public IPv6` | The public IPv6 address of the client |

### Modules

| Modules          | Description                                                                                 |
| ---------------- | ------------------------------------------------------------------------------------------- |
| `Console Module` | Allows sending and receiving commands in the console when the build is for a server.        |
| `Match Module`   | Allows matchmaking, creating, joining, and deleting groups, among other features.           |
| `Tick Module`    | Allows the use of a tick-based system for sending messages and other tick-based operations. |
| `Sntp Module`    | Provides a high-precision synchronized clock between all clients and the server.            |

### Connection Settings

| Option        | Description                                                           |
| ------------- | --------------------------------------------------------------------- |
| `Server Port` | The port number on which the server listens for incoming connections. |
| `Client Port` | The port number on which the client listens for incoming connections. |

| Option         | Description                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------- |
| `Host Address` | The IP address or hostname of the server to which the client will connect.                  |
| `Port`         | The port number on which the server is listening and which will be used for the connection. |

### Miscellaneous

| Option                     | Description                                                                                      |
| -------------------------- | ------------------------------------------------------------------------------------------------ |
| `Tick Rate`                | The rate at which the game loop processes updates, measured in ticks per second.                 |
| `Max Fps On Client`        | The maximum frame rate that the client will attempt to render, measured in frames per second.    |
| `Auto Start Server`        | Whether the server should automatically start when the application launches.                     |
| `Auto Start Client`        | Whether the client should automatically start when the application launches.                     |
| `Use UTF-8 Encoding`       | Whether to use UTF-8 encoding for text data in the application.                                  |
| `Use Binary Serialization` | Coming soon.                                                                                     |
| `Use Unaligned Memory`     | Whether to allow unaligned memory access, which may affect performance and compatibility.        |
| `Run In Background`        | Whether the application should continue running while in the background, even when not in focus. |

### Registered Prefabs

This list is used to automatically instantiate network objects. When a network object is instantiated by name or indexer, the object will be looked up in this list and instantiated automatically. Remember, manual instantiation is also available, and using this list is not required.

### Transporter Settings

The **Transporter Settings** section configures network transport parameters such as disconnection time, network event processing per frame, lag simulation, channels, IPv6 support, and ping interval, etc. The available options depend on the selected transporter.

Currently, we have two available transporters, with plans to add more in the future: Lite and KCP Transporter.
