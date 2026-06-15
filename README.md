# Publish/Subscribe System in BCM4Java

A distributed publish/subscribe messaging system built on the BCM4Java component framework, featuring multi-JVM deployment, gossip-based broker federation, and a weather/windmill application domain.

## Features

### Core Messaging
- **Publish/Subscribe model** — publishers send messages to named channels, subscribers receive them based on topic and content
- **Message filtering** — subscribers can attach filters to selectively receive messages:
  - `MessageFilter` — basic filter on message properties (exact match, greater-than)
  - `WindmillFilter` — domain-specific filter for windmill application logic
  - `TimeFilter` — filters messages by timestamp (e.g., discard stale data)
  - `PropertiesFilter` — filters on message metadata properties
- **Three service classes** — FREE, STANDARD, and PREMIUM, each with differentiated thread pools for delivery

### Client Capabilities
- **Registration** — clients register with the broker to participate in messaging
- **Synchronous publish/subscribe** — standard publish and receive operations
- **Advanced reception** — `waitForNextMessage` (blocking) and `getNextMessage` (Future-based)
- **Asynchronous publish with notification** — `asyncPublishAndNotify` with callback via `AbnormalTerminationNotificationCI`
- **Plugin system** — registration, publication, and subscription plugins for extensible client behavior
- **Privileged clients** — PREMIUM clients can dynamically create and destroy channels

### Distributed Architecture
- **Gossip protocol** — brokers propagate messages across JVMs using an epidemic/gossip protocol with:
  - Configurable topology (ring, mesh, ring+shortcut)
  - Multi-hop propagation
  - Atomic deduplication
- **Multi-JVM deployment** — proven on up to 5 JVMs with 45 concurrent components
- **Timed scenarios** — coordinated distributed tests using shared accelerated clock (60x)

### Application Domain: Weather & Windmills
- **MeteoStation** — publishes wind data (speed, direction, GPS coordinates)
- **MeteoOffice** — publishes weather alerts (storm levels, alert colors)
- **Windmill** — subscribes to wind data and alerts, computes turbine orientation, performs safety stops when alerts affect its region

## Implementation

All source code is located in the **`CPS-Projet/`** directory.

```
CPS-Projet/
├── src/fr/sorbonne_u/
│   ├── messages/          # Message, MessageFilter, WindmillFilter
│   ├── meteo/             # WindData, MeteoAlert, Position, RectangularRegion
│   └── publication/
│       ├── Broker.java             # Message broker (router + channel manager)
│       ├── Client.java             # Client component base
│       ├── CVM_Audit1.java         # Basic pub/sub test
│       ├── CVM_Audit1_filtre.java  # Windmill filter test
│       ├── CVM_PluginTest.java     # Plugin system test
│       ├── CVM_FilterTest.java     # Advanced filters + dynamic channel
│       ├── CVM_ScenarioTest.java   # Large-scale timed scenario (45 clients)
│       ├── CVM_AsyncTest.java      # Async publish + advanced reception
│       ├── CVM_GossipTest.java     # Cross-broker gossip (1 JVM, 2 brokers)
│       ├── DistributedCVM.java     # 5-JVM distributed deployment
│       ├── DistributedScenarioCVM.java  # 3-JVM timed scenario
│       ├── components/      # MeteoStation, MeteoOffice, Windmill
│       ├── connectors/      # Publishing, Receiving, Registration connectors
│       ├── gossip/          # GossipMessage, GossipConnector, gossip ports
│       ├── implementations/ # Service implementations
│       ├── interfaces/      # Component interfaces
│       ├── plugins/         # Client plugins (registration, publication, subscription)
│       └── ports/           # Inbound/outbound ports
├── config.xml              # Deployment configuration
├── config-3jvm.xml         # 3-JVM deployment configuration
└── bin/                    # Compiled classes
```

## Running the System

### Prerequisites
- Java 8+
- BCM4Java framework (`BCM4Java-04032026.jar`)
- Apache Commons Math 3 (`commons-math3-3.6.1.jar`)

### Single-JVM Tests
Launch any CVM test class directly. Each CVM deploys a broker and a set of clients, then runs a scenario.

### Multi-JVM Deployment

1. Start the Global Registry:
   ```
   java -cp <classpath> fr.sorbonne_u.components.registry.GlobalRegistry CPS-Projet/config.xml
   ```

2. Start the CyclicBarrier:
   ```
   java -cp <classpath> fr.sorbonne_u.components.cvm.utils.DCVMCyclicBarrier CPS-Projet/config.xml
   ```

3. Launch each JVM (5-JVM example):
   ```
   java -cp <classpath> fr.sorbonne_u.publication.DistributedCVM jvm1 CPS-Projet/config.xml
   java -cp <classpath> fr.sorbonne_u.publication.DistributedCVM jvm2 CPS-Projet/config.xml
   java -cp <classpath> fr.sorbonne_u.publication.DistributedCVM jvm3 CPS-Projet/config.xml
   java -cp <classpath> fr.sorbonne_u.publication.DistributedCVM jvm4 CPS-Projet/config.xml
   java -cp <classpath> fr.sorbonne_u.publication.DistributedCVM jvm5 CPS-Projet/config.xml
   ```

## Test Coverage

| Test | Scope | Clients | Key Feature |
|------|-------|---------|-------------|
| `CVM_Audit1` | Single JVM | 3 | Basic pub/sub |
| `CVM_Audit1_filtre` | Single JVM | 5 | Domain filtering (WindmillFilter) |
| `CVM_PluginTest` | Single JVM | 6 | Plugins + 3 service classes |
| `CVM_FilterTest` | Single JVM | 8 | TimeFilter, PropertiesFilter, dynamic channel |
| `CVM_ScenarioTest` | Single JVM | 47 | Timed scenario, 45 clients, all features |
| `CVM_AsyncTest` | Single JVM | 16 | Async publish, advanced reception, thread pools |
| `CVM_GossipTest` | Single JVM | 4 | Cross-broker gossip protocol |
| `DistributedCVM` | 5 JVMs | 20 | Multi-hop gossip, 5 brokers, deduplication |
| `DistributedScenarioCVM` | 3 JVMs | 10 | Timed distributed scenario, dynamic channels |

---

# 基于 BCM4Java 的发布/订阅系统

基于 BCM4Java 组件框架构建的分布式发布/订阅消息系统，支持多 JVM 部署、基于 Gossip 协议的 Broker 联邦，以及天气/风车应用领域。

## 功能特性

### 核心消息机制
- **发布/订阅模型** — 发布者向命名通道发送消息，订阅者根据主题和内容接收消息
- **消息过滤** — 订阅者可以附加过滤器来选择性接收消息：
  - `MessageFilter` — 基于消息属性的基础过滤器（精确匹配、大于比较）
  - `WindmillFilter` — 面向风车应用逻辑的领域专用过滤器
  - `TimeFilter` — 按时间戳过滤消息（如丢弃过期数据）
  - `PropertiesFilter` — 基于消息元数据属性进行过滤
- **三种服务等级** — FREE、STANDARD 和 PREMIUM，每种等级使用不同的线程池进行消息投递

### 客户端能力
- **注册** — 客户端向 Broker 注册以参与消息传递
- **同步发布/订阅** — 标准的发布和接收操作
- **高级接收模式** — `waitForNextMessage`（阻塞式）和 `getNextMessage`（基于 Future）
- **异步发布与通知** — `asyncPublishAndNotify` 通过 `AbnormalTerminationNotificationCI` 回调
- **插件系统** — 注册、发布和订阅插件，支持可扩展的客户端行为
- **特权客户端** — PREMIUM 客户端可以动态创建和销毁通道

### 分布式架构
- **Gossip 协议** — Broker 通过流行病/八卦协议跨 JVM 传播消息：
  - 可配置拓扑（环形、全互联、环形+捷径）
  - 多跳传播
  - 原子去重
- **多 JVM 部署** — 已验证支持最多 5 个 JVM、45 个并发组件
- **定时场景** — 使用共享加速时钟（60 倍）协调分布式测试

### 应用领域：天气与风车
- **MeteoStation**（气象站）— 发布风数据（速度、方向、GPS 坐标）
- **MeteoOffice**（气象局）— 发布天气警报（风暴等级、警报颜色）
- **Windmill**（风车）— 订阅风数据和警报，计算涡轮朝向，在警报影响其区域时执行安全停机

## 实现

所有源代码位于 **`CPS-Projet/`** 目录。

```
CPS-Projet/
├── src/fr/sorbonne_u/
│   ├── messages/          # Message、MessageFilter、WindmillFilter
│   ├── meteo/             # WindData、MeteoAlert、Position、RectangularRegion
│   └── publication/
│       ├── Broker.java             # 消息 Broker（路由 + 通道管理）
│       ├── Client.java             # 客户端组件基类
│       ├── CVM_Audit1.java         # 基础发布/订阅测试
│       ├── CVM_Audit1_filtre.java  # 风车过滤器测试
│       ├── CVM_PluginTest.java     # 插件系统测试
│       ├── CVM_FilterTest.java     # 高级过滤器 + 动态通道
│       ├── CVM_ScenarioTest.java   # 大规模定时场景（45 个客户端）
│       ├── CVM_AsyncTest.java      # 异步发布 + 高级接收
│       ├── CVM_GossipTest.java     # 跨 Broker Gossip（1 JVM，2 Broker）
│       ├── DistributedCVM.java     # 5 JVM 分布式部署
│       ├── DistributedScenarioCVM.java  # 3 JVM 定时场景
│       ├── components/      # MeteoStation、MeteoOffice、Windmill
│       ├── connectors/      # 发布、接收、注册连接器
│       ├── gossip/          # GossipMessage、GossipConnector、Gossip 端口
│       ├── implementations/ # 服务实现
│       ├── interfaces/      # 组件接口
│       ├── plugins/         # 客户端插件（注册、发布、订阅）
│       └── ports/           # 入站/出站端口
├── config.xml              # 部署配置
├── config-3jvm.xml         # 3 JVM 部署配置
└── bin/                    # 编译后的类文件
```

## 系统运行

### 环境要求
- Java 8+
- BCM4Java 框架（`BCM4Java-04032026.jar`）
- Apache Commons Math 3（`commons-math3-3.6.1.jar`）

### 单 JVM 测试
直接启动任意 CVM 测试类。每个 CVM 部署一个 Broker 和一组客户端，然后运行测试场景。

### 多 JVM 部署

1. 启动全局注册中心：
   ```
   java -cp <classpath> fr.sorbonne_u.components.registry.GlobalRegistry CPS-Projet/config.xml
   ```

2. 启动 CyclicBarrier：
   ```
   java -cp <classpath> fr.sorbonne_u.components.cvm.utils.DCVMCyclicBarrier CPS-Projet/config.xml
   ```

3. 启动各个 JVM（以 5 JVM 为例）：
   ```
   java -cp <classpath> fr.sorbonne_u.publication.DistributedCVM jvm1 CPS-Projet/config.xml
   java -cp <classpath> fr.sorbonne_u.publication.DistributedCVM jvm2 CPS-Projet/config.xml
   java -cp <classpath> fr.sorbonne_u.publication.DistributedCVM jvm3 CPS-Projet/config.xml
   java -cp <classpath> fr.sorbonne_u.publication.DistributedCVM jvm4 CPS-Projet/config.xml
   java -cp <classpath> fr.sorbonne_u.publication.DistributedCVM jvm5 CPS-Projet/config.xml
   ```

## 测试覆盖

| 测试类 | 范围 | 客户端数 | 核心功能 |
|--------|------|----------|----------|
| `CVM_Audit1` | 单 JVM | 3 | 基础发布/订阅 |
| `CVM_Audit1_filtre` | 单 JVM | 5 | 领域过滤（WindmillFilter） |
| `CVM_PluginTest` | 单 JVM | 6 | 插件 + 三种服务等级 |
| `CVM_FilterTest` | 单 JVM | 8 | TimeFilter、PropertiesFilter、动态通道 |
| `CVM_ScenarioTest` | 单 JVM | 47 | 定时场景，45 客户端，全功能覆盖 |
| `CVM_AsyncTest` | 单 JVM | 16 | 异步发布、高级接收、线程池 |
| `CVM_GossipTest` | 单 JVM | 4 | 跨 Broker Gossip 协议 |
| `DistributedCVM` | 5 JVM | 20 | 多跳 Gossip，5 Broker，消息去重 |
| `DistributedScenarioCVM` | 3 JVM | 10 | 定时分布式场景，动态通道 |
