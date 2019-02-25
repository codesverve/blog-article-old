# JVM堆栈

## 知识点

1. 堆内存默认新生代和老年代空间占比：1 : 2
2. 新生代中默认单个Survivor区和Eden区的空间占比：1 : 8
3. https://blog.csdn.net/leunging/article/details/80599282

## 参数

1. -Xms        堆内存初始大小（记忆方式，m以内存的单词memory记忆，s以small记忆，想象一下衣服的尺号就好记了），使用时：-Xms256m

2. -Xmx        堆内存最大大小（记忆方式同上，最后一个x以extra记忆，同样想象下衣服的加大号就好记了）

3. -Xmn        堆内存中新生代的最大内存和初始内存设为一致的值了，不再变化（记忆方式同上，n使用new记忆）

4. -XX:NewSize        堆内存中新生代的初始内存，使用时：-XX:NewSize=256m

5. -XX:MaxNewSize        堆内存中新生代的最大内存

6. -Xss(或-XX:ThreadStackSize)        每个线程的栈内存的大小（默认1M），该数值影响可以启动的线程的数量，也影响每个线程的栈帧数量（记忆方式，Stack Size）

7. -XX:+PrintGC(别名-verbose:gc)        打印gc的信息

8. -XX:+PrintGCTimeStamps        以基准时间的形式打印GC的时间戳

   不开启时打印如下

   ```
   [GC (Allocation Failure)  155644K->36463K(402432K), 0.0095270 secs]
   ```

   开启时打印如下

   ```
   4.609: [GC (Allocation Failure)  162642K->34898K(396800K), 0.0105201 secs]
   ```

5. -XX:+PrintGCDateStamps        以日期的形式打印GC时间戳（可以设置日期形式和基准形式同时打印）

   开启时打印如下

   ```
   2019-02-25T09:36:33.424+0800: [GC (Allocation Failure)  155673K->36616K(397824K), 0.0084967 secs]
   ```

   同时开启时打印如下

   ```
   2019-02-25T09:38:16.009+0800: 6.488: [GC (Allocation Failure)  169212K->42272K(518656K), 0.0163488 secs]
   ```

6. -XX:+PrintGCDetails        打印gc的详细信息，启动脚本下运行会自动开启-XX:+PrintGC，jinfo附加下运行不会自动开启

   开启时打印如下

   ```
   [GC (Allocation Failure) [PSYoungGen: 137517K->9963K(147456K)] 169528K->41982K(534528K), 0.0124863 secs] [Times: user=0.04 sys=0.00, real=0.01 secs] 
   ```

7. -XX:+PrintHeapAtGC        在进行GC的前后打印出堆的信息

   开启时打印如下

   ```
   {Heap before GC invocations=1 (full 0):
    PSYoungGen      total 147456K, used 131072K [0x00000000f6000000, 0x0000000100000000, 0x0000000100000000)
     eden space 131072K, 100% used [0x00000000f6000000,0x00000000fe000000,0x00000000fe000000)
     from space 16384K, 0% used [0x00000000ff000000,0x00000000ff000000,0x0000000100000000)
     to   space 16384K, 0% used [0x00000000fe000000,0x00000000fe000000,0x00000000ff000000)
    ParOldGen       total 163840K, used 0K [0x0000000087e00000, 0x0000000091e00000, 0x00000000f6000000)
     object space 163840K, 0% used [0x0000000087e00000,0x0000000087e00000,0x0000000091e00000)
    Metaspace       used 14008K, capacity 14230K, committed 14464K, reserved 1062912K
     class space    used 1295K, capacity 1356K, committed 1408K, reserved 1048576K
   [GC (Allocation Failure)  131072K->17089K(311296K), 0.0130938 secs]
   Heap after GC invocations=1 (full 0):
    PSYoungGen      total 147456K, used 16366K [0x00000000f6000000, 0x0000000100000000, 0x0000000100000000)
     eden space 131072K, 0% used [0x00000000f6000000,0x00000000f6000000,0x00000000fe000000)
     from space 16384K, 99% used [0x00000000fe000000,0x00000000feffb9b0,0x00000000ff000000)
     to   space 16384K, 0% used [0x00000000ff000000,0x00000000ff000000,0x0000000100000000)
    ParOldGen       total 163840K, used 723K [0x0000000087e00000, 0x0000000091e00000, 0x00000000f6000000)
     object space 163840K, 0% used [0x0000000087e00000,0x0000000087eb4d20,0x0000000091e00000)
    Metaspace       used 14008K, capacity 14230K, committed 14464K, reserved 1062912K
     class space    used 1295K, capacity 1356K, committed 1408K, reserved 1048576K
   }
   ```

8. -Xloggc:/var/log/java/gc.log        指定GC日志的输出文件

9. -XX:SurvivorRatio        Eden区和Survivor区的内存比值

10. -XX:NewRatio        老年代和新生代的内存比值


