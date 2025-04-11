**Omni** requires Unity version `2021.3` or higher, as it leverages the latest [.NET Standard 2.1+](https://learn.microsoft.com/pt-br/dotnet/standard/net-standard?tabs=net-standard-2-1#net-standard-versions) APIs to compatibility and performance.

Compatibility Table:

![Unity 2021.3 LTS](https://img.shields.io/badge/2021.3_LTS-✅_Supported-success) 
![Unity 2022.3 LTS](https://img.shields.io/badge/2022.3_LTS-✅_Supported-success)
![Unity 2023.2](https://img.shields.io/badge/2023.2_Beta-✅_Supported-success)
![Unity 6000.0 LTS](https://img.shields.io/badge/6000.0_LTS-✅_Supported-success)

!!! warning
    Using versions not listed in the compatibility table may result in unexpected behavior or functionality issues. For the most reliable experience, please check the [Releases](https://github.com/RuanCardoso/Omni-Networking-for-Unity/releases) page for up-to-date version compatibility information.

---

## Installation

### Requirements

!!! info "System Requirements"
    - Unity `2021.3` or higher
    - .NET Standard `2.1+` API Compatibility
    - Git installed on your system(<kbd>Optional</kbd>)

### Quick Install

=== "Package Manager (Recommended)"
    1. Open Unity Package Manager (<kbd>Window</kbd> > <kbd>Package Manager</kbd>)
    2. Click the `+` dropdown in the top-left corner
    3. Select `Add package from git URL`
    4. Paste:
       ```
       https://github.com/RuanCardoso/Omni-Networking-for-Unity.git
       ```
    5. Click `Add`

=== "Manual Installation"
    1. First, install required dependencies via Package Manager:
        1. Open Unity Package Manager (<kbd>Window</kbd> > <kbd>Package Manager</kbd>)
        2. Click the `+` dropdown in the top-left corner
        3. Select `Add package by name`
        4. Add these packages one at a time:
            ```
            com.unity.localization@1.0.0
            com.unity.nuget.newtonsoft-json@3.2.1
            ```

    2. Then install Omni:
        1. Download the latest source from our [GitHub repository](https://github.com/RuanCardoso/Omni-Networking-for-Unity)
        2. Extract the ZIP file
        3. Copy the contents to your Unity project's `Assets` folder

    !!! warning "Dependencies"
        Make sure to install the required dependencies first, otherwise you may encounter compilation errors.

!!! tip "Installation Verification"
    After installation, verify that:
    
    - No console errors are present
    - The Omni menu appears in Unity's top menu bar
    - All dependencies are properly resolved

---

## Dependencies

??? info "📦 Included Dependencies"

    **Core Libraries**

    - ✨ [Newtonsoft Json](https://www.newtonsoft.com/json) - Industry-standard JSON framework for .NET, providing robust serialization and deserialization capabilities
    - 🚀 [MemoryPack](https://github.com/Cysharp/MemoryPack) - High-performance zero-allocation binary serializer optimized for gaming and real-time applications
    - ⚡ [UniTask](https://github.com/Cysharp/UniTask) - Zero allocation async/await solution for Unity, delivering superior performance over standard coroutines
    - 🎯 [DOTween](http://dotween.demigiant.com/) - Fast and efficient animation engine for Unity with a fluent API and extensive feature set
    
    **Database Connectors**

    - 📁 [SQLite](https://www.sqlite.org/) - Self-contained, serverless, zero-configuration database engine perfect for local data storage
    - 🔋 [MySqlConnector](https://mysqlconnector.net/) - High-performance, asynchronous MySQL database connector with connection pooling
    - 🐘 [Npgsql](https://www.npgsql.org/) - Open-source PostgreSQL database connector with full async support and advanced features
    - 📊 [SQLKata](https://sqlkata.com/) - Elegant SQL query builder with support for multiple databases and complex queries
    - and others depending on the database you choose.
    
    **Networking**

    - 🌐 [LiteNetLib](https://github.com/RevenantX/LiteNetLib) - Lightweight and fast UDP networking library with reliability, ordering, and connection management
    - 🔄 [kcp2k](https://github.com/vis2k/kcp2k) - Reliable UDP communication protocol implementation offering low latency and congestion control
    - 🔒 [BCrypt.Net](https://github.com/BcryptNet/bcrypt.net) - Modern cryptographic hashing for passwords with salt generation and verification
    - 🚪 [Open.NAT](https://github.com/lontivero/Open.NAT) - Port forwarding library supporting both UPnP and NAT-PMP for seamless multiplayer connectivity
    
    **Development Tools**

    - 🛠️ [Humanizer](https://github.com/Humanizr/Humanizer) - Developer utility for manipulating and formatting strings, enums, dates, times, and more
    - 🎨 [TriiInspector](https://github.com/codewriter-packages/Tri-Inspector) -  Advanced Unity inspector extension providing enhanced editor customization
    - 🔄 [ParrelSync](https://github.com/VeriorPies/ParrelSync) - Unity editor extension for testing multiplayer gameplay with multiple game instances locally
    - ⚡ [Dapper](https://github.com/DapperLib/Dapper) - High-performance micro-ORM supporting SQL queries with strong typing and object mapping
    - and others...


All dependencies are included in the Omni package and are automatically imported when you install it in your
project using the Package Manager.