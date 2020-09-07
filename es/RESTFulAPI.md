# RESTful API 
## 创建空索引
```
PUT /king  
{ 
"settings": { 
    "number_of_shards": "2",   //分片数       
    "number_of_replicas": "0",  //副本数  
    "write.wait_for_active_shards": 1
    }     
}     

```
## 修改副本数 
```
PUT king/_settings
{ 
"number_of_replicas" : "2" 
}
```
## 删除索引
```
DELETE /king
```
## 插入数据
```
//指定id 
POST /king/_doc/1001 
{   
    "id":1001, 
    "name":"张三",
    "age":20, 
    "sex":"男" 
    
}   
//不指定id  es帮我们自动生成 
POST /king/_doc
{  
    "id":1002,  
    "name":"三哥",  
    "age":20,  
    "sex":"男"
}
```
## 更新数据
```
PUT /king/_doc/1001 
{   
    "id":1009, 
    "name":"king",  
    "age":21, 
    "sex":"哈哈" 
}

```
## 局部更新
```
POST /king/_update/1001
{   
    "doc":{ 
    "age":23  
    } 
}
```
## 删除数据
```
DELETE /king/_doc/1001
```
## 根据id搜索数据
```
GET /king/_doc/1001
```
## 搜索全部数据
```
GET /king/_search    默认最多返回10条数据

GET /king/_search?q=sex:男

POST /bank/_search
{ 
    "query": { "match_all": {} }, 
    "sort": [  
        {     
        "属性名": {  
            "order": "asc"  
            }   
        } 
    ]
}
```

## 搜索数据返回结果
```
//默认返回10条
{
  "took" : 2, 
  "timed_out" : false, //是否超时
  "_shards" : {
    "total" : 3, //分片数
    "successful" : 3,//成功多少
    "skipped" : 0, //跳过数
    "failed" : 0 //失败数
  },
  "hits" : {
    "total" : {
      "value" : 4, //总数
      "relation" : "eq"
    },
    "max_score" : 1.0, //相似度最大得分
    "hits" : [
      {
        "_index" : "king", //索引
        "_type" : "_doc", //类型
        "_id" : "ichzLXMBW1Amo8LgOlWM", //id
        "_score" : 1.0, //得分
        "_source" : { //原始数据
          "id" : 100,
          "name" : "张三",
          "sex" : "男"
        }
      },
      {
        "_index" : "king",
        "_type" : "_doc",
        "_id" : "i8h0LXMBW1Amo8Lgj1Wv",
        "_score" : 1.0,
        "_source" : {
          "id" : 101,
          "name" : "王五",
          "sex" : "女"
        }
      },
      {
        "_index" : "king",
        "_type" : "_doc",
        "_id" : "1001",
        "_score" : 1.0,
        "_source" : {
          "id" : 123,
          "name" : "king"
        }
      },
      {
        "_index" : "king",
        "_type" : "_doc",
        "_id" : "ishzLXMBW1Amo8LgnFXZ",
        "_score" : 1.0,
        "_source" : {
          "id" : 100,
          "name" : "李四",
          "sex" : "女"
        }
      }
    ]
  }
}
```

## 返回数据的指定字段

```
GET /king/_doc/1001?_source=id,name
```
## 只返回原始数据

```
GET /king/_source/1001
```
##  判断文段是否存在

```
HEAD /king/_doc/1001
```
## 批量查询

```
POST /king/_mget
{
  "ids": [
    "1001",
    "ishzLXMBW1Amo8LgnFXZ"
  ]
}
```
## 批量操作 增删改

```
POST _bulk
{"create":{"_index":"king","_id":1002}}
{"id":1002,"name":"赵六","sex":"女"}
{"update":{"_id":1001,"_index":"king"}}
{"doc":{"id":"1234"}}
{"delete":{"_index":"king","_id":1002}}
```

## 分页查询

### 方式一

```
GET /king/_search
{
  "query":{
    "match_all": {}
    
  },
  "size":4,
  "from":2
}
```
### 方式二 深分页

```
//会返回scroll_id
GET /king/_search?scroll=5m
{
  "size":2,
  "from":0,
  "sort": [
    {
      "_id": {
        "order": "desc"
      }
    }
  ]
}

//有了scroll_id后  带上scroll_id 查询下一页数据
GET _search/scroll
{
 "scroll_id":"DnF1ZXJ5VGhlbkZldGNoAwAAAAAAAB0uFkRHZjNILTlMUlJpaTdUZVFnbGVFMlEAAAAAAAAdLBZER2YzSC05TFJSaWk3VGVRZ2xlRTJRAAAAAAAAHS0WREdmM0gtOUxSUmlpN1RlUWdsZUUyUQ==",
 "scroll":"5m"
}
//删除scroll_id
DELETE _search/scroll/DnF1ZXJ5VGhlbkZldGNoAwAAAAAAAB0uFkRHZjNILTlMUlJpaTdUZVFnbGVFMlEAAAAAAAAdLBZER2YzSC05TFJSaWk3VGVRZ2xlRTJRAAAAAAAAHS0WREdmM0gtOUxSUmlpN1RlUWdsZUUyUQ==

```
### 方式三

```
//第一次查询 和分页1一样
GET /king/_search
{
  "size":2,
  "from":0,
  "sort": [
    {
      "_id": {
        "order": "desc"
      }
    }
  ]
}

//非第一次查询 
//加上参数search_after，参数未上一次查询的结果 "sort"
//from 必须是0
//实时性查询

GET /king/_search
{
  "size":2,
  "from":0,
  "sort": [
    {
      "_id": {
        "order": "desc"
      }
    }
  ],
  "search_after":[
    "ishzLXMBW1Amo8LgnFXZ"
    ]
}
```
## 结构化查询

### term查询常用于精确匹配

```
//name.keyword 不会分词
//name 会分词
POST /king/_search
{
  "query": {
    "term": {
      "name.keyword": "李四"
    }
  }
}

POST /king/_search
{
  "query": {
    "term": {
      "name": "三"
    }
  }
}

POST /king/_search
{
  "query": {
    "terms": {
      "name": ["三","四"]
    }
  }
}
```
### 分词测试查询

```
POST /_analyze
{
  "analyzer":"standard",
  "text":"李四"
  
}
```
### 范围查询 range

```
//gt >
//gte >=
//lt <
//lte <=
POST /king/_search
{
  "query":{
    "range": {
      "age": {
        "gte": 10,
        "lte": 30
      }
    }
  }
  
}
```
### 存在查询 exists 判断字段是否存在
```
//返回包含该字段的数据
POST /king/_search
{
  "query":{
    "exists": {
      "field":"sex"
    }
  }
}
```
### match查询

```
//先对查询条件分词再查询
POST /king/_search
{
  "query":{
    "match": {
      "name": "张三"
    }
  }
}
```
### bool查询

```
//must :: 多个查询条件的完全匹配,相当于 and 。 
//must_not :: 多个查询条件的相反匹配，相当于 not 。 
//should :: 至少有一个查询条件匹配, 相当于 or 
//filter 与must相似但是不参与计算分值
POST /king/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "exists": {
            "field": "sex"
          }
        },
        {
          "term": {
            "name": "king"
          }
        }
      ],
      "must": {
        "range": {
          "age": {
            "gte": 20
          }
        }
      }
    }
  }
}
```

## 聚合查询

```
//age_avg 自定义返回结果名称
//size:0  不会返回原始数据
//sum avg max min

POST /king/_search
{
  "aggs": {
    "age_avg": {
      "sum": {
        "field": "age"
      }
    }
  },
  "size": 0
}
```
### 去重统计数量

```
//myname 自定义返回结果名称

POST /king/_search
{
  "aggs": {
    "myname": {
      "cardinality": {
        "field": "age"
      }
    }
  },
  "size": 0
}
```

### 统计总数
```
POST /king/_search
{
  "aggs": {
    "myname": {
      "value_count": {
        "field": "age"
      }
    }
  },
  "size": 0
}
```

## 扩展查询
```
//包括总数 最小值 最大值 平均值 和 方差 标准差等
POST /king/_search
{
  "aggs": {
    "myname": {
      "extended_stats": {
        "field": "age"
      }
    }
  },
  "size": 0
}
```
### 类似于group by

```
//根据filed值作为key分别统计数量
POST /king/_search
{
  "aggs": {
    "myname": {
      "terms": {
        "field": "age"
      }
    }
  },
  "size": 0
}

//取出每组前3条原始数据
POST /king/_search
{
  "aggs": {
    "myname": {
      "terms": {
        "field": "age"
      },
      "aggs": {
        "count": {
          "top_hits": {
            "size": 3
          }
        }
      }
    }
  },
  "size": 0
}

```

### 根据范围分组

```
POST /king/_search
{
  "aggs": {
    "myname": {
      "range": {
        "field": "age",
        "ranges": [
          {
            "from": 10,
            "to": 20
          },
          {
            "from": 20,
            "to": 30
          },
          {
            "from": 30,
            "to": 40
          }
        ]
      }
    }
  },
  "size": 0
}

```