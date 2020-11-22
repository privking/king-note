# ArrayList

通过无参构造方法的方式ArrayList()初始化，则赋值底层数Object[] elementData为一个**默认空数组**Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {}所以数组容量为0，**只有真正对数据进行添加add时，才分配默认DEFAULT_CAPACITY = 10的初始容量**。

![image-20201120175032900](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201120175032900-1605865833-4175cd.png)

![image-20201120175131606](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201120175131606-1605865891-dbb715.png)

扩容时扩容为原来的3/2倍

![image-20201120175739880](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201120175739880-1605866259-3f018a.png)

ArrayList删除原理

![image-20201120180603719](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201120180603719-1605866763-c25a9c.png)

![image-20201120180637517](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201120180637517-1605866797-1a8ac9.png)

![image-20201120180712473](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201120180712473-1605866832-bb33c6.png)

指定位置添加

![image-20201120181028618](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201120181028618-1605867028-035c98.png)

![image-20201120181144846](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201120181144846-1605867104-74bb2d.png)

异常

![image-20201120181827992](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201120181827992-1605867508-617159.png)

![image-20201120181844865](https://raw.githubusercontent.com/privking/king-note-images/master/img/note/image-20201120181844865-1605867524-d8db5e.png)

即使初始化了数组，但是set时仍然会根据**实际大小 size** 来判断，所以会抛出异常