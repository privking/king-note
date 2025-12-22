# ChatModels

## ChatModelAPI

Spring AI Chat Model API被设计成一个简单而便携的界面，用于与各种AI模型进行交互，允许开发人员以最少的代码更改在不同模型之间切换。这种设计符合Spring的模块化和可互换性理念。

此外，在用于输入封装的Prompt和用于输出处理的ChatResponse等配套类的帮助下，聊天模型API将通信与人工智能模型统一。它管理请求准备和响应解析的复杂性，提供直接和简化的API交互。

### ChatModel

```java
public interface ChatModel extends Model<Prompt, ChatResponse>, StreamingChatModel {
    default String call(String message) {
        Prompt prompt = new Prompt(new UserMessage(message));
        Generation generation = this.call(prompt).getResult();
        return generation != null ? generation.getOutput().getText() : "";
    }

    default String call(Message... messages) {
        Prompt prompt = new Prompt(Arrays.asList(messages));
        Generation generation = this.call(prompt).getResult();
        return generation != null ? generation.getOutput().getText() : "";
    }

    ChatResponse call(Prompt prompt);

    default ChatOptions getDefaultOptions() {
        return ChatOptions.builder().build();
    }

    default Flux<ChatResponse> stream(Prompt prompt) {
        throw new UnsupportedOperationException("streaming is not supported");
    }
}
```

### Prompt

封装了消息对象和可选模型请求选项的列表

```java
public class Prompt implements ModelRequest<List<Message>> {

    private final List<Message> messages;

    private ChatOptions modelOptions;

	@Override
	public ChatOptions getOptions() {...}

	@Override
	public List<Message> getInstructions() {...}

    // constructors and utility methods omitted
}
```

### Message

```java
public interface Content {

	String getText();

	Map<String, Object> getMetadata();
}

public interface Message extends Content {

	MessageType getMessageType();
}
```

```java
public enum MessageType {
    USER("user"),
    ASSISTANT("assistant"),
    SYSTEM("system"),
    TOOL("tool");
}
```



![Spring AI Message API](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1766418337-6baeab.jpg)



###  Chat Options

用于定义一些可以传递给AI模型的可移植选项

允许在启动时设置默认配置，同时还提供了在每个请求的基础上覆盖这些设置

```java
public interface ChatOptions extends ModelOptions {

	String getModel();
	Float getFrequencyPenalty();
	Integer getMaxTokens();
	Float getPresencePenalty();
	List<String> getStopSequences();
	Float getTemperature();
	Integer getTopK();
	Float getTopP();
	ChatOptions copy();

}
```

![chat options flow](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/1766418690-0c8074.jpg)

****

OllamaChatOptions  参数解释

```java
public class OllamaChatOptions implements ToolCallingChatOptions {
    // 内部常量：定义了不直接作为 'options' 参数传递给 Ollama API 的字段
    private static final List<String> NON_SUPPORTED_FIELDS = List.of("model", "format", "keep_alive", "truncate");

    /** 运行参数设置 (Runner parameters) **/

    @JsonProperty("numa")
    private Boolean useNUMA; // 是否使用 NUMA 内存优化（适用于多 CPU 插槽的服务器）

    @JsonProperty("num_ctx")
    private Integer numCtx; // 设置上下文窗口大小（Tokens 数量），默认为 2048

    @JsonProperty("num_batch")
    private Integer numBatch; // 设置提示词处理的批处理大小

    @JsonProperty("num_gpu")
    private Integer numGPU; // 要映射到 GPU 的层数。在 macOS 上通常为 1 (启用 Metal)，0 表示仅 CPU

    @JsonProperty("main_gpu")
    private Integer mainGPU; // 指定主 GPU 的索引（多 GPU 环境下使用）

    @JsonProperty("low_vram")
    private Boolean lowVRAM; // 是否启用低显存模式

    @JsonProperty("f16_kv")
    private Boolean f16KV; // 是否在 KV 缓存中使用 FP16 精度

    @JsonProperty("logits_all")
    private Boolean logitsAll; // 是否返回所有词元的 logits , 大模型预测下一个字（Token）时，它会给词表里的每一个字都打一个分,模型在生成文本时会返回词表中所有可能的 Token 的原始得分，而不仅仅是最终选中的那个字

    @JsonProperty("vocab_only")
    private Boolean vocabOnly; // 是否仅加载词汇表，而不加载权重

    @JsonProperty("use_mmap")
    private Boolean useMMap; // 是否使用内存映射（mmap）加载模型，设为 false 加载速度慢但能减少内存压力

    @JsonProperty("use_mlock")
    private Boolean useMLock; // 是否锁定内存，防止模型被交换到磁盘（Swap）

    @JsonProperty("num_thread")
    private Integer numThread; // 设置计算时使用的 CPU 线程数（建议设置为物理核心数）

    /** 文本生成预测参数 (Predict parameters) **/

    @JsonProperty("num_keep")
    private Integer numKeep; // 在上下文截断时保留的初始词元数量

    @JsonProperty("seed")
    private Integer seed; // 设置随机种子。固定种子可使模型输出结果具有确定性

    @JsonProperty("num_predict")
    private Integer numPredict; // 最大生成的 Token 数量（-1 表示无穷，-2 表示填满上下文）

    @JsonProperty("top_k")
    private Integer topK; // 采样范围：仅从概率最高的 K 个词中选择。减小此值可使回答更集中

    @JsonProperty("top_p")
    private Double topP; // 核采样：累积概率阈值。1.0 表示禁用

    @JsonProperty("min_p")
    private Double minP; // 最小概率采样：相对于最可能词元的最小概率阈值

    @JsonProperty("tfs_z")
    private Float tfsZ; // 尾部自由采样 (Tail Free Sampling) 的参数 z

    @JsonProperty("typical_p")
    private Float typicalP; // 典型采样阈值

    @JsonProperty("repeat_last_n")
    private Integer repeatLastN; // 设置回溯多远来检查重复（0=禁用，-1=num_ctx）

    @JsonProperty("temperature")
    private Double temperature; // 温度：越高越发散（创造性），越低越保守（确定性）

    @JsonProperty("repeat_penalty")
    private Double repeatPenalty; // 重复惩罚：大于 1.0 的值会惩罚重复的内容

    @JsonProperty("presence_penalty")
    private Double presencePenalty; // 存在惩罚：基于是否出现过惩罚词元

    @JsonProperty("frequency_penalty")
    private Double frequencyPenalty; // 频率惩罚：基于词元出现频率惩罚

    @JsonProperty("mirostat")
    private Integer mirostat; // 启用 Mirostat 采样以控制困惑度（0=禁用, 1=Mirostat, 2=Mirostat 2.0）

    @JsonProperty("mirostat_tau")
    private Float mirostatTau; // Mirostat 目标熵

    @JsonProperty("mirostat_eta")
    private Float mirostatEta; // Mirostat 学习率

    @JsonProperty("penalize_newline")
    private Boolean penalizeNewline; // 是否惩罚换行符

    @JsonProperty("stop")
    private List<String> stop; // 停止序列：模型遇到这些字符串时将停止生成

    /** 模型与系统级参数 **/

    @JsonProperty("model")
    private String model; // 要使用的模型名称（如 "llama3"）

    @JsonProperty("format")
    private Object format; // 指定输出格式（如 "json" 或结构化 Schema）

    @JsonProperty("keep_alive")
    private String keepAlive; // 模型在内存中保留的时长（例如 "5m" 或 "10s"）

    @JsonProperty("truncate")
    private Boolean truncate; // 是否在请求时强制截断输入以适应上下文窗口

    @JsonProperty("think")
    private ThinkOption thinkOption; // 针对推理模型（如 DeepSeek-R1）的“思考”过程输出配置

    /** 工具调用相关 (Spring AI 扩展) **/

    @JsonIgnore
    private Boolean internalToolExecutionEnabled; // 是否允许 Spring AI 在内部自动执行匹配的工具/函数

    @JsonIgnore
    private List<ToolCallback> toolCallbacks; // 注册的函数回调对象列表

    @JsonIgnore
    private Set<String> toolNames; // 启用的工具函数名称集合

    @JsonIgnore
    private Map<String, Object> toolContext; // 传递给工具执行时的上下文上下文参数
}

```



### ChatResponse

```java
public class ChatResponse implements ModelResponse<Generation> {

    private final ChatResponseMetadata chatResponseMetadata;
	private final List<Generation> generations;

	@Override
	public ChatResponseMetadata getMetadata() {...}

    @Override
	public List<Generation> getResults() {...}

    // other methods omitted
}
```



AssistantMessage 也继承至Message ,messageType =  MessageType.ASSISTANT

```java
public class Generation implements ModelResult<AssistantMessage> {

	private final AssistantMessage assistantMessage;
	private ChatGenerationMetadata chatGenerationMetadata;

	@Override
	public AssistantMessage getOutput() {...}

	@Override
	public ChatGenerationMetadata getMetadata() {...}

    // other methods omitted
}
```



## Demo

```java
@SpringBootTest(classes = Application.class)
public class ChatModelTest {

    @Test
    public void test(@Autowired OllamaChatModel ollamaChatModel) {
        String called = ollamaChatModel.call("你是谁");
        System.out.println(called);
    }


    @Test
    public void test2(@Autowired OllamaChatModel ollamaChatModel) {
        Flux<String> res = ollamaChatModel.stream("写一篇1000字作文");
        res.toIterable().forEach(System.out::println);
    }

  	/**
  	多模态
  	**/
    @Test
    public void testMultimodality(@Autowired OllamaChatModel ollamaChatModel) {
        var imageResource = new ClassPathResource("2.png");

        OllamaChatOptions ollamaOptions = OllamaChatOptions.builder()
                .model("gemma3:4b")
                .build();

        Media media = new Media(MimeTypeUtils.IMAGE_PNG, imageResource);

        ChatResponse response = ollamaChatModel.call(
                new Prompt(
                        UserMessage.builder().media(media)
                                .text("""
                                        描述这幅图的内容
                                       """).build(),
                        ollamaOptions
                )
        );

        System.out.println(response.getResult().getOutput().getText());
    }
}
```