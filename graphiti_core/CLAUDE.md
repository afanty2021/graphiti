[根目录](../../CLAUDE.md) > **graphiti_core**

# Graphiti Core 模块

## 模块职责

`graphiti_core` 是 Graphiti 框架的核心库，提供完整的知识图谱构建和管理功能。该模块包含了所有核心算法、数据结构、数据库驱动和客户端实现。

## 入口与启动

### 主要入口点
- **主类**: `graphiti.py` 中的 `Graphiti` 类
- **初始化**: 创建 Graphiti 实例需要数据库连接和可选的 LLM/嵌入客户端
- **核心方法**: `add_episode()`, `search()`, `build_indices_and_constraints()`

### 基本使用模式
```python
from graphiti_core import Graphiti

# 初始化
graphiti = Graphiti(
    uri="bolt://localhost:7687",
    user="neo4j",
    password="password"
)

# 构建索引
await graphiti.build_indices_and_constraints()

# 添加数据
await graphiti.add_episode(
    name="用户会话",
    episode_body="用户询问关于产品功能的问题",
    source_description="客服对话"
)

# 搜索
results = await graphiti.search("产品功能")
```

## 对外接口

### 核心API接口
1. **数据摄入接口**
   - `add_episode()` - 添加单个剧集
   - `add_episode_bulk()` - 批量添加剧集
   - `add_triplet()` - 添加三元组关系

2. **搜索检索接口**
   - `search()` - 基础混合搜索
   - `search_()` - 高级搜索方法
   - `retrieve_episodes()` - 获取历史剧集

3. **图谱管理接口**
   - `build_indices_and_constraints()` - 构建数据库索引
   - `build_communities()` - 社区发现
   - `close()` - 关闭数据库连接

### 数据模型接口
- **Node类层次**: `Node`, `EntityNode`, `EpisodicNode`, `CommunityNode`
- **Edge类层次**: `Edge`, `EntityEdge`, `EpisodicEdge`, `CommunityEdge`
- **搜索配置**: `SearchConfig`, `SearchFilters`, `SearchResults`

## 关键依赖与配置

### 核心组件依赖
- **数据库驱动** (`driver/`):
  - Neo4j 驱动 (默认)
  - FalkorDB 驱动
  - Kuzu 驱动
  - Neptune 驱动

- **LLM客户端** (`llm_client/`):
  - OpenAI 客户端 (默认)
  - Anthropic 客户端
  - Gemini 客户端
  - Groq 客户端

- **嵌入客户端** (`embedder/`):
  - OpenAI 嵌入 (默认)
  - Gemini 嵌入
  - Voyage AI 嵌入
  - Sentence Transformers

### 配置参数
- **并发控制**: `max_coroutines` 参数控制并发操作数
- **内容存储**: `store_raw_episode_content` 控制是否存储原始内容
- **追踪**: `tracer` 参数支持 OpenTelemetry 分布式追踪

## 数据模型

### 节点类型 (nodes.py)
1. **EntityNode**: 实体节点，代表现实世界中的实体
2. **EpisodicNode**: 剧集节点，存储原始数据内容
3. **CommunityNode**: 社区节点，代表实体聚类

### 边类型 (edges.py)
1. **EntityEdge**: 实体间关系，包含事实陈述
2. **EpisodicEdge**: 剧集与实体的关联关系
3. **CommunityEdge**: 社区与实体的关联关系

### 时间模型
- **created_at**: 记录创建时间
- **valid_at**: 事件发生时间
- **invalidated_at**: 关系失效时间（支持双时态查询）

## 测试与质量

### 测试覆盖
- **单元测试**: `tests/` 目录下的全面测试套件
- **集成测试**: 标记为 `_int` 后缀的数据库集成测试
- **类型检查**: 使用 Pyright 进行静态类型检查

### 质量保证
- **代码格式**: 使用 Ruff 进行格式化和 linting
- **错误处理**: 全面的异常处理和错误传播
- **并发安全**: 支持高并发操作和资源管理

## 常见问题 (FAQ)

### Q: 如何选择合适的数据库后端？
A:
- **Neo4j**: 生产环境推荐，功能最完整
- **FalkorDB**: 轻量级替代方案，适合快速原型
- **Kuzu**: 嵌入式图数据库，适合单机应用
- **Neptune**: AWS 云端图数据库，适合大规模部署

### Q: 如何优化搜索性能？
A:
1. 使用合适的搜索配置 (`search_config_recipes.py`)
2. 根据数据量调整搜索限制
3. 考虑使用重新排序器提高相关性
4. 定期维护数据库索引

### Q: 如何处理大规模数据？
A:
1. 使用 `add_episode_bulk()` 进行批量处理
2. 调整 `SEMAPHORE_LIMIT` 控制并发度
3. 考虑分批处理避免内存溢出
4. 监控 LLM API 使用量和成本

### Q: 如何自定义实体类型？
A:
1. 创建继承自 BaseModel 的 Pydantic 模型
2. 在 `add_episode()` 中传入 `entity_types` 参数
3. 系统会自动提取和分类自定义实体

## 相关文件清单

### 核心文件
- `graphiti.py` - 主要的 Graphiti 类实现
- `nodes.py` - 节点数据结构定义
- `edges.py` - 边数据结构定义
- `search/search.py` - 搜索算法实现
- `search/search_config.py` - 搜索配置管理

### 驱动文件
- `driver/driver.py` - 数据库驱动抽象接口
- `driver/neo4j_driver.py` - Neo4j 驱动实现
- `driver/falkordb_driver.py` - FalkorDB 驱动实现

### 客户端文件
- `llm_client/client.py` - LLM 客户端抽象
- `llm_client/openai_client.py` - OpenAI 客户端
- `embedder/embedder.py` - 嵌入客户端抽象
- `embedder/openai.py` - OpenAI 嵌入实现

### 工具文件
- `utils/bulk_utils.py` - 批量处理工具
- `utils/maintenance/` - 图谱维护操作
- `prompts/` - LLM 提示模板
- `search/search_config_recipes.py` - 搜索配置配方

## 变更记录 (Changelog)

### 2025-11-23 09:20:23 - 模块文档初始化
- 创建 graphiti_core 模块文档
- 添加详细的功能描述和使用指南
- 建立文件清单和FAQ部分