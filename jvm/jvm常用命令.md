### JVM常用命令 ###
------

### 1、 jinfo

输出给定 java 进程所有的配置信息。包括 java 系统属性和 jvm 命令行标记等。

列子：
```
jinfo pid
```

### 2、jps

 jps主要用来输出JVM中运行的进程状态信息。

 命令行参数选项说明：

 * -q 不输出类名、Jar名和传入main方法的参数

 * -m 输出传入main方法的参数

 * -l 输出main类或Jar的全限名

 * -v 输出传入JVM的参数

### 3、jstat

jstat可以实时显示本地或远程JVM进程中类装载、内存、垃圾收集、JIT编译等数据，如果在服务启动时没有指定启动参数-verbose:gc，则可以用jstat实时查看gc情况。

命令格式：

```
jstat [option vmid [interval[s|ms] [count]]]
```
参数说明：

* option 我们一般使用 -gcutil 查看gc情况

* vmid  VM的进程号，即当前运行的java进程号

* interval 间隔时间，单位为秒或者毫秒

* count 打印次数，如果缺省则打印无数次

jstat选项说明：

* -class 监视类装载、卸载数量、总空间及类装载所耗费的时间。

* -gc 监听Java堆状况，包括Eden区、两个Survivor区、老年代、永久代等的容量，以用空间、GC时间合计等信息。

* -gccapacity 监视内容与-gc基本相同，但输出主要关注java堆各个区域使用到的最大和最小空间。

* -gcutil 监视内容与-gc基本相同，但输出主要关注已使用空间占总空间的百分比。

* -gccause 与-gcutil功能一样，但是会额外输出导致上一次GC产生的原因

* -gcnew 监视新生代GC状况

* -gcnewcapacity 监视内同与-gcnew基本相同，输出主要关注使用到的最大和最小空间

* -gcold 监视老年代GC情况

* -gcoldcapacity 监视内同与-gcold基本相同，输出主要关注使用到的最大和最小空间

* -gcpermcapacity  输出永久代使用到最大和最小空间

```
jstat-gc xxx100030 #查看java 进程的gc情况
```


打印结果说明：

* S0C：S0区容量（S1区相同，略）
* S0U：S0区已使用
* EC：E区容量
* EU：E区已使用
* OC：老年代容量
* OU：老年代已使用
* PC：Perm容量
* PU：Perm区已使用
* YGC：Young GC（Minor GC）次数
* YGCT：Young GC总耗时
* FGC：Full GC次数
* FGCT：Full GC总耗时
* GCT：GC总耗时

### 4、jstack
 jstack主要用来查看某个Java进程内的线程堆栈信息,根据堆栈信息我们可以定位到具体代码，所以它在JVM性能调优中使用得非常多。

 语法：
 ```
jstack option pid
 ```

 命令参数说明：
 ```
-l long listings，会打印出额外的锁信息，在发生死锁时可以用jstack -l pid来观察锁持有情况
-m mixed mode，不仅会输出Java堆栈信息，还会输出C/C++堆栈信息（比如Native方法）
 ```

 ### 5、jmap
 jmap用来查看堆内存使用状况，一般结合jhat使用,用于显示当前Java堆和永久代的详细信息（如当前使用的收集器，当前的空间使用率等）。

 语法：
```
jmap [option] pid
```

命令参数说明：

* -dump 生成java堆转储快照

* -heap 显示java堆详细信息(只在Linux/Solaris下有效)

* -histo 显示堆中对象统计信息

列子:
jmap进行dump命令格式如下：
```
jmap -dump:format=b,file=dumpFileName pid

# jmap -dump:format=b,file=/tmp/dump.dat 21711
```

### 6、jhat

用于分析使用jmap生成的dump文件，是JDK自带的工具。

语法：
```
 jhat -J -Xmx512m [file]
```
