# Set

## 不可变

```scala
object Set1 {

  def main(args: Array[String]): Unit = {
    //不可变set
    val set1 = Set(1,2,3,4,5,6,1)
    val set2 = Set(9,8,7,6,5,4)

    //并集
    val set3 = set1 ++ set2
    val set4 = set1 | set2
    val set5 = set1 union set2

    //交集
    val set6 = set1 & set2
    val set7 = set1 intersect set2

    //差集
    val set8 = set1 &~ set2
    val set9 = set1 -- set2
    val set10 = set1 diff set2
  }
}
```

## 可变

```scala
import scala.collection.mutable
object Set2 {
  def main(args: Array[String]): Unit = {
    //创建可变Set
    val set1 = mutable.Set(1,2,3)
  }
}
```

