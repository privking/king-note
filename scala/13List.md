# List 链表

## 不可变

```scala
object List1 {

  def main(args: Array[String]): Unit = {
    //创建不可变list
    //scala.collection.immutable.List
    val l1 = List(1,2,3)
    println(l1)
    //创建空List
    val l2 = List[Int]()
    val l3 = Nil
    val ll3 = ::[Int](1,1::Nil) // 创建一个List 第一个参数是Head ,第二个是List

    //先List添加元素
    //添加到尾部 产生新List
    val l4 = l1:+10

    //List专用
    //往List头部添加元素
    //List要放后面
    val l5 = 100 :: l4

    //合并集合
    val l6 = l1:::l2
    //常规合并
    val l7 = l1++l2
    
  }
}
```

## 可变

```scala
object List2 {
  def main(args: Array[String]): Unit = {
    //可变List
    val l1 = ListBuffer(1,2,3)
    val l2 = ListBuffer[Int]()

    //添加元素
    l2 +=10
    l1 ++=l2
  }
}
```