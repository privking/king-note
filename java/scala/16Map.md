# Map

## 不可变

```scala
object MapDemo {
  def main(args: Array[String]): Unit = {
    //创建不可变Map
    //内部维护了元组
    val map1 = Map[String, Int](("1", 1), ("2", 2))
    val map2 = Map[String, Int]("a" -> 1, "b" -> 2)


    for (m <- map1) {
      println(m._1)
      println(m._2)
    }

    for ((k, v) <- map1) {
      //模式匹配
      println(k)
      println(v)
    }

    for ((k, 1) <- map1) {
      //只有k为1是才进循环
      println(k)
    }

    //添加元素
    //产生新Map
    val map3 = map1 + ("c" -> 3)

    //移除元素
    val map = map1 - "1"

    //把map2添加到map1
    val map4 = map1 ++ map2

    //获取value
    //找不到抛异常
    val value = map1("1")

    val opt = map1.get("1")
    val value2 = opt.get
    //常用
    val i = map1.getOrElse("1",1)

  }

}
```

## 可变

```scala
import scala.collection.mutable

object MapDemo2 {

  def main(args: Array[String]): Unit = {
    val map1 = mutable.Map[String, Int](("1", 1), ("2", 2))

    map1 += (("3", 3))
    map1 += ("4" -> 4)
    map1 += "4" -> 4
    map1 -= "4"

    //修改
    //如果不存在就新增
    map1("1")=100
    map1.update("1",11)

    //如果不存在就插入
    map1.getOrElseUpdate("5",5)

  }
}
```