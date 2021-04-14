这是一个非常经典的脚本，由于某种原因原始版主把脚本删除了，因为日常工作都能用到的脚本，所以这里推荐给大家！

介绍：

    用于快速排查Java的CPU性能问题(top us值过高)，自动查出运行的Java进程中消耗CPU多的线程，并打印出其线程栈，从而确定导致性能问题的方法调用。目前只支持Linux。原因是Mac、Windows的ps命令不支持列出进程的线程id。

    top命令找出有问题Java进程及线程id：开启线程显示模式（top -H，或是打开top后按H） 按CPU使用率排序（top缺省是按CPU使用降序，已经合要求；打开top后按P可以显式指定按CPU使用降序） 记下Java进程id及其CPU高的线程id 用进程id作为参数，jstack有问题的Java进程 手动转换线程id成十六进制（可以用printf %x 1234） 查找十六进制的线程id（可以用vim的查找功能/0x1234，或是grep 0x1234 -A 20） 查看对应的线程栈，以分析问题 查问题时，会要多次上面的操作以分析确定问题，这个过程太繁琐太慢了。

用法

show-busy-java-threads
# 从所有运行的Java进程中找出最消耗CPU的线程（缺省5个），打印出其线程栈
# 缺省会自动从所有的Java进程中找出最消耗CPU的线程，这样用更方便
# 当然你可以手动指定要分析的Java进程Id，以保证只会显示出那个你关心的那个Java进程的信息
show-busy-java-threads -p <指定的Java进程Id>
 
show-busy-java-threads -c <要显示的线程栈数>
 
show-busy-java-threads <重复执行的间隔秒数> [<重复执行的次数>]
# 多次执行；这2个参数的使用方式类似vmstat命令
 
show-busy-java-threads -a <运行输出的记录到的文件>
# 记录到文件以方便回溯查看
 
show-duplicate-java-classes -S <存储jstack输出文件的目录>
# 指定jstack输出文件的存储目录，方便记录以后续分析
 
##############################
# 注意：
##############################
# 如果Java进程的用户 与 执行脚本的当前用户 不同，则jstack不了这个Java进程
# 为了能切换到Java进程的用户，需要加sudo来执行，即可以解决：
sudo show-busy-java-threads
 
show-busy-java-threads -s <指定jstack命令的全路径>
# 对于sudo方式的运行，JAVA_HOME环境变量不能传递给root，
# 而root用户往往没有配置JAVA_HOME且不方便配置，
# 显式指定jstack命令的路径就反而显得更方便了
 
# -m选项：执行jstack命令时加上-m选项，显示上Native的栈帧，一般应用排查不需要使用
show-busy-java-threads -m
# -F选项：执行jstack命令时加上 -F 选项（如果直接jstack无响应时，用于强制jstack），一般情况不需要使用
show-busy-java-threads -F
# -l选项：执行jstack命令时加上 -l 选项，显示上更多相关锁的信息，一般情况不需要使用
# 注意：和 -m -F 选项一起使用时，可能会大大增加jstack操作的耗时
show-busy-java-threads -l
 
# 帮助信息
$ show-busy-java-threads -h
Usage: show-busy-java-threads [OPTION]... [delay [count]]
Find out the highest cpu consumed threads of java, and print the stack of these threads.
 
Example:
  show-busy-java-threads       # show busy java threads info
  show-busy-java-threads 1     # update every 1 second, (stop by eg: CTRL+C)
  show-busy-java-threads 3 10  # update every 3 seconds, update 10 times
 
Output control:
  -p, --pid <java pid>      find out the highest cpu consumed threads from
                            the specified java process, default from all java process.
  -c, --count <num>         set the thread count to show, default is 5.
  -a, --append-file <file>  specifies the file to append output as log.
  -S, --store-dir <dir>     specifies the directory for storing intermediate files, and keep files.
                            default store intermediate files at tmp dir, and auto remove after run.
                            use this option to keep files so as to review jstack/top/ps output later.
  delay                     the delay between updates in seconds.
  count                     the number of updates.
                            delay/count arguments imitates the style of vmstat command.
 
jstack control:
  -s, --jstack-path <path>  specifies the path of jstack command.
  -F, --force               set jstack to force a thread dump.
                            use when jstack <pid> does not respond (process is hung).
  -m, --mix-native-frames   set jstack to print both java and native frames (mixed mode).
  -l, --lock-info           set jstack with long listing. Prints additional information about locks.
 
cpu usage calculation control:
  -d, --top-delay           specifies the delay between top samples, default is 0.5 (second).
                            get thread cpu percentage during this delay interval.
                            more info see top -d option. eg: -d 1 (1 second).
  -P, --use-ps              use ps command to find busy thread(cpu usage) instead of top command,
                            default use top command, because cpu usage of ps command is expressed as
                            the percentage of time spent running during the *entire lifetime*
                            of a process, this is not ideal in general.
 
Miscellaneous:
  -h, --help                display this help and exit.
示例

$ show-busy-java-threads
[1] Busy(57.0%) thread(23355/0x5b3b) stack of java process(23269) under user(admin):
"pool-1-thread-1" prio=10 tid=0x000000005b5c5000 nid=0x5b3b runnable [0x000000004062c000]
   java.lang.Thread.State: RUNNABLE
    at java.text.DateFormat.format(DateFormat.java:316)
    at com.xxx.foo.services.common.DateFormatUtil.format(DateFormatUtil.java:41)
    at com.xxx.foo.shared.monitor.schedule.AppMonitorDataAvgScheduler.run(AppMonitorDataAvgScheduler.java:127)
    at com.xxx.foo.services.common.utils.AliTimer$2.run(AliTimer.java:128)
    at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
    at java.lang.Thread.run(Thread.java:662)
 
[2] Busy(26.1%) thread(24018/0x5dd2) stack of java process(23269) under user(admin):
"pool-1-thread-2" prio=10 tid=0x000000005a968800 nid=0x5dd2 runnable [0x00000000420e9000]
   java.lang.Thread.State: RUNNABLE
    at java.util.Arrays.copyOf(Arrays.java:2882)
    at java.lang.AbstractStringBuilder.expandCapacity(AbstractStringBuilder.java:100)
    at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:572)
    at java.lang.StringBuffer.append(StringBuffer.java:320)
    - locked <0x00000007908d0030> (a java.lang.StringBuffer)
    at java.text.SimpleDateFormat.format(SimpleDateFormat.java:890)
    at java.text.SimpleDateFormat.format(SimpleDateFormat.java:869)
    at java.text.DateFormat.format(DateFormat.java:316)
    at com.xxx.foo.services.common.DateFormatUtil.format(DateFormatUtil.java:41)
    at com.xxx.foo.shared.monitor.schedule.AppMonitorDataAvgScheduler.run(AppMonitorDataAvgScheduler.java:126)
    at com.xxx.foo.services.common.utils.AliTimer$2.run(AliTimer.java:128)
    at java.util.concurrent.ThreadPoolExecutor$Worker.runTask(ThreadPoolExecutor.java:886)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:908)
    at java.lang.Thread.run(Thread.java:662)
上面的线程栈可以看出，CPU消耗最高的2个线程都在执行java.text.DateFormat.format，业务代码对应的方法是shared.monitor.schedule.AppMonitorDataAvgScheduler.run。可以基本确定：

AppMonitorDataAvgScheduler.run调用DateFormat.format次数比较频繁。DateFormat.format比较慢。（这个可以由DateFormat.format的实现确定。） 多执行几次show-busy-java-threads，如果上面情况高概率出现，则可以确定上面的判定。因为调用越少代码执行越快，则出现在线程栈的概率就越低。脚本有自动多次执行的功能，指定 重复执行的间隔秒数/重复执行的次数 参数。

分析shared.monitor.schedule.AppMonitorDataAvgScheduler.run实现逻辑和调用方式，以优化实现解决问题。
