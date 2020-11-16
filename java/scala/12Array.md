# Array

## 定长数组

```scala
object ArrayDemo1 {

  def main(args: Array[String]): Unit = {
    val arr1 = Array[Int](1, 3, 5, 7, 9)
    println(arr1.mkString(","))

    val arr2 = arr1 :+ 10
    //相当于arr1.:+(10)
    //将元素10加在arr1的后面，返回新数组arr2
    //:在前面相当于是左结合，从左往右，即:+是arr1的方法
    println(arr2.mkString(","))


    val arr3 = 11 +: arr1
    //相当于arr1.+:(11)
    //将11加在arr1的前面,并返回新的数组arr3
    //:在后面相当于是右结合，从右往左，即+:是arr1的方法
    println(arr3.mkString(","))


    val arr4 = arr1 ++ arr2
    //相当于arr1.++(arr2)
    //将arr2所有元素加载arr1后面，并且返回新数组arr4
    println(arr4.mkString(","))
  }
}
```

```scala
object ArrayDemo2 {
  def main(args: Array[String]): Unit = {
    //创建长度为10的数组，每个元素默认为0
    val arr = Array[Int](10)
  }
}
```

## 可变数组

```scala
object ArrayDemo3 {
  def main(args: Array[String]): Unit = {
    val buffer = ArrayBuffer(1, 2, 3)
    val buffer2 = new ArrayBuffer[Int]()
    //还是生成新对象
    val buffer3 = buffer :+ 4
    println(buffer3)

    //在原对象上修改，不生成新对象
    //带 = 的一半用于可变集合
    buffer3 += 10
    100 +=: buffer3
    println(buffer3)

    //合并两个array，产生新对象
    val ints = buffer ++ buffer2

    //合并两个Array, 不产生新对象，合并到buffer上
    buffer ++= buffer2
    //合并到buffer2上，但是是追加在buffer2的前面
    buffer ++=: buffer2

    //删除元素，删除满足的第一个
    buffer -= 1

    //移除buffer1中buffer2有的元素
    buffer --=buffer2
  }
}
```

## 多维数组

```scala
object ArrayDemo4 {
  def main(args: Array[String]): Unit = {
    //多维数组
    //生成一个2*3的数组
    val array = Array.ofDim[Int](2,3)
    //遍历
    println(array(0)(1))
    for(a <-array){
      for(b<-a){
        println(b)
      }
    }
  }
}
```