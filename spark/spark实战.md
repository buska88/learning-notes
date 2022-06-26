# spark 实战

## 本地编译spark

下载spark后

安装spark：

1.直接编译spark发现maven版本问题：

2.编译某个模块出现问题

![image-20220519090306575](spark实战.assets/image-20220519090306575.png)

```
[ERROR] Failed to execute goal on project spark-network-common_2.11: Could not resolve dependencies for project org.apache.spark:spark-network-common_2.11:jar:2.2.4-SNAPSHOT: Could not transfer artifact com.fasterxml.jackson.core:jackson-databind:jar:2.6.5 from/to nexus (http://localhost:8081/nexus/content/groups/public/): Connect to localhost:8081 [localhost/127.0.0.1, localhost/0:0:0:0:0:0:0:1] failed: Connection refused: connect -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/DependencyResolutionException
[ERROR]
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :spark-network-common_2.11
/g/spark/build/zinc-0.3.11/bin/nailgun: line 49: ng: command not found

```

重新试了一下就好了，应该是网络问题。

我编译的是spark2.2

参考文档：

https://blog.csdn.net/make__It/article/details/84258916

在windows上编译完成了，使用的maven是本地路径的maven，调大maven内存的操作没什么用；

编译spark时遇到Spark-Parent包test_classpath飘红，解决方案见：https://www.cnblogs.com/limaosheng/p/15807925.html

然后maven-checkstyle-plugin标签的verbose和failonwarning不可使用，首先看了一下dev/checkstyle.xml，发现dtd文件没倒入，于是按小灯泡使其fetch外部资源，但是还是爆红

## wordcount

首先利用maven创建工程，然后发现没有scala的框架支持，解决方法如下：

https://blog.csdn.net/ChrisLu777/article/details/113739910?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-113739910-blog-103887060.pc_relevant_scanpaymentv1&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-113739910-blog-103887060.pc_relevant_scanpaymentv1&utm_relevant_index=1

最后externel library有scala的sdk就可。

![1653359941602.png](spark实战.assets/1653359941602.png)

配置pom.xml,我自己install了2.2.1版本的spark,但是我在wc程序中使用的是2.1.0

![1653360827243.png](spark实战.assets/1653360827243.png)

然后使用idea将项目打成jar包放到spark命令行下执行。