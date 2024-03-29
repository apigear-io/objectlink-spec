# Introduction

The Object Link protocol is a simple protocol to link a local object to a remote object. The protocol is designed to be independent from the transport but was designed with websockets in mind.

For this it can be used from browsers, desktop clients or even smaller embedded devices. Due to support for binary message formats and an efficient protocol structure it is possible to send data very efficient.

The protocol is defined as a clean [protocol specification](../protocol) and as a set of reference implementations of the core protocol.

## Core Specification

The core protocol specification defines the message formats and sequence of messages. It is independent from the transport and language.

- [ObjectLink Core Specification](../protocol)

## Reference Implementations

The core protocol is implemented in the following languages:

- [ObjectLink Core Typescript](https://github.com/apigear-io/objectlink-core-typescript)
- [ObjectLink Core C++ 14](https://github.com/apigear-io/objectlink-core-cpp)
- [ObjectLink Core Python](https://github.com/apigear-io/objectlink-core-python)
- [ObjectLink Core Go](https://github.com/apigear-io/objectlink-core-go)

All reference implementation share similar tests and features and come with examples for using it with different transport libraries.
