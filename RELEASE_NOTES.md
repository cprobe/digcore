# digcore v0.1.0 Release Notes

## 🎉 Initial Release

digcore 是从 catpaw 项目中提取的 AI 驱动的智能诊断框架，用于构建智能监控 Agent。

## 📦 包含的模块

### 核心模块
- **types**: 事件类型定义和状态常量
- **engine**: 事件处理引擎（去重、告警判定、恢复通知）
- **plugins**: 插件接口定义（Gatherer、Initer、Diagnosable）

### AI 诊断
- **diagnose**: AI 诊断引擎和工具注册表
- **diagnose/aiclient**: 多模型支持（OpenAI、Bedrock、Gateway）
- **mcp**: Model Context Protocol 客户端

### 通知系统
- **notify**: 多渠道通知系统
  - Console: 控制台输出
  - WebAPI: 通用 HTTP 推送
  - Flashduty: Flashduty 告警平台
  - PagerDuty: PagerDuty 事件管理

### 配置和工具
- **config**: 配置框架（AI、Notify、MCP）
- **pkg**: 通用工具包（queue、shell、tls、cmdx 等）
- **logger**: 日志工具

## 🚀 使用方式

### 安装

```bash
go get github.com/cprobe/digcore@v0.1.0
```

### 创建插件

```go
import (
    "github.com/cprobe/digcore/types"
    "github.com/cprobe/digcore/plugins"
    "github.com/cprobe/digcore/pkg/safe"
)

type MyPlugin struct {}

func (p *MyPlugin) Gather(queue *safe.Queue[*types.Event]) {
    event := types.BuildEvent(map[string]string{
        "check": "myplugin::health",
    })
    event.SetEventStatus(types.EventStatusOk)
    queue.PushFront(event)
}

func init() {
    plugins.Add("myplugin", func() plugins.Plugin {
        return &MyPlugin{}
    })
}
```

## 📊 统计数据

- **文件数**: 85
- **代码行数**: 12,699
- **模块数**: 10

## 🔗 相关项目

- [catpaw](https://github.com/cprobe/catpaw) - 宿主机监控 Agent（基于 digcore）
- k8spaw - Kubernetes 监控 Agent（商业版，基于 digcore）

## 📝 文档

- [README.md](README.md) - 项目介绍
- [MIGRATION.md](MIGRATION.md) - 迁移指南
- [examples/](examples/) - 使用示例

## 🙏 致谢

感谢 catpaw 项目的所有贡献者，digcore 是从 catpaw 中提取的核心代码。

## 📄 许可证

Apache License 2.0
