# Overview

## What's New in v3.1.7

| Category | Updates |
|----------|---------|
| 🚀 Performance | • Improved network serialization speed by 35%<br>• Reduced memory allocations in hot paths<br>• Optimized packet compression algorithms |
| 🛠️ New Features | • Added Roslyn analyzer for real-time code validation<br>• Introduced Source Generator for dependency injection<br>• Added support for WebSocket secure connections<br> |
| 🐛 Bug Fixes | • Various stability improvements and bug fixes<br>• Enhanced error handling and recovery<br>• Fixed critical networking issues<br>• Improved connection reliability |
| 🔒 Security | • Enhanced encryption protocols<br>• Added built-in DDoS protection<br>• Improved authentication system<br>• Added SSL/TLS support for WebSocket connections |
| 💡 Developer Experience | • Improved error messages and logging<br>• Added extensive code documentation<br>• Enhanced Unity Editor integration |
| ⚡ Stability | • Better handling of network disconnections<br>• Enhanced connection stability<br>• Better handling of edge cases in network synchronization |

!!! tip
    If you plan to translate this documentation, it is recommended to use **Google Chrome** to avoid translation bugs and ensure accurate rendering of the content.

**Omni Networking** is a powerful multiplayer networking solution for Unity games, offering high performance and extensive features. Key capabilities include:

- **Efficient Data Management**: Advanced synchronization and state management
- **Flexible Communication**: Multiple transport protocols and messaging systems
- **Robust Security**: Built-in encryption and authentication
- **Optimized Performance**: Binary serialization and data compression
- **Developer-Friendly**: Easy-to-use API and comprehensive tooling

Whether you're creating competitive multiplayer games, co-op experiences, or massive multiplayer worlds, Omni provides the essential networking infrastructure you need.

---

**Data Storage**

| Feature | Description |
|---------|-------------|
| **Database Management** | Supports multiple relational databases through a built-in ORM (Object-Relational Mapping) system, enabling scalable, efficient, and secure data operations tailored for multiplayer environments. Compatible databases include:<br><br>• Microsoft SQL Server<br>• MySQL<br>• MariaDB<br>• PostgreSQL<br>• Oracle<br>• SQLite<br>• Firebird<br><br>Features advanced querying capabilities, connection pooling, transaction management, and automatic migration support. The ORM provides a type-safe way to work with database entities while maintaining optimal performance for multiplayer scenarios. |

**Network Communication**

| Feature | Description |
|---------|-------------|
| **Network Variables** | Provides automated property synchronization between server and clients, eliminating the need for manual message handling. Ensures real-time data consistency across all networked instances with minimal code overhead. |
| **RPC (Remote Procedure Calls)** | Facilitates direct method invocation between clients and server, supporting both reliable and unreliable transmission modes. Enables synchronized execution of functions across the network with automatic parameter serialization. |
| **gRPC (Global RPC)** | Implements network-wide event broadcasting system for executing procedures across all connected clients simultaneously. Ideal for game-wide announcements, state updates, and synchronized events. |
| **Route Management** | Express.js-inspired routing system for organizing and handling network communications. Supports route parameters, middleware, and structured request/response patterns, enabling clean and maintainable network code architecture. |
| **Custom Messages** | Enables creation of custom message types for specialized network communication, supporting user-defined data structures and transmission formats. Offers flexibility for implementing unique network features and interactions. |

**Data Serialization & Compression**

| Feature | Description |
|---------|-------------|
| **Binary Serialization** | Utilizes the `MemoryPack` library for efficient binary data serialization, minimizing data size for faster transmission and optimized network performance. |
| **JSON Serialization** | Uses the `Json.NET (Newtonsoft)` library for flexible and reliable JSON data serialization, ensuring compatibility with third-party services. |
| **Compression** | Leverages `Brotli` and `LZ4` algorithms to reduce data size, optimizing bandwidth usage and improving transmission speed. |

**Infrastructure & Security**

| Feature | Description |
|---------|-------------|
| **Port Forwarding** | Facilitates NAT traversal and opens network ports using protocols like `PMP` and `UPnP`, ensuring seamless connectivity for multiplayer sessions. |
| **Cryptography** | Utilizes `AES` and `RSA` encryption algorithms to secure data transmission, ensuring protection of sensitive information across the network. |
| **Service Locator** | Implements a centralized registry for managing dependencies and services across networked objects, enabling efficient access to shared resources and components. Supports global and local service locators for flexible dependency management. |
| **Web Sockets** | Implements a lightweight and efficient WebSocket server for real-time communication between clients and server, supporting bidirectional data exchange and event-driven messaging. |
| **Http Server** | Implements a lightweight HTTP server for handling web requests, enabling web-based services and integrations within the multiplayer environment. Supports RESTful API design(Express.js-inspired) and custom route handling for versatile network communication. |

---

## Communication Structure

The Omni framework is structured around four foundational classes designed for general communication. These classes implement methods and properties that simplify and expedite multiplayer functionalities, making the process both efficient and straightforward. Additionally, a "low-level" class is available for advanced communication, which should be utilized in contexts where restrictions or limitations apply, offering finer control for specialized cases.

The Omni framework currently utilizes four base classes, each designed for different networking roles within the multiplayer structure. These classes include:

### Base Classes Overview
| Class | Identity Required | Usage |
|-------|------------------|--------|
| `NetworkBehaviour` | Yes | For objects needing network identity (players, items) |
| `ServerBehaviour` | No | Server-only logic (game state, matchmaking) |
| `ClientBehaviour` | No | Client-only logic (UI, input handling, requests to server) |
| `DualBehaviour` | No | Combined client/server logic |

Each class serves a unique purpose in managing networked objects and handling server or client logic.

---

#### Detailed Description

=== "NetworkBehaviour"
    - **Purpose**: Manage networked objects with unique identities
    - **Common Uses**: 
        - Player characters
        - Interactable items
        - Spawnable objects
    - **Key Features**:
        - Automatic synchronization
        - Network identity management
        - Object ownership
    
    !!! warning "Object Registration"
        **Requirements**

        - Objects must be instantiated to work correctly with `NetworkBehaviour`

=== "ServerBehaviour"
    - **Purpose**: Handle server-side logic
    - **Common Uses**: 
        - Game state management
        - Player authentication
        - Routing and messaging
        - Callbacks and events
    - **Key Features**:
        - Server-only execution
        - No network identity required
        - Performance optimized

=== "ClientBehaviour"
    - **Purpose**: Handle client-side logic
    - **Common Uses**: 
        - Requests to server-side logic and response handling
        - UI management
        - Callbacks and events
    - **Key Features**:
        - Client-only execution
        - No network identity required
        - Local processing

=== "DualBehaviour"
    - **Purpose**: Combined client/server logic
    - **Common Uses**: 
        - Shared game systems
        - Unified managers
    - **Key Features**:
        - Both client/server code
        - Conditional execution
        - Code organization

This structured approach with these base classes simplifies multiplayer development, enabling clear separation of client and server responsibilities while providing flexibility for objects with and without identities.

!!! warning "Behaviour Class Limitations"
    **Scene vs Spawned Objects**

    - `ClientBehaviour`, `ServerBehaviour`, and `DualBehaviour`:
        - ✅ Can be used on scene objects
        - ❌ Cannot be spawned at runtime
        - ❌ No NetworkIdentity support

=== "Scene Objects (✅ Correct)"
    ```csharp
    // Scene objects should use ServerBehaviour/ClientBehaviour
    public class GameManager : ServerBehaviour 
    {
        protected override void OnStart()
        {
            // Server-side game management logic
            Debug.Log("GameManager initialized on server");
        }
    }

    public class LoginManager : ClientBehaviour 
    {
        protected override void OnStart()
        {
            // Client-side UI and authentication logic
            Debug.Log("LoginManager initialized on client");
        }
    }
    ```

=== "Spawnable Objects (❌ Wrong)"
    ```csharp
    // DON'T do this - will cause runtime errors
    public class Player : ClientBehaviour 
    {
        protected override void OnStart()
        {
            // This won't work when spawned!
            Debug.LogError("Players cannot use ClientBehaviour or ServerBehaviour to spawnable objects");
        }
    }
    ```

=== "Spawnable Objects (✅ Correct)"
    ```csharp
    // DO this instead - proper way to handle spawnable objects
    public class Player : NetworkBehaviour 
    {
        protected override void OnStart()
        {
            // Works correctly with spawning system
            if (IsServer)
            {
                Debug.Log("Player spawned on server");
            }

            if (IsClient)
            {
                Debug.Log("Player spawned on client");
            }
        }
    }
    ```

!!! tip "Best Practices"
    - Use `NetworkBehaviour` for any object that needs to be spawned at runtime
    - Use `ServerBehaviour`/`ClientBehaviour` for manager classes and UI elements
    - Keep scene objects and spawnable objects clearly separated in your architecture
    - Always test both scene placement and runtime spawning scenarios

!!! warning "Network Features Requirements"
    **Inheritance Required**

    All scripts using network features must inherit from any:

    - `NetworkBehaviour`
    - `ServerBehaviour`
    - `ClientBehaviour`
    - `DualBehaviour`

---

## Build & Deployment

### Build Guide

When building your project for deployment, ensure the following steps are completed to ensure a successful multiplayer experience:

!!! tip "Runtime Environment Configuration Guide"
    Choose the optimal runtime setup for your client and server components:

    IL2CPP Client + Mono Server(**Most Common Setup**):
    
    ✅ **Benefits**:

    - Client: Enhanced performance and security through IL2CPP
    - Server: Full .NET feature compatibility
    - Faster development iteration cycles
    
    IL2CPP Server + Mono Client(**Specialized Setup**):
    
    ⚠️ **Considerations**:

    - Higher server performance but limited reflection and database capabilities
    - Potential security risks with Mono client
    - Not recommended for most applications
    
    Unified Runtime(**Single Runtime Solution**):

    **IL2CPP Everywhere:**

    - Maximum performance
    - Enhanced security
    - More complex debugging process
    - Limited reflection and database capabilities
    
    **Mono Everywhere:**
    
    - Full .NET feature compatibility
    - Better debugging experience
    - Security risks on client side

    | Configuration | Performance | Security | Development Ease |
    |--------------|-------------|-----------|------------------|
    | IL2CPP Client + Mono Server | ★★★★☆ | ★★★★☆ | ★★★★★ |
    | IL2CPP Server + Mono Client | ★★★☆☆ | ★★☆☆☆ | ★★★☆☆ |
    | All IL2CPP | ★★★★★ | ★★★★★ | ★★★☆☆ |
    | All Mono | ★★★☆☆ | ★★☆☆☆ | ★★★★★ |

!!! warning "Code Stripping for Client Security"
    Always use conditional compilation to remove sensitive code from client builds:

    ```csharp
    #if OMNI_SERVER
        // Server-only sensitive code here
        private void ProcessSecureData() {
            // This code will be stripped from client builds
        }
    #endif
    ```

    ⚠️ **Critical for Mono Clients**: 

    - Mono builds have higher exposure to decompilation
    - Code stripping is *essential* when using Mono client runtime
    - Use `#if OMNI_SERVER` for:
        - Database credentials
        - Security algorithms
        - Server-side validation logic
        - Administrative functions

Before building for deployment:

1. Switch to Release mode in Unity:

    - Go to the Unity Navigation Bar
    - Select `Omni Networking` and click `Change to Release`

2. Build your project:

    - Go to `File -> Build Settings`
    - Select your target platform
    - Click `Build` to create the executable

!!! danger "Build Mode Reset Warning"
    The build configuration will reset to DEBUG when:
    
    - Switching to new target platforms
    - Updating Omni version
    - Switching Unity version
    
    This happens because each platform/version has unique preprocessor directives (#if).

    ✔️ **Always verify Release mode after:**

    - Platform changes
    - Package updates
    - Unity version changes
    
    To verify/fix:

    1. Open `Omni Networking -> Change to Release`
    2. Verify build settings are correct

### Deployment Guide

When deploying your multiplayer game, consider the following best practices to ensure a smooth and secure experience for your players:

**Recommended Providers:**

| Provider | Pros | Starting Cost |
|----------|------|---------------|
| [DigitalOcean](https://digitalocean.com) | Simple pricing, great for beginners | $5/month |
| [Azure](https://azure.microsoft.com) | Strong .NET integration | $15-20/month |
| [Google Cloud](https://cloud.google.com) | Good automation tools | $10-15/month |

**Minimum VPS Requirements:**

```bash
CPU: 1 cores
RAM: 1GB
Storage: 50GB SSD
Network: 10GB Bandwidth
OS: Ubuntu 20.04 LTS
```
#### Setup Ubuntu Server

After deploying your VPS, follow these steps to set up your Ubuntu server for hosting your multiplayer game:

1. **Connect to Server via SSH**:

    Open a terminal and use the following command to connect to your server:

    ```bash
    ssh -i your-key.pem user@server-ip

    # Example
    ssh -i "C:\Users\user\Downloads\my-key.pem" admin@192.168.0.1
    ```

    Maybe on your first connection, you may need to configure the root password.

2. **Update your Server**:

    Run the following commands to update your server:

    ```bash
    sudo apt update
    sudo apt upgrade
    sudo apt install unzip -y
    ```

3. **Database Setup (Optional)**:

    If your game requires a database, install MySQL/MariaDB or other database systems:

    ```bash
    # Install MySQL
    sudo apt install mysql-server -y

    # Secure MySQL installation
    sudo mysql_secure_installation

    # Start MySQL service
    sudo systemctl start mysql
    sudo systemctl enable mysql
    ```

    If you prefer MariaDB:

    ```bash
    # Install MariaDB
    sudo apt install mariadb-server -y
    
    # Secure MariaDB installation
    sudo mysql_secure_installation

    # Start MariaDB service
    sudo systemctl start mariadb
    sydo systemctl enable mariadb
    ```

    !!! warning "Database Security"
        - Use strong passwords (16+ characters)
        - Only allow local connections by default
        - Regularly backup your database:

        **Backup Your Database**:

        ```bash
        # Access MySQL/MariaDB
        sudo mysql -u root -p

        # Backup your database
        mysqldump -u root -p my_database > backup.sql

        # Restore your database
        mysql -u root -p my_database < backup.sql
        ```




