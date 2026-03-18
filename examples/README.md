# digcore Examples

This directory contains example implementations using digcore.

## Basic Plugin Example

See how to create a simple monitoring plugin:

```go
package main

import (
    "github.com/cprobe/digcore/types"
    "github.com/cprobe/digcore/plugins"
    "github.com/cprobe/digcore/pkg/safe"
)

// MyPlugin implements the Gatherer interface
type MyPlugin struct {
    Interval config.Duration `toml:"interval"`
}

func (p *MyPlugin) Gather(queue *safe.Queue[*types.Event]) {
    event := types.BuildEvent(map[string]string{
        "check": "myplugin::health",
        "target": "localhost",
    })

    event.SetEventStatus(types.EventStatusOk)
    event.SetDescription("Everything is fine")

    queue.PushFront(event)
}

func init() {
    plugins.Add("myplugin", func() plugins.Plugin {
        return &MyPlugin{}
    })
}
```

## Event Processing Example

```go
package main

import (
    "github.com/cprobe/digcore/engine"
    "github.com/cprobe/digcore/types"
    "github.com/cprobe/digcore/notify"
)

func main() {
    // Register notifiers
    notify.Register(notify.NewConsoleNotifier(&config.ConsoleConfig{Enabled: true}))

    // Process events
    queue := safe.NewQueue[*types.Event](100)

    event := types.BuildEvent(map[string]string{
        "check": "test::check",
    })
    event.SetEventStatus(types.EventStatusCritical)

    queue.PushFront(event)

    engine.PushRawEvents("test", pluginObj, instance, queue)
}
```

## More Examples

- See [catpaw](https://github.com/cprobe/catpaw) for a complete host monitoring implementation
