# SSE传输协议配置

<cite>
**本文档中引用的文件**  
- [config.yaml](file://mcp_server/config/config.yaml)
- [graphiti_mcp_server.py](file://mcp_server/src/graphiti_mcp_server.py)
- [schema.py](file://mcp_server/src/config/schema.py)
- [test_mcp_transports.py](file://mcp_server/tests/test_mcp_transports.py)
- [README.md](file://mcp_server/README.md)
</cite>

## 目录
1. [SSE传输协议概述](#sse传输协议概述)
2. [配置方法](#配置方法)
3. [已弃用原因与替代方案](#已弃用原因与替代方案)
4. [SSE流式响应实现机制](#sse流式响应实现机制)
5. [客户端消费SSE流示例](#客户端消费sse流示例)
6. [现代应用架构中的推荐方案](#现代应用架构中的推荐方案)
7. [迁移指南](#迁移指南)

## SSE传输协议概述

SSE（Server-Sent Events）是一种服务器向客户端单向推送数据的HTTP流式传输协议。在MCP服务器中，SSE曾作为传输协议之一，用于支持客户端实时接收服务器推送的事件流。然而，根据代码和文档分析，SSE传输已被标记为“已弃用”，不再推荐使用。

SSE协议适用于需要服务器主动向客户端推送更新的场景，如实时通知、日志流或状态更新。其基于HTTP长连接，服务器可以持续向客户端发送事件，而无需客户端频繁轮询。

**Section sources**
- [config.yaml](file://mcp_server/config/config.yaml#L9)
- [graphiti_mcp_server.py](file://mcp_server/src/graphiti_mcp_server.py#L784)
- [schema.py](file://mcp_server/src/config/schema.py#L81)

## 配置方法

在`config.yaml`文件中，SSE传输的配置通过`server.transport`字段设置为`sse`来启用。该字段的默认值为`http`，推荐使用HTTP传输。

```yaml
server:
  transport: "sse"  # 可选值：stdio, sse (已弃用), http
  host: "0.0.0.0"
  port: 8000
```

此外，也可以通过命令行参数`--transport`来覆盖配置文件中的设置。例如：

```bash
uv run graphiti_mcp_server.py --transport sse
```

当`transport`设置为`sse`时，MCP服务器将启动SSE传输模式，监听指定的主机和端口，并在`/sse`路径上提供SSE流服务。

**Section sources**
- [config.yaml](file://mcp_server/config/config.yaml#L8-L12)
- [graphiti_mcp_server.py](file://mcp_server/src/graphiti_mcp_server.py#L781-L785)
- [schema.py](file://mcp_server/src/config/schema.py#L76-L85)

## 已弃用原因与替代方案

SSE传输被标记为“已弃用”的主要原因包括：

1. **功能局限性**：SSE仅支持服务器到客户端的单向通信，无法满足双向交互需求。
2. **兼容性问题**：部分客户端或代理服务器对SSE的支持不完善，可能导致连接中断或数据丢失。
3. **维护成本高**：相比HTTP和STDIO，SSE的实现和维护更为复杂，且使用率较低。
4. **性能瓶颈**：在高并发场景下，SSE的长连接可能消耗大量服务器资源。

替代方案包括：
- **HTTP传输**：推荐使用，支持流式响应和双向通信，兼容性好。
- **STDIO传输**：适用于本地进程间通信，性能高，配置简单。

**Section sources**
- [graphiti_mcp_server.py](file://mcp_server/src/graphiti_mcp_server.py#L784)
- [schema.py](file://mcp_server/src/config/schema.py#L81)
- [README.md](file://mcp_server/README.md#L164)

## SSE流式响应实现机制

SSE流式响应的实现机制包括事件格式、心跳机制和连接保持策略。

### 事件格式
SSE事件以文本格式发送，每个事件由一个或多个字段组成，如`data`、`event`、`id`等。服务器通过`Content-Type: text/event-stream`告知客户端使用SSE协议。

### 心跳机制
为了保持连接活跃，服务器会定期发送心跳事件（通常为空数据或注释），防止代理服务器或客户端因超时而关闭连接。

### 连接保持策略
服务器通过长轮询方式维持连接，客户端在接收到事件后不会立即关闭连接，而是等待下一个事件。若连接中断，客户端需重新建立连接并从上次中断处恢复。

**Section sources**
- [graphiti_mcp_server.py](file://mcp_server/src/graphiti_mcp_server.py#L917-L922)
- [test_mcp_transports.py](file://mcp_server/tests/test_mcp_transports.py#L27-L35)

## 客户端消费SSE流示例

以下是一个使用Python消费SSE流的示例代码：

```python
import asyncio
from mcp.client.sse import sse_client

async def consume_sse_stream():
    async with sse_client("http://localhost:8000/sse") as (read_stream, write_stream):
        session = ClientSession(read_stream, write_stream)
        await session.initialize()
        # 处理流式响应
        async for message in read_stream:
            print(message)

asyncio.run(consume_sse_stream())
```

该示例使用`mcp.client.sse`模块的`sse_client`函数连接到SSE端点，并通过异步循环消费服务器推送的事件流。

**Section sources**
- [test_mcp_transports.py](file://mcp_server/tests/test_mcp_transports.py#L27-L35)

## 现代应用架构中的推荐方案

在现代应用架构中，推荐使用HTTP或STDIO替代SSE，原因如下：

1. **HTTP传输**：支持流式响应和双向通信，兼容性好，易于调试和监控。
2. **STDIO传输**：适用于本地进程间通信，性能高，配置简单，适合CLI工具或本地代理。
3. **更好的生态系统支持**：HTTP和STDIO在各种编程语言和框架中都有广泛支持，而SSE的支持相对有限。
4. **更低的维护成本**：HTTP和STDIO的实现和维护更为简单，减少了潜在的兼容性问题。

**Section sources**
- [README.md](file://mcp_server/README.md#L164)
- [graphiti_mcp_server.py](file://mcp_server/src/graphiti_mcp_server.py#L923-L945)

## 迁移指南

从SSE迁移到推荐的传输协议的步骤如下：

1. **更新配置文件**：将`config.yaml`中的`server.transport`字段从`sse`改为`http`或`stdio`。
2. **更新客户端配置**：根据新的传输协议更新客户端的连接配置。
3. **测试连接**：确保客户端能够成功连接到服务器并正常通信。
4. **监控性能**：观察迁移后的性能表现，确保没有性能下降或连接问题。
5. **文档更新**：更新相关文档，说明迁移后的配置和使用方法。

通过以上步骤，可以顺利从SSE迁移到更现代、更可靠的传输协议。

**Section sources**
- [config.yaml](file://mcp_server/config/config.yaml#L9)
- [README.md](file://mcp_server/README.md#L527-L537)
- [graphiti_mcp_server.py](file://mcp_server/src/graphiti_mcp_server.py#L914-L949)