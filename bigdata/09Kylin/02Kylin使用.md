# Kylin使用

## 创建工程

点击下图中的"+"

![image-20201116162313113](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162313113-1605514993-f740f2.png)

填写项目名称和描述信息，并点击Submit按钮提交

![image-20201116162348114](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162348114-1605515028-6e889e.png)

## 获取数据源

点击DataSource

![image-20201116162423277](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162423277-1605515063-d41b06.png)

点击下图按钮导入Hive表

![image-20201116162434088](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162434088-1605515074-b83bbe.png)

选择所需数据表，并点击Sync按钮

![image-20201116162445482](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162445482-1605515085-5990a0.png)

## 创建model

点击Models，点击"+New"按钮，点击"★New Model"按钮

![image-20201116162522904](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162522904-1605515122-f77651.png)

填写Model信息，点击Next

![image-20201116162535192](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162535192-1605515135-0d56ef.png)

指定**事实表**

![image-20201116162556515](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162556515-1605515156-dd0b13.png)

选择**维度表**，并指定事实表和维度表的关联条件，点击Ok

![image-20201116162620396](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162620396-1605515180-f3a77b.png)

维度表添加完毕之后，点击Next

![image-20201116162645524](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162645524-1605515205-23039b.png)

指定**维度字段**，并点击Next

![image-20201116162703586](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162703586-1605515223-4bc528.png)

指定**度量字段**，并点击Next

![image-20201116162736602](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162736602-1605515256-92bfb7.png)

指定事实表分区字段（仅支持时间分区），点击Save按钮，model创建完毕

![image-20201116162804219](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162804219-1605515284-9318ee.png)

## 构建cube

点击new， 并点击new cube

![image-20201116162857215](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162857215-1605515337-107b52.png)

填写cube信息，选择cube所依赖的model，并点击next

![image-20201116162906908](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162906908-1605515346-b041b1.png)

选择所需的维度，如下图所示

![image-20201116162923323](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162923323-1605515363-8a2a80.png)

选择所需度量值，如下图所示

![image-20201116162935478](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162935478-1605515375-0d0075.png)

cube自动合并设置，cube需按照日期分区字段每天进行构建，每次构建的结果会保存在Hbase中的一张表内，为提高查询效率，需将每日的cube进行合并，此处可设置合并周期。

![image-20201116162957238](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116162957238-1605515397-d9de8b.png)

Kylin高级配置

![image-20201116163011383](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116163011383-1605515411-441742.png)

Kylin相关属性配置覆盖

![image-20201116163035147](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116163035147-1605515435-57de21.png)

Cube信息总览，点击Save，Cube创建完成

![image-20201116163058801](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116163058801-1605515458-726606.png)

构建Cube（计算），点击对应Cube的action按钮，选择build

![image-20201116163110538](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116163110538-1605515470-af3e32.png)

选择要构建的时间区间，点击Submit

![image-20201116163120226](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116163120226-1605515480-fe5a8f.png)

点击Monitor查看构建进度

![image-20201116163127568](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201116163127568-1605515487-123f84.png)

## 使用

构建完成后即可使用sql语句进行查询

可以使用jdbc,restful api等

```groff
jdbc:kylin://<hostname>:<port>/<kylin_project_name>

<dependency>
            <groupId>org.apache.kylin</groupId>
            <artifactId>kylin-jdbc</artifactId>
</dependency>

```



### api

**认证**

```groff
POST http://localhost:7070/kylin/api/user/authentication

Authorization:Basic xxxxJD124xxxGFxxxSDF
Content-Type: application/json;charset=UTF-8
```

 **获取Cube的详细信息**

```groff
GET http://localhost:7070/kylin/api/cubes?cubeName=test_kylin_cube_with_slr&limit=15&offset=0

Authorization:Basic xxxxJD124xxxGFxxxSDF
Content-Type: application/json;charset=UTF-8
```

**然后提交cube构建任务**

```groff
PUT http://localhost:7070/kylin/api/cubes/test_kylin_cube_with_slr/rebuild

Authorization:Basic xxxxJD124xxxGFxxxSDF
Content-Type: application/json;charset=UTF-8
    
{
    "startTime": 0,
    "endTime": 1388563200000,
    "buildType": "BUILD"        //BUILD 、 MERGE 或 REFRESH
}
```

**跟踪任务状态**

`GET http://localhost:7070/kylin/api/jobs/{job_uuid}`

