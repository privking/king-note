# Scala简介

- Scala是一门以Java虚拟机（JVM）为运行环境并将**面向对象**和**函数式编程**的最佳特性结合在一起的**静态类型编程语言**。
- Scala是一门多范式的编程语言，Scala支持**面向对象和函数式编程**。
- Scala源代码（.scala）会被编译成Java字节码（.class），然后运行于JVM之上，**并可以调用现有的Java类库，实现两种语言的无缝对接。**
- Scala单作为一门语言来看，非常的**简洁高效。**
- Scala在设计时，马丁·奥德斯基是参考了Java的设计思想，可以说Scala是源于Java，同时马丁·奥德斯基也加入了自己的思想，将**函数式编程语言的特点融合到JAVA中** 因此，对于学习过Java的同学，只要在学习Scala的过程中，搞清楚Scala和Java相同点和不同点，就可以快速的掌握Scala这门语言。
- 任意语法结构都有返回值（while是Unit，赋值语句是Unit） **最后一行自动作为返回值** 

## scala和java以及jvm关系

![image-20201026164324385](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201026164324385-1603701811-1a7d89.png)



## Scala环境搭建

1. 确保jdk1.8安装成功

2. 下载scala以及scala-source

   ![image-20201026165726925](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201026165726925-1603702647-1723eb.png)

3. 配置环境变量

4. idea 安装scala插件

5. 创建maven项目

6. 在project右键add Framework Support
    ![image-20201026165451296](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201026165451296-1603702491-e54e83.png)

7. new scalaClass

8. Helloworld

## Scala注意事项

1）Scala源文件以“.scala" 为扩展名。

2）Scala程序的执行入口是object中的main()函数。或者继承App

3）Scala语言严格区分大小写。

4）Scala方法由一条条语句构成，每个语句后不需要分号（Scala语言会在每行后自动加分号）。（至简原则）

5）如果在同一行有多条语句，除了最后一条语句不需要分号，其它语句需要分号

6）Scala中如果使用object关键字声明类，在编译时，会同时生成两个类：当前类，当前类$

7）使用当前类$的目的在于模拟静态语法，可以通过类名直接访问方法。

8）Scala将当前类$这个对象称之为“伴生对象”，伴随着类所产生的对象，这个对象



