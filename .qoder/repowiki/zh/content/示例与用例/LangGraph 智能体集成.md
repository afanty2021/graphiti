# LangGraph 智能体集成

<cite>
**本文档中引用的文件**   
- [agent.ipynb](file://examples/langgraph-agent/agent.ipynb)
- [graphiti.py](file://graphiti_core/graphiti.py)
- [search.py](file://graphiti_core/search/search.py)
- [nodes.py](file://graphiti_core/nodes.py)
- [edges.py](file://graphiti_core/edges.py)
- [manybirds_products.json](file://examples/data/manybirds_products.json)
</cite>

## 目录
1. [简介](#简介)
2. [项目结构](#项目结构)
3. [核心组件](#核心组件)
4. [架构概述](#架构概述)
5. [详细组件分析](#详细组件分析)
6. [依赖关系分析](#依赖关系分析)
7. [性能考虑](#性能考虑)
8. [故障排除指南](#故障排除指南)
9. [结论](#结论)

## 简介
本文档提供了一个交互式教程，展示如何将LangGraph智能体与Graphiti知识图谱系统集成。通过Jupyter Notebook示例，详细说明了如何在智能体工作流中嵌入知识图谱能力，实现记忆持久化和上下文检索功能。

## 项目结构
该代码库的结构围绕Graphiti核心功能和示例应用组织。核心功能位于`graphiti_core`目录中，包含图操作、搜索、嵌入和驱动等模块。示例应用位于`examples`目录中，包括LangGraph智能体集成、电子商务场景和快速入门指南等。

**Section sources**
- [agent.ipynb](file://examples/langgraph-agent/agent.ipynb)
- [graphiti.py](file://graphiti_core/graphiti.py)

## 核心组件
文档的核心是LangGraph智能体与Graphiti的集成实现。通过`agent.ipynb`示例，展示了如何配置Graphiti客户端、加载产品数据、创建用户节点，并构建能够利用知识图谱进行上下文感知响应的智能体。

**Section sources**
- [agent.ipynb](file://examples/langgraph-agent/agent.ipynb)
- [graphiti.py](file://graphiti_core/graphiti.py)

## 架构概述
系统架构由LangGraph状态机和Graphiti知识图谱组成。智能体在处理用户请求时，会从图谱中检索相关信息，并将新的交互持久化到图谱中，形成闭环的知识管理。

```mermaid
graph TB
subgraph "LangGraph智能体"
StateGraph[状态机]
MemorySaver[内存保存器]
ToolNode[工具节点]
end
subgraph "Graphiti知识图谱"
GraphitiClient[Graphiti客户端]
Neo4j[Neo4j数据库]
Search[搜索功能]
AddEpisode[添加片段]
end
User[用户输入] --> StateGraph
StateGraph --> Search
Search --> GraphitiClient
GraphitiClient --> Neo4j
Neo4j --> GraphitiClient
GraphitiClient --> StateGraph
StateGraph --> AddEpisode
AddEpisode --> Neo4j
StateGraph --> MemorySaver
```

**Diagram sources**
- [agent.ipynb](file://examples/langgraph-agent/agent.ipynb)
- [graphiti.py](file://graphiti_core/graphiti.py)

## 详细组件分析
### 智能体工作流分析
智能体工作流的核心是`chatbot`函数，它实现了上下文感知的对话处理。该函数首先从知识图谱中检索与当前对话相关的事实，然后构建系统消息，最后生成响应并将其持久化到图谱中。

#### 状态管理
```mermaid
classDiagram
class State {
+messages : list
+user_name : str
+user_node_uuid : str
}
class TypedDict {
+__annotations__ : dict
}
State --|> TypedDict : 继承
```

**Diagram sources**
- [agent.ipynb](file://examples/langgraph-agent/agent.ipynb)

#### 图谱交互
```mermaid
sequenceDiagram
participant User as 用户
participant Agent as 智能体
participant Graphiti as Graphiti客户端
participant Neo4j as Neo4j数据库
User->>Agent : 发送消息
Agent->>Graphiti : search(查询, center_node_uuid)
Graphiti->>Neo4j : 执行图查询
Neo4j-->>Graphiti : 返回相关边
Graphiti-->>Agent : 返回事实字符串
Agent->>LLM : 调用大语言模型
LLM-->>Agent : 生成响应
Agent->>Graphiti : add_episode(响应)
Graphiti->>Neo4j : 持久化新片段
Neo4j-->>Graphiti : 确认保存
Graphiti-->>Agent : 确认
Agent-->>User : 返回响应
```

**Diagram sources**
- [agent.ipynb](file://examples/langgraph-agent/agent.ipynb)
- [graphiti.py](file://graphiti_core/graphiti.py)

### 数据模型分析
Graphiti系统使用节点和边来表示知识。节点代表实体（如用户、产品），边代表实体间的关系。

#### 节点模型
```mermaid
classDiagram
class Node {
+uuid : str
+name : str
+group_id : str
+labels : list[str]
+created_at : datetime
}
class EpisodicNode {
+source : EpisodeType
+source_description : str
+content : str
+valid_at : datetime
+entity_edges : list[str]
}
class EntityNode {
+name_embedding : list[float]
+summary : str
+attributes : dict[str, Any]
}
Node <|-- EpisodicNode
Node <|-- EntityNode
```

**Diagram sources**
- [nodes.py](file://graphiti_core/nodes.py)

#### 边模型
```mermaid
classDiagram
class Edge {
+uuid : str
+group_id : str
+source_node_uuid : str
+target_node_uuid : str
+created_at : datetime
}
class EpisodicEdge {
}
class EntityEdge {
+name : str
+fact : str
+fact_embedding : list[float]
+episodes : list[str]
+expired_at : datetime
+valid_at : datetime
+invalid_at : datetime
+attributes : dict[str, Any]
}
Edge <|-- EpisodicEdge
Edge <|-- EntityEdge
```

**Diagram sources**
- [edges.py](file://graphiti_core/edges.py)

**Section sources**
- [nodes.py](file://graphiti_core/nodes.py)
- [edges.py](file://graphiti_core/edges.py)

## 依赖关系分析
系统依赖关系清晰，LangGraph智能体依赖Graphiti客户端，Graphiti客户端又依赖Neo4j数据库驱动。

```mermaid
graph TD
LangGraphAgent[LangGraph智能体] --> GraphitiClient[Graphiti客户端]
GraphitiClient --> Neo4jDriver[Neo4j驱动]
Neo4jDriver --> Neo4jDB[(Neo4j数据库)]
GraphitiClient --> LLMClient[LLM客户端]
GraphitiClient --> EmbedderClient[嵌入客户端]
LLMClient --> OpenAI[OpenAI API]
EmbedderClient --> OpenAI
```

**Diagram sources**
- [graphiti.py](file://graphiti_core/graphiti.py)

## 性能考虑
在集成过程中需要考虑性能因素。异步操作被用于避免阻塞图执行，特别是在调用`add_episode`方法时使用`asyncio.create_task`。

**Section sources**
- [agent.ipynb](file://examples/langgraph-agent/agent.ipynb)

## 故障排除指南
常见问题包括环境配置、数据库连接和API密钥设置。确保正确设置NEO4J_URI、NEO4J_USER和NEO4J_PASSWORD环境变量。

**Section sources**
- [agent.ipynb](file://examples/langgraph-agent/agent.ipynb)

## 结论
通过本文档，开发者可以了解如何将LangGraph智能体与Graphiti知识图谱集成，构建具备长期记忆能力的复合智能体系统。这种集成方式使得智能体能够利用历史交互数据做出更智能的决策。