背景：最近在梳理Android线程调度的相关内容。在梳理过程中，阅读了部分源码，以及相关的介绍文章，甚至重新翻起了《Linux内核设计与实现》，但是距离理解透彻，并且能够用自己的语言清晰无误地阐述出来，感觉还有点远，还有很多细节需要进一步理论结合实际。为了避免在忙乱的生活节奏中，梳理的目标又草草结束。希望自己能够把目标细分一下，先把几个理解清晰的问题给记录下来，通过不断清晰地回答相关的问题，最终能够完成整个原理的清晰理解与阐述。这篇文章，就是针对Android线程优先级方面，一个一个问题的回答，可能有些凌乱。如果有理解不到位的地方，也希望大家指出来。

问题一：Linux是用什么来描述进程的优先级的？
Linux中存在实时进程，和普通进程。对于普通进程来讲，使用nice来描述进程的优先级，取值范围是[-20,19]。对于实时进程来讲，则有一个实时优先级，取值范围是[0,99]。如下图所示，普通进程与实时进程的优先级对齐后，得到了进程的优先级prio。


整体来看，Linux中进程的优先级分为三类：

静态优先级： 不会时间而改变，内核也不会修改，只能通过系统调用改变nice值的方法区修改。优先级映射公式： static_prio = MAX_RT_PRIO + nice + 20，其中MAX_RT_PRIO = 100，那么取值区间为[100, 139]；对应普通进程；
实时优先级：只对实时进程有意义，取值区间为[0, MAX_RT_PRIO -1]，其中MAX_RT_PRIO = 100，那么取值区间为[0, 99]；对应实时进程；
动态优先级： 调度程序通过增加或减少进程静态优先级的值，来达到奖励IO消耗型或惩罚cpu消耗型的进程，调整后的进程称为动态优先级。区间范围[0, MX_PRIO-1]，其中MX_PRIO = 140，那么取值区间为[0,139]；
我们在Android手机上，可以通过adb shell ps -p -t -P来看到进程的优先级，如下所示：RTPRI字段表示real time priority,只对实时进程有有效。而NICE字段，只对普通进程有效。将RTPRI与NICE统一后，得到进程的PRIO，其中值越小，表示优先级越高。


问题二：java.lang.Thread.setPriority与android.os.Process.setThreadPriority有什么区别和联系？
如何设置进程的优先级呢？一般有两种方式，一种是通过java.lang.Thread.setPriority，还有一种是通过android.os.Process.setThreadPriority。这两种方式的参数是有不同的，而且效果也有细微的差别。我们直接通过源码来分析差别。

 1、android.os.Process.setThreadPriority 

分析android.os.Process.java的源码，我们看到setThreadPriority是一个jni接口，最终实现在native层。其参数的定义在Proce3ss.java中定义了一些常量。结合前面的NICE值，我们知道这个接口应该是直接设置进程的nice值了。


具体是不是设置Linux中的nice值，我们可以看看native层的实现，Process.setThreadPriority对应native的实现在core/jni/android_util_Process.cpp中，代码如下：


最终又调用到libutils中的Threads.cpp,最终到setpriority这个系统调用，可以明确看到就是设置进程的NICE值。


对应Process.java中的getThreadPriority最终也调用了系统调用getpriority。

2、java.lang.Thread.setPriority 

通过Thread.java的源码，我们看到对于priority的设置在MIN_PRIORITY（1），与MAX_PRIORITY（10）之间。按接口的描述，MAX_PRIORITY的优先级应该是大于MIN_PRIORITY，即优先级是递增的。这个跟Linux的NICE值定义是不同的。其实是这个是java中对于线程优先级的规范，具体的实现是按虚拟机来。Android是运行在Linux的内核之上的，最终也要通过系统调用来设置进程的NICE值来调整进程的优先级的。这里，我们有疑问的有两个点，第一个点是，java的线程优先级如何跟NICE对应，第二个点是这个接口，跟前面Process.setThreadPriority除了优先级的定义不同，还有什么差别吗？


为了解决这两个问题，我们再分析native层的实现nativeSetPriority，代码如下：我们看到java的priority与nice值之间有一个对应关系，其中MAX_PRIORITY对应了ANDROID_PRIORITY_URGENT_DISPLAY，也就是Process.java中的THREAD_PRIORITY_URGENT_DISPLAY，即-8。而MIN_PRIORITY对应了ANDROID_PRIORITY_LOWEST，也就是Process.java中的THREAD_PRIORITY_LOWEST。


另外，我们细心分析还发现一个问题，Thread的getPriority接口，直接返回了priority变量。而priority变量只有在Thread初始化的时候，补设置为parent的优先级，以及在调用setPriority时，更新优先级。除此之外，没有其他的数据同步操作。这意味着，如果不是通过java的Thread.setPriority更新的优先级，通过Thread.getPriority是无法同步更新的。

这个问题小结的结论是：

无论是Thread.setPriority还是Process.setThreadPriority最终都会更新进程的nice值。
Thread.setPriority中的[MAX_PRIORITY,MIN_PRIORITY]对应了NICE值的[-8,19],可见Process.java的setThreadPriority对线程的优先级划分得更加细。这就是为什么有人建议通过Process.setThreadPriority来设置线程的优先级的原因了，可以将优先级划分的是更加细一些。
在调整线程的优先级的过程中，也会调整线程的cgroups。
在没有明确设置的情况下，一个线程初始的优先级等于其parent的优先级。如果我们从UI线程来创建一个子线程的，那么这个子线程的优先级就等于UI线程的优先级。
问题三：Android的一些异步线程组件是如何来设置线程的优先级的呢？
1、Thread 

如果没有给线程设置优先级，线程默认的优先级是调用new Thread的当前线程的优先级。由此可知，在UI线程创建一个子线程时，这个被创建的子线程的优先级直接等于UI线程的优先级。


2、AsyncTask 异步任务 

分析AsyncTask的源码我们看到，最终是通过调用Process.setThreadPriority来设置线程的优先级的，这里默认的优先级是Process.THREAD_PRIORITY_BACKGROUND。再结合前面的代码可以看到，如果设置的THREAD_PRIORITY_BACKGROUND(nice=10)，最终会调整线程的调度策略，分配到SP_BACKGROUND调度组中，这样可以避免影响UI主线程的响应。


3、HandlerThread 

分析HandlerThread的代码，可以看到，其默认的priority是Process.THREAD_PRIORITY_DEFAULT(nice=0)，当然我们也可以设置指定的priority。最后也是通过Process.setThreadPriority来设置线程的优先级的。


4、ThreadPoolExecutor 

ThreadPoolExecutor的线程最终是由ThreadFactory提供的，意味着线程的优先级由ThreadFactory来设置。一般来讲，我们在实现ThreadFactory的newThread都会设置线程的优先级。


如果我们没有设置ThreadFactory，则会使用默认的DefaultThreadFactory，默认的ThreadFactory设置了，线程的优先级为Thread.NORM_PRIORITY，也就是对应ANDROID_PRIORITY_NORMAL(nice=0)


5、IntentService 

IntentService内部使用了HandlerThread，而且没有特别设置优先级。结合前面HandlerThread的分析，我们知道，其默认优先级是Process.THREAD_PRIORITY_DEFAULT(nice=0)。


后续：进程的优先级是如何影响进程调度？
见后续分析！

参考内容

[1]http://gityuan.com/2015/10/01/process-priority/

