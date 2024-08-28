# Overview

Omni is capable of serializing various types of data, including primitives, complex classes, structures, dictionaries, and more. Omni offers two types of serialization: JSON-based and binary-based serialization.

All network operations are performed using the `DataBuffer` object, which represents a buffer of data to be sent over the network.

# Basic Usage of DataBuffer

- Basic serialization

```csharp
void Start()
{
    DataBuffer buffer = new DataBuffer(1024); // Size of the buffer in bytes,     this argument is optional, the default size is 32768
    buffer.Write(10);
    buffer.WriteString("Hello World!");
}
```

- Using JSON serialization with `DataBuffer`

```csharp
public class Player
{
    public string name;
    public int score;
}

void Start()
{
    DataBuffer buffer = new DataBuffer(1024);
    buffer.WriteAsJson(new Player());
}
```

- Using binary serialization with `DataBuffer`

```csharp
[MemoryPackable]
public partial class Player
{
    public string health;
    public int score;
}

void Start()
{
    DataBuffer buffer = new DataBuffer(1024);
    buffer.WriteAsBinary(new Player());
}
```

- Complex serialization with `DataBuffer`

```csharp
[MemoryPackable]
public partial class Player
{
    public string health;
    public int score;
}

void Start()
{
    DataBuffer buffer = new DataBuffer(1024);
    buffer.Write(3287382);
    buffer.WriteAsBinary(new Player());
    buffer.WriteAsJson(new Player());
    buffer.WriteString("Hello World!");
}
```

!!! warning

    When sending a `DataBuffer`, you will always receive a `DataBuffer` in response; it is not possible to send and receive data in any other way without using a `DataBuffer`, as all operations utilize it internally. **You must also ensure that the reading and writing occur in the same order.**

    If you write an `integer` followed by a `string`, you must read an `integer` followed by a `string`. Never attempt to read the data in an order different from the one in which it was written, as this will result in incorrect data retrieval and potential errors.

Omni uses the [Newtonsoft.Json](https://www.newtonsoft.com/json) library for JSON serialization.

For binary serialization, Omni employs the [MemoryPack](https://github.com/Cysharp/MemoryPack) library.

For more details, please refer to the documentation:

- **JSON Serialization:** [Newtonsoft.Json Documentation](https://www.newtonsoft.com/json/help/html/introduction.htm)
- **Binary Serialization:** [MemoryPack GitHub Repository](https://github.com/Cysharp/MemoryPack)
