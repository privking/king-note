# Stream

## 串行流

```scala
object Demo1 {
  def main(args: Array[String]): Unit = {
    val l1 = List(1, 2, 3, 4, 9, 7, 5, 8)
    val stream = l1.toStream
    //stream是惰性操作
    //获取第一个
    println(stream.head)
    //获取第二个
    println(stream.tail.head)
    //此时已经有两个数据了
    println(stream)
    //Stream(1, 2, ?)

    //强制获取所有的
    println(stream.force)
    //从无限流获取
    //这里不加force 就只有第一项
    println(getSteam.take(10).force)

    //获取斐波那契数列
    println(fibSeq(10))
  }

  //无限流
  def getSteam:Stream[Int]={
    1 #:: getSteam
  }

  //斐波那契数列
  def fibSeq(n:Int):List[Int]={
    def loop(a:Int,b:Int) :Stream[Int]={
      a #::loop(a,a+b)
    }
    loop(1,1).take(n).force.toList
  }
  //斐波那契数列2
  def fibSeq2(n:Int):List[Int]={
    def loop() :Stream[Int]={
      1 #:: loop.scanLeft(1)(_+_)
    }
    loop.take(n).force.toList
  }
}
```

## 并行流

```scala
object Demo2 {
  def main(args: Array[String]): Unit = {
    val l1 = List(1, 2, 3, 4, 9, 7, 5, 8)
    //并行流
    l1.par.foreach(x=> println(Thread.currentThread().getName))
  }
}
```