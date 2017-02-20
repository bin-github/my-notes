## zero copy简介

#### 
>1、依赖于linux内核2.4版本之后sendfile（）方法实现

>2、主要针对静态文件（程序不需要对该文件做处理）

>3、文件拷贝步骤 的前后对比图：

>>1、一般的拷贝流程：

>
![image](https://github.com/bin-github/my-project/blob/master/files/images/fileCopy.jpg)

>>2、zero copy流程：

>
![image](https://github.com/bin-github/my-project/blob/master/files/images/zeroCopy.jpg)

>4、java实现zero copy的代码：在输入、输出流中获得channel

```
public void testSendfile(String fileName) throws IOException {
		String host = "127.0.0.1";
		int port = 8088;
		SocketAddress sa = new InetSocketAddress(host, port);
		SocketChannel sc = SocketChannel.open();
		sc.connect(sa);
		sc.configureBlocking(true);
		FileChannel fc = new FileInputStream(fileName).getChannel();
		long start = System.nanoTime();
		long nsent = 0, curnset = 0;
		curnset = fc.**transferTo**(0, fc.size(), sc);
		System.out.println("发送的总字节数:" + curnset
				+ " 耗时(ns):"
				+ (System.nanoTime() - start));
		try {
			sc.close();
			fc.close();
		} catch (IOException e) {
			System.out.println(e);
		}
	}
```
