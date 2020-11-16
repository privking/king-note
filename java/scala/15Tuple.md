# Tuple

```scala
object TupleDemo {
  def main(args: Array[String]): Unit = {
    //定义Tuple
    //最多可以Tuple22
    val tuple1 = Tuple2(1,"11")
    println(tuple1._1)
    println(tuple1._2)
    val tuple2 = Tuple3("1","2","3")

    //简化
    val tuple3 = (1,2,3)
    val tuple4 = 1-> 2


  }
}
```