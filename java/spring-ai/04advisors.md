# Advisors

Spring AI Advisor API 为拦截、修改和增强 Spring 应用中的 AI 交互提供了灵活强大的方式。

API由用于非流式的CallAdvisor和CallAdvisorChain以及流式的StreamAdvisor和StreamAdvisorChain组成。

![Advisors API Classes](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1766675926-8df15f.jpg)

## 执行流程

![Advisors Streaming vs Non-Streaming Flow](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1766676143-ab3e36.jpg)

- order值低的先处理request,后处理response
- 如果order值相同，执行顺序不确定
- 链中的每个Adviser都会处理请求，并可能对其进行修改。或者，它可以选择通过不调用`callAdvisorChain.nextCall`来阻止请求。自己返回ChatClientResponse



## API

**顶层接口**

```java
public interface Advisor extends Ordered {

	/**
	 * Useful constant for the default Chat Memory precedence order. Ensures this order
	 * has lower priority (e.g. precedences) than the Spring AI internal advisors. It
	 * leaves room (1000 slots) for the user to plug in their own advisors with higher
	 * priority.
	 * chatMemory默认优先级 ， 确保尽量优先被执行
	 */
	int DEFAULT_CHAT_MEMORY_PRECEDENCE_ORDER = Ordered.HIGHEST_PRECEDENCE + 1000;


	String getName();

}
```

**非流式advisor**

```java
public interface CallAdvisor extends Advisor {

	ChatClientResponse adviseCall(ChatClientRequest chatClientRequest, CallAdvisorChain callAdvisorChain);

}
```

**流式advisor**

```java
public interface StreamAdvisor extends Advisor {

	Flux<ChatClientResponse> adviseStream(ChatClientRequest chatClientRequest, StreamAdvisorChain streamAdvisorChain);

}
```

**BaseAdvisor , 同时支持流式与非流式**

```java
public interface BaseAdvisor extends CallAdvisor, StreamAdvisor {

	Scheduler DEFAULT_SCHEDULER = Schedulers.boundedElastic();

	@Override
	default ChatClientResponse adviseCall(ChatClientRequest chatClientRequest, CallAdvisorChain callAdvisorChain) {
		Assert.notNull(chatClientRequest, "chatClientRequest cannot be null");
		Assert.notNull(callAdvisorChain, "callAdvisorChain cannot be null");

		ChatClientRequest processedChatClientRequest = before(chatClientRequest, callAdvisorChain);
		ChatClientResponse chatClientResponse = callAdvisorChain.nextCall(processedChatClientRequest);
		return after(chatClientResponse, callAdvisorChain);
	}

	@Override
	default Flux<ChatClientResponse> adviseStream(ChatClientRequest chatClientRequest,
			StreamAdvisorChain streamAdvisorChain) {
		Assert.notNull(chatClientRequest, "chatClientRequest cannot be null");
		Assert.notNull(streamAdvisorChain, "streamAdvisorChain cannot be null");
		Assert.notNull(getScheduler(), "scheduler cannot be null");

		Flux<ChatClientResponse> chatClientResponseFlux = Mono.just(chatClientRequest)
			.publishOn(getScheduler())
			.map(request -> this.before(request, streamAdvisorChain))
			.flatMapMany(streamAdvisorChain::nextStream);

		return chatClientResponseFlux.map(response -> {
			if (AdvisorUtils.onFinishReason().test(response)) {
				response = after(response, streamAdvisorChain);
			}
			return response;
		}).onErrorResume(error -> Flux.error(new IllegalStateException("Stream processing failed", error)));
	}

	@Override
	default String getName() {
		return this.getClass().getSimpleName();
	}

	/**
	 * Logic to be executed before the rest of the advisor chain is called.
	 */
	ChatClientRequest before(ChatClientRequest chatClientRequest, AdvisorChain advisorChain);

	/**
	 * Logic to be executed after the rest of the advisor chain is called.
	 */
	ChatClientResponse after(ChatClientResponse chatClientResponse, AdvisorChain advisorChain);

	/**
	 * Scheduler used for processing the advisor logic when streaming.
	 */
	default Scheduler getScheduler() {
		return DEFAULT_SCHEDULER;
	}

}
```



## ChatClient配置Advisor

```java
var chatClient = ChatClient.builder(chatModel)
    .defaultAdvisors(
			MessageChatMemoryAdvisor.builder(chatMemory).build(), // chat-memory advisor
			QuestionAnswerAdvisor.builder((vectorStore).builder() // RAG advisor
		)
	)
    .build();

var conversationId = "678";

String response = this.chatClient.prompt()
	// 运行时设置 advisor 参数
    .advisors(advisor -> advisor.param(ChatMemory.CONVERSATION_ID, conversationId))
    .user(userText)
    .call()
	.content();
```



## 自定义Advisor

```java
public class ReReadingAdvisor implements BaseAdvisor {

	private static final String DEFAULT_RE2_ADVISE_TEMPLATE = """
			{re2_input_query}
			Read the question again: {re2_input_query}
			""";

	private final String re2AdviseTemplate;

	private int order = 0;

	public ReReadingAdvisor() {
		this(DEFAULT_RE2_ADVISE_TEMPLATE);
	}

	public ReReadingAdvisor(String re2AdviseTemplate) {
		this.re2AdviseTemplate = re2AdviseTemplate;
	}

	@Override
	public ChatClientRequest before(ChatClientRequest chatClientRequest, AdvisorChain advisorChain) {
		String augmentedUserText = PromptTemplate.builder()
			.template(this.re2AdviseTemplate)
			.variables(Map.of("re2_input_query", chatClientRequest.prompt().getUserMessage().getText()))
			.build()
			.render();

		return chatClientRequest.mutate()
			.prompt(chatClientRequest.prompt().augmentUserMessage(augmentedUserText))
			.build();
	}

	@Override
	public ChatClientResponse after(ChatClientResponse chatClientResponse, AdvisorChain advisorChain) {
		return chatClientResponse;
	}

	@Override
	public int getOrder() {
		return this.order;
	}

	public ReReadingAdvisor withOrder(int order) {
		this.order = order;
		return this;
	}

}
```



