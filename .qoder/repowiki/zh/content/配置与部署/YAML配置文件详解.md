# YAML配置文件详解

<cite>
**本文档引用的文件**  
- [config.yaml](file://mcp_server/config/config.yaml)
- [config-docker-falkordb.yaml](file://mcp_server/config/config-docker-falkordb.yaml)
- [config-docker-neo4j.yaml](file://mcp_server/config/config-docker-neo4j.yaml)
- [config-docker-falkordb-combined.yaml](file://mcp_server/config/config-docker-falkordb-combined.yaml)
- [schema.py](file://mcp_server/src/config/schema.py)
- [graphiti_mcp_server.py](file://mcp_server/src/graphiti_mcp_server.py)
- [entity_types.py](file://mcp_server/src/models/entity_types.py)
</cite>

## 目录
1. [简介](#简介)
2. [核心配置节详解](#核心配置节详解)
3. [LLM提供商配置](#llm提供商配置)
4. [嵌入模型提供商配置](#嵌入模型提供商配置)
5. [数据库配置与切换机制](#数据库配置与切换机制)
6. [知识图谱实体类型扩展](#知识图谱实体类型扩展)
7. [配置继承与覆盖最佳实践](#配置继承与覆盖最佳实践)
8. [不同部署场景配置差异](#不同部署场景配置差异)

## 简介
本文档深入解析MCP服务器的YAML配置文件结构，涵盖`config.yaml`及其衍生配置文件。详细说明`server`、`llm`、`embedder`、`database`和`graphiti`等顶级配置节的含义与可选参数。重点描述`llm.providers`和`embedder.providers`中各LLM与嵌入模型提供商的具体配置项（如OpenAI、Anthropic、Gemini、Azure OpenAI等）。解释`database.provider`切换机制及对应数据库的连接参数。展示如何通过配置自定义实体类型（`entity_types`）以扩展知识图谱语义。提供配置文件继承与覆盖的最佳实践，并通过对比`config-docker-*.yaml`文件说明不同部署场景下的配置差异。

**Section sources**
- [config.yaml](file://mcp_server/config/config.yaml#L1-L111)

## 核心配置节详解
MCP服务器的配置文件采用模块化设计，包含多个顶级配置节，每个节负责不同的功能模块。这些配置节共同定义了服务器的行为、连接的外部服务以及数据存储方式。

### server配置节
`server`配置节定义了MCP服务器的基本网络和传输设置：

- **transport**: 传输协议类型，可选值包括`stdio`、`sse`（已弃用）和`http`（推荐）
- **host**: 服务器监听的主机地址，默认为`0.0.0.0`，允许所有网络接口访问
- **port**: 服务器监听的端口号，默认为`8000`

该配置节控制服务器如何与外部系统通信，`http`是当前推荐的传输方式，提供了更好的兼容性和性能。

### llm配置节
`llm`配置节定义了语言模型（LLM）的核心设置：

- **provider**: 当前使用的LLM提供商，可选值包括`openai`、`azure_openai`、`anthropic`、`gemini`、`groq`
- **model**: 使用的模型名称，默认为`gpt-5-mini`
- **max_tokens**: 生成文本的最大token数，默认为`4096`
- **providers**: 包含各个LLM提供商的具体配置

LLM提供商的配置支持环境变量扩展，使用`${VAR_NAME}`或`${VAR_NAME:default_value}`语法，这使得配置更加灵活，便于在不同环境中部署。

### embedder配置节
`embedder`配置节定义了嵌入模型（Embedder）的设置：

- **provider**: 当前使用的嵌入模型提供商，可选值包括`openai`、`azure_openai`、`gemini`、`voyage`
- **model**: 使用的嵌入模型名称，默认为`text-embedding-3-small`
- **dimensions**: 嵌入向量的维度，默认为`1536`
- **providers**: 包含各个嵌入模型提供商的具体配置

嵌入模型用于将文本转换为向量表示，是知识图谱搜索和相似性计算的基础。

### database配置节
`database`配置节定义了知识图谱的存储后端：

- **provider**: 当前使用的数据库提供商，可选值包括`falkordb`（默认）和`neo4j`
- **providers**: 包含各个数据库提供商的具体连接参数

数据库提供商的选择决定了知识图谱的存储引擎和查询性能特征。

### graphiti配置节
`graphiti`配置节定义了知识图谱应用的特定设置：

- **group_id**: 图谱的组ID，用于区分不同的知识域
- **episode_id_prefix**: 事件ID前缀
- **user_id**: 用户ID，用于操作追踪
- **entity_types**: 自定义实体类型的定义列表

`entity_types`配置允许用户扩展知识图谱的语义模型，定义新的实体类型及其描述。

**Section sources**
- [config.yaml](file://mcp_server/config/config.yaml#L8-L111)
- [schema.py](file://mcp_server/src/config/schema.py#L76-L292)

## LLM提供商配置
LLM提供商配置位于`llm.providers`节下，为每个支持的LLM服务提供了详细的连接参数。这些配置允许MCP服务器与不同的语言模型API进行交互。

### OpenAI配置
```yaml
openai:
  api_key: ${OPENAI_API_KEY}
  api_url: ${OPENAI_API_URL:https://api.openai.com/v1}
  organization_id: ${OPENAI_ORGANIZATION_ID:}
```

OpenAI配置包含API密钥、API端点URL和组织ID。`api_url`支持默认值，当环境变量未设置时使用指定的默认URL。

### Azure OpenAI配置
```yaml
azure_openai:
  api_key: ${AZURE_OPENAI_API_KEY}
  api_url: ${AZURE_OPENAI_ENDPOINT}
  api_version: ${AZURE_OPENAI_API_VERSION:2024-10-21}
  deployment_name: ${AZURE_OPENAI_DEPLOYMENT}
  use_azure_ad: ${USE_AZURE_AD:false}
```

Azure OpenAI配置包含API密钥、端点URL、API版本、部署名称和Azure AD认证标志。`use_azure_ad`参数控制是否使用Azure Active Directory进行认证。

### Anthropic配置
```yaml
anthropic:
  api_key: ${ANTHROPIC_API_KEY}
  api_url: ${ANTHROPIC_API_URL:https://api.anthropic.com}
  max_retries: 3
```

Anthropic配置包含API密钥、API端点URL和最大重试次数。`max_retries`参数控制在请求失败时的重试策略。

### Gemini配置
```yaml
gemini:
  api_key: ${GOOGLE_API_KEY}
  project_id: ${GOOGLE_PROJECT_ID:}
  location: ${GOOGLE_LOCATION:us-central1}
```

Gemini配置包含Google API密钥、项目ID和位置。`location`参数指定API服务的地理区域。

### Groq配置
```yaml
groq:
  api_key: ${GROQ_API_KEY}
  api_url: ${GROQ_API_URL:https://api.groq.com/openai/v1}
```

Groq配置包含API密钥和API端点URL，兼容OpenAI API格式。

**Section sources**
- [config.yaml](file://mcp_server/config/config.yaml#L18-L44)
- [schema.py](file://mcp_server/src/config/schema.py#L87-L127)

## 嵌入模型提供商配置
嵌入模型提供商配置位于`embedder.providers`节下，为不同的嵌入模型服务提供了连接参数。这些配置确保文本能够被正确地转换为向量表示。

### OpenAI嵌入配置
```yaml
openai:
  api_key: ${OPENAI_API_KEY}
  api_url: ${OPENAI_API_URL:https://api.openai.com/v1}
  organization_id: ${OPENAI_ORGANIZATION_ID:}
```

与LLM配置类似，OpenAI嵌入配置包含API密钥、API端点和组织ID，支持环境变量扩展。

### Azure OpenAI嵌入配置
```yaml
azure_openai:
  api_key: ${AZURE_OPENAI_API_KEY}
  api_url: ${AZURE_OPENAI_EMBEDDINGS_ENDPOINT}
  api_version: ${AZURE_OPENAI_API_VERSION:2024-10-21}
  deployment_name: ${AZURE_OPENAI_EMBEDDINGS_DEPLOYMENT}
  use_azure_ad: ${USE_AZURE_AD:false}
```

Azure OpenAI嵌入配置专门使用嵌入模型的端点和部署名称，与其他Azure OpenAI服务分离，允许独立配置。

### Gemini嵌入配置
```yaml
gemini:
  api_key: ${GOOGLE_API_KEY}
  project_id: ${GOOGLE_PROJECT_ID:}
  location: ${GOOGLE_LOCATION:us-central1}
```

Gemini嵌入配置与LLM配置共享相同的认证参数，确保一致性。

### Voyage嵌入配置
```yaml
voyage:
  api_key: ${VOYAGE_API_KEY}
  api_url: ${VOYAGE_API_URL:https://api.voyageai.com/v1}
  model: "voyage-3"
```

Voyage配置包含API密钥、API端点和模型名称，专为高性能嵌入服务设计。

**Section sources**
- [config.yaml](file://mcp_server/config/config.yaml#L50-L72)
- [schema.py](file://mcp_server/src/config/schema.py#L158-L165)

## 数据库配置与切换机制
数据库配置是MCP服务器的关键部分，决定了知识图谱的存储引擎和性能特征。通过`database.provider`参数可以轻松切换不同的数据库后端。

### FalkorDB配置
```yaml
falkordb:
  uri: ${FALKORDB_URI:redis://localhost:6379}
  password: ${FALKORDB_PASSWORD:}
  database: ${FALKORDB_DATABASE:default_db}
```

FalkorDB配置使用Redis协议URI连接到图数据库，支持密码认证和数据库选择。在Docker环境中，URI通常指向`redis://falkordb:6379`，利用Docker服务发现机制。

### Neo4j配置
```yaml
neo4j:
  uri: ${NEO4J_URI:bolt://localhost:7687}
  username: ${NEO4J_USER:neo4j}
  password: ${NEO4J_PASSWORD}
  database: ${NEO4J_DATABASE:neo4j}
  use_parallel_runtime: ${USE_PARALLEL_RUNTIME:false}
```

Neo4j配置使用Bolt协议URI连接到图数据库，包含用户名、密码、数据库名称和并行运行时标志。`use_parallel_runtime`参数控制查询执行的并行化策略。

### 数据库切换机制
数据库切换通过修改`database.provider`参数实现：

```yaml
database:
  provider: "falkordb"  # 或 "neo4j"
```

当`provider`值改变时，MCP服务器会自动加载相应数据库提供商的配置，并初始化对应的数据库驱动。这种设计实现了数据库后端的热插拔，无需修改代码即可切换存储引擎。

在服务器初始化过程中，`GraphitiService`会根据配置的`provider`值选择合适的数据库驱动：

```python
if self.config.database.provider.lower() == 'falkordb':
    # 初始化FalkorDB驱动
else:
    # 初始化Neo4j驱动（默认）
```

这种机制确保了配置的灵活性和系统的可扩展性。

**Section sources**
- [config.yaml](file://mcp_server/config/config.yaml#L73-L88)
- [schema.py](file://mcp_server/src/config/schema.py#L176-L199)
- [graphiti_mcp_server.py](file://mcp_server/src/graphiti_mcp_server.py#L214-L240)

## 知识图谱实体类型扩展
`graphiti.entity_types`配置允许用户自定义知识图谱中的实体类型，扩展系统的语义能力。这是MCP服务器灵活性的重要体现。

### 内置实体类型
默认配置包含以下实体类型：

- **Preference**: 用户偏好、选择、意见或偏好
- **Requirement**: 具体需求、功能或必须满足的特性
- **Procedure**: 标准操作程序和顺序指令
- **Location**: 活动发生的物理或虚拟场所
- **Event**: 有时限的活动、事件或经历
- **Organization**: 公司、机构、团体或正式实体
- **Document**: 各种形式的信息内容
- **Topic**: 对话主题、兴趣领域或知识域
- **Object**: 物理物品、工具、设备或财产

每个实体类型都有详细的描述，指导系统如何识别和分类实体。

### 自定义实体类型
用户可以通过修改`entity_types`列表来添加新的实体类型：

```yaml
entity_types:
  - name: "CustomType"
    description: "自定义实体类型的描述"
  - name: "AnotherType"
    description: "另一个自定义实体类型的描述"
```

在服务器初始化时，这些配置会被动态转换为Pydantic模型：

```python
for entity_type in self.config.graphiti.entity_types:
    entity_model = type(
        entity_type.name,
        (BaseModel,),
        {
            '__doc__': entity_type.description,
        },
    )
    custom_types[entity_type.name] = entity_model
```

这种动态模型创建机制使得系统能够灵活适应不同的应用场景，无需修改代码即可扩展实体类型。

### 实体类型优先级
某些实体类型具有明确的优先级规则。例如，`Preference`类型被指示优先于大多数其他类型，这确保了用户偏好能够被准确识别和处理。

**Section sources**
- [config.yaml](file://mcp_server/config/config.yaml#L89-L111)
- [schema.py](file://mcp_server/src/config/schema.py#L208-L227)
- [entity_types.py](file://mcp_server/src/models/entity_types.py#L6-L226)
- [graphiti_mcp_server.py](file://mcp_server/src/graphiti_mcp_server.py#L194-L210)

## 配置继承与覆盖最佳实践
MCP服务器的配置系统支持多层继承和覆盖，提供了灵活的配置管理策略。

### 配置加载优先级
配置系统遵循以下优先级顺序（从高到低）：

1. **命令行参数**: 直接通过CLI传递的参数具有最高优先级
2. **环境变量**: 通过环境变量设置的值
3. **YAML配置文件**: 从`config.yaml`等文件加载的配置
4. **默认值**: 在代码中定义的默认配置

这种设计允许用户在不同层级上覆盖配置，实现精细化控制。

### CLI参数覆盖
服务器支持通过命令行参数覆盖配置文件中的设置：

```bash
python main.py \
  --llm-provider anthropic \
  --model claude-3-opus \
  --database-provider neo4j \
  --group-id my-project
```

这些参数会覆盖配置文件中的相应设置，`apply_cli_overrides`方法负责处理这种覆盖逻辑。

### 环境变量扩展
YAML配置文件支持环境变量扩展，使用`${VAR_NAME}`或`${VAR_NAME:default_value}`语法：

```yaml
api_key: ${OPENAI_API_KEY}
uri: ${FALKORDB_URI:redis://localhost:6379}
```

这种机制使得配置文件可以在不同环境中保持不变，只需设置相应的环境变量即可。

### 配置验证
配置系统使用Pydantic进行类型验证和数据验证，确保配置的正确性。`GraphitiConfig`类定义了所有配置项的类型和默认值，提供了强大的类型安全保证。

**Section sources**
- [schema.py](file://mcp_server/src/config/schema.py#L248-L292)
- [graphiti_mcp_server.py](file://mcp_server/src/graphiti_mcp_server.py#L845-L848)

## 不同部署场景配置差异
MCP服务器提供了多个预配置的YAML文件，针对不同的部署场景进行了优化。

### 通用配置 (config.yaml)
`config.yaml`是通用配置文件，适用于本地开发和测试：

- 使用本地回环地址连接数据库
- 包含所有支持的LLM和嵌入模型提供商配置
- 提供详细的注释和文档

### Docker FalkorDB配置
`config-docker-falkordb.yaml`针对Docker Compose部署进行了优化：

```yaml
falkordb:
  uri: ${FALKORDB_URI:redis://falkordb:6379}
```

关键差异在于数据库URI使用`falkordb`作为主机名，利用Docker服务发现机制，而不是`localhost`。

### Docker Neo4j配置
`config-docker-neo4j.yaml`针对Neo4j的Docker部署进行了优化：

```yaml
neo4j:
  uri: ${NEO4J_URI:bolt://neo4j:7687}
  password: ${NEO4J_PASSWORD:demodemo}
```

除了使用`neo4j`作为主机名外，还提供了默认的演示密码，便于快速启动和测试。

### 组合式部署配置
`config-docker-falkordb-combined.yaml`用于单容器组合部署：

```yaml
falkordb:
  uri: ${FALKORDB_URI:redis://localhost:6379}
```

在这种部署模式下，MCP服务器和FalkorDB运行在同一容器内，因此使用`localhost`而不是服务名称进行通信。

这些不同的配置文件展示了如何根据部署架构调整网络连接参数，同时保持核心功能配置的一致性。

**Section sources**
- [config-docker-falkordb.yaml](file://mcp_server/config/config-docker-falkordb.yaml#L74-L76)
- [config-docker-neo4j.yaml](file://mcp_server/config/config-docker-neo4j.yaml#L74-L78)
- [config-docker-falkordb-combined.yaml](file://mcp_server/config/config-docker-falkordb-combined.yaml#L74-L76)
- [config.yaml](file://mcp_server/config/config.yaml#L78-L86)