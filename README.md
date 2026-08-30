# hk4e-Go-GameServer

一个基于 Go 的分布式 ARPG GameServer 项目，当前主要面向 **3.2 客户端**进行开发与完善。

本仓库基于原 `hk4e-go` 项目继续开发。当前目标不是逐函数复刻官方 GameServer 内部实现，而是让 **3.2 客户端应有的功能完整、稳定、行为正确**，并保留后续扩展新版本协议与玩法的能力。

## 项目目标

- 完整支持 3.2 客户端正常登录与游戏流程
- 完善 Player / Avatar / Item / Quest / Scene / World 等核心系统
- 完善场景 Group / Suite / Trigger / Lua 逻辑
- 完善怪物、机关、宝箱、NPC、交互等场景实体
- 完善 Combat / Ability / 元素反应 / 伤害等战斗逻辑
- 完善 Dungeon / Domain / Quest / Reward 等玩法
- 完善多人联机、同步与跨服相关逻辑
- 完善 Gacha / Shop / Mail / Friend 等外围系统
- 保持清晰的模块化、分布式架构，方便继续开发
- 为未来接入其他客户端版本预留协议与资源适配空间

## 设计原则

### 行为兼容优先

项目关注客户端最终看到和使用到的行为是否正确：

```text
3.2 Client
    ↓
Gate
    ↓
GameServer
    ↓
Game Domain
    ↓
Persistence / Resources
```

不要求服务端内部类、函数和官方实现逐一对应，只要协议、玩法、持久化和客户端流程保持正确即可。

### 在线数据驻内存

玩家登录后，主要运行状态由 GameServer 内存中的 Player 对象维护：

```text
Database
   ↓ 登录加载
Player in Memory
   ↓ 在线游戏
定时保存 / 下线保存
   ↓
Database
```

实时游戏逻辑不应依赖频繁同步数据库查询。

### 分布式架构

项目保留原 hk4e-go 的多服务设计：

```text
                 ┌──────────────┐
                 │     Node     │
                 └──────┬───────┘
                        │
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
    Dispatch           Gate             GS
                         │               │
                         └──────┬────────┘
                                ↓
                        NATS / Redis / DB
```

支持多个 Gate 和多个 GameServer 实例，为水平扩展和跨服功能提供基础。

## 主要目录

```text
cmd/        各服务启动入口
common/     公共组件、常量、MQ 等
dispatch/   登录与 Region 服务
gate/       KCP、Session、协议转发与客户端连接
gdconf/     游戏配置、JSON、TXT、Lua 等资源加载
gm/         游戏管理功能
gs/         GameServer 核心逻辑
multi/      多功能/跨服相关服务
node/       节点注册、服务发现与集群管理
pkg/        通用基础库
protocol/   Protobuf 与 Cmd 协议
```

## GameServer 主要模块

`gs/game` 中包含玩家与玩法的核心逻辑，例如：

```text
Player / Login
Avatar / Team
Item / Weapon / Reliquary
World / Scene
Monster / Gadget / NPC
Quest
Dungeon
Combat / Ability
Gacha
Shop
Mail
Friend
Multiplayer
Lua Group / Trigger
```

后续开发会继续围绕这些模块补齐 3.2 功能和行为。

## 游戏资源

仓库包含 `gdconf/game_data_config`，GameServer 会通过 `gdconf` 将资源加载并转换为运行时配置。

主要包括：

```text
gdconf/game_data_config/
├── json/
├── lua/
├── txt/
├── xml/
└── ext/
```

运行时游戏逻辑应优先通过统一配置接口访问资源，而不是直接耦合具体文件路径。

## 当前依赖

- Go >= 1.18
- Protobuf / protoc
- MongoDB
- Redis
- NATS Server
- Docker / Docker Compose（可选）

> 当前数据库层沿用原项目实现。后续可以进一步抽象 Persistence/Repository 层，在不影响游戏逻辑的情况下替换存储方案。

## 编译

首次安装开发工具：

```bash
make dev_tool
```

生成协议：

```bash
make gen_natsrpc
make gen_proto
make gen_client_proto
```

编译全部服务：

```bash
make build
```

## Docker

准备 Docker 配置与镜像：

```bash
make docker_config
make docker_build
```

然后进入：

```bash
cd docker
```

确认 MongoDB、Redis、NATS 以及各服务配置正确后启动：

```bash
docker compose up -d
```

## 开发方向

当前优先级：

```text
1. 3.2 客户端完整可玩
2. 登录 / 存档 / Scene / World 基础稳定
3. Quest / Group / Trigger / Lua 完整度
4. Combat / Ability / 联机同步
5. Dungeon / Gacha / Shop / Mail 等完整玩法
6. 性能、集群与跨服优化
7. 新版本协议与资源适配
```

对于行为不明确或实现存在差异的功能，应以实际 3.2 客户端表现和协议行为作为验证标准，再决定具体服务端实现方式。

## License

本项目沿用仓库中的 `LICENSE`。

## Disclaimer

本项目仅用于技术研究、学习 Go 游戏服务端架构、网络协议、分布式系统与服务端玩法实现。
