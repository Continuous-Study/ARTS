# ARTS

## 0629-0705

每周寄语: 面试造火箭，工作拧螺丝。不要让自己变成只会拧螺丝的，只有更多的技术储备，才能在机会来临时时抓住。

### A.2

// TODO

### R.2

最近有人问 java dump 出来的文件怎么看？才忽然发现自己并不会。所以这周就研究研究 java 的一些工具

结论在末尾哦！

#### 1. GUI 工具

##### 1.1 jconsole

1. 实时监控
1. 可远程：1. jmx 连接 2. jstatd 连接
1. 没法看 dump
1. 据说：遇见大一点的程序容易挂

PS：

1. 开端口，隔着一个堡垒机应该越不过去；线上服务一般也不开 jmx。JMX server 指定的监听端口号外，JMXserver 还会监听一到两个随机端口号，可以通过命令：lsof -i|grep java |grep <pid> 来查看当前 java 进程需要监听的随机端口号，再配置 firewall。基本上生产环境不可行。
1. jstatd 也不太好整，参考这个 https://juejin.im/post/5d6e25ccf265da039e12d88c
1. 一般查问题，是将内存 dump 出来一份，看 dump

##### 1.2 jvisualvm

官方中文指南 https://visualvm.github.io/gettingstarted.html

1. 实时监控
1. 可远程：1. jmx 连接 2. jstatd 连接，问题同上
1. 可以看 dump
1. 可以看：jvm 参数(-D)、cpu、对、类、线程、cpu 采样(按方法、线程)、内存采样(按类、线程)、profiler。采样 & profiler应该会对性能有影响，不要在线上使用。

PS：jconsole、jvisualvm 比较

1. jconsole、jvisualvm 是 jmap、jstack、jstat、jhat 的整合 gui 工具
1. jvisualvm 要新一点(1.6+)，jconsole(1.5+)
1. jconsole 不能看 dump 文件
1. jconsole 可以观测堆的不同区块的占用空间量（优于 jvisualvm 默认功能，）
1. jvisualvm oracle 不维护了，社区维护 https://visualvm.github.io/。菜单 - 工具 - 插件 安装插件，比如 btrace、VisualGC、Buffer Monitor

##### 1.3 jprofiler

商业收费工具，没用过，别人的讲解

* https://blog.csdn.net/u013613428/article/details/53926825
* https://segmentfault.com/a/1190000017795841

基于 javaagent 实现，类似的有 arthas、btrace

想了解 javaagent 可以看 S.1

##### 1.4 mat

Eclipse Memory Analyzer，http://www.eclipse.org/mat/，貌似只支持 Eclipse？

Eclipse 提供的一个用于分析 JVM 堆 Dump 文件的插件。借助这个插件可查看对象的内存占用状况，引用关系，分析内存泄露等。

#### 2. 命令行工具

##### 2.1 jmap

看内存

* `jmap -heap <pid>`       查看 java 程序的堆的概要信息。
* `jmap -histo <pid>`         按类型输出堆使用的柱状图。
* `jmap -dump:format=b,file=/path/to/heamdump.hprof <pid>`       将堆的内容转储到二进制文件中以便使用 jhat 进行分析。

:live 参数只统计活对象，将会触发 fullgc 以提供此信息

* `jmap -histo:live <pid>`
* `jmap -dump:live,format=b,file=/path/to/heamdump.hprof <pid>`

PS：在 mac 上使用 jmap 可能会出现 `Error attaching to process: sun.jvm.hotspot.debugger.DebuggerException: Can't attach symbolicator to the process`，但用 jconsole、jvisualvm 就没事，有查到这个 bug https://bugs.java.com/bugdatabase/view_bug.do?bug_id=8160376，没找到解决方案

PS1：使用 CMS GC 情况下，jmap -heap 的执行有可能会导致 JAVA 进程挂起

PS2：有人反馈 jmap -histo 不带 live 也会导致 fullgc

##### 2.2 jhat

jhat -J-Xmx1024M /path/to/heamdump.hprof

执行后等待 console 中输入 start HTTP server on port 7000 即可使用浏览器访问 IP：7000

##### 2.3 jinfo

用于实时查看和调整目标 JVM 的各项参数。

##### 2.4 jstat

jstat - 轻量级监控工具，可用于获取目标 Java 进程的类加载、JIT 编译、垃圾收集、内存使用等信息。

`jstat -gcutil <pid>` 没用，实践中都是看 gc.log，就不细说了

监控工具，一般公司都有整合方案(falcon、prometheus)，就不细说了

##### 2.5 jcmd

相比 jstat 功能更为全面的工具，可用于获取目标 Java 进程的性能统计、JFR、内存使用、垃圾收集、线程堆栈、JVM 运行时间等信息。

`jcmd <pid> help` 查看可用功能，不展开了

##### 2.6 jstack

`jstack <pid>`

```log
"XXXXXXXXXXXXX" #392 daemon prio=5 os_prio=31 tid=0x00007f9560e67000 nid=0x25903 waiting on condition [0x0000700013d75000]
   java.lang.Thread.State: WAITING (parking)
        at sun.misc.Unsafe.park(Native Method)
        - parking to wait for  <0x00000007b690ad48> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)
        at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
        at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)
        at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1081)
        at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:809)
        at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1074)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1134)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)
```

信息都在第一行，下面是栈

* 线程名称：XXXXXXXXXXXXX
* 线程类型：daemon
* 优先级: 5，默认是 5
* jvm 线程 id：tid=0x00007f9560e67000 内部线程的唯一标识（通过 java.lang.Thread.getId() 获取，通常用自增方式实现）
* 对应系统线程 id(NativeThread ID)：nid=0x25903 top 命令查看的线程 pid 对应，不过一个是 10 进制，一个是 16 进制。（通过命令：top -H -p pid，可以查看该进程的所有线程信息）
* 线程状态：waiting on condition
* 起始栈地址：[0x0000700013d75000]
* Java thread statck trace：是上面 at 行的信息。到目前为止这是最重要的数据，Java stack trace 提供了大部分信息来精确定位问题根源。

可以看下这两篇学学怎么分析

* https://juejin.im/entry/59bf5e66f265da065754c4a8
* https://www.cnblogs.com/zhengyun_ustc/archive/2013/03/18/tda.html

#### 3. 分析工具

jmap、jstack 的输出人类读起来比较困难，可以借助分析工具

heap dump 堆内存分析

* gui 工具，主推 jvisualvm
* 在线分析 XElephant https://memory.console.perfma.com/

jstack 线程分析

* https://thread.console.perfma.com/
* https://fastthread.io/
* https://heaphero.io/
* https://spotify.github.io/threaddump-analyzer/
* https://www.ibm.com/support/pages/ibm-thread-and-monitor-dump-analyzer-java-tmda

gc.log 分析工具

* https://gceasy.io
    * 讲解 https://juejin.im/post/5d0f6b2af265da1b942153d7

Java 虚拟机参数分析

* https://opts.console.perfma.com/

交互式分析工具

* https://www.perfma.com/docs/exam/exam-profile 包含 jvm 参数分析 & 线程分析，基于 javaagent 开发，只能连本地 java 进程

#### 4. 其他工具

btrace、JFR、arthas。这几个高级工具这次先不讲了。还有 google 的 lightweight-profiler 据称对线上影响很小可以随意使用。http://blog.csdn.net/ligeforrent/article/details/74004359

#### 5. 结论

* 堆分析：jmap dump.hprof -> jvisualvm
* 线程分析：jastack -> https://thread.console.perfma.com/
* gc 分析：gc.log -> https://gceasy.io

### T.2

1. 公众号
    1. 内容组织 利用公众号的专辑功能，按 topic 组织系列文章，但因为公众号文章不能修改(大改)，所以原文可以放在 github，用于个人知识图谱的建立
        1. 这个仓库主要作用还是打卡记录(所以我倾向于内容部分贴链接即可)，个人知识图谱的记录可以自己找合适的方法
    1. 排版 作为一个程序员，建议用 markdown https://www.mdnice.com
    1. 封面 & 配图 要用无版权图片哦 https://www.zhihu.com/question/37493361
1. 摸鱼刷题插件 IDEA-leetcode、vscode-leetcode
1. review 和 shared 的区别：review 是输入，shared 是输出。一般初学者是 输入 >> 输出，不要灰心

### S.1

// TODO javaagent
