# Database System

!!! note "Modified Database Libraries"
    Omni Networking uses customized versions of [SQLKata](https://sqlkata.com/docs) and [Dapper](https://github.com/DapperLib/Dapper) modified for Unity:

    **Improved:**

    - ✨ Reduced memory allocations and garbage collector pressure
    - 🔄 Async/await support with Task/ValueTask/UniTask
    - ⚡ Added support for IL2CPP compilation

    These modifications ensure better performance and compatibility within Unity's ecosystem while maintaining the original libraries' ease of use.

    !!! warning "IL2CPP Compatibility"
        Some database features may have limited compatibility when using IL2CPP compilation. However, all essential functionality remains available through alternative approaches supported by the Omni Networking framework.

## Configuration

!!! warning "Server-Side Only"
    The Omni Networking database module can currently only be used on the server side. Attempts to use database functionality on the client will result in errors.

    In the future, we plan to add support for local database operations on the client side, especially for databases like SQLite, enabling persistent offline data storage.

### Supported Databases

| Database      | Description | Compatibility |
|---------------|-----------|----------------|
| SQL Server    | Microsoft's enterprise database system with advanced security features, high scalability, and deep integration with other Microsoft products. Ideal for large-scale multiplayer game backends. | ✅ |
| MariaDB       | Enhanced MySQL fork with better performance, additional storage engines, and stronger community focus. Great for web applications and multiplayer games requiring high transaction rates. | ✅ |
| MySQL         | Popular relational database system optimized for web applications and online games with excellent cost-efficiency and widespread hosting support. | ✅ |
| PostgreSQL    | Advanced open-source database with robust JSON support, extensibility, and geospatial features. Recommended for complex game systems requiring sophisticated data structures. | ✅ |
| Oracle        | Enterprise-grade database system with unmatched reliability, security, and performance for mission-critical applications with high concurrent user loads. | ✅ |
| SQLite        | Self-contained, serverless SQL database engine perfect for client-side storage, mobile devices, and embedded systems where a full database server isn't needed. | ✅ |
| Firebird      | Open-source relational database with multi-generational architecture, excellent referential integrity, and small footprint. Well-suited for distributed applications and game analytics. | ✅ |

### Object Mappers

The framework supports two object mappers for converting between database data and C# objects:

- **Dapper (Default)**: High-performance mapper with low memory allocation but limited IL2CPP compatibility
- **NewtonsoftJson**: Full-featured mapper with complete IL2CPP support

!!! tip "Build Recommendations"
    When using databases in your project, it's recommended to:
    
    - Keep server builds on **Mono + Dapper** for maximum performance.
    - Use **IL2CPP** for client builds with **Newtonsoft Json** mapper when needed.

### Connection Configuration

=== "Using DbCredentials"
    ```csharp
    public class MyDatabaseManager : DatabaseManager
    {
        protected override void OnStart()
        {
            // Configure database connection using DbCredentials
            var credentials = new DbCredentials();
            credentials.SetConnectionString(
                DatabaseType.MySql,
                "localhost",
                "mydb",
                "user",
                "password"
            );
            SetDatabase(credentials);
        }
    }
    ```

=== "Using Connection String Directly"
    ```csharp
    public class MyDatabaseManager : DatabaseManager
    {
        protected override void OnStart()
        {
            // Configure database connection using type and connection string
            var credentials = new DbCredentials(
                DatabaseType.MySql,
                "localhost",
                "mydb",
                "user",
                "password"
            );
            SetDatabase(credentials);
        }
    }
    ```

!!! tip "Automatic Security in Builds"
    The `SetConnectionString` method is automatically stripped from non-server builds in release mode. This ensures database credentials are not included in client builds, enhancing your application's security.

### Connection Pooling

```csharp
// Configure connection pooling
SetDatabase(new DbCredentials
{
    Type = DatabaseType.MySql,
    ConnectionString = "Server=localhost;Database=mydb;Uid=user;Pwd=password;Pooling=true;MinPoolSize=5;MaxPoolSize=100;"
});
```

## Security Best Practices

### Credential Management

!!! warning "Never Hardcode Credentials"
    - Store credentials in environment variables or secure configuration files
    - Use different credentials for development and production
    - Rotate credentials regularly
    - Use managed identities where possible

```csharp
// Good: Use environment variables
var connectionString = Environment.GetEnvironmentVariable("DB_CONNECTION_STRING");

// Bad: Hardcoded credentials
var connectionString = "Server=localhost;Database=mydb;Uid=user;Pwd=password;";
```

### SQL Injection Prevention

!!! success "Always Use Parameterized Queries"
    - The ORM automatically handles parameterization
    - Never concatenate user input into SQL queries
    - Use the query builder for complex queries

```csharp
// Good: Parameterized query
await builder.Where("status", "=", userInput).FirstAsync();

// Bad: String concatenation
var query = $"SELECT * FROM users WHERE status = '{userInput}'";
```

### Data Validation

```csharp
// Validate input before database operations
if (!IsValidUserInput(userInput))
{
    throw new ArgumentException("Invalid input");
}

// Use strong typing
public class User
{
    [Required]
    [StringLength(50)]
    public string Name { get; set; }
    
    [EmailAddress]
    public string Email { get; set; }
}
```

### Access Control

```csharp
// Implement row-level security
public class SecureDatabaseManager : DatabaseManager
{
    protected override async Task<DatabaseConnection> GetConnectionAsync()
    {
        var conn = await base.GetConnectionAsync();
        
        // Set user context for row-level security
        await conn.RunAsync(
            "SET SESSION app.current_user_id = @userId",
            new { userId = GetCurrentUserId() }
        );
        
        return conn;
    }
}
```

## Performance Optimization

### Query Optimization

```csharp
// Use appropriate indexes
await builder
    .Where("status", "=", "active")
    .OrderBy("created_at")  // Ensure index exists on created_at
    .GetAsync<User>();

// Limit result sets
await builder
    .Where("status", "=", "active")
    .Limit(100)  // Prevent large result sets
    .GetAsync<User>();
```

### Connection Management

```csharp
// Properly dispose connections
using (var conn = await GetConnectionAsync())
{
    // Use connection
}

// Use connection pooling
SetDatabase(new DbCredentials
{
    ConnectionString = "Pooling=true;MinPoolSize=5;MaxPoolSize=100;"
});
```

### Caching Strategies

```csharp
// Implement caching for frequently accessed data
public class CachedUserRepository
{
    private readonly MemoryCache _cache = new MemoryCache();
    
    public async Task<User> GetUserAsync(int id)
    {
        var cacheKey = $"user_{id}";
        
        return await _cache.GetOrCreateAsync(cacheKey, async entry => {
            entry.SlidingExpiration = TimeSpan.FromMinutes(5);
            return await builder.Where(new { id }).FirstAsync<User>();
        });
    }
}
```

## Error Handling and Logging

### Structured Error Handling

```csharp
try
{
    using var db = await GetConnectionAsync();
    // Database operations
}
catch (DbException ex)
{
    // Handle database-specific errors
    NetworkLogger.__Log__($"Database error: {ex.Message}", NetworkLogger.LogType.Error);
}
catch (Exception ex)
{
    // Handle general errors
    NetworkLogger.__Log__($"Unexpected error: {ex.Message}", NetworkLogger.LogType.Error);
}
```

### Transaction Management

```csharp
using var db = await GetConnectionAsync();
using var transaction = db.BeginTransaction();

try
{
    // Perform multiple operations
    await builder.InsertAsync(new User { Name = "John" });
    await builder.InsertAsync(new Profile { UserId = 1, Bio = "Developer" });
    
    transaction.Commit();
}
catch
{
    transaction.Rollback();
    throw;
}
```

## Monitoring and Maintenance

### Health Checks

```csharp
public class DatabaseHealthCheck
{
    public async Task<bool> CheckHealthAsync()
    {
        try
        {
            using var db = await GetConnectionAsync();
            return await db.CheckConnectionAsync();
        }
        catch
        {
            return false;
        }
    }
}
```

### Performance Monitoring

```csharp
public class DatabaseMonitor
{
    public async Task LogSlowQueriesAsync(Query query, TimeSpan executionTime)
    {
        if (executionTime > TimeSpan.FromSeconds(1))
        {
            NetworkLogger.__Log__(
                $"Slow query detected: {query.ToSql()}\nExecution time: {executionTime}",
                NetworkLogger.LogType.Warning
            );
        }
    }
}
```

## Backup and Recovery

### Automated Backups

```csharp
public class DatabaseBackup
{
    public async Task CreateBackupAsync()
    {
        using var db = await GetConnectionAsync();
        await db.RunAsync("BACKUP DATABASE MyDB TO DISK = 'backup.bak'");
    }
}
```

### Point-in-Time Recovery

```csharp
public class DatabaseRecovery
{
    public async Task RestoreFromBackupAsync(DateTime pointInTime)
    {
        using var db = await GetConnectionAsync();
        await db.RunAsync(
            "RESTORE DATABASE MyDB FROM DISK = 'backup.bak' WITH RECOVERY, STOPAT = @pointInTime",
            new { pointInTime }
        );
    }
}
```

## Basic Usage

```csharp
// Get a database connection
var conn = await GetConnectionAsync();

// Get a query builder for a specific table
var builder = conn.GetBuilder("users");
```

## ORM Methods

### Basic CRUD Operations

=== "First/FirstAsync"
    ```csharp
    // Get first record without mapping
    var user = await builder.Where(new { id = 1 }).FirstAsync();
    
    // Get first record with mapping to User class
    var user = await builder.Where(new { id = 1 }).FirstAsync<User>();
    
    // Get first record with specific columns
    var user = await builder
        .Select("id", "name", "email")
        .Where(new { id = 1 })
        .FirstAsync<User>();
    ```

=== "All/AllAsync"
    ```csharp
    // Get all records without mapping
    var users = await builder.AllAsync();
    
    // Get all records with mapping to User class
    var users = await builder.AllAsync<User>();
    
    // Get all records with specific columns
    var users = await builder
        .Select("id", "name", "email")
        .AllAsync<User>();
    ```

=== "Insert/InsertAsync"
    ```csharp
    // Insert single record
    await builder.InsertAsync(new User { 
        Name = "John", 
        Email = "john@example.com" 
    });
    
    // Insert and get ID
    var id = await builder.InsertGetIdAsync<int>(new User { 
        Name = "John", 
        Email = "john@example.com" 
    });
    ```

=== "Update/UpdateAsync"
    ```csharp
    // Update single record
    await builder
        .Where(new { id = 1 })
        .UpdateAsync(new { name = "John Doe" });
    
    // Update multiple records
    await builder
        .Where("status", "=", "inactive")
        .UpdateAsync(new { status = "active" });
    ```

=== "Delete/DeleteAsync"
    ```csharp
    // Delete single record
    await builder
        .Where(new { id = 1 })
        .DeleteAsync();
    
    // Delete multiple records
    await builder
        .Where("status", "=", "inactive")
        .DeleteAsync();
    ```

=== "Exists/ExistsAsync"
    ```csharp
    // Check if record exists
    var exists = await builder
        .Where(new { id = 1 })
        .ExistsAsync();
    
    // Check if any records match condition
    var exists = await builder
        .Where("status", "=", "active")
        .ExistsAsync();
    ```

### Advanced ORM Operations

=== "Pagination"
    ```csharp
    // Basic pagination
    var page = await builder
        .ForPage(1, 10)
        .GetAsync<User>();
    
    // Pagination with ordering
    var page = await builder
        .OrderBy("created_at", "desc")
        .ForPage(2, 10)
        .GetAsync<User>();
    ```

=== "Aggregations"
    ```csharp
    // Count records
    var count = await builder.CountAsync<int>();
    
    // Sum values
    var total = await builder.SumAsync<int>("points");
    
    // Average values
    var avg = await builder.AverageAsync<double>("score");
    
    // Min/Max values
    var min = await builder.MinAsync<int>("age");
    var max = await builder.MaxAsync<int>("age");
    ```

=== "Joins"
    ```csharp
    // Basic join
    var results = await builder
        .Join("profiles", "users.id", "profiles.user_id")
        .Select("users.*", "profiles.bio")
        .GetAsync<User>();
    
    // Multiple joins
    var results = await builder
        .Join("profiles", "users.id", "profiles.user_id")
        .Join("roles", "users.role_id", "roles.id")
        .Select("users.*", "profiles.bio", "roles.name as role_name")
        .GetAsync<User>();
    ```

=== "Chunking"
    ```csharp
    // Process records in chunks
    await builder.ChunkAsync(100, async (users, page) => {
        foreach (var user in users)
        {
            // Process each user
        }
        return true; // Continue to next chunk
    });
    ```

## Manual SQL Operations

=== "Raw SQL with Dapper"
    ```csharp
    // Execute raw SQL
    var users = await conn.Connection.QueryAsync<User>(
        "SELECT * FROM users WHERE status = @status",
        new { status = "active" }
    );
    
    // Execute stored procedure
    var result = await conn.Connection.QueryAsync<User>(
        "sp_get_user",
        new { userId = 1 },
        commandType: CommandType.StoredProcedure
    );
    ```

=== "SQLKata Query Builder"
    ```csharp
    // Build complex queries
    var query = builder
        .Select("users.*", "profiles.bio")
        .Join("profiles", "users.id", "profiles.user_id")
        .Where("users.status", "=", "active")
        .Where("profiles.verified", "=", true)
        .OrderBy("users.created_at", "desc")
        .Limit(10);
    
    // Get raw SQL
    var sql = query.ToSql();
    
    // Execute query
    var results = await query.GetAsync<User>();
    ```

## IL2CPP Considerations

When using IL2CPP compilation:

- Some reflection-based features may be limited
- Use strongly-typed queries when possible
- Consider using the `UseLegacyPagination` option for Oracle databases
- Test database operations thoroughly in IL2CPP builds

