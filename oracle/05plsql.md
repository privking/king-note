# PLSQL编程

PL/SQL：Procedure Language/SQL

PL/SQL是ORACLE对SQL语言的过程化扩展

在SQL命令中增加了过程处理语句（分支，循环等），使SQL语言具有过程处理能力

不区分大小写

语句结束加分号

## PL/SQL Developer中文乱码

查询出数据库使用的编码

```sql
select userenv('language') from dual
```

设置环境变量，变量key为NLS_LANG，value为查出的结果

## 程序结构

可以分为三个部分：声明部分、可执行部分、异常处理部分

如果不需要声明变量和游标，可以省略 DECLARE

```plsql
DECLARE
	--声明变量、游标
	I INTEGER
BEGIN
	--执行语句
	
	--[异常处理]
END
```

## HelloWorld

```plsql
-- Created on 2021/7/22 by ADMINISTRATOR 
DECLARE
  -- Local variables here
  I INTEGER;
BEGIN
  -- Test statements here
  DBMS_OUTPUT.PUT_LINE('hello');
END;
```

```
SQL> set serveroutput on
SQL> 
SQL> BEGIN
  2    -- Test statements here
  3    DBMS_OUTPUT.PUT_LINE('hello');
  4  END;
  5  /

hello
PL/SQL procedure successfully completed
```

## 变量

PL/SQL编程中常见变量分为两大类

1. 普通数据类型（char,varchar2,date,number.....）
2. 特殊类型变量(引用型变量，记录型变量)

### 普通变量

声明变量的方式为：

```
变量名 变量类型(变量长度)
例如：v_name varchar2(20)
```

变量赋值的两种方式：

1. 直接赋值语句    `:=`
2. 语句赋值，使用`select...into...`

```plsql
DECLARE
  --定义是直接赋值
  v_name VARCHAR2(20) := 'king';
  v_salary NUMBER;  
  v_address VARCHAR2(200);
  
BEGIN
  --直接赋值
  v_salary := 666666;
  
  --语句赋值
  SELECT '地址地址地址' INTO v_address FROM dual;
  --如果有多个变量赋值 位置对齐
  --SELECT ENAME,SAL INTO v_name,v_salary FROM EMP WHERE EMPNO = 8939
  
  DBMS_OUTPUT.PUT_LINE(v_name||'   '||v_salary||'    '||v_address );
END;
```

### 引用型变量

```plsql
DECLARE
  --应用型变量定义，和指定字段类型一致
  --表名.字段名%TYPE
  v_name emp.ename%TYPE;
  v_salary emp.sal%TYPE;

BEGIN
  --如果有多个变量赋值 位置对齐
  SELECT ENAME,SAL INTO v_name,v_salary FROM EMP WHERE EMPNO = 8939;
  
  DBMS_OUTPUT.PUT_LINE(v_name||'   '||v_salary|| );
END;
```

### 记录型变量

接受表中一行记录

```plsql
DECLARE

  --表名%ROWTYPE
  v_row emp%ROWTYPE;
BEGIN
  
  SELECT * INTO v_row FROM EMP WHERE EMPNO = 8939;
  
  DBMS_OUTPUT.PUT_LINE(v_row.v_name||'   '||v_row.v_salary);
END;
```

## 流程控制

### 条件分支

注意 ： 关键字 elsif 少了一个e

```PLSQL
BEGIN
	IF 条件1 THEN 
		执行1;
	ELSIF 条件2 THEN 
		执行2;
	ELSE 
		执行3;
	END IF; --注意这里有分号
END;

--=========================================

DECLARE

  ROW_COUNT NUMBER;

BEGIN

  SELECT COUNT(1) INTO ROW_COUNT FROM teble_test;

  IF ROW_COUNT >= 100 AND ROW_COUNT < 200 THEN
    DBMS_OUTPUT.PUT_LINE('大于100' || ROW_COUNT);
  ELSIF ROW_COUNT >= 200 AND ROW_COUNT < 300 THEN
    DBMS_OUTPUT.PUT_LINE('大于200');
  ELSE
    DBMS_OUTPUT.PUT_LINE('count: ' || ROW_COUNT);
  END IF;
END;
```

### 循环

#### do-while

```plsql
DECLARE
  I NUMBER := 1;
BEGIN
  LOOP
  
    DBMS_OUTPUT.PUT_LINE(I);
    I := I + 1;
    
    EXIT WHEN I > 10;
  END LOOP;
END;
```

#### for

```plsql

BEGIN
  FOR COUNTER IN 1 .. 5 LOOP
    DBMS_OUTPUT.PUT_LINE('for循环：' || COUNTER);
  END LOOP;
  
  --反转
  FOR COUNTER IN REVERSE 1 .. 5 LOOP
    DBMS_OUTPUT.PUT_LINE('for循环：' || COUNTER);
  END LOOP;
END;
```

#### while

```plsql
DECLARE
  I NUMBER(2);
BEGIN
  I := 1;
  WHILE I < 5 LOOP
    DBMS_OUTPUT.PUT_LINE('while循环：' || I);
    I := I + 1;
  END LOOP;
END;
```

## 游标

`https://blog.csdn.net/qq_32588349/article/details/51550125`

用于零食存储一个查询返回的多行数据，类似集合

使用方式 ：声明->打开->读取->关闭、

### 语法

**声明**

`CURSOR 游标名[(参数列表)]  IS 查询语句`

**游标的打开**

`OPEN 游标名`

**游标的取值**

`FETCH 游标名 INTO 变量列表`

**游标的关闭**

`CLOSE 游标名`

### 游标的属性

| 属性      | 解释                                                         |
| :-------- | ------------------------------------------------------------ |
| %FOUND    | 只有在DML语句影响一行或多行时，才返回true（集合还没有遍历完） |
| %NOTFOUND | 与%FOUND相反                                                 |
| %ROWCOUNT | 返回DML语句影响的行数，没有影响行数则返回0                   |
| %ISOPEN   | 游标打开时返回true,游标关闭后返回false.对于隐式游标来说，此属性一直为false,因为在SQL语句执行完后，Oracle会自动关闭游标 |

### 示例

```plsql
--检索EMP表中的所有JOB为MANAGER的雇员信息
DECLARE
  /*声明游标、(游标输入参数变量为VAR_JOB)可选项*/
  CURSOR CUR_EMP(VAR_JOB  VARCHAR2:='SALESMAN') IS
    /*游标所使用的查询语句*/
    SELECT EMPNO, ENAME, SAL FROM EMP WHERE JOB = VAR_JOB;
  /*声明一个RECORD类型的记录变量*/
  TYPE RECORD_EMP IS RECORD(
    VAR_EMPNO EMP.EMPNO%TYPE,
    VAR_ENAME EMP.ENAME%TYPE,
    VAR_SAL   EMP.SAL%TYPE);
    
  EMP_ROW RECORD_EMP;
BEGIN
  /*打开游标,指定输入参数值为MANAGER*/
  OPEN CUR_EMP('MANAGER');
  /*将游标指向结果集第一行数据并存入RECORD记录变量*/
  FETCH CUR_EMP INTO EMP_ROW;
  /*如果游标有数据就循环*/
  WHILE CUR_EMP%FOUND LOOP
    DBMS_OUTPUT.PUT_LINE('雇员编号：' || EMP_ROW.VAR_EMPNO || ' 雇员姓名：' ||
                         EMP_ROW.VAR_ENAME || ' 雇员薪水：' || EMP_ROW.VAR_SAL);
    /*将游标指向结果集下一条数据*/
    FETCH CUR_EMP INTO EMP_ROW;
  END LOOP;
  /*关闭游标*/
  CLOSE CUR_EMP;
END;
-------------------------------------------------------------------------
--另一种方式,使用%ROWTYPE类型
DECLARE
  CURSOR CUR_EMP IS
    /*使用FETCH+%ROWTYPE查询这里的查询字段必须和表中字段顺序及数量一致*/
    SELECT EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO FROM EMP;
  /*定义一个%ROWTYPE接收变量*/
  VAR_EMP_TYPE EMP%ROWTYPE;
BEGIN
  OPEN CUR_EMP;
  FETCH CUR_EMP INTO VAR_EMP_TYPE;
  WHILE CUR_EMP%FOUND LOOP
    DBMS_OUTPUT.PUT_LINE('雇员编号：' || VAR_EMP_TYPE.EMPNO || ' 雇员姓名：' ||
                         VAR_EMP_TYPE.ENAME || ' 雇员薪水：' ||
                         VAR_EMP_TYPE.SAL);
    FETCH CUR_EMP INTO VAR_EMP_TYPE;
  END LOOP;
  CLOSE CUR_EMP;
END;
-------------------------------------------------------------------------
--使用do-while方式
......
.....
OPEN cur_test;
  LOOP
    FETCH cur_test INTO V_MOBILE;
    EXIT WHEN cur_test%NOTFOUND;
    DBMS_OUTPUT.PUT_LINE(V_MOBILE);	
  END LOOP;
CLOSE cur_test;
-------------------------------------------------------------------------
--for 无需手动打开关闭游标
DECLARE
  CURSOR EMP_CURSOR IS SELECT * FROM EMP;
BEGIN
  /*使用for语句循环游标无需手动打开/关闭游标*/
  FOR V_EMP_RECORD IN EMP_CURSOR LOOP
    DBMS_OUTPUT.PUT_LINE(V_EMP_RECORD.ENAME);
  END LOOP;
END;

```

### 隐式游标

**PL/SQL为所有SQL数据操纵语句(包括返回一行的SELECT)隐士声明游标**,称为隐式游标的原因是用户不能直接命名和控制此类游标。当用户在PL/SQL中使用数据操纵语言(DML)时,Oracle**预先定义一个名为SQL的隐士游标**,通过检查隐式游标的属性可以获取与最近执行的SQL语句相关的信息。

```plsql
DECLARE

BEGIN
  UPDATE EMP SET SAL = SAL + 100 WHERE JOB = 'SALESMAN';
  IF SQL%NOTFOUND THEN
    DBMS_OUTPUT.PUT_LINE('没有符合条件的雇员');
  ELSE
    DBMS_OUTPUT.PUT_LINE('上调了：' || SQL%ROWCOUNT || '个雇员的工资');
  END IF;
END;

```

### 动态游标

静态游标在使用时,使用的查询语句已经确定,如果需要在**运行的时候动态的执行哪种查询**,可以使用REF游标(动态游标)和游标变量

定义动态游标 `type 游标类型名称 is ref cursor [return 游标返回值类型]`

打开动态游标 `open 游标变量名 for 查询语句`

**示例**

```plsql
--弱类型
--将薪水低于3000的雇员薪水增加500,增加后最高不超过3000
DECLARE
  TYPE REF_CURSOR_TYPE IS REF CURSOR; --声明一个弱类型的动态游标类型
  REF_CURSOR REF_CURSOR_TYPE; --定义一个游标为声明的弱类型游标
  V_SAL      EMP.SAL%TYPE;--声明一个变量用来接收雇员薪水
  V_EMPNO    EMP.EMPNO%TYPE;--声明一个变量用来接收雇员编号
BEGIN
  OPEN REF_CURSOR FOR
    SELECT SAL, EMPNO FROM EMP;--打开游标并指定使用的SQL语句
  LOOP
    FETCH REF_CURSOR
      INTO V_SAL, V_EMPNO;--将游标指向一行数据并给变量赋值
    EXIT WHEN REF_CURSOR%NOTFOUND;--当游标无数据时退出
    IF V_SAL < 3000 THEN--薪水低于3000进入IF,这个IF最后会执行更新SQL
      IF V_SAL >= 2500 THEN--薪水低于3000又大于2500进入这个IF
        V_SAL := 3000;
      ELSE
        V_SAL := V_SAL + 500;
      END IF;--结束一个IF,一个IF对应一个END IF；
      UPDATE EMP SET SAL = V_SAL WHERE EMPNO = V_EMPNO;--更新雇员薪水
    END IF;--结束最外层IF
  END LOOP;--结束LOOP循环
  CLOSE REF_CURSOR;--关闭游标
END;

--强类型
--查询所有雇员姓名
DECLARE
  TYPE EMP_REF_CURSOR IS REF CURSOR RETURN EMP%ROWTYPE;--声明一个强类型(指定返回类型)的动态游标
  REF_CURSOR   EMP_REF_CURSOR;--定义一个声明的强类型游标
  V_EMP_RECORD EMP%ROWTYPE;--定义一个接收变量
BEGIN
  OPEN REF_CURSOR FOR SELECT * FROM EMP;
  LOOP
    FETCH REF_CURSOR
      INTO V_EMP_RECORD;
    EXIT WHEN REF_CURSOR%NOTFOUND;
    DBMS_OUTPUT.PUT_LINE(V_EMP_RECORD.ENAME);
  END LOOP;
  CLOSE REF_CURSOR;
END;

```

## 存储过程

1.存储过程是用于特定操作的pl/sql语句块

2.存储过程是预编译过的，经优化后存储在sql内存中，使用时**无需再次编译，提高了使用效率**；

3.存储过程的代码直接存放在数据库中，一般直接通过存储过程的名称调用，**减少了网络流量**，加快了系统执行效率；



sqlplus执行存储过程 -- >`exec 存储过程名称`

### 语法

```plsql
CREATE[OR REPLACE]PROCEDURE procedure_name 

[(parameter1[model] datatype1, parameter2[model] datatype2..)]

IS[AS]

BEGIN

    PL/SQL;

END    [procedure_name];

--删除
DROPPROCEDURE procedure_name;
--编译
ALTER PROCEDURE procedure_name COMPILE
```

### demo

```plsql
CREATE OR REPLACE PROCEDURE PARA_PROCEDURE(PARA1 IN VARCHAR2(20),
                                           PARA2 OUT VARCHAR2(20)) IS
DECLARE  PARA3 VARCHAR2(20);
BEGIN
  PARA3 := PARA1 || 'xxetryertyertyxx';
  PARA2 := PARA3;
END;

-------------------------------------
DECLARE

  PARA2 VARCHAR2(100);

BEGIN
  PARA_PROCEDURE('123', PARA2);
  DBMS_OUTPUT.PUT_LINE(PARA2);
END;
```



## 事务

- 事务用于确保数据的一致性，有一组相关的DML语句组成，该组DML语句所执行的操作要么全部确认，要么全部取消。
- 当执行事务操作DML时，oracle会在被作用的表上加锁，以防止其他用户改变表结构，同时也会在被作用的行上加锁，以防止其他事务在该行上执行DML操作
- 当执行事务提交或者事务回滚时，oracle会确认事务变化或者回滚事务、结束事务、删除保存点、释放锁。
-  提交事务（commit）确认事务变化，结束当前事务、删除保存点，释放锁，使得当前事务中所有未决的数据永久改变
- 保存点（savepoint）在当前事务中，标记事务的保存点
- 回滚操作（rollback）回滚整个事务，删除该事务中所有保存点，释放锁，丢弃所有未决的数据改变
- ROLLBACK TO SAVEPOINT 回滚到指定的保存点

```plsql
--在存储过程中
CREATE OR REPLACE PROCEDURE trancPro
IS 
BEGIN
    INSERT INTO tab1 VALUES('AA','1212','1313');
    COMMIT;
    SAVEPOINT s1;
    INSERT INTO tab1  VALUES('BB','1414','1515');
    DBMS_TRANSACTION.SAVEPOINT('s2');
    UPDATE tab1 SET SNO='1515' WHERE ID='BB';
    COMMIT;
    EXCEPTION WHEN DUP_VAL_ON_INDEX THEN ROLLBACK TO SAVEPOINT s1;
    RAISE_APPLICATION_ERROR(-20010,'ERROR:违反唯一索引约束');
    WHEN OTHERS THEN ROLLBACK;
END trancPro;
```

## 异常

```plsql
　EXCEPTION 
　　WHEN exception_name THEN 
　　Code for handing exception_name 
　　[WHEN another_exception THEN 
　　Code for handing another_exception] 
　　[WHEN others THEN 
　　code for handing any other exception.] 
```

