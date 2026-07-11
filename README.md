# Nova-KB 星跃新知

> 基于 RAG + 工作流 + Agent 的企业级智能问答平台

Nova-KB（星跃新知）是一个强大易用的企业级 AI 智能体平台，支持文档上传、自动爬取、文本拆分、向量化、混合检索等完整 RAG 能力，配备可视化工作流引擎和 30+ 种节点编排，可对接主流大模型，快速构建专属知识库与智能问答系统。

## 核心特性

### RAG 知识库

- 支持 Markdown / Word / PDF / Excel / CSV / HTML / ZIP 等 8 种文档格式
- 树形递归拆分算法，代码块保护，智能分段
- 向量检索 + 关键词检索 + 混合检索三种模式
- HNSW 索引加速，jieba 分词 + 术语库增强
- 文件 SHA256 去重 + zip 压缩存储

### 工作流引擎

- 30+ 种节点：AI 对话、知识库搜索、条件判断、循环、工具调用、多模态等
- 可视化拖拽编排，支持并发执行和流式输出
- 20+ 种条件比较器，支持嵌套循环

### 模型集成

- 国外：OpenAI / Claude / Gemini / Cohere / AWS Bedrock
- 国内：通义千问 / 百度千帆 / 腾讯混元 / 豆包 / 智谱 / MiniMax
- 本地：DeepSeek / Ollama / HuggingFace / Xinference
- 支持类型：LLM / Embedding / TTS / STT / TTI / Reranker

### 多模态

- 原生支持文本、图片、音频、视频的输入与输出

### 零代码嵌入

- 通过 embed.js 快速嵌入第三方系统，秒级赋予智能问答能力

## 技术栈

| 层级 | 技术 |
|------|------|
| 前端 | Vue 3 + Element Plus + Vite + TypeScript + Pinia |
| 后端 | Python 3.11 + Django 5.2 + DRF |
| LLM 框架 | LangChain 1.3 + LangGraph + DeepAgents |
| 数据库 | PostgreSQL 17 + pgvector |
| 缓存 | Redis |
| 任务队列 | Celery + APScheduler |
| 部署 | Docker |

## 快速开始

### Docker 一键部署

```bash
docker run -d --name=nova-kb --restart=always -p 8080:8080 -v ~/.nova-kb:/opt/maxkb 1panel/maxkb
```

访问 `http://localhost:8080`，默认账号：

- 用户名：`admin`
- 密码：`NovaKB@123..`

### 本地开发

#### 环境要求

- Python 3.11
- Node.js 18+
- PostgreSQL 17 + pgvector
- Redis 7+

#### 启动步骤

```bash
# 1. 启动 PostgreSQL 和 Redis（Docker）
docker run -d --name nova-pg -p 5432:5432 \
  -e POSTGRES_USER=root -e POSTGRES_PASSWORD=Password123@postgres \
  -e POSTGRES_DB=maxkb -e POSTGRES_HOST_AUTH_METHOD=trust \
  pgvector/pgvector:pg17

docker run -d --name nova-redis -p 6379:6379 \
  redis:7 --requirepass "Password123@redis"

# 2. 创建数据库扩展
docker exec nova-pg psql -U root -d maxkb -c "CREATE EXTENSION IF NOT EXISTS vector;"

# 3. 后端
python -m venv .venv
source .venv/bin/activate  # Windows: .\.venv\Scripts\Activate.ps1
pip install uv
python -m uv pip install -r pyproject.toml

# 创建 config.yml（参考 config_example.yml）
python main.py upgrade_db    # 数据库迁移
python main.py dev web       # 启动后端 → http://localhost:8080

# 4. 前端（新终端）
cd ui
npm install
npm run dev                  # 启动管理端 → http://localhost:3000
# 或
npm run chat                 # 启动对话端 → http://localhost:3001
```

#### VS Code 调试

项目已配置 `.vscode/launch.json`，按 F5 即可断点调试后端。

## 项目结构

```
nova-kb/
├── apps/                    # 后端
│   ├── maxkb/               # Django 项目配置
│   ├── application/         # 应用管理（聊天管道 + 工作流引擎）
│   │   ├── chat_pipeline/   # 聊天管道
│   │   └── flow/            # 工作流引擎（30+ 节点）
│   ├── knowledge/           # 知识库（RAG 核心）
│   │   ├── models/          # 数据模型
│   │   ├── vector/          # 向量存储（PGVector）
│   │   ├── sql/             # 检索 SQL 模板
│   │   └── task/            # 异步任务（向量化/同步）
│   ├── models_provider/     # 模型提供者（对接各厂商大模型）
│   ├── chat/                # 对话接口
│   ├── tools/               # 工具库
│   ├── trigger/             # 触发器
│   ├── common/              # 公共基础设施
│   │   ├── event/           # 事件监听器（向量化核心逻辑）
│   │   ├── handle/          # 文档拆分处理器
│   │   └── utils/           # 工具函数
│   ├── users/               # 用户管理
│   └── system_manage/       # 系统设置
├── ui/                      # 前端 Vue 项目
│   ├── src/views/           # 页面视图
│   ├── src/workflow/        # 工作流编辑器
│   └── vite.config.ts       # Vite 配置
├── installer/               # Docker 部署
├── config.yml               # 本地开发配置
├── main.py                  # 后端入口
└── pyproject.toml           # Python 依赖
```

## 检索引擎

Nova-KB 支持三种检索模式，全部基于 PostgreSQL + pgvector：

| 模式 | 说明 |
|------|------|
| 向量检索 | 余弦距离 `<=>` 排序，HNSW 索引加速 |
| 关键词检索 | PostgreSQL 全文检索 `ts_rank_cd` + jieba 分词 |
| 混合检索 | 向量召回 top N\*10 → 关键词重排 → 综合分排序 |

## 开发命令

```bash
# 后端
python main.py dev web          # 启动 Web 服务（auto-reload）
python main.py dev celery       # 启动 Celery Worker
python main.py upgrade_db       # 数据库迁移

# 前端
cd ui && npm run dev            # 管理端
cd ui && npm run chat           # 对话端
cd ui && npm run build          # 构建

# 代码检查
ruff check apps/ --fix          # Python lint（line-length=120）
```

## 致谢

本项目基于 [MaxKB](https://github.com/1Panel-dev/MaxKB) 二次开发，感谢飞致云 (FIT2CLOUD) 的开源贡献。

## 开源协议

[GPL-3.0](LICENSE)
