# Mybatis

```xml
<foreach collection="ids" item="item" open="(" separator="," close=")">
      #{item}
</foreach>

<if test="enterpriseName != null and enterpriseName != ''">
       enterprise_name like concat('%',#{enterpriseName},'%') and
</if>
<if test="rectInfo != null">
      where longitude <![CDATA[ <= ]]> #{rectInfo.maxLon} and longitude > #{rectInfo.minLon} and latitude <![CDATA[ <= ]]> #{rectInfo.maxLat} and latitude > #{rectInfo.minLat}
</if>

<resultMap id="queryByEnterpriseIdMap" type="xx.xx.xx.domain.entity.enterpriseFill.EnterpriseFillInfo">
	<id column="id" javaType="java.lang.Long" property="id"></id>
	<result column="enterprise_id" javaType="java.lang.Long" property="enterpriseId"></result>
	<result column="open_id" javaType="java.lang.String" property="openId"></result>
	<result column="lampblack_handle" javaType="java.lang.String" property="lampblackHandle"></result>
	<result column="create_date" javaType="java.util.Date" property="createDate"></result>
	<result column="update_date" javaType="java.util.Date" property="updateDate"></result>
	<association column="{fillId=id}" property="washInfoCount"   select="queryWashInfoCountByFillId"></association>
	<association column="{fillId=id}" property="monitorInfoCount"  select="queryMonitorInfoCountByFillId"></association>
</resultMap>

<resultMap id="queryWashInfoMap" type="xx.xx.xx.domain.vo.EnterpriseFill.EnterpriseEquipmentWashInfoWrapper">
	<id column="id" property="enterpriseEquipmentWashInfo.id"></id>
	<result column="fill_id"  property="enterpriseEquipmentWashInfo.fillId"></result>
	<result column="year_num"  property="enterpriseEquipmentWashInfo.yearNum"></result>
	<result column="month_num" property="enterpriseEquipmentWashInfo.monthNum"></result>
	<result column="create_date" javaType="java.util.Date"  property="enterpriseEquipmentWashInfo.createDate"></result>
	<result column="update_date" javaType="java.util.Date"  property="enterpriseEquipmentWashInfo.updateDate"></result>
	<collection property="fileInfos" column="{sourceId=id,sourceType=source_type}" javaType="ArrayList"
				ofType="xx.xx.xx.domain.entity.file.FileInfo" select="xx.xx.xx.mapper.UploadFileMapper.queryBySourceId">
	</collection>
</resultMap>


<!-- ===================================================================== -->
<!-- MAP嵌套 最多两层 -->
<resultMap id="systemInfoMap" type="xx.xx.filesystem.domain.entity.system.SystemInfo">
	<id column="id" javaType="java.lang.Long" property="id"></id>
	<result column="name" property="name"></result>
	<collection property="serverInfos" column="{systemId=id}" javaType="ArrayList"
				ofType="xx.xx.filesystem.domain.entity.system.ServerInfo" select="queryServerInfosBySystemId">
	</collection>
</resultMap>

<resultMap id="serverInfoMap" type="xx.xx.filesystem.domain.entity.system.ServerInfo">
	<id column="id" javaType="java.lang.Long" property="id"></id>
	<result column="system_id" property="systemId"></result>
	<result column="server_name" property="serverName"></result>
	<result column="server_ip" property="serverIp"></result>
	<collection property="userInfos" column="{serverId=id}" javaType="ArrayList"
				ofType="xx.xx.filesystem.domain.entity.system.UserInfo" select="queryUserInfosByServerId">
	</collection>
</resultMap>
<select id="queryAllSystemInfos" resultMap="systemInfoMap">
	select * from file_system_name
</select>

<select id="queryServerInfosBySystemId" resultMap="serverInfoMap">
	select id,system_id,server_name,server_ip from file_system_server where system_id = #{systemId}
</select>

<select id="queryUserInfosByServerId" resultType="xx.xx.filesystem.domain.entity.system.UserInfo">
	select id,server_id,user_account,user_password,user_name from file_system_user where server_id = #{serverId}
</select>
<!-- ===================================================================== -->

<select>
<!-- case-when使用方法 -->
<!-- 当colume 与condition 条件相等时结果为result -->
case colume 
    when condition then result
    when condition then result
    when condition then result
else result
end
<!--当满足某一条件时，执行某一result-->
case  
    when condition then result
    when condition then result
    when condition then result
else result
end
<!-- 当满足某一条件时，执行某一result,把该结果赋值到new_column_name 字段中 -->
case  
    when condition then result
    when condition then result
    when condition then result
else result
end new_column_name
</seelct>

<!-- ===================================================================== -->
<select>
<!-- -->
<!-- group_concat -->
<!-- group_concat([DISTINCT] 要连接的字段 [Order BY ASC/DESC 排序字段] [Separator '分隔符']) -->
<!--默认逗号分隔-->
select GROUP_CONCAT(id) from  (
SELECT id FROM wc_enterprise_info LIMIT 10 
) as t
<!-- 2,3,4,5,6,7,8,9,10,11-->	
select GROUP_CONCAT(id ORDER BY id desc) from  (
SELECT id FROM wc_enterprise_info LIMIT 10 
) as t
<!--11,10,9,8,7,6,5,4,3,2 -->
select GROUP_CONCAT(id SEPARATOR '-') from  (
SELECT id FROM wc_enterprise_info LIMIT 10 
) as t
<!--2-3-4-5-6-7-8-9-10-11 -->


<!--concat -->
<!--CONCAT(string1,string2,…) 只要其中一个是NULL,那么将返回NULL -->
 enterprise_name like concat('%',#{enterpriseName},'%') and
 
<!--CONCAT_WS--->
<!--CONCAT_WS(separator,str1,str2,...)-->
<!--
concat_ws 代表 concat with separator,第一个参数是其它参数的分隔符。
分隔符的位置放在要连接的两个字符串之间。
分隔符可以是一个字符串，也可以是其它参数。
如果分隔符为 NULL，则结果为 NULL。
函数会忽略任何分隔符参数后的 NULL 值。
-->
 select concat_ws('#','courseName=','NX',null) AS nx_courseName;
<!--
 +--------------+
| nx_courseName |
+---------------+
| courseName=#NX|
+---------------+
-->
<select>


<!-- in条件里面有多组参数查询方式 -->
<select>
SELECT
	id
FROM
	t3
WHERE
	(n1, n2) IN (
		SELECT
			n1,
			n2
		FROM
			t3
		WHERE
			id <= 2
	)
<!-- ===================== -->
SELECT
	id
FROM
	t3
WHERE
	(n1, n2) IN ((1, 'a'),(2, 'b'))
</select>

```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!-- 数据库驱动:选择你的本地硬盘上面的数据库驱动包
    C:\maven\apache-maven-3.6.1\repository\mysql\mysql-connector-java\8.0.18\mysql-connector-java-8.0.18.jar
    -->
    <classPathEntry location="C:\maven\apache-maven-3.6.1\repository\mysql\mysql-connector-java\8.0.18\mysql-connector-java-8.0.18.jar"/>
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <!-- JavaBean 实现 序列化 接口 -->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin" />
        <!-- 生成toString -->
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin" />
        <!-- optional，旨在创建class时，对注释进行控制 -->
        <commentGenerator>
            <property name="suppressDate" value="true"/>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>
        <!--数据库链接URL，用户名、密码 -->
        <jdbcConnection driverClass="com.mysql.cj.jdbc.Driver"
                        connectionURL="jdbc:mysql://x:3306/fruit_develop?useSSL=false&amp;serverTimezone=GMT&amp;generateSimpleParameterMetadata=true"
                        userId="root"
                        password="xxxxxx">

        </jdbcConnection>
        <!-- 类型转换 -->
        <javaTypeResolver >
            <!-- 是否使用bigDecimal,
                false: 把JDBC DECIMAL 和 NUMERIC 类型解析为 Integer(默认)
                true:  把JDBC DECIMAL 和 NUMERIC 类型解析为java.math.BigDecimal
            -->
            <property name="forceBigDecimals" value="false" />
        </javaTypeResolver>
        <!-- 生成模型的包名和位置-->
        <javaModelGenerator targetPackage="priv.king.server.dao" targetProject="src/main/java">
            <!-- 默认false 是否允许子包 -->
            <property name="enableSubPackages" value="true" />
            <!-- 默认false 是否对model添加 构造函数 -->
            <property name="constructorBased" value="false"/>
            <!-- 默认false 建立的Model对象是否 不可改变  即生成的Model对象不会有 setter方法，只有构造方法 -->
            <property name="immutable" value="false"/>
            <!-- 默认false 是否对类CHAR类型的列的数据进行trim操作 -->
            <property name="trimStrings" value="true" />
        </javaModelGenerator>
        <!-- 生成映射文件的包名和位置-->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>
        <!-- 生成DAO的包名和位置-->
        <javaClientGenerator type="XMLMAPPER" targetPackage="priv.king.server.mapper" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>
        <!-- 要生成的表 tableName是数据库中的表名或视图名 domainObjectName是实体类名-->
        <!-- <table tableName="risk_model_order" domainObjectName="DSRiskModelOrder" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
         <table tableName="tel_bill_record" domainObjectName="DSTelBillRecord" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>-->

        <!--<table tableName="message" domainObjectName="Message" enableCountByExample="true" enableUpdateByExample="true" enableDeleteByExample="true" enableSelectByExample="true" selectByExampleQueryId="true"></table>-->
        <!--<table tableName="user_info" domainObjectName="UserInfo" enableCountByExample="true" enableUpdateByExample="true" enableDeleteByExample="true" enableSelectByExample="true" selectByExampleQueryId="true"></table>-->
        <table tableName="user_to_user" domainObjectName="UserToUser" enableCountByExample="true" enableUpdateByExample="true" enableDeleteByExample="true" enableSelectByExample="true" selectByExampleQueryId="true"></table>



    </context>
</generatorConfiguration>

```

