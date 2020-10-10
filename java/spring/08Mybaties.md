# **MyBaties**
## MyBaties与spring整合
* 引入依赖 mybaties和mybaties-spring
* 配置datasource和sqlSessionFactoryBean
* MapperScan
### 整合原理
####  在mapperscan中import MapperScannerRegistrar
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204388-21d426.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204413-1c6b61.png)

####  new ClassPathMapperScanner 解析注解参数并调用scan

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204439-dc825f.png)

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204464-34fdc2.png)

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204516-503e07.png)

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204527-f539ca.png)

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204723-7c693d.png)

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599205669-cf2a73.png)

####  MapperFactoryBean

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204783-7b5c4e.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204797-a2967b.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204846-52aa50.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204863-9d7331.png)

#### MapperRegistry

![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204906-6e0327.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599204921-5389f9.png)

####  MapperProxy
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599205023-5b31f3.png)
#### MapperMethod
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599205038-8dbad5.png)
#### sqlSessionTemplate
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599205088-2e6074.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599205102-8d7210.png)
![clipboard](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/clipboard-1599205114-ef596d.png)