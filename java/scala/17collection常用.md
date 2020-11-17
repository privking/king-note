# Collection常用

```scala
def main(args: Array[String]): Unit = {
  val l1 = List(1, 2, 3, 4)
  //第一个
  println(l1.head)
  //最后一个
  println(l1.last)
  //去掉第一个的list
  val tail = l1.tail
  //去掉最后一个
  val init = l1.init
  //求和
  l1.sum
  //最大值
  l1.max
  //最小值
  l1.min
  //乘积
  l1.product
  
  //获取长度
  l1.length
  l1.size
  //反转
  l1.reverse
  //获取前几个
  val take2 = l1.take(2)
  //获取后几个
  val takeRight = l1.takeRight(2)

  //获取满足条件的前几个，一旦不满足条件就返回
  val takeWhile = l1.takeWhile((i => i > 0))

  //抛弃前几个
  val drop = l1.drop(2)
  //丢弃满足条件的前几个，一旦不满足条件就返回
  l1.dropWhile((i => i > 0))

  //转换成字符串
  l1.mkString(",")
  l1.mkString("(", "-", ")")
  //获取迭代器
  val iterator = l1.iterator
  val iterator2 = l1.toIterator


}
```

## 将java的集合与scala的集合互转

```scala
object convertDemo {
  def main(args: Array[String]): Unit = {
    import scala.collection.JavaConverters._
    var list = new util.ArrayList[Int]()
    list.add(1)
    val l2 = list.asScala
    l2 += 1
    
    val list3 = l2.asJava
    list3.add(1)
    println(l2)
    println(list3)
  }
}
```

