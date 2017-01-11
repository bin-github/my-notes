## java 类加载
**1、三个类加载器：**
- 启动类加载器：Bootstrap 加载System.getProperty("sun.boot.class.path")所指定的路径或jar(<JAVA_HOME>\lib,或者被-Xbootclasspath参数所指定的路径下的class)。

- 扩展类加载器：Extended Loader 加载System.getProperty("java.ext.dirs")所指定的路径或jar(<JAVA_HOME>\lib\ext目录下的，或者被java.ext.dirs系统变量所指定的路径中的所有类库加载)
- 系统类加载器：APPclass Loader 加载System.getProperty("java.class.path")所指定的路径或jar

**2、加载步骤：** 
- 装载：查找和导入class文件
- 链接: 把类的二进制数据合并到jre中；

     >1.校验：检查载入Class的文件数据的正确性

     >2.准备：给类的静态变量分配存储空间
      
     >3.解析：将符号引用转成直接引用
- 初始化：对类发静态变量，静态代码块执行初始化操作

     
**3、加载特点：**
-  运行一个程序时，首先是AppClass Loader加载器进行加载
-  加载时“子类懒加载”，所有的类加载器都交给父类加载器进行加载，父类中找不到的到子类加载器中加载
-  Bootstrap Loader 的父类加载器是null
