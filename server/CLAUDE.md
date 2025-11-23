[根目录](../../CLAUDE.md) > **server**

# Server 模块

## 模块职责

`server` 模块提供基于 FastAPI 的 REST API 服务，将 Graphiti 核心功能暴露为 HTTP 接口。该服务支持数据摄入、检索查询和图谱管理操作，适合作为生产环境中的微服务部署。

## 入口与启动

### 主要入口点
- **主应用**: `graph_service/main.py` 中的 FastAPI 应用实例
- **启动命令**: `uvicorn graph_service.main:app --reload`
- **健康检查**: `/healthcheck` 端点用于服务状态监控

### 服务初始化流程
1. **配置加载**: 从 `config.py` 加载环境配置
2. **Graphiti 初始化**: 在 `zep_graphiti.py` 中初始化核心客户端
3. **路由注册**: 注册数据摄入和检索路由
4. **生命周期管理**: 使用 FastAPI 的 lifespan 管理资源

### 基本启动模式
```bash
cd server/
uv sync --extra dev
uvicorn graph_service.main:app --reload --host 0.0.0.0 --port 8000
```

## 对外接口

### API 路由结构
1. **数据摄入接口** (`routers/ingest.py`)
   - `POST /ingest/episode` - 添加单个剧集
   - `POST /ingest/episodes/bulk` - 批量添加剧集
   - `POST /ingest/triplet` - 添加三元组关系

2. **数据检索接口** (`routers/retrieve.py`)
   - `GET /retrieve/search` - 搜索图谱内容
   - `GET /retrieve/episodes` - 获取剧集列表
   - `GET /retrieve/nodes` - 获取节点信息
   - `GET /retrieve/edges` - 获取边信息

3. **管理接口**
   - `GET /healthcheck` - 服务健康检查
   - `POST /maintenance/indices` - 重建索引
   - `DELETE /maintenance/clear` - 清空图谱

### 数据传输对象 (DTOs)
- **通用DTO** (`dto/common.py`): 基础响应模型
- **摄入DTO** (`dto/ingest.py`): 数据摄入请求模型
- **检索DTO** (`dto/retrieve.py`): 数据检索请求模型

### 响应格式
所有API响应遵循统一格式：
```json
{
  "success": true,
  "message": "操作成功",
  "data": {...},
  "timestamp": "2025-11-23T09:20:23Z"
}
```

## 关键依赖与配置

### 核心依赖
- **FastAPI**: Web 框架，提供高性能异步 API
- **Pydantic**: 数据验证和序列化
- **Uvicorn**: ASGI 服务器
- **Graphiti Core**: 核心知识图谱库

### 配置管理
配置文件位于 `graph_service/config.py`：

```python
class Settings:
    # 数据库配置
    neo4j_uri: str = "bolt://localhost:7687"
    neo4j_user: str = "neo4j"
    neo4j_password: str = "password"

    # LLM 配置
    openai_api_key: str
    model_name: str = "gpt-4o"

    # 服务配置
    host: str = "0.0.0.0"
    port: int = 8000
    debug: bool = False
```

### 环境变量
- `NEO4J_URI`, `NEO4J_USER`, `NEO4J_PASSWORD` - Neo4j 连接参数
- `OPENAI_API_KEY` - OpenAI API 密钥
- `HOST`, `PORT` - 服务绑定参数
- `DEBUG` - 调试模式开关

## 数据模型

### 请求模型
1. **EpisodeRequest**: 剧集添加请求
   ```python
   class EpisodeRequest(BaseModel):
       name: str
       episode_body: str
       source_description: str
       reference_time: datetime
       source: EpisodeType = EpisodeType.text
       group_id: Optional[str] = None
   ```

2. **SearchRequest**: 搜索请求
   ```python
   class SearchRequest(BaseModel):
       query: str
       num_results: int = 10
       group_ids: Optional[List[str]] = None
       search_type: str = "hybrid"
   ```

### 响应模型
1. **SearchResponse**: 搜索结果响应
   ```python
   class SearchResponse(BaseModel):
       edges: List[EntityEdge]
       nodes: List[Node]
       total_count: int
       query_time_ms: float
   ```

2. **EpisodeResponse**: 剧集操作响应
   ```python
   class EpisodeResponse(BaseModel):
       episode_uuid: str
       nodes_created: int
       edges_created: int
       processing_time_ms: float
   ```

## 测试与质量

### 测试覆盖
- **单元测试**: `tests/` 目录下的组件测试
- **集成测试**: `_int` 后缀的端到端测试
- **API测试**: 使用 TestClient 进行的HTTP测试

### 质量保证
- **API文档**: 自动生成的 OpenAPI/Swagger 文档
- **输入验证**: Pydantic 模型进行严格的数据验证
- **错误处理**: 统一的异常处理和错误响应
- **日志记录**: 结构化日志记录所有API调用

### 性能监控
- **请求追踪**: 记录每个API请求的执行时间
- **资源监控**: 监控数据库连接和内存使用
- **错误统计**: 跟踪错误率和失败模式

## 常见问题 (FAQ)

### Q: 如何部署到生产环境？
A:
1. 使用 Docker 容器化部署
2. 配置反向代理 (如 Nginx)
3. 启用 HTTPS 和安全头部
4. 设置适当的资源限制和监控

### Q: 如何处理大量并发请求？
A:
1. 使用 Uvicorn 的多进程模式
2. 配置数据库连接池
3. 实现请求限流和负载均衡
4. 考虑异步任务队列处理长时间操作

### Q: 如何扩展API功能？
A:
1. 在 `routers/` 目录下添加新的路由文件
2. 定义相应的 Pydantic 模型
3. 在 `main.py` 中注册新路由
4. 添加相应的测试用例

### Q: 如何进行API版本管理？
A:
1. 使用路径前缀进行版本控制 (`/v1/`, `/v2/`)
2. 保持向后兼容性
3. 逐步废弃旧版本API
4. 提供迁移指南和工具

## 相关文件清单

### 应用文件
- `graph_service/main.py` - FastAPI 应用主文件
- `graph_service/config.py` - 配置管理
- `graph_service/zep_graphiti.py` - Graphiti 客户端初始化

### 路由文件
- `graph_service/routers/ingest.py` - 数据摄入API
- `graph_service/routers/retrieve.py` - 数据检索API

### 数据模型
- `graph_service/dto/common.py` - 通用数据模型
- `graph_service/dto/ingest.py` - 摄入请求模型
- `graph_service/dto/retrieve.py` - 检索请求模型

### 配置文件
- `pyproject.toml` - Python 项目配置
- `uv.lock` - 依赖锁定文件
- `Dockerfile` - Docker 容器配置 (如存在)

### 开发文件
- `Makefile` - 开发任务自动化
- `README.md` - 模块使用说明

## 变更记录 (Changelog)

### 2025-11-23 09:20:23 - 模块文档初始化
- 创建 server 模块文档
- 添加API接口说明和使用指南
- 建立部署和配置说明