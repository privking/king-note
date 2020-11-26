# AOP
## 什么是AOP
* 与OOP对比，面向切面，传统的OOP开发中的代码逻辑是自上而下的，而这些过程会产生一些横切性问题，这些横切性的问题和我们的主业务逻辑关系不大，这些横切性问题不会影响到主逻辑实现的，但是会散落到代码的各个部分，难以维护。AOP是处理一些横切性问题，AOP的编程思想就是把这些问题和主业务逻辑分开，达到与主业务逻辑解耦的目的。使代码的重用性和开发效率更高

## 应用场景
1. 日志记录
1. 权限验证
1. 效率检查
1. 事务管理
1. exception

## 时期
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599203426-37338e.png)
## springAop和AspectJ的关系
* springAop、AspectJ都是Aop的实现，SpringAop有自己的语法，但是语法复杂，所以SpringAop借助了AspectJ的注解，但是底层实现还是自己的

## 概念
* aspect:切面 一定要给spring去管理
* pointcut:切点表示连接点的集合
* Joinpoint:连接点，目标对象中的方法
* Weaving :把代理逻辑加入到目标对象上的过程叫做织入
* target 目标对象 原始对象
* AOP proxy：动态代理或CGLIB
* advice:通知
    * Before：连接点执行之前，但是无法阻止连接点的正常执行，除非该段执行抛出异常
    * After：连接点正常执行之后，执行过程中正常执行返回退出，非异常退出
    * After throwing：执行抛出异常的时候
    * After (finally)：无论连接点是正常退出还是异常退出，都会执行
    * Around advice: 围绕连接点执行，例如方法调用。这是最有用的切面方式。around通知可以在方法调用之前和之后执行自定义行为。它还负责选择是继续加入点还是通过返回自己的返回值或抛出异常来快速建议的方法执行

## 快速使用aop


```xml
<dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.5</version>
        </dependency>
```

```java

@Configuration
@EnableAspectJAutoProxy
@ComponentScan("priv.king")
public class AppConfig {
}


@Component
@Aspect
public class MyAspect {
    @Pointcut("execution(* priv.king.service.IndexService.*(..))")
    public void pointCut(){

    }
    @Before("pointCut()")
    public void before(){
        System.out.println("before");
    }
}

```
## 切入点语法
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1606144333-6b3a61.png)
* execution：匹配方法执行连接点，粒度为方法级别
* within:将匹配限制为特定类型中的连接点，粒度为类级别
*  args：参数
*  target：目标对象
*  this：代理对象

### execution

```
execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)
这里问号表示当前项可以有也可以没有，其中各项的语义如下
modifiers-pattern：方法的可见性，如public，protected；
ret-type-pattern：方法的返回值类型，如int，void等；
declaring-type-pattern：方法所在类的全路径名，如com.spring.Aspect；
name-pattern：方法名类型，如buisinessService()；
param-pattern：方法的参数类型，如java.lang.String；
throws-pattern：方法抛出的异常类型，如java.lang.Exception；
example:
@Pointcut("execution(* com.chenss.dao.*.*(..))")//匹配com.chenss.dao包下的任意接口和类的任意方法
@Pointcut("execution(public * com.chenss.dao.*.*(..))")//匹配com.chenss.dao包下的任意接口和类的public方法
@Pointcut("execution(public * com.chenss.dao.*.*())")//匹配com.chenss.dao包下的任意接口和类的public 无方法参数的方法
@Pointcut("execution(* com.chenss.dao.*.*(java.lang.String, ..))")//匹配com.chenss.dao包下的任意接口和类的第一个参数为String类型的方法
@Pointcut("execution(* com.chenss.dao.*.*(java.lang.String))")//匹配com.chenss.dao包下的任意接口和类的只有一个参数，且参数为String类型的方法
@Pointcut("execution(* com.chenss.dao.*.*(java.lang.String))")//匹配com.chenss.dao包下的任意接口和类的只有一个参数，且参数为String类型的方法
@Pointcut("execution(public * *(..))")//匹配任意的public方法
@Pointcut("execution(* te*(..))")//匹配任意的以te开头的方法
@Pointcut("execution(* com.chenss.dao.IndexDao.*(..))")//匹配com.chenss.dao.IndexDao接口中任意的方法
@Pointcut("execution(* com.chenss.dao..*.*(..))")//匹配com.chenss.dao包及其子包中任意的方法
```

### within
```
表达式的最小粒度为类
// ------------
// within与execution相比，粒度更大，仅能实现到包和接口、类级别。而execution可以精确到方法的返回值，参数个数、修饰符、参数类型等
@Pointcut("within(com.chenss.dao.*)")//匹配com.chenss.dao包中的任意方法
@Pointcut("within(com.chenss.dao..*)")//匹配com.chenss.dao包及其子包中的任意方法
```
### args

```
args表达式的作用是匹配指定参数类型和指定参数数量的方法,与包名和类名无关


/**
 * args同execution不同的地方在于：
 * args匹配的是运行时传递给方法的参数类型
 * execution(* *(java.io.Serializable))匹配的是方法在声明时指定的方法参数类型。
 */
@Pointcut("args(java.io.Serializable)")//匹配运行时传递的参数类型为指定类型的、且参数个数和顺序匹配
@Pointcut("@args(com.chenss.anno.Chenss)")//接受一个参数，并且传递的参数的运行时类型具有@Classified
```
### target

```
/**
 * 此处需要注意的是，如果配置设置proxyTargetClass=false，或默认为false，则是用JDK代理，否则使用的是CGLIB代理
 * JDK代理的实现方式是基于接口实现，代理类继承Proxy，实现接口。
 * 而CGLIB继承被代理的类来实现。
 * 所以使用target会保证目标不变，关联对象不会受到这个设置的影响。
 * 但是使用this对象时，会根据该选项的设置，判断是否能找到对象。
 */
@Pointcut("target(com.chenss.dao.IndexDaoImpl)")//目标对象，也就是被代理的对象。限制目标对象为com.chenss.dao.IndexDaoImpl类
@Pointcut("this(com.chenss.dao.IndexDaoImpl)")//当前对象，也就是代理对象，代理对象时通过代理目标对象的方式获取新的对象，与原值并非一个
@Pointcut("@target(com.chenss.anno.Chenss)")//具有@Chenss的目标对象中的任意方法
@Pointcut("@within(com.chenss.anno.Chenss)")//等同于@targ
```
### @annotation

```
作用方法级别

上述所有表达式都有@ 比如@Target(里面是一个注解类xx,表示所有加了xx注解的类,和包名无关)

注意:上述所有的表达式可以混合使用,|| && !

@Pointcut("@annotation(com.chenss.anno.Chenss)")//匹配带有com.chenss.anno.Chenss注解的方法
```
### @args
* @within和@annotation分别表示匹配使用指定注解标注的类和标注的方法将会被匹配
* @args则表示使用指定注解标注的类作为某个方法的参数时该方法将会被匹配。

## 联合切点
* 可以使用&&,||,!联合切点

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599203464-c0fa99.png)

## JoinPoint && ProceedingJoinPoint

```java
import org.aspectj.lang.reflect.SourceLocation;  
public interface JoinPoint {  
   String toString();         //连接点所在位置的相关信息  
   String toShortString();     //连接点所在位置的简短相关信息  
   String toLongString();     //连接点所在位置的全部相关信息  
   Object getThis();         //返回AOP代理对象  
   Object getTarget();       //返回目标对象  
   Object[] getArgs();       //返回被通知方法参数列表  
   Signature getSignature();  //返回当前连接点签名  
   SourceLocation getSourceLocation();//返回连接点方法所在类文件中的位置  
   String getKind();        //连接点类型  
   StaticPart getStaticPart(); //返回连接点静态部分  
  }  
 
 public interface ProceedingJoinPoint extends JoinPoint {  
       public Object proceed() throws Throwable;  
       public Object proceed(Object[] args) throws Throwable;  
 }  
```

## Introductions(引入)
* 为代理对象引入一个接口

```java
@Aspect
@Component
public class SingerIntroducer {           
     @DeclareParents(value="com.mengxiang.concert.Performance+", 
                defaultImpl = BackSinger.class)
     public static Singer singer;
}
```
* value:指定被引入的类
* defaultImpl:默认接口实现
* 需要被引入的接口

## 多例
* 默认情况下切面的单例的

```
@Component
@Aspect("perthis(com.xyz.myapp.SystemArchitecture.businessService())")
@Scope("prototype")
public class MyAspect {

    private int someState;

    @Before(com.xyz.myapp.SystemArchitecture.businessService())
    public void recordServiceUsage() {
        // ...
    }

}
//假如切面内有一个成员变量‘someState’
//recordServiceUsage()切面会对变量修改
//那么如果需要someState每次都是原始值就需要多例
```
* 支持AspectJ的perthis和pertarget实例化模型(当前不支持percflow、percflowbelow和pertypewithin)
* 可以通过在@Aspect注释中指定perthis子句来声明perthis切面
* perthis表示如果某个类的代理类符合其指定的切面表达式，那么就会为每个符合条件的目标类都声明一个切面实例；
* pertarget表示如果某个目标类符合其指定的切面表达式，那么就会为每个符合条件的类声明一个切面实例。