# Service Locator Pattern

## Overview

The **Service Locator** pattern is a fundamental design pattern in Omni that provides a centralized registry for managing and accessing service instances at runtime. This pattern is especially crucial for multiplayer game development, offering a robust solution for dependency management while maintaining code flexibility and maintainability.

**Key Benefits**

**Centralized Service Management**

  - Single point of access for all services
  - Simplified service discovery and retrieval
  - Reduced coupling between components

**Runtime Flexibility**

  - Dynamic service registration and replacement
  - Support for named service instances
  - Efficient handling of networked components

**Enhanced Testability**

  - Easy service mocking for unit tests
  - Improved component isolation
  - Better maintainability

**Performance Optimized**

  - Faster than traditional `GetComponent` calls
  - Minimal runtime overhead
  - Efficient caching mechanism

## Service Locator vs. Singleton

The **Singleton** and **Service Locator** patterns are both used to manage service instances. However, while Singletons enforce a single global instance, the **Service Locator** centralizes service management dynamically without direct coupling.

While the **Singleton pattern** is commonly used to ensure only one instance of a class exists, it can lead to issues in multiplayer environments. Singletons often hold global state, which can interfere with networked instances and create unpredictable behavior, especially when managing player-specific data.


### **Problems with Singleton**

**High Coupling**  

   - Components directly depend on the Singleton, making replacements difficult.
   
**Global State Issues**  

   - Shared state can cause inconsistencies in multiplayer games.

**Difficult Testing**  

   - Singletons require workarounds for dependency injection.

**Lifecycle Problems** 

   - Initialization order may cause unexpected behavior.

**Resource Waste**  

   - Persisting instances may lead to memory overhead.


### **Advantages of Service Locator**

✅ **Lower Coupling** – Services can be replaced dynamically.  
✅ **Better State Management** – Supports multiple isolated instances.  
✅ **Improved Testability** – Enables dependency injection.  
✅ **Controlled Lifecycle** – Services can be created/destroyed as needed.  
✅ **Performance Optimized** – Efficient caching, avoiding costly lookups.  

### **Comparison Table**

| Feature                  | Singleton                                  | Service Locator                        |
|--------------------------|-------------------------------------------|-----------------------------------------|
| **Instance Management**  | Single fixed instance                     | Dynamic, managed instances             |
| **Coupling**             | High (direct dependencies)                | Low (flexible service resolution)      |
| **Global State**         | Yes (can cause conflicts in multiplayer)  | No (isolated instances supported)      |
| **Testability**          | Hard (requires modifications)             | Easy (supports mocks and DI)           |
| **Lifecycle Control**    | Persistent throughout execution           | Created/destroyed dynamically          |
| **Flexibility**          | Low (hard to swap instances)              | High (services can be replaced easily) |
| **Multiplayer Support**  | Risky (global state may leak)             | Recommended (per-player isolation)     |

!!! tip
    For **multiplayer games and modular architectures**, **Service Locator** is the preferred choice. It provides **better flexibility, testability, and scalability**, resolving Singleton limitations while ensuring efficient dependency management.

---

## Usage Guide

The following base classes are automatically registered with the Service Locator system: `NetworkBehaviour`, `ServerBehaviour`, `ClientBehaviour`, `DualBehaviour`, and `ServiceBehaviour`.

!!! tip "Service Naming and Customization"

    **Default Naming**
    
       - Automatically uses the script's class name
       - No additional configuration required
    
    **Custom Naming**
    
       - Configurable through Unity Inspector
       - Supports multiple instances of the same service type
       - Useful for distinguishing between similar services

### Service Locator Types

Omni provides two types of Service Locators to manage service instances effectively within different scopes:

**Global Service Locator**

   - Accessible across all networked instances
   - Ideal for game-wide services
   - Manages universal service instances

**Local Service Locator**

   - Unique to each `NetworkIdentity`
   - Perfect for object-specific services
   - Maintains isolation between different networked objects

---

With this dual Service Locator approach, Omni offers a flexible, scalable structure that enhances dependency management in multiplayer environments, ensuring services are easily accessible while maintaining clear separation between global and local contexts.

=== "Global Service Locator"
     ```csharp
     // Example of Global Service Implementation
     public class UIManager : ServiceBehaviour
     {
         public void ShowMenu() { /* Implementation */ }
     }
     
     public class Player : NetworkBehaviour
     {
         void Example()
         {
             // Basic service retrieval
             var uiManager = NetworkService.Get<UIManager>();
             
             // Named service retrieval
             var customUI = NetworkService.Get<UIManager>("MainMenuUI");
         }
     }
     ```

=== "Local Service Locator"
     ```csharp
     // Example of Local Service Implementation
     public class WeaponSystem : NetworkBehaviour
     {
         public void Fire() { /* Implementation */ }
     }
     
     public class Player : NetworkBehaviour
     {
         void Example()
         {
             // Get service from same NetworkIdentity
             var weapon = Identity.Get<WeaponSystem>();
             
             // Get named local service
             var secondaryWeapon = Identity.Get<WeaponSystem>("SecondaryWeapon");
         }
     }
     ```

!!! note
    The service locator can find services regardless of their depth within the hierarchy, ensuring accessibility even in deeply nested objects.

!!! warning "Multiple Services"
    **When using multiple instances of same service:**

    1. Set unique names in Inspector
    2. Use named lookup:
    ```csharp
    // Get specific weapon manager
    var pistol = Identity.Get<WeaponManager>("PistolManager");
    var rifle = Identity.Get<WeaponManager>("RifleManager");
    ```

    An exception will be thrown if the service is not found or if multiple instances of the same type have the same name.

---

### Dependency Injection Support

Omni provides a powerful dependency injection system using source generation:

```csharp
public partial class Player : NetworkBehaviour
{
    // Global service injection
    [GlobalService]
    private ServerManager serverManager;

    // Local service injection
    [LocalService]
    private WeaponSystem primaryWeapon;

    // Named service injection
    [LocalService("Secondary")]
    private WeaponSystem secondaryWeapon;

    // Properties are also supported
}
```

#### Important Considerations

!!! warning "Dependency Injection Lifecycle"

    - Services are **NOT** available during `Awake()`
    - Available from `Start()` onwards
    - Use manual service access in `Awake()` if needed

!!! tip
    The Service Locator is designed to be fast and efficient, with minimal performance cost, unlike `GetComponent`, making it suitable for frequent use.

---

## Common Pitfalls

1. **Multiple Services**
```csharp
// Correct way to handle multiple instances
var pistol = Identity.Get<WeaponSystem>("PistolSystem");
var rifle = Identity.Get<WeaponSystem>("RifleSystem");
```

2. **Service Availability**
```csharp
// Always check service availability
if (NetworkService.TryGet<UIManager>(out var uiManager))
{
    uiManager.ShowMenu();
}
```

---

## Best Practices

**Service Naming**

   - Use clear, descriptive names for services
   - Avoid duplicate names within the same scope
   - Follow a consistent naming convention

**Error Handling**

   - Always check for null references
   - Handle service not found scenarios
   - Use try-get patterns for optional services

**Performance Optimization**

   - Cache service references when possible
   - Avoid unnecessary service lookups
   - Use dependency injection for frequently accessed services

---

*See the API Reference for more information about the Service Locator and its usage.*