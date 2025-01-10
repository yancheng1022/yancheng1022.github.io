---
title: Arthas调试线上代码技巧
date: 2024/03/27
categories:
  - coding
tags:
  - 阿尔萨斯
  - Arthas
  - 代码调试
  - 技术文章
abbrlink: 27962
---

# 1、背景

1、测试没问题，但生产环境没有生效，需要查看生产环境代码是否更新上去（反编译）
2、在没有日志的情况下，我们不知道客户端请求过来的参数是什么，临时加接口日志再发版是一件很复杂的事情（监视方法执行）
3、有时候我们发现应用卡住了， 通常是由于某个线程拿住了某个锁， 并且其他线程都在等待这把锁造成的（thread -b）
4、需要快速定位应用的热点，生成火焰图（profile）
5、进行性能调优时需要在代码里添加大量计时器代码，还要重新部署，太麻烦（trace）

# 2、安装

其实就是下载一个jar包
```shell
curl -O https://arthas.aliyun.com/arthas-boot.jar
java -jar arthas-boot.jar
```

启动之后，就会列出当前系统上所有的java进程，输入进程前面的序号，连接到相应的应用进程。

# 3、反编译代码

``` shell
jad --source-only com.controller.HomePageController getSaleData --lineNumber false
```

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202403271326704.png)

> 注意：访问类的静态成员时（属性或方法），才会进行类加载。所以想要反编译静态方法，需要先确保调用一次

# 4、监视方法执行

```shell
watch com.data.controller.baseInfo.InstrumentBaseController list '{params,returnObj,throwExp}' -n 5 -x 3
```

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202403271353072.png)


# 5、执行静态方法


```shell
ognl '@com.chivd.common.utils.DateUtils@getDate()'
```


# 6、找出当前阻塞其它线程的线程

有时候我们发现应用卡住了， 通常是由于某个线程拿住了某个锁， 并且其他线程都在等待这把锁造成的。 为了排查这类问题， arthas 提供了thread -b， 一键找出那个罪魁祸首

```shell
thread -b
```

# 7、火焰图

profiler命令可以在一段时间内对数据采样，生成火焰图。该命令对cpu进行300秒的采样，采样后生成svg或html格式的火焰图。

```shell
profiler start --duration 300
```

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202403271520463.png)

>y 轴表示调用栈，每一层都是一个函数。调用栈越深，火焰就越高，顶部就是正在执行的函数，下方都是它的父函数。
 x 轴表示抽样数，如果一个函数在 x 轴占据的宽度越宽，就表示它被抽到的次数多，即执行的时间长。注意，x 轴不代表时间，而是所有的调用栈合并后，按字母顺序排列的。
 火焰图就是看顶层的哪个函数占据的宽度最大。只要有"平顶"（plateaus），就表示该函数可能存在性能问题

# 8、性能调优计时

trace命令能记录指定方法的调用路径，并统计方法向下一层调用的耗时。

```shell
trace *InstrumentBaseController list
```

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202403271525052.png)
