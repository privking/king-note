# firstDemo

## BatchDemo

```java
package priv.king.chapter1;

import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

public class BatchDemo {

    public static void main(String[] args) throws Exception {

        //获取执行环境
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //获取自定义的数据流
        DataStream<Tuple2<String, Integer>> dataStream =env.fromElements("Flink batch demo", "batch demo", "demo")
                .flatMap(new Splitter())
                .keyBy(x->x.f0)
                .sum(1);
        //打印数据到控制台
        env.setParallelism(2);
        System.out.println(env.getExecutionPlan());
        dataStream.print();
        //执行任务操作。因为flink是懒加载的，所以必须调用execute方法才会执行
        env.execute("WordCount");


    }

    //使用FlatMapFunction函数分割字符串
    public static class Splitter implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String sentence, Collector<Tuple2<String, Integer>> out) throws Exception {
            for (String word : sentence.split(" ")) {
                out.collect(new Tuple2<String, Integer>(word, 1));
            }
        }
    }
}
```

## StreamDemo

```java
package priv.king.chapter1;


import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;

import java.util.Arrays;

public class StreamDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        // socketStream   nc -lk 10086
        DataStreamSource<String> dataStream = env.socketTextStream("localhost", 10086);

        SingleOutputStreamOperator<Tuple2<String, Integer>> res = dataStream
                //lambda表达式这里必须指定FlatMapFunction 和 .returns
                //不能推断
                .flatMap((FlatMapFunction<String, Tuple2<String, Integer>>) (in, out) -> Arrays.stream(in.split(" ")).forEach(x -> out.collect(new Tuple2<>(x, 1))))
                .returns(Types.TUPLE(Types.STRING, Types.INT))
                .keyBy(x->x.f0)
                .timeWindow(Time.seconds(10))
                .sum(1);
        //sink
        res.print();
        //执行任务
        env.execute("word count");
    }
}
```

## TableDemo

```java
package priv.king.chapter1;


import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;
import org.apache.flink.types.Row;

import static org.apache.flink.table.api.Expressions.$;

public class TableDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        EnvironmentSettings settings = EnvironmentSettings.newInstance()
                .useBlinkPlanner() //使用BlinkPlaner
                .inStreamingMode() //设置流式模式 默认
                .build();

        StreamTableEnvironment tEnv = StreamTableEnvironment.create(env, settings);

        DataStreamSource<String> dataStream = env.socketTextStream("localhost", 10086);

        Table table = tEnv.fromDataStream(dataStream, $("word"));


        Table res = table.where($("word").like("%a%"));

        DataStream<Row> resStream = tEnv.toAppendStream(res, Row.class);

        resStream.print("tb");

        env.execute();

    }
}
```

## SqlDemo

```java
package priv.king.chapter1;

import org.apache.flink.streaming.api.datastream.DataStreamSource;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;
import org.apache.flink.types.Row;

//需要手动导入
import static org.apache.flink.table.api.Expressions.$;

public class SqlDemo {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        StreamTableEnvironment tEnv = StreamTableEnvironment.create(env);

        DataStreamSource<String> dataStream = env.socketTextStream("localhost", 10086);

        Table table = tEnv.fromDataStream(dataStream, $("word"));

        Table table1 = tEnv.sqlQuery("select * from " + table + " where word like '%a%'");

        tEnv.toAppendStream(table1, Row.class).print();

        env.execute();
    }
}

```