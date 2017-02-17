## mysql常用函数小结：

#### 1.concat:连接多个变量为一个变量
>1.连接变量的值不能为空，如果有一个变量的值为null，连接的结果则为null

```
SELECT CONCAT('111','2222',null);
//结果
null

SELECT CONCAT('111','2222','33');
// 结果
111222233
```
#### 2.concat_ws：连接多个变量，并用分隔符分割多个变量
>1.连接之后的变量值之间使用分隔符分割，如果不想要分隔符，可将分隔符设为空

```
SELECT CONCAT_WS('.','11','22',NULL);
// 结果
11.22
SELECT CONCAT_WS('','11','22',NULL);
// 结果
1122
```
>2.由以上结果可知，CONCAT_WS函数和CONCAT函数不同的地方是：一个参数为空时不影响合成结果。

#### 3.cast方法和convert方法：修改字段类型
>1.cast：Cast(字段名 as 转换的类型 )，简洁

```
// 转换类型一般有这几种
CHAR[(N)] 字符型 
DATE  日期型
DATETIME  日期和时间型
DECIMAL  float型
SIGNED  int
TIME  时间型

```
>2.convert: Convert(转换类型,字段名，格式),功能和cast类似，更加灵活
```
// 102是一种日期格式的编号，
//（更多可参考该网页：https://msdn.microsoft.com/en-us/library/ms187928.aspx）
CONVERT(CHAR(10),CURRENT_TIMESTAMP,102)
```


