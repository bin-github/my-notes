#### 1、垃圾回收器简介：
##### 1、cms：
> 1、老年代的并发垃圾收集器，和年轻代的收集器serial和ParNew 配合使用

> 2、使用标记算法，

> 3、多线程并发，对cpu敏感，在CPU较少时发生cpu抢占，线程数= （CPU数 + 3）/ 4

> 4、标记算法产生内存碎片，使用-XX:CMSFullGCsBeforeCompaction = n 参数在多少次回收之后进行一次内存整理，默认为0，即每次都要整理，使得cms退化成标记-清理算法的实现

> 5、并发收集，停顿时间短，但是吞吐量不好，适合低吞吐量高响应时间的系统。

> 6、-XX:+UseCMSInitiatingOccupancyOnly，只有当old代占用确实达到了-XX:CMSInitiatingOccupancyFraction参数所设定的比例时才会触发cms gc；如果不设置UseCMSInitiatingOccupancyOnly，系统会根据统计自行执行gc，不一定达到设置的比例也会执行。

> 7、CMS-concurrent-abortable-preclean阶段说明：
>> 1、加入此阶段的目的是使cms gc更加可控一些，作用也是执行一些预清理，以减少Rescan阶段造成应用暂停的时间     
>> 2、-XX:CMSMaxAbortablePrecleanTime：当abortable-preclean阶段执行达到这个时间时才会结束
     -XX:CMSScheduleRemarkEdenSizeThreshold（默认2m）：控制abortable-preclean开始执行，
      即当eden使用达到此值时，才会开始abortable-preclean阶段
     -XX:CMSScheduleRemarkEdenPenetratio（默认50%）：控制abortable-preclean结束执行。

> 8、full gc:
>> 1、发生concurrent-mode-failure：cms时，old代的空间不足以给新用户新晋级的对象分配内存
>>> 1、在old代占用比例较少时触发gc：-XX:CMSInitiatingOccupancyFraction=60（默认68），要想一定按照该比例收集，必须要配合-XX:+UseCMSInitiatingOccupancyOnly该开关参数，否则是系统自动调整的。      
>>> 2、修改-XX:CMSMaxAbortablePrecleanTime=500，缩小CMS-concurrent-abortable-preclean阶段的时间

>> 2、promotion-failed：old代的空间不足以分配给年轻代晋级的对象，有可能是内存碎片太多导致的。
>>> 1、整理内存碎片：使用-XX:+UseCMSCompactAtFullCollection
       （cms gc后会进行内存的compact）或-XX:CMSFullGCsBeforeCompaction=4（在full gc 4次后会进行compact）参数进行调整

> 9、-XX:+CMSScavengeBeforeRemark，在执行重新标记阶段之前执行一次ygc，以防止年轻代和年老代有跨代依赖的对象，gc无法清除。
##### 2、G1：

#### 2、垃圾回收器的组合
组合 | 新生代| 年老代| 简介
---|---|---|---
组合1|Serial：标记-整理 | Serial Old\CMS（标记）|Serial和Serial Old都是单线程进行GC,CMS（Concurrent Mark Sweep）是并发GC，实现GC线程和应用线程并发工作，不需要暂停所有应用线程。另外，当CMS进行GC失败时，会自动使用Serial Old策略进行GC。
组合2|ParNew 复制 | Serial Old\CMS|1、ParNew是Serial的多线程版本，-XX:ParallelGCThreads选项指定GC的线程数。2、新生代使用ParNew GC策略，年老代默认使用Serial Old GC策略
组合3|Parallel Scavenge 复制| Serial Old\Parallel Old（标记-整理）|1、Parallel Scavenge策略主要是关注一个可控的吞吐量，2、Parallel Old是Serial Old的并行版本
组合4|G1 | G1|主要设置参数如下所示
```
-XX:+UnlockExperimentalVMOptions -XX:+UseG1GC        #开启
-XX:MaxGCPauseMillis =50                  #暂停时间目标
-XX:GCPauseIntervalMillis =200          #暂停间隔目标
-XX:+G1YoungGenSize=512m            #年轻代大小
-XX:SurvivorRatio=6
```



注：
> 1、cms在运行时，预留的内存不够用户现场使用时cms发生Concurrent Mode Failure，收集失败，此时系统会启用Serial Old收集器进行回收，gc时间变长。




#### 2、示例：
> 1、CMS回收实例
```
/**
 * @Date 2017/12/06 21:30
 * @description: 虚拟机参数 -Xms10m -Xmx10m -Xmn3m -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps
 **/
 // 参照网上的一种实现
public class TestCMSGC {
    public byte[] placeHolder = new byte[64 * 1024]; //占位符

    public static void main(String[] args) throws Exception {
        outOfMemoryByExpansionSize();
    }

    private static void outOfMemoryByExpansionSize() throws Exception {
        List<TestCMSGC> list = new ArrayList<TestCMSGC>();
        while (true) {
            TestCMSGC serial = new TestCMSGC();
            list.add(serial);
            Thread.sleep(10);//停顿10毫秒
        }
    }
}

初始标记（CMS initial mark）    stop the Word

并发标记（CMS concurrent mark） 

并发预清理（CMS-concurrent-preclean）

可控的并发预清理（CMS-concurrent-abortable-preclean 加入此阶段的目的是使cmsgc更加可控一些，作用也是执行一些预清理，以减少Rescan阶段造成应用暂停的时间）

重新标记（CMS remark）     stop the Word 主要的停留时间都在这里

并发清除（CMS concurrent sweep） 

并发重置（CMS-concurrent-reset） 

执行结果：
0.149: [GC (Allocation Failure) 0.149: [ParNew: 2505K->256K(2816K), 0.0013358 secs] 2505K->845K(9984K), 0.0013814 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
3.783: [GC (Allocation Failure) 3.783: [ParNew: 2794K->243K(2816K), 0.0035609 secs] 3383K->3390K(9984K), 0.0036466 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
7.714: [GC (Allocation Failure) 7.714: [ParNew: 2787K->211K(2816K), 0.0015026 secs] 5935K->5727K(9984K), 0.0015748 secs] [Times: user=0.06 sys=0.00, real=0.00 secs] 
7.715: [GC (CMS Initial Mark) [1 CMS-initial-mark: 5516K(7168K)] 5791K(9984K), 0.0001816 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
7.716: [CMS-concurrent-mark-start]
7.717: [CMS-concurrent-mark: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
7.717: [CMS-concurrent-preclean-start]
7.717: [CMS-concurrent-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
7.717: [GC (CMS Final Remark) [YG occupancy: 275 K (2816 K)]7.717: [Rescan (parallel) , 0.0001389 secs]7.717: [weak refs processing, 0.0000340 secs]7.717: [class unloading, 0.0003635 secs]7.718: [scrub symbol table, 0.0005848 secs]7.718: [scrub string table, 0.0001857 secs][1 CMS-remark: 5516K(7168K)] 5791K(9984K), 0.0014051 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
7.719: [CMS-concurrent-sweep-start]
7.720: [CMS-concurrent-sweep: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
7.720: [CMS-concurrent-reset-start]
7.720: [CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
9.721: [GC (CMS Initial Mark) [1 CMS-initial-mark: 5515K(7168K)] 7055K(9984K), 0.0003574 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
9.721: [CMS-concurrent-mark-start]
9.723: [CMS-concurrent-mark: 0.002/0.002 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
9.723: [CMS-concurrent-preclean-start]
9.724: [CMS-concurrent-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
9.724: [GC (CMS Final Remark) [YG occupancy: 1540 K (2816 K)]9.724: [Rescan (parallel) , 0.0003285 secs]9.725: [weak refs processing, 0.0000321 secs]9.725: [class unloading, 0.0004879 secs]9.725: [scrub symbol table, 0.0010269 secs]9.726: [scrub string table, 0.0003554 secs][1 CMS-remark: 5515K(7168K)] 7055K(9984K), 0.0023986 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
9.727: [CMS-concurrent-sweep-start]
9.728: [CMS-concurrent-sweep: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
9.728: [CMS-concurrent-reset-start]
9.728: [CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
11.539: [GC (Allocation Failure) 11.539: [ParNew: 2743K->2743K(2816K), 0.0000417 secs]11.539: [CMS: 5433K->7135K(7168K), 0.0085997 secs] 8176K->8004K(9984K), [Metaspace: 3500K->3500K(1056768K)], 0.0087508 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 
11.548: [GC (CMS Initial Mark) [1 CMS-initial-mark: 7135K(7168K)] 8068K(9984K), 0.0002775 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
11.548: [CMS-concurrent-mark-start]
11.550: [CMS-concurrent-mark: 0.002/0.002 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
11.550: [CMS-concurrent-preclean-start]
11.551: [CMS-concurrent-preclean: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
11.551: [GC (CMS Final Remark) [YG occupancy: 933 K (2816 K)]11.551: [Rescan (parallel) , 0.0001726 secs]11.551: [weak refs processing, 0.0000212 secs]11.552: [class unloading, 0.0003914 secs]11.552: [scrub symbol table, 0.0008222 secs]11.553: [scrub string table, 0.0002801 secs][1 CMS-remark: 7135K(7168K)] 8068K(9984K), 0.0018247 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
11.553: [CMS-concurrent-sweep-start]
11.554: [CMS-concurrent-sweep: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
11.554: [CMS-concurrent-reset-start]
11.554: [CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
13.554: [GC (CMS Initial Mark) [1 CMS-initial-mark: 7135K(7168K)] 9333K(9984K), 0.0003519 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
13.554: [CMS-concurrent-mark-start]
13.557: [CMS-concurrent-mark: 0.002/0.002 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
13.557: [CMS-concurrent-preclean-start]
13.557: [CMS-concurrent-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
13.557: [CMS-concurrent-abortable-preclean-start]
13.557: [CMS-concurrent-abortable-preclean: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
13.557: [GC (CMS Final Remark) [YG occupancy: 2197 K (2816 K)]13.557: [Rescan (parallel) , 0.0003644 secs]13.557: [weak refs processing, 0.0000263 secs]13.557: [class unloading, 0.0004658 secs]13.558: [scrub symbol table, 0.0010051 secs]13.559: [scrub string table, 0.0003481 secs][1 CMS-remark: 7135K(7168K)] 9333K(9984K), 0.0023547 secs] [Times: user=0.02 sys=0.00, real=0.00 secs] 
13.560: [CMS-concurrent-sweep-start]
13.560: [CMS-concurrent-sweep: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
13.560: [CMS-concurrent-reset-start]
13.560: [CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 



```
##### 2、新生代回收示例：
> 1、gc后的新生代比gc前大：
>> 1、原因：gc后存活的对象太多，在移动了一部分对象到to区后，to区无法容纳这些对象，old区内存担保失败，old区也无法容纳，此时对所有新生代区的对象都不清理，再将from区的对象移动到to区，to区会有一些重复对象，这就导致了gc后新生代反而增大了。

```
// 使用上例中的代码，将jvm参数修改为：-Xms10m -Xmx10m -Xmn8m -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps
 并修改placeHolder = new byte[1 * 1024 * 1024]

"C:\Program Files\Java\jdk1.8.0_121\bin\java" -Xms10m -Xmx10m -Xmn9m -XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps "-javaagent:D:\Softer\idea\IntelliJ IDEA Community Edition 2017.1.4\lib\idea_rt.jar=53595:D:\Softer\idea\IntelliJ IDEA Community 
0.526: [GC (Allocation Failure) 0.526: [ParNew (promotion failed): 6559K->7369K(8320K), 0.0009775 secs]0.527: [CMS: 2K->0K(1024K), 0.0032892 secs] 6559K->4874K(9344K), [Metaspace: 3500K->3500K(1056768K)], 0.0043475 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
0.734: [GC (Allocation Failure) 0.734: [ParNew (promotion failed): 7063K->7959K(8320K), 0.0008851 secs]0.735: [CMS: 5K->1K(1024K), 0.0028263 secs] 7063K->6892K(9344K), [Metaspace: 3501K->3501K(1056768K)], 0.0037964 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
0.737: [Full GC (Allocation Failure) 0.737: [CMS: 1K->1K(1024K), 0.0021763 secs] 6892K->6873K(9344K), [Metaspace: 3501K->3501K(1056768K)], 0.0022129 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
```
