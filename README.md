# Object Link Core Protocol

The Object Link protocol is a simple protocol to link a local object to a remote object. The protocol is designed to be independent from the transport but was designed with websockets in mind.

For this it can be used from browsers, desktop clients or even smaller embedded devices. Due to support for binary message formats and an efficient protocol structure it is possible to send data very efficient.

[Full Documentation](https://objectlink.netlify.app/)

## Features

* Link/Unlink of remote objects
* Sync properties between remote object and linked local objects
* Asynchronous remote method invocation
* Server side signals to all linked objects
* Fully support for [ApiGear](http://apigear.io) object model and code generation tooling

The protocol itself is isolated from any transport technology to make it easier to port.


## Message Types

* [Lifecycle](lifecycle)
	* `--> LINK` - link the local object with a remote object
	* `<-- INIT` - initialized the local object with properties from the remote object
	* `--> UNLINK` - unlinks a local object from a remote object
* [Properties](properties)
    * `--> SET_PROPERTY`  - send a property change to a remote object
	* `<-- PROPERTY_CHANGE` - sends property changes to all linked client objects
* [Methods](methods)
	* `--> INVOKE` - invoke a method on a remote object
	* `<-- INVOKE_REPLY` - reply of an remote invokation
* [Signals](signals)
	* `<-- SIGNAL` - send remote events back to all linked client objects


## Implementations

* [ObjectLink Core Typescript](https://github.com/apigear-io/objectlink-core-typescript)
* [ObjectLink Core Python](https://github.com/apigear-io/objectlink-core-python)
* [ObjectLink Core C++14](https://github.com/apigear-io/objectlink-core-cpp)


