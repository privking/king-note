# ChatMemory

大语言模型（LLM）本质上是无状态的，这意味着它们不会保留历史交互信息。当需要跨多轮交互保持上下文时，这一特性会带来局限。为此，Spring AI 提供了聊天记忆功能

```xml
<dependency>
  <groupId>org.springframework.ai</groupId>
  <artifactId>spring-ai-autoconfigure-model-chat-memory</artifactId>
</dependency>
```



## ChatMemory

`ChatMemory` 抽象层支持实现多种记忆类型以满足不同场景需求。消息的底层存储由 `ChatMemoryRepository` 处理

Spring AI 自动配置 `ChatMemory` Bean 供直接使用。默认采用内存存储（`InMemoryChatMemoryRepository`）及 `MessageWindowChatMemory` 

```java
public interface ChatMemory {
    String DEFAULT_CONVERSATION_ID = "default";
    String CONVERSATION_ID = "chat_memory_conversation_id";

    default void add(String conversationId, Message message) {
        Assert.hasText(conversationId, "conversationId cannot be null or empty");
        Assert.notNull(message, "message cannot be null");
        this.add(conversationId, List.of(message));
    }

    void add(String conversationId, List<Message> messages);

    List<Message> get(String conversationId);

    void clear(String conversationId);
}
```



### MessageWindowChatMemory

chatMemory的实现，维护固定容量的消息窗口（默认 20 条）。当消息超限时，自动移除较早的对话消息

```java
MessageWindowChatMemory memory = MessageWindowChatMemory.builder()
    .maxMessages(10)
    .build();
```



## ChatMemoryRepository

Spring AI 通过 `ChatMemoryRepository` 抽象层实现聊天记忆存储。

### InMemoryChatMemoryRepository 

基于 `ConcurrentHashMap` 实现内存存储。

### JdbcChatMemoryRepository

是内置的 JDBC 实现，支持多种关系型数据库，适用于需要持久化存储聊天记忆的场景

```xml
<dependency>
  <groupId>org.springframework.ai</groupId>
  <artifactId>spring-ai-starter-model-chat-memory-repository-jdbc</artifactId>
</dependency>
<!--jdbc-->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>


<!--mysql驱动-->
<dependency>
  <groupId>com.mysql</groupId>
  <artifactId>mysql-connector-j</artifactId>
  <version>${mysql-version}</version>
</dependency>
```

```yaml
spring:
  datasource:
    username: root
    password: xxxxxx
    url: jdbc:mysql://localhost:3306/spring_ai?characterEncoding=utf8&useSSL=false
    driver-class-name: com.mysql.cj.jdbc.Driver
```





**手动创建**

引入后可以不用手动创建，可以autowired ChatMemory

```java
ChatMemoryRepository chatMemoryRepository = JdbcChatMemoryRepository.builder()
    .jdbcTemplate(jdbcTemplate)
    .dialect(new PostgresChatMemoryDialect())
    .build();

ChatMemory chatMemory = MessageWindowChatMemory.builder()
    .chatMemoryRepository(chatMemoryRepository)
    .maxMessages(10)
    .build();
```



## ChatClient使用ChatMemory

Spring AI提供了一些内置的Adcisor

### MessageChatMemoryAdvisor

每次交互时从记忆库检索历史对话，以Message注入提示词

### PromptChatMemoryAdvisor

每次交互时从记忆库检索历史对话，以纯文本形式追加至系统（system）提示词

### VectorStoreChatMemoryAdvisor

每次交互时从向量存储检索历史对话，并以纯文本形式追加至系统（system）消息



### Demo

```java
ChatMemory chatMemory = MessageWindowChatMemory.builder().build();

ChatClient chatClient = ChatClient.builder(chatModel)
    .defaultAdvisors(MessageChatMemoryAdvisor.builder(chatMemory).build())
    .build();

String conversationId = "007";

chatClient.prompt()
    .user("Do I have license to code?")
    .advisors(a -> a.param(ChatMemory.CONVERSATION_ID, conversationId))
    .call()
    .content();
```



```java
@Test
    public void testAdviserJdbcMemory(@Autowired OllamaChatModel ollamaChatModel,
                                @Autowired ChatMemory chatMemory) {
        ChatClient chatClient = ChatClient.builder(ollamaChatModel)
                .defaultAdvisors(SimpleLoggerAdvisor.builder().order(1).build(),
                        PromptChatMemoryAdvisor.builder(chatMemory).build())
                .build();

        String content = chatClient.prompt()
                .user("我是张飞") // 用户提示词
                .advisors(advisorSpec -> advisorSpec.param(ChatMemory.CONVERSATION_ID,"1"))
                .call()
                .content();

        System.out.println(content);

        String content2 = chatClient.prompt()
                .user("我是谁") // 用户提示词
                .advisors(advisorSpec -> advisorSpec.param(ChatMemory.CONVERSATION_ID,"1"))
                .call()
                .content();
        System.out.println(content2);
    }

```

