# 最大边际相关性 (MMR)

<cite>
**本文档引用的文件**   
- [search_config.py](file://graphiti_core/search/search_config.py)
- [search_config_recipes.py](file://graphiti_core/search/search_config_recipes.py)
- [search.py](file://graphiti_core/search/search.py)
- [search_utils.py](file://graphiti_core/search/search_utils.py)
</cite>

## 目录
1. [引言](#引言)
2. [MMR实现机制](#mmr实现机制)
3. [配置方法](#配置方法)
4. [参数影响分析](#参数影响分析)
5. [优势与局限性](#优势与局限性)
6. [组合使用场景](#组合使用场景)

## 引言
最大边际相关性（MMR）是一种用于平衡检索结果相关性与多样性的重排序策略。在Graphiti系统中，MMR通过计算候选节点与查询的相关性以及候选节点之间的相似性来避免结果冗余。该策略在处理主题分散的查询时表现出显著优势，能够有效提升检索结果的质量。

## MMR实现机制
最大边际相关性（MMR）算法通过综合考虑候选项目与查询的相关性以及候选项目之间的相似性来实现结果的重排序。其核心公式为：MMR = λ * QuerySimilarity - (1-λ) * MaxSimilarity，其中λ是控制相关性与多样性平衡的参数。

在Graphiti系统中，MMR的实现主要包含以下步骤：
1. 计算查询向量与所有候选项目的相关性
2. 计算候选项目之间的两两相似性
3. 使用MMR公式计算每个候选项目的综合得分
4. 根据综合得分对候选项目进行排序

该算法通过在相关性和多样性之间进行权衡，有效避免了传统检索方法中常见的结果冗余问题。

**Section sources**
- [search_utils.py](file://graphiti_core/search/search_utils.py#L1837-L1876)
- [search.py](file://graphiti_core/search/search.py#L258-L268)

## 配置方法
在Graphiti系统中，MMR可以作为NodeReranker和EdgeReranker的选项进行配置。通过SearchConfig类，用户可以灵活地设置MMR参数。

配置示例：
```python
SearchConfig(
    node_config=NodeSearchConfig(
        search_methods=[NodeSearchMethod.bm25, NodeSearchMethod.cosine_similarity],
        reranker=NodeReranker.mmr,
        mmr_lambda=0.7
    ),
    edge_config=EdgeSearchConfig(
        search_methods=[EdgeSearchMethod.bm25, EdgeSearchMethod.cosine_similarity],
        reranker=EdgeReranker.mmr,
        mmr_lambda=0.7
    )
)
```

系统还提供了预定义的配置配方，如COMBINED_HYBRID_SEARCH_MMR，可以直接使用。

**Section sources**
- [search_config.py](file://graphiti_core/search/search_config.py#L53-L67)
- [search_config_recipes.py](file://graphiti_core/search/search_config_recipes.py#L54-L78)

## 参数影响分析
MMR算法中的lambda参数对结果的多样性有显著影响。当lambda值较高时（接近1.0），系统更注重查询相关性，返回的结果与查询高度相关但可能缺乏多样性；当lambda值较低时（接近0.0），系统更注重结果的多样性，可能会牺牲一定的相关性。

在实际应用中，lambda的默认值为0.5，提供了一个平衡点。用户可以根据具体需求调整该参数：
- 对于需要高相关性的场景，可将lambda设置为0.7-0.9
- 对于需要高多样性的场景，可将lambda设置为0.1-0.3

**Section sources**
- [search_config.py](file://graphiti_core/search/search_config.py#L25)
- [search_utils.py](file://graphiti_core/search/search_utils.py#L65)

## 优势与局限性
MMR在处理主题分散的查询时具有明显优势。当查询涉及多个子主题时，MMR能够确保结果覆盖不同的方面，而不是集中在单一主题上。这种特性使其特别适用于探索性搜索场景。

然而，MMR也存在一些局限性：
1. 计算开销相对较高，需要计算候选项目之间的两两相似性
2. 对于高度相关的查询，可能会引入不相关的多样化结果
3. 参数调优需要领域知识和实验验证

尽管存在这些局限性，MMR仍然是平衡相关性与多样性的重要工具。

**Section sources**
- [search.py](file://graphiti_core/search/search.py#L88-L102)
- [search_utils.py](file://graphiti_core/search/search_utils.py#L1844-L1872)

## 组合使用场景
MMR可以与其他重排序策略组合使用，以实现更复杂的检索需求。常见的组合场景包括：
1. 与交叉编码器（cross_encoder）结合：先使用MMR获得多样化的候选集，再使用交叉编码器进行精细的相关性排序
2. 与节点距离（node_distance）结合：在保持多样性的同时，优先考虑与中心节点接近的结果
3. 与事件提及（episode_mentions）结合：在多样化的基础上，优先考虑被更多事件提及的实体

这些组合策略可以根据具体应用场景灵活配置，以达到最佳的检索效果。

**Section sources**
- [search.py](file://graphiti_core/search/search.py#L258-L306)
- [search_config.py](file://graphiti_core/search/search_config.py#L53-L78)