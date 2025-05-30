# MsgBus

A generic, concurrent message bus implementation in Go that provides a publish-subscribe pattern for distributing messages across multiple subscribers.

## Features

- Generic type support for message payloads
- Thread-safe operations
- Support for multiple subscribers per topic
- Timeout handling for message publishing
- Concurrent message delivery

## Installation

```bash
go get github.com/Aj4x/msgbus
```

Requires Go 1.23.0 or later.

## Dependencies

- [github.com/Aj4x/uuid](https://github.com/Aj4x/uuid) - For generating unique subscription identifiers

## Usage

### Creating a Message Bus

```go
// Create a new message bus with byte slice payload type
bus := msgbus.NewMessageBus[[]byte]()

// Or with any other type
stringBus := msgbus.NewMessageBus[string]()

// For custom types
type MyCustomType struct {
    Data string
}
customBus := msgbus.NewMessageBus[MyCustomType]()
```

### Subscribing to a Topic

```go
package main

import (
    "fmt"
    "github.com/Aj4x/msgbus"
)

func subscribeExample() {
    // Create a message bus
    bus := msgbus.NewMessageBus[[]byte]()

    // Create a handler channel
    handler := make(msgbus.MessageHandler[[]byte], 10)

    // Subscribe to a topic
    topic := msgbus.Topic("my-topic")
    subscriptionID, err := bus.Subscribe(topic, handler)
    if err != nil {
        // Handle error
        fmt.Printf("Error: %v\n", err)
        return
    }

    // Process messages in a goroutine
    go func() {
        for msg := range handler {
            fmt.Printf("Received message on topic %s: %v\n", msg.Topic, msg.Message)
            // Process the message
        }
    }()

    // Use subscriptionID later for unsubscribing
    _ = subscriptionID
}
```

### Publishing Messages

```go
package main

import (
    "github.com/Aj4x/msgbus"
)

func publishExample() {
    // Create a message bus
    bus := msgbus.NewMessageBus[[]byte]()

    // Create a message
    message := msgbus.TopicMessage[[]byte]{
        Topic:   msgbus.Topic("my-topic"),
        Message: []byte("Hello, World!"),
    }

    // Publish the message
    bus.Publish(message)
}
```

### Unsubscribing

```go
package main

import (
    "github.com/Aj4x/msgbus"
)

func unsubscribeExample() {
    // Create a message bus
    bus := msgbus.NewMessageBus[[]byte]()

    // Create a handler channel
    handler := make(msgbus.MessageHandler[[]byte], 10)

    // Subscribe to a topic
    topic := msgbus.Topic("my-topic")
    subscriptionID, _ := bus.Subscribe(topic, handler)

    // Unsubscribe using the topic and subscription ID
    bus.Unsubscribe(topic, subscriptionID)

    // Close the handler channel
    close(handler)
}
```

## Concurrency

The message bus is designed to be thread-safe and can be accessed concurrently from multiple goroutines. Message publishing is done concurrently with a timeout of 5 seconds per subscriber.

## Example

```go
package main

import (
    "fmt"
    "github.com/Aj4x/msgbus"
    "time"
)

func main() {
    // Create a new message bus for string messages
    bus := msgbus.NewMessageBus[string]()

    // Create a handler channel
    handler := make(msgbus.MessageHandler[string], 10)

    // Subscribe to a topic
    topic := msgbus.Topic("greetings")
    _, err := bus.Subscribe(topic, handler)
    if err != nil {
        fmt.Printf("Error subscribing: %v\n", err)
        return
    }

    // Process messages in a goroutine
    go func() {
        for msg := range handler {
            fmt.Printf("Received: %s\n", msg.Message)
        }
    }()

    // Publish messages
    bus.Publish(msgbus.TopicMessage[string]{
        Topic:   topic,
        Message: "Hello, World!",
    })

    bus.Publish(msgbus.TopicMessage[string]{
        Topic:   topic,
        Message: "Welcome to MsgBus!",
    })

    // Wait for messages to be processed
    time.Sleep(time.Second)
}
```

## License

MIT License

Copyright (c) 2025 Aj4x
