# 模拟API接口

<cite>
**本文档引用的文件**
- [backend/app/api/simulation.py](file://backend/app/api/simulation.py)
- [backend/app/services/simulation_manager.py](file://backend/app/services/simulation_manager.py)
- [backend/app/services/simulation_runner.py](file://backend/app/services/simulation_runner.py)
- [backend/app/models/project.py](file://backend/app/models/project.py)
- [backend/app/models/task.py](file://backend/app/models/task.py)
- [backend/app/services/zep_entity_reader.py](file://backend/app/services/zep_entity_reader.py)
- [backend/app/services/oasis_profile_generator.py](file://backend/app/services/oasis_profile_generator.py)
- [backend/app/config.py](file://backend/app/config.py)
- [frontend/src/api/simulation.js](file://frontend/src/api/simulation.js)
- [frontend/src/api/report.js](file://frontend/src/api/report.js)
- [frontend/src/api/graph.js](file://frontend/src/api/graph.js)
</cite>

## 目录
1. [简介](#简介)
2. [项目结构](#项目结构)
3. [核心组件](#核心组件)
4. [架构概览](#架构概览)
5. [详细组件分析](#详细组件分析)
6. [依赖关系分析](#依赖关系分析)
7. [性能考虑](#性能考虑)
8. [故障排除指南](#故障排除指南)
9. [结论](#结论)

## 简介

MiroFish是一个基于OASIS模拟平台的双平台社交媒体模拟系统，支持Twitter和Reddit两个平台的并行模拟运行。该系统通过智能LLM生成Agent Profile和模拟配置，实现了从图谱实体到真实社交模拟的完整自动化流程。

系统的核心目标是为用户提供一个完整的模拟生态系统，包括：
- 图谱构建与实体读取
- Agent Profile自动生成
- 模拟配置智能生成
- 双平台并行模拟运行
- 实时状态监控和结果获取
- 报告生成和分析

## 项目结构

```mermaid
graph TB
subgraph "前端层"
FE1[Vue.js 前端应用]
API1[前端API封装]
end
subgraph "后端层"
BP1[Flask Blueprints]
API2[API路由层]
SVC[服务层]
MDL[模型层]
end
subgraph "数据层"
ZEP[Zep图谱]
FS[文件系统]
DB[(内存/文件存储)]
end
FE1 --> API1
API1 --> BP1
BP1 --> API2
API2 --> SVC
API2 --> MDL
SVC --> ZEP
SVC --> FS
MDL --> DB
```

**图表来源**
- [backend/app/api/__init__.py](file://backend/app/api/__init__.py#L1-L15)
- [backend/app/api/simulation.py](file://backend/app/api/simulation.py#L1-L50)

**章节来源**
- [backend/app/api/__init__.py](file://backend/app/api/__init__.py#L1-L15)
- [backend/app/config.py](file://backend/app/config.py#L1-L76)

## 核心组件

### API路由模块

系统采用Flask蓝图架构，将不同功能模块分离为独立的API路由：

- **graph_bp**: 图谱相关API（本体生成、图谱构建、任务管理）
- **simulation_bp**: 模拟相关API（实体读取、模拟创建、状态管理）
- **report_bp**: 报告相关API（报告生成、获取、对话）

### 服务层架构

```mermaid
classDiagram
class SimulationManager {
+create_simulation()
+prepare_simulation()
+get_simulation()
+list_simulations()
+get_profiles()
+get_simulation_config()
+get_run_instructions()
}
class SimulationRunner {
+start_simulation()
+stop_simulation()
+get_run_state()
+_monitor_simulation()
}
class ZepEntityReader {
+filter_defined_entities()
+get_entity_with_context()
+get_entities_by_type()
}
class OasisProfileGenerator {
+generate_profiles_from_entities()
+save_profiles()
}
class TaskManager {
+create_task()
+update_task()
+get_task()
+complete_task()
}
SimulationManager --> ZepEntityReader : "使用"
SimulationManager --> OasisProfileGenerator : "使用"
SimulationManager --> TaskManager : "使用"
SimulationRunner --> TaskManager : "使用"
```

**图表来源**
- [backend/app/services/simulation_manager.py](file://backend/app/services/simulation_manager.py#L114-L529)
- [backend/app/services/simulation_runner.py](file://backend/app/services/simulation_runner.py#L195-L800)
- [backend/app/services/zep_entity_reader.py](file://backend/app/services/zep_entity_reader.py#L70-L200)
- [backend/app/services/oasis_profile_generator.py](file://backend/app/services/oasis_profile_generator.py#L142-L200)

**章节来源**
- [backend/app/services/simulation_manager.py](file://backend/app/services/simulation_manager.py#L1-L529)
- [backend/app/services/simulation_runner.py](file://backend/app/services/simulation_runner.py#L1-L800)

## 架构概览

### 系统架构图

```mermaid
graph TB
subgraph "用户界面层"
UI[Web界面]
CLI[命令行工具]
end
subgraph "API网关层"
AUTH[认证中间件]
ROUTER[路由分发]
end
subgraph "业务逻辑层"
SIM[模拟管理器]
RUN[模拟运行器]
REP[报告生成器]
ENT[实体读取器]
PROF[Profile生成器]
end
subgraph "数据访问层"
ZEP[Zep Cloud API]
FS[本地文件系统]
MEM[内存缓存]
end
UI --> AUTH
CLI --> AUTH
AUTH --> ROUTER
ROUTER --> SIM
ROUTER --> RUN
ROUTER --> REP
SIM --> ENT
SIM --> PROF
RUN --> FS
ENT --> ZEP
PROF --> ZEP
SIM --> MEM
RUN --> MEM
```

**图表来源**
- [backend/app/api/simulation.py](file://backend/app/api/simulation.py#L1-L800)
- [backend/app/api/report.py](file://backend/app/api/report.py#L1-L800)

### 数据流架构

```mermaid
sequenceDiagram
participant Client as "客户端"
participant API as "API层"
participant Manager as "模拟管理器"
participant Reader as "实体读取器"
participant Generator as "Profile生成器"
participant ConfigGen as "配置生成器"
participant Runner as "模拟运行器"
Client->>API : 创建模拟请求
API->>Manager : create_simulation()
Manager->>Manager : 保存初始状态
Client->>API : 准备模拟请求
API->>Manager : prepare_simulation()
Manager->>Reader : filter_defined_entities()
Reader-->>Manager : 过滤后的实体
Manager->>Generator : generate_profiles_from_entities()
Generator-->>Manager : Agent Profile列表
Manager->>ConfigGen : generate_config()
ConfigGen-->>Manager : 模拟配置
Manager-->>API : 准备完成状态
Client->>API : 启动模拟请求
API->>Runner : start_simulation()
Runner->>Runner : 监控模拟进程
Runner-->>API : 实时状态更新
```

**图表来源**
- [backend/app/api/simulation.py](file://backend/app/api/simulation.py#L164-L237)
- [backend/app/services/simulation_manager.py](file://backend/app/services/simulation_manager.py#L229-L457)
- [backend/app/services/simulation_runner.py](file://backend/app/services/simulation_runner.py#L312-L475)

## 详细组件分析

### 实体读取接口

实体读取接口提供从Zep图谱中获取和过滤实体的功能：

#### 接口规范

**获取所有实体**
- **方法**: GET
- **路径**: `/api/simulation/entities/{graph_id}`
- **查询参数**:
  - `entity_types`: 逗号分隔的实体类型列表（可选）
  - `enrich`: 是否获取相关边信息（默认true）

**获取单个实体详情**
- **方法**: GET
- **路径**: `/api/simulation/entities/{graph_id}/{entity_uuid}`

**按类型获取实体**
- **方法**: GET
- **路径**: `/api/simulation/entities/{graph_id}/by-type/{entity_type}`
- **查询参数**:
  - `enrich`: 是否获取相关边信息（默认true）

#### 数据结构

```mermaid
erDiagram
ENTITY_NODE {
string uuid PK
string name
array labels
string summary
object attributes
array related_edges
array related_nodes
}
FILTERED_ENTITIES {
array entities
set entity_types
int total_count
int filtered_count
}
ENTITY_NODE ||--o{ FILTERED_ENTITIES : "包含"
```

**图表来源**
- [backend/app/services/zep_entity_reader.py](file://backend/app/services/zep_entity_reader.py#L21-L68)

**章节来源**
- [backend/app/api/simulation.py](file://backend/app/api/simulation.py#L47-L160)
- [backend/app/services/zep_entity_reader.py](file://backend/app/services/zep_entity_reader.py#L70-L200)

### 模拟创建接口

模拟创建接口负责初始化模拟状态和基础配置。

#### 接口规范

**创建模拟**
- **方法**: POST
- **路径**: `/api/simulation/create`
- **请求体**:
  ```json
  {
    "project_id": "proj_xxxx",
    "graph_id": "mirofish_xxxx",
    "enable_twitter": true,
    "enable_reddit": true
  }
  ```
- **响应体**:
  ```json
  {
    "success": true,
    "data": {
      "simulation_id": "sim_xxxx",
      "project_id": "proj_xxxx",
      "graph_id": "mirofish_xxxx",
      "status": "created",
      "enable_twitter": true,
      "enable_reddit": true,
      "created_at": "2025-12-01T10:00:00"
    }
  }
  ```

#### 状态管理

```mermaid
stateDiagram-v2
[*] --> Created
Created --> Preparing : "开始准备"
Preparing --> Ready : "准备完成"
Preparing --> Failed : "准备失败"
Ready --> Running : "启动运行"
Running --> Completed : "模拟完成"
Running --> Stopped : "手动停止"
Running --> Failed : "运行失败"
Stopped --> Running : "重新启动"
Failed --> Preparing : "重新准备"
```

**图表来源**
- [backend/app/services/simulation_manager.py](file://backend/app/services/simulation_manager.py#L24-L34)

**章节来源**
- [backend/app/api/simulation.py](file://backend/app/api/simulation.py#L164-L237)
- [backend/app/services/simulation_manager.py](file://backend/app/services/simulation_manager.py#L193-L228)

### 模拟准备接口

模拟准备接口是系统的核心功能，负责自动化生成Agent Profile和模拟配置。

#### 接口规范

**准备模拟**
- **方法**: POST
- **路径**: `/api/simulation/prepare`
- **请求体**:
  ```json
  {
    "simulation_id": "sim_xxxx",
    "entity_types": ["Student", "PublicFigure"],
    "use_llm_for_profiles": true,
    "parallel_profile_count": 5,
    "force_regenerate": false
  }
  ```

**查询准备进度**
- **方法**: POST
- **路径**: `/api/simulation/prepare/status`
- **请求体**:
  ```json
  {
    "task_id": "task_xxxx",
    "simulation_id": "sim_xxxx"
  }
  ```

#### 准备流程

```mermaid
flowchart TD
Start([开始准备]) --> CheckExisting["检查现有状态"]
CheckExisting --> ForceRegenerate{"强制重新生成?"}
ForceRegenerate --> |是| StartPreparation["开始准备流程"]
ForceRegenerate --> |否| AlreadyReady{"已准备完成?"}
AlreadyReady --> |是| ReturnReady["返回已准备状态"]
AlreadyReady --> |否| StartPreparation
StartPreparation --> ReadEntities["读取图谱实体"]
ReadEntities --> GenerateProfiles["生成Agent Profile"]
GenerateProfiles --> GenerateConfig["生成模拟配置"]
GenerateConfig --> CopyScripts["复制脚本文件"]
CopyScripts --> UpdateStatus["更新状态为Ready"]
UpdateStatus --> End([准备完成])
ReturnReady --> End
```

**图表来源**
- [backend/app/api/simulation.py](file://backend/app/api/simulation.py#L358-L635)

**章节来源**
- [backend/app/api/simulation.py](file://backend/app/api/simulation.py#L358-L748)
- [backend/app/services/simulation_manager.py](file://backend/app/services/simulation_manager.py#L229-L457)

### 模拟运行接口

模拟运行接口提供双平台并行模拟运行能力。

#### 接口规范

**启动模拟**
- **方法**: POST
- **路径**: `/api/simulation/start`
- **请求体**:
  ```json
  {
    "simulation_id": "sim_xxxx",
    "platform": "parallel",
    "max_rounds": 100,
    "enable_graph_memory_update": false,
    "graph_id": "zep_graph_xxxx"
  }
  ```

**停止模拟**
- **方法**: POST
- **路径**: `/api/simulation/stop`
- **请求体**:
  ```json
  {
    "simulation_id": "sim_xxxx"
  }
  ```

#### 运行状态监控

```mermaid
classDiagram
class SimulationRunState {
+string simulation_id
+RunnerStatus runner_status
+int current_round
+int total_rounds
+int simulated_hours
+bool twitter_running
+bool reddit_running
+int twitter_actions_count
+int reddit_actions_count
+AgentAction[] recent_actions
+add_action()
+to_dict()
}
class AgentAction {
+int round_num
+string timestamp
+string platform
+int agent_id
+string agent_name
+string action_type
+Dict~Any~ action_args
+string result
+bool success
+to_dict()
}
class RoundSummary {
+int round_num
+string start_time
+string end_time
+int simulated_hour
+int twitter_actions
+int reddit_actions
+int[] active_agents
+AgentAction[] actions
+to_dict()
}
SimulationRunState --> AgentAction : "包含"
SimulationRunState --> RoundSummary : "包含"
```

**图表来源**
- [backend/app/services/simulation_runner.py](file://backend/app/services/simulation_runner.py#L47-L193)

**章节来源**
- [backend/app/api/simulation.py](file://backend/app/api/simulation.py#L750-L800)
- [backend/app/services/simulation_runner.py](file://backend/app/services/simulation_runner.py#L195-L800)

### 报告生成接口

报告生成接口提供模拟结果的分析和报告生成功能。

#### 接口规范

**生成报告**
- **方法**: POST
- **路径**: `/api/report/generate`
- **请求体**:
  ```json
  {
    "simulation_id": "sim_xxxx",
    "force_regenerate": false
  }
  ```

**获取报告状态**
- **方法**: POST
- **路径**: `/api/report/generate/status`
- **请求体**:
  ```json
  {
    "task_id": "task_xxxx",
    "simulation_id": "sim_xxxx"
  }
  ```

#### 报告进度监控

```mermaid
sequenceDiagram
participant Client as "客户端"
participant API as "报告API"
participant Agent as "Report Agent"
participant Task as "任务管理器"
participant Progress as "进度监控"
Client->>API : 生成报告请求
API->>Task : 创建任务
API->>Agent : 启动报告生成
Agent->>Progress : 更新生成进度
Progress-->>API : 进度状态
API-->>Client : 进度更新
Agent->>Task : 完成任务
Task-->>API : 任务完成
API-->>Client : 报告完成
```

**图表来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L24-L196)

**章节来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L1-L800)

## 依赖关系分析

### 外部依赖

```mermaid
graph TB
subgraph "外部服务"
ZEP[Zep Cloud API]
LLM[LLM服务]
OASIS[OASIS模拟平台]
end
subgraph "内部模块"
API[API层]
SVC[服务层]
MDL[模型层]
CFG[配置管理]
end
API --> SVC
SVC --> ZEP
SVC --> LLM
SVC --> OASIS
SVC --> MDL
MDL --> CFG
```

**图表来源**
- [backend/app/config.py](file://backend/app/config.py#L20-L76)

### 内部模块依赖

```mermaid
graph LR
API1[API层] --> SVC1[服务层]
API2[模拟API] --> SVC1
API3[报告API] --> SVC1
API4[图谱API] --> SVC1
SVC1 --> MDL1[模型层]
SVC1 --> CFG1[配置管理]
MDL1 --> ZEP1[Zep实体读取器]
MDL1 --> PROF1[Profile生成器]
MDL1 --> TASK1[任务管理器]
CFG1 --> ENV[环境配置]
```

**图表来源**
- [backend/app/api/__init__.py](file://backend/app/api/__init__.py#L1-L15)

**章节来源**
- [backend/app/config.py](file://backend/app/config.py#L1-L76)

## 性能考虑

### 并行处理优化

系统采用多线程和异步处理来提升性能：

1. **并行Profile生成**: 支持配置并行数量，默认5个
2. **异步任务处理**: 使用线程池处理长时间运行的任务
3. **增量日志读取**: 实时监控模拟进程，避免全量读取
4. **内存缓存**: 缓存模拟状态减少磁盘I/O

### 资源管理

```mermaid
flowchart TD
Resource[资源管理] --> Memory[内存管理]
Resource --> Disk[磁盘I/O优化]
Resource --> Network[网络请求优化]
Memory --> Cache[缓存策略]
Memory --> Pool[连接池]
Disk --> AsyncIO[异步I/O]
Disk --> Buffer[缓冲区]
Network --> Retry[重试机制]
Network --> Timeout[超时控制]
```

### 性能监控指标

- **响应时间**: API请求平均响应时间
- **并发处理**: 同时处理的任务数量
- **内存使用**: 模拟状态和日志的内存占用
- **磁盘I/O**: Profile文件和日志文件的读写性能

## 故障排除指南

### 常见错误及解决方案

#### 配置错误

**错误**: ZEP_API_KEY未配置
**解决方案**: 检查.env文件中的ZEP_API_KEY配置

**错误**: LLM_API_KEY未配置  
**解决方案**: 配置LLM服务的API密钥和基础URL

#### 任务执行错误

**错误**: 任务超时
**解决方案**: 检查任务队列状态，增加超时时间配置

**错误**: 任务失败
**解决方案**: 查看任务错误日志，检查依赖服务状态

#### 模拟运行错误

**错误**: 模拟无法启动
**解决方案**: 检查模拟配置文件，确认脚本路径正确

**错误**: 进程无法终止
**解决方案**: 使用进程ID手动终止，检查僵尸进程

### 调试工具

```mermaid
graph TB
Debug[调试工具] --> Logs[日志查看]
Debug --> Metrics[性能监控]
Debug --> Tracing[请求追踪]
Logs --> API[API日志]
Logs --> System[系统日志]
Logs --> Error[错误日志]
Metrics --> Response[响应时间]
Metrics --> Throughput[吞吐量]
Metrics --> ErrorRate[错误率]
Tracing --> Request[请求追踪]
Tracing --> Trace[链路追踪]
```

**章节来源**
- [backend/app/config.py](file://backend/app/config.py#L66-L75)

## 结论

MiroFish模拟API系统提供了一个完整的双平台社交媒体模拟解决方案。通过模块化的架构设计和完善的错误处理机制，系统能够稳定地支持从图谱构建到模拟运行的完整流程。

### 系统优势

1. **自动化程度高**: 从实体读取到配置生成完全自动化
2. **扩展性强**: 支持新的平台和功能模块扩展
3. **监控完善**: 提供实时状态监控和日志追踪
4. **容错能力强**: 具备完善的错误处理和恢复机制

### 未来发展方向

1. **性能优化**: 进一步提升大规模模拟的处理能力
2. **功能扩展**: 支持更多社交媒体平台和模拟场景
3. **可视化增强**: 提供更丰富的数据可视化和分析功能
4. **部署优化**: 支持容器化和云原生部署方案

该系统为研究人员和开发者提供了一个强大而灵活的模拟平台，能够满足各种社交媒体行为研究和分析需求。