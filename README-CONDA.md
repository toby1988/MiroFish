# MiroFish Conda 环境部署指南

本文档介绍如何使用 Conda 环境来运行 MiroFish 后端服务。

## 前置要求

| 工具 | 版本要求 | 说明 | 安装检查 |
|------|---------|------|---------|
| **Conda** | 最新版 | Python 环境管理器 | `conda --version` |
| **Node.js** | 18+ | 前端运行环境 | `node -v` |

## 快速开始

### 1. 创建并激活 Conda 环境

```bash
# 创建 Python 3.11 环境
conda create -n mirofish python=3.11 -y

# 激活环境
conda activate mirofish
```

### 2. 配置环境变量

```bash
# 复制示例配置文件到项目根目录
cp .env.example .env

# 编辑 .env 文件，填入必要的 API 密钥
```

**必需的环境变量：**

```env
# LLM API配置（支持 OpenAI SDK 格式的任意 LLM API）
# 推荐使用阿里百炼平台qwen-plus模型：https://bailian.console.aliyun.com/
# 注意消耗较大，可先进行小于40轮的模拟尝试
LLM_API_KEY=your_api_key
LLM_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
LLM_MODEL_NAME=qwen-plus

# Zep Cloud 配置
# 每月免费额度即可支撑简单使用：https://app.getzep.com/
ZEP_API_KEY=your_zep_api_key
```

### 3. 安装依赖

```bash
# 进入 backend 目录
cd backend

# 方式一：使用 requirements.txt 安装
pip install -r requirements.txt

# 方式二：使用 pyproject.toml 安装（推荐，支持开发依赖）
pip install -e .
# 或包含开发依赖
pip install -e ".[dev]"
```

### 4. 启动后端服务

```bash
# 确保在项目根目录（.env 文件所在位置）
cd /path/to/MiroFish

# 启动后端
python backend/run.py
```

或使用项目提供的 npm 命令（需先安装前端依赖）：

```bash
# 在项目根目录
npm run backend
```

### 5. 启动前端服务（可选）

```bash
# 安装前端依赖
npm ci

# 启动前端
npm run frontend
```

或使用组合命令同时启动前后端：

```bash
npm run dev
```

## 服务地址

- 前端：`http://localhost:3000`
- 后端 API：`http://localhost:5001`

## 常用 Conda 命令

```bash
# 退出环境
conda deactivate

# 删除环境
conda remove -n mirofish --all

# 查看已安装包
conda list

# 导出环境配置
conda env export > environment.yml

# 从环境文件创建环境
conda env create -f environment.yml
```

## 注意事项

1. **Python 版本**：项目要求 Python `>=3.11, <=3.12`
2. **环境激活**：确保在激活 conda 环境后再安装依赖
3. **环境变量位置**：`.env` 文件需要在项目根目录
4. **依赖安装时间**：`camel-oasis` 和 `camel-ai` 包可能需要较长时间安装，请耐心等待
5. **API 密钥**：使用前请确保已正确配置 LLM_API_KEY 和 ZEP_API_KEY

## 故障排除

### 依赖安装失败

如果 `camel-oasis` 或 `camel-ai` 安装失败，尝试：

```bash
# 更新 pip
pip install --upgrade pip

# 单独安装
pip install camel-ai==0.2.78
pip install camel-oasis==0.2.5
```

### 端口占用

如果启动时提示端口被占用：

```bash
# 查找占用 5001 端口的进程（Windows）
netstat -ano | findstr :5001

# 查找占用 3000 端口的进程（Windows）
netstat -ano | findstr :3000
```

## 与 uv 方式的对比

| 特性 | Conda | uv |
|------|-------|-----|
| 环境管理 | 独立环境 | 项目级虚拟环境 |
| 包管理 | pip + conda | uv（Rust 编写，更快） |
| 启动命令 | `conda activate mirofish` | 自动激活 |
| 适用场景 | 已有 Conda 生态 | 全新项目 |

选择 Conda 的优势：
- 与其他 Conda 项目共享环境管理经验
- 方便与科学计算库（如 NumPy、Pandas）集成
- 适合已有 Conda 工作流的开发者
