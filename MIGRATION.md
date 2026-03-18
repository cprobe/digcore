# digcore 迁移记录

## 概述

digcore 是从 catpaw 项目中提取的共享核心库，用于构建 AI 驱动的智能监控 Agent。

## 架构设计

```
github.com/cprobe/digcore      # 开源诊断核心库
github.com/cprobe/catpaw       # 开源宿主机监控（依赖 digcore）
private-repo/k8spaw            # 闭源 K8s 监控（依赖 digcore）
```

## 迁移的模块

### 核心模块（完整迁移）

| 模块 | 说明 | 文件数 |
|------|------|--------|
| `types/` | 核心事件类型、状态常量 | 2 |
| `pkg/` | 通用工具包（queue、shell、tls 等） | 13 |
| `config/` | 配置框架（AI、Notify、MCP） | 12 |
| `plugins/` | 插件接口定义（Gatherer、Initer、Diagnosable） | 1 |
| `notify/` | 通知层（Console、WebAPI、Flashduty、PagerDuty） | 5 |
| `mcp/` | MCP 客户端 | 5 |
| `diagnose/` | AI 诊断引擎 | 24 |
| `engine/` | 事件处理引擎 | 2 |
| `logger/` | 日志工具 | 1 |
| `server/` | Agent ID 管理 | 1 |

**总计**: 85 个文件，12699 行代码

### catpaw 保留的模块

| 模块 | 说明 |
|------|------|
| `plugins/*/` | 具体插件实现（cpu、mem、disk 等 25+ 插件） |
| `diagnose/tools/` | 宿主机诊断工具（top、ss、dmesg 等 70+ 工具） |
| `agent/` | Agent 生命周期管理 |
| `chat/` | 交互式 Chat REPL |
| `server/` | WebSocket 服务器（除 agentid.go） |
| `main.go` | CLI 入口 |

## 依赖关系

### digcore 的依赖

```go
require (
    github.com/jackpal/gateway v1.1.1
    github.com/toolkits/pkg v1.3.11
    github.com/koding/multiconfig v0.0.0-20171124222453-69c27309b2d7
    go.uber.org/zap v1.27.1
    github.com/gobwas/glob v0.2.3
    github.com/shirou/gopsutil/v3 v3.24.5
    github.com/google/uuid v1.6.0
    // ... 其他标准库
)
```

### catpaw 的依赖

```go
require (
    github.com/cprobe/digcore v0.0.0  // 核心依赖
    // ... 其他 catpaw 特定依赖
)

replace github.com/cprobe/digcore => ../digcore  // 本地开发
```

## 迁移步骤

1. ✅ 创建 digcore 仓库基础结构
2. ✅ 迁移底层模块（types、pkg）
3. ✅ 迁移核心框架（config、plugins、notify、mcp、diagnose、engine）
4. ✅ 修改 catpaw 的 import 路径
5. ✅ 测试验证功能

## 关键修改

### import 路径替换

```go
// 之前
import "github.com/cprobe/catpaw/types"
import "github.com/cprobe/catpaw/engine"

// 之后
import "github.com/cprobe/digcore/types"
import "github.com/cprobe/digcore/engine"
```

### 特殊处理

1. **notify/server.go**: 移回 catpaw（依赖 catpaw/server）
2. **plugins 接口**: 移到 digcore，具体实现保留在 catpaw
3. **config**: 完整迁移到 digcore，catpaw 直接使用

## 验证结果

- ✅ digcore 编译成功（`go mod tidy`）
- ✅ catpaw 编译成功（`./build.sh`）
- ✅ catpaw 功能正常（`./catpaw --version`）

## 下一步

1. 将 digcore 推送到 GitHub
2. 发布 digcore v0.1.0
3. 更新 catpaw 的 go.mod，使用正式版本号
4. 开始开发 k8spaw，依赖 digcore

## 版本管理

### digcore 版本策略

- v0.x.x: 开发版本
- v1.0.0: 稳定版本（API 冻结）
- v1.x.x: 向后兼容的新功能
- v2.0.0: 破坏性变更

### catpaw 和 k8spaw 的依赖

建议固定到 digcore 的小版本号，例如：

```go
require github.com/cprobe/digcore v0.2.0
```

## 注意事项

1. **API 稳定性**: digcore 的公开 API 必须保持稳定
2. **依赖方向**: 绝对不能让 digcore 依赖 catpaw 或 k8spaw
3. **版本同步**: catpaw 和 k8spaw 应使用相同的 digcore 版本
4. **测试覆盖**: 修改 digcore 后，必须同时测试 catpaw 和 k8spaw

## 联系方式

- GitHub: https://github.com/cprobe/digcore
- Issues: https://github.com/cprobe/digcore/issues
