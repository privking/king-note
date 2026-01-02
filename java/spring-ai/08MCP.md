# MCP(Model Context Protocol)

## MCP Client

负责建立和管理与MCP服务器的连接。它实现了协议的客户端

![Java MCP Client Architecture](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1767358958-115e81.jpg)

## MCP Server

客户端提供工具、资源和功能

![Java MCP Server Architecture](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1767359001-b84b93.jpg)



## Spring AI MCP 集成

### Client Starters

- `spring-ai-starter-mcp-client` - Core starter providing STDIO and HTTP-based SSE support
- `spring-ai-starter-mcp-client-webflux` - WebFlux-based SSE transport implementation

标准启动器通过STDIO（进程内）和/或SSE（远程）传输同时连接到一个或多个MCP服务器。SSE连接使用基于httpclient的传输实现。

每个到MCP服务器的连接都会创建一个新的MCP客户端实例。

您可以选择同步或异步MCP客户端（注意：不能混合同步和异步客户端）。

对于生产部署，我们建议使用基于webflux的SSE连接。



**配置参数**

**spring.ai.mcp.client**

| 参数                     | 描述                                       | 默认值               |
| ------------------------ | ------------------------------------------ | -------------------- |
| enable                   | 是否启用MCP client                         | True                 |
| name                     | mcp client实例名称                         | spring-ai-mcp-client |
| version                  | mcp client实例版本                         | 1.0.0                |
| initialized              | 是否创建时初始化                           | true                 |
| request-timeout          | 客户端请求超时时间                         | 20s                  |
| type                     | 客户端类型（SYNC或ASYNC）。不支持混合      | SYNC                 |
| root-change-notification | 根上下文变更通知                           | true                 |
| toolcallback.enabled     | MCP工具回调与Spring AI的工具执行框架的集成 | true                 |



### Client Starter Stdio

**spring.ai.mcp.client.stdio**

| 参数                       | 描述                    | 默认值 |
| -------------------------- | ----------------------- | ------ |
| servers-configuration      | JSON格式的MCP服务器配置 | -      |
| connections                | stdio 连接配置          |        |
| connections.[name].command | 命令                    |        |
| connections.[name].args    | 参数                    |        |
| connections.[name].env     | 环境变量                |        |

connections在yaml中直接配置

```yaml
spring:
  ai:
    mcp:
      client:
        stdio:
          root-change-notification: true
          connections:
            server1:
              command: /path/to/server
              args:
                - --port=8080
                - --mode=production
              env:
                API_KEY: your-api-key
                DEBUG: "true"
```

servers-configuration 配置文件方式

```yaml
spring:
  ai:
    mcp:
      client:
        stdio:
          servers-configuration: classpath:mcp-servers.json
```

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/username/Desktop",
        "/Users/username/Downloads"
      ]
    }
  }
}
```



### Client Starter Sse

**spring.ai.mcp.client.sse**

| 参数                            | 描述        | 默认值 |
| ------------------------------- | ----------- | ------ |
| connections                     | SSE连接配置 | -      |
| connections.[name].url          |             |        |
| connections.[name].sse-endpoint |             |        |

```yaml
spring:
  ai:
    mcp:
      client:
        sse:
          connections:
            server1:
              url: http://localhost:8080
            server2:
              url: http://otherserver:8081
              sse-endpoint: /custom-sse
```



**自动注入**

```java
@Autowired
private List<McpSyncClient> mcpSyncClients;  // For sync client

// OR

@Autowired
private List<McpAsyncClient> mcpAsyncClients;  // For async client
```



当工具回调被启用时（spring.ai.mcp.client.toolcallback.enable），所有MCP客户端的注册MCP工具将作为ToolCallbackProvider实例提供

```java
@Autowired
private SyncMcpToolCallbackProvider toolCallbackProvider;
ToolCallback[] toolCallbacks = toolCallbackProvider.getToolCallbacks();
```





### Server Starters

- `spring-ai-starter-mcp-server` - Core server with STDIO transport support
- `spring-ai-starter-mcp-server-webmvc` - Spring MVC-based SSE transport implementation
- `spring-ai-starter-mcp-server-webflux` - WebFlux-based SSE transport implementation



**配置参数**

**spring.ai.mcp.server**

| 参数                         | 描述                                                         | 默认值       |
| ---------------------------- | ------------------------------------------------------------ | ------------ |
| enabled                      | 是否启用                                                     | true         |
| stdio                        | 是否启用stdio                                                | False        |
| name                         | server名称                                                   | mcp-server   |
| version                      | 版本                                                         | 1.0.0        |
| instructions                 | 可选说明                                                     | null         |
| type                         | Server type (SYNC/ASYNC)                                     | SYNC         |
| capabilities.resource        | 能否让模型访问/理解外部资源                                  | true         |
| capabilities.tool            | 能否调用注册的工具 / 函数                                    | true         |
| capabilities.prompt          | 能否使用结构化 Prompt / PromptTemplate                       | true         |
| capabilities.completion      | 能否进行“纯补全文本”模式                                     | true         |
| resource-change-notification |                                                              | true         |
| prompt-change-notification   |                                                              | true         |
| tool-change-notification     |                                                              | true         |
| tool-response-mime-type      | 响应mime , example image/png                                 | -            |
| sse-message-endpoint         |                                                              | /mcp/message |
| sse-endpoint                 |                                                              | /sse         |
| base-url                     | `base-url=/api/v1` means that the client should access the sse endpoint at `/api/v1` + `sse-endpoint` and the message endpoint is `/api/v1` + `sse-message-endpoint` | -            |
| request-timeout              |                                                              | 20 seconds   |
|                              |                                                              |              |



### Server Starter Tools

```java
@Bean
public ToolCallbackProvider myTools(...) {
    List<ToolCallback> tools = ...
    return ToolCallbackProvider.from(tools);
}

// or using the low-level API:

@Bean
public List<McpServerFeatures.SyncToolSpecification> myTools(...) {
    List<McpServerFeatures.SyncToolSpecification> tools = ...
    return tools;
}
```





### Server Starter Resource

```java
@Bean
public List<McpServerFeatures.SyncResourceSpecification> myResources(...) {
    var systemInfoResource = new McpSchema.Resource(...);
    var resourceSpecification = new McpServerFeatures.SyncResourceSpecification(systemInfoResource, (exchange, request) -> {
        try {
            var systemInfo = Map.of(...);
            String jsonContent = new ObjectMapper().writeValueAsString(systemInfo);
            return new McpSchema.ReadResourceResult(
                    List.of(new McpSchema.TextResourceContents(request.uri(), "application/json", jsonContent)));
        }
        catch (Exception e) {
            throw new RuntimeException("Failed to generate system info", e);
        }
    });

    return List.of(resourceSpecification);
}
```



### Server Starter Management

```java
@Bean
public List<McpServerFeatures.SyncPromptSpecification> myPrompts() {
    var prompt = new McpSchema.Prompt("greeting", "A friendly greeting prompt",
        List.of(new McpSchema.PromptArgument("name", "The name to greet", true)));

    var promptSpecification = new McpServerFeatures.SyncPromptSpecification(prompt, (exchange, getPromptRequest) -> {
        String nameArgument = (String) getPromptRequest.arguments().get("name");
        if (nameArgument == null) { nameArgument = "friend"; }
        var userMessage = new PromptMessage(Role.USER, new TextContent("Hello " + nameArgument + "! How can I assist you today?"));
        return new GetPromptResult("A personalized greeting message", List.of(userMessage));
    });

    return List.of(promptSpecification);
}
```



### Server Starter Completion

```java
@Bean
public List<McpServerFeatures.SyncCompletionSpecification> myCompletions() {
    var completion = new McpServerFeatures.SyncCompletionSpecification(
        "code-completion",
        "Provides code completion suggestions",
        (exchange, request) -> {
            // Implementation that returns completion suggestions
            return new McpSchema.CompletionResult(List.of(
                new McpSchema.Completion("suggestion1", "First suggestion"),
                new McpSchema.Completion("suggestion2", "Second suggestion")
            ));
        }
    );

    return List.of(completion);
}
```



## Demo

stdio

```yaml
# Using spring-ai-starter-mcp-server
spring:
  ai:
    mcp:
      server:
        name: stdio-mcp-server
        version: 1.0.0
        type: SYNC
```

webmvc

```yaml
# Using spring-ai-starter-mcp-server-webmvc
spring:
  ai:
    mcp:
      server:
        name: webmvc-mcp-server
        version: 1.0.0
        type: SYNC
        instructions: "This server provides weather information tools and resources"
        sse-message-endpoint: /mcp/messages
        capabilities:
          tool: true
          resource: true
          prompt: true
          completion: true
```

webflux

```yaml
# Using spring-ai-starter-mcp-server-webflux
spring:
  ai:
    mcp:
      server:
        name: webflux-mcp-server
        version: 1.0.0
        type: ASYNC  # Recommended for reactive applications
        instructions: "This reactive server provides weather information tools and resources"
        sse-message-endpoint: /mcp/messages
        capabilities:
          tool: true
          resource: true
          prompt: true
          completion: true
```



server

```java
@Service
public class WeatherService {

    @Tool(description = "Get weather information by city name")
    public String getWeather(String cityName) {
        // Implementation
    }
}

@SpringBootApplication
public class McpServerApplication {

    private static final Logger logger = LoggerFactory.getLogger(McpServerApplication.class);

    public static void main(String[] args) {
        SpringApplication.run(McpServerApplication.class, args);
    }

	@Bean
	public ToolCallbackProvider weatherTools(WeatherService weatherService) {
		return MethodToolCallbackProvider.builder().toolObjects(weatherService).build();
	}
}
```



