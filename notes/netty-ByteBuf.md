### ByteBuf
#### 1、netty中的ByteBuf概述
##### 1、ByteBuf简介：
> 1、是netty中的数据容器，类似一个字节数组，netty中的所有数据结构都是在ByteBuf上构建的。

> 2、ByteBuf有两个索引，读和写，可以按顺序的读取数据，也可以通过调整读取数据的索引或者直接将读取位置索引作为参数传递给get方法来重复读取数据。
>> 1、读和写索引符合以下规律：0 <= readerIndex <= writerIndex <= capacity(ByteBuf的大小)。     

区间 | 说明
---|---
0 <= readerIndex | 已读区间，可回收，discardReadBytes()方法可回收这部分内存
readerIndex <= writerIndex | 可读区间
writerIndex <= capacity|可写区间，不可读

> 3、discardReadBytes()方法：
>> 1、在执行该方法回收内存的时候，涉及到内存复制，需要将可读区间移动到该bytebuf的起始位置，且readIndex将被置为0，以便增大后面的可用空间。    
>> 2、这样的操作会影响性能，一般在需要马上释放内存的时候使用收益会比较大。  

> 4、ByteBuf的操作：

```
ByteBuf buf = Unpooled.copiedBuffer("Netty in Action rocks!", utf8);     //1
// 返回一个全新的数组，两个数组相互不会冲突
ByteBuf copy = buf.copy(0, 14);

// 返回一个内存共享的数组。
ByteBuf copy = buf.slice(0, 14);
```

> 3、方法：
>> 1、修改读写索引的操作：以 "read" 或 "write" 开头的任何方法都将自动增加相应的索引，如readInt     
>> 1、不修改读写索引的操作："set" 、 "get"操作字节将不会移动索引位置，只是设置或获取对应位置的值。

> 4、使用中的默认大小为Integer.MAX_VALUE，可以在添加长度解码器的时候设定

#### 2、netty中的ByteBuf有三种使用模式： 
> 1、堆内存：

> 2、直接内存：主要用于传输，可使用“零拷贝”加快数据传输

> 3、混合型：CompositeByteBuf 

```
ompositeByteBuf compBuf = Unpooled.compositeBuffer();
// 使用堆内存
ByteBuf heapBuf = Unpooled.buffer(8); 
// 使用直接内存
ByteBuf directBuf = Unpooled.directBuffer(16); 
//添加ByteBuf到CompositeByteBuf 
compBuf.addComponents(heapBuf,directBuf); 
//删除第一个ByteBuf 
compBuf.removeComponent(0); 
```
#### 3、ByteBuf释放：
> 1、ByteBuf是通过引用计数法来判断是否释放 ByteBuf及其相关的资源。

> 2、**谁负责释放， release()方法，在什么时候释放，一般是最后执行的对象负责释放，这么判断是否是最后一个对象？？？**
