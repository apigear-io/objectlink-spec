# Using Typescript

The typescript object link library comes as a client and remote node. On the client side after a socket connection is established the incoming messages must be handled by the client node and the node emit write requests, which must be forwarded to the socket.


## Client Node


### Websocket client using the Client Node

For the client we need a websocket connection to a server to send object link messages. The client needs to send incoming message to the remote node and listen to write request from the client node. There will be one node per websocket connection.

```ts
import WebSocket from 'ws'

ready = false
queue = []

function process() {
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
	process()
}

const ws = new WebSocket(address)
ws.on('open', (ws) => {
	ready = true
	process()
})
ws.on('message', (data) => {
	node.handleMessage(data.toString())
}
```


### Client Object as Object Sink

The object sink is the client side object and act as the counter part of the remote side object source.

```ts

class Counter implements IObjectSink {
	_count:0
	client: ClientNode | null    
	constructor() {
		this.client = ClientNode.addObjectSink(this)
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
		this.client?.invokeRemote('demo.Counter/increment')
	}

	olinkObjectName(): string {
        return 'demo.Counter'
    }
    olinkOnSignal(name: string, value: any): void {
        const path = Name.pathFromName(name)
        console.log('Counter.onSignal', path, value)
    }
    olinkOnPropertyChanged(name: string, value: any): void {
    	// update local property
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


## Remote Node Server

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

