[根目录](../CLAUDE.md) > **examples**

# Examples 模块

## 模块职责

`examples` 模块包含各种使用示例和教程，展示 Graphiti 框架在不同场景下的应用方式。这些示例涵盖了从基础入门到高级集成的各种用例。

## 示例目录

### 快速入门示例
- **quickstart/** - 最基础的 Graphiti 使用示例
  - `quickstart_neo4j.py` - Neo4j 数据库示例
  - `quickstart_falkordb.py` - FalkorDB 数据库示例
  - `quickstart_neptune.py` - Amazon Neptune 示例
  - 支持文本、JSON 和消息格式的数据摄入

### 集成示例
- **azure-openai/** - Azure OpenAI 服务集成示例
- **opentelemetry/** - OpenTelemetry 分布式追踪示例
- **langgraph-agent/** - 与 LangGraph 智能代理的集成

### 应用场景示例
- **ecommerce/** - 电商产品知识图谱构建
- **podcast/** - 播客内容分析和知识提取
- **wizard_of_oz/** - 文本分析和实体提取示例

## 使用方式

每个示例目录都包含：
1. **README.md** - 详细的使用说明
2. **Python 脚本** - 可运行的示例代码
3. **环境配置** - 必要的环境变量设置
4. **数据文件** - 示例数据（如需要）

### 运行示例
```bash
# 进入示例目录
cd examples/quickstart/

# 安装依赖（如果有）
pip install -r requirements.txt

# 设置环境变量
export OPENAI_API_KEY=your_api_key

# 运行示例
python quickstart_neo4j.py
```

## 学习路径

1. **初学者**: 从 quickstart 开始，了解基本概念
2. **进阶用户**: 尝试不同的数据库集成示例
3. **高级用户**: 探索 LangGraph 代理和 OpenTelemetry 集成
4. **特定应用**: 根据需求选择电商、播客等场景示例

## 变更记录 (Changelog)

### 2025-11-23 09:20:23 - 模块文档初始化
- 创建 examples 模块概览文档
- 整理示例分类和学习路径