#### 计算机是如何启动的？

  参考：[https://www.jb51.net/article/208552.htm](https://www.jb51.net/article/208552.htm)

  计算机的启动可以分为两个阶段：引导阶段和加载内核阶段。

  - 先说说引导阶段，在按下开机键的时候，内存中什么数据也没有，因此需要借助某种方式，将操作系统加载到内存中，而完成这项任务的就是BIOS。BIOS是主板芯片上的一个程序，计算机通电后，第一件事情就是读取BIOS。BIOS首先会进行硬件检测，检测硬件是否满足计算机运行的基本条件，如果硬件出现问题，主板会发出不同的蜂鸣声，然后停止启动，如果没有问题，则会显示CPU内存硬盘等信息。
  - 接着，BIOS会读取存储设备的第一个扇区，从中读取一个叫“主引导记录”也就是MBR信息，MBR的主要作用是告诉计算机到哪个硬盘位置上去找操作系统。然后运行“管理启动器”boot loader，由用户选择启动哪个操作系统。
  - 用户选择操作系统后，就到加载内核阶段，操作系统的内核被载入内存中。以Linux为例，先载入/boot目录下的kernel，kernel加载完成后，运行第一个程序/sbin/init。init进程是Linux启动后的第一个进程，pid为1，其他的进程都是它的后代。init进程会加载系统的各个模块，比如窗口程序和网络程序等，直到界面显示登录信息，至此，整个系统启动完毕。


#### 说说Android的启动过程

  参考：[https://www.jb51.net/article/208552.htm](https://www.jb51.net/article/208552.htm)

  1. Android是基于Linux系统的。但是 它没有BIOS程序，取而代之的是BootLoader。BootLoader类似于BIOS，在系统加载前，用于初始化硬件设备。而Android中也没有硬盘，只有ROM，类似于硬盘存放操作系统，用户程序等。当按下电源键后，首先加载BootLoader，BootLoader去初始化硬件设备，然后读取ROM找到系统并将内核加载进RAM中。当内核启动后会初始化各种软硬件环境，加载驱动程序，挂载跟文件系统。最后阶段会启动执行第一个进程init进程。
  2. init进程是用户的第一个进程，pid为1。init进程会创建和挂载启动所需要的文件目录，对属性服务进行初始化，最后会解析init.rc文件。
  3. init.rc文件是Android系统的重要配置文件，主要功能是定义了系统启动时需要执行的一系列动作、设置环境变量和属性和执行特定的service。通过这个脚本启动了几个重要的服务：
      - service_manager：ServiceManager 是 Binder IPC 通信过程中的守护进程，本身也是一个 Binder 服务。ServiceManager 进程主要是启动 Binder，提供服务的查询和注册。
      - surface_flinger：SurfaceFlinger 负责图像绘制，合成所有 Surface 并渲染到显示设备。
      - media_server：MediaServer 进程主要是启动 AudioFlinger 音频服务，CameraService 相机服务。负责处理音频解析播放，相机相关的处理。
      - Zygote：Zygote 进程也叫孵化进程，它主要作用是启动system-server和作为守护进程监听处理“孵化新进程”的请求。
  4. Zygote启动了system-server进程，system-server会启动系统中的各种服务，像AMS，PMS，WMS，等等。
  5. 在启动的AMS服务中，又会去启动Launcher进程，到这里整个Android启动便完成。


#### 系统服务何时启动？如何启动？

  在SystemServer进程启动时，会分批启动所有系统服务。通过SystemServiceManager#startService来启动一个服务，服务启动后需要注册到ServiceManager中，其他进程访问ServiceManager来获取服务的代理对象。


#### Zygote是什么？有什么用？

  参考：[https://chowdera.com/2021/12/20211205120759493s.html](https://chowdera.com/2021/12/20211205120759493s.html)

  Zygote是Android系统中的孵化进程，由用户空间的第一个进程Init进程启动的，Zygote主要作用启动了SystemServer进程，和孵化启动应用程序进程。



#### 什么是Zygote预加载？

  预加载是指在Zygote进程启动的时候就加载一些类库和资源文件，这样系统只需要在第一次启动Zygote时加载这些共用的资源，子进程创建时只需要复用即可无需再次加载。


#### 系统中有几个Zygote进程？

  一般系统中有3个Zygote进程，zygote、zygote64和webview_zygote。zygote用来孵化32位的应用程序，zygote64用来孵化64位的应用程序，webview_zygote用来孵化webview进程。


#### 孵化应用进程这种事为什么不交给SystemServer来做，而专门设计一个Zygote？

  SystemServer里跑了一堆系统服务，这些系统服务是不能继承到应用进程的。所以SystemServer和应用进程里都要用到的资源抽出来单独放在一个进程里，也就是这的zygote进程，然后zygote进程再分别孵化出SystemServer进程和应用进程。


#### Zygote的IPC通信机制为什么使用socket而不采用binder？
  - Zygote通过fork来孵化进程，而fork采用的是CopyOnWrite机制，由于可能存在的死锁问题，Unix禁止fork一个多线程程序。Zygote当然也是多线程的，除了主线程外还有4条守护线程，每次fork前都需要停止这些线程，待fork结束后重新执行。
  - Zygote进程先于SystemServer创建，如果要使用Binder，那么需要等待SystemServer创建完成之后再向SystemServer注册Binder服务，这里需要额外的同步操作。

  Binder机制是需要建立Binder线程池的，代理对象对Binder的调用是在Binder线程池中，在通过线程间通信通知主线程。

  例如Activity启动时，AMS的本地代理IApplicationThread运行在Binder线程池中，处理完毕后通过Handler通知ActivityThread来执行启动Activity的流程。

  Zygote本身只需与SystemServer以及子Zygote进程通信，并不依赖多线程来提升性能，若使用Binder反而增加了Zygote中的线程数，使得性能下降。

  SystemServer不受到此限制，它并不需要fork自身来创建子进程，所以它会在第一时间初始化Binder线程池。

  ---

  1. 同步原因：Zygote进程先于SystemServer创建，如果要使用Binder，那么需要 Zygote 需要等待SystemServer创建完成之后再向SystemServer注册Binder服务，这里需要额外的同步操作。
  2. 性能原因：Binder 有16 个子线程来提升性能，而 Zygote本身只需与SystemServer以及子Zygote进程通信，并不依赖多线程来提升性能，若使用Binder反而增加了Zygote中的线程数，使得性能下降。
  3. 死锁原因：Zygote 采用 binder，binder 是多线程，在 Zygote fock 进程的时候，有可能出现死锁


#### 能说说fock具体是怎么导致死锁的吗？

  [https://www.cnblogs.com/liyuan989/p/4279210.html](https://www.cnblogs.com/liyuan989/p/4279210.html)

  我们知道通过fork创建的一个子进程几乎但不完全与父进程相同。子进程得到与父进程用户级虚拟地址空间相同的（但是独立的）一份拷贝，包括文本、数据和bss段、堆以及用户栈等。子进程还获得与父进程任何打开文件描述符相同的拷贝，这就意味着子进程可以读写父进程中任何打开的文件，父进程和子进程之间最大的区别在于它们有着不同的PID。  但是有一点需要注意的是，在Linux中，fork的时候只复制当前线程到子进程，在fork(2)-Linux Man Page中有着这样一段相关的描述：  

  > The child process is created with a single thread--the one that called fork(). The entire virtual address space of the parent is replicated in the child, including the states of mutexes, condition variables, and other pthreads objects; the use of pthread_atfork(3) may be helpful for dealing with problems that this can cause.  

  也就是说除了调用fork的线程外，其他线程在子进程中“蒸发”了。  

  这就是多线程中fork所带来的一切问题的根源所在了。

  ---

  fork() 时只会把调用线程（就是当前进程跑的线程）拷贝到子进程、其他线程都会立即停止且不会被拷贝，那如果一个线程在 fork() 前占用了某个互斥量（锁），fork() 后该线程被立即停止，这个互斥量（锁）就得不到释放，再去请求该互斥量就会发生死锁了。


####  Zygote的Socket和TCP的Sokcet一样吗？

  不一样。Zygote中使用的Socket是UNIX Domain Socket。UNIX Domain SOCKET 是在Socket架构上发展起来的用于同一台主机的进程间通讯（IPC）。它不需要经过网络协议栈，不需要打包拆包、计算校验和、维护序列号应答等。它只是在单机中，将应用层数据从一个进程拷贝到另一个进程。而我们平常讨论的Socket是：网络中不同主机上的应用进程之间的IPC通信，其基于的是TCP协议。










