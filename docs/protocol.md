# ObjectLink Protocol

ObjectLink is a protocol to link a remote object to a local object using a websocket based IPC.

The communication protocol allows objects to communicate with each other. A object is a collection of properties, operations and signals and are described using the ApiGear ObjectAPI.

In general a local object will communicate with a remote object and properties will be synced, operation can be called and remote signals can be received. The use case is to extend the local objects with remote capabilities on many platforms and technologies. 


* The protocol shall be simple to implement
* The protocol shall be be fast and light
* The protocol shall support all JSON (MsgPack, CBOR) types as payload
* The protocol shall be implementable using different programming languages and technologies.
* The protocol also designed for web sockets shall not depend on web sockets
* TODO: expand requirements for use cases

* The protocol shall be symmetrical (client and server are the same, e.g. nodes)
* The protocol shall be testable with client and service inside one process
* The protocol shall be testable without a network connection
* Many the same sink can connect from many client to one server and share the same source
* A sink can only be available on one client
* The protocol shall fully support ObjectAPIs API spectrum (synced properties, async operations, broadcasted signals)
* A client only subscribes to named interfaces (as objects)
* A server only exposes named interfaces
* To link a sink to a source the source needs to be on a known server and be registered
* A client links a sink using the source name (e.g. `${module}.${interface}`)
* Always fully qualified names are transmitted (e.g. `${module}.${interface}` for interface and `${module}.${interface}/${path}`) for properties, operations and signals.
* The same sinks on all connections are notified about property changes and server side signals.
* Operation results are only written back to the same connection, resulting changes (e.g. property changes, signals) are synced to all the same sinks connected.



## Use Case

* Invoke a remote operation from a client
* Sync properties between different clients which link the same object
* Receive signals by all clients which link the same object

### Remote Operation Invocation

## Message Types

* Lifecycle
	* --> `LINK` - link the socket with the object
	* <-- `INIT` - after link message and sends all collected properties back to client
	* --> `UNLINK` - unlinks a socket from an object
* Properties
  * --> `SET_PROPERTY`  - send a property change to service
	* <-- `PROPERTY_CHANGE` - sends a property change to all linked clients
* Operations
	* --> `INVOKE` - invoke a remote operation
	* <-- `INVOKE_REPLY` - reply of an invoke
* Signals
	* <-- `SIGNAL` - send remote events back to linked clients

## Lifecycle

To use an object the object needs to be linked first. This will populate the local properties from remote, as also enable property changes and signals. After the linking the object is fully usable.

```js
--> [ LINK, "org.demos.Echo"]
```

```js
<-- [ INIT, 'org.demos.Echo', { message: "hello" } ]
```


To release the resources on the server side the object can also be unlinked.

```js
--> [ UNLINK, "org.demos.Echo"]
```


## Property Changes

```yaml
name: org.demos

interfaces:
  - name: Echo
    properties:
      - name: message
        type: string
```

```js
// org.demos.js
class Echo {
  message: string = ""
}
const echo = new Echo()
echo.message = "foo"
```

subscribe to the object

```js
--> [ LINK, "org.demos.Echo"]
```

receive initial property list, which is automatically send after a link.

```js
<-- [ INIT, 'org.demos.Echo', { message: "hello" } ]
```

After the init message the object is fully created and all properties have valid values.

Send to the remote object the changed message property ("hello" -> "foo"). 

```js
--> [ PROPERTY_CHANGE, "org.demos.Echo/message", "foo"]
```

And send to all subscribed clients the changes.

```js
<-- [ PROPERTY_CHANGE, "org.demos.Echo/message", "foo"]
```


## Invoke Operations

To invoke remote operations an operation identifier and the operation arguments must be specified. The reply can only have on value, which can have any complexity.

```yaml
name: org.demos

interfaces:
  - name: Echo
    operations:
      - name: say
        params:
        - name: msg
          type: string
        type: string
```

```js
// org.demos.js
class Echo {
  async say(msg: string): string
}

const echo = new Echo()
console.log(echo.say("hello"))
$> hello
````

Send from the client, the ID (1) is unique for the client

```js
--> [ INVOKE, 1, "org.demos.Echo/say", ["echo"]]
```

Send reply to the same client

```js
<-- [ INVOKE_REPLY, 1, "echo"]
```


## Signal Notification

After a client is linked to an object it will start receiving property changes and signals.


```yaml
name: org.demos

interfaces:
  - name: Echo
    signals:
      - name: shutdown
        params:
        - name: timeout
          type: int
```

```js
// org.demos.js
class Echo {
  notifyShutdown(timeout: number)	
}
const echo = new Echo()
echo.notifyShutdown(10)
```


```js
--> [ LINK, "org.demos.Echo"]
```

```js
<-- [ INIT, "org.demos.Echo", {}]
```

```js
<-- [ SIGNAL, "org.demos.Echo/shutdown", [10]]
```

# Advanced

## Resource Identifier

It is possible to provide a resource identifier ID map, which maps numeric IDs to identifier strings.

```
{
  1: "org.demos.Echo/shutdown"
}
```

This map can be auto-generated from the API description can be loaded by the client on startup.

When enabled the resource identifier string will be replaced with the numeric IDs.

So instead of sending

```js
--> [ SIGNAL, "org.demos.Echo/shutdown", [10]]
```

```js
--> [ SIGNAL, 1, [10]]
```


# Issues
