# Structured Output

Spring AI 结构化输出转换器（**Structured Output Converter**）帮助将 LLM 输出转为结构化格式

在 LLM 调用前，转换器向提示词追格式说明

LLM 调用后，转换器（**Converter**）将模型的原始文本输出转换为结构化类型实例。该转换过程包括解析原始文本输出，并将其映射 为JSON、XML 等

![Structured Output Converter Architecture](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1766923343-8c0e1a.jpg)

## API

```java
public interface StructuredOutputConverter<T> extends Converter<String, T>, FormatProvider {

}

public interface FormatProvider {
	String getFormat();
}
```

![Structured Output API](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1766923569-89c36b.jpg)

### Available Converters

![Structured Output Class Hierarchy](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1766923655-f9e3be.jpg)



## Demo

**responseEntity**

```java
@Test
public void testConvertor(@Autowired OllamaChatModel ollamaChatModel) {
  ChatClient chatClient = ChatClient.builder(ollamaChatModel)
    .defaultAdvisors(SimpleLoggerAdvisor.builder().order(1).build())
    .build();

  ResponseEntity<ChatResponse, ActorMovie> res = chatClient.prompt()
    .user("说出1个中国有名的演员以及其代表作") // 用户提示词
    .advisors(advisorSpec -> advisorSpec.param(ChatMemory.CONVERSATION_ID, "1"))
    .call()
    .responseEntity(ActorMovie.class);  //可以获取到转换后的entity 和 response
  /**
         说出1个中国有名的演员以及其代表作
         Your response should be in JSON format.
         Do not include any explanations, only provide a RFC8259 compliant JSON response following this format without deviation.
         Do not include markdown code blocks in your response.
         Remove the ```json markdown from the output.
         Here is the JSON Schema instance your output must adhere to:
         ```{
         \"$schema\" : \"https://json-schema.org/draft/2020-12/schema\",
         \"type\" : \"object\",
         \"properties\" : {
         \"movie\" : {
         \"type\" : \"string\"
         },
         \"name\" : {
         \"type\" : \"string\"
         }
         },
         \"required\" : [ \"movie\", \"name\" ],
         \"additionalProperties\" : false
         }```

         */


  //{"name":"Jackie Chan","movie":"Drunken Master"}
  System.out.println(res.getResponse().getResult().getOutput().getText());
  //ActorMovie[name=Jackie Chan, movie=Drunken Master]
  System.out.println(res.getEntity());

}
```

**entity**

```java
 @Test
    public void testConvertor2(@Autowired OllamaChatModel ollamaChatModel) {
        ChatClient chatClient = ChatClient.builder(ollamaChatModel)
                .defaultAdvisors(SimpleLoggerAdvisor.builder().order(Advisor.LOWEST_PRECEDENCE).build())
                .build();

        ActorMovie res = chatClient.prompt()
                .user("说出1个中国有名的演员以及其代表作") 
                .advisors(advisorSpec -> advisorSpec.param(ChatMemory.CONVERSATION_ID, "1"))
                .call()
                .entity(ActorMovie.class);  //只需要entity
        System.out.println(res);
    }
```

**泛型**

```java
@Test
public void testConvertor3(@Autowired OllamaChatModel ollamaChatModel) {
  ChatClient chatClient = ChatClient.builder(ollamaChatModel)
    .defaultAdvisors(SimpleLoggerAdvisor.builder().order(1).build())
    .build();

  List<ActorMovie> res = chatClient.prompt()
    .user("说出5个中国有名的演员以及其代表作") // 用户提示词
    .advisors(advisorSpec -> advisorSpec.param(ChatMemory.CONVERSATION_ID, "1"))
    .call()
    .entity(new ParameterizedTypeReference<List<ActorMovie>>(){});
  System.out.println(res);
}
```

