[根目录](../../CLAUDE.md) > **mcp_server**

# MCP Server 模块

## 模块职责

`mcp_server` 模块实现 Model Context Protocol (MCP) 服务器，让 AI 助手能够通过标准化协议访问 Graphiti 的知识图谱功能。该服务器为 Claude、Cursor 等 AI 客户端提供强大的知识图谱记忆能力。

## 入口与启动

### 主要入口点
- **主服务**: `src/graphiti_mcp_server.py` 中的 FastMCP 服务器
- **配置系统**: `config/schema.py` 中的统一配置管理
- **启动命令**: 支持多种传输模式的灵活启动

### 服务启动模式
```bash
cd mcp_server/
uv sync

# HTTP 模式 (推荐)
python src/graphiti_mcp_server.py --transport http --host 0.0.0.0 --port 3000

# stdio 模式 (传统)
python src/graphiti_mcp_server.py --transport stdio

# 使用 Docker Compose
docker-compose up
```

### 服务配置
支持 YAML 配置文件 `config/config.yaml`：
```yaml
llm:
  provider: openai
  model: gpt-4o
  api_key: ${OPENAI_API_KEY}

embedder:
  provider: openai
  model: text-embedding-3-small

database:
  provider: neo4j
  uri: bolt://localhost:7687
  user: neo4j
  password: password

graphiti:
  group_id: default
  destroy_graph: false

server:
  transport: http
  host: 0.0.0.0
  port: 3000
```

## 对外接口

### MCP 工具接口
1. **记忆管理工具**
   - `add_memory()` - 添加记忆内容 (文本/JSON/消息)
   - `search_memory_facts()` - 搜索相关事实
   - `search_nodes()` - 搜索实体节点
   - `get_episodes()` - 获取剧集列表

2. **图谱操作工具**
   - `get_entity_edge()` - 获取特定关系
   - `delete_entity_edge()` - 删除关系
   - `delete_episode()` - 删除剧集
   - `clear_graph()` - 清空图谱

3. **状态监控工具**
   - `get_status()` - 获取服务状态
   - `/health` - HTTP 健康检查端点

### 数据类型模型
- **EpisodeSearchResponse**: 剧集搜索结果
- **FactSearchResponse**: 事实搜索结果
- **NodeSearchResponse**: 节点搜索结果
- **StatusResponse**: 服务状态响应
- **SuccessResponse/ErrorResponse**: 统一响应格式

### 使用示例
```python
# 添加文本记忆
add_memory(
    name="产品特性",
    episode_body="我们的新产品支持实时协作和自动保存功能",
    source="text",
    group_id="product_team"
)

# 搜索相关事实
search_memory_facts(
    query="产品协作功能",
    max_facts=10,
    group_ids=["product_team"]
)

# 搜索实体节点
search_nodes(
    query="产品",
    entity_types=["Product", "Feature"],
    max_nodes=5
)
```

## 关键依赖与配置

### 核心组件
- **FastMCP**: MCP 协议的快速实现框架
- **Graphiti Core**: 知识图谱核心库
- **Pydantic**: 数据验证和配置管理
- **PyYAML**: YAML 配置文件解析

### 服务工厂类
- **LLMClientFactory**: 创建 LLM 客户端
- **EmbedderFactory**: 创建嵌入客户端
- **DatabaseDriverFactory**: 创建数据库驱动

### 并发控制
- **SEMAPHORE_LIMIT**: 控制并发处理数量 (默认10)
- **QueueService**: 异步任务队列服务
- **批处理优化**: 提高大规模数据处理效率

### 环境变量配置
```bash
# LLM 提供商配置
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key
GOOGLE_API_KEY=your_google_key

# 数据库配置
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=password

# 性能调优
SEMAPHORE_LIMIT=10
BROWSER=1  # 启用 FalkorDB 浏览器 UI
```

## 数据模型

### 配置模型
1. **GraphitiConfig**: 主配置类
2. **LLMConfig**: LLM 客户端配置
3. **EmbedderConfig**: 嵌入客户端配置
4. **DatabaseConfig**: 数据库配置
5. **ServerConfig**: 服务器配置

### 响应模型
```python
class SuccessResponse(BaseModel):
    message: str
    timestamp: str = Field(default_factory=lambda: datetime.now().isoformat())

class ErrorResponse(BaseModel):
    error: str
    timestamp: str = Field(default_factory=lambda: datetime.now().isoformat())
```

### 搜索结果模型
```python
class FactResult(BaseModel):
    uuid: str
    fact: str
    source_node: str
    target_node: str
    created_at: str
    group_id: str
```

## 测试与质量

### 测试覆盖
- **单元测试**: `tests/` 目录下的组件测试
- **集成测试**: 数据库集成测试 (`_int` 后缀)
- **端到端测试**: 完整的 MCP 协议测试

### 质量保证
- **类型安全**: 使用 Pydantic 进行严格的数据验证
- **错误处理**: 统一的异常处理和错误响应
- **日志记录**: 结构化日志记录所有操作
- **配置验证**: 启动时配置完整性检查

### 性能优化
- **连接池**: 数据库连接复用
- **批处理**: 大规模数据的批量处理
- **缓存**: 查询结果缓存机制
- **异步处理**: 非阻塞的并发操作

## 常见问题 (FAQ)

### Q: 如何选择传输模式？
A:
- **HTTP 模式**: 适合生产环境，支持监控和负载均衡
- **stdio 模式**: 适合本地开发和简单集成
- **SSE 模式**: 已弃用，建议使用 HTTP 模式

### Q: 如何优化 LLM API 成本？
A:
1. 调整 `SEMAPHORE_LIMIT` 控制并发
2. 选择合适的模型大小
3. 启用结果缓存
4. 监控 API 使用量

### Q: 如何处理大数据量？
A:
1. 使用批量操作接口
2. 增加处理超时时间
3. 考虑分片处理策略
4. 监控内存和数据库性能

### Q: 如何集成到 AI 助手？
A:
1. 配置 MCP 客户端连接到服务器
2. 使用工具名调用 Graphiti 功能
3. 遵循提供的交互模式
4. 处理异步操作和错误响应

## 相关文件清单

### 核心服务
- `src/graphiti_mcp_server.py` - MCP 服务器主文件
- `src/services/factories.py` - 服务工厂类
- `src/services/queue_service.py` - 异步队列服务
- `src/utils/formatting.py` - 结果格式化工具

### 配置管理
- `config/schema.py` - 配置数据模型
- `config/config.yaml` - 默认配置文件
- `.env.example` - 环境变量模板

### 数据模型
- `models/response_types.py` - API 响应模型
- `models/` - 其他数据模型定义

### 容器化
- `docker/docker-compose.yml` - Docker Compose 配置
- `docker/Dockerfile` - 容器镜像构建
- `docker/docker-compose-neo4j.yml` - Neo4j 数据库服务

### 开发文件
- `pyproject.toml` - Python 项目配置
- `uv.lock` - 依赖锁定文件
- `README.md` - 模块使用说明
- `Makefile` - 开发任务自动化

### 测试文件
- `tests/` - 测试套件目录
- `tests/conftest.py` - 测试配置
- `tests/run_tests.py` - 测试运行脚本

## 变更记录 (Changelog)

### 2025-11-23 09:20:23 - 模块文档初始化
- 创建 mcp_server 模块文档
- 添加 MCP 接口说明和配置指南
- 建立 Docker 部署和集成说明