# java api
## maven
```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```
## 配置

```
@Configuration
public class ElasticsearchConfig extends AbstractElasticsearchConfiguration {
    @Override
    public RestHighLevelClient elasticsearchClient() {

        return RestClients.create(ClientConfiguration.create("172.18.95.136:9200")).rest();
    }

    // no special bean creation needed
}
```

## 实体类

```
@Document(indexName = "es_rest_goods_test", type = "_doc", shards = 1, replicas = 3, createIndex = true,
        useServerConfiguration = false, versionType = VersionType.EXTERNAL)
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Goods {
    //id注解必须有
    @Id
    private Long id;

    @Field(analyzer = "ik_max_word",type = FieldType.Text)
    private String name;

    @Field(type = FieldType.Keyword)
    private Double prise;

    @Field(analyzer = "ik_max_word",type = FieldType.Text)
    private String desc;

    @Field(type = FieldType.Integer)
    private Integer count;

    @Version
    private Long version;
}

```
## mapper

### 父类有很多方法
### 通过方法名称
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599207653-7e2b2f.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599207669-f32761.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599207749-02949b.png)

* 支持的返回类型
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599207762-cd3673.png)
### 通过注解
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599207775-4ebd21.png)
```
public interface GoodsMapper extends ElasticsearchRepository<Goods,String> {


}

```

## 查询demo

```java
@SpringBootTest
class ElasticsearchDemoApplicationTests {

    @Autowired
    private ElasticsearchRestTemplate elasticsearchTemplate;

    @Autowired
    private GoodsMapper goodsMapper;

    //创建索引以及映射
    @Test
    void contextLoads() {
        elasticsearchTemplate.createIndex(Goods.class);
    }

    //插入数据 修改数据
    @Test
    void contextLoads2() {
        Goods goods = Goods.builder().id(1001L).name("macBook pro").desc("苹果电脑").prise(100.2).count(998).version(1L).build();
        Goods save = goodsMapper.save(goods);
        System.out.println(save);
    }

    //删除
    @Test
    void tset3() {
        //Goods goods = Goods.builder().id(1001L).build();
        //goodsMapper.delete(goods);
        goodsMapper.deleteById("1002");
    }

    //查询
    @Test
    void test4() {
        //所有
        // goodsMapper.findAll();
        //id查询
        // goodsMapper.findById("1003");
        //ids 查询
        //goodsMapper.findAllById(new HashSet<>());
        PageRequest pageRequest = PageRequest.of(0, 20, Sort.Direction.ASC, "id");
        Page<Goods> goodsPage = goodsMapper.findAll(pageRequest);
        Iterator<Goods> iterator = goodsPage.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    //判断是否存在
    @Test
    void test5() {
        goodsMapper.existsById("1001");
    }

    //查询 goodsMapper.search(builder);
    @Test
    void test6() {
        //过时 只能查询没有分词的字段
        //各种Builder
        //MatchQueryBuilder match查询
        //TermsQueryBuilder term 查询
        //ExistsQueryBuilder exists查询
        //BoolQueryBuilder bool 查询
        //MatchQueryBuilder queryBuilder = new MatchQueryBuilder("name","mac");
        TermsQueryBuilder termsQueryBuilder = new TermsQueryBuilder("count", "998");
        Iterable<Goods> goodsPage = goodsMapper.search(termsQueryBuilder);
        Iterator<Goods> iterator = goodsPage.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }

    //分页查询
    @Test
    void testPage() {
        SearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withQuery(matchAllQuery())
                .withIndices("es_rest_goods_test")
                .withPageable(PageRequest.of(0, 10))
                .build();

        ScrolledPage<Goods> scroll = elasticsearchTemplate.startScroll(1000, searchQuery, Goods.class);
        String scrollId = scroll.getScrollId();
        List<Goods> content = scroll.getContent();
        content.forEach(item -> {
            System.out.println(item);
        });
        System.out.println(scrollId);
        ScrolledPage<Goods> scroll1 = elasticsearchTemplate.continueScroll(scrollId, 1000, Goods.class);
        String scrollId1 = scroll1.getScrollId();
        List<Goods> content1 = scroll1.getContent();
        content1.forEach(item -> {
            System.out.println(item);
        });
        System.out.println(scrollId1);
        //scrollId1 == scrollId
        elasticsearchTemplate.clearScroll(scrollId);

    }

    //平均
    @Test
    void test7() {
        AvgAggregationBuilder builder = AggregationBuilders.avg("avg_name").field("prise");
        NativeSearchQuery query = new NativeSearchQueryBuilder().addAggregation(builder).withIndices("es_rest_goods_test").build();
        Aggregations aggregations = elasticsearchTemplate.query(query, response -> response.getAggregations());
        // String scrollId = elasticsearchTemplate.query(query, response -> response.getScrollId());
        aggregations.asMap().forEach((key, value) -> {
            ParsedAvg avg = (ParsedAvg) value;
            System.out.println(key + " " + avg.getName() + " " + avg.getValue() + " " + avg.value());
            //avg_name avg_name 149.2098979383412 149.2098979383412
        });

    }

    //去重统计总数
    @Test
    void test8() {
        CardinalityAggregationBuilder builder = AggregationBuilders.cardinality("prise_cardinality").field("prise");
        NativeSearchQuery query = new NativeSearchQueryBuilder().addAggregation(builder).withIndices("es_rest_goods_test").build();
        Aggregations aggregations = elasticsearchTemplate.query(query, response -> response.getAggregations());
        aggregations.asMap().forEach((key, value) -> {
            //prise_cardinality
            System.out.println(key);
            ParsedCardinality parsedCardinality = (ParsedCardinality)value;
            //100
            System.out.println(parsedCardinality.getValue());

        });
    }
    //range查询
    @Test
    void test9() {
        RangeAggregationBuilder rangeAggregationBuilder = AggregationBuilders.range("price_range").field("count")
                .addRange(920,930).addRange(930,940).addRange(940,950);
        NativeSearchQuery query = new NativeSearchQueryBuilder().addAggregation(rangeAggregationBuilder).withIndices("es_rest_goods_test").build();
        Aggregations aggregations = elasticsearchTemplate.query(query, response -> response.getAggregations());
        // String scrollId = elasticsearchTemplate.query(query, response -> response.getScrollId());
        aggregations.asMap().forEach((key, value) -> {
            //price_range
            System.out.println(key);
            ParsedRange range = (ParsedRange) value;
            /**
             * 920.0-930.0 10
             * 930.0-940.0 10
             * 940.0-950.0 10
             */
            range.getBuckets().forEach((bucket)->{
                System.out.println(bucket.getKeyAsString()+" "+bucket.getDocCount());
            });
        });
    }

    //top_hits 根据权值据合
    @Test
    void test10() {
        TermsAggregationBuilder termsAggregationBuilder = AggregationBuilders.terms("price_name").field("count");
        TopHitsAggregationBuilder topHitsAggregationBuilder = AggregationBuilders.topHits("top").size(3);
        termsAggregationBuilder.subAggregation(topHitsAggregationBuilder);
        NativeSearchQuery query = new NativeSearchQueryBuilder().addAggregation(termsAggregationBuilder).withIndices("es_rest_goods_test").build();
        Aggregations aggregations = elasticsearchTemplate.query(query, response -> response.getAggregations());
        // String scrollId = elasticsearchTemplate.query(query, response -> response.getScrollId());
        aggregations.asMap().forEach((key, value) -> {
            //price_range
            System.out.println(key);
            ParsedLongTerms parsedLongTerms = (ParsedLongTerms)value;
            parsedLongTerms.getBuckets().forEach((bucket)->{
                Aggregation aggregation = bucket.getAggregations().getAsMap().get("top");
                TopHits topHits= (TopHits) aggregation;
                Iterator<SearchHit> iterator = topHits.getHits().iterator();
                while (iterator.hasNext()){
                    SearchHit next = iterator.next();
                    Object object= JSONObject.parse(next.getSourceAsString());
                    System.out.println(object);
                }
            });
        });
    }


}
```