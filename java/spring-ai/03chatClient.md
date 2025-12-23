# ChatClient

ChatClient提供了一个流畅的API，用于与AI进行通信。它同时支持同步和流媒体编程模型。

## 创建ChatClient

**自动注入ChatClient.Builder**

```java
@Test
public void testChatClientBuilder(@Autowired ChatClient.Builder builder) {
  ChatClient chatClient = builder.build();

  String content = chatClient.prompt()
    .user("Hello") // 用户提示词
    .call()
    .content();
  System.out.println(content);
}
```

**通过chatModel创建**

> [!NOTE]
>
> 可以适配多模型场景

```java
@Test
public void testChatClient(@Autowired OllamaChatModel ollamaChatModel) {

  ChatClient chatClient = ChatClient.builder(ollamaChatModel).build();

  String content = chatClient.prompt()
    .user("Hello") // 用户提示词
    .call()
    .content();
  System.out.println(content);
}
```

```java
@Test
public void testPrompt3(@Autowired OllamaChatModel ollamaChatModel) {

  PromptTemplate promptTemplate = PromptTemplate.builder()
    .renderer(StTemplateRenderer.builder().startDelimiterToken('<').endDelimiterToken('>').build())
    .template("""
              你好 <addr>有什么吃的
              """)
              .build();


              ChatClient chatClient = ChatClient.builder(ollamaChatModel)
              .defaultSystem(ds -> ds
                             .text(new ClassPathResource("system-prompt.st"))
                             .param("lang", "繁体中文")
                            )
              .build();

  String content = chatClient.prompt()
    .user(promptTemplate.render(Map.of("addr", "北京"))) // 用户提示词
    .call()
    .content();
  System.out.println(content);
}
```

