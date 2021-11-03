# Mybatis查询数据慢问题排查

#### 问题背景：

在加压数据上开发导出功能时，发现从数据库中获取数据很慢，从20W的数据中获取1W数据需要花费43s，可是对应的sql在数据库内直接执行仅仅用了0.055s。

![1.png](https://i.loli.net/2021/02/26/ibtAnFH7pIO36e4.png)

![2.png](https://i.loli.net/2021/02/26/9ineI1N7tc8FK3y.png)

#### 问题分析：

之前就遇到过mybatis查询大数据量数据的时候会很慢，猜测是mybatis转实体bean的时候有开销导致，由于当时通过增加分页处理规避掉了，就没探究实质。如今使用场景无法使用分页，所以只能找到根本原因并优化，请教了博哥后，了解到遇到调用方法后长时间没有返回结果的问题时，可以通过打印对应线程的堆栈信息，从堆栈信息中可以得知线程在作什么事情，从而定位到问题，现将排查步骤整理如下。

#### 问题排查：

1. 首先执行jps命令，查找当前jvm进程id。

```shell
jps
```

![3.png](https://i.loli.net/2021/02/26/yPL3zuBIEms6wVZ.png)

2. 通过jstack输出所有线程的堆栈信息，找到对应执行数据查询的线程，进行分析。

```shell
jstack -l 7884
```

![4.png](https://i.loli.net/2021/02/26/FAchLnVTp7gsxKt.png)

3. 找到对应调用数据查询方法的线程堆栈信息。

![5.png](https://i.loli.net/2021/02/26/xRYSqU9sDCfv6tp.png)

4. 分析得知线程阻塞在了sqlfx的getDebugInfo()上，于是修改jdbc日志输出级别解决了该问题。

![6.png](https://i.loli.net/2021/02/26/8lSqLQvp17dwIVG.png)

5. 验证

![7.png](https://i.loli.net/2021/02/26/ajukevphUfPQ5Ic.png)

![8.png](https://i.loli.net/2021/02/26/c7RbS4FhOnNt6Ql.png)

#### 技能点分享（一）

###### 虚拟机性能分析和故障解决常用工具的使用：

链接: [https://pan.baidu.com/s/1LwYNRwEo1ZEzR8lVIbeLXQ](https://pan.baidu.com/s/1LwYNRwEo1ZEzR8lVIbeLXQ)  密码: fpb1

#### 问题扩展

排查查询慢的问题时，在jsconsole内观察虚拟机性能参数时，发现如果查询数据量很大的时候，使用list会占用大量内存，考虑到并发的情况下容易oom，所以采用cursor游标进行优化。

#### 性能分析

##### 1.  jvm参数配置如下：

##### -Xms2000m -Xmx2000m -Xmn1000m -XX:SurvivorRatio=8

将数据扩大到30W，先在jsconsole中执行一次GC，后进行一次调用，过程中用jsconsole观察jvm堆内存的变化情况。

![9.png](https://i.loli.net/2021/02/26/JiY9nrE5q7magNl.png)

由上图jvm内存变化可以看出，在获取数据的过程中GC了一次，GC后内存较开始时增长了500m，由此可以判断30W的数据占用内存一定大于500m。

又此数据分析可以得知，如果并发几个导出线程，jvm便会oom。

##### 2.  jvm参数配置如下：

##### -Xms500m -Xmx500m -Xmn250m -XX:SurvivorRatio=8

再次执行一次查询，控制台报了oom。

![10.png](https://i.loli.net/2021/02/26/yudEQ3LwZTnCtaN.png)

#### 采用游标进行优化

![11.png](https://i.loli.net/2021/02/26/LnGMoRzeryQNPsU.png)
