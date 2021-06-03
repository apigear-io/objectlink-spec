# Using ObjectLink

This shows the usage of the different object link impementations using conceptual code snippets. For the exact usage, please consult the documentation of different implementation repositories.

An object link implementation comes with a `ClientNode` and a `RemoteNode` class. A client node manages the client objects, called sinks and the remote node the server objects called sources. 

Internally these nodes use a global registry where all sinks (or sources) will be registered with their object name. This means the same object name can not be registered twice in the same program.


## Example Client Side

The client side consist of object sinks which define the interface to the remote objects and client nodes for each connection to the remote object server.


### Object Sink

The object sink is the client side interface for the object and act as the counter part of the remote side object source.

*Note: In the future there will be generic base classes to minimize the client code.*

Here is a typescript example.

```ts

class Counter implements IObjectSink {
	// our properties
	_count:0
	node: ClientNode | null    
	constructor() {
		// register this sink
		this.node = ClientNode.addObjectSink(this)
	}
    get count(): number {
        return this._count
    }
    set count(value: number) {
    	if(_count !== value) { // guard the changes
        	this.node?.setRemoteProperty('demo.Counter/count', value)
        }
    }

   	increment() {
   		// remote method invokation, without args or callback
		this.node?.invokeRemote('demo.Counter/increment')
	}

	olinkObjectName(): string {
        return 'demo.Counter'
    }
    olinkOnSignal(name: string, value: any): void {
    	// handle server side signals
        const path = Name.pathFromName(name)
        console.log('Counter.onSignal', path, value)
    }
    olinkOnPropertyChanged(name: string, value: any): void {
    	// handle server side property changes
        const path = Name.pathFromName(name)
        this[`_${path}`] = value
    }
    olinkOnInit(name: string, props: any): void {
    	// receive initial properties
        this.count = props['count']
    }
    olinkOnRelease(): void {
    	// release sink
    }    
}

```


### Websocket client using the Client Node

For the client we need a websocket connection to a server to send object link messages. The client needs to send incoming message to the remote node and listen to write request from the client node. There will be one node per websocket connection.

```ts
import WebSocket from 'ws'
import ClientNode from 'olink'

ready = false
queue = []

function processMessages() {
	if(!ready) {
		return
	}
	// empty queue
	while(queue.length > 0) {
		const msg = this.queue.shift()
		ws.send(msg)
	}
}

const node = ClientNode()
node.onWrite((msg) => {
	// node might send message 
	// while we are not connected yet
	queue.push(msg)
	processMessages()
}

// create websocket
const ws = new WebSocket(address)
ws.on('open', (ws) => {
	ready = true
	//
	processMessages()
})
ws.on('message', (data) => {
	// let our node handle all messages
	node.handleMessage(data.toString())
}
```


## Example Server Side

The remote server contains the object sources for the object communication using the websocket protocol.

### WebSocket Server using Remote Node

For the server we need a websocket server which sends messages to a remote node and listens to write requests from the remote node.

```ts
import WebSocket from 'ws'

const wss = new WebSocket.Server(options);
wss.on('connection', (ws) => {
	const remote = new RemoteNode()
	remote.onWrite((msg) => {
		ws.send(msg)
	})
	ws.on('message', (data) =>  {
		remote.handleMessage(data.toString())
	}
});
```

To register objects to the remote node, we need to create an object adapter and register it.

### Implementation Object

First we create our implementation object, which will carry our business logic.

```ts
class Counter {
	count: 0
	increment() {
		this.count++
		RemoteNode.propertyChange('demo.Counter/count')
	}
	signalShutdown() {
		RemoteNode.signal('demo.Counter/shutdown')	
	}
}

```


### Object Source the Adapters

An adapter will then use the implementation and communicate with client using the remote node.


```ts
class CounterAdapter implements IObjectSource {
	impl: Couter 
	constructor(impl: Counter) {
		this.impl = impl
		RemoteNode.addObjectSource(this)
	}
    olinkObjectName(): string {
        return 'demo.Counter'
    }
    olinkInvoke(name: string, args: any[]): any {
        const path = Name.pathFromName(name)
        switch(path) {
        	case 'increment':
        		this.impl.increment()
        		break;
        }
    }
    olinkSetProperty(name: string, value: any): void {
    	const path = Name.pathFromName(name)
    	if (this.impl[path] !== value) {
    		this.impl[path] = value
    		RemoteNode.propertyChange(`demo.Counter/{path}`, value)
    	}
    }
    olinkCollectProperties(): any {
    	return {
    		'count': this.impl.count
    	}
    }
    olinkLinked(name: string, node: RemoteNode): any {
    	// object was linked
    }
}

```


## ApiGear Object Model

Using ApiGears object model and the associated template, all code on client ans remote side including a websocket server will be generated for you including a simulation model for the data format and documentation.

