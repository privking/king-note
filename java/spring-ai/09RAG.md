# RAG(Retrieval Augmented Generation)
检索增强生成（RAG）有助于克服大型语言模型在处理长篇内容、事实准确性和上下文感知方面的局限性。

Spring AI 通过提供模块化架构来支持 RAG，该架构允许自行构建自定义 RAG 流，或使用 Advisor API 来使用现成的 RAG 流。






## QuestionAnswerAdvisor
当用户向模型提出问题时，QuestionAnswerAdvisor会查询向量数据库，以获取与用户问题相关的文档。

向量数据库的响应被附加到用户文本中，为模型生成响应提供上下文。


### 依赖
```xml
<dependency>
   <groupId>org.springframework.ai</groupId>
   <artifactId>spring-ai-advisors-vector-store</artifactId>
</dependency>
```


```java
ChatResponse response = ChatClient.builder(chatModel)
        .build().prompt()
        .advisors(QuestionAnswerAdvisor.builder(vectorStore).build())
        .user(userText)
        .call()
        .chatResponse();

//指定 相似度 和 TOPK
var qaAdvisor = QuestionAnswerAdvisor.builder(vectorStore)
        .searchRequest(SearchRequest.builder().similarityThreshold(0.8d).topK(6).build())
        .build();

// 元数据动态过滤
String content = this.chatClient.prompt()
    .user("Please answer my question XYZ")
    .advisors(a -> a.param(QuestionAnswerAdvisor.FILTER_EXPRESSION, "type == 'Spring'"))
    .call()
    .content();
```

## RetrievalAugmentationAdvisor
为常见的RAG flow提供了开箱即用的实现
### 依赖
```xml
<dependency>
   <groupId>org.springframework.ai</groupId>
   <artifactId>spring-ai-advisors-vector-store</artifactId>
</dependency>

<dependency>
   <groupId>org.springframework.ai</groupId>
   <artifactId>spring-ai-rag</artifactId>
</dependency>
```

### native rag
```java
Advisor retrievalAugmentationAdvisor = RetrievalAugmentationAdvisor.builder()
        .documentRetriever(VectorStoreDocumentRetriever.builder()
                .similarityThreshold(0.50)
                .vectorStore(vectorStore)
                .build())
        .build();

String answer = chatClient.prompt()
        .advisors(retrievalAugmentationAdvisor)
        .user(question)
        .call()
        .content();


// 默认情况 不允许检索到的上下文为空。
// 当出现这种情况时，它会指示模型不要回答用户查询
// 可以设置allowEmptyContext 允许空上下文。
Advisor retrievalAugmentationAdvisor = RetrievalAugmentationAdvisor.builder()
        .documentRetriever(VectorStoreDocumentRetriever.builder()
                .similarityThreshold(0.50)
                .vectorStore(vectorStore)
                .build())
        .queryAugmenter(ContextualQueryAugmenter.builder()
                .allowEmptyContext(true)
                .build())
        .build();

// 设置默认元数据filter
DocumentRetriever retriever = VectorStoreDocumentRetriever.builder()
    .vectorStore(vectorStore)
    .similarityThreshold(0.73)
    .topK(5)
    .filterExpression(new FilterExpressionBuilder()
        .eq("genre", "fairytale")
        .build())
    .build();

// 设置动态元数据filter
String answer = chatClient.prompt()
        .advisors(retrievalAugmentationAdvisor)
        .advisors(a -> a.param(VectorStoreDocumentRetriever.FILTER_EXPRESSION, "type == 'Spring'"))
        .user(question)
        .call()
        .content();
```

### advanced rag
```java
Advisor retrievalAugmentationAdvisor = RetrievalAugmentationAdvisor.builder()
        // 指定transformer
        // pre-retrieval
        .queryTransformers(RewriteQueryTransformer.builder()
                .chatClientBuilder(chatClientBuilder.build().mutate())
                .build())
        .documentRetriever(VectorStoreDocumentRetriever.builder()
                .similarityThreshold(0.50)
                .vectorStore(vectorStore)
                .build())
        .build();

String answer = chatClient.prompt()
        .advisors(retrievalAugmentationAdvisor)
        .user(question)
        .call()
        .content();

// 还可以使用DocumentPostProcessor API对检索到的文档进行后处理，然后再将其传递给模型。例如，您可以使用此类接口根据检索到的文档与查询的相关性对其进行重新排序，删除不相关或冗余的文档，或者压缩每个文档的内容以减少噪声和冗余。
```

#### Query Transformation - CompressionQueryTransformer
利用大模型将对话历史和后续查询压缩成一个独立的查询
```java
Query query = Query.builder()
        .text("And what is its second largest city?")
        .history(new UserMessage("What is the capital of Denmark?"),
                new AssistantMessage("Copenhagen is the capital of Denmark."))
        .build();

QueryTransformer queryTransformer = CompressionQueryTransformer.builder()
        .chatClientBuilder(chatClientBuilder)
        .build();

Query transformedQuery = queryTransformer.transform(query);
```

#### Query Transformation - RewriteQueryTransformer
利用大型语言模型对用户查询进行重写，以便在查询向量存储或网络搜索引擎时提供更优结果。
当用户查询冗长、含糊不清或包含可能影响搜索结果质量的不相关信息时，此转换器有用。
```java
Query query = new Query("I'm studying machine learning. What is an LLM?");

QueryTransformer queryTransformer = RewriteQueryTransformer.builder()
        .chatClientBuilder(chatClientBuilder)
        .build();

Query transformedQuery = queryTransformer.transform(query);
```

#### Query Transformation - TranslationQueryTransformer
翻译， 如果查询本身已经是目标语言，则原样返回。如果查询的语言未知，也原样返回。
```java
Query query = new Query("Hvad er Danmarks hovedstad?");

QueryTransformer queryTransformer = TranslationQueryTransformer.builder()
        .chatClientBuilder(chatClientBuilder)
        .targetLanguage("english")
        .build();

Query transformedQuery = queryTransformer.transform(query);
```

#### Query Expansion - MultiQueryExpander
利用模型将查询扩展为多个语义不同的变体
```java
MultiQueryExpander queryExpander = MultiQueryExpander.builder()
    .chatClientBuilder(chatClientBuilder)
    .numberOfQueries(3)
    .includeOriginal(false) // 默认会包含原始查询
    .build();
List<Query> queries = queryExpander.expand(new Query("How to run a Spring Boot app?"));
```


#### Retrieval - VectorStoreDocumentRetriever
在向量存储中检索与输入查询语义相似的文档。
支持基于元数据、相似度阈值和前k个结果的过滤。
```java
DocumentRetriever retriever = VectorStoreDocumentRetriever.builder()
    .vectorStore(vectorStore)
    .similarityThreshold(0.73)
    .topK(5)
    .filterExpression(new FilterExpressionBuilder()
        .eq("genre", "fairytale")
        .build())
    .build();
List<Document> documents = retriever.retrieve(new Query("What is the main character of the story?"));

// query  context 指定 filter
Query query = Query.builder()
    .text("Who is Anacletus?")
    .context(Map.of(VectorStoreDocumentRetriever.FILTER_EXPRESSION, "location == 'Whispering Woods'"))
    .build();
List<Document> retrievedDocuments = documentRetriever.retrieve(query);
```


#### Retrieval - ConcatenationDocumentJoiner
将文档组合在一起。
如果存在重复的文档，则保留第一次出现的文档。
每个文档的评分保持不变。
```java
Map<Query, List<List<Document>>> documentsForQuery = ...
DocumentJoiner documentJoiner = new ConcatenationDocumentJoiner();
List<Document> documents = documentJoiner.join(documentsForQuery);
```


## ETL
从原始数据源到结构化向量存储的流程
### DocumentReader
```java
public interface DocumentReader extends Supplier<List<Document>> {

    default List<Document> read() {
        return get();
    }
}
```
**JSON**
```java
List<Document> doc = new JsonReader(resource, "description", "content").get()
```
构造器
* JsonReader(Resource resource)
* JsonReader(Resource resource, String… jsonKeysToUse)
* JsonReader(Resource resource, JsonMetadataGenerator jsonMetadataGenerator, String… jsonKeysToUse)


**Text**
```java
TextReader textReader = new TextReader(resource);
textReader.getCustomMetadata().put("filename", "text-source.txt");
List<Document> doc = textReader.read();
```
构造器
* TextReader(String resourceUrl)
* TextReader(Resource resource)

**HTML**
```java
JsoupDocumentReaderConfig config = JsoupDocumentReaderConfig.builder()
            .selector("article p") // Extract paragraphs within <article> tags
            .charset("ISO-8859-1")  // Use ISO-8859-1 encoding
            .includeLinkUrls(true) // Include link URLs in metadata
            .metadataTags(List.of("author", "date")) // Extract author and date meta tags
            .additionalMetadata("source", "my-page.html") // Add custom metadata
            .build();

JsoupDocumentReader reader = new JsoupDocumentReader(resource, config);
List<Document> doc = reader.get();

```

**Markdown**
```java
MarkdownDocumentReaderConfig config = MarkdownDocumentReaderConfig.builder()
    .withHorizontalRuleCreateDocument(true) // 分割线 创建新文档
    .withIncludeCodeBlock(false) // 代码块创建新文档
    .withIncludeBlockquote(false) // 引用块 创建新文档
    .withAdditionalMetadata("filename", "code.md")
    .build();

MarkdownDocumentReader reader = new MarkdownDocumentReader(resource, config);
List<Document> doc = reader.get();
```
config:
* 头部信息会成为Document对象中的元数据。
* 段落成为Document对象的内容。
* 代码块可以单独作为独立的文档对象，也可以与周围的文本一起包含在内。
* 引用块可以单独作为独立的文档对象，也可以与周围的文本一起包含在内。
* 水平规则可用于将内容拆分为单独的Document对象。


**others**
pdf
docx
...


### DocumentTransformer
```java
public interface DocumentTransformer extends Function<List<Document>, List<Document>> {

    default List<Document> transform(List<Document> transform) {
        return apply(transform);
    }
}
```

**TokenTextSplitter**
根据token数切割
```java
public List<Document> splitCustomized(List<Document> documents) {
        TokenTextSplitter splitter = new TokenTextSplitter(1000, 400, 10, 5000, true);
        return splitter.apply(documents);
    }
```
构造器 `TokenTextSplitter(int defaultChunkSize, int minChunkSizeChars, int minChunkLengthToEmbed, int maxNumChunks, boolean keepSeparator)`
* defaultChunkSize: The target size of each text chunk in tokens (default: 800).
* minChunkSizeChars: The minimum size of each text chunk in characters (default: 350).
* minChunkLengthToEmbed: The minimum length of a chunk to be included (default: 5).
* maxNumChunks: The maximum number of chunks to generate from a text (default: 10000).
* keepSeparator: Whether to keep separators (like newlines) in the chunks (default: true).



**KeywordMetadataEnricher**
使用大模型抽取关键字
```java
List<Document> enrichDocuments(List<Document> documents) {
        KeywordMetadataEnricher enricher = KeywordMetadataEnricher.builder(chatModel)
                .keywordCount(5)
                .build();

        // Or use custom templates
        KeywordMetadataEnricher enricher = KeywordMetadataEnricher.builder(chatModel)
               .keywordsTemplate(YOUR_CUSTOM_TEMPLATE)
               .build();

        return enricher.apply(documents);
    }
```

**SummaryMetadataEnricher**
生成文档摘要
```java
SummaryMetadataEnricher  enricher  =  new SummaryMetadataEnricher(aiClient,
            List.of(SummaryType.PREVIOUS, SummaryType.CURRENT, SummaryType.NEXT));

List<Document> doc =  this.enricher.apply(documents);
```




### DocumentWriter
```java
public interface DocumentWriter extends Consumer<List<Document>> {

    default void write(List<Document> documents) {
        accept(documents);
    }
}
```

**FileDocumentWriter**
```java
 FileDocumentWriter writer = new FileDocumentWriter("output.txt", true, MetadataMode.ALL, false);
        writer.accept(documents);
```

**VectorStore**
```java
@Autowired VectorStore vectorStore;

// ...

List<Document> documents = List.of(
    new Document("Spring AI rocks!! Spring AI rocks!! Spring AI rocks!! Spring AI rocks!! Spring AI rocks!!", Map.of("meta1", "meta1")),
    new Document("The World is Big and Salvation Lurks Around the Corner"),
    new Document("You walk forward facing the past and you turn back toward the future.", Map.of("meta2", "meta2")));

// Add the documents to PGVector
vectorStore.add(documents);

// Retrieve documents similar to a query
List<Document> results = this.vectorStore.similaritySearch(SearchRequest.builder().query("Spring").topK(5).build());
```