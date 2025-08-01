# NanoHUB 新手入门

NanoHUB 是一个将 **NanoMDM**、**NanoCMD** 和 **KMFDDM** 融合在一起的 Apple MDM 服务器实现。项目既可以作为 Go 语言库使用，也提供了一个示例命令行服务器。在阅读本仓库的代码或尝试运行服务之前，可以先通过本文档快速了解项目目标、工作原理以及目录结构。

## 项目概览

- **目的**：提供统一的 MDM 服务器，同时支持传统的命令驱动模式和 Apple 的 Declarative Management 功能。
- **组成**：核心库位于 `nanohub` 目录，命令行程序位于 `cmd/nanohub`。其它目录提供了命令工作流服务、Declarative Management 适配器等功能模块。
- **使用方式**：可以在自己的 Go 项目中引入 `github.com/micromdm/nanohub`，也可以通过编译 `cmd/nanohub` 直接运行参考实现。

## 工作原理

1. **启动服务器**：`cmd/nanohub/nanohub.go` 中的 `main` 函数解析启动参数，配置存储后创建 `nanohub` 实例。
2. **核心服务**：`nanohub` 包负责构建各种 MDM 服务，包括证书验证、命令下发、状态上报等。可通过不同的选项启用命令工作流 (NanoCMD) 和 Declarative Management。
3. **命令工作流**：若配置命令存储，会在 `cmdservice` 包中创建适配器，将 NanoCMD 的工作流事件转换为 MDM 命令和结果处理逻辑。
4. **Declarative Management**：`ddmadapter` 包负责将 KMFDDM 的声明式管理消息转换为 NanoMDM 可处理的格式，并在需要时存储设备的状态报告。
5. **推送与排队**：`enqueue` 包提供通用的推送和命令排队器，确保设备收到新的命令或配置更新。
6. **接口与 API**：参考服务器在 `/api/v1/` 下提供 REST API，可通过 `-api-key` 参数启用基本认证。更多选项和使用细节见 `docs/operations-guide.md`。

## 目录结构

项目顶层主要目录如下：

```
cmd/              参考命令行程序（main 函数在 cmd/nanohub）
cmdservice/       将 NanoCMD 工作流适配为 MDM 服务
ddmadapter/       Declarative Management 适配器
enqueue/          推送与命令队列封装
nanohub/          核心库，实现 MDM 服务器逻辑
docs/             文档（操作指南及本入门文档）
.github/          CI/CD 配置
```

其它文件说明：

- `go.mod`、`go.sum`：Go 依赖与版本管理。
- `Dockerfile`：提供容器化运行配置。
- `Makefile`：常用构建与测试命令（执行 `make test` 可运行所有单元测试）。

## 文件流程简介

以下给出从入口到核心流程的一个大致路线，便于快速定位代码：

1. **入口程序**：`cmd/nanohub/nanohub.go` 解析命令行参数并创建 `nanohub.New()`。
2. **配置与初始化**：`nanohub/config.go` 读取选项，决定是否启用推送、工作流等组件。
3. **核心服务**：`nanohub/nanohub.go` 根据配置组装各子服务，并暴露 HTTP Handler（如 `/mdm`、`/checkin`、`/api/v1/`）。
4. **工作流与适配**：若启用命令工作流，`cmdservice` 会从 NanoCMD 接收事件并转为 MDM 命令；Declarative Management 消息则由 `ddmadapter` 处理并写入存储。
5. **推送与排队**：`enqueue` 提供统一的推送接口，使得命令或配置更新能够触发 APNs 推送。

## 进一步阅读

- [操作指南](operations-guide.md)：介绍所有启动参数以及各功能模块的详细用法。
- [GoDoc](https://pkg.go.dev/github.com/micromdm/nanohub)：若想在其他项目中引用，可查看包文档了解具体 API。

通过以上内容，你应该能对 NanoHUB 的整体架构和代码布局有一个初步的认识，之后可根据需要深入查看对应的包实现。

