# digcore 快速开始

## 安装

```bash
go get github.com/cprobe/digcore@v0.1.0
```

## 本地开发（推荐）

如果你需要同时修改 digcore 和你的项目，使用本地 replace：

```go
// go.mod
require github.com/cprobe/digcore v0.1.0

replace github.com/cprobe/digcore => ../digcore
```

## 基础用法

### 1. 创建插件

```go
package myplugin

import (
    "github.com/cprobe/digcore/config"
    "github.com/cprobe/digcore/plugins"
    "github.com/cprobe/digcore/types"
    "github.com/cprobe/digcore/pkg/safe"
)

type Plugin struct {
    Interval config.Duration `toml:"interval"`
    Labels   map[string]string `toml:"labels"`
}

func (p *Plugin) GetLabels() map[string]string {
    return p.Labels
}

func (p *Plugin) GetInterval() config.Duration {
    return p.Interval
}

func (p *Plugin) Gather(queue *safe.Queue[*types.Event]) {
    event := types.BuildEvent(map[string]string{
        "check": "myplugin::health",
        "target": "localhost",
    })
    
    // 设置状态
    event.SetEventStatus(types.EventStatusOk)
    
    // 设置描述
    event.SetDescription("Service is healthy")
    
    // 设置当前值（可选）
    event.SetCurrentValue("100%")
    
    queue.PushFront(event)
}

func init() {
    plugins.Add("myplugin", func() plugins.Plugin {
        return &Plugin{}
    })
}
```

### 2. 注册通知渠道

```go
package main

import (
    "github.com/cprobe/digcore/config"
    "github.com/cprobe/digcore/notify"
)

func main() {
    // Console 通知
    notify.Register(notify.NewConsoleNotifier(&config.ConsoleConfig{
        Enabled: true,
    }))
    
    // WebAPI 通知
    notify.Register(notify.NewWebAPINotifier(&config.WebAPIConfig{
        URL: "https://your-api.example.com/events",
        Headers: map[string]string{
            "Authorization": "Bearer token",
        },
    }))
    
    // Flashduty 通知
    notify.Register(notify.NewFlashdutyNotifier(&config.FlashdutyConfig{
        IntegrationKey: "your-key",
    }))
}
```

### 3. 处理事件

```go
package main

import (
    "github.com/cprobe/digcore/engine"
    "github.com/cprobe/digcore/plugins"
)

func main() {
    // 创建插件实例
    pluginObj := plugins.PluginCreators["myplugin"]()
    
    // 创建事件队列
    queue := safe.NewQueue[*types.Event](100)
    
    // 采集数据
    plugins.MayGather(pluginObj, queue)
    
    // 处理事件
    engine.PushRawEvents("myplugin", pluginObj, instance, queue)
}
```

### 4. 注册诊断工具（可选）

```go
package myplugin

import (
    "context"
    "github.com/cprobe/digcore/diagnose"
)

func (p *Plugin) RegisterDiagnoseTools(registry *diagnose.ToolRegistry) {
    registry.Register(diagnose.Tool{
        Name: "check_service_status",
        Description: "Check service status",
        Execute: func(ctx context.Context, params map[string]string) (string, error) {
            // 实现诊断逻辑
            return "Service is running", nil
        },
    })
}
```

## 事件状态

```go
types.EventStatusCritical  // 严重告警
types.EventStatusWarning   // 警告
types.EventStatusInfo      // 信息
types.EventStatusOk        // 正常（恢复）
```

## 配置示例

```toml
# config.toml
[global]
interval = "60s"
labels = { env = "prod", region = "us-west" }

[notify.console]
enabled = true

[notify.webapi]
url = "https://your-api.example.com/events"
method = "POST"
timeout = "10s"

[notify.flashduty]
integration_key = "your-key"

[ai]
enabled = true
model_priority = ["default"]

[ai.models.default]
base_url = "https://api.openai.com/v1"
api_key = "${OPENAI_API_KEY}"
model = "gpt-4o"
```

## 完整示例

参考 [catpaw](https://github.com/cprobe/catpaw) 项目查看完整的实现示例。

## 下一步

- 阅读 [MIGRATION.md](MIGRATION.md) 了解架构设计
- 查看 [examples/](examples/) 目录获取更多示例
- 访问 [catpaw](https://github.com/cprobe/catpaw) 查看生产级实现

## 获取帮助

- GitHub Issues: https://github.com/cprobe/digcore/issues
- 参考项目: https://github.com/cprobe/catpaw
