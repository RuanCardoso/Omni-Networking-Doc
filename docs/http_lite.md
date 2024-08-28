# Overview

**Http Lite** is a simple simulation of `Express.js` and is one of the most useful features of the API. It can be easily used to request a route and receive a response from the server. Routes can also send responses to multiple clients beyond the one that originally requested the route.

# Basic Usage

1. Import the `Http Lite` module with `using static Omni.Core.HttpLite;`
2. Register the routes on the `Awake` method or on the `Start` method, eg:

!!! Note

    Http Lite supports both asynchronous and synchronous operations, with all functions having their asynchronous versions.

```csharp
Http.GetAsync("/login", (res) =>
{
    res.WriteString("Hello World!");
    res.Send();
});
```

!!! tip

    Send() has overloads and optional arguments, allowing you to specify the recipient, delivery method, group, channel, cache, and more.

- Send with Target and Delivery Method (optional)

```csharp
Http.GetAsync("/login", (res) =>
{
    res.WriteString("Hello World!");
    res.Send(HttpTarget.GroupMembers, DeliveryMode.ReliableOrdered); // Send the response to all group members & deliver in reliable order
});
```

!!! note

    When specifying that a route should send the response to clients other than the one that requested the route, you must ensure that these clients also register the route to receive the response.

    The default `HttpTarget` is `Self`, meaning that only the client that requested the route will receive the response.

- Request the route from the client.

```csharp
async void RequestLogin()
{
    using var res = await Fetch.GetAsync("/login");
    string result = res.ReadString();
    print(result);
}
```

Here is the complete `example` if you want to test it. Attach this code to a script that is in the scene and press `Enter`.

```csharp
void Start()
{
    Http.GetAsync("/login", (res) =>
    {
        res.WriteString("Hello World!");
        res.Send();
    });
}

async void RequestLogin()
{
    using var res = await Fetch.GetAsync("/login");
    string result = res.ReadString();
    print(result);
}

private void Update()
{
    if (Input.GetKeyDown(KeyCode.Return))
    {
        RequestLogin();
    }
}
```

After pressing `Enter`, the console should display -> "Hello World".

**Remember, this example uses the asynchronous method, which I always recommend using.**

Here is an example of the asynchronous version of `Post`.

- Register a `Post` route.

```csharp
Http.PostAsync("/login", (req, res) =>
{
    // Read the username sent in the request
    string username = req.ReadString()
    // Send a response
    res.WriteString("Your are logged in as " + username);
    res.Send();
});
```

- Request the route from the client.

```csharp
using var res = await Fetch.PostAsync("/login", req =>
{
    req.WriteString("John Doe");
})

string result = res.ReadString();
print(result);
```

Here is the complete `example` if you want to test it. Attach this code to a script that is in the scene and press `Enter`.

```csharp
void Start()
{
    Http.PostAsync("/login", (req, res) =>
    {
        // Read the username sent in the request
        string username = req.ReadString();
        // Send a response
        res.WriteString("Your are logged in as " + username);
        res.Send();
    });
}

async void RequestLogin()
{
    using var res = await Fetch.PostAsync("/login", req =>
    {
        req.WriteString("John Doe");
    });

    string result = res.ReadString();
    print(result);
}

private void Update()
{
    if (Input.GetKeyDown(KeyCode.Return))
    {
        RequestLogin();
    }
}
```

!!! tip

    On the server side, it is possible to obtain the client that is requesting the route by adding a third argument when registering the route.

    ```csharp
    Http.PostAsync("/login", (req, res, peer) => {});
    Http.GetAsync("/login", (res, peer) => {});
    ```

    `Peer` is a `NetworkPeer` object that represents the client that is requesting the route.
