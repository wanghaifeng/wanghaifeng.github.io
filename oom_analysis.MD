# OutOfMemory分析方案与实施步骤

OutOfMemory异常（简称OOM）发生的原因通常都是因为JVM的垃圾回收机制失效，内存耗尽的同时，有新的数据继续写入共享内存堆栈。

查找及分析的策略主要为：
+ 通过检查垃圾回收日志，对内存分配及回收过程进行调优；
+ 对系统加压并跟踪内存状态，找出无法回收的大数据对象；
+ 通过步骤2的结果，调整JVM内存配置，或者优化代码逻辑。

## 具体方案及策略

### 利用垃圾回收日志进行GC优化
#### 应用服务器启用GC日志，及相关参数
<pre><code>
-Xloggc:/tmp/logs/gc-%t.log
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-XX:+PrintGCCause
-XX:+PrintTenuringDistribution
-XX:+UseGCLogFileRotation
-XX:NumberOfGCLogFiles=10
-XX:GCLogFileSize=5M
</code></pre>
#### 初始优化建议
* 根据数据对象大小，放宽年轻代（Young Generation）的对象大小限制。优点是大数据对象可以在使用后快速回收，风险是会加大年轻代的内存占用率
<pre><code>-XX:PretenureSizeThreshold=300M</code></pre>
* 增加年轻代存活区数据进入老年代（Old Generation）之前的交替次数。优缺点同上。
<pre><code>-XX:MaxTenuringThreshold=20</code></pre>
* 选择性调整触发老年代Full GC的内存使用率。优点是加快老年代的回收频次，缺点是应用会变慢。
<pre><code>-XX:CMSInitiatingOccupancyFraction=50</code></pre>
#### 根据GC日志给出的后续优化建议

TODO

### 对系统加压找出内存溢出原因
压力测试及性能分析工具很多，以Jmeter（Free） + Yourkit（注册后免费试用）为例
* 应用服务器植入Yourkit代理
https://www.yourkit.com/docs/java/help/profiling_j2ee_remote.jsp
* 按照逐步加压的方式，分别记录并对比Yourkit反馈的内存使用情况
* 分别获取内存快照，对比大数据对象。
* 可选的代码性能优化步骤：1、单用户单线程运行，记录每个URL的响应时间；2、10用户加压，记录并对比每个URL的相应时间，选择可能有问题的URL；3、50用户加压，记录对比主要URL相应时间；4、加压后运行效率明显变差的URL列表

### 优化JVM内存配置或者代码优化建议
