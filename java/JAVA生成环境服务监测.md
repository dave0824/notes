## 一、生产环境服务器变慢，诊断思路和性能评估谈谈?

1. liunx 上查看整机使用情况命令： top

2. CPU:  vmstat 命令：

   查看CPU：vmstat -n 2 3

   ![vmstat1](..\asset\java\vmstat1.bmp)

   ![vmstat2.](..\asset\java\vmstat2.bmp)

   - 查看所有CPU核信息:  mpstat -P ALL 2
   - 每个进程使用cpu的用量分解信息:  pidstat -u 1 -p 进程编号

3. 内存: free

   - 应用程序可用内存数：free

     ![free](..\asset\java\free.bmp)

   - 查看额外： pidstat -p 进程号 -r 采样间隔秒数

4. 硬盘：df

   查看磁盘剩余空闲数：df -h

5. 磁盘IO：iostat

    - 磁盘I/O性能评估:

      ![vmstat2.](..\asset\java\iostat.bmp)

      ![vmstat2.](..\asset\java\iostat2.bmp)

   - 额外查看：pidstat -d 采样间隔秒数 -p 进程号

6. 网络IO：ifstat

   - 默认本地没有，下载ifstat:

     ![vmstat2.](..\asset\java\ifstat.bmp)

   - 查看网络IO

     ![vmstat2.](..\asset\java\ifstat2.bmp)

     ![vmstat2.](..\asset\java\ifstat3.bmp)

## 二、 假如生产环境出现CPU占用过高，请谈谈你的分析思路和定位

结合Linux和JDK命令一块分析

案例：

1. 先用top命令找出CPU占比最高的

   ![vmstat2.](..\asset\java\anli1.bmp)

2. ps -ef或者jps进一步定位，得知是一个怎么样的一个后台程序

3. 定位到具体线程或者代码:  ps -mp 进程 -o THREAD,tid,time

   ![vmstat2.](..\asset\java\anli2.bmp)

   参数解释:

   	- -m 显示所有线程
   	- -p pid进程使用cpu的时间
   	- -o 该参数后是用户自定义格式

4. 将需要的线程ID转换为16进制格式(英文小写格式)

5. jstack 进程ID | grep tid(16进制线程ID小写英文) -A60

## 三、对于JDK自带的JVM监控和性能分析工具用过哪些？一般你是怎么用的？

是什么：

![vmstat2.](..\asset\java\tools.bmp)

性能监控工具:

- jps(虚拟机进程状况工具)
- jinfo(Java配置信息工具)
- jmap(内存映像工具)
- jstat(统计信息监控工具)