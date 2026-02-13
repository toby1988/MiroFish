# 报告API接口

<cite>
**本文档引用的文件**
- [backend/app/api/report.py](file://backend/app/api/report.py)
- [backend/app/services/report_agent.py](file://backend/app/services/report_agent.py)
- [backend/app/services/zep_tools.py](file://backend/app/services/zep_tools.py)
- [backend/app/models/project.py](file://backend/app/models/project.py)
- [backend/app/models/task.py](file://backend/app/models/task.py)
- [backend/app/utils/logger.py](file://backend/app/utils/logger.py)
- [frontend/src/api/report.js](file://frontend/src/api/report.js)
- [README.md](file://README.md)
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

MiroFish报告API接口是基于多智能体技术的下一代AI预测引擎的重要组成部分。该系统通过ReACT（Reasoning + Acting）模式的智能Agent，结合Zep图谱检索工具，为用户提供完整的模拟分析报告生成功能。

系统支持四种核心报告API接口：
- **报告生成接口**：异步生成模拟分析报告
- **报告获取接口**：查询和获取报告详情
- **报告下载接口**：下载Markdown格式的报告
- **深度交互采访接口**：与模拟Agent进行深度对话采访

## 项目结构

MiroFish项目采用前后端分离架构，后端使用Flask框架，前端使用Vue.js技术栈。报告API接口位于后端的`backend/app/api/report.py`文件中，核心业务逻辑由`ReportAgent`类实现。

```mermaid
graph TB
subgraph "前端层"
FE[Vue.js前端]
API[前端API封装]
end
subgraph "后端层"
FLASK[Flask应用]
ROUTES[报告API路由]
SERVICES[服务层]
MODELS[模型层]
UTILS[工具层]
end
subgraph "数据层"
ZEP[Zep图谱服务]
STORAGE[文件存储]
LOGS[日志系统]
end
FE --> API
API --> FLASK
FLASK --> ROUTES
ROUTES --> SERVICES
SERVICES --> MODELS
SERVICES --> UTILS
SERVICES --> ZEP
MODELS --> STORAGE
UTILS --> LOGS
```

**图表来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L1-L1016)
- [backend/app/services/report_agent.py](file://backend/app/services/report_agent.py#L1-L2420)

**章节来源**
- [README.md](file://README.md#L81-L88)

## 核心组件

### 报告API路由层

报告API路由层负责处理HTTP请求和响应，提供RESTful接口。主要包含以下功能模块：

- **异步报告生成**：处理报告生成任务的创建和状态查询
- **报告查询管理**：提供报告详情查询、列表获取等功能
- **文件下载服务**：支持报告文件的下载和流式传输
- **智能对话接口**：提供与ReportAgent的深度交互能力
- **日志监控服务**：实时获取报告生成过程的详细日志

### ReportAgent智能代理

ReportAgent是系统的核心智能体，采用ReACT模式进行推理和行动：

- **ReACT推理模式**：思考-行动-观察的循环机制
- **多工具集集成**：整合三种核心检索工具和深度采访功能
- **结构化报告生成**：按照预定义的大纲结构生成高质量报告
- **实时进度监控**：提供详细的生成进度和状态反馈

### Zep工具服务

Zep工具服务提供强大的图谱检索能力：

- **InsightForge深度洞察**：多维度的智能检索和分析
- **Panorama广度搜索**：获取事件全貌和演变过程
- **QuickSearch快速检索**：高效的精确信息查询
- **InterviewAgents深度采访**：真实Agent的多视角采访

**章节来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L24-L196)
- [backend/app/services/report_agent.py](file://backend/app/services/report_agent.py#L469-L530)
- [backend/app/services/zep_tools.py](file://backend/app/services/zep_tools.py#L377-L400)

## 架构概览

系统采用分层架构设计，确保各层职责明确、耦合度低：

```mermaid
sequenceDiagram
participant Client as 客户端
participant API as 报告API
participant Agent as ReportAgent
participant Tools as Zep工具
participant Storage as 存储系统
Client->>API : POST /api/report/generate
API->>API : 验证输入参数
API->>Agent : 创建ReportAgent实例
Agent->>Tools : 检索图谱信息
Tools-->>Agent : 返回检索结果
Agent->>Agent : 规划报告大纲
Agent->>Tools : 深度检索各章节内容
Tools-->>Agent : 返回详细信息
Agent->>Storage : 保存报告文件
API-->>Client : 返回任务ID和状态
Client->>API : GET /api/report/generate/status
API->>Storage : 查询任务状态
API-->>Client : 返回进度信息
```

**图表来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L24-L196)
- [backend/app/services/report_agent.py](file://backend/app/services/report_agent.py#L1358-L1591)

### 数据流架构

```mermaid
flowchart TD
Start([用户请求]) --> Validate[参数验证]
Validate --> CreateTask[创建异步任务]
CreateTask --> InitAgent[初始化ReportAgent]
InitAgent --> PlanOutline[规划报告大纲]
PlanOutline --> GenerateSections[逐章节生成]
GenerateSections --> ToolCalls[工具调用]
ToolCalls --> ReActLoop[ReACT循环]
ReActLoop --> CheckComplete{章节完成?}
CheckComplete --> |否| ToolCalls
CheckComplete --> |是| SaveSection[保存章节]
SaveSection --> UpdateProgress[更新进度]
UpdateProgress --> NextSection{还有章节?}
NextSection --> |是| GenerateSections
NextSection --> |否| AssembleReport[组装完整报告]
AssembleReport --> SaveReport[保存最终报告]
SaveReport --> Complete[任务完成]
```

**图表来源**
- [backend/app/services/report_agent.py](file://backend/app/services/report_agent.py#L1456-L1564)

## 详细组件分析

### 报告生成接口

报告生成接口提供异步任务处理能力，支持强制重新生成和进度查询功能。

#### 接口规范

**POST /api/report/generate**
- **请求参数**：
  - `simulation_id` (必填)：模拟ID
  - `force_regenerate` (可选)：强制重新生成标志

- **响应结构**：
  - `success`：布尔值，请求是否成功
  - `data`：包含任务信息的对象
  - `task_id`：任务唯一标识符
  - `report_id`：报告唯一标识符
  - `status`：任务状态
  - `message`：状态描述

#### 生成流程

```mermaid
sequenceDiagram
participant Client as 客户端
participant API as 生成接口
participant Manager as 管理器
participant Agent as ReportAgent
participant Thread as 后台线程
Client->>API : POST /api/report/generate
API->>Manager : 验证模拟状态
Manager-->>API : 返回项目信息
API->>Manager : 创建任务
API->>Thread : 启动后台生成
Thread->>Agent : 初始化ReportAgent
Agent->>Agent : 规划报告大纲
Agent->>Agent : 生成各章节内容
Agent->>Manager : 保存报告文件
API-->>Client : 返回任务状态
```

**图表来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L24-L196)
- [backend/app/services/report_agent.py](file://backend/app/services/report_agent.py#L1358-L1591)

**章节来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L24-L196)

### 报告获取接口

报告获取接口提供多种查询方式，满足不同场景的需求。

#### 接口规范

**GET /api/report/{report_id}**
- **路径参数**：`report_id` - 报告唯一标识符
- **响应**：完整的报告详情，包括大纲、内容和元数据

**GET /api/report/by-simulation/{simulation_id}**
- **路径参数**：`simulation_id` - 模拟唯一标识符
- **响应**：对应模拟的报告信息

**GET /api/report/list**
- **查询参数**：
  - `simulation_id` (可选)：按模拟ID过滤
  - `limit` (可选，默认50)：返回数量限制
- **响应**：报告列表和总数

**章节来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L272-L391)

### 报告下载接口

报告下载接口支持Markdown格式的报告文件下载，提供灵活的文件处理机制。

#### 接口规范

**GET /api/report/{report_id}/download**
- **路径参数**：`report_id` - 报告唯一标识符
- **响应**：Markdown格式的报告文件
- **返回类型**：文件流，Content-Type: text/markdown

#### 下载机制

```mermaid
flowchart TD
Request[下载请求] --> CheckFile{检查文件是否存在}
CheckFile --> |存在| DirectDownload[直接下载文件]
CheckFile --> |不存在| GenerateTemp[生成临时文件]
GenerateTemp --> WriteContent[写入报告内容]
WriteContent --> SendFile[发送文件]
DirectDownload --> End[下载完成]
SendFile --> Cleanup[清理临时文件]
Cleanup --> End
```

**图表来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L393-L437)

**章节来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L393-L437)

### 深度交互采访接口

深度交互采访接口提供与模拟Agent的智能对话能力，支持多视角的深度分析。

#### 接口规范

**POST /api/report/chat**
- **请求参数**：
  - `simulation_id` (必填)：模拟ID
  - `message` (必填)：用户消息
  - `chat_history` (可选)：对话历史记录

- **响应结构**：
  - `response`：Agent回复内容
  - `tool_calls`：调用的工具列表
  - `sources`：信息来源

#### 对话机制

```mermaid
sequenceDiagram
participant User as 用户
participant ChatAPI as 对话接口
participant Agent as ReportAgent
participant Tools as 检索工具
participant Report as 报告内容
User->>ChatAPI : POST /api/report/chat
ChatAPI->>Agent : 初始化对话
Agent->>Report : 获取报告内容
Agent->>Tools : 检索相关信息
Tools-->>Agent : 返回检索结果
Agent->>Agent : 分析工具结果
Agent-->>User : 返回对话响应
```

**图表来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L467-L559)
- [backend/app/services/report_agent.py](file://backend/app/services/report_agent.py#L1592-L1728)

**章节来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L467-L559)

### 报告进度监控接口

系统提供完善的进度监控机制，支持实时获取报告生成状态。

#### 接口规范

**GET /api/report/generate/status**
- **请求参数**：
  - `task_id` (可选)：任务ID
  - `simulation_id` (可选)：模拟ID

- **响应**：任务状态和进度信息

**GET /api/report/{report_id}/progress**
- **路径参数**：`report_id` - 报告ID
- **响应**：详细的生成进度，包括当前章节和已完成章节

**GET /api/report/{report_id}/sections**
- **路径参数**：`report_id` - 报告ID
- **响应**：已生成章节列表和完成状态

**章节来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L198-L268)
- [backend/app/api/report.py](file://backend/app/api/report.py#L564-L654)

### 日志监控接口

系统提供完整的日志监控功能，支持结构化和控制台日志的实时获取。

#### 接口规范

**GET /api/report/{report_id}/agent-log**
- **路径参数**：`report_id` - 报告ID
- **查询参数**：`from_line` (可选)：起始行号
- **响应**：结构化Agent日志

**GET /api/report/{report_id}/console-log**
- **路径参数**：`report_id` - 报告ID
- **查询参数**：`from_line` (可选)：起始行号
- **响应**：控制台风格日志

#### 日志结构

```mermaid
classDiagram
class ReportLogger {
+string report_id
+string log_file_path
+datetime start_time
+log_start() void
+log_planning_start() void
+log_section_start() void
+log_tool_call() void
+log_llm_response() void
+log_section_content() void
+log_section_full_complete() void
+log_report_complete() void
+log_error() void
}
class ReportConsoleLogger {
+string report_id
+string log_file_path
+FileHandler _file_handler
+_ensure_log_file() void
+_setup_file_handler() void
+close() void
}
ReportLogger --> ReportConsoleLogger : "配合使用"
```

**图表来源**
- [backend/app/services/report_agent.py](file://backend/app/services/report_agent.py#L35-L304)
- [backend/app/services/report_agent.py](file://backend/app/services/report_agent.py#L306-L386)

**章节来源**
- [backend/app/api/report.py](file://backend/app/api/report.py#L751-L926)

## 依赖关系分析

### 核心依赖关系

```mermaid
graph TB
subgraph "外部依赖"
ZEP[Zep Cloud API]
LLM[大语言模型API]
FS[文件系统]
end
subgraph "内部模块"
API[报告API]
AGENT[ReportAgent]
TOOLS[Zep工具服务]
MANAGER[报告管理器]
TASK[任务管理器]
LOGGER[日志系统]
end
API --> AGENT
AGENT --> TOOLS
AGENT --> TASK
AGENT --> MANAGER
AGENT --> LOGGER
TOOLS --> ZEP
TOOLS --> LLM
MANAGER --> FS
TASK --> FS
LOGGER --> FS
```

**图表来源**
- [backend/app/services/report_agent.py](file://backend/app/services/report_agent.py#L1-L50)
- [backend/app/services/zep_tools.py](file://backend/app/services/zep_tools.py#L1-L30)

### 数据模型关系

```mermaid
erDiagram
SIMULATION {
string simulation_id PK
string project_id FK
string graph_id
string status
datetime created_at
datetime updated_at
}
PROJECT {
string project_id PK
string name
string status
string graph_id
string simulation_requirement
datetime created_at
datetime updated_at
}
REPORT {
string report_id PK
string simulation_id FK
string graph_id
string status
string outline
string markdown_content
datetime created_at
datetime completed_at
}
TASK {
string task_id PK
string task_type
string status
int progress
string message
json metadata
datetime created_at
datetime updated_at
}
SIMULATION }o--|| PROJECT : "属于"
SIMULATION ||--o{ REPORT : "生成"
REPORT ||--o{ TASK : "关联"
```

**图表来源**
- [backend/app/models/project.py](file://backend/app/models/project.py#L26-L99)
- [backend/app/models/task.py](file://backend/app/models/task.py#L22-L52)

**章节来源**
- [backend/app/models/project.py](file://backend/app/models/project.py#L1-L306)
- [backend/app/models/task.py](file://backend/app/models/task.py#L1-L185)

## 性能考虑

### 异步处理机制

系统采用异步任务处理模式，避免长时间阻塞请求：

- **后台线程池**：使用独立线程处理报告生成任务
- **任务状态管理**：通过TaskManager跟踪任务进度
- **内存优化**：分章节生成和保存，避免内存峰值过高

### 缓存策略

```mermaid
flowchart LR
Request[API请求] --> CacheCheck{检查缓存}
CacheCheck --> |命中| ReturnCache[返回缓存数据]
CacheCheck --> |未命中| ProcessRequest[处理请求]
ProcessRequest --> GenerateReport[生成报告]
GenerateReport --> SaveCache[保存到缓存]
SaveCache --> ReturnResult[返回结果]
ReturnCache --> End[完成]
ReturnResult --> End
```

### 错误处理机制

系统提供多层次的错误处理和恢复机制：

- **工具重试**：Zep工具服务支持自动重试
- **异常捕获**：完整的异常捕获和日志记录
- **状态回滚**：失败时的状态回滚和清理

## 故障排除指南

### 常见问题诊断

**报告生成失败**
1. 检查模拟ID是否有效
2. 验证图谱是否已正确构建
3. 查看Agent日志获取详细错误信息

**工具调用异常**
1. 确认Zep API密钥配置正确
2. 检查网络连接和API可用性
3. 查看重试日志和错误码

**文件下载失败**
1. 检查报告文件是否存在
2. 验证文件权限设置
3. 确认磁盘空间充足

### 日志分析

系统提供详细的日志记录功能，便于问题诊断：

- **结构化日志**：agent_log.jsonl文件记录完整执行流程
- **控制台日志**：console_log.txt文件记录标准输出
- **错误日志**：统一的日志格式便于搜索和分析

**章节来源**
- [backend/app/utils/logger.py](file://backend/app/utils/logger.py#L1-L127)

## 结论

MiroFish报告API接口系统通过智能化的设计和完善的架构，为用户提供了一套完整的模拟分析报告解决方案。系统的主要优势包括：

1. **智能推理能力**：基于ReACT模式的ReportAgent能够自主规划和生成高质量报告
2. **多工具集集成**：整合深度检索、广度搜索和快速查询等多种工具
3. **实时监控机制**：提供完整的进度监控和日志记录功能
4. **异步处理架构**：支持大规模并发处理和资源优化
5. **灵活的API设计**：满足不同场景下的报告生成和查询需求

该系统不仅适用于专业的预测分析场景，也为普通用户提供了易用的报告生成工具，是构建智能预测分析平台的理想选择。