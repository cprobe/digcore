# digcore

**AI-powered diagnostic framework for infrastructure monitoring**

digcore provides the core engine for building intelligent monitoring agents that can automatically diagnose issues using AI and diagnostic tools.

## Features

- Event-driven architecture with deduplication and alert management
- AI diagnosis engine with tool orchestration
- Multi-channel notification (Console, WebAPI, Flashduty, PagerDuty)
- MCP integration for external data sources
- Plugin system for extensibility

## Projects built with digcore

- [catpaw](https://github.com/cprobe/catpaw) - Host monitoring agent
- k8spaw - Kubernetes monitoring (commercial)

## Installation

```bash
go get github.com/cprobe/digcore@latest
```

## Quick Start

See [examples/](examples/) for sample implementations.

## License

Apache License 2.0
