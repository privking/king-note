# pz-main-overview

## 多数据源配置

![image-20201216094600749](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201216094600749-1608083169-71fe9a.png)

## PollutionSourceController/污染源tab&应急预案

### getPollutionSourceCountInfo/数量基本信息

包括三个数据来源：

- ADS_APPLICATION_PZ.ADS_ENTERPRISE_INFO（当前）
- 彭州餐饮认领小程序（当前）
-  ADS_APPLICATION_PZ.pollution_source_count_info（历史缓存数据）

**在拉取彭州餐饮认领小程序可能会出现问题** 配置不同环境下`rpc.adminEnterpriseClaimServer`，小程序后端地址

### PollutionSourceService.forTask()/定时任务

`pollutionSourceMapper.insertCountInfo(queryCountInfoNowByDate());`

保存基本数量信息

每天`00:00:01`执行一次

### createContingencyPlan/创建应急预案

- 应急预案名称，同名校验
- 默认未审核
- 更新图片信息

### queryContingencyPlan/查询应急预案

`mapper resultMap`级联查询应急预案以及文件信息

查询出结果后，遍历`FileInfo`,设置`viewOnlineUrl`，值为注入属性`onlineView.url`

**WARNING:该接口可能有bug,在分页处。前端查询列表偶尔有问题，不知道是不是后端问题**

```java
public PageInfo queryContingencyPlan(ContingencyPlanQueryer queryer) {
    PageHelper.startPage(queryer.getPage(), queryer.getPageSize());
    List<ContingencyPlanWrapper> contingencyPlanWrappers = contingencyPlanMapper.queryByPlan(queryer);
    PageInfo<ContingencyPlanWrapper> of = PageInfo.of(contingencyPlanWrappers);

    List<ContingencyPlanWrapper> collect = of.getList().stream().map(obj -> {
        List<FileInfo> fileInfos = obj.getFileInfos();
        List<FileInfo> info2 = fileInfos.stream().map(obj2 -> {
            obj2.setViewOnlineUrl(onlineViewUrl);
            return obj2;
        }).collect(Collectors.toList());
        obj.setFileInfos(info2);
        return obj;
    }).collect(Collectors.toList());
    of.setList(collect);
    return of;
}
```





# pz-gas-bigscreen

## VideoController/视频通话

### getSign/获取签名

用户传userId返回具体签名信息，userId好像是手机号，不重要，**没有特殊字符即可**

可以具体指定权限，但是在该应用中，放开了所有权限

具体签名逻辑拷贝自官方demo

### queryStationInfo/查询站点信息

基本上所有逻辑都是拷贝过来的

### queryGriderInfo/查询网格员信息

从`ADS_APPLICATION_PZ.pz_grider_info`拷贝出所有信息 -->**pzGriderInfos**

从redis查询缓存的视频通话准备的网格员（邀请了，没有进入房间的）-->**pzGriderInfos1**

从redis查询缓存的视频通话在线的网格员（正在通话的）-->**pzGriderInfos2**

遍历**pzGriderInfos** 匹配**pzGriderInfos1**，**pzGriderInfos2**设置网格员状态返回前端

### queryPerpareAccess/查询待进入房间列表

同上面**pzGriderInfos1**

### updateGriderStatus/更新网格员视频通话状态

在用户进入离开房间都会调用该方法

进入房间，从redis中带进入房间列表移除，新增到在线列表

退出房间，从在线列表移除

**如果小程序退出房间，存在不调用该接口的可能的话，网格员就会一直处于在线状态。**

### updateGriderInfo/更新网格员信息

更新网格员信息

**更新网格员经纬度**

### sendMessage/发送通知

直接发送短信，没有异步

发送短信短信成功后，回调往redis设置添加网格员待进入房间状态

**设置延迟任务**

### ScaduleTask/延迟任务类

该类`@PostConstruct`方法在类被加入单例工厂的时候，删除以前缓存的数据（考虑到会存在有重启过后，提交任务失效，那么网格员未进入房间就会一直处于待进入状态，但是如果是正常流程也可能会出现邀请了过后变成未邀请）并且开启一个有延迟功能的线程池。

该类目前只被发送通知的时候调用，redis set没找到单个member失效的方法，就开启延迟任务，**5分钟后手动删除网格员准备状态**。





# enterprise-fill-config

这个项目有点混乱，后台管理系统、小程序、pc填报端都在同一个项目中。

在controller中，Fill开头的controller基本上是后台管理系统相关

在controller中，App开头的controller是具体填报相关，多数为pc和小程序公用。

但是pc和小程序都有地方调用的是Fill开头的controller。

## 权限控制

后台管理系统，token前缀 `1_`

pc填报端，token前缀`2_`

小程序填报，token前缀`3_`

在过滤器中，放开了token  `1_xxxxxxxxxxxxx`,有一段时间做过免登录。后面可以删除

## FillEnterpriseController/填报信息

### queryFillEnterpriseInfoList/获取填报列表

### queryFillInformation/查看填报信息

级联查询，基本查询了所有相关的信息

前端根据有无数据进行处理

### generateFillData/解析填报信息

通过fillId,获取到所有企业的填报数据

主要解析过程在`parseJSON`方法

数据是两层包在两层数组下

反序列化可用数据，`FillData`

在`FillData`中可以获取到单个填报框的中文名，类型等，具体数据在`FillData`中的`Options`

`Options`中的的数据字段以及数据类型与填报框类型相关

所以定义`FillDataTypeEnum.getParser`枚举类对于不同的数据类型，获取不同的解析实现类

目前只支持几种

![image-20201216150356265](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201216150356265-1608102236-64e3ed.png)

拿到不同的解析器解析数据，最后把结果封装在`ParseResult`返回

解析完成后，将数据包装成`ParseInfo`保存到数据库

在`pageInfo`中`order`字段是指定顺序字段，比如日期范围，开始日期`order`为0，结束时间`order`为1

`multiLineIndex`字段是多行填报下标。

**后面如果需要新加解析方案，只需要实现`OptionsParse`,然后添加到`FillDataTypeEnum`中即可。**





## AppAccountController

负责填报端的用户注册，登录。

注册必须在微信小程序上注册，注册时发送短信验证手机号，并且绑定微信openId

注册后第二次免登录，校验的时候，拿到openId，返回用户信息以及token

注册后，可以在pc填报端通过手机号密码登录

**WARNING：目前没有接入短信，在发送短信的时候只是存了redis,通过临时接口getMessageCode查询验证码**





# dingding-server

主要分为三大部分

1. 定时从sqlServer缓存数据到redis
2. 从redis获取缓存数据
3. 除**工地机械**tab,其余都有手动缓存的接口CacheController

该程序主要问题在于，sqlServer库的写入并发很高，所以查询经常死锁

在程序中已经尽量在抛出异常后重试

但是还是有出现缓存失败的问题

**出现缓存失败目前的解决办法是手动触发缓存接口，缓存丢失的历史数据**





# wx-fill-manage

`EnterpriseFillController.getDropDownBox（）`获取下拉框方法可以进行优化，现在的逻辑是从`fill_manage_system.enterprise_fill_option`拿到所有的下拉框数据，然后每个type都去过滤一遍过滤出数据

可以根据根据type为key,collect成map,一次循环就解决所有事情，减少循环次数





# admin-enterprise-claim

## 导出Excel

在该系统中采用阿里巴巴开源 EasyExcel。

Head是一个`List<List<String>>`,如果需要合并，名称相同即可，内层List表示一行，外层List表示有多少行

Data是`List<List<Object>>` 内层List表示一行，外层List表示有多少行



## 新建任务

整体流程为:

- 传入任务类型，查询参数。
- 根据查询参数先直接查询一次，将企业包装好后，批量存储到任务详情表中（不存储的话查询，提交任务不能马上任务相关企业列表）
- 将查询的参数保存到redis
- 开启定时任务，每三分钟从redis拉取一次数据
- 然后再查询一次数据，根据查询结果发送短信

发送短信执行流程：

- 任务分为认领，填报清洗记录，填报监测报告
- 所以将查询出来的list分为：已认领，未认领，只没有填报监测报告，只没有填报清洗记录，两个都填报了，有油烟设备但是两个都没有填，以及没有油烟设备
- 最后根据勾选的是企业认领or油烟检测报告or设备清洗记录判断是否需要发送短信
- 发送短信成功后回调修改数据库
- 发送短信使用并行流

















