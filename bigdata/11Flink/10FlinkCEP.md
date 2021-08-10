# FlinkCEP

FlinkCEP是在Flink上层实现的**复杂事件处理**库。 它可以让你在无限事件流中检测出特定的事件模型

## 依赖

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-cep-scala_2.11</artifactId>
    <version>1.13.0</version>
</dependency>
```

## Demo

```scala
val input: DataStream[Event] = ...

val pattern = Pattern.begin[Event]("start").where(_.getId == 42)
  .next("middle").subtype(classOf[SubEvent]).where(_.getVolume >= 10.0)
  .followedBy("end").where(_.getName == "end")

val patternStream = CEP.pattern(input, pattern)

val result: DataStream[Alert] = patternStream.process(
    new PatternProcessFunction[Event, Alert]() {
        override def processMatch(
              patterns: util.Map[String, util.List[Event]],
              ctx: PatternProcessFunction.Context,
              out: Collector[Alert]): Unit = {
            out.collect(createAlertFrom(patterns))
        }
    })
```

## 模式API

模式API可以让你定义想从输入流中抽取的复杂模式序列。

每个模式必须有一个独一无二的名字，你可以在后面使用它来识别匹配到的事件

 模式的名字不能包含字符`:`.

```scala
val start : Pattern[Event, _] = Pattern.begin("start")
```

### 个体模式

组成复杂规则的每一个单独的模式定义，就是“个体模式”

**个体模式**可以是一个**单例**或者**循环**模式。单例模式只接受一个事件，循环模式可以接受多个事件。 在模式匹配表达式中，模式`"a b+ c? d"`（`"a"`，后面跟着一个或者多个`"b"`，再往后可选择的跟着一个`"c"`，最后跟着一个`"d"`）， `a`，`c?`，和 `d`都是单例模式，`b+`是一个循环模式。

默认情况下，模式都是单例的，你可以通过使用**量词**把它们转换成循环模式。 

每个模式可以有一个或者多个条件来决定它接受哪些事件。

#### 量词

```scala
// 期望出现4次
start.times(4)

// 期望出现0或者4次
start.times(4).optional()

// 期望出现2、3或者4次
start.times(2, 4)

// 期望出现2、3或者4次，并且尽可能的重复次数多
start.times(2, 4).greedy()

// 期望出现0、2、3或者4次
start.times(2, 4).optional()

// 期望出现0、2、3或者4次，并且尽可能的重复次数多
start.times(2, 4).optional().greedy()

// 期望出现1到多次
start.oneOrMore()

// 期望出现1到多次，并且尽可能的重复次数多
start.oneOrMore().greedy()

// 期望出现0到多次
start.oneOrMore().optional()

// 期望出现0到多次，并且尽可能的重复次数多
start.oneOrMore().optional().greedy()

// 期望出现2到多次
start.timesOrMore(2)

// 期望出现2到多次，并且尽可能的重复次数多
start.timesOrMore(2).greedy()

// 期望出现0、2或多次
start.timesOrMore(2).optional()

// 期望出现0、2或多次，并且尽可能的重复次数多
start.timesOrMore(2).optional().greedy()
```

#### 条件

对每个模式你可以指定一个条件来决定一个进来的事件是否被接受进入这个模式

通过`pattern.where()`、`pattern.or()`或者`pattern.until()`方法

这些可以是`IterativeCondition`或者`SimpleCondition`。

##### 迭代条件（IterativeCondition）

下面是一个迭代条件的代码，它接受"middle"模式下一个事件的名称开头是"foo"， 并且前面已经匹配到的事件加上这个事件的价格小于5.0。

`ctx.getEventsForPattern(...)`可以获得所有前面已经接受作为可能匹配的事件。 调用这个操作的代价可能很小也可能很大，所以在实现你的条件时，**尽量少使用它**。

```scala
middle.oneOrMore()
    .subtype(classOf[SubEvent])
    .where(
        (value, ctx) => {
            lazy val sum = ctx.getEventsForPattern("middle").map(_.getPrice).sum
            value.getName.startsWith("foo") && sum + value.getPrice < 5.0
        }
    )
```

##### 简单条件（SimpleCondition）

```scala
start.where(event => event.getName.startsWith("foo"))
```

##### or

默认是and逻辑

```scala
//subtype 和where就是and逻辑
//subtype限制接受的事件类型是初始事件的子类型
start.subtype(classOf[SubEvent]).where(subEvent => ... /* 一些判断条件 */)
```

```scala
pattern.where(event => ... /* 一些判断条件 */).or(event => ... /* 一些判断条件 */)
```

##### until

如果使用循环模式（`oneOrMore()`和`oneOrMore().optional()`），你可以指定一个停止条件，例如，接受事件的值大于5直到值的和小于50。

为了更好的理解它，看下面的例子。给定

- 模式如`"(a+ until b)"` (一个或者更多的`"a"`直到`"b"`)
- 到来的事件序列`"a1" "c" "a2" "b" "a3"`
- 输出结果会是： `{a1 a2} {a1} {a2} {a3}`.

你可以看到`{a1 a2 a3}`和`{a2 a3}`由于停止条件没有被输出。

#### 个体模式api

| 模式操作                    | 描述                                                         |
| --------------------------- | ------------------------------------------------------------ |
| where(condition)            | 当前模式定义一个条件。为了匹配这个模式，一个事件必须满足某些条件。 |
| or(condition)               | 增加一个新的判断，和当前的判断取或。一个事件只要满足至少一个判断条件就匹配到模式 |
| until(condition)            | 为循环模式指定一个停止条件。意思是满足了给定的条件的事件出现后，就不会再有事件被接受进入模式了。 |
| subtype(subClass)           | 为当前模式定义一个子类型条件。一个事件只有是这个子类型的时候才能匹配到模式 |
| oneOrMore()                 | 指定模式期望匹配到的事件至少出现一次。默认（在子事件间）使用松散的内部连续性。 |
| timesOrMore(#times)         | 指定模式期望匹配到的事件至少出现#times次。默认（在子事件间）使用松散的内部连续性。 |
| times(#ofTimes)             | 指定模式期望匹配到的事件正好出现的次数。默认（在子事件间）使用松散的内部连续性。 |
| times(#fromTimes, #toTimes) | 指定模式期望匹配到的事件出现次数在#fromTimes和#toTimes之间。默认（在子事件间）使用松散的内部连续性。 |
| optional()                  | 指定这个模式是可选的，也就是说，它可能根本不出现。这对所有之前提到的量词都适用。 |
| greedy()                    | 指定这个模式是贪心的，也就是说，它会重复尽可能多的次数。这只对量词适用，现在还不支持模式组。 |

### 组合模式

可以增加更多的模式到模式序列中并指定它们之间所需的*连续条件*，组成一个完整的模式序列

**连续策略：**

1. **严格连续 next()**: 期望所有匹配的事件严格的一个接一个出现，中间没有任何不匹配的事件。
2. **松散连续 followedBy()**: 忽略匹配的事件之间的不匹配的事件。
3. **不确定的松散连续 followedByAny()**: 更进一步的松散连续，允许忽略掉一些匹配事件的附加匹配。
4. **严格连续的NOT模式 notNext**: 不想后面直接连着一个特定事件
5. **松散连续的NOT模式 notFollowedBy()**:不想一个特定事件发生在两个事件之间的任何地方

**模式序列不能以`notFollowedBy()`结尾**

**一个`NOT`模式前面不能是可选的模式**

```scala
// 严格连续
val strict: Pattern[Event, _] = start.next("middle").where(...)

// 松散连续
val relaxed: Pattern[Event, _] = start.followedBy("middle").where(...)

// 不确定的松散连续
val nonDetermin: Pattern[Event, _] = start.followedByAny("middle").where(...)

// 严格连续的NOT模式
val strictNot: Pattern[Event, _] = start.notNext("not").where(...)

// 松散连续的NOT模式
val relaxedNot: Pattern[Event, _] = start.notFollowedBy("not").where(...)
```

**松散连续意味着跟着的事件中，只有第一个可匹配的事件会被匹配上，而不确定的松散连接情况下，有着同样起始的多个匹配会被输出。** 

举例来说，模式`"a b"`，给定事件序列`"a"，"c"，"b1"，"b2"`，会产生如下的结果

1. `"a"`和`"b"`之间严格连续： `{}` （没有匹配），`"a"`之后的`"c"`导致`"a"`被丢弃。
2. `"a"`和`"b"`之间松散连续： `{a b1}`，松散连续会"跳过不匹配的事件直到匹配上的事件"。
3. `"a"`和`"b"`之间不确定的松散连续： `{a b1}`, `{a b2}`，这是最常见的情况。

**也可以为模式定义一个有效时间约束**。 例如，你可以通过`pattern.within()`方法指定一个模式应该在10秒内发生。 这种时间模式支持处理时间和事件时间.

```scala
//一个模式序列只能有一个时间限制。如果限制了多个时间在不同的单个模式上，会使用最小的那个时间限制。
next.within(Time.seconds(10))
```

#### 循环模式中的连续性

一个模式序列`"a b+ c"`（`"a"`后面跟着一个或者多个（不确定连续的）`"b"`，然后跟着一个`"c"`） 输入为`"a"，"b1"，"d1"，"b2"，"d2"，"b3"，"c"`，输出结果如下：

1. **严格连续**: `{a b3 c}` – `"b1"`之后的`"d1"`导致`"b1"`被丢弃，同样`"b2"`因为`"d2"`被丢弃。
2. **松散连续**: `{a b1 c}`，`{a b1 b2 c}`，`{a b1 b2 b3 c}`，`{a b2 c}`，`{a b2 b3 c}`，`{a b3 c}` - `"d"`都被忽略了。
3. **不确定松散连续**: `{a b1 c}`，`{a b1 b2 c}`，`{a b1 b3 c}`，`{a b1 b2 b3 c}`，`{a b2 c}`，`{a b2 b3 c}`，`{a b3 c}` - 注意`{a b1 b3 c}`，这是因为`"b"`之间是**不确定松散连续产生的**。

对于循环模式（例如`oneOrMore()`和`times()`)），默认是*松散连续*。如果想使用***严格连续***，你需要使用`consecutive()`方法明确指定， 如果想使用***不确定松散连续***，你可以使用`allowCombinations()`方法。

```scala
Pattern.begin("start").where(_.getName().equals("c"))
  .followedBy("middle").where(_.getName().equals("a"))
                       .oneOrMore().consecutive()
  .followedBy("end1").where(_.getName().equals("b"))


Pattern.begin("start").where(_.getName().equals("c"))
  .followedBy("middle").where(_.getName().equals("a"))
                       .oneOrMore().allowCombinations()
  .followedBy("end1").where(_.getName().equals("b"))
```

#### 模式组

定义一个模式序列作为`begin`，`followedBy`，`followedByAny`和`next`的条件

这个模式序列在逻辑上会被当作匹配的条件， 并且返回一个`GroupPattern`，可以在`GroupPattern`上使用`oneOrMore()`，`times(#ofTimes)`， `times(#fromTimes, #toTimes)`，`optional()`，`consecutive()`，`allowCombinations()`。

```scala
val start: Pattern[Event, _] = Pattern.begin(
    Pattern.begin[Event]("start").where(...).followedBy("start_middle").where(...)
)

// 严格连续
val strict: Pattern[Event, _] = start.next(
    Pattern.begin[Event]("next_start").where(...).followedBy("next_middle").where(...)
).times(3)

// 松散连续
val relaxed: Pattern[Event, _] = start.followedBy(
    Pattern.begin[Event]("followedby_start").where(...).followedBy("followedby_middle").where(...)
).oneOrMore()

// 不确定松散连续
val nonDetermin: Pattern[Event, _] = start.followedByAny(
    Pattern.begin[Event]("followedbyany_start").where(...).followedBy("followedbyany_middle").where(...)
).optional()
```

| 模式操作                             |                             描述                             |
| :----------------------------------- | :----------------------------------------------------------: |
| **begin(#name)**                     | 定一个开始模式：```scala val start = Pattern.begin[Event]("start") ``` |
| **begin(#pattern_sequence)**         | 定一个开始模式：```scala val start = Pattern.begin( Pattern.begin[Event]("start").where(...).followedBy("middle").where(...) ) ``` |
| **next(#name)**                      | 增加一个新的模式，匹配的事件必须是直接跟在前面匹配到的事件后面（严格连续）：```scala val next = start.next("middle") ``` |
| **next(#pattern_sequence)**          | 增加一个新的模式。匹配的事件序列必须是直接跟在前面匹配到的事件后面（严格连续）：```scala val next = start.next( Pattern.begin[Event]("start").where(...).followedBy("middle").where(...) ) ``` |
| **followedBy(#name)**                | 增加一个新的模式。可以有其他事件出现在匹配的事件和之前匹配到的事件中间（松散连续）：```scala val followedBy = start.followedBy("middle") ``` |
| **followedBy(#pattern_sequence)**    | 增加一个新的模式。可以有其他事件出现在匹配的事件和之前匹配到的事件中间（松散连续）：```scala val followedBy = start.followedBy( Pattern.begin[Event]("start").where(...).followedBy("middle").where(...) ) ``` |
| **followedByAny(#name)**             | 增加一个新的模式。可以有其他事件出现在匹配的事件和之前匹配到的事件中间， 每个可选的匹配事件都会作为可选的匹配结果输出（不确定的松散连续）：```scala val followedByAny = start.followedByAny("middle") ``` |
| **followedByAny(#pattern_sequence)** | 增加一个新的模式。可以有其他事件出现在匹配的事件序列和之前匹配到的事件中间， 每个可选的匹配事件序列都会作为可选的匹配结果输出（不确定的松散连续）：```scala val followedByAny = start.followedByAny( Pattern.begin[Event]("start").where(...).followedBy("middle").where(...) ) ``` |

#### 匹配后跳过策略

- NO_SKIP: 每个成功的匹配都会被输出。
- SKIP_TO_NEXT: 丢弃以相同事件开始的所有部分匹配（找到一个匹配的，即使后面来的加到现在的基础上还符合匹配规则都不要了）。
- SKIP_PAST_LAST_EVENT: 丢弃起始在这个匹配的开始和结束之间的所有部分匹配（保留最长的匹配）。
- SKIP_TO_FIRST: 丢弃在被触发的匹配的第一个事件映射到*PatternName*之前开始的每个部分匹配。
- SKIP_TO_LAST: 丢弃在映射到*PatternName*的触发匹配的最后一个事件之前开始的每个部分匹配。

例如，给定一个模式`b+ c`和一个数据流`b1 b2 b3 c`，不同跳过策略之间的不同如下：

| 跳过策略                 | 结果                          | 描述                                                         |
| :----------------------- | :---------------------------- | :----------------------------------------------------------- |
| **NO_SKIP**              | `b1 b2 b3 c` `b2 b3 c` `b3 c` | 找到匹配`b1 b2 b3 c`之后，不会丢弃任何结果。                 |
| **SKIP_TO_NEXT**         | `b1 b2 b3 c` `b2 b3 c` `b3 c` | 找到匹配`b1 b2 b3 c`之后，不会丢弃任何结果，因为没有以`b1`开始的其他匹配。 |
| **SKIP_PAST_LAST_EVENT** | `b1 b2 b3 c`                  | 找到匹配`b1 b2 b3 c`之后，会丢弃其他所有的部分匹配。         |
| **SKIP_TO_FIRST**[`b`]   | `b1 b2 b3 c` `b2 b3 c` `b3 c` | 找到匹配`b1 b2 b3 c`之后，会尝试丢弃所有在`b1`(**第一个匹配b+的**)之前开始的部分匹配，但没有这样的匹配，所以没有任何匹配被丢弃。 |
| **SKIP_TO_LAST**[`b`]    | `b1 b2 b3 c` `b3 c`           | 找到匹配`b1 b2 b3 c`之后，会尝试丢弃所有在`b3`（**最有一个匹配b+的**）之前开始的部分匹配，有一个这样的`b2 b3 c`被丢弃。 |

在看另外一个例子来说明NO_SKIP和SKIP_TO_FIRST之间的差别： 模式： `(a | b | c) (b | c) c+.greedy d`，输入：`a b c1 c2 c3 d`，结果将会是：

| 跳过策略                |                     结果                     |                             描述                             |
| :---------------------- | :------------------------------------------: | :----------------------------------------------------------: |
| **NO_SKIP**             | `a b c1 c2 c3 d` `b c1 c2 c3 d` `c1 c2 c3 d` |       找到匹配`a b c1 c2 c3 d`之后，不会丢弃任何结果。       |
| **SKIP_TO_FIRST**[`c*`] |        `a b c1 c2 c3 d` `c1 c2 c3 d`         | 找到匹配`a b c1 c2 c3 d`之后，会丢弃所有在`c1`（**第一个匹配c*的**）之前开始的部分匹配，有一个这样的`b c1 c2 c3 d`被丢弃。 |

为了更好的理解NO_SKIP和SKIP_TO_NEXT之间的差别，看下面的例子： 模式：`a b+`，输入：`a b1 b2 b3`，结果将会是：

| 跳过策略         |             结果              |                             描述                             |
| :--------------- | :---------------------------: | :----------------------------------------------------------: |
| **NO_SKIP**      | `a b1` `a b1 b2` `a b1 b2 b3` |            找到匹配`a b1`之后，不会丢弃任何结果。            |
| **SKIP_TO_NEXT** |            `a b1`             | 找到匹配`a b1`之后，会丢弃所有以`a`开始的部分匹配。这意味着不会产生`a b1 b2`和`a b1 b2 b3`了。 |

想指定要使用的跳过策略，只需要调用下面的方法创建`AfterMatchSkipStrategy`：

| 方法                                              |                          描述                          |
| :------------------------------------------------ | :----------------------------------------------------: |
| `AfterMatchSkipStrategy.noSkip()`                 |                  创建**NO_SKIP**策略                   |
| `AfterMatchSkipStrategy.skipToNext()`             |                创建**SKIP_TO_NEXT**策略                |
| `AfterMatchSkipStrategy.skipPastLastEvent()`      |            创建**SKIP_PAST_LAST_EVENT**策略            |
| `AfterMatchSkipStrategy.skipToFirst(patternName)` | 创建引用模式名称为*patternName*的**SKIP_TO_FIRST**策略 |
| `AfterMatchSkipStrategy.skipToLast(patternName)`  | 创建引用模式名称为*patternName*的**SKIP_TO_LAST**策略  |

```scala
val skipStrategy = ...
Pattern.begin("patternName", skipStrategy)
```

## 匹配

在指定了要寻找的模式后，该把它们应用到输入流上来发现可能的匹配了。为了在事件流上运行你的模式，需要创建一个`PatternStream`。 给定一个输入流`input`，一个模式`pattern`和一个可选的用来对使用事件时间时有**同样时间戳或者同时到达**的事件进行排序的比较器`comparator`， 你可以通过调用如下方法来创建`PatternStream`：

```scala
val input : DataStream[Event] = ...
val pattern : Pattern[Event, _] = ...
var comparator : EventComparator[Event] = ... // 可选的

val patternStream: PatternStream[Event] = CEP.pattern(input, pattern, comparator)
```

## 事件抽取

`PatternProcessFunction`有一个`processMatch`的方法在**每找到一个匹配的事件序列时都会被调用**。 它按照`Map<String, List<IN>>`的格式接收一个匹配，映射的键是你的模式序列中的每个模式的名称，值是被匹配的事件列表（`IN`是输入事件的类型）。 模式的输入事件按照时间戳进行排序。为每个模式返回一个匹配的事件列表的原因是当使用循环模式（比如`oneToMany()`和`times()`）时， 对一个模式会有不止一个事件被匹配到。

```java
class MyPatternProcessFunction<IN, OUT> extends PatternProcessFunction<IN, OUT> {
    @Override
    public void processMatch(Map<String, List<IN>> match, Context ctx, Collector<OUT> out) throws Exception;
        IN startEvent = match.get("start").get(0);
        IN endEvent = match.get("end").get(0);
        out.collect(OUT(startEvent, endEvent));
    }
}
```

### 超时

当一个模式上通过`within`加上窗口长度后，部分匹配的事件序列就可能因为超过窗口长度而被丢弃。可以使用`TimedOutPartialMatchHandler`接口 来处理超时的部分匹配。这个接口可以和其它的混合使用。也就是说你可以在自己的`PatternProcessFunction`里另外实现这个接口。 `TimedOutPartialMatchHandler`提供了另外的`processTimedOutMatch`方法，这个方法对每个超时的部分匹配都会调用。

```java
class MyPatternProcessFunction<IN, OUT> extends PatternProcessFunction<IN, OUT> implements TimedOutPartialMatchHandler<IN> {
    @Override
    public void processMatch(Map<String, List<IN>> match, Context ctx, Collector<OUT> out) throws Exception;
        ...
    }

    @Override
    public void processTimedOutMatch(Map<String, List<IN>> match, Context ctx) throws Exception;
        IN startEvent = match.get("start").get(0);
        ctx.output(outputTag, T(startEvent));
    }
}
```

